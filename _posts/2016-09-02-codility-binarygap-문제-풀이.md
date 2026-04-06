---
title: "Codility - BinaryGap 문제 풀이 (난이도 : 하)"
date: 2016-09-02 18:05:08 +0900
categories: [알고리즘]
---

# [Codility - BinaryGap](https://codility.com/programmers/task/binary_gap/) 문제 풀이 (난이도 : 하)

사이트에서 문제 복제 및 배포(publish)는 허가하지 않는다니 그냥 풀이만......

```
class Solution {
    
    public int solution(int N) {
        int mode = 0;
        
        //remove 0 gaps without 1
        do{
            mode = N%2;
            N = N/2;
        }while(mode == 0);
        
        //count 0 gaps and get max 0 gap count
        int maxGapCount = 0;
        int gapCount = 0;
        while(N != 0){
            mode = N%2;
            N = N/2;
            if(mode == 1){
                maxGapCount = Math.max(maxGapCount,gapCount);
                gapCount = 0;
            }
            else{
                gapCount++;
            }
        }
        
        return maxGapCount;
    }
}
```
