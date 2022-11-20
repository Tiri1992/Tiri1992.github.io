---
layout: post
title: "Binary Search - Introduction"
mermaid: true
categories: [algorithms,binary_search]
tags: [python, algorithms, binary_search, recursion]
---

A binary search algorithm allows you to search an ordered list of elements using divide and conquer strategy. But before we dive in, lets have a gentle reminder of a linear search.

## Whats linear search?

* Iterate through an array sequentially looking for your target element
* Runtime of `O(N)`, where `N` is input size

> Binary search is more efficient than linear search, just the requirement is that the array is sorted.
{: .prompt-tip}

I'm going to walkthrough this tutorial with several example so that you can familiarise yourself with questions that look like they could be solved with a binary search algorithm. But first a bit of a background on **how** to implement binary search both through **iteration** and **recursion**. Lets dive in!

--- 
## What is binary search?

* Requires array to be sorted
* Each iteration the middle element in the array is compared to that target
* If the target is less than the middle element we discard the right half of the search space and repeat on the left half (or vice versa)
* We keep doing this until we either find the target element or find it doesn't exist


```python
def binary_search(data: list, target: int) -> bool:
  low = 0
  high = len(data) - 1

  while low <= high:
    mid = (low + high) // 2
    if target == data[mid]:
      # Found
      return True 
    elif target < data[mid]:
      # Discard right half of search space
      high = mid - 1
    else:
      # Discard left half of search space
      low = mid + 1
  return False
```

The above algorithm implies time complexity of `O(LogN)`. This is because at each iteration we are dividing the search space in half.

---
 
## Recursive binary search?

```python
def binary_search_recursive(data: list, target: int, low: int, high: int): -> bool:
  if low > high:
    return False

  else:
    mid = (low + high) // 2
    if target == data[mid]:
      return True 
    elif target < data[mid]:
      return binary_search_recursive(data, target, low, mid - 1)
    else:
      return binary_search_recursive(data, target, mid + 1, high)
```

In the recursive approach we've added the parameters, `low` and `high`. This is to help us define our base case `if low > high`. Then we proceed as normal, i.e. find the midpoint and test if thats our target, or recursively call the function with the left or right half of the search space.

--- 
## Find the closest number

Given a sorted array and a target. Goal is to find a number in the array that is closest to target.

### Solution - Iterative

Here we give the solution using an iterative method.

```python
def closest_number(nums, target):
    low, high = 0, len(nums) - 1

    while low <= high:
        mid = (low + high) // 2
        left_side = right_side = None
        # We need to evaluate the numbers either side
        if mid - 1 >= 0:
            left_side = abs(nums[mid - 1] - target)
        if mid + 1 < len(nums):
            right_side = abs(nums[mid + 1] - target)
        # If we find target return
        if nums[mid] == target:
            return nums[mid]
        elif left_side and right_side and left_side <= right_side:
            # walk left
            high = mid - 1
        elif left_side and right_side and left_side > right_side:
            # walk right
            low = mid + 1
        elif left_side:
            # Compare midpoit to leftside
            return nums[mid - 1] if left_side < nums[mid] else nums[mid]
        else:
            return nums[mid + 1] if right_side < nums[mid] else nums[mid]


if __name__ == "__main__":
    nums = [2, 5, 6, 7, 8, 8, 9]
    target = 4

    print(closest_number(nums, target))
```

---
## Find fixed number

Lets try solving a different problem, given an array of `n` **distinct integers sorted** in ascending order, write a function that returns a `fixed point` in the array. If there is no fixed point, return `None`.

A fixed point is defined as an index `i` in array `A` such that `A[i] = i`.

Lets take an example

```python
A = [0, 2, 5, 8, 17]
```

If we did a bit of adhoc investigation, we know the most simplest solution would be to formulate a linear solution. 

```python
def linearSolution(A):
    # O(N) solution
    for i in range(len(A)):
        if A[i] == i:
            return A[i]
    return None 
```

We traverse the array once and look for the fixed point. Assuming each iteration is computed in constant `O(1)` time, we arrive at a total time complexity of `O(N)`. However, if we look a bit closer at the question, we've been given two key propositions that allow us to take advantage of using a binary search algorithm with a time complexity of `O(LogN)`. These two propositions are:

* Array is *sorted*
* Array has *distinct* elements


If we look at `A` we see that it meets both criterias above (sorted and distinct). So at each stage we look at the midpoint and compare. Lets just walk through some pseudo code using a couple stages of the first steps, to give us the inituition of whats happening.

### Steps to take

* Assume `low = 0` and `high = 4` (i.e. `len(A) - 1`) 
* Pick the `mid = (0 + 4) // 2` -> `2`
* Now `A[2] = 5`, we have that `A[2] > 2` and `5 > 2`. As the array is sorted, it means everything to the right of this midpoint can be discarded because they will all be greater than there indexes!
* Divide the search space and consider the left half now i.e. `high = mid - 1` and `low = previous_low`
* Keep repeating until we get to our base cases where either `low > high` or `A[i] = i` (found value)

