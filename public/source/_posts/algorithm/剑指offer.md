---
title: 算法-剑指Offer
categories: algorithm
tags:
  - java
  - algorithm
keywords: 'java,algorithm'
abbrlink: 1ef368f8
date: 2022-01-03 03:40:00
---
## 09. 用两个栈实现队列

题目描述：

> 用两个栈实现一个队列。队列的声明如下，请实现它的两个函数 appendTail 和 deleteHead ，分别完成在队列尾部插入整数和在队列头部删除整数的功能。(若队列中没有元素，deleteHead 操作返回 -1 )



首先明确栈和队列的数据结构的特点：栈（先进后出），队列（先进先出）

使用栈来实现队列，那么只要能找到队头和队尾就可以了。一个栈，很容易就得到了队尾，不断的压栈就是在不断的向队列的尾部插入数据，问题是如何得到队头？我们把栈中的元素挨个出栈再压入到另外一个栈中就得到了入队的顺序，也就得到了队头。



```java
/**
 * 两个栈实现一个队列，一个负责入队，一个负责出队
 */
public class CQueue {

    /**
     * 入队用
     */
    private Stack<Integer> in = new Stack<>();
    /**
     * 出队用
     */
    private Stack<Integer> out = new Stack<>();

    public CQueue() {

    }

    public void appendTail(int value) {
        in.push(value);
    }

    public int deleteHead() {
        if (out.isEmpty()) {
            while (!in.isEmpty()) {
                Integer pop = in.pop();
                out.push(pop);
            }
        }
        if (out.isEmpty()) {
            return -1;
        }
        return out.pop();
    }
}
```





## 30. 包含min函数的栈

题目描述：

> 定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的 min 函数在该栈中，调用 min、push 及 pop 的时间复杂度都是 O(1)。



只需要把栈包装以下，增加一个求最小值的方法即可。

```java
/**
 * 包含min函数的栈
 */
public class MinStack {

    private final Stack<Integer> stack;
    private Integer min;

    public MinStack() {
        stack = new Stack<>();
    }

    public void push(int x) {
        if (min != null) {
            if (min > x) {
                min = x;
            }
        } else {
            min = x;
        }
        stack.push(x);
    }

    public void pop() {
        Integer pop = stack.pop();
        if (pop <= min) {
            // 最小值出去了，需要重新计算最小值
            Integer[] items = stack.toArray(new Integer[0]);
            if (stack.isEmpty()) {
                min = null;
            } else {
                min = Arrays.stream(items).min(Comparator.naturalOrder()).get();
            }
        }
    }

    public int top() {
        return stack.peek();
    }

    public int min() {
        return min;
    }
}
```



##  06. 从尾到头打印链表

题目描述：

> 输入一个链表的头节点，从尾到头反过来返回每个节点的值（用数组返回）。输入：head = [1,3,2]，输出：[2,3,1]



最容易想到的是，把链表遍历下，然后倒序输出。这样的话需要两个集合遍历两次。倒序输出，正好是栈的特点。所以使用栈更方便。

```java
public int[] reversePrint(ListNode head) {
        Stack<ListNode> stack = new Stack<ListNode>();
        ListNode temp = head;
        while (temp != null) {
            stack.push(temp);
            temp = temp.next;
        }
        int size = stack.size();
        int[] print = new int[size];
        for (int i = 0; i < size; i++) {
            print[i] = stack.pop().val;
        }
        return print;
    }

```



##  24. 反转链表

题目描述：

> 定义一个函数，输入一个链表的头节点，反转该链表并输出反转后链表的头节点。



示例:

输入: 1->2->3->4->5->NULL

输出: 5->4->3->2->1->NULL



反序输出，这正好符合栈的特点 入栈出栈后便是反序的结果，所以可以考虑使用栈来实现

