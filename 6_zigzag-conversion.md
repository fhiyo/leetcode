# 6. Zigzag Conversion

## 1st

### ①

ジグザグに動いて作った2次元のリストを1つの文字列に結合する。

所要時間: 8:55

n: len(s)
- 時間計算量: O(n)
- 空間計算量 (auxiliary): O(n)

```py
class Solution:
    def convert(self, s: str, numRows: int) -> str:
        assert numRows > 0
        if numRows == 1:
            return s
        zigzag_rows = [[] for _ in range(numRows)]
        row = 0
        direction = -1
        for c in s:
            zigzag_rows[row].append(c)
            if row == 0:
                direction = 1
            elif row == numRows - 1:
                direction = -1
            row += direction
        return ''.join(map(lambda row: ''.join(row), zigzag_rows))
```

### ②

計算でどの位置が何行目に属するかを求める。

なお、zigzag_rowsにappendするリストやformer, latterをgeneratorに変えようとしたところ、異なる答えになった。

たとえば、入力: s = "PAYPALISHIRING", numRows = 4 で
出力: "PINYAHRYAHRPI", 期待する出力: "PINALSIGYAHRPI"
のように真ん中辺りの文字列だけ異なる。

おそらく遅延評価されてループ内のformer, latterで参照するiがnumRows-2になっているのだと思われる。


所要時間: 24:42

n: len(s), m: numRows
- 時間計算量: O(mn)
- 空間計算量 (auxiliary): O(n)

```py
class Solution:
    def convert(self, s: str, numRows: int) -> str:
        assert numRows > 0
        if numRows == 1:
            return s
        zigzag_rows = []
        # nr: numRows
        # 0, 2nr-2, 2*(2nr-2), ...
        zigzag_rows.append([c for i, c in enumerate(s) if i % (2 * numRows - 2) == 0])
        for i in range(1, numRows - 1):
            # i, 2nr-2+i, 2*(2nr-2)+i, ...
            former = [c for j, c in enumerate(s) if j % (2 * numRows - 2) == i]
            # 2nr-2-i, 2*(2nr-2)-i, ...
            latter = [c for j, c in enumerate(s) if j % (2 * numRows - 2) == 2 * numRows - 2 - i]
            row_elements = chain.from_iterable(zip_longest(former, latter, fillvalue=''))
            zigzag_rows.append(list(row_elements))
        # nr-1, (2nr-2)+nr-1, 2*(2nr-2)+nr-1, ...
        zigzag_rows.append([c for i, c in enumerate(s) if i % (2 * numRows - 2) == numRows - 1])
        return ''.join(map(lambda row: ''.join(row), zigzag_rows))
```

### ③

②をもう少し効率化したバージョン。
`for i in range(1, numRows - 1)` のiはrowとして、中のループ変数をjじゃなくiにした方が間違いにくいかもしれない。

所要時間: 3:55

n: len(s)
- 時間計算量: O(n)
- 空間計算量 (auxiliary): O(n)

```py
class Solution:
    def convert(self, s: str, numRows: int) -> str:
        assert numRows > 0
        if numRows == 1:
            return s
        zigzag_chars = []
        cycle = 2 * numRows - 2
        for i in range(0, len(s), cycle):
            zigzag_chars.append(s[i])
        for i in range(1, numRows - 1):
            for j in range(i, len(s), cycle):
                zigzag_chars.append(s[j])
                next_j = j + cycle - 2 * i
                if next_j < len(s):
                    zigzag_chars.append(s[next_j])
        for i in range(numRows - 1, len(s), cycle):
            zigzag_chars.append(s[i])
        return ''.join(zigzag_chars)
```


## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1196472827457589338/1248961797457842196
  - https://github.com/Mike0121/LeetCode/pull/26

行って戻る、をループの中で繰り返す。

```py
class Solution:
    def convert(self, s: str, numRows: int) -> str:
        assert numRows > 0
        if numRows == 1:
            return s
        zigzag_rows = [[] for _ in range(numRows)]
        i = 0
        while i < len(s):
            for j in range(numRows):
                if i >= len(s):
                    break
                zigzag_rows[j].append(s[i])
                i += 1
            for j in range(numRows - 2, 0, -1):
                if i >= len(s):
                    break
                zigzag_rows[j].append(s[i])
                i += 1
        return ''.join(map(lambda row: ''.join(row), zigzag_rows))
```

- https://discord.com/channels/1084280443945353267/1233603535862628432/1233762537598877750
  - https://github.com/goto-untrapped/Arai60/pull/5
- https://discord.com/channels/1084280443945353267/1201211204547383386/1232754805907132556
  - https://github.com/shining-ai/leetcode/pull/60

③の別バージョン。
ループ内のrowはスコープが嫌な感じだが、 `row = position if position < numRows else cycle - position` と三項演算子で書くのもきれいな感じにならず仕方なくこちらで書いた。

```py
class Solution:
    def convert(self, s: str, numRows: int) -> str:
        assert numRows > 0
        if numRows == 1:
            return s
        zigzag_rows = [[] for _ in range(numRows)]
        cycle = 2 * numRows - 2
        for i in range(len(s)):
            position = i % cycle
            if position < numRows:
                row = position
            else:
                row = cycle - position
            zigzag_rows[row].append(s[i])
        return ''.join(map(lambda row: ''.join(row), zigzag_rows))
```

- https://discord.com/channels/1084280443945353267/1225849404037009609/1229834874823774278
  - https://github.com/SuperHotDogCat/coding-interview/pull/4
- https://discord.com/channels/1084280443945353267/1200089668901937312/1225104265933230101
  - https://github.com/hayashi-ay/leetcode/pull/71
- https://discord.com/channels/1084280443945353267/1217527351890546789/1218854624237326338
  - https://github.com/cheeseNA/leetcode/pull/4


## 3rd

自分的に一番素直な感じがしたのでこれを3rdにした。

```py
class Solution:
    def convert(self, s: str, numRows: int) -> str:
        assert numRows > 0
        if numRows == 1:
            return s
        zigzag_rows = [[] for _ in range(numRows)]
        row = 0
        direction = 1
        for c in s:
            if row == 0:
                direction = 1
            elif row == numRows - 1:
                direction = -1
            zigzag_rows[row].append(c)
            row += direction
        return ''.join(map(lambda row: ''.join(row), zigzag_rows))
```
