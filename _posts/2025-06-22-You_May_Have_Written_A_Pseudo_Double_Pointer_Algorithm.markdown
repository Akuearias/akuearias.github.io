---
layout: post
title:  "[BS, D&C, 2P] Codings 2. You May Have Written a Pseudo Double Pointer Algorithm..."
date:   2025-06-22 15:00:00 -0400
categories: Codings
---

When it comes to the double pointer technique (or more specifically, the sliding window variant), many people might think of it like this:

```python

from typing import List

def Dummy(nums: List[List[int]]):
    m = len(nums)
    n = len(nums[0])
    for i in range(m):
        for j in range(n):
            pass # Do something here...

```
At first glance, there are two variables, i and j, iterating through the data. Some beginners might confidently say, “Isn’t this just a double pointer?”

Wrong. Absolutely wrong.

This is **NOT** a double pointer approach; it’s a **brute force enumeration**. To be precise, this is brute force disguised as “double pointer” — a **pseudo double pointer**.

A real double pointer is not merely the presence of two variables, but more importantly, the coordinated movement logic between them — they don’t run in nested loops, but advance **cooperatively**. By doing so, double pointer reduces unnecessary traversals and reduces the time complexity of the algorithm.

For example:

#### LeetCode #15. 3Sum

Let’s first look at a wrong solution — or more precisely, a solution with excessively high time complexity that will likely exceed time limits on large inputs.

```python

from typing import List

class Solution:
    def threeSum(self, nums: List[int]) -> List[List[int]]:
        nums.sort()
        triples = []
        nums2 = nums.copy()
        for i in range(len(nums)):
            nums2.remove(nums[i])
            mat = [[None] * (len(nums) - 1) for _ in range(len(nums) - 1)]
            for r in range(len(nums) - 1):
                for c in range(len(nums) - 1):
                    if r != c:
                        mat[r][c] = nums[i] + nums2[r] + nums2[c]
                        if mat[r][c] == 0:
                            if not triples.__contains__(sorted([nums[i], nums2[r], nums2[c]])):
                                triples.append([nums[i], nums2[r], nums2[c]])
            nums2.append(nums[i])
        return triples

```

This algorithm:

- Tries finding all triplets summing to zero via enumerating all possible pairs _(r, c)_ for each fixed _i_;
- Uses a nested loop inside loops, resulting in O(n^3) time complexity.

In addition:

- The _triples_ list is checked linearly to avoid duplicates, which reduces efficiency as well.

This is often a misunderstanding that beginners make when learning double pointers - simply thinking that "double pointers are two pointers moving from one end to the other". 
We cannot say that the statement "double pointers are two pointers moving" is wrong in itself, but this misunderstanding will not only **increase the time complexity of the algorithm**, 
but also seriously affect the running efficiency of the program when the amount of data is large.

Then let's have a glance on the official solution to this problem.

```python

from typing import List

class Solution:
    def threeSum(self, nums: List[int]) -> List[List[int]]:
        n = len(nums)
        nums.sort()
        ans = list()
        
        for first in range(n):
            if first > 0 and nums[first] == nums[first - 1]:
                continue
            
            third = n - 1
            target = -nums[first]
            
            for second in range(first + 1, n):

                if second > first + 1 and nums[second] == nums[second - 1]:
                    continue
                while second < third and nums[second] + nums[third] > target:
                    third -= 1
                if second == third:
                    break
                if nums[second] + nums[third] == target:
                    ans.append([nums[first], nums[second], nums[third]])
        
        return ans

```

The real double pointer is **NOT** just two pointers moving, but **more importantly**, they move forward together **"strategically and planned"** - the two move forward together and finally converge to a certain target position.

For this problem, the double pointers are the second and third indices that move within the sorted array:

•	We fix the first pointer first and then use two pointers second and third to find pairs such that the sum of nums[second] + nums[third] equals the target -nums[first].

•	The pointer second moves forward from the left (just after first), while third moves backward from the right end.

•	By comparing the sum nums[second] + nums[third] to the target, we decide which pointer to move:

    • If the sum is greater than the target, move third backward to reduce the sum.
    • If the sum is less than the target, move second forward to increase the sum.
•	This coordinated movement prunes unnecessary checks and avoids nested iterations over all pairs, reducing time complexity from O(n³) to O(n²).

•	The pointers move cooperatively: each step depends on the comparison result, ensuring we explore only promising pairs.

In summary, this demonstrates how a real double pointer algorithm:

	• Operates on sorted data to leverage order,
	• Uses two pointers that strategically move towards each other,
	• Avoids brute-force enumeration by skipping impossible combinations efficiently.
As the two pointers move forward together, there are 3 kinds of moving strategies:

### **1. Face-to-face** 

The two pointers move from the two ends of the sequence to the middle, and are often used to find combinations or intervals that meet certain conditions in an ordered array.

Example: LeetCode #11

```python

from typing import List

class Solution:
    def maxArea(self, height: List[int]) -> int:
        l, r = 0, len(height) - 1
        ans = 0
        while l < r:
            area = min(height[l], height[r]) * (r - l)
            ans = max(ans, area)
            if height[l] <= height[r]:
                l += 1
            else:
                r -= 1
        return ans

# References: https://leetcode.cn/problems/container-with-most-water/solutions/207215/sheng-zui-duo-shui-de-rong-qi-by-leetcode-solution/

```

