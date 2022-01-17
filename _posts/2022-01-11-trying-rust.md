---
layout: post
title: Trying out Rust by diving deep
date: 2022-01-11
categories:
  - Blog
tags:
  - Rust
  - Code
  - PRNG
---
It is a new year, and I hope everyone is doing well; I did something I hoped I would never have to do; Give the programming language Rust a chance. I was an anti-Rust guy for the last 3 or so years, mostly because I was put off with how forward and, in some cases, overly zealous the community was/is. They would say frankly offputting things like "Algebraic type system," "The rust compiler is all-knowing!", or "If it compiles, it is correct!". As someone whose primary systems programming language is C++, this irked me. The perceived community message was that all other systems languages were bad, buggy and riddled with undefined behavior. While there are legitimate criticisms about the Rust ``unsafe`` vs. ``safe`` code (I still have many), these are mainly overblown. So last winter, I gave it a legitimate shot, porting my library ``SmallPRNG`` from C++ to Rust. The original library is approximately 700 lines of template metaprogramming [SFINAE](https://en.cppreference.com/w/cpp/language/sfinae) shenanigans to get high performance, so this is not a 'simple' job and forced me to interact with many features (and engineered limitations) of Rust.

As it turns out, Rust is a delight to program in (after you are done fighting the borrow checker), and its expressiveness is on par with C++. It took approximately 10 days from start to end to properly translate the C++ template metaprogramming in an idiomatic Rust style while still keeping the compile-time simplifications of the metaprogramming. In Rust, the template-like features are not quite as fleshed out as in C++, but there is active development on that front, so hopefully, it will catch up someday. Rust forced me into design decisions that were ultimately better (not that I didn't resent it at the time), and while there is a bit more code duplication, I appreciate the reasons these design rules are held.

An example of this duplication can be seen here; in the C++ code, I only have to define one templated state struct (with a specialization). So technically, if I used this object in any other way than the specific way I used it, then according to the C++ standard, I would be in [undefined behavior land](https://en.cppreference.com/w/cpp/language/union#:~:text=It's%20undefined%20behavior%20to%20read,inactive%20members%20of%20a%20union.&text=Each%20member%20is%20allocated%20as,only%20member%20of%20the%20class.). 

```C++	
template<int N>
struct prng_state {
	static_assert(N > 0, "prng_state must have a postive amount of memory, for prng_state<N> N >=1 ");
	union {
		uint16_t i16[N * 2];
		uint32_t i32[N];
		uint64_t i64[N >> 1];
	};
};

```

However, in Rust, I have to define this state struct for every algorithm I wish to implement. Otherwise, I would have to hit an ``unsafe`` code blocks every time I wanted to access the state. However, this wasn't really a problem as the code duplication is minimal. It also removed the need to have the prng state be mappable to 32-bit chunks of unsigned ints.

```rust
pub struct LCG {
    pub(crate) data: u64,
}
```

Not to get too far in the weeds with specific implementation details made the code a LOT more modular and much more straightforward to grock with. Check out ``SmolPRNG`` on [crates.io](https://crates.io/crates/smolprng). 

There are two killer features of Rust that I don't see talked about anywhere, and that is embedding benchmarks and unit tests a first-class feature. To write a microbenchmark of code, all that needs to be done is to decorate a function with ``#[bench]``. Likewise, to make a unit test, just decorate with ``#[test]``. This makes test-driven development dead simple and tracking performance regressions immediate. I implemented everything from SmallPRNG, added tests, and benches with only 300 more lines for everything, including documentation comments.

If you have read this rambling mess to this point, you might notice that my attitude has shifted somewhat towards Rust. While I think there are some fundamental problems with the ``safe`` vs. ``unsafe`` model, such as just because you wrapped a bomb in a wet paper sack doesn't mean the [bomb](https://doc.rust-lang.org/std/env/fn.remove_var.html) is now gone. I have run into at least one common compiler bug and another [compiler qwerk](https://github.com/rust-lang/rust/issues/92347). That being said, the massive push towards removing memory bugs as a type of possible error is very worthwhile, as something on the order of 70% of bugs are memory bugs. So while Rust is not the panacea for all ills, it does remove the majority of the bugs while keeping lightning-fast performance.
