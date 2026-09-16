---
title: "[Jungle] GDB Cheat Sheet"
date: 2026-09-16 00:00:00 +0900
categories:
  - Krafton Jungle
tags:
  - jungle-13
  - gdb
  - debugging
  - c
description: GDB 명령어와 축약형 정리, NULL 포인터·Use-After-Free·배열 범위 초과 디버깅 예시
---

## 시작하며

이 외에도 더 많은 명령어와 조합이 있으며, 소스 코드와 변수 이름을 보면서 디버깅하려면 디버그 심볼이 포함된 실행 파일이 필요하다.

`<...>`는 실제 값으로 바꿔 넣는다. 축약형은 기본 GDB 기준이며, 별칭 또는 모호하지 않은 명령어 앞부분을 사용한다.

```bash
# 디버그 정보를 포함하여 컴파일
gcc -g -o <program> <source>.c

# 최적화 없이 디버그 정보를 포함하여 컴파일
gcc -g -O0 -o <program> <source>.c

# 더 많은 디버그 정보를 포함하여 컴파일 (매크로 정보까지 포함)
gcc -g3 -O0 -o <program> <source>.c
```

## GDB 시작 / 재시작 / 종료

`gdb ...`는 터미널에서, `run` / `quit` 등은 GDB 안에서 입력한다.

`gdb ./<program> | run | quit`

- GDB 실행 시작
> `gdb ./<program>`

- 인자와 함께 GDB 실행 시작
> `gdb --args ./<program> arg1 arg2 arg3`

- 이미 실행 중인 프로세스로 GDB 실행 시작
> `gdb -p <PID> 또는 gdb -p $(pgrep <program>)` <- `pgrep`은 Linux 등의 셸에서 사용하며, PID가 하나만 나올 때 사용

- GDB 실행 후 프로그램 시작 / 프로그램 재실행
> `run 또는 r`

- GDB 실행 후 인자와 함께 프로그램 시작 / 인자와 함께 프로그램 재실행
> `run arg1 arg2 또는 r arg1 arg2`

- GDB 종료
> `quit 또는 q`

GDB를, 인자 없이 `run`하면 이전 인자를 재사용한다. 인자를 없애려면 `set args 또는 set arg`를 입력한 뒤 실행한다.

이미 실행 중인 프로세스에 연결하면 먼저 멈추며, 실행을 이어가려면 `continue 또는 c`, 연결을 풀고 실행을 이어가려면 `detach 또는 det`를 사용한다.

## 중단점

지정한 소스 코드 위치나 함수에 "도달하면", 해당 코드가 실행되기 전에 멈춘다.

`break <줄 또는 함수 이름> | info breakpoints | delete <중단점 번호>`

- 중단점 설정
> `break main 또는 b main` <- `main()` 함수에 중단점 설정

- 특정 "줄"에 중단점 설정
> `break 42 또는 b 42`

- 특정 "함수"에 중단점 설정
> `break foo 또는 b foo`

- 중단점 목록
> `info breakpoints 또는 i b`

- 중단점 비활성화 (설정은 유지)
> `disable <number> 또는 dis <number>`

- 비활성화한 중단점 다시 활성화
> `enable <number> 또는 ena <number>`

- 중단점 삭제 (중단점 자체를 제거)
> `delete <number> 또는 d <number>`


## 조건부 중단점

지정한 코드 위치에 도달했을 때, "조건이 참인 경우"에만 멈춘다.

`break <위치> if <조건> | condition <number> <조건>`

- 특정 조건을 만족할 때만 중단
> `break <줄 또는 함수 이름> if <조건> 또는 b <줄 또는 함수 이름> if <조건>` <- 예: `break 42 if i == 100`

- 이미 설정한 중단점에 조건 추가 / 변경
> `condition <number> <조건> 또는 cond <number> <조건>` <- 예: `condition 1 count == 100`

