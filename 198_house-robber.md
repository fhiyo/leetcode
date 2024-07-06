# 198. House Robber

## 1st

### ①

DPで解く解法を何となく覚えていた。漸化式から前2つだけ覚えておけば良いことがわかる。

`len(nums) == 2`のときは特別扱いしなくて良かった。

所要時間: 10:43

n: len(nums)
- 時間計算量: O(n)
- 空間計算量: O(1)

```py
class Solution:
    def rob(self, nums: List[int]) -> int:
        if not nums:
            return 0
        if len(nums) == 1:
            return nums[0]
        if len(nums) == 2:
            return max(nums[0], nums[1])
        max_money_so_far = [nums[0], max(nums[0], nums[1])]
        for i in range(2, len(nums)):
            prev = max_money_so_far[1]
            max_money_so_far[1] = max(max_money_so_far[1], max_money_so_far[0] + nums[i])
            max_money_so_far[0] = prev
        return max_money_so_far[1]
```

### ②

メモ化再帰。rob_helperはmax_robbable_money_fromという名前でもいいかもしれない。

所要時間: 7:35

n: len(nums)
- 時間計算量: O(n)
- 空間計算量: O(n)

```py
class Solution:
    def rob(self, nums: List[int]) -> int:
        @cache
        def rob_helper(i: int) -> int:
            if i == len(nums) - 1:
                return nums[-1]
            if i == len(nums) - 2:
                return max(nums[-2], nums[-1])
            return max(rob_helper(i + 1), rob_helper(i + 2) + nums[i])

        if not nums:
            return 0
        return rob_helper(0)
```

### ③

逆からメモ化再帰。こっちの方が素直か。rob_helperはmax_robbable_money_untilという名前でもいいかもしれない。

所要時間: 1:47

n: len(nums)
- 時間計算量: O(n)
- 空間計算量: O(n)

```py
class Solution:
    def rob(self, nums: List[int]) -> int:
        @cache
        def rob_helper(i: int) -> int:
            if i == 0:
                return nums[0]
            if i == 1:
                return max(nums[0], nums[1])
            return max(rob_helper(i - 1), rob_helper(i - 2) + nums[i])

        if not nums:
            return 0
        return rob_helper(len(nums) - 1)
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1192736784354918470/1253694708719550516

maxはiterableを取れるので、①のmax_money_so_farの初期化は `max_money_so_far = [nums[0], max(nums[:1])]` で良い。

- https://discord.com/channels/1084280443945353267/1227073733844406343/1244882790638420060
- https://discord.com/channels/1084280443945353267/1201211204547383386/1223216025919553666

maxのdefaultキーワードを使って入力がから配列だったときの考慮をしている。

漸化式の初項を0, 二番目の項をnums[0]とする見方もある。

```py
class Solution:
    def rob(self, nums: List[int]) -> int:
        if len(nums) < 2:
            return max(nums, default=0)
        max_money_so_far = [0, nums[0]]
        for i in range(1, len(nums)):
            prev = max_money_so_far[1]
            max_money_so_far[1] = max(max_money_so_far[1], max_money_so_far[0] + nums[i])
            max_money_so_far[0] = prev
        return max_money_so_far[1]
```

書いてみると、どちらも趣味の範囲な気がした。

- https://discord.com/channels/1084280443945353267/1200089668901937312/1217116484203970701


## 3rd

```py
class Solution:
    def rob(self, nums: List[int]) -> int:
        if not nums:
            return 0
        if len(nums) == 1:
            return nums[0]
        max_money_so_far = [nums[0], max(nums[:2])]
        for i in range(2, len(nums)):
            prev = max_money_so_far[1]
            max_money_so_far[1] = max(max_money_so_far[1], max_money_so_far[0] + nums[i])
            max_money_so_far[0] = prev
        return max_money_so_far[1]
```
