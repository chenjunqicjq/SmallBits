# 题目

输入一个链表，输出该链表中倒数第k个结点。 

# 思路

这题比较简单，直接用两个指针模拟出一个长为k的尺子，先让一个指针走k个节点，然后再两个指针联动遍历，当前面一个指针跑完时，后一个指针正好指向倒数第k个节点

**代码如下**

```java
/*
public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}*/
public class Solution {
    public ListNode FindKthToTail(ListNode head,int k) {
         ListNode curr,last;
        curr=last=head;
        for(int i=0;i<k;i++){
            if(last!=null){
                last=last.next;
            }else {
                return null;
            }
        }
        while(last!=null){
            curr=curr.next;
            last=last.next;
        }
        return curr;
    }
}


```

