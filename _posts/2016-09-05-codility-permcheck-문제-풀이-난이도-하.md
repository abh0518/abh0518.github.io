---
layout: post
title: "Codility - PermCheck 문제 풀이 (난이도 : 하)"
date: 2016-09-05 14:35:52 +0900
categories: [알고리즘]
---

[Codility - PermCheck](https://codility.com/programmers/task/perm_check/) 문제 풀이 (난이도 : 하)

```
// you can also use imports, for example:
// import java.util.*;

// you can write to stdout for debugging purposes, e.g.
// System.out.println("this is a debug message");

class Solution {
    public int solution(int[] A) {
        // write your code in Java SE 8
        
        boolean checker[] = new boolean;
        
        for(int i = 0 ; i < A.length; i++){
            int value = A;
            if(value < checker.length){
                checker = true;   
            }
        }
        
        int result = 1;
        for(int i = 1 ; i < checker.length ; i++){
            if(checker == false){
                result = 0;
                break;
            }
        }
        
        return result;
    }
}
```
