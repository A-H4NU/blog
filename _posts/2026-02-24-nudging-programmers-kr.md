---
title:  "프로그래머를 가스라이팅하라"
date: 2026-02-24 00:30:00 +0900
categories: [Coding]
tags: korean coding
toc: true
---

<!--toc:start-->
- [서론](#서론)
- [넛지(Nudge)](#넛지nudge)
- [문법적 가스라이팅의 예시들](#문법적-가스라이팅의-예시들)
  - [C++: static_cast](#c-static_cast)
  - [Rust: unsafe](#rust-unsafe)
  - [Go: if err != nil](#go-if-err--nil)
- [결론](#결론)
<!--toc:end-->

> There is an [English version]({% post_url 2026-02-24-nudging-programmers-en %}) of this post.
{: .prompt-info }

## 서론

야생의 프로그래머들은 위험하다. 대부분의 프로그래머들은 그 언어의 (엄밀한
의미에서의) 구문(syntax)[^1]과 의미(semantics)[^2]를 완전히
이해하지 않고 '사용할 수 있는 결과물'을 원한다. 그리고 이것은 전혀 나쁜 것이
아니며 오히려 프로그래밍 언어의 설계자들이야말로 그 언어의 사용자의 생태를
이해해야 할 필요가 있는 것이다.

하지만 프로그래밍 언어는 기계가 어떻게 동작해야 할지를 기술하는 것인데,
그 프로그래머가 그 언어의 명세(specification)에 따라 정확한 코드를 작성하지 않으면
'인간과 기계의 소통 수단'을 사용하는 데에 어떤 의미가 있다는 말인가? 프로그래밍
언어의 설계자들은 결국 잘못 짜인 코드를 두고 그 코드를 작성한 프로그래머를
비난하는 것 외에 아무것도 할 수 없다는 말인가? 만약 세상에 신이 있다면, 이것은
결코 현실이 아닐 것이다.

예시로, 다음 C 코드를 보자:

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

이 C 프로그램의 결과물(`result`)는 무엇일까? 아마 대부분 답을 2나 3으로 했겠지만
정답은 '둘 다'이다.[^3] (실제로 필자의 컴퓨터에서 `gcc`로 컴파일했을 때는 `3`,
`clang`으로 컴파일했을 때는 `2`가 출력되었다.) 왜냐하면 7번째 줄에서 `add`를
호출할 때 두 인수인 `x++`와 `++x` 중 무엇을 먼저 계산해야 하는지 정해져 있지
않기 때문이다.

이번에는 `gcc`와 `clang`을 통한 컴파일 경험을 살펴보자:

```console
$ gcc ub.c

$ clang ub.c
ub.c:7:23: warning: multiple unsequenced modifications to 'x' [-Wunsequenced]
    7 |     int result = add(x++, ++x);
      |                       ^   ~~
1 warning generated.
```

`gcc`는 이 '모호한 코드'를 보고 임의로 인자의 계산 순서를 선택하여 조용히
컴파일을 완료하였다. 반면 `clang`은 프로그래머에게 경고를 출력하였다. (물론
`-Wall` 플래그를 통해 `gcc`에서도 비슷한 경고를 활성화할 수 있다.) 프로그래머의
사용자 경험으로 볼 때, 이 부분에 대해서는 `gcc`보다 `clang`을 썼을 때 "자신의
의도와 같은 프로그램"을 작성할 가능성이 높은 것이다. 그런 의미에서, 노련한 C
프로그래머들은 컴파일러에 `-Wall -Wextra -pedantic` 플래그를 추가해 의도하지
않은 효과를 최대한 방지한다.

## 넛지(Nudge)

행동경제학의 [넛지(Nudge)](https://ko.wikipedia.org/wiki/넛지_이론)는 강제성
없이 선택 설계(choice architecture)를 조작하여 개인의 행동을 설계자가 원하는
방향으로 유도하는 기법이다. 프로그래밍 언어와 컴파일러 설계에서 이 넛지는 주로
'의도된 마찰'의 형태로 구현된다.

설계자는 프로그래머가 위험하거나 비관용적인(non-idiomatic) 패턴을 작성하려 할
때, 해당 구문의 타이핑을 의도적으로 번거롭게 만들거나 코드를 시각적으로 흉측하게
만든다. 이는 명시적인 금지나 오류가 아님에도 불구하고, 프로그래머가 무의식적으로
더 안전한 우회로를 선택하거나 자신의 코드 설계를 재고하도록 유도한다.
표면적으로는 자유를 제공하는 듯하지만, 실질적으로는 언어 설계자의 철학에 맞춰
프로그래머의 사고방식을 통제하고 교정하는 **정교한 시스템적 가스라이팅**으로
기능한다.

## 문법적 가스라이팅의 예시들

### C++: static_cast

C++의 창시자 Bjarne Stroustrup은 C++의 캐스팅 연산자들을
의도적으로 길고, 타이핑하기 귀찮고, 시각적으로 흉측하게 만들었다.[^4]

C 언어 스타일의 캐스팅은 간결하다.

```c
int *p = (int *)malloc(sizeof(int));
```

하지만 이 `(type)` 문법은 너무 강력해서 `const`를 날려버리든, 전혀 다른 타입으로
비트 단위 재해석을 하든 묵묵히 수행한다. 실수를 저지르기 딱 좋다.

반면 C++는 이를 세분화하고 타이핑 비용을 높였다.

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

`static_cast`나 `reinterpret_cast`를 타이핑하면서 프로그래머는 필연적으로
멈칫하게 된다. "내가 지금 하려는 게 타입 시스템을 거스르는 짓인가?"를 자문하게
만드는 것이다. 또한, 코드가 터졌을 때 grep으로 범인을 찾기도 훨씬 쉽다. 이것은
언어 설계자가 프로그래머의 게으름을 방지하기 위해 심어둔 의도된 마찰이다.

### Rust: unsafe

Rust는 메모리 안전성을 보장한다고 자랑하지만, 시스템 프로그래밍을 하다 보면 어쩔
수 없이 그 규칙을 깨야 할 때가 온다. 이때 Rust는 `unsafe` 키워드를 강제한다.

```rust
let mut num = 5;

let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;

unsafe {
    println!("r1 is: {}", *r1); // r1을 dereference할 수 있어야 함
    *r2 += 1;                   // r2가 가리키는 대상이 mutable해야 함
    println!("r2 is: {}", *r2);
}
```

이 unsafe 블록은 기술적인 기능이라기보다는 심리적 저지선에 가깝다. "여기서부터
발생하는 모든 메모리 오류는 컴파일러 탓이 아니라 전적으로 네 탓이다"라는
서약서에 서명하는 것과 같다. 프로그래머는 이 블록을 작성하며 긴장감을 느끼고,
코드 리뷰어는 이 부분만 눈에 불을 켜고 감시하게 된다. 또한, 만약 프로그램에
메모리 오류 (`segfault` 등)이 발생하면, 그것은 아주 높은 확률로 `unsafe` 블록 안의
코드가 잘못되었다는 것을 의미하기 때문에 디버깅할 때에도 편리하다.

### Go: if err != nil

대부분의 현대 언어는 예외(Exception) 처리를 통해 에러를 '우아하게' 건너뛸 수
있게 해준다. `try`-`catch`로 감싸지 않으면 에러는 상위로 전파되거나 런타임에
터지지만, 코드의 흐름 자체는 깔끔해 보인다.

하지만 Go 언어는 예외가 없다. 함수는 결과값과 에러를 동시에 반환하고,
프로그래머는 숨 쉬듯이 `if err != nil`을 타이핑해야 한다.

```go
f, err := os.Open("file.bin")
if err != nil {
    log.Fatal(err)
}
```

Go 프로그래머들은 이 패턴이 지겹다고
[불평하지만](https://www.reddit.com/r/golang/comments/11ct9ss/reducing_if_err_nil/),
이는 "성공한 상황만 가정하지 말라"는 강력한 [세뇌
교육](https://www.reddit.com/r/golang/comments/11ct9ss/comment/ja6c025/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button)이다.
에러를 명시적으로 변수에 담고, 그것이 nil인지 확인하는 과정을 강제함으로써,
프로그래머가 에러 처리를 '나중에 할 일'로 미루는 것을 원천 봉쇄한다.

## 결론

결국, 좋은 프로그래밍 언어란 단순히 기계에게 명령을 내리는 도구가 아니다. 그
언어는 불완전한 인간(프로그래머)의 사고방식을 교정하고, 위험한 충동을 억제하며,
올바른 길로 가도록 끊임없이 간섭하고 가스라이팅하는 도구여야만 하는 것이다.

아래는 필자가 흥미롭게 읽은 관련 논문들이다:

- [Nudging Students Toward Better Software Engineering Behaviors](https://arxiv.org/abs/2103.09685)
- [Nudging Software Developers Toward Secure Code](https://ieeexplore.ieee.org/document/9740708)

---

[^1]: <https://ko.wikipedia.org/wiki/구문_(프로그래밍_언어)>
[^2]: <https://ko.wikipedia.org/wiki/의미론_(컴퓨터_과학)>
[^3]: <https://en.wikipedia.org/wiki/Sequence_point>
[^4]: <https://www.stroustrup.com/bs_faq2.html#static-cast>
