# DFS & BFS



## 深度优先搜索基本框架

DFS 是一直访问到最底层，再返回，类似二叉树的前序遍历。

```java
public void dfs(int[] arr, int cur) {
    // 访问过当前元素或到达数组边界时返回
    if (cur >= arr.length || visited.contains(cur)) {
        return;
    }
    // 标记当前元素为已访问
    visited.add(cur);
    for (int i = 0; i < arr.length; ++i) {
        // 继续往下访问
        dfs(arr, i);
    }
    visited.remove(cur);
}
```



