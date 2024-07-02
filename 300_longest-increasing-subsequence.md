# 300. Longest Increasing Subsequence

## 1st

### ①

解法を覚えていた。lisが何の略かコメントで書くくらいしても良いと思った。

所要時間: 2:53

n: len(nums)
- 時間計算量: O(nlogn)
- 空間計算量: O(n)

```py
class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        lis = []
        for num in nums:
            i = bisect_left(lis, num)
            if i == len(lis):
                lis.append(num)
            else:
                lis[i] = num
        return len(lis)
```

### ②

後ろから、 `nums[i]` が左端になるLISの長さを確定させていき最大を求めるやり方。LISElementというクラス名はもう少し何とかしたいが良いものが思いつかなかった。

ループ内でスライスを取るのに抵抗があったが回避策を思いつかなかった。

所要時間: 13:59

n: len(nums)
- 時間計算量: O(n^2)
- 空間計算量: O(n)

```py
from dataclasses import dataclass


@dataclass
class LISElement:
    index: int
    value: int
    lis_length: int # length of lis starting `index` in the list


class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        # initialize lis_length to -1
        lis_elements = [LISElement(i, num, -1) for i, num in enumerate(nums)]
        for lis_element in reversed(lis_elements):
            # extract candidates of lis_element's successor
            successor_candidates = list(filter(
                lambda e: e.value > lis_element.value,
                lis_elements[lis_element.index + 1:]
            ))
            if not successor_candidates:
                lis_element.lis_length = 1
                continue
            successor = max(successor_candidates, key=lambda e: e.lis_length)
            lis_element.lis_length = successor.lis_length + 1
        if not lis_elements:
            return 0
        return max(lis_elements, key=lambda e: e.lis_length).lis_length
```

### ③

②を書いた後、もっと素直な書き方があるなと思い実装し直したバージョン。先にこれが出てきて欲しい感。
`lis_lengths[i]` に何が入っているかはコメントで書くべきだった気がする。 `nums[i]` がLISの左端になるときのLISの長さが入る。

所要時間: (未計測)

n: len(nums)
- 時間計算量: O(n^2)
- 空間計算量: O(n)

```py
class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        if not nums:
            return 0
        lis_lengths = [1] * len(nums)
        for i in reversed(range(len(nums))):
            for j in range(i+1, len(nums)):
                if nums[i] >= nums[j]:
                    continue
                lis_lengths[i] = max(lis_lengths[i], lis_lengths[j] + 1)
        return max(lis_lengths)
```

### ④

BITを用いた解法。アルゴリズムを思い出しながら書いた。座標圧縮しない方法もあるが入力の最大値に依存するので少し抵抗がある。


所要時間: 20:28

n: len(unique(nums))
- 時間計算量: O(nlogn)
- 空間計算量: O(n)

```py
class MaxBIT:
    def __init__(self, size: int):
        self.arr = [0] * (size + 1)

    def get(self, i: int) -> int:
        result = 0
        while i > 0:
            result = max(result, self.arr[i])
            i -= i & (-i)
        return result

    def update(self, i: int, value: int):
        while i < len(self.arr):
            self.arr[i] = max(self.arr[i], value)
            i += i & (-i)


class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        def compress(nums: list[int]) -> tuple[list[int], int]:
            sorted_nums = sorted(set(nums))
            compressed = []
            for num in nums:
                i = bisect_left(sorted_nums, num)
                compressed.append(i + 1) # 1-index
            return compressed, len(sorted_nums)

        compressed, max_compressed_value = compress(nums)
        bit = MaxBIT(max_compressed_value)
        for num in compressed:
            length = bit.get(num - 1)
            bit.update(num, length + 1)
        return bit.get(max_compressed_value)
```


### ⑤

セグメント木を用いた解法。(アルゴリズムの理解が曖昧なため) 分岐で等号を付けておらずバグ取りに時間がかかった。BITと同じく座標圧縮している。

