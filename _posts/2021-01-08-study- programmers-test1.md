---
layout: post
title:  "[프로그래머스] 완주하지 못한 선수"
date:   '2021-01-08' 
categories: study 
tags : [programmers,  collections,Counter,dict,hash]
comments : true
---
[프로그래머스]: https://programmers.co.kr/learn/courses/30/lessons/42576#

> **collections 모듈의 Counter 클래스** 
> Counter()는 **dict와 같이 hash형 자료**들의 값의 개수를 셀 때 사용한다.


```python
#예제
import collections 
collections.Counter('hello world')
```

    Counter({'h': 1, 'e': 1, 'l': 3, 'o': 2, ' ': 1, 'w': 1, 'r': 1, 'd': 1})
----
## 문제 설명    
수많은 마라톤 선수들이 마라톤에 참여하였습니다. **단 한 명의 선수**를 제외하고는 모든 선수가 마라톤을 완주하였습니다.

마라톤에 참여한 선수들의 이름이 담긴 배열 participant와 완주한 선수들의 이름이 담긴 배열 completion이 주어질 때, 완주하지 못한 선수의 이름을 return 하도록 solution 함수를 작성해주세요. 

---
## 제한사항

- 마라톤 경기에 참여한 선수의 수는 1명 이상 100,000명 이하입니다.
- completion의 길이는 participant의 길이보다 1 작습니다.
- 참가자의 이름은 1개 이상 20개 이하의 알파벳 소문자로 이루어져 있습니다.
- 참가자 중에는 동명이인이 있을 수 있습니다.


```python
#문제 적용
import collections
def solution(participant,completion):
    answer=collections.Counter(participant)-collections.Counter(completion)
    return list(answer.keys())[0]
```


```python
p=["leo", "kiki", "eden"]
c=["eden", "kiki"]
solution(p,c)
```


    'leo'

