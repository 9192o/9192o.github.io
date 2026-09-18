---
title: "[Jungle] 함수 포인터와 다형성, 그리고 vtable"
date: 2026-09-18 00:00:00 +0900
categories:
  - Krafton Jungle
tags:
  - jungle-13
  - c
  - cpp
  - function-pointer
  - polymorphism
  - vtable
description: 함수 포인터와 콜백에서 시작해 C의 VTable과 C++ 가상 함수의 동작 방식까지
---

## 시작하며

> "함수가 호출되면, 내부의 PC 레지스터는 거기로 어떻게 찾아갈까?"

`call add`와 `call rax`. `add()` 함수의 주소를 알아내 거기로 PC를 옮기는 것과, `rax`로 PC를 옮기는 것...? `rax`? 
## 함수 포인터

> "함수도 주소고, 함수 포인터도 포인터다."

변수의 주소를 포인터에 저장하는 건 이제 익숙한 것 같다.

```C
int x = 10;
int *p = &x; 
```

`x`라는 변수에 10을 저장, 그리고 `x의 주소`를 포인터 변수 `p`에 저장한다. 쉽게 말해서 포인터 변수 `p`는 "`x`가 있는 메모리" 를 가리키고 있다.

> "그렇다면 함수도 실행 파일 영역 어딘가에 저장되어있을 텐데, 주소가 있지 않을까? 그러면 그걸 가리키는..."

```C
int add(int a, int b)
{
	return a + b;
}
```

간단히 `int a + int b`를 반환하는 함수를 정의하고, `add`함수의 주소를 `0x12345678` 이라고 가정해보자. 이를 가리키는 포인터는 어떻게 만들까? 먼저 `int x`를 가리키는 포인터는 `p`에 `int *`를 씌워서 만들었으니까...

```C
// 먼저 fp 라는 변수를 선언해보자.
fp
|
// 그 다음 fp를 포인터 변수로...
*fp
|
// 그 다음, 함수니까, 인자를 받아야 하고...
*fp(int, int) // add는 int 두 개를 받으니
|
// 반환값도 넣어야 하고...
int *fp(int, int) // add는 int를 반환하니
|
// 뭔가 이상한데...
```

가 될 것 인데, 문제는 이러면

> "`fp`는 `int *`를 반환하는 함수다!"

될테니 결국 "나 포인터 변수에요!" 하고 괄호를 씌워야 할 것이다. 그리고 선언과 할당을 해보면

```C
// 나는 "add 함수"를 가리키는 함수 포인터다!
int (*fp)(int, int) = &add;
```

또, 함수의 이름은 사실 **대부분의 표현식에서 해당 함수를 가리키는 함수 포인터로 표현된다**.
```C
int (*fp)(int, int) = add; // 이렇게 선언해도 된다.
```

그럼 작성법은 다음과 같을 것이다.

```C
<반환형>(*포인터변수)(인자1, 인자2, ...) = <함수>

int (*fp)(int, int) = add;
```

> "반환형과, 매개변수 타입을 가리켜야 하니, 다르면 포인터에 할당 못하겠구나!"

```C
double div(double a, double b)
{
	if (b != 0)
		return a / b;
	else
		return a;
}

int main(void)
{
	int (*fp)(int, int) = div; // 안된다!
	double (*fp)(double, double) = div; // 된다!
}
```

그리고 사용법은 함수랑 똑같다!

```C
int add(int a, int b)
{
	return a + b;
}

int main(void)
{
	int (*fp)(int, int) = add;
	
	int x = fp(10, 20); // 인라인 쓰는 느낌
}
```

## Callback(콜백) 그리고 다형성

> "포인터도 만들었으니, 함수를 인자로 받아서 쓰고 싶다!"

이제 함수를 포인터로 받는 방법을 알았으니, **"인자"**로 받아보자.

```C
void hello(void)
{
	printf("Hello World!\n");
}
```

`Hello World!`를 출력하는 함수 `hello`를 만들었으니, 함수 포인터를 이용해서 다른 함수에 넣어보자.

```C
// 해당 함수는 반환형: void, 인자: void 니까...
void (*callback)(void)
	|
// 이를 인자에 넣자!
void execute(void (*callback)(void))
	|
// 그리고 함수 안에서 실행시키자!
void execute(void (*callback)(void))
{
	callback();
}
```

```C
Hello World!
```

사실 이게 **콜백 함수**다. 그냥 **"함수를 다른 함수에게 인자로 넘겨주고, 그 함수가 필요할 때 호출하도록"** 한다.

`main()` 에서 `execute()` 함수한테 `hello()` 함수 주소를 넘김 -> `callback` 인자에 `hello()` 함수 저장 -> 필요할 때 `callback()` 형태로 호출.

> "그렇다면 인자를 다양하게, 그러니까 함수를 다양하게 넘길 수 있겠네?"

따라서 다음이 가능하다.

```C
execute(hello); // Hello World! 출력
execute(goodbye); // Goodbye! 출력
execute(print_something); // "something" 출력
```

보다보면 곧 `execute()` 함수는...

