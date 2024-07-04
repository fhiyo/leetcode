# 62. Unique Paths

## 1st

### ①

grid全部を持たなくても1行 or 1列のみ持ってそれを開始地点側から到着地点側へ移動させていけば良い。
今回はrowを取り回しているがrowとcolumnで短い方を取り回すようにすれば空間計算量はO(min(m,n))になる。

変数名、rowではないな。num_pathsとかか。

所要時間: 4:56

- 時間計算量: O(mn)
- 空間計算量: O(n)

```py
class Solution:
    def uniquePaths(self, m: int, n: int) -> int:
        row = [1] * n # first row's unique path count
        for _ in range(1, m):
            for i in range(1, n):
                row[i] += row[i - 1] # sum unique path count from left and up
        return row[-1]
```

### ②

再帰。(row, col)の位置からゴールに行くためのpathの数を再帰的に計算する。
cacheを使うと無制限にデータを貯めてしまうので少し抵抗がある。今回の問題はgridの大きさは大したことにならないはずなのでいいかもしれないが。

所要時間: 5:58

- 時間計算量: O(mn)
- 空間計算量: O(mn)

```py
class Solution:
    def uniquePaths(self, m: int, n: int) -> int:
        @cache
        def count_paths_from(row: int, col: int) -> int:
            if row == m - 1 and col == n - 1:
                return 1
            count = 0
            if row < m - 1:
                count += count_paths_from(row + 1, col)
            if col < n - 1:
                count += count_paths_from(row, col + 1)
            return count

        return count_paths_from(0, 0)
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1233603535862628432/1255522219258286240

gridの短い方の辺を取り回すように書く。

```py
class Solution:
    def uniquePaths(self, m: int, n: int) -> int:
        if m < n:
            m, n = n, m
        num_paths = [1] * n # first row's number of paths
        for _ in range(1, m):
            for i in range(1, n):
                num_paths[i] += num_paths[i - 1] # sum number of paths from left and up
        return num_paths[-1]

```

nCkで解く。

```py
class Solution:
    def uniquePaths(self, m: int, n: int) -> int:
        return comb(m + n - 2, m - 1)
```

combのソース: https://github.com/python/cpython/blob/17d5b9df10f53ae3c09c8b22f27d25d9e83b4b7e/Modules/mathmodule.c#L3805 ざっとだが読んだ。
そもそも任意精度をPythonでどう表現しているのかが気になったので、調べて出てきた記事 [Pythonの整数型はどのように実装されているのか](https://zenn.dev/yukinarit/articles/afb263bf68fff2) を読んだ。PyObjectの中でob_sizeとob_digitという変数を使って2^30の値を配列で持つことで大きな数を表現しているらしい。

combの簡易的な自作。

```py
class Solution:
    def uniquePaths(self, m: int, n: int) -> int:
        @cache
        def comb_(n: int, k: int) -> int:
            if k == 0:
                return 1
            if k == 1:
                return n
            j = k // 2
            return comb_(n, j) * comb_(n - j, k - j) // comb_(k, j)

        return comb_(m + n - 2, n - 1)
```

- https://discord.com/channels/1084280443945353267/1192736784354918470/1250055847225069631
- https://discord.com/channels/1084280443945353267/1227073733844406343/1243465298850349089

nCk = n-1Ck + n-1Ck-1 があったか。

```py
class Solution:
    def uniquePaths(self, m: int, n: int) -> int:
        @cache
        def comb_(n: int, k: int) -> int:
            if n == 0 or k == 0 or n == k:
                return 1
            if k == 1:
                return n
            return comb_(n - 1, k) + comb_(n - 1, k - 1)

        return comb_(m + n - 2, n - 1)
```


## 3rd

```py
class Solution:
    def uniquePaths(self, m: int, n: int) -> int:
        if m < n:
            m, n = n, m
        num_paths = [1] * n # first row's number of paths
        for _ in range(1, m):
            for i in range(1, n):
                num_paths[i] += num_paths[i - 1] # sum number of paths from left and up
        return num_paths[-1]
```
