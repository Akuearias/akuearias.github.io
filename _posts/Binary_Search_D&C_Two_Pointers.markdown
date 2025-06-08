---
layout: post
title:  "[BS, D&C, 2P] Codings 1. Binary Search - Fantastic method"
date:   2025-05-24 23:52:52 -0400
categories: Codings
---


In exercises in LeetCode, there are three common labels for exercises:

**Binary Search**

**Divide and Conquer (D&C)**

**Two Pointers**

Though sometimes they share things in common, they are three distinct methods in programming.
Also, these algorithms are not limited in particular problem solutions, but very flexible in many kinds of problems,

In this blog, I will explain the principles of these three methods, and how they can be used in a flexible way
in programming.

In this part I will discuss Binary Search.

## Binary Search

In algorithm, Binary Search is originally a searching algorithm used in a sorted array,
which sorks by repeatedly dividing the search interval in halves.

The key point of Binary Search is to make use of a **sorted** array,
and the advantage of Binary Search algorithm is that the time complexity can
be reduced to O(log n), avoiding unnecessary searching in Brute Force.

The first thing we can think about Binary Search is like "Price guessing contest" - the
participant guesses the price of a product whose price is between 1 and 1000, within limited attempts (10 in this case).
In this case, if we use Brute Force algorithm to do so, it will be very inefficient and may run out of attempts.
On the contrary, Binary Search is good at reducing unnecessary attempts:

`Suppose that the price of the product is 404.`

`Attempt 1: 500 -> too big` (0, 1000) -> 500

`Attempt 2: 250 -> too small` (0, 500) -> 250

`Attempt 3: 375 -> too small` (250, 500) -> 375

`Attempt 4: 437 -> too big` (375, 500) -> 437

`Attempt 5: 406 -> too big` (375, 437) -> 406

`Attempt 6: 390 -> too small` (375, 406) -> 390

`Attempt 7: 398 -> too small` (390, 406) -> 398

`Attempt 8: 402 -> too small` (398, 406) -> 402

`Attempt 9: 404 -> Correct answer!` (402, 406) -> 404

In such a game, the number of steps using Binary Search is around log 1000 = 9.966, between 9 and 10.

Maybe for most of us, the only image that comes to mind when we hear “binary search” is the classic “guess a number” or other textbook-style halving problems.

But binary search is far more versatile. It often appears in disguise — and sometimes, when you spot it, you’ll think, “Wow, that was unexpected!”

Let’s take a look at a problem on LeetCode:

⸻

LeetCode #1760. Minimum Limit of Balls in a Bag

You’re given an array nums, where nums[i] represents the number of balls in the i-th bag, and an integer maxOperations.
Each operation allows you to pick a bag and split it into two bags (each with a positive number of balls).
After all operations, your penalty is defined as the maximum number of balls in any single bag.
Your task: minimize this penalty within at most maxOperations splits.

Maybe most will us will wonder:

- How is it related to binary search?
- Should we use Brute Force? Backtracking? Greedy?

However, we may have ignored the following **key points**:

- The penalty value is bounded.
- The lower the penalty you want, the more operations it costs.

Based on the key points above, we may reconsider this problem as a Binary Search on Answer (BSOA):

- Minimize the maximum of the number of balls -> Judge whether the target maximum max_ is available based on the given attempts. If yes, we can reduce the max_, otherwise increase it.

Then we have a function to judge whether max_ is available:
- For every original bag size _x_, we divide it into several bags with the maximum number of balls in the bag _mid_. Then the number of operations is:
- `splits = LowerBound(x / mid) - 1
- If mid is not more than maxOperations, mid is a possible solution.

Then, for binary search:

```python
from typing import List


def minimumSize(nums: List[int], maxOperations: int) -> int:
    def canAchieve(mid: int) -> bool:
        ops = sum((x - 1) // mid for x in nums)
        return ops <= maxOperations

    left, right = 1, max(nums)
    while left < right:
        mid = (left + right) // 2
        if canAchieve(mid):
            right = mid
        else:
            left = mid + 1
    return left

```

Instead of focusing on how to separate the balls, this algorithm:
- considers an upper bound of the maximum number of balls;
- infers whether the number of steps for separating balls to make this maximum number is within the maximum operations.

This is the **core** of binary search - considering from our purpose instead of the action, and then converge via binaries.


#### Sometimes, Binary Search is not that evident. However, they may use the soul of binary search - pruning.

⸻


LeetCode Exercise #456. 132 Pattern

Given an array of n integers `nums`, a `132 pattern` is a subsequence of three integers `nums[i]`, `nums[j]` and `nums[k]` 
such that `i < j < k` and `nums[i] < nums[k] < nums[j]`.

Return `true` if there is a `132 pattern` in `nums`, otherwise, return `false`.

Firstly, for this problem, we may think:

- How is this related to Binary Search?
- 1-3-2 pattern? This is so easy! Just Dynamic Programming!

Do NOT be confused! This is not a simple continuous 1-3-2 (i.e. peak). Even **NON-CONTINUOUS** subsequence will be a 132 pattern
as long as it satisfies the conditions in the question!

So in this case dynamic programming does not work!

Here is one way to solve it via pruning:

We will traverse the array from right to left.

i.e.:
- nums[i] is our candidate L;
- Then we maintain a stack to store "potential M";
- At the same time we will use variable max_k to record maximum M that meets our requirements.


Steps:

- Initialization of max_k: 
`max_k = -math.inf`

Then we traverse the array backwards:
- if nums[i] < max_k: This is a legal 1-3-2 pattern. Return True.
- Otherwise:
  - As long as the stack is not empty and the top of the stack is less than nums[i], we pop the stack and update max_k.
  - Push nums[i] into the stack, meaning it may become the future candidate of R.

Code:

```python
from typing import List


class Solution:
    def find132pattern(self, nums: List[int]) -> bool:
        stack = []
        max_k = float('-inf')  # record the max nums_k
        
        for i in range(len(nums) - 1, -1, -1):  # Traverse from R to L
            if nums[i] < max_k:
                return True  # This means we find a legal 1-3-2.
            while stack and stack[-1] < nums[i]:
                max_k = stack.pop()  # The current value is less than nums[i], may become nums[k]
            stack.append(nums[i])  # The current value is set as possible nums[j]
        
        return False

```
 




