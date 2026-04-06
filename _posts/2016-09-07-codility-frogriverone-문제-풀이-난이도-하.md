---
title: "Codility - FrogRiverOne 문제 풀이 (난이도 : 하)"
date: 2016-09-07 13:13:32 +0900
categories: [알고리즘]
---

[Codility - FrogRiverOne](https://codility.com/programmers/task/frog_river_one/) 문제 풀이 (난이도 : 하)

```java
class Solution {
    public int solution(int X, int[] A) {
        // write your code in Java SE 8
        int leafList[] = new int;
        for(int i = 0 ; i < leafList.length ; i++){
            leafList = -1;   
        }
        
        for(int i = 0 ; i < A.length ; i++){
            int leaf = A;
            if(leaf <= X ){
                if(leafList == -1){
                    leafList = i;
                }
                else{
                    leafList = Math.min(i, leafList);    
                }
            }
        }
        
        int result = 0;
        for(int i = 1 ; i < leafList.length ; i++){
            if(leafList == -1){
                result = -1;
                break;
            }
            result = Math.max(result, leafList);
        }
        
        return result;
    }
}
```
