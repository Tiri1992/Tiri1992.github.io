---
layout: post
title: "Binary Search Tree - Validate BST"
mermaid: true
categories: [data_structures,binary_search_tree]
tags: [python, data_structures, binary_search_tree]
---

# Question

Validate if a `binary tree` satisifies the `binary search tree` property. 

### Example: Valid BST

```mermaid
flowchart TD
    8((8)) --> 3((3))
    8((8)) --> 10((10))
    3((3)) --> 1((1))
    3((3)) --> 6((6)) 
```

### Example: Invalid BST

```mermaid
flowchart TD
    7((7)) --> 3((3))
    7((7)) --> 10((10))
    3((3)) --> 1((1))
    10((10)) --> 5((5)) 
    10((10)) --> 14((14)) 
```

This is not valid because `5 < 7` and `5` belongs to the left subtree of `7`.


```python
class Node:

    def __init__(self, data):
        self.data = data 
        self.left = None 
        self.right = None  

def bst_validate(node, lower, upper):
    if not node:
        # Empty node is ordered correctly
        return True 

    val = node.data 

    if val <= lower or val >= upper:
        # Current node should be greater than all the values in the left subtree but also less than all the values of the right subtree
        return False


    if not bst_validate(node.right, node.data, upper):
        return False 

    if not bst_validate(node.left, lower, node.data):
        return False 
    
    return True 
```
