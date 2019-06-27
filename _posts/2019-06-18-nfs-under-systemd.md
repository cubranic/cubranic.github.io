---
title: NFS under systemd
---

I like _systemd_, I really do. Yes, it has occasional bugs (always annoying in
essential system services, and sometimes compounded by its developers' stubborn
refusal to accept that a bug report is a. a bug; and b. one that should be fixed
in _systemd_, not somewhere else), decidedly off-putting leadership, and an
arguably megalomanic tendency to re-implement every single system service (I'm
holding out for the fireworks when they take on email). But it's a sane design
that is far more robust and pleasant to operate than the collection of shell
scripts that is _init_. As Justin Cormack described it in his [talk][Linux2018]
"The operating system in 2018", a lot has changed in the way OSs are operated,
and firmly believe that it's overwhelmingly a good thing that _systemd_ was
designed with this kind of environment in mind.

Anyways, yesterday I had a chance to go down some dark byways of NFS and
_systemd_. Making old-style services fit the _systemd_ model can be a rather
procrustean exercise, as I've found out. I'm documenting my exploration here in
case it comes in handy to anyone else (or if you want to see how sausage is
made).

It all started with an NFS-related warning in the system log, "nfsd: too many
open connections, consider increasing the number of threads". How to do that,
you ask? Well, as the man page for `rpc.nfsd` describes it, it can be done by
just running `rpc.nfsd` with an argument for the number of threads (e.g.,
`rpc.nfsd 256`), and the number of system threads "will be increased or
decreased to match this number." Unfortunately, this change will be lost when
the service is restarted or node rebooted, .

But, since `rpc.nfsd` is run as the nfs-server service, let's change it there.
The unit file `nfs-server.service` has the following two lines:
```
EnvironmentFile=-/run/sysconfig/nfs-utils
ExecStart=/usr/bin/rpc.nfsd $RPCNFSDARGS
```

Now, `/run/sysconfig/nfs-utils` contains the line `RPCNFSDARGS=" 8"`, so this
means that this file holds the configuration settings that will be passed to
`rpc.nfsd` as arguments at startup. Unfortunately, as `/run` is a temporary
filesystem, any changes to it will also not survive a reboot. (Actually, in this
case, they won't survive the restart of `nfs-service` either, read on for why.)
So it has to be written or copied from some template elsewhere. 

A bit of searching led me to script `nfs-utils_env.sh` (in
`/usr/lib/systemd/scripts`). This script sources `/etc/sysconfig/nfs`[^Ubuntu],
sets undefined variables to their defaults and writes them out to
`/run/sysconfig/nfs-utils` in format accepted by the NFS-related .services units
as the `EnvironmentFile`. (In particular, `RPCNFSDARGS` is a concatenation of
`RPCNFSDARGS`, `NFSD_V4_GRACE`, `NFSD_V4_LEASE`, and `RPCNFSDCOUNT`.)
        
We are almost there: `/etc/sysconfig/nfs` is obviously in the standard
place for configuration files, so my setting should go there. It is even nicely
populated, on Centos 7 at least, with all available settings documented, and
commented out so defaults take place. Sure enough, it includes:
```
#RPCNFSDCOUNT=8
```
so all I need to do is change that line with a higher value, re-run
`nfs-utils_env.sh` to regenerate `/run/sysconfig/nfs-utils`, and then restart
`nfs-server`. Actually, the config file suggests restarting
`nfs-config.service`. What's that? As it's unit file describes, it's a
prerequisite for `nfs-server`, a one-shot service that just runs our old friend
`nfs-utils_env.sh`. And since it's a prerequisite, restarting it will also
restart `nfs-server`. (And, a corner of _systemd_ I hadn't encountered before, its
being a "one-shot" service with "RemainAfterExit=no" setting means that it will
be re-run on "nfs-server" restart, too, so either one would work.)
      
And that is where we end up. It was a little roundabout way to end up just
editing a line in a config file in `/etc`, but now I understand how it fits with
the working system. It also seems a very roundabout way to relay configuration
to a service, but that's how it goes as we slowly transition from the
accumulated cruft of decades of Unix to the warm, all-encompassing _systemd_
future.

[Linux2018]: https://qconsf.com/sf2018/presentation/operating-system-2018

[^Ubuntu]: On Ubuntu, this is done after sourcing /etc/defaults/nfs-common and nfs-kernel-server