> "같은 함수를 다르게 쓸 수는 없을까?"

같은 질문에 기반한 함수처럼 보인다. `execute()`라는 동일한 코드인데 **넘겨받은 함수에 따라 동작이 바뀐다**.

이를 더 확장시켜보자.

```C
#include <stdio.h>

int add(int a, int b)
{
	return a + b;
}

int sub(int a, int b)
{
	return a - b;
}

int mul(int a, int b)
{
	return a * b;
}

double divide(double a, double b)
{
	if (b != 0)
		return a / b;
	else
		return a;
}

int main(void)
{
	printf("%d", add(10, 20));
	printf("%d", sub(10, 20));
	printf("%d", mul(10, 20));
	printf("%lf", divide(10, 20));
	return 0;
}
```

더하기 프로그램을 확장시켜서 빼기, 곱하기, 나누기가 가능한 프로그램을 만들었다.

만약 여기서 요즘 계산기 프로그램처럼 여러 기능까지 넣어버리면, "인자 두 개를 받아 여러 가지 연산" 을 하는 함수들이 더 늘어날 것이다. 그리고 이렇게, `add(10, 20)`, `sub(10, 20)`...일일히 호출하고 싶지 않다. 

이제 배웠던 형태로 바꿔보자. `calculate()`함수를 하나 만들어서 콜백 함수를 넘기는 형태로.

```C
int calculate(int a, int b, int (*callback)(int, int))
{
	return callback(a, b); // 연산 함수를 받아서 실행시킨 값을 반환한다.
}
```

```C
// calculate 하나로 다 된다...
printf("%d", calculate(10, 20, add);
printf("%d", calculate(10, 20, sub);
printf("%d", calculate(10, 20, mul);

// 얘는 인자가 다르니까 안된다!
// printf("%lf", calculate(10, 20, divide));
```

이제 `calculate()` 함수에 인자와 각 연산 함수를 넘기는 것으로, 호출 형식을 통일했다! 그리고 
`calculate()` 함수는 "나는 너가 더하는지, 빼는지, 뭘 하는지 모르긴하는데, 형식은 맞으니 실행은 시켜줄게" 라고 말한다.

이렇게 **같은 방법으로 호출했는데, 실제로 하는 일이 다른 것**, 하나의 형태가 여러 모습으로 동작할 수 있는 것을 **"Polymorphism"**, **다형성**(엄밀히 말하면, 이 예제는 함수 포인터로 동작을 전달하는 Callback/Strategy에 가깝다고 한다)이라고 한다. 

즉, **"사용하는 쪽에서는 같은 인터페이스로 요청하고, 실제 동작은 대상에 따라 달라진다."**
## 가상 메소드 테이블, 즉 vtable

> "함수 포인터 하나로 이렇게 동작을 바꿀 수 있다면, **아예 여러 개로 묶어서 뭘 어떻게 할 수 없을까?**"

`01_use_after_free` 문제에서는 `VTable` 구조체가 하나 정의되어 있다.

```C
typedef struct
{
    void (*render)(Widget *self);
    void (*on_event)(Widget *self, int code);
} VTable;
```

아까는 함수 포인터를 하나만 사용했었는데, 이번엔 `render(), on_event()` 함수 포인터를  **구조체로 묶어 놓았다**. 이 구조체를 선언해서, 각 함수의 **주소를 할당해준다면?**

```
VTable 구조체 변수

|-----------------|
|  render() 주소  |
|-----------------|
| on_event() 주소 |
|-----------------|
```

이런 느낌. **함수 주소들을 모아놓은 표(Table)**.

그리고 `Widget` 구조체는 다음과 같이 정의되어 있다.

```C
struct Widget
{
    const VTable *vtbl;
    int id;
    int closed;
    char label[24];
};
```

**`VTable` 구조체를 가리키는 포인터를 볼 수 있다**. 그렇다면, 저 `VTable`을 이용하여 함수를 호출할 수 있지 않을까?

`event()` 함수들은 다음과 같이 정의되어 있다.

```C
static void widget_noop_event(Widget *self, int code)
{
    (void)self;
    (void)code;
}

static void dialog_on_event(Widget *self, int code)
{
    if (code == 1)
    {
        self->closed = 1; 
        widget_destroy(self);
    }
}
```

그리고 각 `VTable`은

```C
static void dialog_on_event(Widget *self, int code);

static const VTable BUTTON_VT = {button_render, widget_noop_event};
static const VTable LABEL_VT = {label_render, widget_noop_event};
static const VTable DIALOG_VT = {dialog_render, dialog_on_event};
```

다음과 같이 선언되어 있으며, 각 `VTable` 객체의 `render()`, `on_event()` 함수 포인터들이 저 `button_render`, `widget_noop_event` 들을 **가리킨다**.

이제 그렇다면, `Widget`에서 저 `VTable`을 가리키는 `*vtbl`로 각 함수에 접근할 수 없을까?

먼저 `render()` 접근.

```C
static void screen_render(Screen *s)
{
    for (int i = 0; i < s->count; i++)
    {
        Widget *w = s->items[i];
        w->vtbl->render(w); // VTable을 이용하여 render() 접근!
    }
}
```

