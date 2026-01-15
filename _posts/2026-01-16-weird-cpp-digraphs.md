---
title:  "Weird C++ Syntax: Digraphs and Alternative Tokens"
date: 2026-01-15 23:00:00 +0900
categories: [Coding]
tags: english cpp trivia
toc: true
---

C++ is often criticized for its complex and sometimes cryptic syntax. However,
there's a "hidden" layer of syntax that many modern developers have never
encountered. If you ever see a C++ file that looks more like a weird mix of
Python and some ancient dialect, you might be looking at **digraphs** and
**alternative tokens**.

Take a look at this perfectly valid C++ code:

```cpp
%:include <iostream>

%:define PRINTLN(X)                               \
  do <%                                           \
    std::cout << %:X << ": " << (X) << std::endl; \
  %> while (0)

void print_and_modify(int bitand a) <%
  PRINTLN(a);
  a = 159;
%>
void print(int const and a) <%
    PRINTLN(a);
%>

int main() <%
  int a = 314;
  print_and_modify(a);
  print(static_cast<int and>(a));
  return 0;
%>
```

```shell
$ g++ -std=c++11 test.cc && ./a.out
a: 314
a: 159
```

At first glance, this looks like a syntax error in every single line. But it
compiles and runs perfectly. Let's break down why.

## What are Digraphs?

Digraphs are two-character sequences that are treated as a single punctuation
mark by the compiler.[^1] They were introduced to allow C++ programming on
systems with keyboards or character sets (like ISO 646) that lacked certain
characters required by the language.

| Digraph | Equivalent |
| :--- | :--- |
| `<%` | `{` |
| `%>` | `}` |
| `<:` | `[` |
| `:>` | `]` |
| `%:` | `#` |
| `%:%:` | `##` |

In our example, `%:include` is actually `#include`, and `<% ... %>` are just
curly braces `{ ... }`.

## Alternative Operator Tokens

C++ also provides keyword-based alternatives for many operators. These are not
macros; they are built into the language itself as primary tokens.

| Token | Operator |
| :--- | :--- |
| `and` | `&&` |
| `and_eq` | `&=` |
| `bitand` | `&` |
| `bitor` | `|` |
| `compl` | `~` |
| `not` | `!` |
| `not_eq` | `!=` |
| `or` | `||` |
| `or_eq` | `|=` |
| `xor` | `^` |
| `xor_eq` | `^=` |

In the code above:

- `int bitand a` is equivalent to `int &a` (a reference).
- `int const and a` is equivalent to `int const &&a` (an rvalue reference).
- `static_cast<int and>(a)` is `static_cast<int &&>(a)`, effectively moving the variable.

## What about Trigraphs?

You might have also heard of **trigraphs** (three-character sequences starting
with `??`). They served a similar purpose but were much more dangerous because
they were replaced by the preprocessor everywhere—even inside string literals!

For example, `??!` would become `|`. This caused so many bugs and confusion that
trigraphs were finally removed from the language in **C++17**.

## Why does this exist?

These features are fossils of an era when character encoding wasn't as
standardized as UTF-8 is today. While they are almost never used in professional
codebases now (except perhaps for some very specialized embedded systems or for
[obfuscated code contests](https://www.ioccc.org/)), they remain part of the
standard for backward compatibility.

Next time you want to confuse a colleague during a code review, maybe try
swapping out a few `&` for `bitand`.

---

[^1]: *For more details, check out the [cppreference page on alternative
      operators](https://en.cppreference.com/w/cpp/language/operator_alternative).*
