# 108. Convert Sorted Array to Binary Search Tree

## 1st

### ①

再帰の深さがlogオーダー (定数倍も問題ない) なので問題ないと思い再帰で実装した。

所要時間: 5:33

n: len(nums)
- 時間計算量: O(n)
- 空間計算量: O(log(n))

```py
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def sortedArrayToBST(self, nums: List[int]) -> Optional[TreeNode]:
        def to_bst(begin: int, end: int) -> Optional[TreeNode]:
            if begin == end:
                return None
            mid = (begin + end) // 2
            node = TreeNode(nums[mid])
            node.left = to_bst(begin, mid)
            node.right = to_bst(mid + 1, end)
            return node

        return to_bst(0, len(nums))
```

### ②

スタックに直した版。node_and_ranges_stackにappendする値を間違えており (right側で(node.right, mid, end)を積むようにしていた)、図を書くまで何が問題か分からず時間がかかった。
こういう勘違いはこのアルゴリズムで何をしているかきちんと分かっており言語化できればあまり起こらないと思うので、理解不足なのだろう。

node_and_ranges_stackはもう少しいい名前があるかも。Pythonでは問題ないがendはちょっとキーワード感あるかもしれない？

所要時間: 16:50

n: len(nums)
- 時間計算量: O(n)
- 空間計算量: O(log(n))

```py
class Solution:
    def sortedArrayToBST(self, nums: List[int]) -> Optional[TreeNode]:
        if not nums:
            return None
        root = TreeNode()
        node_and_ranges_stack = [(root, 0, len(nums))]
        while node_and_ranges_stack:
            node, begin, end = node_and_ranges_stack.pop()
            mid = (begin + end) // 2
            node.val = nums[mid]
            if begin < mid:
                node.left = TreeNode()
                node_and_ranges_stack.append((node.left, begin, mid))
            if mid + 1 < end:
               node.right = TreeNode()
               node_and_ranges_stack.append((node.right, mid + 1, end))
        return root
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1196472827457589338/1238907849422274622
- https://discord.com/channels/1084280443945353267/1200089668901937312/1237995792371945573
- https://discord.com/channels/1084280443945353267/1227073733844406343/1237649500630421527
- https://discord.com/channels/1084280443945353267/1201211204547383386/1217525976691507221

parent, left, rightをstackから取るならこんな感じになるだろうか。
平衡二分探索木を作っていることがちょっと分かりづらいかもしれない。
parent_with_range_stackはいい名前が思いつかない。

```py
class Solution:
    def sortedArrayToBST(self, nums: List[int]) -> Optional[TreeNode]:
        if not nums:
            return None
        sentinel = TreeNode()
        parent_with_range_stack = [(sentinel, 0, len(nums), True)]
        while parent_with_range_stack:
            parent, begin, end, is_left = parent_with_range_stack.pop()
            if begin == end:
                continue
            mid = (begin + end) // 2
            node = TreeNode(nums[mid])
            if is_left:
                parent.left = node
            else:
                parent.right = node
            parent_with_range_stack.append((node, begin, mid, True))
            parent_with_range_stack.append((node, mid + 1, end, False))
        return sentinel.left
```


行き帰りで異なる処理をするようにstackで実装した版。
これをleft, rightへの参照を行きのときに作ってstackに積むようにすれば②のようにできる。

```py
from dataclasses import dataclass

class Solution:
    def sortedArrayToBST(self, nums: List[int]) -> Optional[TreeNode]:
        root_ref = Box(None)
        stack = [('go', root_ref, Box(None), Box(None), 0, len(nums))]
        while stack:
            go_back, node_ref, left_node_ref, right_node_ref, begin, end = stack.pop()
            if go_back == 'go':
                if begin == end:
                    continue
                mid = (begin + end) // 2
                node_ref.value = TreeNode(nums[mid])
                stack.append(('back', node_ref, left_node_ref, right_node_ref, begin, end))
                stack.append(('go', left_node_ref, Box(None), Box(None), begin, mid))
                stack.append(('go', right_node_ref, Box(None), Box(None), mid + 1, end))
            node_ref.value.left = left_node_ref.value
            node_ref.value.right = right_node_ref.value
        return root_ref.value


@dataclass
class Box:
    value: Optional[TreeNode]
```


## 3rd

この問題を解けと言われたら再帰で実装するだろうが、今回は時間がかかったスタックの方を練習した。

```py
class Solution:
    def sortedArrayToBST(self, nums: List[int]) -> Optional[TreeNode]:
        if not nums:
            return None
        root = TreeNode()
        node_with_range_stack = [(root, 0, len(nums))]
        while node_with_range_stack:
            node, begin, end = node_with_range_stack.pop()
            mid = (begin + end) // 2
            node.val = nums[mid]
            if begin < mid:
                node.left = TreeNode()
                node_with_range_stack.append((node.left, begin, mid))
            if mid + 1 < end:
                node.right = TreeNode()
                node_with_range_stack.append((node.right, mid + 1, end))
        return root
```