- 중단점의 조건만 제거 (중단점은 유지)
> `condition <number> 또는 cond <number>` <- 예: `condition 1`

조건에 사용하는 변수는 해당 중단점 위치에서 접근할 수 있어야 한다.

## 디버깅 시작

"소스 코드 단위"로 실행을 제어하며, 중단점 / Watchpoint / 신호에 의해 실행 도중 멈출 수도 있다.

`bt | continue | next | step | finish`

- 호출 스택 출력
> `backtrace 또는 bt 또는 where`

- 계속 실행 (중단점 / Watchpoint / 정지하도록 설정된 신호를 만나거나 종료될 때까지)
> `continue 또는 c`

- 다음 소스 코드 줄까지 실행 (함수 호출은 내부로 들어가지 않고 실행)
> `next 또는 n`

- 다음 소스 코드 줄까지 실행 (줄 정보가 있는 함수 호출은 내부로 들어감)
> `step 또는 s`

- 선택한 프레임의 함수가 반환한 직후까지 실행 (반환값이 있으면 출력)
> `finish 또는 fin`

## 출력 (변수, 포인터, 레지스터, 메모리...)

프로그램이 멈춘 상태에서 "변수 값"과 "메모리 내용"을 확인한다.

### 변수 출력

`print`는 표현식의 값을, `info`는 변수 목록이나 선택한 프레임의 변수 값을 보여준다.

`print | info`

- 변수 값 출력
> `print <variable> 또는 p <variable>`

- 변수 값을 16진수로 출력
> `print/x <variable> 또는 p/x <variable>`

- 변수 값을 10진수로 출력
> `print/d <variable> 또는 p/d <variable>`

- 변수 값을 2진수로 출력
> `print/t <variable> 또는 p/t <variable>`

- 변수 값을 정수로 변환하여 숫자와 문자 표현 출력
> `print/c <variable> 또는 p/c <variable>`

- 변수의 주소 출력
> `print &<variable> 또는 p &<variable>`

- 현재 선택한 프레임의 지역 변수 출력
> `info locals 또는 i lo`

- 전역 변수 / 파일 범위 정적 변수 목록 (값이 아닌 이름과 타입 출력)
> `info variables 또는 i va`

- 특정 이름이 포함된 전역 변수 / 정적 변수 검색
> `info variables <이름 또는 정규식> 또는 i va <이름 또는 정규식>` <- 예: `info variables count`

- 현재 선택한 프레임의 함수 인자 출력
> `info args 또는 i ar`

### 포인터 출력

`p ptr`는 포인터에 저장된 주소를, `p *ptr`는 가리키는 값을 출력한다.

`print`

- 포인터가 가리키는 값 출력
> `print *<pointer> 또는 p *<pointer>`

- 배열 출력
> `print <array> 또는 p <array>`

- 포인터가 가리키는 연속된 원소를 특정 개수만큼 출력
> `print *<pointer>@<개수> 또는 p *<pointer>@<개수>` <- 예: `p *arr@5`는 `arr[0] ~ arr[4]` 출력

### 구조체 출력

구조체 변수는 `.`으로, 구조체 포인터는 `->`로 멤버에 접근한다.

`print`

- 구조체 변수의 멤버 출력
> `print <struct>.<member> 또는 p <struct>.<member>`

- 구조체 포인터가 가리키는 멤버 출력
> `print <struct_pointer>-><member> 또는 p <struct_pointer>-><member>`

### 레지스터 출력

레지스터 조회는 "선택한 프레임" 기준이며, 실제 정지 상태를 확인하려면 `frame 0`을 선택한다.

`info registers | print $<register>`

- 일반 레지스터 출력 (부동소수점 / 벡터 레지스터 제외)
> `info registers 또는 i r`

- 부동소수점 / 벡터 레지스터까지 모두 출력
> `info all-registers 또는 i all-r`

- 특정 레지스터 출력
> `info registers rax rbx rcx rdx rsp rbp rip 또는 i r rax rbx rcx rdx rsp rbp rip`

