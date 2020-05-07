---
layout: post
title: "Longest Substring Without Repeating Characters - LeetCode [코테/알고리즘]"
description: LeetCode - explore > top-interview-questions > medium
author: kimchanjung
date: 2020-05-07 00:00:00 +0900
categories: /algorithm/leetcode/explore/medium/arrays-and-strings
published: true
---

## [LeetCode] LeetCode - Longest Substring Without Repeating Characters 
[문제바로가기](https://leetcode.com/explore/interview/card/top-interview-questions-medium/103/array-and-strings/779/)

### 이 문제는 주어진 문자열에 중복이 없는 부분 문자열의 길이를 찾는 문제
```bash
abcabcbb => abc
bbbbb => b
pwwkew => wke

```
### 해결책
문자열의 첫번째 문자열 부터 시작 하여 하나씩 증가 시켜고 다음 문자열이 포함되는지 체크한다.
중복없는 문자열의 길이중 가장 긴 값을 리턴한다.
```
abcabcbb
a <- b  // 포함 하지 않으면 다음 문자열까지 잘라서 비교한다.
ab <- c // 포함 하지 않으면 다음 문자열까지 잘라서 비교한다.
abc <- a // a가 포함 되었다. 비교문자를 자르는 시작을 a 다음 인 b 부터 시작한다
bca <- b // b가 포함 되었다. 비교문자를 자르는 시작을 c 다음 인 c 부터 시작한다
cab <- c // c가 포함 되었다. 비교문자를 자르는 시작을 c 다음 인 a 부터 시작한다
..
```


### 소스코드
도출된 점화식을 재귀로 그대로 표현하면 

```java
public class LongestSubstringWithoutRepeatingCharacters {

    public static int lengthOfLongestSubstring(String s) {
        int start = 0, max = 0;
        s = s + " ";

        for (int i = 1; i < s.length(); i++) {
            String str = s.substring(start, i);
    
            max = Math.max(str.length(), max);

            /**
             * 문자가 비교할 문자열에 포함이 되어 있다면
             * 비교할 문자열의 시작위치를 포함된 문자열의 마지막 인덱스 다음 값으로 한다.
             * abcabcbb 에서 
             * ---abcb- abc에 b가 포함 되었는데 잘라낸 문자열 abc에서 b의 인덱스는 1이 된다
             * 우리가 원하는건 abcabcbb에서 
             *             ----b--- 
             *                 4    4번 index값을 찾아서 다음 비교를 시작 해야하므로 
             * 처음부터 현재까지 즉 abcabcbb 중에서 abcabc 까지에서  b 중 마지막에 있는 b의 index를 찾아낸다 
             * 그래야 abcabcbb 
             *      -----cb 와 b를 다시 검사 할 수 있다.
            */    
            if (str.indexOf(s.charAt(i)) > -1) {
                start = s.substring(0, i).lastIndexOf(s.charAt(i)) + 1;
            }
        }

        return max;
    }
}
```

