# 322. Coin Change

## 1st

### ①

DP。coinsのうち使えないものは最初にfilterで弾いた。 [itertools.filterfalse](https://docs.python.org/ja/3/library/itertools.html#itertools.filterfalse)を使って `coins = list(filterfalse(lambda c: c > amount, coins))` でも良い。

new_amountはもう少し良い変数名を出したいが...next_amountの方がマシだろうか。

所要時間: 12:36

m: amount, n: len(coins)
- 時間計算量: O(mn)
- 空間計算量: O(m)

```py
class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        coins = list(filter(lambda c: c <= amount, coins))
        num_coins_list = [-1] * (amount + 1)
        num_coins_list[0] = 0
        for i in range(amount):
            if num_coins_list[i] == -1:
                continue
            for coin in coins:
                new_amount = i + coin
                if new_amount > amount:
                    continue
                if num_coins_list[new_amount] == -1:
                    num_coins_list[new_amount] = num_coins_list[i] + 1
                else:
                    num_coins_list[new_amount] = min(num_coins_list[new_amount], num_coins_list[i] + 1)
        return num_coins_list[-1]
```

### ②

再帰。inf (とかsys.maxsizeとか) を使うのは少し抵抗があるが処理が簡潔になるのもあり使った。再帰の上限には注意。

所要時間: 11:49

m: amount, n: len(coins)
- 時間計算量: O(mn)
- 空間計算量: O(m)

```py
class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        @cache
        def calculate_min_num_coins(amount: int) -> int:
            if amount < 0:
                return inf
            if amount == 0:
                return 0
            min_num_coins = inf
            for coin in coins:
                min_num_coins = min(min_num_coins, calculate_min_num_coins(amount - coin) + 1)
            return min_num_coins

        min_num_coins = calculate_min_num_coins(amount)
        if isinf(min_num_coins):
            return -1
        return min_num_coins
```


### ③

非再帰のDFS。実行時間が前2つと比べて明らかに遅い (前2つが1s未満に対してこちらは4s〜6sくらい) が、理由がわからなかった...
stは本当はnum_coins_and_amount_stackとかにするだろうか。

`calculated = [False] * (amount + 1)` のような変数を用意してnum_coins_listの各要素が計算済みかを別で管理しても良い。

所要時間: 14:22

m: amount, n: len(coins)
- 時間計算量: O(mn)
- 空間計算量: O(m)

```py
class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        coins = list(filter(lambda c: c <= amount, coins))
        num_coins_list = [inf] * (amount + 1)
        st = [(0, 0)]
        while st:
            num_coins, current_amount = st.pop()
            num_coins_list[current_amount] = min(num_coins_list[current_amount], num_coins)
            for coin in coins:
                next_amount = current_amount + coin
                if next_amount > amount:
                    continue
                if num_coins_list[next_amount] <= num_coins + 1:
                    continue
                st.append((num_coins + 1, next_amount))
        if isinf(num_coins_list[amount]):
            return -1
        return num_coins_list[amount]
```


## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1233603535862628432/1259093012521418813
  - https://github.com/goto-untrapped/Arai60/pull/34/files

最短経路問題と見れるのでBFSで解く。

演算子の優先順位は in operator > or operator (https://docs.python.org/3/reference/expressions.html#operator-precedence)

```py
class Solution:
    def coinChange(self, coins: List[int], target_amount: int) -> int:
        coins = list(filter(lambda c: c <= target_amount, coins))
        queue = deque([(0, 0)])
        seen = set([0])
        while queue:
            num_coins, amount = queue.popleft()
            if amount == target_amount:
                return num_coins
            for coin in coins:
                next_amount = amount + coin
                if next_amount in seen or next_amount > target_amount:
                    continue
                queue.append((num_coins + 1, next_amount))
                seen.add(next_amount)
        return -1
```

下のように書いており、leetcode上でメモリ超過でエラーとなりしばらく考えていた。
下のコードだと1円, 2円の2枚の硬貨で考えると、(1枚, 1円), (1枚, 2円)がキューに入っている状態で前側が取り出されると、次に(2枚, 2円)がキューに入ってしまうがこれは無駄になる。

```py
class Solution:
    def coinChange(self, coins: List[int], target_amount: int) -> int:
        coins = list(filter(lambda c: c <= target_amount, coins))
        calculated_amounts = set()
        queue = deque([(0, 0)])
        while queue:
            num_coins, amount = queue.popleft()
            if amount == target_amount:
                return num_coins
            calculated_amounts.add(amount)
            for coin in coins:
                next_amount = amount + coin
                if next_amount in calculated_amounts or next_amount > target_amount:
                    continue
                queue.append((num_coins + 1, next_amount))
        return -1
```

- https://github.com/sakupan102/arai60-practice/pull/41/files
- https://discord.com/channels/1084280443945353267/1218823830743547914/1230504662822686830
  - https://github.com/ryoooooory/LeetCode/pull/4
- https://github.com/thonda28/leetcode/pull/1
- https://discord.com/channels/1084280443945353267/1200089668901937312/1224284238032011305
  - https://github.com/hayashi-ay/leetcode/pull/68


## 3rd


```py
class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        num_coins_list = [-1] * (amount + 1)
        num_coins_list[0] = 0
        for i in range(amount):
            if num_coins_list[i] == -1:
                continue
            for coin in coins:
                next_amount = i + coin
                if next_amount > amount:
                    continue
                if num_coins_list[next_amount] == -1:
                    num_coins_list[next_amount] = num_coins_list[i] + 1
                else:
                    num_coins_list[next_amount] = min(num_coins_list[next_amount], num_coins_list[i] + 1)
        return num_coins_list[amount]
```
