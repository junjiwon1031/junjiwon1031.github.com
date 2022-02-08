---
title: 258\. Add Digits
layout: single
categories: programming
tags:
 - Leetcode
 - Programming
 - Python
---

오늘의 Daily LeetCoding Challege 문제는 바로 이것!

![오늘의 문제는 뭘까요~](/assets/images/leetcode/20220208_add_digits.png)

주어진 숫자의 각 자릿수를 더하고, 만약 한 자릿수가 아니라면 각 자릿수를 더하는 것을 반복한다.

최종적으로 얻어진 한 자릿수를 return 하는 것이 목표이다.

프로그래밍 적으로는 굉장히 쉬운 문제라서 easy 난이도인데,
너무 쉬운 것이라 그런지 recursive/loop 없이 `O(1)`으로도 도전해보라는 문구가 있다.

두 방법 모두 Try 해보자!

## 조건
1. `0 <= num <= 2^31 - 1`

## 아이디어 1
문제에서 주어진 그대로 실행하는 것이다.
각 자릿수를 모두 더하는 것을 한 자릿수가 나올 때까지 반복하는 것이다.

사실 `num`의 최대값이 $$ 2^{31} - 1 = 2,147,483,647 $$라 최대 10자리 수이고, 자릿수가 모두 9 라고 해도 각 자릿수의 합이 최대 `9 * 10 = 90`정도라 반복 횟수도 굉장히 적다.

## 풀이
각 자릿수를 합하는 방식은 여러가지가 있을 수 있지만, 필자가 가장 좋아하는 방식은 str으로 변환하는 방식이다.

`num`을 str으로 만들면 별도의 처리없이 각 자릿수가 iterable 하게 변경이 되기에 map을 사용하면 쉽게 sum을 구할 수 있다.

속도는 모든 자릿수를 확인하기 때문에, `O(log n)`이 됩니다.

```python
class Solution:
    def addDigits(self, num: int) -> int:
        # num 이 한 자릿수가 될 때까지 반복!
        while num >= 10:
            #                   (generator, not list!)
            # ex) 123 ---> "123" ---> [1, 2, 3] --> 6
            #         str      map(int)         sum
            num = sum(map(int, str(num)))
        
        return num           
```

## 아이디어 2
Easy 문제를 Easy로 끝내면 그날 문제 풀었다는 느낌이 안들어서 너무 아쉽다.
Follow up 으로 O(1)으로 가능한지 물어봤으니 이 방식도 반드시 해봐야 한다.

Loop가 없이 이 단순한 문제를 풀으라 했으니 분명히 수학, 그 것도 정수론을 이용하라는 의미일 가능성이 높다.

실제로 Related Topic을 확인해보면 Number Theory라고 대놓고 힌트를 주고 있다. 한번 생각해보자.

1. `k` 자릿수인 `num`의 `n`번째 자릿수를 `a_n`이라고 하면, `num`은 다음과 같이 표현 가능하다.\
\
$$ num = \sum_{n = 0}^{k-1} {a_n \times 10 ^ n} $$

2. 각 자릿수를 더하는 것은 다음과 같이 표현이 가능하다.\
\
$$\sum_{n = 0}^{k-1} {a_n} $$

3. 둘의 차이는 `a_n`에 달려 있는 $$ 10^n $$ 뿐인데, 이걸 $$ 1 $$ 로 바꿀 수 있는 방법만 있다면 문제 풀이가 가능할 것 같다.

4. $$ 10^n = 99...9 + 1 $$ 이니까 $$ 10^n \equiv 1 \mod 9 $$라는 것은 매우 자명! 즉 아래의 식이 참임을 알 수 있다.\
\
$$ num \equiv \sum_{n = 0}^{k-1} {a_n} \mod 9 $$

5. $$ \sum_{n = 0}^{k-1} {a_n} $$ 도 어떤 하나의 숫자일 테니 이를 `num2`라고 해보자.\
`num2`도 4번과 동일하게 적용하면 자릿수의 합이 `num2 mod 9`과 같은데, 이 값은 `num1 mod 9`와도 같다.

6. 5번의 과정을 한 자릿수가 될 때까지 반복해도 그 값에 `mod 9`를 취하면 모두 `num1 mod 9`와 같다.

7. 따라서 `function addDigits`는 `num % 9`로 표현하는 것이 가능하다.

다만 `num` 이 9의 배수일 때를 조심해야 한다.

`9n % 9 == 0` 이므로, modulo 값이 0일 때 9를 return하면 될 것 같지만, `n == 0`인 경우엔 return이 0이 되어야 한다.
이 부분만 예외처리를 하면 정답을 얻을 수 있다.

## 풀이 2
위의 아이디어를 그대로 코드로 옮기면 아래와 같다.
```python
class Solution:
    def addDigits(self, num: int) -> int:
        # 예외처리
        if num == 0:
            return 0
        
        # 그 이외에는 num % 9 로 모든 case 커버 가능
        return 9 if num % 9 == 0 else num % 9
```

아주 짧게 `O(1)`으로 정답을 얻어낼 수 있었다.

+) 0을 제외하면 (num - 1) % 9 + 1을 통해서 정답을 얻어낼 수 있다.\
(123456780 이 되던 연산을 012345678로 깔끔하게 바꿈!)\
그리고 사실.. python에서 사용하는 `%`는 modulo과는 다른 연산자라서 0에 대한 예외처리도 필요없다 :dizzy_face:\
modulo 는 $$ -1 \equiv 8 \mod 9 $$ 이지만, `%`는 remainer 이기 때문에 `-1 % 9 == -1` 라서 0을 대입해봐도 정답이 나오는 것을 알 수 있다...

따라서 다음처럼 구현해도 정답이 나온다.
```python
class Solution:
    def addDigits(self, num: int) -> int:
      return (num - 1) % 9 + 1
```


### 잡담 
Coding interview는 수학적인 증명 문제를 물어보는 시험이 아니기 때문에 풀이 2는 솔직히 도움되지 않는다고 생각합니다.\
수학적 풀이를 좋아하는 것이 아니라면 풀이 1으로 가볍게 풀고 넘어갑시다.

풀이 2에 대해 더 자세히 알고 싶다면 이 개념 자체를 정리한 Digital Root 라는 개념을 살펴보면 좋습니다.