- 특정 레지스터 "값" 출력
> `print $rax 또는 p $rax`

- 특정 레지스터 "값"을 "16진수"로 출력
> `print/x $rax 또는 p/x $rax`

- 현재 스택 포인터 출력
> `print/x $rsp 또는 p/x $rsp`

- 선택한 프레임의 명령어 주소 출력
> `print/x $rip 또는 p/x $rip`

```text
각 레지스터 별 주요 용도 정리 
(Linux x86-64 System V ABI 기준, 정수 / 포인터 인자)

RIP : 현재 실행 위치
RSP : Stack Pointer
RBP : Stack Frame Base (프레임 포인터를 사용하는 경우)
RAX : 정수 / 포인터 반환값 등에 자주 사용
RDI : 첫 번째 함수 인자
RSI : 두 번째 함수 인자
RDX : 세 번째 함수 인자
RCX : 네 번째 함수 인자
R8  : 다섯 번째 함수 인자
R9  : 여섯 번째 함수 인자
```

### 메모리 출력

변수의 타입과 무관하게 지정한 주소에서 메모리를 읽고, 형식과 단위에 따라 해석한다.

`x/<개수><출력 형식><단위> <주소>`

- 특정 주소의 값을 "16진수"로 출력
> `x/x <address>`

- `rsp`부터 "16개", "8바이트"씩, "16진수"로 출력
> `x/16gx $rsp`

- `rsp`부터 "32개", "1바이트"씩, "16진수"로 출력
> `x/32bx $rsp` 

- 특정 주소를 "문자열"로 해석
> `x/s <address>`

- `rdi` 값을 "문자열" 시작 주소로 해석하여 출력
> `x/s $rdi`

- 현재 실행할 명령어 출력 (프로그램 카운터)
> `x/i $rip`

- 현재 명령어부터 "10개" 명령어 출력
> `x/10i $rip`

개수를 생략하면 1개를 출력하며, 형식 / 단위는 이전 설정을 이어받는다. `x/s`는 기본적으로 1바이트 문자로 된 NULL 종료 문자열을 읽는다.

### 출력 형식

값의 표시 방법을 정하며, `i`는 `print` 대신 `x` / `display`에서 사용한다.

| 문자  | 의미                               |
| --- | -------------------------------- |
| `x` | 16진수 (Hex)                       |
| `d` | signed 10진수 (Decimal)            |
| `u` | unsigned 10진수 (Unsigned Decimal) |
| `o` | 8진수 (Octal)                      |
| `t` | 2진수 (Binary)                     |
| `c` | 문자 (Character)                   |
| `s` | 문자열 (String)                     |
| `i` | CPU 명령어                          |

### 단위

메모리 출력에서 한 항목을 몇 바이트씩 읽을지 지정한다 (`i` 형식에서는 단위를 무시).

| 문자  | 크기               |
| --- | ---------------- |
| `b` | Byte, 1바이트       |
| `h` | Halfword, 2바이트   |
| `w` | Word, 4바이트       |
| `g` | Giant word, 8바이트 |

## 어셈블리 / 명령어 단위 디버깅

소스 코드 줄 대신 "CPU 명령어 단위"로 실행하며, 아래 레지스터 예시는 x86-64 기준이다.

```text
step/s  : 소스 코드 한 줄 + 함수 내부 진입
next/n  : 소스 코드 한 줄 + 함수 내부 진입 X

stepi/si : CPU 명령어 한 개 + call 내부 진입
nexti/ni : CPU 명령어 한 개 + call 내부 진입 X
```

`disassemble | stepi | nexti`

- 함수의 어셈블리 출력
> `disassemble <function> 또는 disas <function>`

- Intel 문법으로 변경
> `set disassembly-flavor intel 또는 set disassembly intel`

- 현재 실행할 명령어 출력 (프로그램 카운터)
> `x/i $rip`

