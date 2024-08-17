# 283. Move Zeroes

## 1st

### ①

非ゼロの要素を別のリストに避けておいてnumsを上書きする。
最初バブルソートっぽくO(n^2)でやろうとしたらTime Limit Exceededになり、昔この問題をやったときには通った記憶があったのでそこでなぜだと思い時間を使ってしまった (実際にそれで通したのはC++だった)。

所要時間: 14:32

n: len(nums)
- 時間計算量: O(n)
- 空間計算量: O(n)

```py
class Solution:
    def moveZeroes(self, nums: List[int]) -> None:
        nonzeros = []
        for num in nums:
            if num != 0:
                nonzeros.append(num)
        for i in range(len(nonzeros)):
            nums[i] = nonzeros[i]
        for i in range(len(nonzeros), len(nums)):
            nums[i] = 0
```

### ②

クイックソートのpartitionアルゴリズムに似た解法。コード内のコメントにあるのがループ不変条件で、非ゼロの要素を (順序を保ったまま) すべて左に持ってくるようにしているので解になる。

所要時間: 5:05

n: len(nums)
- 時間計算量: O(n)
- 空間計算量: O(1)

```py
class Solution:
    def moveZeroes(self, nums: List[int]) -> None:
        last_nonzero_index = 0 # any elements in nums[:last_nonzero_index] are not 0
        for i in range(len(nums)):
            if nums[i] != 0:
                nums[last_nonzero_index], nums[i] = nums[i], nums[last_nonzero_index]
                last_nonzero_index += 1
```

### ③

例外処理の使い方が気持ち悪いが一応こうも書ける。
ループを回してpop()を使っても良い。そちらの場合try-exceptは使わず書ける。その上remove()と違って毎回頭から探索しないから、あえて書くならpop()を使う方がいいな。

所要時間: 1:16

n: len(nums)
- 時間計算量: O(n^2)
- 空間計算量: O(1)

```py
class Solution:
    def moveZeroes(self, nums: List[int]) -> None:
        num_nonzeros = 0
        while True:
            try:
                nums.remove(0)
                num_nonzeros += 1
            except ValueError:
                break
        nums.extend([0] * num_nonzeros)
```

`list.remove()` を確認した。 https://github.com/python/cpython/blob/35d8ac7cd7ed6cd3d84af721dce970da59bd5f68/Objects/listobject.c#L3260

ループでリストの頭から各要素をremoveの引数であるvalueと比較し、等しかったら[list_ass_slice_lock_held()](https://github.com/python/cpython/blob/35d8ac7cd7ed6cd3d84af721dce970da59bd5f68/Objects/listobject.c#L833)を実行してリストからその要素を削除して終了。リスト内にvalueが存在しなければValueErrorを送出する。


## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1226508154833993788/1250091281246453800
  - https://github.com/nittoco/leetcode/pull/18

①の解法は別のリストを用意しなくても良い。こうするとほとんど②と同じになる。

```py
class Solution:
    def moveZeroes(self, nums: List[int]) -> None:
        last_nonzero_index = 0
        for num in nums:
            if num == 0:
                continue
            nums[last_nonzero_index] = num
            last_nonzero_index += 1
        for i in range(last_nonzero_index, len(nums)):
            nums[i] = 0
```

- https://discord.com/channels/1084280443945353267/1239148130679783424/1249699208773107733
  - https://github.com/goto-untrapped/Arai60/pull/25

last_nonzero_indexとそこよりも右にある初めての非ゼロの要素を交換し続ける。考えてることは②と大体同じだが手順が違う。ごちゃついてるし②の方がいいだろう (書き方が悪いだけかもしれないが)。

```py
class Solution:
    def moveZeroes(self, nums: List[int]) -> None:
        next_nonzero_index = -1
        last_nonzero_index = 0 # any elements in nums[:last_nonzero_index] are not 0
        while True:
            while last_nonzero_index < len(nums) and nums[last_nonzero_index] != 0:
                last_nonzero_index += 1
            next_nonzero_index = max(next_nonzero_index, last_nonzero_index)
            while next_nonzero_index < len(nums) and nums[next_nonzero_index] == 0:
                next_nonzero_index += 1
            if next_nonzero_index == len(nums):
                break
            nums[last_nonzero_index], nums[next_nonzero_index] = nums[next_nonzero_index], nums[last_nonzero_index]
            last_nonzero_index += 1
```

- https://discord.com/channels/1084280443945353267/1196472827457589338/1248287760255684722
  - https://github.com/Mike0121/LeetCode/pull/24
- https://discord.com/channels/1084280443945353267/1225849404037009609/1246132840152699021
  - https://github.com/SuperHotDogCat/coding-interview/pull/25
- https://discord.com/channels/1084280443945353267/1201211204547383386/1230566847058018344
  - https://github.com/shining-ai/leetcode/pull/54
- https://discord.com/channels/1084280443945353267/1200089668901937312/1221011776188059668
  - https://github.com/hayashi-ay/leetcode/pull/58
- https://discord.com/channels/1084280443945353267/1210494002277908491/1211045822012072006


## 3rd

```py
class Solution:
    def moveZeroes(self, nums: List[int]) -> None:
        last_nonzero_index = 0 # any elements in nums[:last_nonzero_index] are not 0
        for i in range(len(nums)):
            if nums[i] == 0:
                continue
            nums[last_nonzero_index], nums[i] = nums[i], nums[last_nonzero_index]
            last_nonzero_index += 1
```
