# 剑指 Offer 37. 序列化二叉树

https://leetcode-cn.com/problems/xu-lie-hua-er-cha-shu-lcof/

## 题目描述

请实现两个函数，分别用来序列化和反序列化二叉树。

你需要设计一个算法来实现二叉树的序列化与反序列化。这里不限定你的序列 / 反序列化算法执行逻辑，你只需要保证一个二叉树可以被序列化为一个字符串并且将这个字符串反序列化为原始的树结构。

**提示：**输入输出格式与 LeetCode 目前使用的方式一致，详情请参阅 [LeetCode 序列化二叉树的格式](https://support.leetcode-cn.com/hc/kb/category/1018267/)。你并非必须采取这种方式，你也可以采用其他的方法解决这个问题。

 

**示例：**

![](https://images.yingwai.top/picgo/20210701144239.jpg)

```
输入：root = [1,2,3,null,null,4,5]
输出：[1,2,3,null,null,4,5]
```

 

注意：本题与主站 297 题相同：https://leetcode-cn.com/problems/serialize-and-deserialize-binary-tree/



## 思路

层序遍历：

根据题目提供的二叉树格式，可以知道对应着二叉树的层序遍历序列，只是不忽略中间的 `null` 值。因此可以借助层序遍历来进行序列化和反序列化：

* **序列化：**
  * 直接使用层序遍历构造字符串即可，注意数字的处理以及最后多余 `null` 值的删除；
* **反序列化：**
  * 首先构造出二叉树节点值的列表，不忽略其中的 `null` 值；
  * 然后借助两个队列开始层序遍历，一个队列 `q` 存储的节点为还没有设置父节点的节点集合，另一个队列 `wait` 存储的节点为已经设置了父节点但还没有设置孩子节点的节点集合；
  * 每访问到 `wait` 中的一个值，就把 `q` 队列中的前两个元素设置为它的左右子节点（根据层序遍历特点，`q` 中的前两个元素一定为当前访问节点的左右子节点），然后将其左右子节点加入到队列 `wait` 中；
  * 重复执行第三步，直到 `wait` 队列为空；
  * 一开始 `wait` 队列中只有 `root` 一个元素。



## 代码

```java
public class Codec {

    // Encodes a tree to a single string.
    public String serialize(TreeNode root) {
        StringBuilder sb = new StringBuilder();
        sb.append('[');
        Queue<TreeNode> q = new LinkedList<>();
        q.offer(root);
        // 记录最后一次出现数字的位置
        int lastIndex = 0;
        // 层序遍历构造字符串
        while (q.size() > 0) {
            int size = q.size();
            for (int i = 0; i < size; ++i) {
                TreeNode tmp = q.poll();
                if (tmp != null) {
                    // 节点不为null，记录节点值
                    if (tmp.val < 0) {
                        sb.append('-');
                    }
                    // 将数值转字符串
                    sb.append(itoa(Math.abs(tmp.val)));
                    sb.append(',');
                    lastIndex = sb.length();
                    // 将当前访问节点的子节点加入队列
                    q.offer(tmp.left);
                    q.offer(tmp.right);
                } else {
                    // 否则记录null值
                    sb.append("null,");
                }
            }
        }
        if (lastIndex > 0) {
            // 最后还要删除多余的null值
            sb.delete(lastIndex-1, sb.length());
        }
        sb.append(']');
        return sb.toString();
    }

    // Decodes your encoded data to tree.
    public TreeNode deserialize(String data) {
        List<Integer> list = new ArrayList<>();
        char[] c = data.toCharArray();
        // 构造节点值列表
        for (int i = 0; i < c.length; ++i) {
            if (c[i] == 'n') {
                // 存储空节点
                list.add(null);
            } else if ((c[i] >= '0' && c[i] <= '9') || c[i] == '-'){
                // 处理非空节点的值
                int tmp = 0, signal = 1;
                if (c[i] == '-') {
                    // 处理负数
                    signal = -1;
                    ++i;
                }
                while (c[i] >= '0' && c[i] <= '9') {
                    tmp = tmp*10 + c[i] - '0';
                    ++i;
                }
                list.add(tmp * signal);
            }
        }
        // 层序遍历构造二叉树
        // q队列存储子节点列表，与wait队列配合
        Queue<Integer> q = new LinkedList<>();
        for (Integer e : list) {
            q.offer(e);
        }
        // 根节点不为空才开始构造
        if (q.size() > 0 && q.peek() != null) {
            // wait队列存储还没设置子节点的节点
            Queue<TreeNode> wait = new LinkedList<>();
            TreeNode root = new TreeNode(q.poll());
            wait.offer(root);
            while (wait.size() > 0) {
                TreeNode tmp = wait.poll();
                if (q.size() > 0) {
                    // 根据层序遍历的特点，q队列的头两个元素一定是当前遍历节点的左右子节点
                    Integer leftVal = q.poll();
                    Integer rightVal = q.poll();
                    if (leftVal != null) {
                        TreeNode left = new TreeNode(leftVal);
                        tmp.left = left;
                        wait.offer(left);
                    }
                    if (rightVal != null) {
                        TreeNode right = new TreeNode(rightVal);
                        tmp.right = right;
                        wait.offer(right);
                    }
                }
            }
            return root;
        }
        return null;
    }

    private String itoa(Integer i) {
        StringBuilder sb = new StringBuilder();
        if (i > 0) {
            while (i > 0) {
                sb.append((char)(i%10 + '0'));
                i /= 10;
            }
        } else {
            sb.append('0');
        }
        return sb.reverse().toString();
    }

}
```

