# 39. Combination Sum

## 1st

### ①

先にcandidatesをsortしておけば、total+candidates[i] > target となったときにi以降の要素を見る必要はないのでサボれる。
yield するタイミングはループ内でも良いが、自分の中で処理が分かりやすかったので外出しした (ループ内で行う方がtargetが0でない限り効率的ではある)

所要時間: 10:55

n: len(candidates)
- 時間計算量: O((target/min(candidates))^n) (緩いかも)
- 空間計算量: O(target/min(candidates))

```py
class Solution:
    def combinationSum(self, candidates: List[int], target: int) -> List[List[int]]:
        candidates.sort()

        def generate_combinations(combination: list[int], total: int, start: int) -> Iterator[list[int]]:
            if total == target:
                yield combination
                return
            for i in range(start, len(candidates)):
                if total + candidates[i] > target:
                    break # candidates are sorted in ascending order, so total+candidates[j] > target, if j >= i
                yield from generate_combinations(combination + [candidates[i]], total + candidates[i], i)

        return list(generate_combinations([], 0, 0))
```

### ②

yieldを使わない版。

所要時間: 3:48

n: len(candidates)
- 時間計算量: O((target/min(candidates))^n) (緩いかも)
- 空間計算量: O(target/min(candidates))

```py
class Solution:
    def combinationSum(self, candidates: List[int], target: int) -> List[List[int]]:
        candidates.sort()
        all_combinations = []

        def make_combinations(combination: list[int], total: int, start: int):
            for i in range(start, len(candidates)):
                if total + candidates[i] > target:
                    break
                if total + candidates[i] == target:
                    all_combinations.append(combination + [candidates[i]])
                    break
                make_combinations(combination + [candidates[i]], total + candidates[i], i)

        make_combinations([], 0, 0)
        return all_combinations
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1226508154833993788/1260595147683528754
  - https://github.com/nittoco/leetcode/pull/25

indexの要素を使わない場合と使う場合で分けて再帰する。

```py
class Solution:
    def combinationSum(self, candidates: List[int], target: int) -> List[List[int]]:
        def generate_combinations(combination: list[int], total: int, index: int) -> Iterator[list[int]]:
            if index == len(candidates):
                return
            yield from generate_combinations(combination, total, index + 1)
            if total + candidates[index] == target:
                yield combination + [candidates[index]]
                return
            if total + candidates[index] < target:
                yield from generate_combinations(combination + [candidates[index]], total + candidates[index], index)

        return list(generate_combinations([], 0, 0))
```

スタックを使用。

```py
class Solution:
    def combinationSum(self, candidates: List[int], target: int) -> List[List[int]]:
        candidates.sort()
        all_combinations = []
        stack = [([], 0, 0)]
        while stack:
            combination, total, start = stack.pop()
            if total == target:
                all_combinations.append(combination)
            for i in range(start, len(candidates)):
                if total + candidates[i] > target:
                    break
                stack.append((combination + [candidates[i]], total + candidates[i], i))
        return all_combinations
```

- https://discord.com/channels/1084280443945353267/1233295449985650688/1242109180844703754
  - https://github.com/Exzrgs/LeetCode/pull/13
- https://discord.com/channels/1084280443945353267/1233603535862628432/1236493345195298867
  - https://github.com/goto-untrapped/Arai60/pull/15
- https://discord.com/channels/1084280443945353267/1225849404037009609/1234193259832672416
  - https://github.com/SuperHotDogCat/coding-interview/pull/11
- https://discord.com/channels/1084280443945353267/1196472827457589338/1232678133665370163
  - https://github.com/Mike0121/LeetCode/pull/1
- https://discord.com/channels/1084280443945353267/1201211204547383386/1229854274184544357
  - https://github.com/shining-ai/leetcode/pull/52
- https://discord.com/channels/1084280443945353267/1200089668901937312/1222146483751485440
  - https://github.com/hayashi-ay/leetcode/pull/65


## 3rd

自分的にはyieldで書くのが分かりやすい...この問題では返り値がlistだから旨味はないが、lazyに計算できるのでメモリ効率も本来なら良いし。

```py
class Solution:
    def combinationSum(self, candidates: List[int], target: int) -> List[List[int]]:
        candidates.sort()

        def generate_combinations(combination: list[int], total: int, start: int) -> Iterator[list[int]]:
            if total == target:
                yield combination
                return
            for i in range(start, len(candidates)):
                if total + candidates[i] > target:
                    break # candidates are sorted in ascending order, so total+candidates[j] > target if j >= i
                yield from generate_combinations(combination + [candidates[i]], total + candidates[i], i)

        return list(generate_combinations([], 0, 0))
```
