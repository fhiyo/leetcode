# 779. K-th Symbol in Grammar

## 1st

観察すると、ある行は一つ前の行をコピーしてから反転させたものをくっつけた形になっている。その特徴を利用して計算すれば解ける。

powerの計算は `2 ** floor(log2(k - 1))` でも良いが、計算誤差が怖いので整数の世界で計算している。


所要時間: 15:28

- 時間計算量: O((log n)^2)
- 空間計算量: O(1)

```py
class Solution:
    def kthGrammar(self, n: int, k: int) -> int:
        if k == 1:
            return 0
        power = 1
        while 2 * power < k:
            power *= 2
        return 1 - self.kthGrammar(n, k - power)

# 0
# 0 1
# 0 1 10
# 0 1 10 1001
# 0 1 10 1001 10010110
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1233603535862628432/1249699176418377789
  - https://github.com/goto-untrapped/Arai60/pull/26

nを利用すれば関数呼び出しのたびにpowerを計算しなくて済む。

```py
class Solution:
    def kthGrammar(self, n: int, k: int) -> int:
        if k == 1:
            return 0
        assert n >= 2
        half_num_elements = 2 ** (n - 2) # 2**(n-1) // 2
        if k > half_num_elements:
            return 1 - self.kthGrammar(n - 1, k - half_num_elements)
        return self.kthGrammar(n - 1, k)
```

答えを0か1かのどちらかに決め打ちして、ルールに則って反転させながらkを小さくして1番目に到達したときに0なら合っていて、1なら違ってたので反転させたものが答えになる、というやり方。

```py
class Solution:
    def kthGrammar(self, n: int, k: int) -> int:
        if k == 1:
            return 0
        assert n >= 2
        half_num_elements = 2 ** (n - 2) # 2**(n-1) // 2
        assumption = 0
        x = assumption
        while k > 1:
            if k > half_num_elements:
                k -= half_num_elements
                x = 1 - x
            half_num_elements >>= 1
        if x == 0:
            return assumption
        return 1 - assumption
```

法則的にこれでいいらしい。なるほど。

```py
class Solution:
    def kthGrammar(self, n: int, k: int) -> int:
        return (k - 1).bit_count() % 2
```

- https://discord.com/channels/1084280443945353267/1239148130679783424/1247954393773510678
  - https://github.com/SuperHotDogCat/coding-interview/pull/26

0 -> 01, 1 -> 10 と変換されるので、変換前の値と比較して同じ数字は奇数番目、異なる数字は偶数番目に位置することを使って計算する。

```py
class Solution:
    def kthGrammar(self, n: int, k: int) -> int:
        if k == 1:
            return 0
        before = self.kthGrammar(n, (k + 1) // 2)
        if k % 2 == 0:
            return 1 - before
        return before
```

- https://discord.com/channels/1084280443945353267/1196472827457589338/1242846190500839505
  - https://github.com/Mike0121/LeetCode/pull/18
- https://discord.com/channels/1084280443945353267/1233295449985650688/1241770690164818070
  - https://github.com/Exzrgs/LeetCode/pull/12
- https://discord.com/channels/1084280443945353267/1201211204547383386/1227255433408811100
  - https://github.com/shining-ai/leetcode/pull/46
- https://discord.com/channels/1084280443945353267/1200089668901937312/1215965653500956722
  - https://github.com/hayashi-ay/leetcode/pull/46

## 3rd


```py
class Solution:
    def kthGrammar(self, n: int, k: int) -> int:
        if k == 1:
            return 0
        assert n >= 2
        half_num_elements = 2 ** (n - 2) # 2**(n-1) // 2
        if k > half_num_elements:
            return 1 - self.kthGrammar(n - 1, k - half_num_elements)
        return self.kthGrammar(n - 1, k)
```