- 현재 명령어부터 "10개" 명령어 출력
> `x/10i $rip`

- CPU 명령어 한 개 실행 (`call`을 만나면 함수 안으로 들어감)
> `stepi 또는 si`

- CPU 명령어 한 개 실행  (`call`을 만나면 함수 내부를 전부 실행하고 다음 명령어로 이동)
> `nexti 또는 ni`

## 호출 스택 / 스택 프레임

"함수 호출 관계"를 확인하고 "조회할 프레임을 선택"한다. 프레임 선택은 프로그램 실행을 되돌리지 않는다.

`bt | frame | up | down | info frame`

- 호출 스택 출력
> `backtrace 또는 bt`

```text
출력 예시:

#0 foo()
#1 bar()
#2 main()

현재 호출 스택:

main() -> bar() -> foo() (현재 foo()에 위치함)
```

- 상세한 호출 스택 출력
> `backtrace full 또는 bt full`

- 안쪽 프레임부터 특정 "개수"의 호출 스택 출력 (양수 기준)
> `backtrace <number> 또는 bt <number>`

- 조회할 스택 프레임 선택
> `frame <number> 또는 f <number>`

- 호출자 (상위) 프레임 선택
> `up`

- 피호출자 (하위) 프레임 선택
> `down 또는 do`

- 현재 선택한 스택 프레임의 상세 정보
> `info frame 또는 i f`

## Watchpoint

Watchpoint는 "특정 값의 변경이나 메모리 접근을 감시"한다.

`watch | rwatch | awatch | info watchpoints`

- 값이 변경될 때 중단 (같은 값을 다시 쓰는 경우는 제외)
> `watch <variable> 또는 wa <variable>` <- 예: `watch count`

- 값을 읽을 때 중단
> `rwatch <variable> 또는 rw <variable>` <- 예: `rwatch count`

- 값을 읽거나 쓸 때 중단 (값이 그대로여도 중단)
> `awatch <variable> 또는 aw <variable>` <- 예: `awatch count`

- 포인터가 가리키는 값의 변경 감시
> `watch *<pointer> 또는 wa *<pointer>` <- 예: `watch *ptr`

- 현재 포인터가 가리키는 메모리 위치를 고정하여 감시
> `watch -location *<pointer> 또는 wa -l *<pointer>` <- 예: `watch -location *ptr`

- Watchpoint 목록
> `info watchpoints 또는 i watch`

`watch ptr`는 포인터의 주소 값 변경을, `watch *ptr`는 가리키는 값의 변경을 감시한다. 비활성화 / 활성화 / 삭제는 중단점과 같은 명령어를 사용한다.

지역 변수는 해당 변수에 접근할 수 있는 위치에서 설정한다. 일반 지역 변수 Watchpoint는 범위를 벗어나면 삭제되며, `rwatch` / `awatch`는 하드웨어 지원이 필요하다.

## 자동 출력

프로그램이 멈출 때마다 "등록한 값을 자동으로 출력"한다 (`next` / `step` 후에도 출력).

`display | info display | undisplay`

- 변수 / 표현식 자동 출력
> `display <expression> 또는 disp <expression>` <- 예: `display count`

- 레지스터 값을 "16진수"로 자동 출력
> `display/x $rsp 또는 disp/x $rsp`

- 현재 실행할 명령어 자동 출력
> `display/i $rip 또는 disp/i $rip`

- 자동 출력 목록
> `info display 또는 i disp`

- 자동 출력 비활성화
> `disable display <number> 또는 dis disp <number>`

- 비활성화한 자동 출력 다시 활성화
> `enable display <number> 또는 ena disp <number>`

- 자동 출력 제거
> `undisplay <number> 또는 undis <number>` <- 예: `undisplay 1`

자동 출력 번호는 중단점 번호와 별개이다. 지역 변수는 해당 변수를 사용할 수 있는 위치에서만 출력할 수 있다.

## 값 변경

