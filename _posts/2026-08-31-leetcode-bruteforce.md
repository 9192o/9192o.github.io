---
title: 149. Max Points on a Line
date: 2026-08-31
categories: Krafton Jungle
tags:
  - jungle-13
  - algorithm
description: 149. Max Points on a Line 풀이
---

![LeetCode 149 문제 화면](/assets/img/posts/2026-08-31-leetcode-bruteforce/01-problem.png)

## 문제

> Given an array of `points` where `points[i] = [xi, yi]` represents a point on the **X-Y** plane, return the maximum number of points that lie on the same straight line.

## 예시

**Example 1:**

![Example 1](/assets/img/posts/2026-08-31-leetcode-bruteforce/02-example-1.jpg)

> **Input:** points = `[[1,1],[2,2],[3,3]]`
>
> **Output:** 3

**Example 2:**

![Example 2](/assets/img/posts/2026-08-31-leetcode-bruteforce/03-example-2.jpg)

> **Input:** points = `[[1,1],[3,2],[5,3],[4,1],[2,3],[1,4]]`
>
> **Output:** 4

## 생각의 흐름

1. 한 직선 위에 다른 점이 있다는 말은 무슨 말인가? → 각 기울기가 같다는 말이다. 그렇다면 어떤 원점에서 한 점의 기울기와 다른 한 점의 기울기가 같다는 말은 세 점이 한 직선 위에 있다는 말일 것이다.

2. 만약 주어진 점들의 배열 `points` 안에 두 점이나 한 점밖에 없다면? → 그냥 각각 두 개, 한 개를 반환하게 하자.

```python
if len(points) <= 2:
    return len(points)
```

3. 그렇다면 `points` 배열이 3개 이상부터는? → 최솟값은 2겠네?

```python
m = 2  # 나중에 최댓값을 저장할 변수
```

4. 그런데 기울기는 결국 `(y의 변화량) / (x의 변화량)`이다. 즉 어떤 원점 `(x, y)`가 있고 `(x1, y1)`이 있다면 기울기는 `(y1 - y) / (x1 - x)`일 텐데, 컴퓨터의 부동소수점 표현 방식 때문에 기울기 비교를 신뢰하기 어렵다. 그렇다면 그냥 `dx`, `dy`로 두고 풀어야 하나?

5. 그리고 어떻게 구하지? → `points` 안의 점들을 배열 끝 원소를 제외하고 한 번씩 원점으로 정하고, 나머지 점들에 대한 기울기를 구해서 저장해 놓자.

```python
for i in range(len(points) - 1):
    # 한 점을 정하고, 나머지 점들에 대한 기울기를 저장할 딕셔너리
    table = {}
    for j in range(i + 1, len(points)):
        # 기울기 (dx, dy)
        dx = points[j][0] - points[i][0]
        dy = points[j][1] - points[i][1]
```

6. 이제 간단한 경우를 생각해 보자. 입력이 `[(1, 1), (2, 2), (3, 3)]`이라면 실행 흐름은 다음과 같을 것이다.

```text
1) (1, 1)을 원점으로 시작 (i = 0)
2) 각 (2, 2), (3, 3) (j = 1, 2)에 대한 기울기 (1, 1), (2, 2) 계산

→ (1, 1)에서 나머지 점들과 기울기 정보는 (1, 1), (2, 2). 2개

3) (2, 2)로 원점 이동 (i = 1)
4) 다음 점인 (3, 3) (j = 2)과 기울기 (1, 1) 계산

→ (2, 2)에서 나머지 점들과 기울기 정보는 (1, 1). 1개

5) 정규화한 (1, 1)의 개수는 2개가 최대고, 이에 대한 직선 위의 점은 3개가 될 것.
```

7. 문제가 있다. 실제로 이 세 점은 한 직선 위에 있는데 방금 계산으로는 `dx`, `dy`가 `(1, 1)`, `(2, 2)`로 다르다. → 크기를 최소로 정규화하자.

8. `(dx, dy)`가 만약 `(1, 2)`, `(2, 4)`, `(3, 6)`이라는 말은 각 `dx`, `dy`를 `GCD(dx, dy)`로 나눈 값이 같다는 뜻이다. 예시의 각 GCD는 1, 2, 3이므로 모두 `(1, 2)`로 통일된다.

```python
g = math.gcd(dx, dy)
dx //= g
dy //= g
```

9. 중간 코드 1)

