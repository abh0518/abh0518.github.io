---
layout: post
title: "LeedCode - Longest Substring Without Repeating Characters 문제풀이 (난이도:중)"
date: 2016-10-04 17:24:24 +0900
categories: [알고리즘]
---

[LeedCode - Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/) 문제풀이 (난이도:중)

Given a string, find the length of the longest substring without repeating characters.

Examples:
Given "abcabcbb", the answer is "abc", which the length is 3.
Given "bbbbb", the answer is "b", with the length of 1.
Given "pwwkew", the answer is "wke", with the length of 3. Note that the answer must be a substring, "pwke" is a subsequence and not a substring.

```
public class Solution {
    public int lengthOfLongestSubstring(String s) {
        boolean[] cache = new boolean;
        int count = 0;
        int maxCount = 0;
        int head = 0, tail = 0;
        while(head < s.length()){
            if(cache != true){
                cache = true;
                maxCount = Math.max(head-tail+1, maxCount);
                head++;
            }
            else{
                cache = false;
                tail++;
            }
        }
        
        return maxCount;
    }
}
```
