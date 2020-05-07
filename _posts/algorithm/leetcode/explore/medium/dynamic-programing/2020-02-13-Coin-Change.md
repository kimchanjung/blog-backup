---
layout: post
title: "Coin Change  - LeetCode [코테/알고리즘]"
description: LeetCode - explore > top-interview-questions > medium
author: kimchanjung
date: 2020-05-07 00:00:00 +0900
categories: /algorithm/leetcode/explore/medium/dynamic-programing
published: true
---

## [LeetCode] LeetCode - Coin Change
[문제바로가기](hhttps://leetcode.com/explore/interview/card/top-interview-questions-medium/111/dynamic-programming/809)

### 문제설명
#### 1원, 2원, 5원 짜리 동전을 가지고 있는 경우 11원을 만드는 경우 최소 동전의 개수를 구하는 문제  
> 주어진 동전의 종류 = [1, 2, 5]   
> 총 개수는 = 5+5+1 = 3개  

#### 이 문제를 이해 하기 위해서는 유명한 피보나치 수열 이용한 계단 문제를 먼저 이해 해야한다.  
> 계단의 개수 = 3개  
> 계단은 1칸, 2칸 씩만 올라 갈수 있다.  

```bash
 3개의 계단 까지 오르는 방법의 수는  
 첫번째 계단 = 1가지 (1칸 올라 갈수 있다)  
 두번째 계단 = 2가지 ([1,1], [2] => 1칸씩 올라가는 방법 1가지, 2칸을 한꺼번에 올라가는 방법 1가지)  
 새번째 계단 = 3가지 ([1,1,1], [1,2], [2,1])
            첫번째 계단까지 1가지 + 두번째 계단 까지 2가지를 합한 3가지 방법이 된다.  
```
#### 점화식으로 표현하면
> a1 = 1  
> a2 = 2  
> a3 = a2 + a1  
> ...  
> a(n) = a(n-1)+ a(n-2) 가 도출 된다.  

#### 다시 원래 문제로 돌아 오면
11원을 만들기 위해 최소의 동전 개수를 구하기 위해 1원 부터 시작 해본다.
```
1원 = 1개 (1원)
2원 = 1개 (1원+1원, 2원) 최소 동전 개수
3원 = 2개 (1원+1원+1원, 1원+2원) 최소 동전 개수
4원 = 2개 (1원+1원+1원+1원, 1원+1원+2원, 2원+2원) 최소 동전 개수 
5원 = 1개 
     (5원 - 1원 = 4원을 만들 수 있는 최소 개수 = 2개)
     (5원 - 2원 = 3원을 만들 수 있는 최소 개수 = 2개)
     (5원 - 5원 = 0원을 만들 수 있는 최소 개수 = 0개) 
```

#### 셋 중 최소 개수에 + 1을 하는데, 그 이유는 1,2,5 동전으로 5원을 만들기 위해서
```
1은 4원이 필요하고 => 4원을 만드는데 최소 개수 + 1원(1개) = 2개(2원 + 2원) + 1개 5원 
2은 3원이 필요하고 => 3원을 만드는데 최소 개수 + 2원(1개) = 2개(2원 + 1원) + 1개 5원
5원 0원이 필요하고 => 5원을 만드는데 최소 개수 + 5원(1개) = 0개(0원     ) + 1개 5원

그래서 5원을 만드는데 각 1,2,5 원 자기 자신을 제외한 돈을 만드는 개수를 가져와 
그중 최소 개수에 + 1을 하게 되는 것이다
```
#### 점화식으로 표현하면
> a(n) = min( a(n-1), a(n-2), a(n-5) ) + 1  

#### min( a(n-1), a(n-2), a(n-5) ) 부분 즉 동전 종류가 고정으로 주어지지 않으므로 코드로 표현하면
```java
// 동전 개수가 동적이므로 동전 개수만큼 해당 동전별 최소 개수를 모두 가져와 최소 개수를 도출한다.
for(int i = 0; i < coins.length; i++) {
    min = min(min, a(n-conin[i])
}
```

### 소스코드
도출된 점화식을 재귀로 그대로 표현하면 

```java
public class CoinChange {
    public static int coinChange(int[] coins, int amount) {
        int[] amounts = new int[amount + 1];
        int recur = recur(coins, amounts, amount);
        // 문제에서 특별히 주어진 동전종류로 원하는 금액을 만들 수 없는 경우 -1을 
        // 리턴 해야 하므로 아래 처림 처리
        return recur > amount ? -1 : recur;
    }

    public static int recur(int[] coins, int[] amounts, int amount) {
        // 동전을 만들 수 없는 경우는 제외 하므로
        // 1원 = 1원만 필요하고 2원을 필요하지 않기 때문에 max값을 리턴하여 최소 개수계산에 제외 된다.
        if (amount < 0) return Integer.MAX_VALUE - 1;
        if (amount == 0) return 0;
        if (amounts[amount] > 0) return amounts[amount]; // 불필요한 계산을 피하기위한 memorization
        int min = Integer.MAX_VALUE - 1;

        // 점화식이 표현 된 부분
        for (int i = 0; i < coins.length; i++) {
            min = Math.min(min, recur(coins, amounts, amount - coins[i]) + 1);
        }

        return amounts[amount] = min;
    }
}
```

