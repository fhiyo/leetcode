# 560. Subarray Sum Equals K

## 1st

### ①

解法を覚えていた。i番目までの累積和を $s_{i+1}$ ($s_0 = 0$) として、s_j + k == s_i となるようなs_j ($j \in [0,..., i)$) が存在すれば合計がkとなる部分配列もそれに対応して存在する (i番目の要素がその部分配列の右端となる)。jの範囲はiを含まないので、s_jの探索時点でs_iが辞書に含まれないように注意する。

cumsum_countsはdefaultdictなので、辞書内にそのキーが存在しないとき新たにkey valueが追加されている点は少し気にはなる。解を出す際に問題にはならないが。

cumsum_countsはcumsum_to_countsの方がdictっぽいか。個人的にはtoが無くても十分分かるかなと思う。

所要時間: 4:46

n: len(nums)
- 時間計算量: O(n)
- 空間計算量: O(n)

```py
class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        sum_so_far = 0
        num_sum_is_k = 0
        cumsum_counts = defaultdict(int)
        cumsum_counts[0] = 1
        for num in nums:
            sum_so_far += num
            num_sum_is_k += cumsum_counts[sum_so_far - k]
            cumsum_counts[sum_so_far] += 1
        return num_sum_is_k
```

### ②

time limit exceededで通らないが、累積和を使った全探索。

n: len(nums)
- 時間計算量: O(n^2)
- 空間計算量: O(n)

```py
class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        num_sum_is_k = 0
        cumsum = [0]
        for num in nums:
            cumsum.append(cumsum[-1] + num)
        for i, cumsum_left in enumerate(cumsum):
            for cumsum_right in cumsum[i+1:]:
                if cumsum_right - cumsum_left == k:
                    num_sum_is_k += 1
        return num_sum_is_k
```

### ③

累積和使わない版。

n: len(nums)
- 時間計算量: O(n^2)
- 空間計算量: O(1)

```py
class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        num_sum_is_k = 0
        for i in range(len(nums)):
            total = 0
            for j in range(i, len(nums)):
                total += nums[j]
                if total == k:
                    num_sum_is_k += 1
        return num_sum_is_k
```


## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1227073733844406343/1233981980308148235
- https://discord.com/channels/1084280443945353267/1218823830743547914/1222533955798962197
- https://discord.com/channels/1084280443945353267/1200089668901937312/1210882278557884417

DP使ってるのがなるほどと思ったので、自分なりに書いてみた。
ついでにdefaultdict使わない版も練習で書いてみた

```py
class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        num_sum_is_k = 0
        prev_cumsum_counts = {}
        for num in nums:
            cumsum_counts = {}
            for cumsum in prev_cumsum_counts.keys():
                cumsum_counts[cumsum + num] = prev_cumsum_counts[cumsum]
            cumsum_counts[num] = cumsum_counts.get(num, 0) + 1
            num_sum_is_k += cumsum_counts.get(k, 0)
            prev_cumsum_counts = cumsum_counts
        return num_sum_is_k
```


## 3rd

コメントを書いた。

```py
class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        # sum(nums[:j]) + k == sum(nums[:i]) (j=0,...,i-1) となるjがx個存在すれば
        # 対応する部分配列 nums[j:i] もx個存在する
        sum_so_far = 0 # sum(nums[:i])
        num_sum_is_k = 0
        cumsum_counts = defaultdict(int) # { sum(nums[:j]): x }
        cumsum_counts[0] = 1 # sums(num[:0]) は1通り存在する
        for num in nums:
            sum_so_far += num
            num_sum_is_k += cumsum_counts[sum_so_far - k] # sum(nums[:i]) - k となるsum(nums[:j])の個数を足す
            cumsum_counts[sum_so_far] += 1
        return num_sum_is_k

```
