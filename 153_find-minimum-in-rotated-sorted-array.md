# 153. Find Minimum in Rotated Sorted Array

## 1st

探索範囲の中央の値Aと右端の値Bを比べて、 `A<=B` ならAかAより左側に最小値があり、 `A>B` ならAより右側に最小値がある。よって二分探索で答えが見つかる。

所要時間: 8:06

n: len(nums)
- 時間計算量: O(log n)
- 空間計算量: O(n)

```py
class Solution:
    def findMin(self, nums: List[int]) -> int:
        assert len(nums) > 0
        low = 0
        high = len(nums) - 1
        while low < high:
            mid = (low + high) // 2
            if nums[mid] <= nums[high]:
                high = mid
            else:
                low = mid + 1
        return nums[low]
```


## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1227073733844406343/1253995913114943529
  - https://github.com/sakupan102/arai60-practice/pull/43

> 右端より大きい値で一番右側にあるものを探すと考えることもできる

なるほど。右端以下か？をクエリにすると `[False, False, ..., True, True, ..., True]` のようなnumsに対応したリストが得られ、これのTrueになる左端を二分探索したらその位置は最小値である。この考え方は明快で良い。

```py
class Solution:
    def findMin(self, nums: List[int]) -> int:
        assert len(nums) > 0
        index = bisect_left(nums, True, key=lambda x: x <= nums[-1])
        return nums[index]
```

これを自前で書くとこうなるはず。

```py
class Solution:
    def findMin(self, nums: List[int]) -> int:
        assert len(nums) > 0
        low = 0
        high = len(nums)
        while low < high:
            mid = (low + high) // 2
            if nums[mid] <= nums[-1]:
                high = mid
            else:
                low = mid + 1
        return nums[low]
```

- https://discord.com/channels/1084280443945353267/1239148130679783424/1254344839546273793
  - https://github.com/Kitaken0107/arai60/pull/1
- https://discord.com/channels/1084280443945353267/1233603535862628432/1248982661138219019
  - https://github.com/goto-untrapped/Arai60/pull/24
- https://discord.com/channels/1084280443945353267/1233295449985650688/1240217729718292530
  - https://github.com/Exzrgs/LeetCode/pull/11

lowが `x>nums[-1]` 側、highが `x<=nums[-1]` 側として境界を求めるやり方。
`mid = (low + high + 1) // 2` の+1は割り切れないときにhigh側に寄せるための調整。
(mid = (low + high) // 2 でも動くが、low = -1, high = 0のときにmidが-1になり気持ち悪い。Pythonだからたまたま配列外参照にならず動く感じになる)


```py
class Solution:
    def findMin(self, nums: List[int]) -> int:
        assert len(nums) > 0
        low = -1
        high = len(nums) - 1
        while high - low > 1:
            mid = (low + high + 1) // 2
            if nums[mid] > nums[-1]:
                low = mid
            else:
                high = mid
        return nums[high]
```

- https://discord.com/channels/1084280443945353267/1192736784354918470/1235593875662307369
  - https://github.com/YukiMichishita/LeetCode/pull/9

再帰で実装するパターンがあり、なるほどとなった (単純にループを再帰に直すだけではあるのだが)。

```py
class Solution:
    def findMin(self, nums: List[int]) -> int:
        assert len(nums) > 0

        def find_min_helper(low: int, high: int) -> int:
            if low == high:
                return nums[low]
            mid = (low + high) // 2
            if nums[mid] <= nums[-1]:
                return find_min_helper(low, mid)
            return find_min_helper(mid + 1, high)

        return find_min_helper(0, len(nums) - 1)
```

- https://discord.com/channels/1084280443945353267/1230079550923341835/1233783130540867584
  - https://github.com/thonda28/leetcode/pull/7
- https://discord.com/channels/1084280443945353267/1201211204547383386/1225793820692840509
  - https://github.com/shining-ai/leetcode/pull/42
- https://discord.com/channels/1084280443945353267/1200089668901937312/1215610986774536284
  - https://github.com/hayashi-ay/leetcode/pull/45

右端とではなく、左端との比較とすることもできる。自分は最初できないんじゃないかと思ったが、ソート済みの状態を別で考えれば `x < nums[0]` をクエリにすれば `[False, ..., True, ...]` となるようにできる。

```py
class Solution:
    def findMin(self, nums: List[int]) -> int:
        assert len(nums) > 0
        if len(nums) == 1 or nums[0] < nums[-1]:
            return nums[0]
        low = 0
        high = len(nums)
        while low < high:
            mid = (low + high) // 2
            if nums[mid] < nums[0]:
                high = mid
            else:
                low = mid + 1
        return nums[low]
```


## 3rd

```py
class Solution:
    def findMin(self, nums: List[int]) -> int:
        assert len(nums) > 0
        low = 0
        high = len(nums) - 1
        while low < high:
            mid = (low + high) // 2
            if nums[mid] <= nums[-1]:
                high = mid
            else:
                low = mid + 1
        return nums[low]
```
