---
layout: post
title: "JAVA - 자료구조, Collection 별 특징 정리"
description: "JAVA - 기본"
author: kimchanjung
date: 2020-01-15 00:00:00 +0900
categories: java
published: true
---

## JAVA - 자료구조, Collection 별 특징 정리

### Collection 특징 구분

|구분   | 종류      | 중복허용 | 순서 존재 | 정렬여부 | Thread-safe |
|------|---------------|------|-------|------|--------------|
| LIST | ArrayList <br>  LinkedList <br> Vector | Yes <br>Yes<br>Yes | Yes <br>Yes<br>Yes   | No <br> No  <br> No  |  No  <br> No  <br>Yes |
|SET| HashSet<br>LinkedHashSet<br>TreeSet  | No <br> No <br> No | No  <br> Yes<br>Yes | No<br> No<br> Yes   | No<br>   No<br>No|
|MAP| HashMap <br> LinkedHashMap <br>   Hashtable <br> TreeMap | No <br> No <br> No <br> No  | No  <br> Yes  <br>No  <br>Yes  | Yes(Key)  <br>   No <br> No <br> Yes          |No  <br> No  <br> Yes <br>  No|


### 자료 구조 특징 구분

| 종류 | 특징       | 중복허용              | Null 허용                            |
|---------------|------------------------|-------------------|----------------|
| List          | 원하는 순서로 Element 삽입가능 <br> 각 요소는 Index 번호를 부여 받는다\.| Yes| \-|
| Set           | 중복 Element 불가능 <br> 그러므로 쉽게 여부 중복확인 가능\. 특정 순서\(Order\) 정할수 없음\. | No| \-|
| Queue         | Output으로 나올 Element만 기본적으로 접근 가능하다<br>| Yes| \-|
| PriorityQueue | 가장 우선순위가 높은 Element가 Head First가 된다| Yes| No|
| Deque         | 양 끝단에서 모두 삽입 / 삭제 가능| Yes| \-|
| Map           | Key / Value로 구성된다\.| 키 : No \- 값 : Yes | 키 : 하나의 \(null\) 키 허용\) \- 값 : Yes |


### LIST

| Class Name | Add  | Remove | Get  | Contains |
|------------|------|--------|------|----------|
| ArrayList  | O(1) | O(n)   | O(1) | O(n)     |
| LinkedList | O(1) | O(1)   | O(n) | O(n)     |

### SET

| Class Name    | Add      | Contains    | Next     |
|---------------|----------|-------------|----------|
| HashSet       | O(1)     | O(1)        | O(h/n)   |
| LinkedHashSet | O(1)     | O(1)        | O(1)     |
| EnumSet       | O(1)     | O(1)        | O(1)     |
| TreeSet       | O(log n) | O(log n)    | O(log n) |

### QUEUE

| Class Name    | Offer    | Peak | Poll     | Size |
|---------------|----------|------|----------|------|
| PriorityQueue | O(log n) | O(1) | O(log n) | O(1) |
| LinkedList    | O(1)     | O(1) | O(1)     | O(1) |
| ArrayDequeue  | O(1)     | O(1) | O(1)     | O(1) |
| DelayQueue    | O(log n) | O(1) | O(log n) | O(1) |

### MAP

| Class Name    | Get      | ContainsKey | Next     |
|---------------|----------|-------------|----------|
| HashMap       | O(1)     | O(1)        | O(h/n)   |
| LinkedHashMap | O(1)     | O(1)        | O(1)     |
| WeakHashMap   | O(1)     | O(1)        | O(h/n)   |
| EnumMap       | O(1)     | O(1)        | O(1)     |
| TreeMap       | O(log n) | O(log n)    | O(log n) |
