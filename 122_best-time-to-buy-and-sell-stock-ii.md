# 122. Best Time to Buy and Sell Stock II

## 1st

当日と前日を比べて当日の方が高くなっていたら、前日に買ったことにしてその場で売る、を繰り返せばよい。

所要時間: 3:07

n: len(prices)
- 時間計算量: O(n)
- 空間計算量: O(1)

```py
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        if not prices:
            return 0
        max_profit = 0
        for i in range(1, len(prices)):
            if prices[i] - prices[i - 1] > 0:
                max_profit += prices[i] - prices[i - 1]
        return max_profit
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1227073733844406343/1247938453082210324
  - https://github.com/hayashi-ay/leetcode/pull/56/files

ある日の状態はその株を持っているか持っていないかの2択。i日目に株を持っている状態の最良の収支と持っていない状態の最良の収支を考えると、

- i+1日目に株を持っている状態での最良の収支は以下の行動のどちらかから得られる
  1. i日目に持っている株をキープ
  2. i+1日目に株を買う
- i+1日目に株を持っていない状態での最良の収支は以下の行動のどちらかから得られる
  1. i日目に持っている株を売る
  2. 引き続き買わない

となるのでこれを繰り返せば最良を更新し続けられる。最終日で株を持っていない状態での収支が答え (最後株を持ってるなら売った方がその分得だから)。

```py
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        if not prices:
            return 0
        profit_with_stock = -prices[0]
        profit_without_stock = 0
        for i in range(1, len(prices)):
            profit_with_stock = max(profit_with_stock, profit_without_stock - prices[i])
            profit_without_stock = max(profit_without_stock, profit_with_stock + prices[i])
        return profit_without_stock
```

- https://discord.com/channels/1084280443945353267/1239148130679783424/1247938356382404649
- https://discord.com/channels/1084280443945353267/1201211204547383386/1224003297145389096
- https://discord.com/channels/1084280443945353267/1210494002277908491/1210521728699076628


## 3rd

```py
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        if not prices:
            return 0
        max_profit = 0
        for i in range(1, len(prices)):
            difference = prices[i] - prices[i - 1]
            if difference > 0:
                max_profit += difference
        return max_profit
```
