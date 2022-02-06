---
title: 80\. Remove Duplicates from Sorted Array II
layout: single
categories: programming
tags:
 - Leetcode
 - Programming
 - Python
---

오늘의 Daily LeetCoding Challege 문제는~~~~ 바로 이것!

![오늘의 문제는 뭘까요~](/assets/images/leetcode/20220206_remove-duplicates-from-sorted-array-ii.png)

오늘 문제는 비내림차순 정렬된 array인 input에서 각 element 값이 2번까지만 중복될 수 있도록 그 이상 중복된 element들을 제거하는 문제이다.
제거한 후의 크기를 알 수 있게 제거한 뒤의 array의 크기를 return으로 주어야 한다.

제거한 array를 return 하는 문제였다면 아마 난이도 easy였겠지만, **다른 array 생성을 위한 별도의 space는 할당하지 마라**가 있어서 medium으로 설정된 것 같다. 


## 조건
1. `1 <= nums.length <= 3 * 10^4`
2. `- 10^4 <= nums[i] <= 10^4`
3. `nums`(input)은 비내림차순으로 정렬되어 있다.

## 아이디어
![이렇게 변경 된다](/assets/images/leetcode/80_remove_array_element.png)

array 전체를 확인하면서 자신에게 copy를 하되, 3번 이상 중복된 element는 copy의 대상에서 제외하면 되는 간단한 문제이다.

이미 정렬까지 되어있기 때문에 그냥 loop를 돌면서 보고있는 element가 이미 2번 이상 반복된 것인지 확인하는 로직만 있으면 바로 풀 수 있다.

## 풀이 1
최적화 생각하지 않고 생각나는 대로 풀어보았다.

array 전체를 확인하면서 다음과 같은 조건문을 반복하면 될 것 같다.
1. 전에 보고 있던 숫자를 기억하고, 현재 숫자와 비교해본다.
2. 해당 숫자가 아직 2번 이하로 반복되었다면 copy하고
3. 그보다 많이 반복되었다면 pass 하면 된다.
 
```python
class Solution:
    def removeDuplicates(self, nums: List[int]) -> int:
        cur = 0  # after array의 index
        repeated = 0  # 몇 번 반복?
        b = None # 마지막에 본 숫자

        # array 전체 확인
        for i in range(len(nums)):
            # 숫자가 이미 2번 반복되었다면 copy하지 않고 continue
            if b == nums[i] and repeated >= 2:
                continue
            
            # 마지막에 본 숫자와 다르면 repeated 초기화
            if b != nums[i]:
                repeated = 0
           
           
            b = nums[i]  # 방금 확인한 number로 b를 update
            repeated += 1  # 반복횟수 + 1
            # cur 에 지금 숫자를 copy하고, cur + 1
            nums[cur] = nums[i]  
            cur += 1
        
        # copy를 마친 array의 길이인 cur를 return
        return cur
```

제출해보니 정답은 맞았지만 다른 풀이에 비해 굉장히 speed가 느린 것을 확인하였다. (112ms, 하위 9.73%)

이는 다른 풀이에 비해 필요없는 비교가 많다는 것이기 때문에 좀 더 좋은 풀이를 고민해보자.

## 풀이 2
우리의 목표는 element의 중복이 최대 2개까지인 array를 만드는 것이다.
그렇다면 이 결과물`nums`은  2보다 큰 인덱스 `i`에 대해 아래의 명제가 무조건 성립하게 된다.

> `nums[i]` 는 2번보다 많이 반복될 수 없기 때문에, `nums[i - 2]`과 무조건 다르다.

그렇다면 array 전체를 확인할 때, 굳이 마지막으로 본 element를 기억하지 않고, `nums[i - 2]` 만 비교해서 copy할 대상인지 아닌지 확인이 가능하다.

```python
class Solution:
    def removeDuplicates(self, nums: List[int]) -> int:
        # 길이가 2 이하면 볼 필요 없이 바로 return
        if len(nums) < 3: 
            return len(nums)

        # 2번까지 반복을 허용하기 때문에, 처음 두 개는 그대로 두고
        # 3번째 숫자 (index로는 2)부터 확인하면 된다.
        cur = 2
        for i in range(2, len(nums)):
            # nums[i] == nums[cur - 2] 일 경우,
            # nums[i]는 최소 3번째 반복되고 있는 숫자라는 뜻이다. 개같이 무시
            if nums[i] != nums[cur - 2]:
                nums[cur] = nums[i]
                cur += 1

        # copy를 마친 array의 길이인 cur를 return
        return cur
                
```

풀이 1에 비해서 훨씬 깔끔해지고 비교도 단 한번만 진행하는 좋은 풀이가 되었다.

제출해보니 44~71ms로 기존 풀이에 비해 최대 3배까지 빨라지고, runtime 지표도 99.51% 까지 나온 것을 확인하였다.

*돌릴 때마다 runtime이 좀 크게 차이나는 편이니까, 풀이는 맞는거 같은데 너무 느린거 같으면 또 돌려보자!*

![99.51% 조.아.](/assets/images/leetcode/80_submit_results.png)
