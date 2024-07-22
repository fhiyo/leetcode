# 46. Permutations

## 1st

### ①

[cpythonのpermutationの内容](https://github.com/python/cpython/blob/bc264eac3ad14dab748e33b3d714c2674872791f/Modules/itertoolsmodule.c#L2603)を思い出しながら書いた。ギリギリ何とか書けたという感じ。

所要時間: 23:52

n: len(nums)
- 時間計算量: O(n*n!)
- 空間計算量: O(n!) (出力でn!個返すため)

```py
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        return list(self._perms(nums))

    def _perms(self, nums: list[int]) -> Iterator[list[int]]:
        length = len(nums)
        indices = list(range(length))
        yield [nums[i] for i in indices]
        cycles = list(range(length, 0, -1))
        while length:
            for i in reversed(range(length)):
                cycles[i] -= 1
                if cycles[i] == 0:
                    indices[i:] = indices[i+1:] + indices[i:i+1]
                    cycles[i] = length - i
                else:
                    j = cycles[i]
                    indices[i], indices[-j] = indices[-j], indices[i]
                    yield [nums[i] for i in indices]
                    break
            else:
                return
```

### ②

上の解法はnPrの各要素を計算をするための方法。r==nでいいならもっと簡単に書ける。
`if i in indices` の部分は、どうせnが小さいので辞書にしてO(1)にしてもオーバーヘッド分で相殺されるかなと思い最適化はしていない。

`if len(indices) == len(nums)` の終わりはreturnすればよかった。

所要時間: 8:11

n: len(nums)
- 時間計算量: O(n^2 * n!)
- 空間計算量: O(n!)

```py
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        def perm(indices: list[int]) -> Iterator[list[int]]:
            if len(indices) == len(nums):
                yield [nums[i] for i in indices]
            for i in range(len(nums)):
                if i in indices:
                    continue
                indices.append(i)
                yield from perm(indices)
                indices.pop()

        return list(perm([]))
```

### ③

1. 後ろから初めて昇順になっているindexを探す
2. 後ろから、1で見つけたindexの要素より初めて大きくなる要素のindexを探す
3. 上で見つけた2つのindexの位置を交換する
4. 1のindexより右側を逆順にする (アルゴリズム上、必ず降順になっているので逆にしたら辞書順最小になる)

所要時間: 17:48

n: len(nums)
- 時間計算量: O(n*n!)
- 空間計算量: O(n!)

```py
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        if len(nums) <= 1:
            return [nums[:]]

        def perm(indices: list[int]) -> Iterator[list[int]]:
            i = len(nums) - 2
            while i >= 0 and indices[i] > indices[i + 1]:
                i -= 1
            if i == -1:
                return
            j = len(indices) - 1
            while indices[i] > indices[j]:
                j -= 1
            indices[i], indices[j] = indices[j], indices[i]
            indices[i+1:] = reversed(indices[i+1:])
            yield [nums[i] for i in indices]
            yield from perm(indices)

        return [nums[:]] + list(perm(list(range(len(nums)))))
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1226508154833993788/1255879916113756202
  - https://github.com/nittoco/leetcode/pull/21


[e_1, e_2, ..., e_k] のリストがあるとき、要素の順番を保存したままe_{k+1}を挿入すると
- [e_1, e_2, ..., e_k, e_{k+1}]
- [e_1, e_2, ..., e_{k+1}, e_k]
- ...
- [e_{k+1}, e_1, e_2, ..., e_k]

のk+1通りのリストができる。これをすべての要素の順番について列挙すれば、[e_1, ..., e_{k+1}]の要素を使ったすべての順列を書き出すことができる。

```py
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        def perm(index: int) -> Iterator[list[int]]:
            if index == len(nums):
                yield []
                return
            for permutation in perm(index + 1):
                permutation.append(nums[index])
                yield permutation[:]
                for i in reversed(range(len(permutation) - 1)):
                    permutation[i], permutation[i+1] = permutation[i+1], permutation[i]
                    yield permutation[:]

        return list(perm(0))
```

ループでall_permutationsを更新していくやり方。変数名に自信はない。

```py
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        all_permutations = [[]]
        for num in reversed(nums):
            derived_permutations = []
            for permutation in all_permutations:
                permutation.append(num)
                copied = permutation[:]
                for i in reversed(range(len(copied) - 1)):
                    copied[i], copied[i+1] = copied[i+1], copied[i]
                    derived_permutations.append(copied[:])
            all_permutations += derived_permutations
        return all_permutations
```

- https://discord.com/channels/1084280443945353267/1233295449985650688/1245243664259878932
  - https://github.com/Exzrgs/LeetCode/pull/19
- https://discord.com/channels/1084280443945353267/1196472827457589338/1241302279303204915
  - https://github.com/Mike0121/LeetCode/pull/14


順列の置換を列挙すれば良い。個人的には一番分かりやすい気がする。

スライスによるindicesのコピーはしておくとスレッドセーフになる (はず)。マルチスレッドにするならコードも変えないといけないが...

```py
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        if not nums:
            return [[]]

        def perm(indices: list[int], i: int) -> Iterator[list[int]]:
            if i == len(nums) - 1:
                yield [nums[j] for j in indices]
                return
            for j in range(i, len(nums)):
                indices[i], indices[j] = indices[j], indices[i]
                yield from perm(indices[:], i + 1)
                indices[i], indices[j] = indices[j], indices[i]

        indices = list(range(len(nums)))
        return list(perm(indices[:], 0))
```

- https://discord.com/channels/1084280443945353267/1233603535862628432/1237775088762490900
  - https://github.com/goto-untrapped/Arai60/pull/16


スタックでも書いてみる。

```py
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        if not nums:
            return [[]]
        stack = [(list(range(len(nums))), 0)]
        all_permutations = []
        while stack:
            indices, index = stack.pop()
            if index == len(nums) - 1:
                all_permutations.append([nums[i] for i in indices])
                continue
            for i in range(index, len(nums)):
                indices[index], indices[i] = indices[i], indices[index]
                stack.append((indices[:], index + 1))
                indices[index], indices[i] = indices[i], indices[index]
        return all_permutations
```

- https://discord.com/channels/1084280443945353267/1225849404037009609/1235272786474303560
  - https://github.com/SuperHotDogCat/coding-interview/pull/12
- https://discord.com/channels/1084280443945353267/1201211204547383386/1228915333310713856
  - https://github.com/shining-ai/leetcode/pull/50
- https://discord.com/channels/1084280443945353267/1200089668901937312/1220995877624217620
  - https://github.com/hayashi-ay/leetcode/pull/57


## 3rd

全体的に命名をサボりがちかも。ただこのくらいなら読めるんじゃないか？という気もしてこうなっている (が、後で読んだら読めないとかあるのかもしれない)


```py
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        if not nums:
            return [[]]

        def perm(indices: list[int], i: int) -> Iterator[list[int]]:
            if i == len(nums) - 1:
                yield [nums[j] for j in indices]
                return
            for j in range(i, len(nums)):
                indices[i], indices[j] = indices[j], indices[i]
                yield from perm(indices[:], i + 1)
                indices[i], indices[j] = indices[j], indices[i]

        indices = list(range(len(nums)))
        return list(perm(indices[:], 0))
```
