# 树



## 遍历

二叉树一共有四种遍历方式，下面给出各种遍历方式的代码实现。

### 前序遍历

* **递归：**

  ```java
  class Solution {
      public List<Integer> preorderTraversal(TreeNode root) {
          List<Integer> res = new ArrayList<>();
          recur(root, res);
          return res;
      }
  
      private void recur(TreeNode root, List<Integer> list) {
          if (root != null) {
              list.add(root.val);
              recur(root.left, list);
              recur(root.right, list);
          }
      }
  }
  ```

* **迭代：**

  ```java
  class Solution {
      public List<Integer> preorderTraversal(TreeNode root) {
          Stack<TreeNode> stack = new Stack<>();
          List<Integer> res = new ArrayList<>();
          if (root != null) {
              stack.push(root);
              while (!stack.isEmpty()) {
                  TreeNode p = stack.pop();
                  while (p != null) {
                      res.add(p.val);
                      stack.push(p.right);
                      p = p.left;
                  }
              }
          }
          return res;
      }
  }
  ```

  



### 中序遍历

* **递归：**

  ```java
  class Solution {
      public List<Integer> inorderTraversal(TreeNode root) {
          List<Integer> res = new ArrayList<>();
          recur(root, res);
          return res;
      }
  
      public void recur(TreeNode node, List<Integer> arr) {
          if (node == null) return;
          inOrder(node.left, arr);
          arr.add(node.val);
          inOrder(node.right, arr);
      }
  }
  ```

* **迭代：**

  ```java
  class Solution {
      public List<Integer> inorderTraversal(TreeNode root) {
          Stack<TreeNode> s = new Stack<>();
          List<Integer> res = new ArrayList<>();
          TreeNode p = root;
          while (p != null || !s.isEmpty()) {
              while (p != null) {
                  s.push(p);
                  p = p.left;
              }
              p = s.pop();
              res.add(p.val);
              p = p.right;
          }
          return res;
      }
  }
  ```

  迭代算法中需要用到一个栈，每到一个节点先将其入栈，遍历其左子树，然后访问该节点，访问完后该节点就可以出栈了，最后遍历其右子树。



### 后序遍历

* 递归：

  ```java
  class Solution {
      public List<Integer> postorderTraversal(TreeNode root) {
          List<Integer> res = new LinkedList<>();
          recur(root, res);
          return res;
      }
  
      private void recur(TreeNode root, List<Integer> list) {
          if (root != null) {
              recur(root.left, list);
              recur(root.right, list);
              list.add(root.val);
          }
      }
  }
  ```

* 迭代：

  ```java
  class Solution {
      public List<Integer> postorderTraversal(TreeNode root) {
          Stack<TreeNode> stack = new Stack<>();
          List<Integer> res = new LinkedList<>();
          TreeNode pre = null;
          while (root != null || !stack.isEmpty()) {
              while (root != null) {
                  stack.push(root);
                  root = root.left;
              }
              root = stack.pop();
              if (root.right == null || root.right == pre) {
                  res.add(root.val);
                  pre = root;
                  root = null;
              } else {
                  stack.push(root);
                  root = root.right;
              }
          }
          return res;
      }
  }
  ```

  迭代算法需要维护一个变量 `pre` 用于判断是第几次遇到父节点。根据后序遍历的性质可知，在遍历节点的过程中第二次遇到对应节点才对对应节点进行访问，`pre` 就起到判断的作用。



### 层序遍历

使用队列，存储待访问的节点。每次循环开始时，先统计队列当前的长度，以便区分当前层与下一层的节点。每访问队列中的一个元素，就将其加入到结果中，然后将其左右子节点加入到队列中，直到遍历完当前层的所有节点。然后进入下一层继续上面的操作，直到访问完所有的元素。

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> res = new ArrayList<>();
        if (root != null) {
            TreeNode p = root;
            Queue<TreeNode> q = new LinkedList<>();
            q.offer(p);
            while (q.size() > 0) {
                List<Integer> list = new ArrayList<>();
                int size = q.size();
                // 只访问当前层的节点
                for (int i = 0; i < size; ++i) {
                    p = q.poll();
                    list.add(p.val);
                    if (p.left != null) {
                        q.offer(p.left);
                    }
                    if (p.right != null) {
                        q.offer(p.right);
                    }
                }
                res.add(list);
            }
        }
        return res;
    }
}
```

