# 121. Best Time to Buy and Sell Stock

## 1st

### ①

`prices[i] - min(prices[:i+1])` をループで回してその最大値を取れば良い。

所要時間: 8:05

n: len(prices)
- 時間計算量: O(n)
- 空間計算量: O(1)

```py
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        if not prices:
            return 0
        max_profit = 0
        min_price = prices[0]
        for i in range(1, len(prices)):
            min_price = min(min_price, prices[i])
            max_profit = max(max_profit, prices[i] - min_price)
        return max_profit
```

### ②

①とは逆から。 `max(prices[i:]) - prices[i]` を後ろからループで回してその最大値を取る。max_priceの初期値をpriceの値域の下限にしておりprices[0]を使っていないので `if not prices: return 0` は不要。

n: len(prices)
- 時間計算量: O(n)
- 空間計算量: O(1)

```py
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        max_price = 0
        max_profit = 0
        for price in reversed(prices):
            max_price = max(max_price, price)
            max_profit = max(max_profit, max_price - price)
        return max_profit
```


### ③

全探索。入力長が最大10^5なのでこれだとTLEする。

n: len(prices)
- 時間計算量: O(n^2)
- 空間計算量: O(1)

```py
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        max_profit = 0
        for i in range(len(prices)):
            for j in range(i + 1, len(prices)):
                max_profit = max(max_profit, prices[j] - prices[i])
        return max_profit
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1229085360403775569/1247918104978259969
- https://discord.com/channels/1084280443945353267/1227073733844406343/1247938453082210324
- https://discord.com/channels/1084280443945353267/1183683738635346001/1239989564135706726
- https://discord.com/channels/1084280443945353267/1201211204547383386/1224003297145389096
- https://discord.com/channels/1084280443945353267/1200089668901937312/1219264054602891274
- https://discord.com/channels/1084280443945353267/1206101582861697046/1218990153306079327
- https://discord.com/channels/1084280443945353267/1192728121644945439/1218232472559423529

省略

## 3rd

```py
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        max_profit = 0
        max_price = 0
        for price in reversed(prices):
            max_price = max(max_price, price)
            max_profit = max(max_profit, max_price - price)
        return max_profit
```
