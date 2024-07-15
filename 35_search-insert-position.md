# 35. Search Insert Position

## 1st

### ①

標準ライブラリを使うだけ。

所要時間: 0:59

n: len(nums)
- 時間計算量: O(log n)
- 空間計算量: O(n)

```py
class Solution:
   def searchInsert(self, nums: List[int], target: int) -> int:
        return bisect_left(nums, target)
```

### ②

自前で二分探索を書く。 `nums[middle] == target` ならばreturn middleをしてもよいし、要求的にそちらの方が自然かも。

実装を確認: https://github.com/python/cpython/blob/94bee45dee41876e88fe023b9163178d376355dc/Lib/bisect.py


所要時間: 2:52

n: len(nums)
- 時間計算量: O(log n)
- 空間計算量: O(n)

```py
class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        left = 0
        right = len(nums)
        while left < right:
            middle = (left + right) // 2
            if nums[middle] < target:
                left = middle + 1
            else:
                right = middle
        return left
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1227073733844406343/1249743040852262972
  - https://github.com/sakupan102/arai60-practice/pull/42
- https://discord.com/channels/1084280443945353267/1233603535862628432/1236493345195298867
  - https://github.com/goto-untrapped/Arai60/pull/14
- https://discord.com/channels/1084280443945353267/1225849404037009609/1233119417479467019
  - https://github.com/SuperHotDogCat/coding-interview/pull/9/files
- https://discord.com/channels/1084280443945353267/1200089668901937312/1214540783990997044
  - https://github.com/hayashi-ay/leetcode/pull/40/files

省略

## 3rd

```py
class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        low = 0
        high = len(nums)
        while low < high:
            mid = (low + high) // 2
            if nums[mid] == target:
                return mid
            if nums[mid] < target:
                low = mid + 1
            else:
                high = mid
        return low
```
