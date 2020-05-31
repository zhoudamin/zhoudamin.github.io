---
title: Java之二叉树算法
categories:
  - 算法与数学
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2018-08-05 23:04:25
tags: 算法与数学
---

二叉树算法
<!--more-->

# 二叉查找树
## Validate Binary Search Tree 验证二叉搜索树
Given a binary tree, determine if it is a valid binary search tree (BST).

Assume a BST is defined as follows:

The left subtree of a node contains only nodes with keys less than the node's key.
The right subtree of a node contains only nodes with keys greater than the node's key.
Both the left and right subtrees must also be binary search trees.

思路，用递归，左<根<右 && 左根递归 && 右根递归
关键代码
```java
public boolean isValidBST(TreeNode root){
    return isValidBST(root,INT_MIN,INT_MAX);
}

bool isValidBST (TreeNode* root,int lower ,int upper ){
    if(root == nullptr) return true;

    return root.val>lower && root.val<upper 
           && isValidBST(root.left,lower,root.val)
           && isValidBST(root.right,root.val,upper);
}
```