---
layout: post
title: JAVA - 자료구조, Collection 별 특징 정리
description: JAVA - 기본
author: kimchanjung
date: 2020-01-15 00:00:00 +0900
categories: java
published: true
---

## JAVA - 자료구조, Collection 별 특징 정리
### Collection 특징 구분
| 종류            | 중복허용 | 순서 존재 | 정렬여부 | Thread\-safe |
|---------------|------|-------|------|--------------|
| ArrayList     | Yes  | Yes   | No   | No           |
| LinkedList    | Yes  | Yes   | No   | No           |
| Vector        | Yes  | Yes   | No   | Yes          |
| HashSet       | No   | No    | No   | No           |
| LinkedHashSet | No   | Yes   | No   | No           |
| TreeSet       | No   | Yes   | Yes  | No           |
| HashMap       | No   | No    | No   | No           |
| LinkedHashMap | No   | Yes   | No   | No           |
| Hashtable     | No   | No    | No   | Yes          |
| TreeMap       | No   | Yes   | Yes  | No           |


### 자료 구조 특징 구분

| 종류            | 특징                                                            | 중복허용              | Null 허용                            |
|---------------|---------------------------------------------------------------|-------------------|------------------------------------|
| List          | 원하는 순서로 Element 삽입가능\. 각 요소는 Index 번호를 부여 받는다\.               | Yes               | \-                                 |
| Set           | 중복 Element 불가능\. 그러므로 쉽게 여부 중복확인 가능\. 특정 순서\(Order\) 정할수 없음\. | No                | \-                                 |
| Queue         | Output으로 나올 Element만 기본적으로 접근 가능하다\.                          | Yes               | \-                                 |
| PriorityQueue | 가장 우선순위가 높은 Element가 Head First가 된다\.                         | Yes               | No                                 |
| Deque         | 양 끝단에서 모두 삽입 / 삭제 가능                                          | Yes               | \-                                 |
| Map           | Key / Value로 구성된다\.                                           | 키 : No \- 값 : Yes | 키 : 하나의 \(null\) 키 허용\) \- 값 : Yes |