### Solution - Recursive

Here we give a recursive solution.

```python
def binary_search(A, low, high):

    # 1. Base case: Not found
    if low > high:
        return None

    mid = (low + high) // 2
    # 2. Base case: Found
    if A[mid] == mid:
        return A[mid]
    
    # Recursive step
    elif A[mid] > mid:
        # We can discard the right side as we know the list is sorted
        return binary_search(A, low, mid - 1)
    else:
        # equiv A[mid] < mid: So discard the left side
        return binary_search(A, mid + 1, high)

if __name__ == "__main__":
    A = [0, 2, 5, 8, 17]
    res = binary_search(A, 0, len(A) - 1)
    print(res) 
```

---
## Find Bitonic Peak

First let me define what a `bitonic peak` is. A `bitonically sorted` array is one that starts off with increasing terms and then concludes with decreasing terms. In any such sequence, there is a 'peak' element (index `K`) which is the largest element in the sequence. Sequences **do not** contain any duplicates.

Lets have a look at some of the sequence types

```python

>>> [1, 2, 3, 4, 1] # peaks at 4
>>> [-1, 4, 6, 2, 0] # peaks at 6

```

This one is far trickier than the others because of the *reverse* ordering that can happen somewhere between the first and last element of the array. The first thing you should be thinking with a binary search solution is, what am I comparing the midpoint to? Or how do I know if my peak is in the left or right half of my subspace **OR** have I reached it?

Let me just go through the linear solution so we have a rough idea of what the naive approach would be.

```python
def linear_solution(A):
    for i in range(1, len(A) - 1):
        # Peak is one that is greater than its element to left and right side
        if (A[i - 1] < A[i]) and (A[i] > A[i + 1]):
            # Found peak
            return A[i]

if __name__ == "__main__":
    A = [1, 2, 3, 4, 1]
    print(linear_solution(A))
```

A `base case` you might notice with this is that we need 3 or more elements otherwise we can't compare the `current_value` with its `left` and `right` neighbours. 

### Solution - Recursive

```python
def binary_search(A, low, high):
    if len(A) < 3:
      # Not enough elements to have a peak
      return None
    # Special case when K = (N - 1) i.e. reverts to a asc sorted list
    if A[low] < A[low + 1] and A[high] > A[high - 1]:
        return A[high]
    # Base cases is if they are all in order
    mid = (low + high) // 2
    if (A[mid - 1] < A[mid]) and (A[mid] > A[mid + 1]):
        # Found peak
        return A[mid]
    elif A[mid] < A[mid + 1]:
        # Peak is to the right
        return binary_search(A, mid + 1, high)
    else:
        # Peak is to the left
        return binary_search(A, low, mid - 1)

if __name__ == "__main__":
    A = [1, 2, 3, 4, 1]
    print(binary_search(A, 0, len(A) - 1))
```

A couple things to note here:

* The first basecase is when we don't have enough elements to create a peak (we require 3 or more elements).
* The second basecase is when the peak is right at the end of the sequence, this just defaults to a sorted ascending array. Hence peak is last element
* The third basecase is if we find the peak element. This conditional is the same as the linear solution, i.e. our current mid is greater than its left & right neighbours
* Then we decide whether to remove the left or right search space by deciding if we've reached the peak or not. 
* If we have an element to the right (i.e. `A[mid + 1]`) of our current `A[mid]` thats larger, we know the peak is in the right half of the search space
* Else the peak is to the left half of the search space.

--- 
## Find the first entry in the list with duplicates

Write a function that takes an array of sorted integers and a key and returns the index of the first occurences of that key from the array. A simple example would be

```python
A = [-14, -10, 2, 108, 108, 243, 285, 285, 285, 401]
target = 108

# Ans = 3
```

A quick fire linear solution would give us an `O(N)` time complexity.

```python
def linear_solution(A, target):
    for i in range(len(A)):
        if A[i] == target:
            return i 
```

Now we can improve on this but with some tweaks. The benefit we have is that its sorted, the trickier bit is that the array contains some duplicates so we need to handle these gracefully. The core logic is being aware that if you find the index value such that `A[i] == target`, it may not be the **first** index with that target. But because we know its sorted, we need to peak to our left! Just make sure you're aware that you account for index overflows (i.e. if your current mid is at `index = 0`).


```python
def binary_search(A, target):
    low = 0
    high = len(A) - 1

    while low <= high:
        mid = (low + high) // 2
        # IF we find target, we need to check its the first
        if A[mid] == target and (A[mid - 1] != target or mid == 0):
            return mid 
        elif A[mid] == target and A[mid - 1] == target:
            # To the left
            high = mid - 1
        elif target < A[mid]:
            high = mid - 1
        else:
            low = mid + 1


if __name__ == "__main__":

    A = [-14, -10, 2, 108, 108, 243, 285, 285, 285, 401] 
    target = 108

    print(binary_search(A, target))
```

