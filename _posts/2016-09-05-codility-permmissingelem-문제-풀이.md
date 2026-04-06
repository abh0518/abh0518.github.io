---
layout: post
title: "Codility - PermMissingElem 문제 풀이 (난이도 : 하)"
date: 2016-09-05 14:05:44 +0900
categories: [알고리즘]
---

[Codility - PermMissingElem](https://codility.com/programmers/task/perm_missing_elem/) 문제 풀이 (난이도 : 하)

```
// you can also use imports, for example:
// import java.util.*;

// you can write to stdout for debugging purposes, e.g.
// System.out.println("this is a debug message");

class Solution {
    public int solution(int[] A) {
        // write your code in Java SE 8
        
        boolean occurred[] = new boolean;
        
        for(int i = 0 ; i < A.length ; i++){
            int occurredNumber = A;
            occurred = true;
        }
        
        for(int i = 1 ; i < occurred.length ; i++){
            if(!occurred){
                return i;
            }
        }
        
        return 0;
    }
}
```
