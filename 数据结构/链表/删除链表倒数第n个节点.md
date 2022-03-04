# 删除链表倒数第n个节点

给你一个链表，删除链表的倒数第 n 个结点，并且返回链表的头结点。 

示例 1：

![](https://assets.leetcode.com/uploads/2020/10/03/remove_ex1.jpg)

```
输入：head = [1,2,3,4,5], n = 2
输出：[1,2,3,5]
```

示例 2：

```
输入：head = [1], n = 1
输出：[]
```

示例 3：

```
输入：head = [1,2], n = 1
输出：[1]
```


提示：

链表中结点的数目为 sz

```
1 <= sz <= 30
0 <= Node.val <= 100
1 <= n <= sz
```

进阶：你能尝试使用一趟扫描实现吗？

## 算法思路

1. 为了简化边界条件，方便操作，首先定义哨兵节点summary。
2. 定义两个指针front和behind，都指向哨兵节点。
3. 由于需要删除倒数第n个节点，因此，先让front移动n个节点。
4. front和behind指针共同开始移动，此时它们之间的距离恰好是n。
5. 当front到达尾部时（front=null），behind指向的节点恰好是到时第n个节点。
6. 将behind的下一个节点删除即可。

## 代码实现

```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode summary = new ListNode(0, head);
    ListNode front = summary;
    ListNode behind = summary;
    int i = 1;
    while ((front = front.next) != null)
        if (i++ > n) behind = behind.next;
    behind.next = behind.next.next;
    return summary.next;
}
```

## 算法分析

只扫描了一趟，时间复杂度O(n)。