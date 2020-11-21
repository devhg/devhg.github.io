---
title: 单链表排序的三种方式
permalink: linked-list-sort
toc: true
date: 2020-11-21 19:16:24
tags:
  - 链表
categories:
  - 数据结构
---

<!--more-->

**题目描述**：在O(nlogn)时间内对链表进行排序。 

**进阶：**

- 你可以在 `O(nlogn)` 时间复杂度和常数级空间复杂度下，对链表进行排序吗？

来源：[LeetCode#148](https://leetcode-cn.com/problems/sort-list/)

能达到此事件复杂度的排序算法如下

|          | 时间复杂度            | 空间复杂度         | 稳定性 |
| -------- | --------------------- | ------------------ | ------ |
| 归并排序 | O(nlogn)              | O(n)               | 稳定   |
| 快速排序 | O(nlogn) / 最坏O(n^2) | O(logn) / 最坏O(n) | 不稳定 |
| 希尔排序 | O(nlogn)              | O(1)               | 不稳定 |
| 堆排序   | O(nlogn)              | O(1)               | 不稳定 |

待排序的数据用链表保存，所以堆排序和希尔排序不太适合。因此这里用快速排序和归并排序实现。

快速排序速度内存情况比归并排序差很多。

### 快速排序

思路：快慢指针交换小于哨兵的节点，快指针用于探路，慢指针用于交换数据

```go
// 时间复杂度O(nlogn) 最坏 O(n^2)   空间复杂度 O(logn) 最坏(n)
func sortList(head *ListNode) *ListNode {
    quickSort(head, nil)
    return head
}

func quickSort(head, end *ListNode) {
	if head != end {
		node := sortPart(head, end)
		quickSort(head, node)
		quickSort(node.Next, end)
	}
}

func sortPart(head, end *ListNode) *ListNode {
	l, r := head, head.Next
	for r != end {
		if r.Val < head.Val {
			l = l.Next
			temp := l.Val
			l.Val = r.Val
			r.Val = temp
		}
		r = r.Next
	}

	if l != head {
		temp := l.Val
		l.Val = head.Val
		head.Val = temp
	}
	return l
}
```



### 归并排序

自顶向下归并排序
对链表自顶向下归并排序的过程如下。

1. 找到链表的中点，以中点为分界，将链表拆分成两个子链表。寻找链表的中点可以使用快慢指针的做法，快指针每次移动 2步，慢指针每次移动 1步，当快指针到达链表末尾时，慢指针指向的链表节点即为链表的中点。

2. 以中间节点为界，拆分链表。然后递归地拆左右两个链表。拆到只剩一个元素的时候合并

3. 链表合并之后，得到完整的排序后的链表。

上述过程可以通过递归实现。递归的终止条件是链表的节点个数小于或等于 1，即当链表为空或者链表只包含 1 个节点时，不需要对链表进行拆分和排序。

```go
// 时间复杂度O(nlogn)   空间复杂度 O(logn)
func sortList(head *ListNode) *ListNode {
    return sort(head)
}

func sort(head *ListNode) *ListNode {
	if head == nil || head.Next == nil {
		return head
	}

	slow, fast := head, head
	// 快慢指针  注意一下 两个的情况
	for fast.Next != nil && fast.Next.Next != nil {
		slow = slow.Next
		fast = fast.Next.Next
	}
	right := slow.Next
	slow.Next = nil
	return merge(sort(head), sort(right))
}

func merge(head1, head2 *ListNode) *ListNode {
	newHead := &ListNode{}
	cur := newHead

	for head1 != nil && head2 != nil {
		if head1.Val < head2.Val {
			cur.Next = head1
			head1 = head1.Next
		} else {
			cur.Next = head2
			head2 = head2.Next
		}
		cur = cur.Next
	}
	if head1 != nil {
		cur.Next = head1
	}
	if head2 != nil {
		cur.Next = head2
	}
	return newHead.Next
}

```



### 归并排序升级版

使用自底向上的方法实现归并排序，则可以达到 O(1) 的空间复杂度。首先求得链表的长度 length，然后将链表拆分成子链表进行合并。

具体做法如下：

1. 用 `subLength` 表示每次需要排序的子链表的长度，初始时`subLength=1`。

2. 每次将链表拆分成若干个长度为`subLength` 的子链表（最后一个子链表的长度可以小于`subLength`），按照每两个子链表一组进行合并，合并后即可得到若干个长度为 `subLength×2` 的有序子链表（最后一个子链表的长度可以小于` subLength×2`）。
3. 将 `subLength` 的值加倍，重复第 2 步，对更长的有序子链表进行合并操作，直到有序子链表的长度大于或等于 `length`，整个链表排序完毕。

如何保证每次合并之后得到的子链表都是有序的呢？可以通过数学归纳法证明。

1. 初始时` subLength=1`，每个长度为 1的子链表都是有序的。

2. 如果每个长度为`subLength` 的子链表已经有序，合并两个长度为`subLength` 的有序子链表，得到长度为`subLength×2` 的子链表，一定也是有序的。

3. 当最后一个子链表的长度小于`subLength` 时，该子链表也是有序的，合并两个有序子链表之后得到的子链表一定也是有序的。

因此可以保证最后得到的链表是有序的。

时间复杂度O(nlogn)   空间复杂度 O(1)

```go
func sortList(head *ListNode) *ListNode {
    if head == nil {
		return head
	}

	length := 0
	for node := head; node != nil; node = node.Next {
		length++
	}

	newHead := &ListNode{Next: head}
	for subLen := 1; subLen < length; subLen *= 2 {
		prev, cur := newHead, newHead.Next
		for cur != nil {
			head1 := cur
			//subLen < length  cur 不可能为nil
			for i := 1; i < subLen && cur.Next != nil; i++ {
				cur = cur.Next
			}

			head2 := cur.Next
			cur.Next = nil
			cur = head2
			//注意这里cur可能为nil
			for i := 1; i < subLen && cur != nil && cur.Next != nil; i++ {
				cur = cur.Next
			}

			// 合并之前，如果head2尾后还有，这里必须断开
			var next *ListNode
			if cur != nil {
				next = cur.Next
				cur.Next = nil
			}

			prev.Next = merge(head1, head2)
			for prev.Next != nil {
				prev = prev.Next
			}
			cur = next
		}
	}
	return newHead.Next
}

func merge(head1, head2 *ListNode) *ListNode {
	newHead := &ListNode{}
	cur := newHead

	for head1 != nil && head2 != nil {
		if head1.Val < head2.Val {
			cur.Next = head1
			head1 = head1.Next
		} else {
			cur.Next = head2
			head2 = head2.Next
		}
		cur = cur.Next
	}
	if head1 != nil {
		cur.Next = head1
	}

	if head2 != nil {
		cur.Next = head2
	}
	return newHead.Next
}
```



### 总结

归并排序，更适合链表的排序，仅此而已，又水一篇

