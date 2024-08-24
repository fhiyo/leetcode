# 31. Next Permutation

## 1st

コメント書く & 等号の有無でも間違えて詰まったので時間を使った。実務ではもうちょい具体的な例を出して動きを分かるようにするかも。

[アルゴリズムの考え方メモ]

nums[i+1:]が降順になっている限りnums[i+1:]の要素を使う順列の中では辞書順最大なので、そうでない (降順じゃなくなる) 点までindexのiを右に進める。そのようなindexが存在しなければnumsそのものが辞書順最大なので、その次は辞書順最小のものになるので単に反転させれば良い。

非負のiが求まった場合、次に辞書順として大きいものを探す。それはnums[:i+1]が「numsの要素全体を使って」次に大きい辞書順になり、nums[i+1:]が「交換された後のnums[i+1:]内の要素だけを使ったときの」辞書順最小 (昇順にソートされていれば良い) となれば求まる。

まずはnums[:i+1]の部分リストに対して考えると、次に大きいnums[:i+1]は「iの位置にnums[i]より大きいがnums[i+1:]の中で最小の要素 (その位置をjとする)」 を持ってくれば良い。
次にnums[i+1:]をnums[i+1:]内の要素を使って辞書順最小にする。これはソートしてもよいが、jの位置にnums[i]を交換して持ってくればnums[i+1:]が降順にソートされている状態を保てるので、反転させることで求めることもできる。

---

所要時間: 24:41

n: len(nums)
- 時間計算量: O(n)
- 空間計算量 (auxiliary): O(1)


```py
class Solution:
    def nextPermutation(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        if len(nums) <= 1:
            return
        i = len(nums) - 2
        while i >= 0 and nums[i] >= nums[i + 1]:
            i -= 1
        if i == -1:
            # nums is sorted descending order
            nums.reverse()
            return
        # i is the index which satisfies nums[i] < nums[i+1] and nums[i+1:] is sorted descending order
        j = len(nums) - 1
        while nums[i] >= nums[j]:
            j -= 1
        # j must be larger than i, because nums[i] < nums[i+1]
        # and nums[j] is the smallest value in nums[i+1:] satisfies nums[i] < nums[j]
        # so, the below process is the swapping nums[i] and the next larger value after i
        nums[i], nums[j] = nums[j], nums[i]
        # nums[i+1:] is sorted descending order, so reverse this list to compute next larger list lexcographically
        nums[i + 1:] = reversed(nums[i + 1:])
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1196472827457589338/1241295873837764650
  - https://github.com/Mike0121/LeetCode/pull/15

たしかに二重ループでもよい。辞書順で次を構築するという意味が分かりやすいと思う。

```py
class Solution:
    def nextPermutation(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        for i in range(len(nums) - 2, -1, -1):
            for j in range(len(nums) - 1, i, -1):
                if nums[i] >= nums[j]:
                    continue
                nums[i], nums[j] = nums[j], nums[i]
                nums[i + 1:] = reversed(nums[i + 1:])
                return
        nums.reverse()
```

- https://discord.com/channels/1084280443945353267/1225849404037009609/1232393910643589200
  - https://github.com/SuperHotDogCat/coding-interview/pull/8
- https://discord.com/channels/1084280443945353267/1201211204547383386/1232010197694939196
  - https://discord.com/channels/1084280443945353267/1201211204547383386/1232010197694939196
- https://discord.com/channels/1084280443945353267/1200089668901937312/1222836527156166707
  - https://github.com/hayashi-ay/leetcode/pull/67
- https://discord.com/channels/1084280443945353267/1210494002277908491/1211059642806042655
  - https://discord.com/channels/1084280443945353267/1210494002277908491/1211061690490425524

関数に切り出す箇所のイメージが浮かばなかったのでベタっと書いたが (それでも良いと思う)、C++の実装を見るとキレイな切り出し方があったのでそれを参考にして書いた。が、bisect_leftが明らかに分かりにくいのでこれも切り出した方がよかった。

ref
  - https://en.cppreference.com/w/cpp/algorithm/next_permutation


```py
class Solution:
    def nextPermutation(self, nums: List[int]) -> None:
        def is_reversed_sorted_until(seq: Sequence[int]) -> int:
            if len(seq) <= 1:
                return -1
            i = len(seq) - 2
            while i >= 0 and seq[i] >= seq[i + 1]:
                i -= 1
            return i

        left = is_reversed_sorted_until(nums)
        if left == -1:
            nums.reverse()
            return
        right = bisect_left(nums, -nums[left], left + 1, key=lambda x: -x) - 1
        nums[left], nums[right] = nums[right], nums[left]
        nums[left + 1:] = reversed(nums[left + 1:])
```

## 3rd

ある程度適切な粒度で関数に切り出せて見やすくなったかなと思うので3rdはこれにした。bisect_leftは最適化のトレードオフを考え、今回は保守性を取って使わない方向にした。二重ループも悪くないが、ある程度大きな入力を考えるならこちらでいきたい。


```py
class Solution:
    def nextPermutation(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        left = self._is_decending_order_from(nums)
        if left == -1:
            nums.reverse()
            return
        right = self._get_next_larger_index(nums, left)
        nums[left], nums[right] = nums[right], nums[left]
        nums[left + 1:] = reversed(nums[left + 1:])

    def _is_decending_order_from(self, seq: Sequence[int]) -> int:
        if len(seq) <= 1:
            return -1
        i = len(seq) - 2
        while i >= 0 and seq[i] >= seq[i + 1]:
            i -= 1
        return i

    def _get_next_larger_index(self, seq: Sequence[int], start: int) -> int:
        assert seq[start] < seq[start + 1] # start < len(seq) - 1
        i = len(seq) - 1
        while seq[start] >= seq[i]:
            i -= 1
        return i
```
