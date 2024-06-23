# 617. Merge Two Binary Trees

## 1st

### ①

再帰で実装するのが一番最初に思いついたのでそれで書いた。
> The number of nodes in both trees is in the range [0, 2000].

なのでデフォルトの設定 (sys.getrecursionlimit()が 1000) では再帰の上限に達しうることは注意する。

与えられた引数のnode1, node2をそのまま返すので、返した値をいじると元の木にも影響を与えるようになっていることは注意する。

所要時間: 5:48

n: 2つの木でoverlapしているノードの数, h: 根から2つの木でoverlapしている部分までの高さ
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
    def mergeTrees(self, node1: Optional[TreeNode], node2: Optional[TreeNode]) -> Optional[TreeNode]:
        if not node1 and not node2:
            return None
        if not node1:
            return node2
        if not node2:
            return node1
        node = TreeNode(node1.val + node2.val)
        node.left = self.mergeTrees(node1.left, node2.left)
        node.right = self.mergeTrees(node1.right, node2.right)
        return node
```

### ②

①をstackに直した。時間がかかっている。

所要時間: 18:04

n: 2つの木でoverlapしているノードの数, h: 根から2つの木でoverlapしている部分までの高さ
- 時間計算量: O(n)
- 空間計算量: O(h)


```py
class Solution:
    def mergeTrees(self, root1: Optional[TreeNode], root2: Optional[TreeNode]) -> Optional[TreeNode]:
        def merge(node1: Optional[TreeNode], node2: Optional[TreeNode]) -> Optional[TreeNode]:
            if not node1 and not node2:
                return None
            if not node1:
                return node2
            if not node2:
                return node1
            return TreeNode(node1.val + node2.val)

        merged_root = merge(root1, root2)
        if not root1 or not root2:
            return merged_root
        nodes = [(root1, root2, merged_root)]
        while nodes:
            node1, node2, merged_node = nodes.pop()
            merged_node.left = merge(node1.left, node2.left)
            if node1.left and node2.left:
                nodes.append((node1.left, node2.left, merged_node.left))
            merged_node.right = merge(node1.right, node2.right)
            if node1.right and node2.right:
                nodes.append((node1.right, node2.right, merged_node.right))
        return merged_root
```

### ③

所要時間: 1:59

n: 2つの木の木のノード数の総和
- 時間計算量: O(n)
- 空間計算量: O(n) (deepcopyで再帰するため)


```py
class Solution:
    def mergeTrees(self, node1: Optional[TreeNode], node2: Optional[TreeNode]) -> Optional[TreeNode]:
        if not node1 and not node2:
            return None
        if not node1:
            return deepcopy(node2)
        if not node2:
            return deepcopy(node1)
        node = TreeNode(node1.val + node2.val)
        node.left = self.mergeTrees(node1.left, node2.left)
        node.right = self.mergeTrees(node1.right, node2.right)
        return node
```

deepcopyについて調べた。

- [copy — Shallow and deep copy operations — Python 3.12.4 documentation](https://docs.python.org/3/library/copy.html)
- コード: https://github.com/python/cpython/blob/main/Lib/copy.py#L118

  一応コードを読んだがあまり理解できていない...
  - `_deepcopy_dispatch` はlistやtupleなどの特定の型に対するコピー用の関数を登録している？
  - `_deepcopy_dispatch = d = {}` とわざわざdという変数でも同じ辞書を指すようにし、あとで `del d` と消しているがこうする理由が思いつかなかった (読みにくくなるだけではないのだろうか)
  - 再帰する際にメモ化することで、無限ループに陥るのを防いでいるっぽい。
    > keeping a memo dictionary of objects already copied during the current copying pass
  - コードの中に出てくる `reductor` がいまいち分からない。pickleモジュールでオブジェクトをシリアライズ / デシリアライズするときに使われ、コピー後のオブジェクトを構築するために使われるようだが...



## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1227073733844406343/1237408037241749544
  root1かroot2、あるいは両方がNoneのときの処理を `if not root1 or not root2: return root1 or root2` と短く書けるのは参考になった。ただパッと見て分かるかというと微妙かなと思うので、こうも書けることは頭に置きつつ冗長目に書こうかと思う
- https://discord.com/channels/1084280443945353267/1228700203327164487/1229107625325760543
- https://discord.com/channels/1084280443945353267/1201211204547383386/1217139317554679949
- https://discord.com/channels/1084280443945353267/1200089668901937312/1204041603468103740

省略

## 3rd


```py
class Solution:
    def mergeTrees(self, root1: Optional[TreeNode], root2: Optional[TreeNode]) -> Optional[TreeNode]:
        def merge(node1: Optional[TreeNode], node2: Optional[TreeNode]) -> Optional[TreeNode]:
            if not node1 and not node2:
                return None
            if not node1:
                return node2 # 新しくオブジェクトを作って返したい場合はdeepcopyする
            if not node2:
                return node1 # 新しくオブジェクトを作って返したい場合はdeepcopyする
            node = TreeNode(node1.val + node2.val)
            return node

        merged_root = merge(root1, root2)
        if not root1 or not root2:
            return merged_root
        nodes = [(root1, root2, merged_root)]
        while nodes:
            node1, node2, merged_node = nodes.pop()
            assert node1 and node2
            merged_node.left = merge(node1.left, node2.left)
            if node1.left and node2.left:
                nodes.append((node1.left, node2.left, merged_node.left))
            merged_node.right = merge(node1.right, node2.right)
            if node1.right and node2.right:
                nodes.append((node1.right, node2.right, merged_node.right))
        return merged_root
```
