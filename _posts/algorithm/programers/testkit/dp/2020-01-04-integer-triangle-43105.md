---
layout: post
title: "정수삼각형 (43105) - 프로그래머스 [코테][알고리즘]"
description: 프로그래머스 - 코딩테스트 연습 > 동적계획법(Dynamic Programming) > 정수 삼각형 (43105)
author: kimchanjung
date: 2020-01-04 00:00:00 +0900
categories: algorithm/programers/testkit/dp
published: true
---

## [프로그래머스] 코딩테스트 연습 > 동적계획법(Dynamic Programming) > 정수 삼각형 (43105)
[문제바로가기](https://programmers.co.kr/learn/courses/30/lessons/43105)
### 트리탐색의 후위순회를 예를 들어 설명하면
```bash
  1 (1+3 = 4)
2   3
```
탐색 순서는 2 -> 3 -> 1 됩니다.

### 해결방법
1. 자식 노드 2 와 3중에 큰 값과 1을 더한 값을 저장합니다. 
즉 1의 두 자식 노드 중 큰 값과 자기 자신을 계속 적으로 더해 나가는 방법을 사용합니다. 
점화식으로 표현하면 아래와 같습니다.
>부모합산 = 부모 + max(오른쪽자식, 왼쪽자식) 

2. 재귀를 사용하여 해결한 코드

```java
public int preOrder(부모노드){
    if (노드의 번호가 초과하면 ) return 0;
    if (합산값이 이미 있다면) return 합산[부모노드];
   
    return 합산[부모] = 부모 + max( preOrder(왼쪽자식),   preOrder(오른쪽자식))
}
```

### 소스코드
```java
class Solution {
    private static int[][] nodes;
    private static int[][] nodesSum;

    public static int solution(int[][] triangle) {
        nodes = triangle;
        nodesSum = new int[triangle.length][triangle.length];
        return preOrder(0, 0);
    }

    private static int preOrder(int row, int col) {
        if (nodes.length == row) return 0;
        if (nodesSum[row][col] > 0) return nodesSum[row][col];
        return nodesSum[row][col] = nodes[row][col] + Math.max(preOrder(row + 1, col), preOrder(row + 1, col + 1));
    }

}
```