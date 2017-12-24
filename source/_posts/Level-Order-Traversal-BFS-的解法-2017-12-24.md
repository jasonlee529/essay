---
title: Level-Order Traversal (BFS)的解法
date: 2017-12-24 21:07:25
categories:
- 算法
tags:
- 算法
---
## 问题：
给定一个数组，如下所示。目的：按层级输出每个节点的值。
![你想输入的替代文字](/img/1460000919-b3d7973e9e-BinaryTree.png)

输出：
> 4 2 6 1 3 5 7
## 解决思路：
对于level-order-travels类的问题，简单适用的算法是：breadth-first-search (BFS)。
广度优先算法（Breadth-First-Search），简称BFS，是一种图形搜索演算法。简单的说，BFS是从根节点开始，沿着树的宽度遍历树的节点，如果发现目标，则演算终止。

### 算法分析:
BFS是一种盲目搜寻法，目的是系统地展开并检查图中的所有节点，以找寻结果。换句话说，它并不考虑结果的可能位置，彻底地搜索整张图，直到找到结果为止。

### 伪代码：
``` Java
levelOrder(BinaryTree t) {
    if(t is not empty) {
        // enqueue current root
        queue.enqueue(t)

        // while there are nodes to process
        while( queue is not empty ) {
            // dequeue next node
            BinaryTree tree = queue.dequeue();

            process tree's root;

            // enqueue child elements from next level in order
            if( tree has non-empty left subtree ) {
                queue.enqueue( left subtree of t )
            }
            if( tree has non-empty right subtree ) {
                queue.enqueue( right subtree of t )
            }
        }
    }
}
```

## 实现
``` Java
void levelOrder(Node root) {
       Queue<Node> queue = new LinkedList<Node>();
       if (root != null) {
           // enqueue current root
           queue.offer(root);

           // while there are nodes to process
           while (queue != null && queue.size() > 0) {
               // dequeue next node
               Node tree = queue.poll();

               System.out.print(tree.data + " ");

               // enqueue child elements from next level in order
               if (tree.left != null){
                   queue.offer(tree.left);
               }
               if (tree.right != null){
                   queue.offer(tree.right);
               }
           }
       }
   }
```
