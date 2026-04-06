---
title: "Codility - MaxCounters 문제 풀이 (난이도:중)"
date: 2016-09-07 16:18:18 +0900
categories: [알고리즘]
---

[Codility - MaxCounters](https://codility.com/programmers/task/max_counters/) 문제 풀이 (난이도:중)

```
// you can also use imports, for example:
// import java.util.*;

// you can write to stdout for debugging purposes, e.g.
// System.out.println("this is a debug message");

class Solution {
    public int[] solution(int N, int[] A) {
        
        int maxCounter = N+1;
        int counters[] = new int;
        for(int i = 0 ; i < counters.length ; i++){
            counters = 0;   
        }
        
        int nextMax = 0;
        int curMax = 0;
        for(int i = 0 ; i < A.length ; i++){
            int counterNumber = A;
            int counterIndex = counterNumber - 1;
            
            if(counterNumber < maxCounter){
                if(counters <= curMax){
                    counters = curMax;
                }
                counters++;
                nextMax = Math.max(nextMax, counters);
            }
            else{
                curMax = nextMax;
            }
        }
        
        for(int i = 0 ; i < counters.length ; i++){
            if(counters < curMax){
                counters = curMax;   
            }
        }
        
        return counters;
    }
}
```