프로그램이 멈춘 상태에서 변수 / 레지스터 / 메모리 값을 직접 변경한다.

`set variable | set $<register> | set {<type>}<address>`

- 변수 값 변경
> `set variable <variable> = <value> 또는 set var <variable> = <value>` <- 예: `set variable count = 100`

- 포인터가 가리키는 값 변경
> `set variable *<pointer> = <value> 또는 set var *<pointer> = <value>` <- 예: `set variable *ptr = 100`

- 레지스터 값 변경
> `set $<register> = <value>` <- 예: `set $rax = 10`

- 스택 포인터 변경
> `set $rsp = <address>`

- 다음 실행할 명령어 주소 변경
> `set $rip = <address>`

- 특정 주소를 지정한 타입으로 해석하여 값 변경
> `set {<type>}<address> = <value>` <- 예: `set {int}0x404000 = 100`

변수 변경은 GDB 설정 명령어와 이름이 충돌하지 않도록 `set variable`을 사용한다. 메모리 주소는 실제로 쓰기 가능한 주소로 지정해야 한다.

`$rip` 변경은 스택 프레임을 바꾸지 않는다. `$rsp` / `$rip`를 잘못 변경하면 정상 실행할 수 없다. 원래의 프로그램 실행 순서와 다르게 동작할 수 있기에...


## Segmentation Fault 분석 시나리오

GDB에서 실행하다 `SIGSEGV`가 발생하면 보통 해당 위치에서 멈춘다. 다시 `run` 하기 전에 호출 스택과 변수 값을 확인한다.

`run -> bt -> frame -> info -> print / x`

- 프로그램 실행 후 오류 발생 위치 확인
> `run 또는 r`

```text
Program received signal SIGSEGV, Segmentation fault.
```

- 호출 스택에서 문제가 발생한 함수 확인
> `backtrace 또는 bt`
> `backtrace full 또는 bt full`

- 오류가 발생한 프레임 선택 / 주변 소스 코드 확인
> `frame 0 또는 f 0`
> `list 또는 l`

- 함수 인자 / 지역 변수 확인
> `info args 또는 i ar`
> `info locals 또는 i lo`

- 의심되는 포인터의 주소 / 가리키는 값 확인
> `print ptr 또는 p ptr`
> `print *ptr 또는 p *ptr`

- 의심되는 주소의 메모리 확인
> `x/16bx ptr`

- 현재 레지스터 / 오류가 발생한 명령어 확인 (x86-64 기준)
> `info registers 또는 i r`
> `x/i $rip`

```gdb
bt
frame 0
list
info args
info locals
print ptr
print *ptr
x/16bx ptr
info registers
x/i $rip
```

`ptr`는 실제로 확인할 포인터 이름으로 바꾼다. `ptr == 0x0`이면 NULL 포인터이며, 잘못된 주소를 조회하면 `Cannot access memory`가 나올 수 있다.

`#0`이 라이브러리 함수라면 `up`으로 호출한 코드도 확인한다. 해제된 메모리는 읽힐 수도 있으므로, 메모리가 출력된다는 것만으로 유효한 포인터라고 판단하지 않는다.

주로 확인할 문제: NULL 포인터 역참조 / 해제된 메모리 접근 (UAF) / 배열 범위 초과 / 초기화되지 않은 포인터 / 스택 오버플로.

### 구체적인 예시를 따라가기 전에

아래 코드는 각각 `null.c` / `uaf.c` / `bounds.c`로 저장하는 예시이다. 디버그 정보를 포함해 컴파일하고, 해당 실행 파일을 GDB로 연다.

```bash
gcc -g -O0 -o null null.c
gdb ./null
```

나머지 예시는 파일 이름을 `uaf` / `bounds`로 바꾼다. 코드를 수정한 뒤에는 다시 컴파일하고 재실행한다. 출력의 주소 / 프레임 번호 / 라이브러리 함수 이름은 환경에 따라 달라질 수 있으며, 아래 호출 스택은 설명을 위해 단순화했다.

