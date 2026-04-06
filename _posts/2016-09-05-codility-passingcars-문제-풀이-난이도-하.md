---
layout: post
title: "Codility - PassingCars 문제 풀이 (난이도 : 하)"
date: 2016-09-05 15:10:21 +0900
categories: [알고리즘]
---

# [Codility - PassingCars](https://codility.com/programmers/task/passing_cars/) 문제 풀이 (난이도 : 하)

```
// you can also use imports, for example:
// import java.util.*;

// you can write to stdout for debugging purposes, e.g.
// System.out.println("this is a debug message");

class Solution {
    public int solution(int[] A) {
        // write your code in Java SE 8
        int toEast = 0;
        
        long pairCount = 0;
        
        for(int i = 0 ; i < A.length ; i++){
            if(A == 0){
                toEast++; 
            } else{
                pairCount += toEast; 
            }
        }

        if(pairCount > 1000000000){
            return -1;   
        }
        
        return (int)pairCount;
    }
}
```