First, we set two pointers at the far left and far right.

Next, we calculate the area corresponding to the two pointers and try to update the maximum area. Then, according to the "short board principle", we move the pointer at the shorter end one step inward, because the capacity of the barrel is limited by the shortest plank. Moving the shorter end has the opportunity to find a higher "short board", thereby increasing the area.

We repeat this process, and the two pointers gradually converge to the middle until they meet.

The key to this is that each step of movement is trying to find a larger area, so the maximum area obtained in the end is accurate.

Compared to the brute force algorithm, the double pointer method here avoids a lot of unnecessary traversals and reduces the overall time complexity to O(n). 
However, the brute force method requires nested loops on two variables, and the time complexity is as high as O(n²), which is far less efficient than the double pointer method.

### **2. Sliding Window** 

In addition to two pointers moving toward each other, the second common pattern is two pointers moving in the same direction, which forms what we call a sliding window. 
This technique is especially useful when we need to track a subarray or substring that satisfies certain conditions while scanning the input only once, such as Rabin-Karp pattern matching.

Unlike the “facing” two-pointer method that reduces search space, the sliding window maintains a dynamic “window” between two pointers — one that expands or shrinks depending on the situation.

Example: LeetCode #3

```python

class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        occ = set()
        n = len(s)

        rk, ans = -1, 0
        for i in range(n):
            if i != 0:
                occ.remove(s[i - 1])
            while rk + 1 < n and s[rk + 1] not in occ:
                occ.add(s[rk + 1])
                rk += 1
            ans = max(ans, rk - i + 1)
        return ans

# https://leetcode.cn/problems/longest-substring-without-repeating-characters/solutions/227999/wu-zhong-fu-zi-fu-de-zui-chang-zi-chuan-by-leetc-2/

```

We use two pointers:

    • left marks the start of the current window,
    • right moves forward to expand the window.

•	If we detect a duplicate character, we move the left pointer forward and shrink the window until the duplicate is removed.

•	seen is a set that keeps track of the characters in the current window.

•	At each step, we update the maximum length of the substring seen so far.

The key idea here is: the window always contains a valid substring (no repeating characters), and both pointers move only forward, giving us an O(n) time complexity.

What Makes This “Sliding Window”?

	•	The two pointers move in the same direction, forming a moving “window”.
	•	The window expands when the current condition is met and shrinks when it breaks.
	•	The data structure (set, dict, counter, etc.) inside the window helps maintain the validity of the window.
	•	No nested loops are needed — each character is processed at most twice (once by right, once by left), keeping the solution linear.

This kind of coordinated forward movement is the essence of sliding window — a powerful subtype of the double pointer technique that avoids redundant work by maintaining a local view.

### **3. Slow-and-quick (Or Floyd algorithm)** 

While both pointers move in the same direction, they advance at different speeds. This approach is useful when you want one pointer to “chase” the other and detect a structural pattern, such as a cycle or midpoint in a linked list.

Example: LeetCode #141

```python

class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None

class Solution:
    def hasCycle(self, head: ListNode) -> bool:
        slow = head
        fast = head

        while fast and fast.next:
            slow = slow.next         # move 1 step
            fast = fast.next.next    # move 2 steps
            if slow == fast:
                return True

        return False

```

Here:

	• The slow pointer moves one step at a time.
	• The fast pointer moves two steps at a time.
	• If there is a cycle in the linked list, eventually the fast pointer will “lap” the slow one and they will meet.
	• If there is no cycle, the fast pointer will reach the end (None) first.

This method is elegant because it doesn’t require any extra space — unlike the brute-force approach of tracking visited nodes using a set — and runs in O(n) time with O(1) space.

The back-to-back double pointer technique is used when:

	• You need to detect intersections, cycles, or time-based collisions.
	• You want to use constant space instead of hash sets or extra structures.
	• You rely on relative speeds or positions to determine a structural property.

**Why is it called "Slow-and-fast"?**

• The two pointers are initialized together but immediately move apart.

• They never “follow each other” or “move toward each other” — instead, they diverge.

• The relative speed difference causes them to eventually meet only if a cycle exists, much like two runners on a track.

Common patterns that use this technique include:

	• Cycle detection in linked lists
	• Finding the middle of a linked list
	• Removing the nth node from the end (two-pointer gap)

## Summary :Not All algorithms having two variables traversing are "double pointer algorithms"

In this post, we’ve dismantled a common misunderstanding — that any code using two variables to iterate must be a “double pointer” solution. 
As we’ve seen, true double pointer techniques are not just about having two pointers, but more importantly about how those pointers collaborate, move strategically, and reduce unnecessary work.

We explored three major coordination modes of double pointers:

	1.	Facing Each Other: Two pointers moving toward each other to narrow down a search space (e.g., 3Sum, Container With Most Water).
	2.	Moving in the Same Direction: Includes sliding windows and slow-fast pointer techniques for substring, cycle detection, or structural insights.
	3.	Slow-and-Quick: A special case of same-direction movement where one pointer chases the other, often used in linked list problems.

Understanding these patterns not only helps avoid brute force traps, but also empowers us to design elegant and efficient solutions to many common algorithm problems.

Therefore, when writing double pointer algorithms, consider:

    • Are you really considering how to cooperate the two pointers? 
    • Or are you still using a brute force just by looping two variables over and over?
