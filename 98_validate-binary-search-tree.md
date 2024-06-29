# 98. Validate Binary Search Tree

## 1st

### ①

inorder traversalをしたときに、ノードの値が昇順に整列されていれば良い。
is_headはなくても、 `if prev_value and prev_value >= node.val` のように判定する方がきれいな気がする。

(↑追記: ②同様、prev_valueが0のときにバグるので `if prev_value is not None and ...` とする必要がある)


所要時間: 11:11

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
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        if not root:
            return True
        nodes = []
        node = root
        prev_value = None
        is_head = True
        while nodes or node:
            while node:
                nodes.append(node)
                node = node.left
            node = nodes.pop()
            if is_head:
                is_head = False
            elif prev_value >= node.val:
                return False
            prev_value = node.val
            node = node.right
        return True
```

### ②

inorder traversalすればいいので再帰でも良い。最大10K再帰するので環境次第でRecursionError。
あまり上手く書けている感じはしない。
prev_valueはint | Noneなので、 `if prev_value is not None and prev_value >= node.val` の行で `if prev_value and ...` と書くと prev_value が 0のときにFalseになってしまうので注意 (そこのデバッグに時間を使った)。

所要時間: 13:19

n: ノード数
- 時間計算量: O(n)
- 空間計算量: O(n)

```py
class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        def get_is_valid_and_prev_value(
            node: Optional[TreeNode],
            prev_value: Optional[int]
        ) -> tuple[bool, int]:
            if node.left:
                is_valid, prev_value = get_is_valid_and_prev_value(node.left, prev_value)
                if not is_valid:
                    return False, None
            if prev_value is not None and prev_value >= node.val:
                return False, None
            if node.right:
                return get_is_valid_and_prev_value(node.right, node.val)
            return True, node.val

        if not root:
            return True
        return get_is_valid_and_prev_value(root, None)[0]
```

### ③

問題文にあるvalid BSTの条件に沿って書いてみたもの。
1. 左部分木があればその最大値がノードの値より小さい
2. 右部分木があればその最小値がノードの値より大きい

という条件がすべてのノードについて満たされていれば良い (ただし、ノードが存在しなければvalidとしている)

cacheだとプロセスが生きている限り際限なくキャッシュされるので微妙かもしれない。
lru_cacheのmaxsizeで調節しても良いかも。最大10Kなのでそれを設定するのが無難？
calculate_min_maxは構造体を作ってそれを返すようにしてもいいが、今回は省略。

所要時間: 15:00

n: ノード数
- 時間計算量: O(n)
- 空間計算量: O(n)

```py
class Solution:
    def isValidBST(self, node: Optional[TreeNode]) -> bool:
        if not node:
            return True
        if node.left:
            _, left_max_val = self.calclate_min_max(node.left)
            if left_max_val >= node.val:
                return False
        if node.right:
            right_min_val, _ = self.calclate_min_max(node.right)
            if right_min_val <= node.val:
                return False
        return self.isValidBST(node.left) and self.isValidBST(node.right)

    @cache
    def calclate_min_max(self, node: TreeNode) -> tuple[int, int]:
        min_val = node.val
        max_val = node.val
        if node.left:
            left_min_val, left_max_val = self.calclate_min_max(node.left)
            min_val = min(min_val, left_min_val)
            max_val = max(max_val, left_max_val)
        if node.right:
            right_min_val, right_max_val = self.calclate_min_max(node.right)
            min_val = min(min_val, right_min_val)
            max_val = max(max_val, right_max_val)
        return min_val, max_val
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1227073733844406343/1238704664862785586

①のprev_valueは初期値を`-inf`にすればスッキリはするなと気づいた。Noneにする方が条件を素直に書いているとは思うが。

(is_valid, min_val, max_val)の3つ組を再帰的に返す。下から情報を受け取る方法。

```py
ValidAndMinMaxOfTree = namedtuple('ValidAndMinMaxOfTree', ['is_valid', 'min_val', 'max_val'])


class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        def is_valid_bst_helper(node: Optional[TreeNode]) -> ValidAndMinMaxOfTree:
            min_val = node.val
            max_val = node.val
            if not node:
                return ValidAndMinMaxOfTree(True, -float('inf'), float('inf'))
            if node.left:
                left_is_valid, left_min_val, left_max_val = is_valid_bst_helper(node.left)
                if not left_is_valid or left_max_val >= node.val:
                    return ValidAndMinMaxOfTree(False, -float('inf'), float('inf'))
                min_val = left_min_val
            if node.right:
                right_is_valid, right_min_val, right_max_val = is_valid_bst_helper(node.right)
                if not right_is_valid or right_min_val <= node.val:
                    return ValidAndMinMaxOfTree(False, -float('inf'), float('inf'))
                max_val = right_max_val
            return ValidAndMinMaxOfTree(True, min_val, max_val)

        return is_valid_bst_helper(root).is_valid
```

