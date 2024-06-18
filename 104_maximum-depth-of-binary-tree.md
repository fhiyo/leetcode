# 104. Maximum Depth of Binary Tree

## 1st

### ①

再帰が簡単そうだったのでそれで実装した。ノード数が最大10Kで、深さも最大10Kになるのでleetcodeでなければ上限を超えそうで怪しいが。

所要時間: 3:12

n: ノード数, h: 木の深さ
- 時間計算量: O(n)
- 空間計算量: O(h)

```py
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def maxDepth(self, node: Optional[TreeNode]) -> int:
        if node is None:
            return 0
        return max(self.maxDepth(node.left), self.maxDepth(node.right)) + 1
```

### ②

幅優先探索。ある高さにあるノードの数が多いとその分メモリを消費する。

所要時間: 5:11

n: ノード数
- 時間計算量: O(n)
- 空間計算量: O(n)

```py
class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        def append_if_exists(nodes: list[TreeNode], node: Optional[TreeNode]):
            if node is None:
                return
            nodes.append(node)

        if root is None:
            return 0
        depth = 0
        nodes = [root]
        while nodes:
            next_depth_nodes = []
            for node in nodes:
                append_if_exists(next_depth_nodes, node.left)
                append_if_exists(next_depth_nodes, node.right)
            nodes = next_depth_nodes
            depth += 1
        return depth
```

### ③

深さ優先探索でスタックを使用。帰りがけの処理がないので楽に書ける。

所要時間: 4:16

n: ノード数, h: 木の深さ
- 時間計算量: O(n)
- 空間計算量: O(h)

```py
class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        if root is None:
            return 0
        max_depth = 0
        nodes_with_depth = [(root, 1)]
        while nodes_with_depth:
            node, depth = nodes_with_depth.pop()
            max_depth = max(max_depth, depth)
            if node.left is not None:
                nodes_with_depth.append((node.left, depth + 1))
            if node.right is not None:
                nodes_with_depth.append((node.right, depth + 1))
        return max_depth
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1226508154833993788/1245729221754617908
- https://discord.com/channels/1084280443945353267/1227073733844406343/1236232864098684928
  - https://discord.com/channels/1084280443945353267/1227073733844406343/1236695050902048899

  帰りがけの処理を行う再帰をスタックで実装したバージョン。

```py
class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        result_depth_ref = [None]
        stack = [('go', root, result_depth_ref, [None], [None])]
        while stack:
            direction, node, depth_ref, left_depth_ref, right_depth_ref = stack.pop()
            if direction == 'go':
                if node is None:
                    depth_ref[0] = 0
                    continue
                stack.append(('back', node, depth_ref, left_depth_ref, right_depth_ref))
                stack.append(('go', node.left, left_depth_ref, [None], [None]))
                stack.append(('go', node.right, right_depth_ref, [None], [None]))
                continue
            assert direction == 'back' \
                and left_depth_ref[0] is not None \
                and right_depth_ref[0] is not None
            depth_ref[0] = max(left_depth_ref[0], right_depth_ref[0]) + 1
        return result_depth_ref[0]
```

参考にしたコードではdirectionを使わないようにしていたが、行き帰りが分かりやすいかなと思って使うようにした。

- https://discord.com/channels/1084280443945353267/1196472827457589338/1235261800875036672
- https://discord.com/channels/1084280443945353267/1200089668901937312/1206165123727630406


## 3rd

```py
class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        def append_if_exists(node: Optional[TreeNode], nodes: list[TreeNode]):
            if node is None:
                return
            nodes.append(node)

        if root is None:
            return 0
        depth = 0
        nodes = [root]
        while nodes:
            depth += 1
            next_depth_nodes = []
            for node in nodes:
                append_if_exists(node.left, next_depth_nodes)
                append_if_exists(node.right, next_depth_nodes)
            nodes = next_depth_nodes
        return depth
```
