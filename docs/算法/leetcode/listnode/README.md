# 链表



## 插入节点

插入节点有尾插法与头插法。

### 尾插法

```java
public ListNode insert(ListNode head, List<Integer> list) {
    ListNode p = head;
    for (int val : list) {
        ListNode node = new ListNode(val);
        p.next = node;
        p = node;
    }
    return head;
}
```

### 头插法

```java
public ListNode insert(ListNode head, List<Integer> list) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    for (int val : list) {
        ListNode node = new ListNode(val);
        node.next = dummy.next;
        dummy.next = node;
    }
    return dummy.next;
}
```





## 快慢指针

快慢指针可以用来寻找链表的中点，具体操作为：定义两个指针 `fast, slow`，从头结点开始每次循环 `fast` 走两步、`slow` 走一步，直到 `fast` 或 `fast` 的下一个节点为空。

最终 `slow` 指针会指向后半部分链表的前一个位置（若节点数为奇数则指向链表的中间节点）。

```java
public ListNode findMid(ListNode head) {
    ListNode fast = head, slow = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}
```