上からそのノードの範囲を限定していく方法。左の子は親より小さく、右の子は親より大きい、という条件がどのノードでも満たされていればvalidな二分探索木。
`min_val < node.val < max_val` は条件を反転させてearly returnぽく書いても良い (is_valid_bst_helper(node.left, ...)とかも同じだが)。

```py
class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        def is_valid_bst_helper(node: Optional[TreeNode], min_val: float, max_val: float) -> bool:
            if not node:
                return True
            return (
                min_val < node.val < max_val
                and is_valid_bst_helper(node.left, min_val, node.val)
                and is_valid_bst_helper(node.right, node.val, max_val)
            )

        return is_valid_bst_helper(root, -float('inf'), float('inf'))
```

- https://discord.com/channels/1084280443945353267/1192736784354918470/1230313494218412115
  - https://discord.com/channels/1084280443945353267/1192736784354918470/1235116690661179465

inorder traversalした結果が昇順に整列されているか見る別解。シンプルで分かりやすい。再帰でyieldする解法は中々思いつかないので訓練したい。

```py
class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        def inorder_sort(node: Optional[TreeNode]) -> Generator[TreeNode, None, None]:
            if not node:
                return
            if node.left:
                yield from inorder_sort(node.left)
            yield node
            if node.right:
                yield from inorder_sort(node.right)

        prev_value = -float('inf')
        for node in inorder_sort(root):
            if prev_value >= node.val:
                return False
            prev_value = node.val
        return True
```

- https://discord.com/channels/1084280443945353267/1196472827457589338/1236007436440305664
- https://discord.com/channels/1084280443945353267/1201211204547383386/1218927581605396572

2つ上の解答をスタックを使ったものに直す。

```py
class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        if not root:
            return True
        node_and_bounds_stack = [(root, -float('inf'), float('inf'))]
        while node_and_bounds_stack:
            node, lower_bound, upper_bound = node_and_bounds_stack.pop()
            if not (lower_bound < node.val < upper_bound):
                return False
            if node.left:
                node_and_bounds_stack.append((node.left, lower_bound, node.val))
            if node.right:
                node_and_bounds_stack.append((node.right, node.val, upper_bound))
        return True
```

## 3rd

### inorder traversalでスタックを使用

```py
class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        if not root:
            return True

        def append_current_and_left_nodes(nodes: list[TreeNode], node: TreeNode):
            while node:
                nodes.append(node)
                node = node.left

        nodes = []
        append_current_and_left_nodes(nodes, root)
        prev_value = None
        while nodes:
            node = nodes.pop()
            if prev_value is not None and prev_value >= node.val:
                return False
            prev_value = node.val
            append_current_and_left_nodes(nodes, node.right)
        return True
```

### inorderにソート

```py
class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        def inorder_sort(node: Optional[TreeNode]) -> Generator[TreeNode, None, None]:
            if not node:
                return
            if node.left:
                yield from inorder_sort(node.left)
            yield node
            if node.right:
                yield from inorder_sort(node.right)

        prev_value = None
        for node in inorder_sort(root):
            if prev_value is not None and prev_value >= node.val:
                return False
            prev_value = node.val
        return True
```

### 親のノードから値の範囲を狭める

```py
class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        def is_valid_bst_helper(node: Optional[TreeNode], min_val: int, max_val: int) -> bool:
            if not node:
                return True
            return (
                min_val < node.val < max_val
                and is_valid_bst_helper(node.left, min_val, node.val)
                and is_valid_bst_helper(node.right, node.val, max_val)
            )

        return is_valid_bst_helper(root, -inf, inf)
```

### 子から部分木の情報をもらう

```py
class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        def is_valid_bst_helper(node: TreeNode) -> tuple[bool, int, int]:
            min_val = node.val
            max_val = node.val
            if node.left:
                is_valid, left_min_val, left_max_val = is_valid_bst_helper(node.left)
                if not is_valid or left_max_val >= node.val:
                    return False, -inf, inf
                min_val = left_min_val
            if node.right:
                is_valid, right_min_val, right_max_val = is_valid_bst_helper(node.right)
                if not is_valid or right_min_val <= node.val:
                    return False, -inf, inf
                max_val = right_max_val
            return True, min_val, max_val

        if not root:
            return True
        return is_valid_bst_helper(root)[0]
```
