---
title: 删除链表的倒数第N个节点
categories:
  - 算法与数学
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2020-04-26 22:46:26
tags: leetcode
---
<p>给定一个链表，删除链表的倒数第&nbsp;<em>n&nbsp;</em>个节点，并且返回链表的头结点。</p>

<p><strong>示例：</strong></p>


```js
给定一个链表: 1->2->3->4->5, 和 n = 2.

当删除了倒数第二个节点后，链表变为 1->2->3->5.
````

<p><strong>说明：</strong></p>

<p>给定的 <em>n</em>&nbsp;保证是有效的。</p>

<p><strong>进阶：</strong></p>

<p>你能尝试使用一趟扫描实现吗？</p>
<div><div>Related Topics</div><div><li>链表</li><li>双指针</li></div></div>

<!--more-->
双指针，new一个新的头结点，返回的时候要返回次节点
```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode pre = new ListNode(0);
    pre.next = head;
    ListNode first = pre ;
    ListNode last = pre ;
    for (int i=0;i<n;i++){
        last = last.next;
    }
    
    while (last.next!=null){
        first = first.next;
        last = last.next;
    }
    first.next=first.next.next;
    return pre.next;
}
```