`bt`는 "현재 호출 관계"를 보여준다. 이전에 실행한 코드를 거꾸로 재생하는 명령은 아니므로, 문제가 생긴 과정을 확인하려면 중단점을 걸고 다시 실행한다.

### 상황 1. 문자열 길이를 구하다가 죽는다 (NULL 포인터)

라이브러리 함수에서 멈췄더라도, 그 함수를 호출할 때 넘긴 인자가 원인일 수 있다.

```c
#include <stdio.h>
#include <string.h>

void print_name(const char *name)
{
    printf("%zu\n", strlen(name));
}

int main(void)
{
    const char *name = NULL;
    print_name(name);
    return 0;
}
```

`run`으로 실행했더니 `SIGSEGV`가 발생했다. GDB에는 내가 작성한 함수 대신 `strlen` 내부가 보인다.

> "어디서부터 여기까지 온 거지? 호출 스택을 확인해봐야지."

- 현재 호출 관계 확인
> `backtrace 또는 bt`

```text
#0 strlen (...)                  <- 오류가 발생한 위치
#1 print_name (name=0x0)          <- 내가 작성한 함수
#2 main ()
```

> "strlen에 넘긴 문자열이 이상한가? 내 함수의 인자를 확인해봐야지."

- `bt`에서 확인한 `print_name` 프레임 선택 (위 예시에서는 1번)
> `frame 1 또는 f 1`

- 전달받은 인자 / 포인터 값 확인
> `info args 또는 i ar`
> `print name 또는 p name`

```text
$1 = (const char *) 0x0
```

`name`이 NULL이다. `strlen`은 유효한 NULL 종료 문자열을 받아야 하므로, 이 포인터로 문자열을 읽을 수 없다.

> "그럼 누가 NULL을 넘겼지? 호출한 쪽으로 올라가봐야지."

- `main` 프레임을 선택하여 호출부 / 지역 변수 확인
> `up`
> `list 또는 l`
> `info locals 또는 i lo`

이 예시에서는 `main`이 `name`을 NULL로 초기화한 뒤 그대로 넘겼다. 실제 코드에서는 입력을 받았는지, 검색이 실패했는지, 포인터가 초기화됐는지도 확인한다.

- 이 예시의 수정: 실제 문자열을 전달

```c
const char *name = "Jungle";
print_name(name);
```

NULL을 전달할 수 있는 함수라면, 그 경우를 어떻게 처리할지도 함수 안에 정해야 한다.

### 상황 2. 리스트를 지우다가 이상한 주소에 접근한다 (Use-After-Free)

해제된 노드는 주소가 남아 있어도 더 이상 멤버를 읽으면 안 된다. 바로 죽지 않고, 이후 엉뚱한 주소를 사용하면서 오류가 드러날 수도 있다.

```c
#include <stdlib.h>

typedef struct Node {
    int value;
    struct Node *next;
} Node;

void clear_list(Node *curr)
{
    while (curr != NULL) {
        free(curr);
        curr = curr->next;   // 이미 해제한 노드의 next를 읽음
    }
}

int main(void)
{
    Node *head = malloc(sizeof(*head));
    if (head == NULL) return 1;
    head->value = 10;
    head->next = NULL;
    clear_list(head);
    return 0;
}
```

리스트를 정리하다 멈췄거나, 다음 노드 주소가 이상하게 바뀌었다. 이 코드는 UAF이지만, 실행 환경에 따라 오류 없이 끝날 수도 있다.

> "curr는 NULL이 아닌데 왜 이상하지? 포인터와 노드 내용을 확인해봐야지."

- 호출 스택에서 `clear_list` 프레임을 찾아 선택
> `backtrace 또는 bt`
> `frame <clear_list의 프레임 번호> 또는 f <clear_list의 프레임 번호>`