```java
/**
 * 单链表反转,借助栈结构
 */
public class Solution {
    public static void main(String[] args) {
        ListNode l1 = new ListNode(1);
        ListNode l2 = new ListNode(2);
        ListNode l3 = new ListNode(3);
        l1.setNext(l2);
        l2.setNext(l3);
        ListNode listNode = reverseList(l1);
        System.out.println(listNode);
    }

    public static ListNode reverseList(ListNode head) {
        Stack<Integer> stack = new Stack<>();
        if (head == null) {
            return null;
        }
        // 入栈，出栈后便是反转后的链表
        ListNode tmp = head;
        stack.push(head.val);
        while (tmp.next != null) {
            tmp = tmp.next;
            stack.push(tmp.val);
        }
        // 重新构造链表
        ListNode headNode = new ListNode(stack.pop());
        ListNode next = headNode;
        while (!stack.isEmpty()) {
            next.next = new ListNode(stack.pop());
            next = next.next;
        }
        return headNode;
    }

    static class ListNode implements Serializable {
        int val;
        ListNode next;

        ListNode(int x) {
            val = x;
        }

        public void setNext(ListNode next) {
            this.next = next;
        }

        @Override
        public String toString() {
            return "ListNode{" +
                    "val=" + val +
                    ", next=" + next +
                    '}';
        }
    }
}
```



可以在遍历的链表的时候，就地反转链表，只需要记录下前后的节点然后改变引用即可。

