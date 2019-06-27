---
title: C is not (any longer) a low-level language
---

I loved David Chisnall [article][] in July 2018 CACM "C Is Not a Low-Level
Language". The subtitle, "And your computer is not a fast PDP-11", says it all.

> [Low-level languages] should be easy to translate into fast code without
> requiring a particularly complex compiler. … It's easy to argue that C was a
> low-level language for the PDP-11. They both described a model in which
> programs executed sequentially, in which memory was a flat space, and even the
> pre- and post-increment operators cleanly lined up with the PDP-11 addressing
> modes. [...] Since then, implementations of C have had to become increasingly
> complex to maintain the illusion that C maps easily to the underlying hardware
> and gives fast code.

This has real-world costs, as well as negative consequences:


> In spite of the heroic efforts that processor architects invest in trying to
  design chips that can run C code fast, the levels of performance expected by C
  programmers are achieved only as a result of incredibly complex compiler
  transforms.
  
> Both [Meltdown and Spectre] vulnerabilities involved processor … features that
> … were added to let C programmers continue to believe they were programming in
> a low-level language, when this hasn't been the case for decades.

The fact that fixes for these vulnerabilities have a 25% performance penalty,
and turn the last decade of work on simultaneous multi-threading (SMT) into a
dead end might be a blessing in disguise. As Chisnall writes:

> Perhaps it is time to stop trying to make C code fast and instead think about
> what programming models would look like on a processor designed to be fast.

There are research CPUs going back a couple of decades that have explored
alternatives to the high-ILP approach Intel has taken---not to mention the
GPUs---and we know the general direction we should go:

> A processor designed purely for speed, not for a compromise between speed and
> C support, would likely support large numbers of threads, have wide vector
> units, and have a much simpler memory model. 

Other design avenues mentioned in the article were news to me, but all the more
exciting: simplifying cache coherence protocols by limiting mutability to
thread-local variables, or even combining garbage collection with cache.

Unfortunately, as far as seeing this kind of processor in the real world,
Chisnall concludes with a cautionary:

> Running C code on such a system would be problematic, so, given the large
> amount of legacy C code in the world, it would not likely be a commercial
> success.

[article]: https://cacm.acm.org/magazines/2018/7/229036-c-is-not-a-low-level-language/fulltext