- 함수 인자 / 포인터 / 노드 내용 확인
> `info args 또는 i ar`
> `print curr 또는 p curr`
> `print *curr 또는 p *curr`

주소가 `0x0`이 아니라는 사실만으로 살아 있는 노드라고 판단할 수 없다. 해제된 메모리도 읽힐 수 있어, `print *curr`가 성공한다고 안전한 것은 아니다.

> "언제 해제된 건지 확인해야겠다. free 전후를 한 줄씩 따라가봐야지."

- `clear_list` 입구에 중단점을 걸고 재실행
> `break clear_list 또는 b clear_list`
> `run 또는 r`

- 현재 코드를 확인하고 `free(curr);`가 다음 실행 줄로 표시될 때까지 이동
> `list 또는 l`
> `next 또는 n`

- 해제 전 주소 / 다음 노드 확인
> `print curr 또는 p curr`
> `print curr->next 또는 p curr->next`

- `free(curr);`를 실행한 뒤 포인터 값 확인
> `next 또는 n`
> `print curr 또는 p curr`

`free()`는 노드를 해제하지만, 지역 변수 `curr`를 자동으로 NULL로 바꾸지 않는다. 따라서 다음 줄의 `curr->next`는 해제된 객체에 접근하는 코드이다.

- 수정: 다음 노드 주소를 "해제 전에" 저장

```c
while (curr != NULL) {
    Node *next = curr->next;
    free(curr);
    curr = next;
}
```

`watch curr`는 포인터 값의 변경을 감시한다. `free(curr)` 자체는 `curr`의 값을 바꾸지 않으므로, 이것만으로 노드가 해제되는 순간을 잡을 수는 없다.

### 상황 3. 배열 끝을 한 칸 넘어 쓴다 (Off-by-One)

배열 범위 초과는 해당 줄에서 바로 죽거나, 주변 메모리를 망가뜨린 뒤 다른 곳에서 드러날 수 있다. 잘못된 인덱스를 사용하는 순간에 멈추면 원인을 확인하기 쉽다.

```c
#include <stdio.h>

void fill_array(int *arr, int length)
{
    for (int i = 0; i <= length; i++) {
        arr[i] = i;
    }
}

int main(void)
{
    int arr[4] = {0};
    fill_array(arr, 4);
    printf("%d\n", arr[3]);
    return 0;
}
```

`arr`에는 4개의 원소가 있으므로 유효한 인덱스는 `0 ~ 3`이다. 하지만 `i <= length` 때문에 `arr[4]`까지 쓴다. 이 코드도 반드시 `SIGSEGV`가 발생하는 것은 아니다.

> "죽은 위치보다 앞에서 배열을 잘못 쓴 걸 수도 있겠네. 범위를 넘는 순간에 멈춰봐야지."

- 배열에 쓰는 줄에 조건부 중단점 설정 (위 코드를 그대로 저장한 `bounds.c`의 6번 줄)
> `break bounds.c:6 if i >= length 또는 b bounds.c:6 if i >= length`

- 재실행하여 잘못된 접근 "직전"에 중단
> `run 또는 r`

코드를 수정했다면 줄 번호도 실제 `arr[i] = i;` 위치에 맞춘다. 반복 변수 `i`는 함수 입구에서는 아직 없으므로, `i`에 접근할 수 있는 반복문 안쪽에 조건을 건다.

> "지금 몇 번째 칸에 쓰려는 거지? 인덱스와 길이를 확인해봐야지."

- 인덱스 / 배열 길이 확인
> `print i 또는 p i`
> `print length 또는 p length`

```text
(gdb) print i
$1 = 4
(gdb) print length
$2 = 4
```

아직 `arr[4] = 4;`를 실행하기 전이다. 인덱스가 배열 길이와 같아지는 순간부터 범위를 벗어난다는 것을 확인했다.

> "마지막 인덱스는 length - 1인데, 반복 조건에 등호를 넣었구나."

