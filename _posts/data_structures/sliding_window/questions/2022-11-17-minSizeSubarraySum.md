---
layout: post
title: "Leetcode 209 - Minimum Size Subarray Sum"
categories: [algorithms,sliding_window]
tags: [python, algorithms, sliding_window, prefix_sum]
---

Given an array of positive integers nums and a positive integer target, return the minimal length of a subarray whose sum is greater than or equal to target. If there is no such subarray, return 0 instead.

Full question details [here](https://leetcode.com/problems/minimum-size-subarray-sum/).

```python
from typing import List 

class Solution:
    def minSubArrayLen(self, target: int, nums: List[int]) -> int:
        # Require to get the minimum length of contiguous subarray of which the sum is greater than
        # or equal to target
        # We can initialise a prefixSum array
        prefixSum = [0]
        for i in range(len(nums)):
            prefixSum.append(prefixSum[-1] + nums[i])
            
        # 0 2 5 6 8 12 15
        # s = 0, e = 1
        # e -> 4 
        # Create two pointers to keep track of the size of the array
        minSize = None
        start, end = 0, 1
        while start < end < len(prefixSum):
            currentTotal = prefixSum[end] - prefixSum[start]
            if currentTotal < target:
                # end pointer walks
                end += 1
                
            else:
                # We've found a subarray
                if minSize is None:
                    minSize = end - start
                else:
                    minSize = min(minSize, end - start)
                # increment start 
                start += 1
        return minSize if minSize else 0
```