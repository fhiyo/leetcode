# 1011. Capacity To Ship Packages Within D Days

## 1st

ある最大積載量の船で荷物を運んだら何日かかるか、を計算して、それがdays以内に収まるかをクエリすれば二分探索の形にできる。

daysが0の場合はassertで弾いた方がよかったかもしれない。

所要時間: 15:12

m: len(weights), n: sum(weights)
- 時間計算量: O(m*logn)
- 空間計算量: O(m)

```py
class Solution:
    def shipWithinDays(self, weights: List[int], days: int) -> int:
        if not weights:
            return 0

        def calculate_days(capacity: int) -> int:
            days = 0
            total_weight = 0
            for weight in weights:
                # precondition: weight <= capacity
                if total_weight + weight > capacity:
                    total_weight = 0
                    days += 1
                total_weight += weight
            return days + 1

        low = max(weights)
        high = sum(weights)
        while low < high:
            mid = (low + high) // 2
            days_with_capacity = calculate_days(mid)
            if days_with_capacity <= days:
                high = mid
            else:
                low = mid + 1
        return low
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1227073733844406343/1255121030138298470
  - https://github.com/sakupan102/arai60-practice/pull/45

can_be_shipped()を計算することで関数内でクエリの計算をする。この関数にするなら、あるweightがcapacityより大きいときはFalseを返すようにできる (このコードはlowをmax(weights)にしているが0でもよくなる)。

```py
class Solution:
    def shipWithinDays(self, weights: List[int], days: int) -> int:
        assert days > 0
        if not weights:
            return 0

        def can_be_shipped(capacity: int) -> bool:
            days_with_capacity = 0
            total_weight = 0
            for weight in weights:
                if weight > capacity:
                    return False
                if total_weight + weight > capacity:
                    total_weight = 0
                    days_with_capacity += 1
                    if days_with_capacity > days:
                        return False
                total_weight += weight
            return days_with_capacity + 1 <= days

        low = max(weights)
        high = sum(weights)
        while low < high:
            mid = (low + high) // 2
            if can_be_shipped(mid):
                high = mid
            else:
                low = mid + 1
        return low
```

bisect_left (rightでもいいが) 使ったらこうなるか。

```py
class Solution:
    def shipWithinDays(self, weights: List[int], days: int) -> int:
        assert days > 0
        if not weights:
            return 0

        def can_be_shipped(capacity: int) -> bool:
            total_weight = 0
            days_with_capacity = 0
            for weight in weights:
                if weight > capacity:
                    return False
                if total_weight + weight > capacity:
                    total_weight = 0
                    days_with_capacity += 1
                    if days_with_capacity > days:
                        return False
                total_weight += weight
            return days_with_capacity + 1 <= days

        return bisect_left(range(sum(weights) + 1), True, key=can_be_shipped)
```

- https://discord.com/channels/1084280443945353267/1225849404037009609/1248632267946070066
  - https://github.com/SuperHotDogCat/coding-interview/pull/27
- https://discord.com/channels/1084280443945353267/1192736784354918470/1236998781711683615
  - https://github.com/YukiMichishita/LeetCode/pull/10
- https://discord.com/channels/1084280443945353267/1201211204547383386/1226562256234614855
  - https://github.com/shining-ai/leetcode/pull/44
- https://discord.com/channels/1084280443945353267/1200089668901937312/1220683396213116979
  - https://github.com/hayashi-ay/leetcode/pull/55


## 3rd

```py
class Solution:
    def shipWithinDays(self, weights: List[int], days: int) -> int:
        assert days > 0
        if not weights:
            return 0

        def can_be_shipped(capacity: int) -> bool:
            total_weight = 0
            days_with_capacity = 0
            for weight in weights:
                if weight > capacity:
                    return False
                if total_weight + weight > capacity:
                    total_weight = 0
                    days_with_capacity += 1
                    if days_with_capacity > days:
                        return False
                total_weight += weight
            return days_with_capacity + 1 <= days

        low = 0
        high = sum(weights)
        while low < high:
            mid = (low + high) // 2
            if can_be_shipped(mid):
                high = mid
            else:
                low = mid + 1
        return low
```
