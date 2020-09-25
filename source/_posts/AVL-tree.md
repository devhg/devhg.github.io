---
title: AVL平衡二叉树学习
date: 2019-12-14 21:16:49
tags:
  - 数据结构
categories:
  - 数据结构
	- 树
---

**平衡二叉树(AVL)，**是一种二叉排序树，其中每个结点的左子树和右子树的高度差至多等于1。它是一种高度平衡的二叉排序树。高度平衡？意思是说，要么它是一棵空树，要么它的左子树和右子树都是平衡二叉树，且左子树和右子树的深度之差的绝对值不超过1。

<!--more-->

### 平衡二叉树特点

**平衡因子BF：**
指的是二叉树左子树深度减去右子树深度。
平衡因子只能是-1,0,1。只要二叉树有一个节点的平衡因子绝对值大于1，则该树就不平衡了。

一棵AVL树有如下必要条件：

1.  条件一：它必须是二叉排序树。
2. 条件二：每个节点的左子树和右子树的高度差至多为1。也就是说平衡因子绝对值不超过1



### 实用性分析

AVL平衡二叉树的查找、插入、删除操作在平均和最坏的情况下都是O（logn），这得益于它时刻维护着二叉树的平衡。如果我们需要查找的集合本身没有顺序，在频繁查找的同时也经常的插入和删除，AVL树是不错的选择。不平衡的二叉查找树在查找时的效率是很低的，因此，AVL如何维护二叉树的平衡是我们的学习重点。



<hr>

### 四种旋转方式

