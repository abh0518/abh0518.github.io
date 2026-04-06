---
title: "Codility -  TapeEquilibrium 문제 풀이 (난이도 : 하)"
date: 2016-09-05 10:56:33 +0900
categories: [알고리즘]
---

# [Codility -  TapeEquilibrium](https://codility.com/programmers/task/tape_equilibrium/) 문제 풀이 (난이도 : 하)

```
// you can also use imports, for example:
// import java.util.*;

// you can write to stdout for debugging purposes, e.g.
// System.out.println("this is a debug message");

class Solution {
    public int solution(int[] A) {
        // write your code in Java SE 8
        
        int front = 0;
        int back = 0;
        for(int i = 0 ; i < A.length ; i++){
            back += A;   
        }
        
        int minDiff = Integer.MAX_VALUE;
        int p = 1;
        for(int i = 1 ; i < A.length ; i++){
            front += A;
            back -= A;
            p = i;
            minDiff = Math.min(minDiff, Math.abs(front - back));
        }
        
        return minDiff;
        
    }
}
```
