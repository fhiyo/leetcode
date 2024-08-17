# 392. Is Subsequence

## 1st

### ①

tの文字を順番に見ていき、sの注目している文字と一致したら注目する部分を一つ右にずらす、をsの頭から繰り返して最終的に尻尾までいけばOK.

所要時間: 8:32

m: len(s), n: len(t)
- 時間計算量: O(m + n)
- 空間計算量: O(1)

```py
class Solution:
    def isSubsequence(self, s: str, t: str) -> bool:
        i = 0
        for c in t:
            if i == len(s):
                return True
            if s[i] == c:
                i += 1
        return i == len(s)
```

### ②

正規表現で解く。解き始める前から考えていたのですぐ書けた。ただしアルファベットが英小文字のみなので良いが、特殊文字が入力に来うる場合は破綻する。その場合、sを `re.escape(s)` に変更すればよいだろうか？多分それで良いと思うんだが自信がない。

所要時間: 0:47

m: len(s), n: len(t)
- 時間計算量: 分からない。1文字あたりパターンに対して指数時間かかるとすると O(n * 2^m) か？
- 空間計算量: 分からない。パターンだけならO(m)だが追加でどのくらい空間を使っている？

```py
class Solution:
    def isSubsequence(self, s: str, t: str) -> bool:
        return bool(re.fullmatch('.*' + '.*'.join(s) + '.*', t))
```

### ③

follow-upを考えてみた。新しいsが来るたびにtを舐めるのは非効率なので、tを前処理しておきたい。tの各文字に対するindexの整列されたリストを辞書として持っておけば、tのどのindexまで見たかをprev_indexに入れておき、(注目している文字についての) tのindexのリストを辞書から取り出してprev_index + 1を二分探索することで次のindexの有無と、あればその値が分かる。
変数名が微妙だが上手いものが思いつかなかった...

所要時間: 13:49

m: len(s), n: len(t)
- 時間計算量: O(m + n) (ストリームの長さをl, s_iの平均の長さをmとすると O(lm * log(n)))
- 空間計算量: O(n)

```py
class Solution:
    def isSubsequence(self, s: str, t: str) -> bool:
        char_to_indexes = defaultdict(list)
        for i, c in enumerate(t):
            char_to_indexes[c].append(i)
        prev_index = -1
        for c in s:
            indexes = char_to_indexes[c]
            i = bisect_left(indexes, prev_index + 1)
            if i == len(indexes):
                return False
            prev_index = indexes[i]
        return True
```

実際にsが複数来るとこのコードではsごとに辞書を構築してしまうので厳密にはダメだが、インターフェース的に書きにくかったので省略した。インターフェースの想定は以下のような感じ。

```py
def isSubsequences(self, stream: Sequence[str], t: str) -> Iterator[bool]:
    ... # 辞書の構築
    for s in stream:
      yield self._isSubsequence(s, char_to_indexes)

def _isSubsequence(self, s: str, char_to_indexes: dict[str, list[str]]) -> bool:
    prev_index = -1
    for c in s:
        indexes = char_to_indexes[c]
        i = bisect_left(indexes, prev_index + 1)
        if i == len(indexes):
            return False
        prev_index = indexes[i]
    return True
```

### ④

sでループを回して、対応するtの文字のindexを検索する方法。

所要時間: 3:02

m: len(s), n: len(t)
- 時間計算量: O(m + n)
- 空間計算量: O(1)

```py
class Solution:
    def isSubsequence(self, s: str, t: str) -> bool:
        i = -1
        for c in s:
            i = t.find(c, i + 1)
            if i == -1:
                return False
        return True
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1253694251271852095/1273659786369700012
  - https://github.com/rihib/leetcode/pull/19
- https://discord.com/channels/1084280443945353267/1196472827457589338/1248643497695641633
  - https://github.com/Mike0121/LeetCode/pull/25

番兵を使って①の処理を簡略化する。番兵としてどの文字を採用するか実務では難しそうではあるが。
ヌル文字を使ってみた。

```py
class Solution:
    def isSubsequence(self, s: str, t: str) -> bool:
        s += '\0'
        i = 0
        for c in t:
            if s[i] == c:
                i += 1
        return s[i] == '\0'
```

- https://discord.com/channels/1084280443945353267/1226508154833993788/1247210316501090345
  - https://github.com/nittoco/leetcode/pull/16

whileでsとtのindexを進める。見てみるとsとtで対称性があってシンプルだった。

```py
class Solution:
    def isSubsequence(self, s: str, t: str) -> bool:
        si = 0
        ti = 0
        while True:
            if si == len(s):
                return True
            if ti == len(t):
                return False
            if s[si] == t[ti]:
                si += 1
            ti += 1
```

こっちのパターンもある。

```py
class Solution:
    def isSubsequence(self, s: str, t: str) -> bool:
        si = 0
        ti = 0
        while si < len(s) and ti < len(t):
            if s[si] == t[ti]:
                si += 1
            ti += 1
        return si == len(s)
```

- https://discord.com/channels/1084280443945353267/1225849404037009609/1243237772039294988
  - https://github.com/SuperHotDogCat/coding-interview/pull/21
- https://discord.com/channels/1084280443945353267/1233603535862628432/1238808534729359401
  - https://github.com/goto-untrapped/Arai60/pull/19
- https://discord.com/channels/1084280443945353267/1201211204547383386/1231617397446803466
  - https://github.com/shining-ai/leetcode/pull/57

最長共通部分列の長さを求めて、その値がsの長さと一致しているかを調べる。

```py
class Solution:
    def isSubsequence(self, s: str, t: str) -> bool:
        # lcs: longest common subsequence
        # lcs_count_table[i][j]: length of lcs between s[:si] and t[:ti]
        lcs_count_table = [[0] * (len(s) + 1) for _ in range(len(t) + 1)]
        for ti in range(1, len(t) + 1):
            for si in range(1, len(s) + 1):
                if s[si - 1] == t[ti - 1]:
                    lcs_count_table[ti][si] = lcs_count_table[ti - 1][si - 1] + 1
                else:
                    lcs_count_table[ti][si] = max(lcs_count_table[ti][si - 1], lcs_count_table[ti - 1][si])
        return lcs_count_table[-1][-1] == len(s)
```

- https://discord.com/channels/1084280443945353267/1200089668901937312/1222146248056770610
  - https://github.com/hayashi-ay/leetcode/pull/64


## 3rd

```py
class Solution:
    def isSubsequence(self, s: str, t: str) -> bool:
        i = 0
        for c in t:
            if i == len(s):
                return True
            if s[i] == c:
                i += 1
        return i == len(s)
```