所要時間: 40:56

n: len(unique(nums))
- 時間計算量: O(nlogn)
- 空間計算量: O(n)

```py
class SegmentTree:
    def __init__(self, right_most: int):
        self.right_most = right_most
        num_leaves = 1
        while num_leaves <= right_most:
            num_leaves *= 2
        self.values = [0] * (2 * num_leaves - 1) # the size of binary tree

    def query(self, qleft: int, qright: int) -> int:
        assert qleft <= qright, f"qleft must be lower than or equal to qright. (qleft, qright): ({qleft}, {qright})"
        return self._query(0, self.right_most, 0, qleft, qright)

    def update(self, target: int, value: int):
        return self._update(0, self.right_most, 0, target, value)

    def _query(self, left: int, right: int, index: int, qleft: int, qright: int) -> int:
        if qright < left or right < qleft:
            return 0
        if qleft <= left and right <= qright:
            return self.values[index]
        middle = (left + right) // 2
        left_value = self._query(left, middle, 2 * index + 1, qleft, qright)
        right_value = self._query(middle + 1, right, 2 * index + 2, qleft, qright)
        return max(left_value, right_value)

    def _update(self, left: int, right: int, index: int, target: int, value: int):
        if left == right:
            # binary search's base case
            # left == right == target
            self.values[index] = max(self.values[index], value)
            return
        middle = (left + right) // 2
        if target <= middle:
            self._update(left, middle, 2 * index + 1, target, value)
        else:
            self._update(middle + 1, right, 2 * index + 2, target, value)
        self.values[index] = max(self.values[2 * index + 1], self.values[2 * index + 2])


class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        if not nums:
            return 0

        def compress(nums: list[int]) -> tuple[list[int], int]:
            assert len(nums) > 0
            sorted_unique_nums = sorted(set(nums))
            compressed = []
            for num in nums:
                i = bisect_left(sorted_unique_nums, num)
                compressed.append(i + 1) # 1-index
            return compressed, len(sorted_unique_nums)

        compressed, max_compressed_value = compress(nums)
        st = SegmentTree(max_compressed_value)
        for num in compressed:
            length = st.query(0, num - 1)
            st.update(num, length + 1)
        return st.query(0, max_compressed_value)
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1225849404037009609/1249410700741443750
- https://discord.com/channels/1084280443945353267/1233295449985650688/1245022196712345611


③は前から更新した方がより分かりやすいかもしれない。

```py
class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        if not nums:
            return 0
        lis_lengths = [1] * len(nums) # lis_lengths[i]: length of lis starting from the i-th element
        for i in range(len(nums)):
            for j in range(i):
                if nums[j] < nums[i]:
                    lis_lengths[i] = max(lis_lengths[i], lis_lengths[j] + 1)
        return max(lis_lengths)
```

- https://discord.com/channels/1084280443945353267/1227073733844406343/1242496716188553357
- https://discord.com/channels/1084280443945353267/1228700203327164487/1242148168062206085
- https://discord.com/channels/1084280443945353267/1233603535862628432/1238686546698309664
- https://discord.com/channels/1084280443945353267/1192736784354918470/1227975138419671160

bisectモジュールを使わず自前で実装している解答が多いのでやる。

```py
class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        def lower_bound(list_: list[int], value: int) -> int:
            left = 0
            right = len(list_)
            while left < right:
                middle = (left + right) // 2
                if value <= list_[middle]:
                    right = middle
                else:
                    left = middle + 1
            return left

        lis = []
        for num in nums:
            i = lower_bound(lis, num)
            if i == len(lis):
                lis.append(num)
            else:
                lis[i] = num
        return len(lis)
```

## 3rd

```py
class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        lis = [] # longest increasing subsequence
        for num in nums:
            i = bisect_left(lis, num)
            if i == len(lis):
                lis.append(num)
            else:
                lis[i] = num
        return len(lis)
```
