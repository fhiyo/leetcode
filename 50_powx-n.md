# 50. Pow(x, n)

## 1st

### ①

対数オーダーしかスタックに積まれないので再帰で実装。PythonだとC++などと違い値の範囲を気にする場面が減って楽ではある。C++だと n = -2^31 のときに符号を逆にするとsigned intではオーバーフローするのに注意しないといけない。

所要時間: 5:28

- 時間計算量: O(log * |n|)
- 空間計算量: O(log * |n|)

```py
class Solution:
    def myPow(self, x: float, n: int) -> float:
        if n < 0:
            return 1 / self.myPow(x, -n)
        if n == 0:
            return 1
        ans = self.myPow(x, n // 2)
        ans *= ans
        if n % 2 != 0:
            ans *= x
        return ans
```

### ②

iterativeに計算。ansという変数名でもこれくらいなら十分だとは思う。poweredとかでもいいかもしれない。

floor(log2(n))は `n.bit_length()` を使えば良かった。


所要時間: 8:51
- 時間計算量: O(log |n|)
- 空間計算量: O(1)

```py
class Solution:
    def myPow(self, x: float, n: int) -> float:
        if n < 0:
            return 1 / self.myPow(x, -n)
        if n == 0:
            return 1
        ans = 1
        for i in range(floor(log2(n)) + 1):
            if n & (1 << i) != 0:
                ans *= x
            x *= x
        return ans
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1233603535862628432/1252973832822853642
  - https://github.com/goto-untrapped/Arai60/pull/29

再帰は①よりこっちの方がキレイな気がした (現状これで末尾再帰最適化はされないと思うが)。

```py
class Solution:
    def myPow(self, x: float, n: int) -> float:
        if n < 0:
            return 1 / self._pow(x, -n, 1)
        return self._pow(x, n, 1)

    def _pow(self, x: float, n: int, result: float) -> float:
        assert n >= 0
        if n == 0:
            return result
        if n % 2 == 0:
            return self._pow(x * x, n // 2, result)
        return self._pow(x * x, n // 2, result * x)
```

- https://discord.com/channels/1084280443945353267/1233295449985650688/1247564378178326620
  - https://github.com/Exzrgs/LeetCode/pull/28
    - powのcpython実装
      - https://github.com/python/cpython/blob/bdab67e1c795443a0d8f8a5bbeb3a91ac4fd5a19/Objects/longobject.c#L4894
      - ざっとしか見ていない...
- https://discord.com/channels/1084280443945353267/1226508154833993788/1249032207092154379
  - https://github.com/nittoco/leetcode/pull/17
- https://discord.com/channels/1084280443945353267/1196472827457589338/1242419019558948904
  - https://github.com/Mike0121/LeetCode/pull/17
- https://discord.com/channels/1084280443945353267/1225849404037009609/1236005160413696041
  - https://github.com/SuperHotDogCat/coding-interview/pull/15
- https://discord.com/channels/1084280443945353267/1201211204547383386/1226870892559335527
  - https://github.com/shining-ai/leetcode/pull/45
- https://discord.com/channels/1084280443945353267/1200089668901937312/1214840632657313832
  - https://github.com/hayashi-ay/leetcode/pull/41
  - https://discord.com/channels/1084280443945353267/1200089668901937312/1214840985788350494


## 3rd

```py
class Solution:
    def myPow(self, x: float, n: int) -> float:
        if n < 0:
            return 1 / self._pow(x, -n, 1)
        return self._pow(x, n, 1)

    def _pow(self, x: float, n: int, result: float) -> float:
        assert n >= 0
        if n == 0:
            return result
        if n & 1:
            return self._pow(x * x, n >> 1, result * x)
        return self._pow(x * x, n >> 1, result)
```
