# 102. Binary Tree Level Order Traversal

## 1st

### ①

幅優先探索で左から右に順々に処理すればよい。
append_if_existsはあっても無くてもいいと思う。
キューはレベルごとに用意しているが、同じキューに入れてループを回すタイミングで入っているノードの数分だけ処理をする、を繰り返しても良い。
今回はdequeを使ったけどlistでも良い。内側の `while nodes` が `for node in nodes` になる
level_order_valuesは `list[list[int]]`, current_level_valuesは `list[int]` になっていて少し分かりづらいだろうか。
valuesはcurrent_level_と接頭辞があるので、合わせてnodesの方もcurrent_level_nodesとした方が今回は良かったかもしれない。


所要時間: 9:01

n: ノード数
- 時間計算量: O(n)
- 空間計算量: O(n)

```py
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        def append_if_exists(nodes: deque[TreeNode], node: Optional[TreeNode]):
            if node:
                nodes.append(node)

        if not root:
            return []
        level_order_values = []
        nodes = deque([root])
        while nodes:
            next_level_nodes = deque([])
            current_level_values = []
            while nodes:
                node = nodes.popleft()
                current_level_values.append(node.val)
                append_if_exists(next_level_nodes, node.left)
                append_if_exists(next_level_nodes, node.right)
            nodes = next_level_nodes
            level_order_values.append(current_level_values)
        return level_order_values
```

### ②

再帰。最悪2000回スタックに積まれるので環境によってはRecursionError。
変数名が全体的に長いのでゴチャ付いてる感じがある。やっぱりcurrent_level_の接頭辞は無くてもいいか？

所要時間: 5:32

n: ノード数
- 時間計算量: O(n)
- 空間計算量: O(n)

```py
class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        def level_order_helper(current_level_nodes: list[TreeNode], level_order_values: list[list[int]]) -> list[list[int]]:
            if not current_level_nodes:
                return level_order_values
            current_level_values = []
            next_level_nodes = []
            for node in current_level_nodes:
                current_level_values.append(node.val)
                if node.left:
                    next_level_nodes.append(node.left)
                if node.right:
                    next_level_nodes.append(node.right)
            level_order_values.append(current_level_values)
            return level_order_helper(next_level_nodes, level_order_values)

        if not root:
            return []
        return level_order_helper([root], [])
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1227073733844406343/1238016881412149249
- https://discord.com/channels/1084280443945353267/1196472827457589338/1235538738927370322
- https://discord.com/channels/1084280443945353267/1201211204547383386/1218448578767491152
- https://discord.com/channels/1084280443945353267/1201211204547383386/1218244966946701363
- https://discord.com/channels/1084280443945353267/1200089668901937312/1211206574022991882


あえてこちらで実装するのはそんなにしなさそうだが、深さ優先探索で書いてみる。
`while depth >= len(level_order_values)` は
`if depth == len(level_order_values): # depth <= len(level_order_values)` みたいにコメントでdepthとlistの長さの関係を補足してif文にする方が良かったかも。

```py
class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if not root:
            return []
        node_depth_pairs = [(root, 0)]
        level_order_values = []
        while node_depth_pairs:
            node, depth = node_depth_pairs.pop()
            while depth >= len(level_order_values):
                level_order_values.append([])
            level_order_values[depth].append(node.val)
            if node.right:
                node_depth_pairs.append((node.right, depth + 1))
            if node.left:
                node_depth_pairs.append((node.left, depth + 1))
        return level_order_values
```

## 3rd


```py
class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if not root:
            return []
        nodes = [root]
        level_order_values = []
        while nodes:
            next_level_nodes = []
            values = []
            for node in nodes:
                values.append(node.val)
                if node.left:
                    next_level_nodes.append(node.left)
                if node.right:
                    next_level_nodes.append(node.right)
            nodes = next_level_nodes
            level_order_values.append(values)
        return level_order_values
```
