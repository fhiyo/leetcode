# 213. House Robber II

## 1st

### ①

両端の家のどちらかを襲わないと決めれば、[House Robber - LeetCode](https://leetcode.com/problems/house-robber/description/)と同様のアルゴリズムで計算できる。
コメントを書くのに時間がかかったが、それを引いても10分以上はかかっている...
begin_index, end_indexを引数に持つように書いたが、このくらいならO(n)かけてコピーした配列を引数に持たせるようにした方が処理が分かりやすくていいかもしれない (Pythonであまりスライスによるコピーコストを気にしても意味が薄い場面も多そう。o(n)な計算量にしたいときはそれが律速になるので避けるが)。

所要時間: 16:30

n: len(nums)
- 時間計算量: O(n)
- 空間計算量: O(1)

```py
from collections.abc import Iterable

class Solution:
    def rob(self, nums: List[int]) -> int:
        if not nums:
            return 0
        if len(nums) == 1:
            return nums[0]
        return max(self._rob_helper(nums, 1, len(nums)), self._rob_helper(nums, 0, len(nums) - 1))

    def _rob_helper(self, nums: Iterable[int], begin_index: int, end_index: int) -> int:
        assert begin_index < end_index
        if end_index - begin_index == 1:
            return nums[begin_index]
        # max_money[i]: maximum amount of money from houses between begin_index-th to i-th
        # max_money_so_far == [max_money[begin_index], max_money[begin_index+1]]
        max_money_so_far = [nums[begin_index], max(nums[begin_index:begin_index + 2])]
        for i in range(begin_index + 2, end_index):
            # max_money_so_far == [max_money[i-2], max_money[i-1]]
            prev = max_money_so_far[1]
            max_money_so_far[1] = max(max_money_so_far[1], max_money_so_far[0] + nums[i])
            max_money_so_far[0] = prev
            # max_money_so_far == [max_money[i-1], max_money[i]]
        return max_money_so_far[1] # max_money[end_index-1]
```

### ②

メモ化再帰。_rob_helperの引数の型はlist[int]でなくSequence[int]のより広い型にしてみた。
[typing — Support for type hints — Python 3.12.4 documentation](https://docs.python.org/3/library/typing.html#typing.List)に

> Note that to annotate arguments, it is preferred to use an abstract collection type such as Sequence or Iterable rather than to use list or typing.List.

という記述を見つけたため。

所要時間: 9分くらい (計測ミスで正確な時間が不明)

n: len(nums)
- 時間計算量: O(n)
- 空間計算量: O(n)

```py
from collections.abc import Sequence

class Solution:
    def rob(self, nums: List[int]) -> int:
        if not nums:
            return 0
        if len(nums) == 1:
            return nums[0]
        return max(self._rob_helper(nums[1:]), self._rob_helper(nums[:-1]))

    def _rob_helper(self, nums: Sequence[int]) -> int:
        @cache
        def max_money_until(i: int) -> int:
            if i < 2:
                return max(nums[:i + 1])
            return max(max_money_until(i - 1), max_money_until(i - 2) + nums[i])

        return max_money_until(len(nums) - 1)
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1227073733844406343/1245669420156588032
- https://discord.com/channels/1084280443945353267/1201211204547383386/1223520553583902905
- https://discord.com/channels/1084280443945353267/1200089668901937312/1218564286624960582

省略

## 3rd


```py
class Solution:
    def rob(self, nums: List[int]) -> int:
        if not nums:
            return 0
        if len(nums) == 1:
            return nums[0]
        return max(self._rob_helper(nums[1:]), self._rob_helper(nums[:-1]))

    def _rob_helper(self, nums: Sequence[int]) -> int:
        if len(nums) == 1:
            return nums[0]
        # max_money[i]: maximum amount of money taken until i-th house
        # max_money_so_far: [max_money[0], max_money[1]]
        max_money_so_far = [nums[0], max(nums[:2])]
        for i in range(2, len(nums)):
            # max_money_so_far: [max_money[i-2], max_money[i-1]]
            prev = max_money_so_far[1]
            max_money_so_far[1] = max(max_money_so_far[1], max_money_so_far[0] + nums[i])
            max_money_so_far[0] = prev
            # max_money_so_far: [max_money[i-1], max_money[i]]
        return max_money_so_far[1] # max_money[-1]
```
