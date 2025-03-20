---
title: "Leetcode 496: Next Greater Element I -- Performance Boost with Less Branches & Monotonic Stack"
date: 2025-03-20
---

## Problem

The next greater element of some element `x` in an array is the first greater element that is to the right of `x` in the same array.

You are given two distinct 0-indexed integer arrays `nums1` and `nums2`, where `nums1` is a subset of `nums2`.

For each `0 <= i < nums1.length`, find the index `j` such that `nums1[i] == nums2[j]` and determine the next greater element of `nums2[j]` in `nums2`. If there is no next greater element, then the answer for this query is -1.

Return an array ans of length `nums1.length` such that `ans[i]` is the next greater element as described above.



**Example 1:**

> Input: `nums1 = [4,1,2]`, `nums2 = [1,3,4,2]`
> Output: `[-1,3,-1]`
> Explanation: The next greater element for each value of nums1 is as follows:
> - 4 is underlined in `nums2 = [1,3,4,2]`. There is no next greater element, so the answer is -1.
> - 1 is underlined in `nums2 = [1,3,4,2]`. The next greater element is 3.
> - 2 is underlined in `nums2 = [1,3,4,2]`. There is no next greater element, so the answer is -1.


**Example 2:**

> Input: `nums1 = [2,4]`, `nums2 = [1,2,3,4]`
> Output: `[3,-1]`
> Explanation: The next greater element for each value of nums1 is as follows:
> - 2 is underlined in `nums2 = [1,2,3,4]`. The next greater element is 3.
> - 4 is underlined in `nums2 = [1,2,3,4]`. There is no next greater element, so the answer is -1.


Constraints:

- `1 <= nums1.length <= nums2.length <= 1000`
- `0 <= nums1[i]`, `nums2[i] <= 104`
- All integers in `nums1` and `nums2` are unique.
- All the integers of `nums1` also appear in `nums2`.

## Solution

With the *monotonic stack* technique, we could achieve O(n) time complexity.

```go
func nextGreaterElement(nums1 []int, nums2 []int) []int {
	greater := map[int]int{}
	decreasing := []int{}
	for _, n := range nums2 {
		for len(decreasing) > 0 && n > decreasing[len(decreasing)-1] {
			greater[decreasing[len(decreasing)-1]] = n
			decreasing = decreasing[:len(decreasing)-1]
		}
		decreasing = append(decreasing, n)
	}

	var res []int
	// !!!!!!!!!!!
	for _, n := range nums1 {
		if v, ok := greater[n]; ok {
			res = append(res, v)
		} else {
			res = append(res, -1)
		}
	}

	return res
}
```

But the data shows it only beats **17%** solutions in terms of runtime.

What happened?

Notice the `!!!!!!!!!!!` comment line in the solution, the code under it is very suspicious, as we all (maybe) know,
`if` statement inside a loop causes performance penalties, because of the *branch prediction failure*,
okay, let's eliminate the branch: instead of determin whether the number exists in the map, we loop over the stack,
and assign `-1` to the corresponding element in the map **in advance**.

```go
func nextGreaterElement(nums1 []int, nums2 []int) []int {
	greater := map[int]int{}
	decreasing := []int{}
	for _, n := range nums2 {
		for len(decreasing) > 0 && n > decreasing[len(decreasing)-1] {
			greater[decreasing[len(decreasing)-1]] = n
			decreasing = decreasing[:len(decreasing)-1]
		}
		decreasing = append(decreasing, n)
	}

	// added here
	for _, n := range decreasing {
		greater[n] = -1
	}

	var res []int
	for _, n := range nums1 {
		res = append(res, greater[n])
	}

	return res
}
```

Submit it again, and boom! **100%** beats!

Algorithms aren't the only core of performance, but also subtle places like this, which directly related to the
structure of modern CPU.
