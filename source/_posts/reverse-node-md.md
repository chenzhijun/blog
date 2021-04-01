---
title: 翻转链表
date: 2017-04-06 12:29:41
tags:
	- 算法
categories: 算法
---

## 翻转链表

> 比如链表1-2-3-4-5 翻转为 链表5-4-3-2-1 格式

构建基础链表，链表的数据结构比较简单，就是一个数据项，一个下一个节点。我在这里其实遇到过一个理解问题，总是把node当成了链表。其实一个node就是一个小块。不是那一条链，只不过它的中间有下一个node，这样一层一层组成了一个链表。所以记得一个链表中，node指的是一个节点，而不是一个链。能这样理解，那么翻转链表也不是什么问题了。
<!-- more-->

```java
package node;

/**
 * Created by alvin on 4/2/17.
 */
public class Node {
    private int data;

    private Node nextNode;


    public int getData() {
        return data;
    }

    public void setData(int data) {
        this.data = data;
    }

    public Node getNextNode() {
        return nextNode;
    }

    public void setNextNode(Node nextNode) {
        this.nextNode = nextNode;
    }


    public Node(int data) {
        this.data = data;
    }

    public static Node initNode(int num) {
        Node node = new Node(0);
        Node temp = node;
        for (int i = 1; i < num; i++) {
            Node nextNode = new Node(i);
            temp.setNextNode(nextNode);
            temp = nextNode;
        }

        return node;
    }


    public static void main(String[] args) {
        Node node = initNode(5);
        out(node,"init:");
    }

    public static void out(Node node,String prompt){
        System.out.print(prompt);
        while (null != node) {
            System.out.print(node.getData()+",");
            node = node.getNextNode();
        }
    }
}
```
之后我们写测试代码：
``` java
package node;

/**
 * Created by alvin on 4/2/17.
 */
public class MainClass {

    public static void main(String[] args) {
        Node node = Node.initNode(4);
        Node.out(node,"遍历init:");

        System.out.println("\n==========");
        Node reversNode = reverseNode(node);
        Node.out(reversNode,"遍历翻转:");


        System.out.println("\n==========");

        Node node1 = Node.initNode(4);
        Node.out(node1,"递归init:");
        Node reversNode2 = reverseNode1(node1);
        System.out.println("\n==========");
        Node.out(reversNode2,"递归翻转:");

    }

}

```

### 遍历反转
![遍历链表排序](/images/reverse-node.jpeg)
像图片中所示的一样，第一次在上面，我们定义pre，cur，next三个指示量，链表的好处在于，只要将链表中的节点指向另一个地方就完成了对这个节点链表的操作。每一次我们先将第一个节点保存，将第二个节点的nextNode 指向第一个节点。依次类推，得到的最终node链表就是我们翻转后的了。
```java
    //遍历翻转
    private static Node reverseNode(Node node) {
        Node pre = node;
        Node cur = node.getNextNode();
        Node next = null;
        pre.setNextNode(null);
        while (null != cur) {
            next = cur.getNextNode();
            cur.setNextNode(pre);
            pre = cur;
            cur = next;
        }
        node = pre;

        return node;
    }
```

### 递归翻转
***递归的程序必须有一个终止递归的条件。*** 递归的思路是递归链表到最后那个节点，然后一层一层的将nextNode 至为上一个node，而上一个node设置为null，这样可以防止在递归回来时，第一个节点和第二个节点造成节点死循环。

```java 
//递归翻转
    private static Node reverseNode1(Node node) {
        if (null == node || null == node.getNextNode()) {
            return node;
        }

        Node reHead = reverseNode1(node.getNextNode());

        node.getNextNode().setNextNode(node);

        node.setNextNode(null);

        return reHead;
    }
```