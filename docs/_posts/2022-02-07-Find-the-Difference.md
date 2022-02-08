---
title: 389\. Find the Difference
layout: single
categories: programming
tags:
 - Leetcode
 - Programming
 - Python
---

오늘의 Daily LeetCoding Challege 문제는 바로 이것!

![오늘의 문제는 뭘까요~](/assets/images/leetcode/20220207_find_the_difference.png)

두 개의 string은 s, t가 있으며 t는 s를 random shuffle + 무작위 문자 한개 로 이루어져 있다.
이번 문제는 이 t에 무작위로 추가된 문자가 무엇인지 찾는 문제이다.

문제 길이에 맞게 easy 난이도의 쉬운 문제이다.

> ![참을 수 없는 달력에 도장찍기](/assets/images/leetcode/2022_01_all_clear.png)
>
> *매일 도장을 찍기 위해서 풀긴하는 데 easy는 그래도 좀...*


## 조건
1. `0 <= s.length <= 1000`
2. `t.length == s.length + 1`
3. `s` 와 `t` 는 알파벳 소문자로 구성되어 있다.

## 아이디어
`s`와 `t`를 모두 탐색하여 알파벳 개수를 센 뒤에 t가 더 많은 알파벳을 찾으면 되는 간단한 문제이다.

## 풀이
s 와 t 의 알파벳 개수를 세는 것은 for문으로 일일히 찾아봐도 되지만, 이왕 python을 쓴다면 제공해주는 기능을 사용하는 게
깔끔하기도 하고 속도도 더 빠르게 결과를 얻을 수 있다 (이 부분은 나중에 post를 하나 쓸 계획)

이번 문제와 같이 개수를 세는 것은 `collections` 모듈의 `Counter`를 사용하면 된다.
`Counter`는 받은 input을 hash를 이용해서 count해주는 dict subclass 이다.
string을 Counter 에 넣으면 각 letter를 key로 하고 개수를 value로 하는 dictionary로 사용 가능한
Counter object 를 return 해준다.

이를 이용해서 풀면 아래와 같이 짧고 깔끔한 풀이가 나온다. 

```python
class Solution:
    def findTheDifference(self, s: str, t: str) -> str:
        s_c = Counter(s)  # s 안의 문자 counting
        t_c = Counter(t)  # t 안의 문자 counting
        
        for i in t_c:     # t_c 의 모든 key 에 대해서 탐색
            if s_c[i] < t_c[i]:   # t가 더 많이 가지고 있는 letter가 우리가 찾는 것!
                return i
            
```


 

