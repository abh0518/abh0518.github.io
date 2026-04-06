---
title: "Codility - CountDiv 문제 풀이 (난이도 : 하)"
date: 2016-09-07 11:43:13 +0900
categories: [알고리즘]
---

[Codility - CountDiv](https://codility.com/programmers/task/count_div/) 문제 풀이 (난이도 : 하)

```java
// you can also use imports, for example:
// import java.util.*;

// you can write to stdout for debugging purposes, e.g.
// System.out.println("this is a debug message");

class Solution {
    public int solution(int A, int B, int K) {
        // write your code in Java SE 8
        int result = B/K + 1;
        if(A != 0){
            result -= ((A-1)/K + 1);
        }
        return result;
    }
}
```
