# 103. Binary Tree Zigzag Level Order Traversal

## 1st

### ①

何度か練習したので覚えていた。from_leftはis_from_leftでもいいが十分boolであることは分かると思いこうしている。

所要時間: 5:39

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
    def zigzagLevelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        def append_if_exists(nodes: list[TreeNode], node: Optional[TreeNode]):
            if node:
                nodes.append(node)

        if not root:
            return []
        nodes = [root]
        from_left = True
        level_order_values = []
        while nodes:
            values = []
            next_level_nodes = []
            while nodes:
                node = nodes.pop()
                values.append(node.val)
                if from_left:
                    append_if_exists(next_level_nodes, node.left)
                    append_if_exists(next_level_nodes, node.right)
                else:
                    append_if_exists(next_level_nodes, node.right)
                    append_if_exists(next_level_nodes, node.left)
            level_order_values.append(values)
            nodes = next_level_nodes
            from_left = not from_left
        return level_order_values
```

### ②

右からのときはリストを反転させるパターン。難しく考えず、こういう解法を選択肢として出せるようにしておきたい。

所要時間: 3:38

n: ノード数

- 時間計算量: O(n)
- 空間計算量: O(n)

```py
class Solution:
    def zigzagLevelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if not root:
            return []
        nodes = [root]
        level_order_values = []
        from_right = False
        while nodes:
            values = []
            next_level_nodes = []
            for node in nodes:
                values.append(node.val)
                if node.left:
                    next_level_nodes.append(node.left)
                if node.right:
                    next_level_nodes.append(node.right)
            if from_right:
                values.reverse()
            level_order_values.append(values)
            nodes = next_level_nodes
            from_right = not from_right
        return level_order_values
```

### ③

再帰の練習として書いた。

所要時間: 9:46

n: ノード数、h: 木の高さ

- 時間計算量: O(n)
- 空間計算量: O(h)

```py
class Solution:
    def zigzagLevelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if not root:
            return []
        level_order_values = []

        def append_if_exists(nodes: list[TreeNode], node: Optional[TreeNode]):
            if node:
                nodes.append(node)

        def zigzag_level_order_helper(nodes: list[TreeNode], from_left: bool):
            if not nodes:
                return
            next_level_nodes = []
            values = []
            while nodes:
                node = nodes.pop()
                values.append(node.val)
                if from_left:
                    append_if_exists(next_level_nodes, node.left)
                    append_if_exists(next_level_nodes, node.right)
                else:
                    append_if_exists(next_level_nodes, node.right)
                    append_if_exists(next_level_nodes, node.left)
            level_order_values.append(values)
            zigzag_level_order_helper(next_level_nodes, not from_left)

        zigzag_level_order_helper([root], True)
        return level_order_values
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1192736784354918470/1239577010200641607
- https://discord.com/channels/1084280443945353267/1227073733844406343/1238810748856045570
- https://discord.com/channels/1084280443945353267/1196472827457589338/1236595428557062214


前の問題と同様、あまりDFSで解く気にならないがやってみる。

```py
class Solution:
    def zigzagLevelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if not root:
            return []
        level_to_values = defaultdict(deque)
        nodes = [root]
        level = 0
        while nodes:
            next_level_nodes = []
            for node in nodes:
                if level % 2 == 0:
                    level_to_values[level].append(node.val)
                else:
                    level_to_values[level].appendleft(node.val)
                if node.left:
                    next_level_nodes.append(node.left)
                if node.right:
                    next_level_nodes.append(node.right)
            nodes = next_level_nodes
            level += 1
        return [list(values) for values in level_to_values.values()]
```

## 3rd


```py
class Solution:
    def zigzagLevelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        def append_if_exists(nodes: list[TreeNode], node: Optional[TreeNode]):
            if not node:
                return
            nodes.append(node)

        if not root:
            return []
        nodes = [root]
        level_order_values = []
        from_left = True
        while nodes:
            next_level_nodes = []
            values = []
            while nodes:
                node = nodes.pop()
                values.append(node.val)
                if from_left:
                    append_if_exists(next_level_nodes, node.left)
                    append_if_exists(next_level_nodes, node.right)
                else:
                    append_if_exists(next_level_nodes, node.right)
                    append_if_exists(next_level_nodes, node.left)
            nodes = next_level_nodes
            level_order_values.append(values)
            from_left = not from_left
        return level_order_values
```
