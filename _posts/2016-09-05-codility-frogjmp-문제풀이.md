---
layout: post
title: "Codility - FrogJmp 문제풀이 (난이도 : 하)"
date: 2016-09-05 13:28:45 +0900
categories: [알고리즘]
---

# [Codility - FrogJmp](https://codility.com/programmers/task/frog_jmp/) 문제풀이 (난이도 : 하)

```
// you can also use imports, for example:
// import java.util.*;

// you can write to stdout for debugging purposes, e.g.
// System.out.println("this is a debug message");

class Solution {
    public int solution(int X, int Y, int D) {
        // write your code in Java SE 8
        int distance = Y - X;
        return distance%D == 0 ? distance/D : distance/D + 1;
    }
}
```
