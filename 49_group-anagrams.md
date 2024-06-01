# 49. Group Anagrams

## 1st

### ①

各文字列をsortすることでcanonicalな形にしてそれをkeyにしてグループ分けする。
`0 <= strs[i].length <= 100` の制約から、Cで実装されたsorted() (O(nlogn))の方がO(n)なPythonの関数よりも速いだろうと考えそちらを使った。
    - https://github.com/niklas-heer/speed-comparison
    - https://benchmarksgame-team.pages.debian.net/benchmarksgame/box-plot-summary-charts.html


所要時間: 4:57

n: len(strs), m: str_の長さの平均
- 時間計算量: O(nm*log(m))
- 空間計算量: O(nm)

```py
class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        sorted_str_to_anagrams = defaultdict(list)
        for str_ in strs:
            sorted_str_to_anagrams[''.join(sorted(str_))].append(str_)
        return sorted_str_to_anagrams.values()
```

### ②

ordやchrが思い出せず時間を使った。

所要時間: 10:22

n: len(strs), m: str_の長さの平均
- 時間計算量: O(nm)
- 空間計算量: O(nm)

```py
class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        def counting_sort(s: str) -> str:
            counts = [0] * 26 # a-z
            for ch in s:
                counts[ord(ch) - ord('a')] += 1
            return ''.join(map(lambda t: chr(ord('a') + t[0]) * t[1], enumerate(counts)))

        sorted_str_to_anagrams = defaultdict(list)
        for str_ in strs:
            sorted_str_to_anagrams[counting_sort(str_)].append(str_)
        return sorted_str_to_anagrams.values()
```

文字列を作ってjoinしている部分が遅いかもと思いtupleを使うように変えた。

```py
class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        def histogram(s: str) -> Tuple[int]:
            counts = [0] * 26 # a-z
            for ch in s:
                counts[ord(ch) - ord('a')] += 1
            return tuple(counts)

        histogram_to_anagrams = defaultdict(list)
        for str_ in strs:
            histogram_to_anagrams[histogram(str_)].append(str_)
        return histogram_to_anagrams.values()
```

joinが140ms, tupleが100ms。全体の実行速度の分布から見て誤差の範囲ではなさそう。
ちなみにsortした版は83ms。tuple版より若干速いか？


## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1226508154833993788/1245034421363539969
    - https://github.com/nittoco/leetcode/pull/13#discussion_r1618111569
      > 返り値がlistになってないです。
      > https://docs.python.org/3/library/stdtypes.html#dict.values

typeが違うこと自体は認識していたが、考えが足らなかった。改めて確認すると、とくに

```py
>>> {'foo': 10}.values()[0]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'dict_values' object is not subscriptable
```

や

```py
>>> d = {'foo': 10}
>>> values = d.values()
>>> values
dict_values([10])
>>> d['foo'] = 20
>>> values
dict_values([20]) # 変わる
```

辺りがlistだと思っているとまずそう。 (Dictionary view objects: https://docs.python.org/3/library/stdtypes.html#dictionary-view-objects)

- https://discord.com/channels/1084280443945353267/1227102763222171648/1237603095773708289
- https://discord.com/channels/1084280443945353267/1227073733844406343/1231928145452601436
- https://discord.com/channels/1084280443945353267/1200089668901937312/1205882615035338823


```py
class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        sorted_to_anagrams = defaultdict(list)
        for str_ in strs:
            sorted_to_anagrams[''.join(sorted(str_))].append(str_)
        return list(sorted_to_anagrams.values())
```

```py
class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        def histogram(s: str) -> Tuple[int]:
            counts = [0] * 26 # a-z
            for ch in s:
                counts[ord(ch) - ord('a')] += 1
            return tuple(counts)

        histogram_to_anagrams = defaultdict(list)
        for str_ in strs:
            histogram_to_anagrams[histogram(str_)].append(str_)
        return list(histogram_to_anagrams.values())
```

## 3rd

```py
class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        sorted_to_anagrams = defaultdict(list)
        for str_ in strs:
            sorted_to_anagrams[''.join(sorted(str_))].append(str_)
        return list(sorted_to_anagrams.values())
```
