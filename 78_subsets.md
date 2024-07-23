# 78. Subsets

## 1st

### ①

あるindexの要素について入れる or 入れないで分岐させながらindexを進めていけば良い。

所要時間: 6:11

n: len(nums)
- 時間計算量: O(n * 2**n)
- 空間計算量: O(2**n)

```py
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        def subsets_helper(subset: list[int], index: int) -> Iterator[list[int]]:
            if index == len(nums):
                yield subset
                return
            yield from subsets_helper(subset[:], index + 1)
            yield from subsets_helper(subset + [nums[index]], index + 1)

        return list(subsets_helper([], 0))
```

### ②

bitパターンを使って列挙する。 `2 ** len(nums)` は `1 << len(nums)` でもいい。

所要時間: 5:52

n: len(nums)
- 時間計算量: O(n * 2**n)
- 空間計算量: O(2**n)

```py
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        def subsets_helper() -> Iterator[list[int]]:
            for i in range(2 ** len(nums)):
                subset = []
                for j in range(i.bit_length()):
                    if (i >> j) & 1:
                        subset.append(nums[j])
                yield subset

        return list(subsets_helper())
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1233603535862628432/1263109149932261417
  - https://github.com/goto-untrapped/Arai60/pull/39


backtracking。yieldするときだけリストのコピーを作っている。

```py
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        def generate_subsets(subset: list[int], index: int) -> Iterator[list[int]]:
            if index == len(nums):
                yield subset[:]
                return
            subset.append(nums[index])
            yield from generate_subsets(subset, index + 1)
            subset.pop()
            yield from generate_subsets(subset, index + 1)

        return list(generate_subsets([], 0))
```

ループで回してsubsetに入れる要素を選んでいく。indexが末尾に到達する前にyieldしているのが個人的には分かりにくい。index以降の要素を入れないと決めていると考えれば分かるか。

```py
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        def generate_subsets(subset: list[int], index: int) -> Iterator[list[int]]:
            yield subset[:]
            for i in range(index, len(nums)):
                subset.append(nums[i])
                yield from generate_subsets(subset, i + 1)
                subset.pop()

        return list(generate_subsets([], 0))
```

空のリストから始め、今までのsubsetsにnums[i]を入れるパターンのものを追加していくことで全列挙する。

```py
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        all_subsets = [[]]
        for num in nums:
            new_subsets = []
            for subset in all_subsets:
                new_subsets.append(subset + [num])
            all_subsets.extend(new_subsets)
        return all_subsets
```

スタックに入れてDFS。

```py
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        def generate_subsets() -> Iterator[list[int]]:
            stack = [([], 0)]
            while stack:
                subset, index = stack.pop()
                if index == len(nums):
                    yield subset
                    continue
                stack.append((subset[:], index + 1))
                stack.append((subset + [nums[index]], index + 1))

        return list(generate_subsets())
```

- https://discord.com/channels/1084280443945353267/1226508154833993788/1250813107690012712
  - https://github.com/nittoco/leetcode/pull/19
- https://discord.com/channels/1084280443945353267/1225849404037009609/1237822530753138780
  - https://github.com/SuperHotDogCat/coding-interview/pull/18
- https://discord.com/channels/1084280443945353267/1201211204547383386/1229482326967189667
  - https://github.com/shining-ai/leetcode/pull/51
- https://discord.com/channels/1084280443945353267/1218823830743547914/1231089205116141588
  - https://github.com/ryoooooory/LeetCode/pull/5
- https://discord.com/channels/1084280443945353267/1200089668901937312/1221804658838999070
  - https://github.com/hayashi-ay/leetcode/pull/63


あえて[itertools.combinations()](https://docs.python.org/3.11/library/itertools.html#itertools.combinations)を実装して解く。

```py
T = TypeVar('T')

class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        all_subsets = []
        for i in range(len(nums) + 1):
            all_subsets.extend(map(lambda t: list(t), self._my_combinations(nums, i)))
        return all_subsets

    def _my_combinations(self, iterable: Iterable[T], r: int) -> Iterable[tuple[T]]:
        pool = tuple(iterable)
        n = len(pool)
        if r > n:
            return
        indices = list(range(r))
        yield tuple(pool[i] for i in indices)
        while True:
            for i in reversed(range(r)):
                if indices[i] != i + n - r:
                    break
            else:
                return
            indices[i] += 1
            for j in range(i+1, r):
                indices[j] = indices[j-1] + 1
            yield tuple(pool[i] for i in indices)
```

bitパターンで解く方法は[itertools.compress()](https://docs.python.org/3.11/library/itertools.html#itertools.compress)を使っても良い。
`f"{i:0{len(nums)}b}"` はちょっと汚いが...
reversedは不要だが、わかり易さ優先で付けた。

```py
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        def generate_subsets() -> Iterator[list[int]]:
            for i in range(2 ** len(nums)):
                selector = map(lambda b: b == '1', reversed(f"{i:0{len(nums)}b}"))
                yield compress(nums, selector)

        return list(generate_subsets())
```


## 3rd

```py
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        def generate_subsets(subset: list[int], index: int) -> Iterator[list[int]]:
            if index == len(nums):
                yield subset
                return
            yield from generate_subsets(subset[:], index + 1)
            yield from generate_subsets(subset + [nums[index]], index + 1)

        return list(generate_subsets([], 0))
```
