# 349. Intersection of Two Arrays

## 1st

### ①

素直に両方setにしてintersectionを取ることを思いついた。
`list(set(nums1) & set(nums2))` と書いても良い。

所要時間: 1:23

n1: len(nums1), n2: len(nums2)
- 時間計算量: 平均 O(n1 + n2), 最悪 O(unique(n1)*unique(n2))
    - 最悪は https://wiki.python.org/moin/TimeComplexity#set 参照。片方のsetのentryについて他方に入っているか確認するときに毎回最悪を引くとなるはず
- 空間計算量: O(unique(n1) + unique(n2))

```py
class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        return list(set(nums1).intersection(set(nums2)))
```

### ②

あえてintersection()を使わないならこうだろうか。

所要時間: 1:48

n1: len(nums1), n2: len(nums2)
- 時間計算量: 平均 O(n1 + n2), 最悪 O(unique(n1)*unique(n2))
- 空間計算量: O(unique(n1) + unique(n2))

```py
class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        larger = set(nums1)
        smaller = set(nums2)
        if len(larger) < len(smaller):
            larger, smaller = smaller, larger
        intersection = []
        for num in smaller:
            if num in larger:
                intersection.append(num)
        return intersection
```

### ③

set()も使わない場合はこういう感じだろうか。ソートするコストはあるが余分な空間を使わない。リストのサイズが高々1000なので十分候補になりそう。

所要時間: 9:11

n1: len(nums1), n2: len(nums2)
- 時間計算量: 平均 `O(n1*log(n1) + n2*log(n2))`
- 空間計算量: `O(1)`


```py
class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        nums1.sort()
        nums2.sort()
        index1 = 0
        index2 = 0
        intersection = []
        while index1 < len(nums1) and index2 < len(nums2):
            if nums1[index1] < nums2[index2]:
                index1 += 1
                continue
            elif nums1[index1] > nums2[index2]:
                index2 += 1
                continue
            # in case of nums1[index1] == nums2[index2]
            num = nums1[index1]
            intersection.append(num)
            while index1 < len(nums1) and nums1[index1] == num:
                index1 += 1
            while index2 < len(nums2) and nums2[index2] == num:
                index2 += 1
        return intersection
```


## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1226508154833993788/1246488449092161581
- https://discord.com/channels/1084280443945353267/1183683738635346001/1188146024565444689

片方だけsetにしてlistを舐めていく形を追加。

```py
class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        if len(nums1) > len(nums2):
            num_set = set(nums1)
            num_list = nums2
        else:
            num_set = set(nums2)
            num_list = nums1
        intersection = set()
        for num in num_list:
            if num in num_set:
                intersection.add(num)
        return list(intersection)
```

## 3rd

```py
class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        nums1.sort()
        nums2.sort()
        index1 = 0
        index2 = 0
        intersection = []
        while index1 < len(nums1) and index2 < len(nums2):
            if nums1[index1] < nums2[index2]:
                index1 += 1
                continue
            elif nums1[index1] > nums2[index2]:
                index2 += 1
                continue
            # in case of nums1[index1] == nums2[index2]
            num = nums1[index1]
            intersection.append(num)
            while index1 < len(nums1) and nums1[index1] == num:
                index1 += 1
            while index2 < len(nums2) and nums2[index2] == num:
                index2 += 1
        return intersection
```

