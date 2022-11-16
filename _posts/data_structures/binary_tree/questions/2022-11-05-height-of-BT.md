---
layout: post
title: "Binary Trees - Height of tree"
mermaid: true
categories: [data_structures,binary_tree,questions]
tags: [python, data_structures, binary_tree, recursion]
---

# Question

Given a binary tree, write a recursive algorithm to find the height of the tree.

```python
class Node:

    def __init__(self, value) -> None:
        self.value = value 
        self.left = None
        self.right = None

def height(root: Node):
    if root is None:
        # Returning -1 will cancel out with leaf node (+1) to give 0
        return -1 
    return max(height(root.left), height(root.right)) + 1
```