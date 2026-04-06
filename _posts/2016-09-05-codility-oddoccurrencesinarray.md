---
layout: post
title: "Codility - OddOccurrencesInArray 문제 풀이 (난이도 : 하)"
date: 2016-09-05 10:34:43 +0900
categories: [알고리즘]
---

# [Codility - OddOccurrencesInArray](https://codility.com/programmers/task/odd_occurrences_in_array/) 문제 풀이 (난이도 : 하)

```
// you can also use imports, for example:
// import java.util.*;
import java.util.*;

// you can write to stdout for debugging purposes, e.g.
// System.out.println("this is a debug message");
class Solution {
    public int solution(int[] A) {
        // write your code in Java SE 8
        HashMap<Integer, Boolean> pairedMap = new HashMap();

        for(int i = 0 ; i < A.length ; i++){
            int occurredNumber = A;
            Boolean paired = pairedMap.get(occurredNumber);
            if(paired == null || paired == true){
                pairedMap.put(occurredNumber, false);
            }
            else{
                pairedMap.put(occurredNumber, true);
            }
        }

        Iterator iterator = pairedMap.keySet().iterator();
        int minValue = Integer.MAX_VALUE;
        while(iterator.hasNext()){
            int key = iterator.next();
            Boolean paired = pairedMap.get(key);
            if(paired == false){
                minValue = Math.min(minValue, key);
            }
        }

        return minValue;
    }
}
```
