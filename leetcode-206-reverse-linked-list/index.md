# LeetCode 206 Reverse Linked List（反转链表)


------

反转一个单链表。

**示例:**

```
输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL
```

**进阶:**
你可以迭代或递归地反转链表。你能否用两种方法解决这道题？  

<!-- more -->

## 解法一：迭代法 

利用一个哑节点，来依次反转节点

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode reverseList(ListNode head) {
        if(head == null || head.next == null) {
            return head;
        }
        ListNode dummyNode = null;
        while(head != null) {
            ListNode next = head.next;    //暂存下一个待遍历节点
            head.next = dummyNode;		 //当前到的当前节点指向新的链表头
            dummyNode = head;			//更新 新的链表头
            head = next;				//迭代下一个节点
        }
        return dummyNode;
    }
}
```



## 解法二：递归 

递归法本质是一种不断分解问题规模的思想，子问题除了规模更小之外，其他和父问题都一样。**递归主要是找到递归公式和终止条件**。  

>1. 现在需要把A->B->C->D进行反转， 
>2. 可以先假设B->C->D已经反转好，已经成为了D->C->B,那么接下来要做的事情就是将D->C->B看成一个整体，让这个整体的next指向A，所以问题转化了反转B->C->D。  
>3. 那么，可以先假设C->D已经反转好，已经成为了D->C, 
>4. 那么接下来要做的事情就是将D->C看成一个整体，让这个整体的next指向B，所以问题转化了反转C->D。 
>5. 那么,可以先假设D(其实是D->NULL)已经反转好，已经成为了D(其实是head->D),那么接下来要做的事情就是将D(其实是head->D)看成一个整体，让这个整体的next指向C，所以问题转化了反转D。    

```java
    public ListNode reverseList(ListNode head) {
        if (head == null || head.next == null) {  //链表为空直接返回，而head.next为空是递归终止条件
            return head;
        }
        //一直循环到链尾，返回的newHead即为旧链表尾部，新的链表头
        ListNode newHead = reverseList(head.next); 
        head.next.next = head;   //翻转链表的指向，形成一个环，例如：由4->5变成一个环，4指向5，5也指向4
        head.next = null;         //赋值NULL，将上一步形成的链表环切掉，防止链表错乱
        return newHead;            //新链表头永远指向的是原链表的链尾
    }
```

**head.next.next = head** 表示head的下一个节点翻转指向自己head，形成一个环，**head.next = null** 表示切断之前的链接方向，避免环路。