![](https://blog.lichenghao.cn/upload/2022/07/06104853.png)





```java
/**
 * 单链表反转
 */
public class Solution {
    public static void main(String[] args) {
        ListNode l1 = new ListNode(1);
        ListNode l2 = new ListNode(2);
        ListNode l3 = new ListNode(3);
        l1.setNext(l2);
        l2.setNext(l3);

        ListNode listNode = reverseList(l1);
        System.out.println(listNode);
    }

    public static ListNode reverseList(ListNode head) {
        ListNode curr = head;
        ListNode pre = null;
        while (curr != null) {
            ListNode next = curr.next;
            curr.next = pre;
            pre = curr;
            curr = next;
        }
        return pre;
    }

    static class ListNode implements Serializable {
        int val;
        ListNode next;

        ListNode(int x) {
            val = x;
        }

        public void setNext(ListNode next) {
            this.next = next;
        }

        @Override
        public String toString() {
            return "ListNode{" +
                    "val=" + val +
                    ", next=" + next +
                    '}';
        }
    }
}
```



## 35. 复杂链表的复制

题目描述：

> 请实现 copyRandomList 函数，复制一个复杂链表。在复杂链表中，每个节点除了有一个 next 指针指向下一个节点，还有一个 random 指针指向链表中的任意节点或者 null。

示例：

输入：head = [[7,null],[13,0],[11,4],[10,2],[1,0]]

输出：[[7,null],[13,0],[11,4],[10,2],[1,0]]

![](https://blog.lichenghao.cn/upload/2022/07/1649227088.png)

刚开始题都没读懂，很尴尬😓。简单来说就是把链表的每个节点都复制一遍。如果没有 random 属性的话，那就是遍历链表然后挨个复制节点就行了。

这个 random 属性引用的可能是相同的节点，或者在复制的过程中可能 random 引用的节点还没创建好，那么就需要先去把引用的节点去创建好，如果引用的节点还有 random，那么就需要进一步的去创建节点...明显这是个递归的过程。

```java
/**
 * 复杂链表拷贝
 */
public class Solution {
    public static void main(String[] args) {
        Node n1 = new Node(3);
        Node n2 = new Node(3);
        Node n3 = new Node(3);

        n1.next = n2;
        n2.next = n3;
        n2.random = n1;

        // 拷贝后结果
        Node curr = copyRandomList(n1);
        System.out.println(curr.val + "-" + curr.random + "-" + curr);
        while (curr.next != null) {
            Node next = curr.next;
            System.out.println(next.val + "--" + next.random + "-" + next);
            curr = next;
        }
    }

    /**
     * 使用 map 记录已经拷贝的节点
     */
    static Map<Node, Node> cachedNode = new HashMap<>();

    public static Node copyRandomList(Node head) {
        if (head == null) {
            return null;
        }
        if (!cachedNode.containsKey(head)) {
            Node headNew = new Node(head.val);
            cachedNode.put(head, headNew);
            headNew.next = copyRandomList(head.next);
            headNew.random = copyRandomList(head.random);
        }
        return cachedNode.get(head);
    }

    static class Node implements Serializable {
        int val;
        Node next;
        Node random;

        public Node(int val) {
            this.val = val;
            this.next = null;
            this.random = null;
        }
    }
}
```

递归操作并不是很容易去理解，可以考虑在遍历链表的时候直接把当前节点拷贝一份，这样可以得到前后关系，然后再遍历一次处理 random 关系，最后遍历一次把新的节点给拿出来。

![](https://blog.lichenghao.cn/upload/2022/07/06162339.png)

```java
/**
 * 复杂链表拷贝
 */
public class Solution {
    public static void main(String[] args) {
        Node n1 = new Node(1);
        Node n2 = new Node(2);
        Node n3 = new Node(3);

        n1.next = n2;
        n2.next = n3;
        n2.random = n1;

        // 拷贝后结果
        Node curr = copyRandomList(n1);
        System.out.println(curr.val + "-" + curr.random + "-" + curr);
        while (curr.next != null) {
            Node next = curr.next;
            System.out.println(next.val + "--" + next.random + "-" + next);
            curr = next;
        }
    }

    public static Node copyRandomList(Node head) {
        if (head == null) {
            return null;
        }
        // 复制节点
        for (Node node = head; node != null; node = node.next.next) {
            Node nodeNew = new Node(node.val);
            nodeNew.next = node.next;
            node.next = nodeNew;
        }
        // 复制 random 引用
        for (Node node = head; node != null; node = node.next.next) {
            // 下一个节点就是复制的节点
            Node next = node.next;
            if (node.random != null) {
                // 给复制的节点设置 random
                next.random = node.random.next;
            } else {
                next.random = null;
            }
        }
        // 再把老的节点指向老的节点，新的节点指向新的节点
        Node headNew = head.next;
        for (Node node = head; node != null; node = node.next) {
            Node nodeNew = node.next;
            node.next = node.next.next;
            nodeNew.next = (nodeNew.next != null) ? nodeNew.next.next : null;
        }
        return headNew;
    }

    static class Node implements Serializable {
        int val;
        Node next;
        Node random;

        public Node(int val) {
            this.val = val;
            this.next = null;
            this.random = null;
        }
    }
}
```



## 04. 二维数组中的查找

题目描述：

> 在一个 n * m 的二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个高效的函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

示例：

```java
[
  [1,   4,  7, 11, 15],
  [2,   5,  8, 12, 19],
  [3,   6,  9, 16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]

给定 target = 5，返回 true。
给定 target = 20，返回 false。
```



如果你很长时间不使用二维数组，那么你可能定义个二维数组都有点费劲！ 暴力的方式直接遍历二维数据。

```java
/**
 * 二维数组查找
 */
public class Solution {

    public boolean findNumberIn2DArray(int[][] matrix, int target) {
        if (matrix.length == 0) {
            return false;
        }
        for (int[] ints : matrix) {
            for (int i1 = 0; i1 < matrix[0].length; i1++) {
                if (ints[i1] == target) {
                    return true;
                }
            }
        }
        return false;
    }

    public static void main(String[] args) {
        int[][] s = {{1, 4, 7, 11, 15}, {2, 5, 8, 12, 19}, {3, 6, 9, 16, 22}, {10, 13, 14, 17, 24}, {18, 21, 23, 26, 30}};
        int[][] s2 = new int[0][];
        System.out.println(new Solution().findNumberIn2DArray(s2, 1));
    }
}
```

因为这个二维数组是有特点的，从左上到右下是成增长趋势的，所以根据大小可以排除一部分数据。例如20 以上的数字肯定不在前两行，同理 15 以下的数字肯定不在最后一列。

那么我们从每一行的最后一列往前找，如果当前数值 n>target 那么，那么这一列肯定不用考虑了（因为这一列下面的会更大）；那么继续往前找，当前的值 n<target 了说明前面的值肯定都小于 target了，那么就找下一行；如果一样就返回了；

![](https://blog.lichenghao.cn/upload/2022/07/11105353.png)



代码

```java
/**
 * 二维数组查找
 */
public class Solution {

    public boolean findNumberIn2DArray(int[][] matrix, int target) {
        if (matrix.length == 0 || matrix[0].length == 0) {
            return false;
        }
        int rowLength = matrix.length;
        int colLength = matrix[0].length;
        int row = 0;
        int col = colLength - 1;
        while (row < rowLength && col >= 0) {
            int curr = matrix[row][col];
            if (curr == target) {
                return true;
            } else if (curr > target) {
                col--;
            } else {
                row++;
            }
        }
        return false;
    }

    public static void main(String[] args) {
        int[][] s = {{1, 4, 7, 11, 15}, {2, 5, 8, 12, 19}, {3, 6, 9, 16, 22}, {10, 13, 14, 17, 24}, {18, 21, 23, 26, 30}};
        int[][] s2 = new int[0][];
        int[][] s3 = {{-1}, {-1}};
        int[][] s4 = {{-5}};
        System.out.println(new Solution().findNumberIn2DArray(s4, -5));
    }
}
```



## 32 - I. 从上到下打印二叉树

题目描述：

> 从上到下打印出二叉树的每个节点，同一层的节点按照从左到右的顺序打印。

给定二叉树: [3,9,20,null,null,15,7]

```
    3
   / \
  9  20
    /  \
   15   7
```

结果：

```
[3,9,20,15,7]
```



每层按照左右顺序打印，这其实就是 BFS 广度优先搜索。可以利用队列的先进先出的特点，将节点按照顺序入队，然后出队即可。

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Queue;
import java.util.concurrent.LinkedBlockingDeque;

/**
 * 二叉树层序遍历（BFS,广度优先搜索）
 */
public class Solution {

    public int[] levelOrder(TreeNode root) {
        if (root == null) {
            return new int[]{};
        }
        Queue<TreeNode> queue = new LinkedBlockingDeque<>();
        List<Integer> list = new ArrayList<>();
        queue.add(root);
        while (!queue.isEmpty()) {
            TreeNode poll = queue.poll();
            list.add(poll.val);
            if (poll.left != null) {
                queue.offer(poll.left);
            }
            if (poll.right != null) {
                queue.offer(poll.right);
            }
        }
        if (list.isEmpty()) {
            return new int[]{};
        }
        int[] res = new int[list.size()];
        for (int i = 0; i < list.size(); i++) {
            res[i] = list.get(i);
        }
        return res;
    }

    static class TreeNode {
        int val;
        TreeNode left;
        TreeNode right;

        TreeNode(int x) {
            val = x;
        }

        @Override
        public String toString() {
            return "TreeNode{" +
                    "val=" + val +
                    ", left=" + left +
                    ", right=" + right +
                    '}';
        }
    }

    public static void main(String[] args) {
        TreeNode treeNode_3 = new TreeNode(3);
        TreeNode treeNode_9 = new TreeNode(9);
        TreeNode treeNode_20 = new TreeNode(20);
        TreeNode treeNode_15 = new TreeNode(15);
        TreeNode treeNode_7 = new TreeNode(7);

        treeNode_3.left = treeNode_9;
        treeNode_3.right = treeNode_20;
        treeNode_20.left = treeNode_15;
        treeNode_20.right = treeNode_7;

        int[] ints = new Solution().levelOrder(treeNode_3);
        System.out.println(Arrays.toString(ints));
    }
}
```



## 32 - II. 从上到下打印二叉树 II

题目描述：

> 从上到下按层打印二叉树，同一层的节点按从左到右的顺序打印，每一层打印到一行。



例如:
给定二叉树: `[3,9,20,null,null,15,7]`,

```
    3
   / \
  9  20
    /  \
   15   7
```

返回结果：

返回其层次遍历结果：

```
[
  [3],
  [9,20],
  [15,7]
]
```



相比上一个打印，这次需要将每一层的节点放在一起。只需要在队列中记录每一层的节点就可以了。

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Queue;
import java.util.concurrent.LinkedBlockingDeque;

/**
 * 二叉树层序遍历
 */
public class Solution {

    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> list = new ArrayList<>();
        // 记录每一层的节点
        Queue<List<TreeNode>> queue = new LinkedBlockingDeque<>();
        if (root == null) {
            return list;
        }
        // 初始化队列
        queue.offer(Collections.singletonList(root));
        while (!queue.isEmpty()) {
            // 当前层的节点
            List<TreeNode> currList = queue.poll();
            List<Integer> value = new ArrayList<>();
            // 下一层的节点
            List<TreeNode> nextList = new ArrayList<>();
            // 拿当前层的值，同时把下一层的节点入队列
            for (TreeNode treeNode : currList) {
                int val = treeNode.val;
                value.add(val);
                if (treeNode.left != null) {
                    nextList.add(treeNode.left);
                }
                if (treeNode.right != null) {
                    nextList.add(treeNode.right);
                }
            }
            if (!nextList.isEmpty()) {
                queue.offer(nextList);
            }
            list.add(value);
        }
        return list;
    }

    static class TreeNode {
        int val;
        TreeNode left;
        TreeNode right;

        TreeNode(int x) {
            val = x;
        }

        @Override
        public String toString() {
            return "TreeNode{" +
                    "val=" + val +
                    ", left=" + left +
                    ", right=" + right +
                    '}';
        }
    }

    public static void main(String[] args) {
        TreeNode treeNode_3 = new TreeNode(3);
        TreeNode treeNode_9 = new TreeNode(9);
        TreeNode treeNode_20 = new TreeNode(20);
        TreeNode treeNode_21 = new TreeNode(21);
        TreeNode treeNode_22 = new TreeNode(22);
        TreeNode treeNode_15 = new TreeNode(15);
        TreeNode treeNode_7 = new TreeNode(7);

        treeNode_3.left = treeNode_9;
        treeNode_3.right = treeNode_20;
        treeNode_9.left = treeNode_7;
        treeNode_9.right = treeNode_15;
        treeNode_20.left = treeNode_21;
        treeNode_20.right = treeNode_22;

        List<List<Integer>> lists = new Solution().levelOrder(treeNode_3);
        System.out.println(lists);
    }
}
```



## 32 - III. 从上到下打印二叉树 III

题目描述：

> 请实现一个函数按照之字形顺序打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右到左的顺序打印，第三行再按照从左到右的顺序打印，其他行以此类推。

例如:
给定二叉树: [3,9,20,null,null,15,7],

```
    3
   / \
  9  20
    /  \
   15   7
```

返回其层次遍历结果：

```
[
  [3],
  [20,9],
  [15,7]
]
```

有了上面的层序遍历结果后，将结果按照不同的顺序排序下就可以了

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Queue;
import java.util.concurrent.LinkedBlockingDeque;

/**
 * 二叉树层序遍历
 */
public class Solution {

    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> list = new ArrayList<>();
        // 记录每一层的节点
        Queue<List<TreeNode>> queue = new LinkedBlockingDeque<>();
        if (root == null) {
            return list;
        }
        // 初始化队列
        queue.offer(Collections.singletonList(root));
        while (!queue.isEmpty()) {
            // 当前层的节点
            List<TreeNode> currList = queue.poll();
            List<Integer> value = new ArrayList<>();
            // 下一层的节点
            List<TreeNode> nextList = new ArrayList<>();
            // 拿当前层的值，同时把下一层的节点入队列
            for (TreeNode treeNode : currList) {
                int val = treeNode.val;
                value.add(val);
                if (treeNode.left != null) {
                    nextList.add(treeNode.left);
                }
                if (treeNode.right != null) {
                    nextList.add(treeNode.right);
                }
            }
            if (!nextList.isEmpty()) {
                queue.offer(nextList);
            }
            list.add(value);
        }
        List<List<Integer>> res = new ArrayList<>();
        for (int i = 0; i < list.size(); i++) {
            List<Integer> tmpList = list.get(i);
            if (i % 2 == 0) {
                res.add(tmpList);
            } else {
                List<Integer> t = new ArrayList<>();
                for (int i1 = tmpList.size() - 1; i1 >= 0; i1--) {
                    Integer integer = tmpList.get(i1);
                    t.add(integer);
                }
                res.add(t);
            }
        }
        return res;
    }

    static class TreeNode {
        int val;
        TreeNode left;
        TreeNode right;

        TreeNode(int x) {
            val = x;
        }

        @Override
        public String toString() {
            return "TreeNode{" +
                    "val=" + val +
                    ", left=" + left +
                    ", right=" + right +
                    '}';
        }
    }

    public static void main(String[] args) {
        TreeNode treeNode_3 = new TreeNode(3);
        TreeNode treeNode_9 = new TreeNode(9);
        TreeNode treeNode_20 = new TreeNode(20);
        TreeNode treeNode_15 = new TreeNode(15);
        TreeNode treeNode_7 = new TreeNode(7);

        treeNode_3.left = treeNode_9;
        treeNode_3.right = treeNode_20;
        treeNode_20.left = treeNode_15;
        treeNode_20.right = treeNode_7;

        List<List<Integer>> lists = new Solution().levelOrder(treeNode_3);
        System.out.println(lists);
    }
}
```





## 26. 树的子结构

题目描述：

> 输入两棵二叉树A和B，判断B是不是A的子结构。(约定空树不是任意一个树的子结构)B是A的子结构， 即 A中有出现和B相同的结构和节点值。

例如:
给定的树 A:

```
   3
  / \
 4   5
 / \
1   2
```

给定的树 B：

```
  4 
  /
 1
```

返回 true，因为 B 与 A 的一个子树拥有相同的结构和节点值。



如果 B 是 A 的子树的话，那么A和 B的根节点有如下三种情况：

- A 和 B 的根节点是一样的
- B 在 A 的左子树上
- B 在 A 的右子树上

如果上面的情况都不满足，那么 B 肯定不是 A 的子树。假如确定了 B 的根节点在左子树上，那么就需要判断下 此时 A的左子树和 B 的左子树是否一样，然后比较右子树，直到没有。如果期间出现了不一致那么 B 就不是 A 的子树。



假如我们的 A 和 B 就是根一样的，那么只需要继续判断 A 和 B 的左右子树，这明显是一个递归操作。

假如 A 和 B 的根不是一样的，那么就要变换 A 的节点比如把左子树的根作为根节点去比较（同理右子树）这明显也是一个递归操作。

所以我们这里是两个递归操作，一个是找相同的根，一个是根一样后判断子树是否一致！



```java
/**
 * 子树
 */
public class Solution {

    public boolean isSubStructure(TreeNode A, TreeNode B) {
        if (A == null || B == null) {
            return false;
        }
        return eq(A, B) || isSubStructure(A.left, B) || isSubStructure(A.right, B);
    }

    /**
     * A的根节点和B的根节点相同的情况下，进行如下的比较
     *
     * @param A A树
     * @param B B树
     * @return boolean
     */
    public boolean eq(TreeNode A, TreeNode B) {
        // B 找完了,A 还没完事说明B是A的子树
        if (B == null) {
            return true;
        }
        // 如果节点不一样肯定不是子节点了
        if (A == null || A.val != B.val) {
            return false;
        }
        // 根节点一样了，继续比较左右子树是否一样。相当于把A和B的左子树的根作为根节点继续比较。
        return eq(A.left, B.left) && eq(A.right, B.right);
    }


    static class TreeNode {
        int val;
        TreeNode left;
        TreeNode right;

        TreeNode(int x) {
            val = x;
        }

        @Override
        public String toString() {
            return "TreeNode{" +
                    "val=" + val +
                    ", left=" + left +
                    ", right=" + right +
                    '}';
        }
    }

    public static void main(String[] args) {
        TreeNode treeNode_1 = new TreeNode(1);
        TreeNode treeNode_2 = new TreeNode(2);
        TreeNode treeNode_3 = new TreeNode(3);
        TreeNode treeNode_4 = new TreeNode(4);
        TreeNode treeNode_5 = new TreeNode(5);

        treeNode_3.left = treeNode_4;
        treeNode_3.right = treeNode_5;
        treeNode_4.left = treeNode_1;
        treeNode_4.right = treeNode_2;


        TreeNode treeNode_b_4 = new TreeNode(4);
        TreeNode treeNode_b_1 = new TreeNode(1);
        treeNode_b_4.left = treeNode_b_1;

        boolean subStructure = new Solution().isSubStructure(treeNode_3, treeNode_b_4);
        System.out.println(subStructure);
    }
}
```



##  27. 二叉树的镜像

题目描述：

> 请完成一个函数，输入一个二叉树，该函数输出它的镜像。

例如输入：

```
     4
   /   \
  2     7
 / \   / \
1   3 6   9
```

输出

```
     4
   /   \
  7     2
 / \   / \
9   6 3   1
```



镜像就是左右相反的，交换左右子树

```java
/**
 * 二叉树镜像
 */
public class Solution {
    public TreeNode mirrorTree(TreeNode root) {
        if (root == null) {
            return null;
        }
        TreeNode left = mirrorTree(root.left);
        TreeNode right = mirrorTree(root.right);
        root.left = right;
        root.right = left;
        return root;
    }


    static class TreeNode {
        int val;
        TreeNode left;
        TreeNode right;

        TreeNode(int x) {
            val = x;
        }

        @Override
        public String toString() {
            return "TreeNode{" +
                    "val=" + val +
                    ", left=" + left +
                    ", right=" + right +
                    '}';
        }
    }

    public static void main(String[] args) {
        TreeNode treeNode_1 = new TreeNode(1);
        TreeNode treeNode_2 = new TreeNode(2);
        TreeNode treeNode_3 = new TreeNode(3);
        TreeNode treeNode_4 = new TreeNode(4);
        TreeNode treeNode_5 = new TreeNode(5);

        treeNode_3.left = treeNode_4;
        treeNode_3.right = treeNode_5;
        treeNode_4.left = treeNode_1;
        treeNode_4.right = treeNode_2;

        TreeNode treeNode = new Solution().mirrorTree(treeNode_3);
        System.out.println(treeNode);
    }
}
```



##  28. 对称的二叉树

题目描述：

> 请实现一个函数，用来判断一棵二叉树是不是对称的。如果一棵二叉树和它的镜像一样，那么它是对称的。

例如，二叉树 [1,2,2,3,4,4,3] 是对称的。

```
    1
   / \
  2   2
 / \ / \
3  4 4  3
```

但是下面这个 [1,2,2,null,3,null,3] 则不是镜像对称的:

```
    1
   / \
  2   2
   \   \
   3    3
```

如果一棵树的左子树和右子树是镜像对称的，那么这棵树是对称的。

如果根节点相同，每个树的右子树都与另一个树的左子树镜像对称，那么两棵树是镜像对称的。

所以用两个指针去遍历树，并且不同的方向，一个左一个右。如果左==右，继续判断子树。

```java
/**
 * 对称二叉树
 */
public class Solution {

    public boolean isSymmetric(TreeNode root) {
        return check(root, root);
    }

    public boolean check(TreeNode r1, TreeNode r2) {
        if (r1 == null && r2 == null) {
            return true;
        }
        if (r1 == null || r2 == null) {
            return false;
        }
        return r1.val == r2.val && check(r1.left, r2.right) && check(r1.right, r2.left);
    }

    static class TreeNode {
        int val;
        TreeNode left;
        TreeNode right;

        TreeNode(int x) {
            val = x;
        }

        @Override
        public String toString() {
            return "TreeNode{" +
                    "val=" + val +
                    ", left=" + left +
                    ", right=" + right +
                    '}';
        }
    }

    public static void main(String[] args) {
        TreeNode treeNode_1 = new TreeNode(1);
        TreeNode treeNode_2 = new TreeNode(2);
        TreeNode treeNode_3 = new TreeNode(3);
        TreeNode treeNode_4 = new TreeNode(4);
        TreeNode treeNode_5 = new TreeNode(5);

        treeNode_3.left = treeNode_4;
        treeNode_3.right = treeNode_5;
        treeNode_4.left = treeNode_1;
        treeNode_4.right = treeNode_2;

        System.out.println(new Solution().isSymmetric(treeNode_3));
    }
}
```







