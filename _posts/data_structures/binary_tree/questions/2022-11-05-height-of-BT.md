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

def height(root):
    if root is None:
        # Returning -1 will cancel out with leaf node (+1) to give 0
        return -1 
    return max(height(root.left), height(root.right)) + 1

if __name__ == "__main__":
    # Initialise a tree
    tree = Node(1)
    tree.left = Node(2)
    tree.right = Node(3)
    tree.left.left = Node(4)
    tree.left.right = Node(5)

    print(height(tree))
```