그리고 `on_event()` 접근.

```C
for (int i = 0; i < s->count; i++)
{
	Widget *w = s->items[i];
	w->vtbl->on_event(w, code); // VTable을 이용하여 on_event() 접근!
}
```

**각 "객체"에 매핑된 함수로 실행이 된다.** 즉

```text
1. widget에서 vtbl 포인터를 찾아서
2. 그 vtbl 포인터를 이용해서 render/on_event 함수 포인터를 찾고
3. 그 함수 포인터(주소)를 이용해서 실제 함수를 호출한다.

그림으로 보면

Widget --> VTable --> render/on_event에 매핑된 실제 함수 주소 --> 실제 함수 실행
```

그리고 재미있는 점은, `Widget`이 어떤 `VTable`을 가리키느냐에 따라 호출 코드는 똑같지만, **실행되는 함수가 달라질 수 있다는 것이다**. 아까 Callback 함수에서 "호출하는 쪽은 구체적으로 무슨 함수가 실행되는지 몰라도 된다." 라는 구조가 여러 개의 함수 묶음으로 확장된 것이다.

> "근데 왜 `Widget *self` 는 계속 넘겨주지?"

함수는 자신이 어느 객체를 대상으로 호출되었는지, **자동으로 알 수 없기 때문이다**. 따라서 여기에서는

> "이 함수가 어느 객체의 데이터를 가지고 작업해야 하지?"

```C
static void dialog_render(Widget *self)
{
    printf("%s\n", self->label);
}
```

함수가 특정 `Widget`의 데이터를 사용하려면 **어느 `Widget`인지 알려줘야 한다**. 그래서 직접 객체의 주소를 넘긴다.

`widget->vtbl->render(widget)` 

뭔가 이상하지만...

> "`widget`의 `render()`를 실행하는데, 대상 객체는 `widget`이야."

`Python`에서도 **`self`** 에 **현재 객체 자신이 들어온다!**

## 그리고 C++ 에서의...

> "`virtual`?"

C에는 알다시피 `class, virtual, override` 같은 문법이 없지만, 객체지향적인 구조 자체를 만들 수 없는 것은 아니다.

1. 객체의 `상태`는 구조체로 구현하고,
```C
struct Widget
{
	const VTable *vtbl;
	int id;
	int closed;
}
```

2. 객체의 `행동`은 함수로 표현하고,
```C
void widget_render(Widget *self);
```

3. 함수 포인터들을 묶어 `테이블`로 만든 뒤,
```C
typedef struct
{
	void (*render)(Widget *self);
} VTable;
```

4. 객체가 그 `테이블`을 가리키게 하면 된다.
```C
widget->vtbl->render(widget);
```

그리고 C++에서는 이를 훨씬 편하게 표현할 수 있다.

1. 부모 클래스 `Widget` 이 있다고 하고

```C++
class Widget
{
	public:
		// virtual: 가상 함수 키워드. 
		// 런타임의 실제 객체 타입에 따라 override된 함수가 선택된다.
		virtual void render() 
		{
			std::cout << "Widget Render\n";
		}
};
```

2. 이를 상속하는 `Dialog`를 만들면

```C++
class Dialog : public Widget
{
	public:
		// override: 이 함수는 부모의 가상(virtual) 함수를 재정의 하는 중이다!
		// 실제로 부모 클래스의 virtual 함수를 override하고 있는지, 
		// 컴파일러에게 검사하게 한다.
		void render() override
		{
			std::cout << "Dialog Render\n";
		}
}
```

3. 부모 클래스 포인터가 자식 객체를 가리킬 수 있다.
```C++
Widget *widget = new Dialog();
```

포인터는 `Widget *` 형인데 `widget->render();`를 실행하면

```C++
widget->render();

// Widget::render() 가 아닌
// Dialog::render() 가 실행된다.
"Dialog Render"
```

> "다형성인건 알겠는데, 어떻게 `Widget *`가 `Dialog::render()`를 찾아가지?"

C++ 에서 **반드시 `vtable`이라는 구조를 사용하라고 하는 건 아니지만** 많은 C++ 컴파일러들이 가상 함수를 구현하기 위해, `vtable`과 `vptr` 방식을 사용한다. `01_use_after_free` 문제에서 구현되어있던 것 같이, 보이지 않는 포인터(`vptr`)를 하나 추가한다.

```text
Dialog 객체
- vptr -------------------------> Dialog vtable
- Widget 데이터					  - Dialog::render 주소 
- Dialog 데이터
```

그러면 이런 느낌으로

```text
widget --> 객체의 vptr --> vtable --> render 주소 --> Dialog::render()
```

`widget->vtbl->render(widget)` 과 굉장히 비슷하다! C++ 컴파일러가 이를 숨겨주기 때문에 단순히 `widget->render();` 만 써도 되는 것이다.

```text
C 에서는

객체 --> VTable 포인터 --> 함수 포인터 --> 함수 호출

C++ 에서는

객체 --> vptr --> vtable --> 가상 함수 주소 --> 함수 호출
```
