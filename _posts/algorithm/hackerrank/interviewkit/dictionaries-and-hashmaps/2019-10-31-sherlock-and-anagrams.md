---
layout: post
title: "Sherlock and anagrams - hackerank [코테/알고리즘]"
description: "코테 코딩테스트 해커랭크 hackerank Sherlock and anagrams 문제의 풀이와 해설을 합니다."
author: kimchanjung
date: 2020-05-08 00:00:00 +0900
top: true
categories: algorithm/hackerrank/interviewkit/dictionaries-and-hashmaps
published: true
---

## 특정 문자열의 개별 문자를 쌍으로 구성하는 경우 가능한 케이스의 개수를 찾는문제 

[문제바로가기](https://www.hackerrank.com/challenges/sherlock-and-anagrams)

### 문제 조건
* 주어진 문자열 자체의 순서는 변경할 수 없다 
> "abcd" -> "bacd" 이렇게 변경 할수 없다.
* 개별 문자열은 순서는 상관없다
> "ifailuhkqq"
> "ifa" == "fai"는 같은 것으로 본다 => ["ifa", "fai"] 아나그램 조합이 허용된다.
>  
* 문자열 "abba"의 경우 가능한 케이스는

```bash
[a,a] [ab, ba] [b,b] [abb, bba]  => 4가지
```
* 문자열 "ifailuhkqq" 경우는

```bash
[i,i] [g,g] [ifa, fai] => 처럼 3가지다  
[ifa, fai] => 순서는 다르지만 각각 i, f, a 포함 되었으므로 같은 아나그램으로 본다.
```

* 문자열 "kkkk" 경우는

```bash
[k,k] [k,k] [k,k] [k,k]
[1,2] [2,3] [3,4] [1,4] 
처럼 개별 자리의 문자로 인식한다
즉 [k,k]가 있다고 [k,k] 케이스가 또있네 라고 햇갈릴수 있는데
각각 개별 자리의 문자를 독립적으로 본다
```

### 해결책
문자열 "aaa"가 있을 때 각각 문자열a, aa, aaa를 아래와 같이 저장한다.
> map에 key = 문자, value = 각 문자열의 개수(1,2,3,...n 수열의 차수로 사용될 것이다.)

문자열 "aaaa" 경우 개별 문자열의 개수 아래와 같다.
```bash
a    = 4개 
aa   = 3개 
aaa  = 2개
aaaa = 1개
```
문자열 "aaaa" 개별 문자열 개수가 도출 된 이유를 설명하면
```bash
1   2   3   4 (문자열 순번)
a   a   a   a

문자                                개수(n차)  조합개수
a    = a(1), a(2), a(3), a(4)    = 4개     = 6개
aa   = aa(1,2), aa(2,3), aa(3,4) = 3개     = 3개
aaa  = aaa(1,2,3), aaa(2,3,4)    = 2개     = 1개
aaaa = aaaa(1,2,3,4)             = 1개     = 0개 (아나그램으로 조합할 수가 없다 1개만 존재하므로)
```


이렇게 도출된 개수를 수열로 보면
```bash
aaaa  aaa   aa     a (각문자)
 1     2     3     4 (n차)
 0     1     3     6 (조합개수) = 총 10개(정답)
```
조합개수 1, 3, 6을 가만히 보면 
```bash
1 2 3 (n차)
1 3 6 => 계차 수열임
 2 3  => 공차가 1인 등차수열을 가지는 
  1   => 공차(1)
``` 

더 자세히 보면 공차가 1인 등차수열을 가지는 계차수열이기도 하지만
각각의 일반항이 n차 자연수의 합과 같다는 것을 알수 있다.
```bash
1     2     3   (n 차)
1     3     6   (n일 때 일반항의 개수(자연수의 합))
1    1+2  1+2+3 (n이 1일 때 일반항 1, n이 2일 때 일반항 3(1+2 => 자연수의 합공식) )
```

자연수의 합 공식은  
> n(n+1) / 2 

자연수의 합공식을 이용하여 각 일반항을 모두 더해주면 정답이 나오는데
n 차수가 다르다.
```bash
원래는 차수 1부터 순차적으로 증가할 때 각 n차의 일반항의 개수가 아래와 같은데
1  2  3 (n차)
1  3  6 (조합개수) = 총 10개(정답)

이 경우 차수가 +1씩 크다
1  2  3  4 (n차)
0  1  3  6 (조합개수) = 총 10개(정답)
```
그래서 자연수 합 공식에서 n =2 일때 1이 되어야 하므로 n-1을 해주어야 한다.

자연수 합 공식 n(n+1) / 2 에서 n을 -1 씩 하게 되면    
> (n-1){(n-1)+1} / 2 

되고 전개 해서 풀이하면 결국    
> n(n-1) / 2 

와 같은 식이 도출된다.
 
n(n-1)/2 공식을 적용하여 aaa 3일때 각 조합은 
```bash
a   = 3개 이고 공식 적용하면 = 3
aa  = 2개 이고 공식적용 = 1
aaa = 1개라 공식적용 = 0
1+3 = 모두 더하면 4가 된다.
```

### 소스코드

```java
public class SherlockAndAnagrams {
    public static int sherlockAndAnagrams(String s) {
        HashMap<String, Integer> dic = new HashMap<>();

        for (int i = 0; i < s.length(); i++) {
            for (int j = i + 1; j < s.length() + 1; j++) {
                String str = s.substring(i, j).chars()
                        .sorted()
                        .mapToObj(String::valueOf)
                        .collect(Collectors.joining());
                dic.merge(str, 1, (c, d) -> c + 1);
            }
        }
        
        return dic.values().stream()
                .mapToInt(n -> n * (n - 1) / 2)
                .sum();
    }
}
```