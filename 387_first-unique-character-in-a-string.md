# 387. First Unique Character in a String

## 1st

### ①

素直に文字ごとの出現頻度を辞書にして、2パス目で左から1回しか出ていない文字を確認するようにした。

defaultdictの代わりにCounter()を使っても良い。

所要時間: 4:00

n: len(s)
- 時間計算量: O(n)
- 空間計算量: O(n)

```py
class Solution:
    def firstUniqChar(self, s: str) -> int:
        char_to_frequency = defaultdict(int)
        for c in s:
            char_to_frequency[c] += 1
        for i, c in enumerate(s):
            if char_to_frequency[c] == 1:
                return i
        return -1
```

### ②

Counter版

所要時間: 0:41

n: len(s)
- 時間計算量: O(n)
- 空間計算量: O(n)

```py
class Solution:
    def firstUniqChar(self, s: str) -> int:
        char_to_frequency = Counter(s)
        for i, c in enumerate(s):
            if char_to_frequency[c] == 1:
                return i
        return -1
```

### ③

辞書を作らないのであれば、左右から検索して最初に見つかった位置が一致していればunique, という探し方もできる。

所要時間: 1:33

n: len(s)
- 時間計算量: O(n^2)
- 空間計算量: O(1)

```py
class Solution:
    def firstUniqChar(self, s: str) -> int:
        for c in s:
            left_index = s.find(c)
            if left_index == s.rfind(c):
                return left_index
        return -1
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1198621745565937764/1244101017507991613
- https://discord.com/channels/1084280443945353267/1227073733844406343/1233831347051565137

議論にあったdefaultdictを書いてみる

継承

```py
class DefaultDict(dict):
  def __init__(self, default_factory=None):
    self.default_factory = default_factory

  def __missing__(self, key):
    if self.default_factory is None:
      raise KeyError
    self[key] = self.default_factory()
    return self[key]
```

委譲。こんな感じか？ `del d['foo']` とかはできるようにしていない

```py
class DefaultDict:
    def __init__(self, default_factory=None):
        self.d = {}
        self.default_factory = default_factory

    def __missing__(self, key):
        if self.default_factory is None:
            raise KeyError
        self.d[key] = self.default_factory()
        return self.d[key]

    def __getitem__(self, key):
        if not key in self:
            return self.__missing__(key)
        return self.d[key]

    def __setitem__(self, key, value):
        self.d[key] = value

    def __contains__(self, key):
        return key in self.d
```

- https://discord.com/channels/1084280443945353267/1200089668901937312/1209879956436291645
  - https://github.com/hayashi-ay/leetcode/pull/28/files

1-passの解法が面白かった。自分も参考に書いてみる。

```py
class Solution:
    def firstUniqChar(self, s: str) -> int:
        unique_occurrence_indexes = {} # 2回以上出現した場合はlen(s)が入る
        for i, c in enumerate(s):
            if c in unique_occurrence_indexes:
                unique_occurrence_indexes[c] = len(s)
                continue
            unique_occurrence_indexes[c] = i
        candidate = min(iter(unique_occurrence_indexes.values()))
        if candidate == len(s):
            return -1
        return candidate
```

## 3rd

```py
class Solution:
    def firstUniqChar(self, s: str) -> int:
        char_to_frequency = defaultdict(int)
        for c in s:
            char_to_frequency[c] += 1
        for i, c in enumerate(s):
            if char_to_frequency[c] == 1:
                return i
        return -1
```
