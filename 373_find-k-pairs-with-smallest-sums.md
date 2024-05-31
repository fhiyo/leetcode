# 373. Find K Pairs with Smallest Sums

## 1st

### ①

所要時間: 7:30

ごく最近解いており記憶に残っていたためすぐに解けたが、前解いたときはもっとかかっていた (時間は測っていないが数十分くらい？)。
ValueWithIndexesはどういう値なのか分かりにくいと感じたのでコメントで補足をした。
smallest_tuplesも名前微妙かな...

i行j列目の値がnums1[i]+nums2[j]となるようなテーブルを考える。
最初に(0,0)と対応する合計値をMinHeapに入れ、loopで取り出した(i,j)について

1. i == 0 なら次のi+1番目の行が次点で小さい候補に初めて入ってくるので、(i+1,0)の要素をMinHeapに入れられるなら入れる
2. (i,j+1)の要素が次点で小さい候補になるのでそれをMinHeapに入れられるなら入れる

をk回繰り返せば良い。「入れられるなら」はテーブルからはみ出なければ、という意味。


- 時間計算量: O(k)
- 空間計算量: O(min(k, len(nums1)))
    - index1 < len(nums1) - 1 and index2 == 0 and index2 < len(nums2) - 1 のときだけheapのサイズが1だけ増え、そのときindex1がインクリメントされるから。
    - あまり自信がない


```py
class Solution:
    def kSmallestPairs(self, nums1: List[int], nums2: List[int], k: int) -> List[List[int]]:
        assert len(nums1) > 0 and len(nums2) > 0
        # (nums1[i] + nums2[j], i, j)
        ValueWithIndexes = namedtuple('ValueWithIndexes', ['value', 'index1', 'index2'])
        smallest_tuples = [ValueWithIndexes(nums1[0] + nums2[0], 0, 0)]
        k_smallest_pairs = []
        for _ in range(k):
            value, index1, index2 = heapq.heappop(smallest_tuples)
            k_smallest_pairs.append([nums1[index1], nums2[index2]])
            if index1 < len(nums1) - 1 and index2 == 0:
                heapq.heappush(smallest_tuples, ValueWithIndexes(nums1[index1+1] + nums2[0], index1 + 1, 0))
            if index2 < len(nums2) - 1:
                heapq.heappush(smallest_tuples, ValueWithIndexes(nums1[index1] + nums2[index2+1], index1, index2 + 1))
        return k_smallest_pairs
```

### ②

時間もメモリも余裕がある状況なら全探索しても良い。

所要時間: 約4分 (Memory Limit Exceeded)

```py
class Solution:
    def kSmallestPairs(self, nums1: List[int], nums2: List[int], k: int) -> List[List[int]]:
        # (nums1[i] + nums2[j], nums1[i], nums2[j])
        TotalAndValues = namedtuple('TotalAndValues', ['total', 'value1', 'value2'])
        candidates = []
        for num1 in nums1:
            for num2 in nums2:
                heapq.heappush(candidates, TotalAndValues(num1+num2, num1, num2))
        k_smallest_values = []
        for _ in range(k):
            _, value1, value2 = heapq.heappop(candidates)
            k_smallest_values.append([value1, value2])
        return k_smallest_values
```



## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1196472827457589338/1245390479172833342
  - 値を負にすることでMaxHeapにして、heapのサイズをk個に維持してsmallest_kを取っている。自分の②よりメモリ効率がいいし素直な気がする。こんな感じか↓

    ```py
    class Solution:
        def kSmallestPairs(self, nums1: List[int], nums2: List[int], k: int) -> List[List[int]]:
            assert len(nums1) > 0 and len(nums2) > 0
            # (-(nums1[i]+nums2[i]), i, j)
            NegativeTotalAndValues = namedtuple('NegativeTotalAndValues', ['negative_total', 'value1', 'value2'])
            heap = [] # 追記: heapにしちゃったがcandidatesくらいが良いか
            for num1 in nums1:
                for num2 in nums2:
                    values = NegativeTotalAndValues(-(num1+num2), num1, num2)
                    if len(heap) < k:
                        heapq.heappush(heap, values)
                    else:
                        heapq.heappushpop(heap, values)
            return [[num1, num2] for _, num1, num2 in heap]
    ```
- https://discord.com/channels/1084280443945353267/1227073733844406343/1233792341173866609
  - https://github.com/sakupan102/arai60-practice/pull/11#discussion_r1578893854
    > while len(minumum_num_pairs) < kの方が素直だと思うんですよね。「k回ループを回す、そうするとminumum_num_pairsの要素数がk個になる」より「minumum_num_pairsの要素数がk個になるまでループを回す」の方が素直だと思うんですよね。

    うーむなるほど。自分の①だったら `for _ in range(k)` の方がk回回してそのたびに要素を追加することが分かると思うので、ループの回し方が素直な方がいいかな...？と感じた。ループの中がもう少し複雑な処理ならwhileにする気がする。

- https://discord.com/channels/1084280443945353267/1200089668901937312/1222557634092073051

## 3rd

所要時間: 6:09

```py
class Solution:
    def kSmallestPairs(self, nums1: List[int], nums2: List[int], k: int) -> List[List[int]]:
        assert len(nums1) > 0 and len(nums2) > 0
        # (nums1[i]+nums2[j], i, j)
        TotalAndIndexes = namedtuple('TotalAndIndexes', ['total', 'index1', 'index2'])
        candidates = [TotalAndIndexes(nums1[0] + nums2[0], 0, 0)]
        k_smallest_pairs = []
        for _ in range(k):
            _, index1, index2 = heapq.heappop(candidates)
            k_smallest_pairs.append([nums1[index1], nums2[index2]])
            if index1 < len(nums1) - 1 and index2 == 0:
                heapq.heappush(candidates, TotalAndIndexes(nums1[index1+1] + nums2[0], index1 + 1, 0))
            if index2 < len(nums2) - 1:
                heapq.heappush(candidates, TotalAndIndexes(nums1[index1] + nums2[index2+1], index1, index2 + 1))
        return k_smallest_pairs
```
