- Feature Name: `complex-numbers`
- Start Date: 2022-11-13
- RFC PR: [rust-lang/rfcs#0000](https://github.com/rust-lang/rfcs/pull/0000)
- Rust Issue: [rust-lang/rust#0000](https://github.com/rust-lang/rust/issues/0000)

# Summary
[summary]: #summary

Add support for complex numbers in the standard library.

# Motivation
[motivation]: #motivation

Plenty of calculations rely on complex numbers, to the point where it can be considered a common object in mathematics.  
As such, having complex numbers in the standard library would be natural.

And, while there already exists implementation of complex numbers outside of the standard library, they are harder to find; having them in the standard library could help prevent new developers from reinventing the wheel.

Finally, having "standard library privilege" can allow complex numbers to make use of syntactic sugar for ease of use.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

## Basics

There exists a complex equivalent to every numeric type, named by prefixing a `c` to the type.  
E.g. `cu8`, `ci32`, `cf64`, ...

Complex numbers are composed of two numbers, a real and an imaginary part :
```rs
struct cu32 {
	real: u32,
	imag: u32,
}
```

A complex value can be constructed from literals by adding (or subtracting) the imaginary part to real part:
```rs
let a = 3+2i;
let b = 2-8i;
let c = 3.7+2.9i;

let d = 1 + 0i; // Does not compile, spaces makes it harder to read as a single value
let e = 2.5+3i; // Does not compile, mixes float and integer
```

The type can be explicitly declared by either specifying the variable's type, or by specifying the real part's type:  
```rs
let f: cu16 = 7-2i;
let g: cf32 = 8+1i;
let h = -7u128-1i;

let i = 6+1iu32; // Does not compile, ambiguous and hard to read
```

Both the real and imaginary part **NEED** to be present, even if one of them is zero:
```rs
let j: cu32 = 8; // Does not compile, missing the imaginary part
let k: cf64 = 9.5i; // Does not compile, missing the real part

let l: cu32 = 8+0i;
let m: cf64 = 0+9.5i;
```

You can also construct a complex value from variables, albeit in a less terse form:
```rs
let n = cu32::new(real_part, imag_part);
```

## Mathematical point of view

Complex numbers implement most traits that would make sense from a mathematical point of view (and do not implement those who don't make sense):

- You are able to perform arithmetic operations between them
  ```rs
  let o = 3-2i + -6+3i; // Equal to -3+1i
  let p = 2+3i * 4-1i; // Equal to 11+10j
  ```
- They implement `PartialEq` and integer complex implement `Eq` as well
  ```rs
  assert_eq!(1+3i, 1+3i);
  assert_eq!(2.5+1.2i, 2.5+1.2i);
  ```
- They **DO NOT** implement `PartialOrd`
  ```rs
  assert!(2+3i > 4-2i); // Does not compile
  ```

Explain the proposal as if it was already included in the language and you were teaching it to another Rust programmer. That generally means:

- Introducing new named concepts.
- Explaining the feature largely in terms of examples.
- Explaining how Rust programmers should *think* about the feature, and how it should impact the way they use Rust. It should explain the impact as concretely as possible.
- If applicable, provide sample error messages, deprecation warnings, or migration guidance.
- If applicable, describe the differences between teaching this to existing Rust programmers and new Rust programmers.
- Discuss how this impacts the ability to read, understand, and maintain Rust code. Code is read and modified far more often than written; will the proposed feature make code easier to maintain?

For implementation-oriented RFCs (e.g. for compiler internals), this section should focus on how compiler contributors should think about the change, and give examples of its concrete impact. For policy RFCs, this section should provide an example-driven introduction to the policy, and explain its impact in concrete terms.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

This is the technical portion of the RFC. Explain the design in sufficient detail that:

- Its interaction with other features is clear.
- It is reasonably clear how the feature would be implemented.
- Corner cases are dissected by example.

The section should return to the examples given in the previous section, and explain more fully how the detailed proposal makes those examples work.

# Drawbacks
[drawbacks]: #drawbacks

This could be considered standard library bloat by some.

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

- Why is this design the best in the space of possible designs?
- What other designs have been considered and what is the rationale for not choosing them?
- What is the impact of not doing this?
- If this is a language proposal, could this be done in a library or macro instead? Does the proposed change make Rust code easier or harder to read, understand, and maintain?

# Prior art
[prior-art]: #prior-art

Discuss prior art, both the good and the bad, in relation to this proposal.
A few examples of what this can include are:

- For language, library, cargo, tools, and compiler proposals: Does this feature exist in other programming languages and what experience have their community had?
- For community proposals: Is this done by some other community and what were their experiences with it?
- For other teams: What lessons can we learn from what other communities have done here?
- Papers: Are there any published papers or great posts that discuss this? If you have some relevant papers to refer to, this can serve as a more detailed theoretical background.

This section is intended to encourage you as an author to think about the lessons from other languages, provide readers of your RFC with a fuller picture.
If there is no prior art, that is fine - your ideas are interesting to us whether they are brand new or if it is an adaptation from other languages.

Note that while precedent set by other languages is some motivation, it does not on its own motivate an RFC.
Please also take into consideration that rust sometimes intentionally diverges from common language features.

# Unresolved questions
[unresolved-questions]: #unresolved-questions

- Should `j` be used in place of `i` as the specifier for the imaginary part (like Python)?  
  Using `im` could also be considered (like Julia)?

- Should the name of types reflect the space taken by the entire value, instead of the space taken by its parts?  
  E.g. Two `u8` forming a `cu16`, instead of a `cu8`

- What parts of the design do you expect to resolve through the RFC process before this gets merged?
- What parts of the design do you expect to resolve through the implementation of this feature before stabilization?
- What related issues do you consider out of scope for this RFC that could be addressed in the future independently of the solution that comes out of this RFC?

# Future possibilities
[future-possibilities]: #future-possibilities

Quaternions could be considered for the standard library as well; mostly because of their usage in 3D physics computation.  

The same goes for other mathematical objects, such as 2D/3D points or vectors.

Overall, a new base module `math` could be considered for these new objects, needing to be explicitly included in a project's `Cargo.toml`, in order to minimize standard library bloat while still giving a standard implementation.
