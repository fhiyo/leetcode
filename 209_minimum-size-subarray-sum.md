# 209. Minimum Size Subarray Sum

## 1st

### ①

右端をループで回しながら、subarray_sum < target となるまで左端を進める、を繰り返す。

所要時間: 8:15

n: len(nums)
- 時間計算量: O(n)
- 空間計算量: O(n)

```py
class Solution:
    def minSubArrayLen(self, target: int, nums: List[int]) -> int:
        assert target > 0
        left = -1 # exclusive
        subarray_sum = 0
        min_length = len(nums) + 1
        for right in range(len(nums)):
            subarray_sum += nums[right]
            while subarray_sum >= target:
                min_length = min(min_length, right - left)
                left += 1
                subarray_sum -= nums[left]
        if min_length == len(nums) + 1:
            return 0
        return min_length
```

### ②

左端をループで回すパターン。

所要時間: 16:45

n: len(nums)
- 時間計算量: O(n)
- 空間計算量: O(n)

```py
class Solution:
    def minSubArrayLen(self, target: int, nums: List[int]) -> int:
        min_length = len(nums) + 1
        subarray_sum = 0
        end = 0
        for begin in range(len(nums)):
            while end < len(nums) and subarray_sum < target:
                subarray_sum += nums[end]
                end += 1
            if subarray_sum >= target:
                min_length = min(min_length, end - begin)
            subarray_sum -= nums[begin]
        if min_length == len(nums) + 1:
            return 0
        return min_length
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1233603535862628432/1264033965724012656
  - https://github.com/goto-untrapped/Arai60/pull/40

累積和を使ってもよい。

```py
class Solution:
    def minSubArrayLen(self, target: int, nums: List[int]) -> int:
        assert target > 0
        cumsum = [0]
        for num in nums:
            cumsum.append(cumsum[-1] + num)
        begin = 0
        min_length = len(nums) + 1
        for end in range(len(cumsum)):
            while cumsum[end] - cumsum[begin] >= target:
                min_length = min(min_length, end - begin)
                begin += 1
        if min_length == len(nums) + 1:
            return 0
        return min_length
```

- https://discord.com/channels/1084280443945353267/1225849404037009609/1253024249040474258
  - https://github.com/SuperHotDogCat/coding-interview/pull/31
- https://discord.com/channels/1084280443945353267/1196472827457589338/1246777474927562752
  - https://github.com/Mike0121/LeetCode/pull/22

累積和と二分探索。

```py
class Solution:
    def minSubArrayLen(self, target: int, nums: List[int]) -> int:
        cumsum = [0]
        for num in nums:
            cumsum.append(cumsum[-1] + num)
        min_length = len(nums) + 1
        for i in range(len(cumsum)):
            j = bisect_left(cumsum, cumsum[i] + target)
            if j < len(cumsum):
                min_length = min(min_length, j - i)
        if min_length == len(nums) + 1:
            return 0
        return min_length
```

- https://discord.com/channels/1084280443945353267/1201211204547383386/1228603635693387808
  - https://github.com/shining-ai/leetcode/pull/49
- https://discord.com/channels/1084280443945353267/1200089668901937312/1219219869510140005
  - https://github.com/hayashi-ay/leetcode/pull/51

全探索。TLE。

```py
class Solution:
    def minSubArrayLen(self, target: int, nums: List[int]) -> int:
        min_length = len(nums) + 1
        for begin in range(len(nums)):
            subarray_sum = 0
            end = begin
            while end < len(nums) and subarray_sum < target:
                subarray_sum += nums[end]
                end += 1
            if subarray_sum >= target:
                min_length = min(min_length, end - begin)
        if min_length == len(nums) + 1:
            return 0
        return min_length
```


## 3rd

```py
class Solution:
    def minSubArrayLen(self, target: int, nums: List[int]) -> int:
        assert target > 0
        min_length = len(nums) + 1
        subarray_sum = 0
        left = -1 # exclusive
        for right in range(len(nums)):
            subarray_sum += nums[right]
            while subarray_sum >= target:
                min_length = min(min_length, right - left)
                left += 1
                subarray_sum -= nums[left]
        if min_length == len(nums) + 1:
            return 0
        return min_length
```