![旋转方式](https://tvax3.sinaimg.cn/large/006nIlf0ly1g9wkw68fdvj319s0oegph.jpg)



#### LL型旋转

由于在A的左孩子(L)的左子树(L)上插入新结点，使原来平衡二叉树变得不平衡，此时A的平衡因子由1增至2。下面图1是LL型的最简单形式。显然，按照大小关系，结点B应作为新的根结点，其余两个节点分别作为左右孩子节点才能平衡，A结点就好像是绕结点B顺时针旋转一样。

LL型调整的一般形式如下图所示，表示在A的左孩子B的左子树BL(不一定为空)中插入结点(图中阴影部分所示)而导致不平衡( h 表示子树的深度)。这种情况调整如下：①将A的左孩子B提升为新的根结点；②将原来的根结点A降为B的右孩子；③各子树按大小关系连接(BL和AR不变，BR调整为A的左子树)。

![](https://tva2.sinaimg.cn/large/006nIlf0ly1g9wl6dpfngj30ms0a3mxe.jpg)

#### RR型旋转

和LL型旋转类似

表示在A的右孩子B的右子树BR(不一定为空)中插入结点(图中阴影部分所示)而导致不平衡( h 表示子树的深度)。这种情况调整如下：①将A的右孩子B提升为新的根结点；②将原来的根结点A降为B的左孩子；③各子树按大小关系连接(AL和BR不变，BL调整为A的右子树)。

![](https://tva1.sinaimg.cn/large/006nIlf0ly1g9wl6izdyqj30mi0agq36.jpg)

#### LR型旋转

表示在A的左孩子B的右子树(根结点为C，不一定为空)中插入结点(图中两个阴影部分之一)而导致不平衡( h 表示子树的深度)。这种情况调整如下：①将B的右孩子C提升为新的根结点；②将原来的根结点A降为C的右孩子，B变为C的左孩子；③各子树按大小关系连接(BL和AR不变，CL和CR分别调整为B的右子树和A的左子树)。



![](https://tva1.sinaimg.cn/large/006nIlf0ly1g9wlbzq4lhj30nv0ant94.jpg)



**等价于 先一次RR旋转再一次LL旋转。先对B右孩子C及C的右孩子进行RR旋转，再对A,B,C进行LL旋转。**

![](https://tva1.sinaimg.cn/large/006nIlf0ly1g9wlpkpjw3j30m906njs4.jpg)

#### RL型旋转

表示在A的右孩子B的左子树(根结点为C，不一定为空)中插入结点(图中两个阴影部分之一)而导致不平衡( h 表示子树的深度)。这种情况调整如下：①将B的左孩子C提升为新的根结点；②将原来的根结点A降为C的左孩子，B变为C的右孩子；③各子树按大小关系连接(AL和BR不变，CL和CR分别调整为A的右子树和B的左子树)。



![](https://tva4.sinaimg.cn/large/006nIlf0ly1g9wlc3skaej30nc0a23yy.jpg)



**等价于 先一次LL旋转再一次RR旋转。先对B左孩子C及C的左孩子进行LL旋转，再对A,B,C进行RR旋转。**

![](https://tvax2.sinaimg.cn/large/006nIlf0ly1g9wlpmyadhj30m706nmxx.jpg)

<hr>

### C/C++代码实现

```c++
// E [3374] - 数据结构实验之查找二：平衡二叉树
#include <cstdio>
#include <cstdlib>
#include <cstring>
#include <iostream>
using namespace std;
struct node {
    int data;  //记录关键字数值
    node *l, *r;
    int height;  //当前节点的高度
};
int height(node *p)  //求树的深度
{
    if (p == NULL) return -1;
    return p->height;
}
node *LL(node *p)  //对LL型直接在不平衡结点进行左旋转
{
    node *q = p->l;  // q是p的左子树
    p->l = q->r;  // q的右子树值一定小于p->data  所以p->l连上q->r
    q->r = p;     // q->r连上p 画图理解
    p->height = max(height(p->l), height(p->r)) + 1;  // 要更新结点的高度值
    q->height = max(height(q->l), p->height) + 1;
    return q;
}
node *RR(node *p)  //对RR型直接在不平衡结点进行右旋转
{
    node *q = p->r;  // 连接原理与LL相同
    p->r = q->l;
    q->l = p;
    p->height = max(height(p->l), height(p->r)) + 1;
    q->height = max(height(q->r), p->height) + 1;
    return q;
}
node *LR(node *p) {
    p->l = RR(p->l);  //在不平衡结点p的左孩子以及左孩子的孩子进行RR旋转
    return LL(p);  //在不平衡结点p处进行LL并返回新的根
}
node *RL(node *p) {
    p->r = LL(p->r);  //在不平衡结点p的右孩子以及右孩子的孩子进行LL旋转
    return RR(p);  //在不平衡结点p处进行RR并返回新的根
}
void insert(node *&p, int k) {
    //待插入的值赋给新开辟的结点
    if (p == NULL) {
        p = new node;
        p->data = k;
        p->height = 0;
        p->l = p->r = NULL;
    } else if (k < p->data) {
        // 若待插入的值小于p的关键字数值，则插入到左子树中
        insert(p->l, k);
        // 若该树出现不平衡 旋转
        if (height(p->l) - height(p->r) == 2) {
            if (k < p->l->data)  // 若待插入的值插到了左子树的左子树上则单旋转
                p = LL(p);
            else  //若待插入的值插到了左子树的右子树上则单旋转
                p = LR(p);
        }
    } else if (k > p->data) {
        // 若待插入的值小于p的关键字数值，则插入到右子树中
        insert(p->r, k);
        if (height(p->r) - height(p->l) == 2) {
            // 若待插入的值插到了右子树的右子树上则单旋转
            if (k > p->r->data)
                p = RR(p);
            else  // 若待插入的值插到了右子树的左子树上则单旋转
                p = RL(p);
        }
    }
    p->height = max(height(p->l), height(p->r)) + 1;  // 修改高度
}
int main() {
    int n, k;
    while (~scanf("%d", &n)) {
        node *head = NULL;
        for (int i = 0; i < n; i++) {
            scanf("%d", &k);
            insert(head, k);
        }
        printf("%d\n", head->data);
    }
    return 0;
}

```



### 推荐学习资料

b站视频连接：
https://www.bilibili.com/video/av37955102?from=search&seid=14638889623357631324

https://www.bilibili.com/video/av37955178?from=search&seid=14638889623357631324

https://www.bilibili.com/video/av37955231?from=search&seid=14638889623357631324



### 参考资料

- [平衡二叉树](https://blog.csdn.net/isunbin/article/details/81707606)
- []()