- 수정: 배열 길이 "미만"일 때만 반복

```c
for (int i = 0; i < length; i++) {
    arr[i] = i;
}
```

수정 후 다시 컴파일하고 같은 중단점을 걸어 실행하면, 이 예시에서는 `i >= length`인 상태로 쓰기 줄에 도달하지 않는다. 오류 없이 실행됐다는 사실에 더해, 잘못된 접근이 사라졌는지도 확인한 것이다.

## 유용한 기타 명령어

소스 코드 / 타입 / 심볼 / 프로세스 정보를 조회하고 명령어 사용법을 찾는다.
### 소스 코드 확인

기본적으로 "주변 10줄을 출력"하며, `list`를 반복하면 다음 부분을 이어서 출력한다.

`list`

- 현재 위치 / 이전 출력에 이어서 소스 코드 출력
> `list 또는 l`

- 특정 함수 주변 소스 코드 출력
> `list <function> 또는 l <function>` <- 예: `list main`

- 특정 줄 주변 소스 코드 출력
> `list <line> 또는 l <line>` <- 예: `list 42`

- 특정 파일의 줄 주변 소스 코드 출력
> `list <file>:<line> 또는 l <file>:<line>` <- 예: `list main.c:42`

- 특정 범위의 소스 코드 출력
> `list <시작 줄>,<끝 줄> 또는 l <시작 줄>,<끝 줄>` <- 예: `list 20,40`

- 직전에 출력한 코드의 앞부분 출력
> `list - 또는 l -`

### 타입 확인

`whatis`는 "타입 이름"을, `ptype`은 typedef를 풀어 "타입의 상세 정의"를 보여준다.

`ptype | whatis`

- 변수 / 표현식의 타입 이름 확인
> `whatis <expression> 또는 wha <expression>` <- 예: `whatis ptr`

- 변수 또는 타입의 상세 구조 확인 (구조체 멤버 등)
> `ptype <variable 또는 type> 또는 pt <variable 또는 type>` <- 예: `ptype struct node`

### 함수 / 심볼 확인

"함수 이름을 검색"하거나, "심볼 이름과 저장 위치를 서로 연결하여 확인"한다.

`info functions | info address | info symbol`

- 함수 목록 (이름과 타입) 출력
> `info functions 또는 i fun`

- 특정 이름이 포함된 함수 검색
> `info functions <이름 또는 정규식> 또는 i fun <이름 또는 정규식>` <- 예: `info functions foo`

- 심볼이 저장된 위치 확인
> `info address <symbol> 또는 i addr <symbol>` <- 예: `info address main`

- 주소에 해당하는 심볼 / 가까운 심볼과의 거리 확인
> `info symbol <address> 또는 i sym <address>` <- 예: `info symbol $rip`

`info address`는 심볼에 따라 주소 / 레지스터 / 스택 오프셋 등을 알려준다. 변수의 현재 주소는 출력 섹션의 `print &<variable>`로 확인한다.

### 프로세스 / 메모리 매핑 확인

Linux 등 지원되는 환경에서 사용한다. 의심되는 주소가 코드 / Heap / 공유 라이브러리 / Stack 중 어느 영역에 있는지 확인할 때 유용하다.

`info proc | info proc mappings`

- 디버깅 중인 프로세스 정보 확인
> `info proc 또는 i proc`

- 메모리 영역의 주소 범위 / 매핑된 파일 확인
> `info proc mappings 또는 i proc mappings`

### 도움말 / 명령어 검색

GDB 내장 "도움말"에서 명령어 문법과 축약형을 확인하고 관련 명령어를 검색한다.

`help | apropos`

- 명령어 사용법 확인
> `help <command> 또는 h <command>` <- 예: `help watch` 또는 `h wa`

- 이름 / 설명에서 특정 단어 또는 정규식으로 명령어 검색
> `apropos <keyword> 또는 apr <keyword>` <- 예: `apropos breakpoint`
