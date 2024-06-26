# 112. Path Sum

## 1st

### ①

再帰で書ける問題はそちらが書きやすいことが多いので、1stは再帰になることが多い。
最大ノード数くらい再帰されるので、問題設定上5000回再帰され得るので上手くいかないケースもあるねという話はするだろう。

`_is_leaf()`は関数化するか微妙なライン。何を調べてるか分かりやすくはなると思う。
`not (node.left or node.right)` は `not node.left and not node.right` でもどちらでも。字面は前者の方がいいけど理解のし易さなら後者だろうか。

所要時間: 8:35

n: ノード数, h: 木の高さ
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
    def hasPathSum(self, node: Optional[TreeNode], targetSum: int) -> bool:
        if not node:
            return False
        if self._is_leaf(node):
            return targetSum == node.val
        rest = targetSum - node.val
        return self.hasPathSum(node.left, rest) or self.hasPathSum(node.right, rest)

    def _is_leaf(self, node: TreeNode) -> bool:
        assert node
        return not (node.left or node.right)
```

### ②

スタックを使った版。キューを使ってもいいがコードはほとんど同じなので省略する。
DFSとBFSはこの問題ならどちらを使っても大して変わらないんじゃないだろうか (どちらに対してもそれぞれに意地悪な入力は作れそう)。

所要時間: 3:19

n: ノード数, h: 木の高さ
- 時間計算量: O(n)
- 空間計算量: O(h)

```py
class Solution:
    def hasPathSum(self, root: Optional[TreeNode], targetSum: int) -> bool:
        def is_leaf(node: TreeNode) -> bool:
            assert node
            return not (node.left or node.right)

        if not root:
            return False
        node_target_pairs = [(root, targetSum)]
        while node_target_pairs:
            node, target = node_target_pairs.pop()
            if is_leaf(node) and node.val == target:
                return True
            if node.left:
                node_target_pairs.append((node.left, target - node.val))
            if node.right:
                node_target_pairs.append((node.right, target - node.val))
        return False
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1227073733844406343/1237658778166624276

再帰の引数がOptionalじゃないバージョン。
`node.right and self._has_path_sum_helper(node.right, rest)` は直接returnすればよかった。その場合boolにするために `node.right is not None and ...` と書くか。


```py
class Solution:
    def hasPathSum(self, root: Optional[TreeNode], targetSum: int) -> bool:
        if not root:
            return False
        return self._has_path_sum_helper(root, targetSum)

    def _has_path_sum_helper(self, node: TreeNode, target: int) -> bool:
        if not (node.left or node.right):
            return node.val == target
        rest = target - node.val
        if node.left and self._has_path_sum_helper(node.left, rest):
            return True
        if node.right and self._has_path_sum_helper(node.right, rest):
            return True
        return False
```

- https://discord.com/channels/1084280443945353267/1200089668901937312/1210852552749355008

targetSumから引くのではなく0から足して葉でtargetSumと同じになったらOK、のパターン。
totalはsum_so_farでも。

```py
class Solution:
    def hasPathSum(self, root: Optional[TreeNode], targetSum: int) -> bool:
        def has_path_sum_helper(node: Optional[TreeNode], total: int) -> bool:
            if not node:
                return False
            if is_leaf(node):
                return total + node.val == targetSum
            total += node.val
            return has_path_sum_helper(node.left, total) or has_path_sum_helper(node.right, total)

        def is_leaf(node: TreeNode) -> bool:
            assert node
            return not node.left and not node.right

        return has_path_sum_helper(root, 0)
```

## 3rd

```py
class Solution:
    def hasPathSum(self, root: Optional[TreeNode], targetSum: int) -> bool:
        def is_leaf(node: TreeNode) -> bool:
            assert node
            return not node.left and not node.right

        if not root:
            return False
        node_target_pairs = [(root, targetSum)]
        while node_target_pairs:
            node, target = node_target_pairs.pop()
            if is_leaf(node) and target == node.val:
                return True
            rest = target - node.val
            if node.left:
                node_target_pairs.append((node.left, rest))
            if node.right:
                node_target_pairs.append((node.right, rest))
        return False
```

