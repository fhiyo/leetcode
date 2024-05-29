# 347. Top K Frequent Elements

## 1st

所要時間: 3:19

n: len(nums), m: unique(nums)
- 時間計算量: O(n + k*log(m)) (most_common()はnがNoneでなければheapq.nlargest()を実行する [コード](https://github.com/python/cpython/blob/3.12/Lib/collections/__init__.py#L618))
- 空間計算量: O(m)

Counterを知っていたのですぐ解けた。 `[num for num, _ in tally.most_common(k)][:k]` の方がいいか

```python
class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        tally = Counter(nums)
        return list(map(lambda t: t[0], tally.most_common(k)))
```

---

defaultdictを使った。何となくdataclassを使う。どうせならfrozen=Trueにしても良かったか。
`__lt__` だけの実装にしたが、確認したところ[pep8](https://peps.python.org/pep-0008/#programming-recommendations)では推奨していないらしい。

> When implementing ordering operations with rich comparisons, it is best to implement all six operations (__eq__, __ne__, __lt__, __le__, __gt__, __ge__) rather than relying on other code to only exercise a particular comparison.
> To minimize the effort involved, the functools.total_ordering() decorator provides a tool to generate missing comparison methods.

一方 functools.total_ordering() も速度やスタックトレースが複雑になるというデメリットがある模様。 https://docs.python.org/3/library/functools.html#functools.total_ordering

> While this decorator makes it easy to create well behaved totally ordered types, it does come at the cost of slower execution and more complex stack traces for the derived comparison methods. If performance benchmarking indicates this is a bottleneck for a given application, implementing all six rich comparison methods instead is likely to provide an easy speed boost.


所要時間: 8:11

n: len(nums), m: unique(nums)
- 時間計算量: O(n + mlog(m))
- 空間計算量: O(m)

```python
from dataclasses import dataclass

@dataclass
class NumWithFreq:
    num: int
    freq: int

    def __lt__(self, other):
        return self.freq < other.freq

class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        tally = defaultdict(int)
        for num in nums:
            tally[num] += 1
        return list(map(
            lambda num_with_freq: num_with_freq.num,
            sorted(map(lambda t: NumWithFreq(*t), tally.items()), reverse=True)[:k]
        ))
```

---
quick selectによる解法

所要時間: 49:15

バグが全く取れず苦戦した。namedtupleで定義したNumWithFreqがコードに残っており、そのせいで比較がおかしくなっていた。

n: len(nums), m: unique(nums)
- 時間計算量: 平均O(m)
- 空間計算量: O(m)

```python
from dataclasses import dataclass

@total_ordering
@dataclass(frozen=True)
class NumWithFreq:
    num: int
    freq: int

    def __lt__(self, other):
        return self.freq < other.freq

    def __eq__(self, other):
        return self.freq == other.freq


class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        assert len(nums) > 0
        tally = defaultdict(int)
        for num in nums:
            tally[num] += 1
        num_with_freqs = list(map(lambda t: NumWithFreq(*t), tally.items()))
        left = 0
        right = len(num_with_freqs) - 1
        while left <= right:
            pivot = self._partition(num_with_freqs, left, right)
            if pivot == k - 1:
                return list(map(lambda e: e.num, num_with_freqs))[:k]
            if pivot < k - 1:
                left = pivot + 1
            else:
                right = pivot - 1
        return [] # never reached

    def _partition(self, values: List[NumWithFreq], left: int, right: int) -> int:
        pivot = self._getPivot(values, left, right)
        values[pivot], values[right] = values[right], values[pivot]
        i = left - 1
        for j in range(left, right):
            # descendent order
            if values[j] > values[right]:
                i += 1
                values[i], values[j] = values[j], values[i]
        values[i+1], values[right] = values[right], values[i+1]
        return i + 1

    # 3点の真ん中の値を返す
    def _getPivot(self, values: List[NumWithFreq], left: int, right: int) -> int:
        mid = (left + right) // 2
        if values[left] <= values[mid]:
            if values[mid] <= values[right]:
                return mid
            elif values[left] <= values[right]:
                return right
            return left
        # case of values[mid] < values[left]
        if values[left] <= values[right]:
            return left
        elif values[mid] <= values[right]:
            return right
        return mid
```


## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1231966485610758196/1243547482169016341
- https://github.com/cheeseNA/leetcode/pull/13
- https://github.com/hayashi-ay/leetcode/pull/60

略

## 3rd

略
