---
title:  "Gaslight Programmers"
date: 2026-02-24 00:30:00 +0900
categories: [Coding]
tags: english coding
toc: true
---

> [한국어 버전]({% post_url 2026-02-24-nudging-programmers-kr %})이 있습니다.
{: .prompt-info }

## Introduction

Wild programmers are dangerous. Most programmers desire a 'usable output'
without fully understanding the syntax[^1] and semantics[^2] (in the strict
sense) of the language. This is not a negative trait; rather, it is the
programming language designers who must understand the ecology of the language's
users.

However, a programming language dictates how a machine should operate. If the
programmer does not write accurate code according to the language's
specification, what meaning remains in utilizing this 'means of communication
between human and machine'? Does this mean programming language designers can do
nothing but criticize the programmer who wrote the malformed code? If a god
exists in this world, this would never be reality.

As an example, consider the following C code:

```c
int add(int a, int b) {
    return a + b;
}

int main(void) {
    int x = 0;
    int result = add(x++, ++x);
    return result;
}
```

{: file='ub.c'}

What is the result (`result`) of this C program? Most would likely answer 2 or
3, but the correct answer is 'both'.[^3] (In practice, compiling with `gcc` on
the author's computer output `3`, whereas compiling with `clang` output `2`.)
This occurs because when calling `add` on line 7, the evaluation order of the
two arguments `x++` and `++x` is undefined.

Examine the compilation experience using `gcc` and `clang`:

```console
$ gcc ub.c

$ clang ub.c
ub.c:7:23: warning: multiple unsequenced modifications to 'x' [-Wunsequenced]
    7 |     int result = add(x++, ++x);
      |                       ^   ~~
1 warning generated.
```

`gcc` observed this 'ambiguous code', arbitrarily selected an argument
evaluation order, and silently completed compilation. Conversely, `clang` output
a warning to the programmer. (A similar warning can be enabled in `gcc` via the
`-Wall` flag.) From the perspective of the programmer's user experience, using
`clang` over `gcc` for this scenario yields a higher probability of writing a
"program identical to their intent". In that context, veteran C programmers
append `-Wall -Wextra -pedantic` flags to the compiler to prevent unintended
effects.

## Nudge

[Nudge](https://en.wikipedia.org/wiki/Nudge_theory) in behavioral economics is a
technique that manipulates choice architecture without coercion to direct an
individual's behavior toward a designer's intended direction. In programming
language and compiler design, this nudge is predominantly implemented as
'intended friction'.

When a programmer attempts to write a dangerous or non-idiomatic pattern, the
designer intentionally makes the syntax cumbersome to type or the code visually
repulsive. Although not an explicit prohibition or error, this subconsciously
forces the programmer to select a safer alternative or reconsider their code
architecture. While ostensibly offering freedom, practically, it functions as
**sophisticated systemic gaslighting** that controls and recalibrates the
programmer's cognitive process to align with the language designer's philosophy.

## Examples of Syntactical Gaslighting

### C++: static_cast

Bjarne Stroustrup, the creator of C++, intentionally engineered C++'s casting
operators to be long, cumbersome to type, and visually
repulsive.[^4]

C-style casting is concise.

```c
int *p = (int *)malloc(sizeof(int));

```

However, this `(type)` syntax is excessively powerful, silently executing
operations whether stripping `const` or reinterpreting bits into a completely
different type. It is optimized for generating errors.

In contrast, C++ granularized this and increased the typing cost.

```c++
int x = 63;
double d = static_cast<double>(x); // safe
std::cout << d << std::endl;

const int cx = x;
int &rx = const_cast<int &>(cx); // removing const
rx += 1;
std::cout << rx << std::endl;

int *ptr = &x;
char *cptr = reinterpret_cast<char *>(ptr); // dangerous!
std::cout << *cptr << std::endl;

return 0;
```

While typing `static_cast` or `reinterpret_cast`, the programmer inevitably
pauses. It forces the self-inquiry: "Is what I am attempting an act that defies
the type system?" Furthermore, when code fails, isolating the fault with grep is
significantly streamlined. This represents intended friction embedded by the
language designer to counteract programmer negligence.

### Rust: unsafe

Rust markets memory safety guarantees, but during system programming, rules must
occasionally be bypassed. In these instances, Rust mandates the `unsafe`
keyword.

```rust
let mut num = 5;

let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;

unsafe {
    println!("r1 is: {}", *r1); // must be able to dereference r1
    *r2 += 1;                   // the target pointed to by r2 must be mutable
    println!("r2 is: {}", *r2);
}
```

This unsafe block operates more as a psychological barrier than a technical
feature. It functions as signing a contract stating: "Every memory error
originating from this perimeter is entirely your fault, not the compiler's." The
programmer experiences tension while constructing this block, and the code
reviewer scrutinizes this specific section with elevated vigilance.
Additionally, if a memory error (`segfault`, etc.) triggers in the program, it
is highly probable that the code inside the `unsafe` block is defective,
isolating the debugging target.

### Go: if err != nil

Most modern languages permit bypassing errors 'gracefully' via Exception
handling. If omitted from a `try`-`catch` block, the error propagates upwards or
crashes at runtime, yet the visual code flow remains uncluttered.

The Go language lacks exceptions. Functions return a result value and an error
simultaneously; the programmer must type `if err != nil` reflexively.

```go
f, err := os.Open("file.bin")
if err != nil {
    log.Fatal(err)
}
```

Go programmers
[complain](https://www.reddit.com/r/golang/comments/11ct9ss/reducing_if_err_nil/)
that this pattern is tedious, but it is a rigorous [brainwashing
education](https://www.reddit.com/r/golang/comments/11ct9ss/comment/ja6c025/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button)
dictating: "do not assume only the successful scenario." By enforcing the
explicit assignment of the error to a variable and verifying if it is nil, it
structurally blocks the programmer from deferring error handling.

## Conclusion

A competent programming language is not merely a tool for issuing machine
commands. The language must serve as an instrument that corrects the cognitive
framework of the flawed human (programmer), suppresses hazardous impulses, and
continuously interferes and gaslights to direct them onto the correct
trajectory.

Below are related papers the author reviewed with interest:

- [Nudging Students Toward Better Software Engineering Behaviors](https://arxiv.org/abs/2103.09685)
- [Nudging Software Developers Toward Secure Code](https://ieeexplore.ieee.org/document/9740708)

---

[^1]: <https://en.wikipedia.org/wiki/Syntax_(programming_languages)>
[^2]: <https://en.wikipedia.org/wiki/Semantics_(programming_languages)>
[^3]: <https://en.wikipedia.org/wiki/Sequence_point>
[^4]: <https://www.stroustrup.com/bs_faq2.html#static-cast>
