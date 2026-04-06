---
title: "Codility - CyclicRotation 문제 풀이 (난이도 : 하)"
date: 2016-09-02 18:10:14 +0900
categories: [알고리즘]
---

# [Codility - CyclicRotation](https://codility.com/programmers/task/cyclic_rotation/) 문제 풀이 (난이도 : 하)

문제 유출은 안된다니 풀이만......

```
class Solution {
    public int[] solution(int[] A, int K) {
        // write your code in Java SE 8
        
        int[] a = new int;
        
        for(int i = 0 ; i < A.length ; i++){
            a[(i+K)%a.length] = A;
        }
        
        return a;
    }
}
```
