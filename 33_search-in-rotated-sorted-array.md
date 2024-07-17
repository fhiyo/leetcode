# 33. Search in Rotated Sorted Array

## 1st

### ①

一つ前の問題でrotateされたソート済みリストの最小値を見つける問題があるので、それを応用して解く。最小値の位置がrotate前の位置からどのくらいずれているかのoffsetを計算して、ズレを修正しながら二分探索すればいい。

get_original_index内でlen(nums)で割り算しており呼び出す側も考慮していないので、 len(nums)は0だとZeroDivisionErrorを吐きそうなコードであることに後から気づいた。 (get_offset()内で別目的でassertしてるので問題はないのだが)

所要時間: 14:58

n: len(nums)
- 時間計算量: O(log n)
- 空間計算量: O(n)

```py
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        def get_offset(nums: list[int]) -> int:
            assert len(nums) > 0
            low = 0
            high = len(nums) - 1
            while low < high:
                mid = (low + high) // 2
                if nums[mid] <= nums[-1]:
                    high = mid
                else:
                    low = mid + 1
            return low

        def get_original_index(index: int, offset: int) -> int:
            return (index + offset) % len(nums)

        offset = get_offset(nums)
        low = 0
        high = len(nums)
        while low < high:
            mid = (low + high) // 2
            if nums[get_original_index(mid, offset)] >= target:
                high = mid
            else:
                low = mid + 1
        index = get_original_index(low, offset)
        if low == len(nums) or nums[index] != target:
            return -1
        return index
```

### ②

最小値より左と右でリストを分割すればその中では整列しているので、二分探索を二回することでtargetを効率的に発見できる。

search_helperではlow, highのindexを引数に取っているが、ここでnumsをコピーするとそこでO(n)かかってしまい律速になる (C実装なので速いは速いが)。

所要時間: 10:18

n: len(nums)
- 時間計算量: O(log n)
- 空間計算量: O(n)

```py
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        def find_minimum_numbers_position(nums: list[int]) -> int:
            assert len(nums) > 0
            low = 0
            high = len(nums)
            while low < high:
                mid = (low + high) // 2
                if nums[mid] <= nums[-1]:
                    high = mid
                else:
                    low = mid + 1
            return low

        def search_helper(low: int, high: int) -> int:
            upper = high
            while low < high:
                mid = (low + high) // 2
                if nums[mid] < target:
                    low = mid + 1
                else:
                    high = mid
            if low == upper or nums[low] != target:
                return -1
            return low

        partition = find_minimum_numbers_position(nums)
        target_index1 = search_helper(0, partition)
        if target_index1 != -1:
            return target_index1
        target_index2 = search_helper(partition, len(nums))
        if target_index2 != -1:
            return target_index2
        return -1
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1233295449985650688/1239440783824781362
  - https://github.com/Exzrgs/LeetCode/pull/8

一気に二分探索をするでも解けるようだ...一応書きはしたが難しい。 `nums[mid] == target` の条件がないと書けなかった。

```py
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        low = 0
        high = len(nums) - 1
        while low <= high:
            mid = (low + high) // 2
            if nums[mid] == target:
                return mid
            if nums[low] <= nums[mid]:
                if nums[low] <= target < nums[mid]:
                    high = mid - 1
                else:
                    low = mid + 1
            else:
                if nums[mid] < target <= nums[high]:
                    low = mid + 1
                else:
                    high = mid - 1
        return -1
```

- https://discord.com/channels/1084280443945353267/1227073733844406343/1254039251243237438
  - https://github.com/sakupan102/arai60-practice/pull/44
- https://discord.com/channels/1084280443945353267/1235971495696662578/1240447377303928873
  - https://github.com/sakzk/leetcode/pull/7
- https://discord.com/channels/1084280443945353267/1233603535862628432/1236137930913742889
  - https://github.com/goto-untrapped/Arai60/pull/13
- https://discord.com/channels/1084280443945353267/1225849404037009609/1233470868878004245
  - https://github.com/SuperHotDogCat/coding-interview/pull/10

bisect_leftを使って解く。

```py
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        def get_offset(nums: list[int]) -> int:
            assert len(nums) > 0
            low = 0
            high = len(nums) - 1
            while low < high:
                mid = (low + high) // 2
                if nums[mid] <= nums[-1]:
                    high = mid
                else:
                    low = mid + 1
            return low

        offset = get_offset(nums)
        if nums[offset] <= target <= nums[-1]:
            low = offset
            high = len(nums)
        else:
            low = 0
            high = offset
        index = bisect_left(nums, target, low, high)
        if index == len(nums) or nums[index] != target:
            return -1
        return index
```

- https://discord.com/channels/1084280443945353267/1201211204547383386/1226448175653715989
  - https://github.com/shining-ai/leetcode/pull/43
- https://discord.com/channels/1084280443945353267/1200089668901937312/1218494932369670244
  - https://github.com/hayashi-ay/leetcode/pull/49



## 3rd


```py
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        if not nums:
            return -1

        def get_offset(nums: list[int]) -> int:
            low = 0
            high = len(nums)
            while low < high:
                mid = (low + high) // 2
                if nums[mid] <= nums[-1]:
                    high = mid
                else:
                    low = mid + 1
            return low

        def get_original_index(index: int, offset: int) -> int:
            return (index + offset) % len(nums)

        offset = get_offset(nums)
        low = 0
        high = len(nums)
        while low < high:
            mid = (low + high) // 2
            if nums[get_original_index(mid, offset)] >= target:
                high = mid
            else:
                low = mid + 1
        index = get_original_index(low, offset)
        if low == len(nums) or nums[index] != target:
            return -1
        return index
```
