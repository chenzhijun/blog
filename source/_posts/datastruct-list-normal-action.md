---
title: 数据结构单链表的一些简单操作
copyright: true
date: 2019-09-09 20:42:20
tags: 链表
categories: 数据结构与算法
---


# 数据结构单链表的一些简单操作

Node的代码：

```java

public class Node<T> {
    private Node nextNode;
    private T v;

    public Node(T v) {
        this.v = v;
    }

    public Node() {
    }

    public Node(Node nextNode, T v) {
        this.nextNode = nextNode;
        this.v = v;
    }

    public Node getNextNode() {
        return nextNode;
    }

    public void setNextNode(Node nextNode) {
        this.nextNode = nextNode;
    }

    public T getV() {
        return v;
    }

    public void setV(T v) {
        this.v = v;
    }

    public void printAllNode() {//我这里是有一个headNode不做数据存储
        Node tmpN = this;
        while (tmpN.getNextNode() != null) {
            System.out.print(tmpN.getNextNode().getV());
            System.out.print((tmpN.getNextNode().getNextNode() == null ? "" : ","));
            tmpN = tmpN.getNextNode();
        }
    }
}

```
<!--more-->
## 反转链表

平常一条链表大多是`a->b->c->d`;链表反转后为:`a<-b<-c<-d`;

```java
package me.chenzhijun.gk_time_datastruct.chapter7;


public class ReverseNode {
    public static void main(String[] args) {
        Node head = new Node();
        Node tmp = head;
        for (int i = 1; i < 10; i++) {
            Node newNode = new Node();
            newNode.setV(i);
            tmp.setNextNode(newNode);
            tmp = tmp.getNextNode();
        }
        head.printAllNode();
        System.out.println();
        Node node = reverseNode(head);
        node.printAllNode();

    }

    //反转链表实际的代码
    private static Node reverseNode(Node pHead) {
        Node prev = null;
        Node pNode = pHead.getNextNode();
        while (pNode!= null) {
            Node next = pNode.getNextNode();
            pNode.setNextNode(prev);
            prev = pNode;
            pNode = next;
        }
        Node reverseNode = new Node();
        reverseNode.setNextNode(prev);
        return reverseNode;

    }

}

```

首先用`prev`，`pNode`作两个指针，prev 指的是前一个node，pNode 是指当前node。可以这样理解，pNode指向链表遍历时的当前node，prev指向pNode的链条中前一个节点。

我们先用一个临时变量`next`存储当前pNode的下一个节点位置，然后将当前pNode的位置反转，设置为prev的位置，然后prev移动到pNode的位置上，这样pNode的这个位置就存在了prev，所以pNode可以继续向下遍历节点，而我们之前刚好将pNode的下一个节点存储在了next临时变量中，所以有 pNode=next ；一直遍历到pNode为null，说明链表已经遍历完了。这时如果不喜欢头节点就可以直接返回prev，如果喜欢设置头节点，就可以new一个新节点，然后将nextNode设置为prev。这样一条新的链就完成了，链表就被反转了。

一开始初始化，普通链表：

![2019-09-09-20-04-33](/images/qiniu/2019-09-09-20-04-33.png)

之后我们开始做操作，先获取0-next节点位置，然后将1-nextNode指向prev所指向的位置，之后2-prev指向pNode所指向的位置，之后3-pNode指向next。

![2019-09-09-20-15-02](/images/qiniu/2019-09-09-20-15-02.png)

这样一直循环后，我们就可以实际得到一个反转如下的链表：

![2019-09-09-20-20-10](/images/qiniu/2019-09-09-20-20-10.png)

可以看到，其实如果直接返回prev就已经是一个反转链表了。

复杂度分析：

时间复杂度：O(n)，取决与链表长度；

空间复杂度：O(1)，只新建了pNode,next,prev指针变量,没有额外空间的；


## 两个排序链表的合并

链表的操作中还有一个是假设两个有序链表的合并，如果[1,3,5],[2,4,6,7]合并为[1,2,3,4,5,6,7]，代码如下：

```java
    private static Node mergeNode1AndNode2(Node head1, Node head2) {
        Node node = new Node();
        Node head = node;
        Node tmp = null;
        while (head1 != null && head2 != null) {
            if ((int) head1.getV() < (int) head2.getV()) {
                tmp = head1;
                head1 = head1.getNextNode();
            } else {
                tmp = head2;
                head2 = head2.getNextNode();
            }
            node.setNextNode(tmp);
            node = node.getNextNode();
        }
//        if (head1 != null) {
//            node.setNextNode(head1);
//        }
//        if (head2 != null) {
//            node.setNextNode(head2);
//        }
        node.setNextNode(head1 != null ? head1 : head2);
        return head;
    }
```

我们肯定都是要循环链表的，那么循环到一个链表为空的时候，将另一个链表的值加进来即可。

如`while (head1 != null && head2 != null)`这里就是判断链表1和链表2两者看是否为空。如果有一个为空的条件，就退出循环。

你可能看到`node.setNextNode(head1 != null ? head1 : head2);`这句话的作用其实就是说，如果head1不为空，那我们就将head1加入到设置为nextNode，反之就是head2。

合并两个列表的关键点在于，一个要判断两个链表是否为空，这是退出链表循环的条件。（当然如果是个循环链表，这里可能就得改一下了）。另外就是，当链表退出后，要记得将另一个非空链表的剩余值加入到新链中，保证不丢值。

## 删除链表倒数第 n 个结点

有一个链表，然后需要删除链表的倒数第n个位置的节点：

```java
    /**
     * 删除倒数第n个位置的node
     *
     * @param i
     */
    private static Node removeNode(Node node, int i) {
        if (i <= 0 || node.getNextNode() == null) {
            System.out.println("i为0或者node为空，不改动，i从1开始");
            return node;
        }

        Node tmp = node;
        while (i > 0) {
            tmp = tmp.getNextNode();
            if (tmp == null) {
                //长度不够
                System.out.println("node长度不够,不改动");
                return node;
            }
            i--;
        }
        Node n0 = node;
        Node n1 = node;
        //两个指针，一个先走i步，另一个再开始走
        while (tmp != null) {
            tmp = tmp.getNextNode();
            n0 = n1;
            n1 = n1.getNextNode();
        }

        n0.setNextNode(n1.getNextNode());
        return node;
    }
```


## 链表中间节点

计算链表中间节点位置：

```java

public Node computeNode(Node head) {
    Node quick = head.getNextNode();
    Node slow = head.getNextNode();
    while (quick != null && quick.getNextNode() != null) {
        slow = slow.getNextNode();
        quick = quick.getNextNode().getNextNode();
    }
    System.out.println();
    System.out.println(slow.getV());
}

```


## 是否为循环链表

判断一个链表是否为循环链表：

```java
    private static boolean isCycleList2(Node head) {
        Set<Node> set = new HashSet<>();
        while (head != null) {
            if (set.contains(head)) {
                return true;
            } else {
                set.add(head);
                head = head.getNextNode();
            }
        }
        return false;
    }

    private static boolean isCycleList(Node head) {
        Node n1 = head;
        Node n2 = head;

        while (n1 != null && n2.getNextNode() != null) {
            n1 = n1.getNextNode();
            n2 = n2.getNextNode().getNextNode();
            if (n1 == n2) {
                return true;
            }
        }
        return false;
    }
```


