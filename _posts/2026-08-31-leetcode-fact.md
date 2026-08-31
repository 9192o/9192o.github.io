---
title: 172. Factorial Trailing Zeroes
date: 2026-08-31
tags:
  - jungle-13
  - algorithm
categories: Krafton Jungle
---
![LeetCode 172 문제 화면](/assets/img/posts/2026-08-31-leetcode-fact/01-problem.png)

## 문제

>Given an integer `n`, return _the number of trailing zeroes in_ `n!`.
>

## 예시

**Example 1:**

>**Input:** n = 3
>**Output:** 0
>**Explanation:** 3! = 6, no trailing zero.

**Example 2:**

>**Input:** n = 5
>**Output:** 1
>**Explanation:** 5! = 120, one trailing zero.

**Example 3:**

>**Input:** n = 0
>**Output:** 0

## 생각의 흐름

1. Factorial(N) 을 계산하고, 10을 나눠가며 0의 개수를 셈 -> Integer 범위를 벗어남.

2. 그렇다면 재귀하면서, 그때그때 0을 센 후, 그 다음에 10을 나눈 값을 넣어주면 어떨까. -> 좋은 생각이나, 더 나은 방법이 있을 수 있겠다 판단함.

3. 먼저 수학적으로 접근해봄. 어떤 상황에서 뒷 자리가 0이 될까 -> (2 x 5) 꼴이 곱해져 있을 때. -> 그렇다면 (2 x 5) 꼴의 개수를 세면 되겠네. -> 2의 개수는 항상 5의 개수보다 많음. -> 그렇다면 5의 개수를 세면 어떨까?

4. 중간 코드 1)

```python
class Solution:
    def trailingZeroes(self, n: int) -> int:

        count = 0
        def fact(n):
            if n == 0:
                return
            if n % 5 == 0:
                nonlocal count
                count += 1
            fact(n - 1)
        fact(n)
        return count
```

![첫 번째 풀이 결과](/assets/img/posts/2026-08-31-leetcode-fact/02-first-attempt.png)

5. 케이스를 하나씩 훑어봄. -> N >= 5 이상일때 1, 2, 3, ...씩 늘어남. -> N >= 25부터, 예상했던 5가 아닌 6이 나옴. -> 그렇다면 N = 25 일때 +1 을 한 번 더 하자.
6. 중간 코드 2)

```python
class Solution:
    def trailingZeroes(self, n: int) -> int:

        count = 0
        def fact(n):
            if n == 0:
                return
            if n % 5 == 0:
                nonlocal count
                count += 1
            if n % 25 == 0:
                count += 1
            fact(n - 1)

        fact(n)
        return count
```

![두 번째 풀이 결과](/assets/img/posts/2026-08-31-leetcode-fact/03-second-attempt.png)

7. 이번에는 125에서 막힘. -> 내가 모르는 규칙이 더 있구나 -> N = 125 일때도 +1 을 추가함 -> N = 625 일때도 +1 을 추가함 -> N = 3125 일때도 +1 을 추가함.
8. 최종 코드

```python
class Solution:
    def trailingZeroes(self, n: int) -> int:
		return  n // 3125 + n // 625 + n // 125 + n // 25 + n // 5
```

## 왜 그럴까?

1. 앞서 뒤의 0 하나는 (2 x 5) 꼴일때 생김. -> N! 에는 5보다 2가 훨씬 많기 때문에 굳이 2를 안세고, 5만 세도됨.

2. **5는 한 개, 5^2 는 5를 2개, 5^3 은 5를 3개 제공함**. 따라서 위 코드가 가능함. 단, 입력 조건이 (0 <= n <= 10^4)

## 리팩토링

위의 사고 과정을 그대로 코드로 구현한 것이다.

```python
class Solution:
	def trailingZeroes(self, n: int) -> int:
		i = 1
		count = 0
		while n // 5**i > 0:
			count += (n // 5**i)
			i += 1

		return count
```

또는 더 짧게

```python
class Solution:
    def trailingZeroes(self, n: int) -> int:
        count = 0
        while n // 5 > 0:
            count += n // 5
            n //= 5

        return count
```
