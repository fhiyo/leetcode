# 53. Maximum Subarray

## 1st

### ①

Kadane's algorithm. sum_so_farには `nums[i]` が末尾に来るsubarrayのうち和が最大の配列のsumが入る。`sum_so_far = nums[0]` に初期化して `for in nums[1:]` としてもいいし、 `for i in range(1, len(nums)):` でもいい。

所要時間: 5:43

n: len(nums)
- 時間計算量: O(n)
- 空間計算量: O(1)

```py
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        assert len(nums) > 0
        max_sum = nums[0]
        sum_so_far = 0
        for num in nums:
            if sum_so_far < 0:
                sum_so_far = 0
            sum_so_far += num
            max_sum = max(max_sum, sum_so_far)
        return max_sum
```

### ②

一応再帰でも書けるので書いてみた。変数名が思いつかず1回書き直している。
`if index == len(nums) - 1` でmax_sumを更新していないのはmax_sumの初期値を `nums[-1]` にしているからだが、分かりにくいとは思う。


所要時間: 6:27 (2回目)

n: len(nums)
- 時間計算量: O(n)
- 空間計算量: O(n)

```py
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        assert len(nums) > 0

        max_sum = nums[-1]
        # calculate sum of subarray starting from the index
        def calculate_subarray_sum_from(index: int) -> int:
            nonlocal max_sum
            if index == len(nums) - 1:
                return nums[-1]
            next_max_sum = calculate_subarray_sum_from(index + 1)
            if next_max_sum < 0:
                next_max_sum = 0
            sum_so_far = nums[index] + next_max_sum
            max_sum = max(max_sum, sum_so_far)
            return sum_so_far

        calculate_subarray_sum_from(0)
        return max_sum
```

### ③

全探索。TLEした。

n: len(nums)
- 時間計算量: O(n^2)
- 空間計算量: O(n)

```py
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        assert len(nums) > 0
        cumsum = [0]
        for num in nums:
            cumsum.append(cumsum[-1] + num)
        max_sum = nums[0]
        for i in range(len(cumsum)):
            for j in range(i+1, len(cumsum)):
                max_sum = max(max_sum, cumsum[j] - cumsum[i])
        return max_sum
```


## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1233603535862628432/1253911871841370294

全探索、自分は累積和を使った (③) がこちらにある書き方の方が素直だと思った。

```py
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        assert len(nums) > 0
        max_subarray_sum = nums[0]
        for i in range(len(nums)):
            subarray_sum = 0
            for j in range(i, len(nums)):
                subarray_sum += nums[j]
                max_subarray_sum = max(max_subarray_sum, subarray_sum)
        return max_subarray_sum
```

- https://github.com/goto-untrapped/Arai60/pull/30/files#r1649857466

自分で書くと下のようになった。(i番目までの累積和) - (i-1番目までの累積和の最小値) を計算することでi番目の要素を右端とする部分配列の和の最大値が手に入る。discord上で議論されているのは見ていたが理解していなかった...


```py
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        assert len(nums) > 0
        sum_so_far = 0
        min_cumsum = 0
        max_subarray_sum = nums[0]
        for num in nums:
            sum_so_far += num
            max_subarray_sum = max(max_subarray_sum, sum_so_far - min_cumsum)
            min_cumsum = min(min_cumsum, sum_so_far)
        return max_subarray_sum
```

- https://discord.com/channels/1084280443945353267/1233295449985650688/1254012425611513856
- https://discord.com/channels/1084280443945353267/1192736784354918470/1249368844829589557


## 3rd


```py
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        assert len(nums) > 0
        max_subarray_sum = nums[0]
        subarray_sum = nums[0]
        for i in range(1, len(nums)):
            if subarray_sum < 0:
                subarray_sum = 0
            subarray_sum += nums[i]
            max_subarray_sum = max(max_subarray_sum, subarray_sum)
        return max_subarray_sum
```
