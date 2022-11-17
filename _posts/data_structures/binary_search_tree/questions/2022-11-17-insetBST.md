---
layout: post
title: "Binary Search Tree - Insert into BST"
mermaid: true
categories: [data_structures,binary_search_tree]
tags: [python, data_structures, binary_search_tree, recursion]
---


Write a function that takes in a `root`, `value` as arguments and returns the `root` with the new node inserted into the correct position of the `binary search tree`.

```python
class Node:

    def __init__(self, data):
        self.data = data 
        self.left = None 
        self.right = None 

def insert_helper(root: Node, new_value: int):
    if root.data < new_value:
        # Go right
        if root.right:
            # Recursively call the right subtree
            insert_helper(root.right, new_value)
        else:
            # Node to the right is empty, hence we can insert
            root.right = Node(new_value)
    else:
        # Go left
        if root.left:
            insert_helper(root.left, new_value)
        else:
            # Node to the left is empty, hence we can insert
            root.left = Node(new_value)
```