```python
class Solution:
    def maxPoints(self, points: List[List[int]]) -> int:
        if len(points) <= 2:
            return len(points)

        m = 2
        for i in range(len(points) - 1):
            for j in range(i + 1, len(points)):
                # 기울기 (dx, dy)
                dx = points[j][0] - points[i][0]
                dy = points[j][1] - points[i][1]

                # dx, dy를 gcd(dx, dy)로 나누어 줌
                g = math.gcd(dx, dy)
                dx //= g
                dy //= g
```

10. `(dx, dy)`가 `(1, 2)`, `(2, 4)`, `(3, 6)` 같은 것은 모두 `(1, 2)`로 통일되었다. 그런데 만약 점들이 `[(1, 2), (-2, -4), (3, 6)]`이면? 점 `(1, 2)`에서 나머지 점들에 대한 `(dx, dy)`는 `(-3, -6)`, `(2, 4)`. 앞에서처럼 나누어도 `(-1, -2)`, `(1, 2)`로 다르다고 나온다. 실제로는 한 직선 위에 있는데 말이다.

11. 방향을 통일해야 할 것 같다. 우리는 지금 `(dx, dy)`인 `(-1, -2)`가 `(1, 2)`와 동일하다고 보고 싶다. 둘 다 음수일 때 `dx`와 `dy`의 부호를 양수로 바꿔 버리면 안 될까?

```python
# 처음 생각했던 코드
if dx < 0 and dy < 0:
    dx *= -1
    dy *= -1
```

12. 해결한 줄 알았으나 또 다른 문제가 있었다. 만약 `(dx, dy)`가 `(-1, 2)`, `(1, -2)`라면? 둘은 같은 방향을 서로 반대로 표현한 것이다. 여러 부호 조합을 따로 처리하면 원래 `(-1, 2)`가 갑자기 `(1, -2)`로 잘못 바뀔 수도 있다.

13. 한쪽만 기준을 세우자. `dx`가 음수일 때만 `(dx, dy)`에 `-1`을 곱해 반전시켜 버리자.

```python
# 수정된 코드
if dx < 0:
    dx *= -1
    dy *= -1
```

14. 진짜 해결한 줄 알았으나 테스트 케이스에 복병이 숨어 있었다.

```python
[[0,1],[0,0],[0,4],[0,-2],[0,-1],[0,3],[0,-4]]
```

15. 수직선은 현재 검사 조건으로 방향이 통일되지 않을 수 있다. → 그냥 `(0, 1)`로 통일시켜 버리자.

```python
if dx < 0:
    dx *= -1
    dy *= -1
if dx == 0:
    dy = 1
```

16. 이제 기울기가 같은 점들의 개수를 세어 봐야 한다. 여기서 Python의 딕셔너리를 쓰자. 튜플 `(dx, dy)`를 키로 두고 하나 중복될 때마다 값을 1 올린다.

```python
table = {}

delta = (dx, dy)
if table.get(delta) is None:  # 딕셔너리에 해당 기울기 정보가 없을 때
    table[delta] = 2          # 첫 직선에는 점 2개
else:
    table[delta] += 1
```

17. 첫 원점과 나머지 모든 점에 대한 기울기 정보를 다 저장했으면 딕셔너리 값의 최댓값을 뽑아낸다. 모든 반복이 끝나면 반환한다.

```python
m = max(m, max(table.values()))
```

18. 시간 복잡도는 `O(n²)`, 공간 복잡도는 `O(n)`.

## 최종 코드

```python
import math


class Solution:
    def maxPoints(self, points: List[List[int]]) -> int:
        if len(points) <= 2:
            return len(points)

        m = 2
        for i in range(len(points) - 1):
            table = {}
            for j in range(i + 1, len(points)):
                # 기울기 (dx, dy)
                dx = points[j][0] - points[i][0]
                dy = points[j][1] - points[i][1]

                # 같은 기울기의 크기를 gcd로 정규화
                g = math.gcd(dx, dy)
                dx //= g
                dy //= g

                # dx를 기준으로 방향을 통일
                if dx < 0:
                    dx *= -1
                    dy *= -1

                # 수직선은 (0, 1)로 통일
                if dx == 0:
                    dy = 1

                delta = (dx, dy)
                if table.get(delta) is None:
                    table[delta] = 2
                else:
                    table[delta] += 1

            m = max(m, max(table.values()))

        return m
```
