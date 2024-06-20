# 111. Minimum Depth of Binary Tree

## 1st

### ①

素朴に書いた。ただエッジの重みが等しいグラフ上の最短距離だから幅優先探索を思いつくのが自然な気がする (https://github.com/fhiyo/leetcode/pull/22 でも同じ状況ですぐに思いつかなかった)。

min_depth の初期値はどうするべきか迷う。整数が入る変数に浮動小数点数を混ぜるのに少し抵抗もあるが、有限の任意の値より大きいというminの初期値としての利点があるので実利を取った。 `sys.maxsize` もあるが、Pythonにおいては (大抵の場合十分だろうが) 適当に大きな数以上のものではない気がする。

細かいけど、 `nodes_with_depth` か `node_with_depths` かで迷った。「nodeとdepth」のコレクションだからと思い後者で書いたけど、前者の方が普通だろうか？ あとは `node_depth_pairs` ？

枝刈りは実装したときは思いつかなかった (これくらいは頑張りたいところな気がする)。depthがmax_depth以上ならそれ以上探索する必要はない。

所要時間: 6:15

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
    def minDepth(self, root: Optional[TreeNode]) -> int:
        if root is None:
            return 0
        min_depth = float('inf')
        node_with_depths = [(root, 1)]
        while node_with_depths:
            node, depth = node_with_depths.pop()
            is_leaf = True
            if node.left is not None:
                is_leaf = False
                node_with_depths.append((node.left, depth + 1))
            if node.right is not None:
                is_leaf = False
                node_with_depths.append((node.right, depth + 1))
            if is_leaf:
                min_depth = min(min_depth, depth)
        return min_depth
```

### ②

幅優先探索で実装。今回はqueueを一つ使う方法で実装した。
`is_leaf()`, `append_if_exists()` と関数を作ったが、①のように `is_leaf` 変数を取り回す方が比較の回数は一応少ない。読みやすさも大差ないと思う。

`append_if_exists(nodes, root)` については、書いてみたが `if root is None: return 0` の後に `nodes = deque([root])` とする方が好みかも。

最後の行には到達しない。コメントを書いて済ませたが `assert False` の方が分かりやすいかな？

所要時間: 10:56

n: ノード数
- 時間計算量: O(n)
- 空間計算量: O(n)

```py
class Solution:
    def minDepth(self, root: Optional[TreeNode]) -> int:
        def append_if_exists(nodes: deque[TreeNode], node: TreeNode):
            if node is None:
                return
            nodes.append(node)

        def is_leaf(node: TreeNode) -> bool:
            return node.left is None and node.right is None

        nodes = deque()
        append_if_exists(nodes, root)
        depth = 0
        while nodes:
            depth += 1
            num_current_depth_nodes = len(nodes)
            for _ in range(num_current_depth_nodes):
                node = nodes.popleft()
                if is_leaf(node):
                    return depth
                append_if_exists(nodes, node.left)
                append_if_exists(nodes, node.right)
        return 0 # when root is None
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1196472827457589338/1237821650527981608

  こちらの実装を見て、自分の `node.left is None and node.right is None` はちょっと長いので、 `not node.left and not node.right` の方がいいかなと思った

- https://discord.com/channels/1084280443945353267/1227073733844406343/1236949934545047583
  - https://github.com/sakupan102/arai60-practice/pull/23#discussion_r1590804846

  whileのすぐ後に値をインクリメントするのが違和感があるというコメント (多分)。まあ分かるかも。

- https://discord.com/channels/1084280443945353267/1201211204547383386/1216397657271046436
- https://discord.com/channels/1084280443945353267/1183683738635346001/1202680261108826184

幅優先を書き直した。下は `append_if_exists()` 使って無駄な比較が起こらないように書いてみたが、素直じゃない感じがある。boolを返すこと自体はC++の [map::insert](https://en.cppreference.com/w/cpp/container/map/insert) 的な感じで自分的には許容範囲。

```py
class Solution:
    def minDepth(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0
        nodes = [root]
        depth = 1
        while nodes:
            next_nodes = []
            for node in nodes:
                is_leaf = True
                if node.left:
                    is_leaf = False
                    next_nodes.append(node.left)
                if node.right:
                    is_leaf = False
                    next_nodes.append(node.right)
                if is_leaf:
                    return depth
            nodes = next_nodes
            depth += 1
        assert False # never reached
```

```py
class Solution:
    def minDepth(self, root: Optional[TreeNode]) -> int:
        def append_if_exists(nodes: list[TreeNode], node: TreeNode) -> bool:
            if not node:
                return False
            nodes.append(node)
            return True

        if not root:
            return 0
        depth = 1
        nodes = [root]
        while nodes:
            next_nodes = []
            for node in nodes:
                is_internal = False
                is_internal |= append_if_exists(next_nodes, node.left)
                is_internal |= append_if_exists(next_nodes, node.right)
                if not is_internal:
                    return depth
            nodes = next_nodes
            depth += 1
        assert False # never reached
```

## 3rd


```py
class Solution:
    def minDepth(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0
        depth = 1
        nodes = [root]
        while nodes:
            next_nodes = []
            for node in nodes:
                is_leaf = True
                if node.left:
                    is_leaf = False
                    next_nodes.append(node.left)
                if node.right:
                    is_leaf = False
                    next_nodes.append(node.right)
                if is_leaf:
                    return depth
            nodes = next_nodes
            depth += 1
        assert False # never reached
```
