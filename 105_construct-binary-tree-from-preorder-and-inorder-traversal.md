# 105. Construct Binary Tree from Preorder and Inorder Traversal

## 1st

preorder内での部分木の開始位置、inorder内での部分木の開始位置、部分木のサイズの3つ組を渡してノードを作っていくようにした。

subrootと書いてるが普通にnodeと書けば良かった。
inorderでのsubrootのindexを求めるところは、先にkeyがノードのval, valueがinorderでのindexである辞書を作っておけばO(1)でlookupできるが、ノードの数が高々3Kなので二分探索で十分速いだろうと思いそのままにしている。

assertはin_root_indexとright_lengthについて書いたが、left_lengthとright_lengthの2つについて書いたほうがわかりやすそう。

lengthは木のサイズを表すのでsizeという変数名の方がいいか。

所要時間: 15:47

n: ノード数, h: 木の高さ
- 時間計算量: O(nlogn)
- 空間計算量: O(h)

```py
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        def build_tree_helper(pre_start: int, in_start: int, length: int) -> Optional[TreeNode]:
            if length == 0:
                return None
            subroot = TreeNode(preorder[pre_start])
            in_root_index = inorder.index(subroot.val)
            assert in_start <= in_root_index <= in_start + length
            left_length = in_root_index - in_start
            right_length = length - left_length - 1
            assert right_length >= 0
            subroot.left = build_tree_helper(pre_start + 1, in_start, left_length)
            subroot.right = build_tree_helper(pre_start + left_length + 1, in_root_index + 1, right_length)
            return subroot

        return build_tree_helper(0, 0, len(preorder))
```

↓気になったところを書き直した。上のコードもだが、これも一発で書けなかったので微妙なところがあるかもしれない。

```py
class Solution:
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        def build(pre_start: int, in_start: int, size: int) -> Optional[TreeNode]:
            if size == 0:
                return None
            node = TreeNode(preorder[pre_start])
            in_node_index = inorder.index(node.val)
            left_tree_size = in_node_index - in_start
            right_tree_size = size - left_tree_size - 1
            assert left_tree_size >= 0
            assert right_tree_size >= 0
            node.left = build(pre_start + 1, in_start, left_tree_size)
            node.right = build(pre_start + left_tree_size + 1, in_node_index + 1, right_tree_size)
            return node

        return build(0, 0, len(preorder))
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1192736784354918470/1247343552262963240

preorderの順にノードを見ていき、木の根から順にノードの場所をinorderの位置を見ながら確定していく。

- preorderの順で i < j => iのノードの位置はjのノードと同じかそれより上
- inorderの順で i < j => iのノードの位置はjより左側

という性質を使っている。上の性質がわかれば結構素直に書けていいなと思った。

```py
class Solution:
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        value_to_inorder_index = {}
        for i, value in enumerate(inorder):
            value_to_inorder_index[value] = i
        if not preorder:
            return None
        root = TreeNode(preorder[0])
        for value in preorder[1:]:
            node = root
            inorder_index = value_to_inorder_index[value]
            while True:
                assert inorder_index != value_to_inorder_index[node.val]
                if inorder_index < value_to_inorder_index[node.val]:
                    if not node.left:
                        node.left = TreeNode(value)
                        break
                    node = node.left
                else:
                    if not node.right:
                        node.right = TreeNode(value)
                        break
                    node = node.right
        return root
```

1stのスタック版も書いた。

```py
class Solution:
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        if not preorder:
            return None
        value_to_inorder_index = {}
        for i, value in enumerate(inorder):
            value_to_inorder_index[value] = i
        root = TreeNode(preorder[0])
        tree_info_stack = [(root, 0, 0, len(preorder))] # node, preorder_start, inorder_start, tree_size
        while tree_info_stack:
            node, pre_start, in_start, size = tree_info_stack.pop()
            in_node_index = value_to_inorder_index[node.val]
            left_tree_size = in_node_index - in_start
            right_tree_size = size - left_tree_size - 1
            if left_tree_size:
                pre_left_node_index = pre_start + 1
                node.left = TreeNode(preorder[pre_left_node_index])
                tree_info_stack.append((node.left, pre_left_node_index, in_start, left_tree_size))
            if right_tree_size:
                pre_right_node_index = pre_start + left_tree_size + 1
                node.right = TreeNode(preorder[pre_right_node_index])
                tree_info_stack.append((node.right, pre_right_node_index, in_node_index + 1, right_tree_size))
        return root
```

- https://discord.com/channels/1084280443945353267/1227073733844406343/1239602171952234629
- https://discord.com/channels/1084280443945353267/1196472827457589338/1239554028120051783


## 3rd

```py
class Solution:
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        def build(pre_start: int, in_start: int, size: int) -> Optional[TreeNode]:
            if not size:
                return None
            root = TreeNode(preorder[pre_start])
            in_root_index = inorder.index(root.val)
            left_tree_size = in_root_index - in_start
            right_tree_size = size - left_tree_size - 1
            assert left_tree_size >= 0
            assert right_tree_size >= 0
            root.left = build(pre_start + 1, in_start, left_tree_size)
            root.right = build(pre_start + left_tree_size + 1, in_root_index + 1, right_tree_size)
            return root

        return build(0, 0, len(preorder))
```
