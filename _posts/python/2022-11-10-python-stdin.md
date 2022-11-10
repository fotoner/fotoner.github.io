---
title: "Python에서 좀더 빠른 Input을 받는 방법"
last_modified_at: 2022-11-10T20:29:00
categories: 
 - Python
tags:
 - Python
 - Algorithm
---

필자는 정말 오랜만에 [문제(BOJ:11160)](https://www.acmicpc.net/problem/11660)를 풀며 오랜 기간 물리적으로 접하지 못한 파이썬 재활치료를 하고 있었다.

그러던 중 . . .

![image](/assets/images/posts/python/2022-11-10-python-stdin/img1.png)
시간초과가 나던게 아닌가???? 
<br/>

아무리 몇 달 만에 잡는 파이썬이라고는 하였지만 DP도 제대로 적용했고(개선할 점은 있지만) 알고리즘 자체로는 시간 초과가 날 이유가 존재하지 않았다...
<br/>

그러면서 PyPy로도 채점해 보고 어떻게든 개선할 부분을 찾아보던 와중 Q&A 글에서 제시한 해결법 중 다음과 같은 예시 문제를 보았다.

[문제(BOJ:15552)](https://www.acmicpc.net/problem/15552)
> 본격적으로 for문 문제를 풀기 전에 주의해야 할 점이 있다. 입출력 방식이 느리면 여러 줄을 입력받거나 출력할 때 시간초과가 날 수 있다는 점이다.

그렇다 사실 필자는 가장 기본적인 input 처리를 간과한 것이다. 아래의 필자의 코드를 살펴보자

```python
prefix_sum = [0]
cur_sum = 0

n, m = list(map(int, input().split())) # 이부분

for _ in range(n):
    for val in map(int, input().split()): # 이부분
        cur_sum = cur_sum + val
        prefix_sum.append(cur_sum)
        
for _ in range(m): 
    res = 0
    y1, x1, y2, x2 = list(map(lambda val: int(val) - 1, input().split())) 
    # 특히 이 부분

    for cur_y in range(y2 - y1 + 1):
        res += prefix_sum[x2 + (y1 + cur_y) * n + 1] - prefix_sum[x1 + (y1 + cur_y) * n]

    print(res)
```
다음과 같은 주석 부분에서 입력을 받게 된다. 특히 문제에서 m 번의 입력은 최대 10만 번을 입력받게 되는데, 기본 input 함수로는 감당하지 못하고 1초의 시간제한에 걸리는 것이다!
<br/>

```python
import time

start = time.time()

for _ in range(1000000):
    res = sum(map(int, input().split()))

end = time.time()

print(f'{end - start}s')
```
[문제(BOJ:15552)](https://www.acmicpc.net/problem/15552)와 같이 100만 번의 입력을 가지는 상황을 만들어 실험해 보자. 필자는 임의로 100만 줄의 입력 데이터를 만들고 위의 코드에 입력 스트림으로 넣어보기로 했다.

![image](/assets/images/posts/python/2022-11-10-python-stdin/img3.png)
결과는 1초 이상이 걸린다! 이 상태로는 [문제(BOJ:15552)](https://www.acmicpc.net/problem/15552)를 해결하지 못한다. 그렇다면 어떻게 처리하면 좋을까?

```python
import sys
import time

Input = lambda: sys.stdin.readline().rstrip() #<<<

start = time.time()

for _ in range(1000000):
    res = sum(map(int, Input().split())) #<<<

end = time.time()

print(f'{end - start}s')
```
![image](/assets/images/posts/python/2022-11-10-python-stdin/img4.png)
이게 전부이다.... 

<br/>

## 결론
파이썬의 공식 문서에서 input()함수를 다음과 같이 정의한다. 
>If the prompt argument is present, it is written to standard output without a trailing newline. The function then reads a line from input, converts it to a string (stripping a trailing newline), and returns that. When EOF is read, EOFError is raised.

즉, 개행 처리에 관한 별도의 처리라던가 prompt message를 받아 출력을 하는 기능 등이 존재하며 사용자의 입력 하나하나마다 버퍼에 저장하기에 다수의 입력에서는 비효율 적이다...
<br/>

대신 sys.stdin.readline()의 경우 개행문자를 포함한 단위까지를 버퍼로 취급하고 별도의 처리를 사용자가 하기에 좀 더 효율적이라 할 수 있다.

![image](/assets/images/posts/python/2022-11-10-python-stdin/img5.png)
~~여러분은 필자처럼 바로 맞을 수 있을 문제를 fail 스택 한 번 더 쌓아서 풀지 말자~~