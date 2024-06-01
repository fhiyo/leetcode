# 1. Two Sum

## 1st

### ①

問題設定上 `return []` には到達しない。例外を返すか異常終了させるかも迷ったがこうしてみた。使われ方によって選択を変えるだろう。

remainingじゃなくてrestの方がより合ってる？そもそも形容詞だから微妙か。discord上の議論を見てcomplementもいいかもと思った。


所要時間: 6:26

n: len(nums)
- 時間計算量: O(n)
- 空間計算量: O(n)

```py
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        num_to_index = {}
        for i, num in enumerate(nums):
            remaining = target - num
            if remaining in num_to_index:
                return [num_to_index[remaining], i]
            num_to_index[num] = i
        return [] # never reached
```

### ②

全探索。

所要時間: 1:31

n: len(nums)
- 時間計算量: O(n^2)
- 空間計算量: O(1)

```py
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        for i in range(len(nums)):
            for j in range(i+1, len(nums)):
                if nums[i]+nums[j] == target:
                    return [i, j]
        return [] # never reached
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1227073733844406343/1231561279374757999
- https://discord.com/channels/1084280443945353267/1226508154833993788/1227088321537376296

    > > 一応、問題文ではそれ以外のものが来ない約束になっていますが、(約束が変わることというのはどうしてもあるので、)どうしたらより親切なコードかは考えておきましょう。
    > >
    > > Exception を投げる
    > > なんらかの特殊な値を返す
    > > プログラムが終了する
    > >
    > > のどれかが状況によって好まれます。
    >
    > > 今回の問題では特に影響はないと考えます。
    > もちろん、入出力が一致したコードを書くという目的ではこれでいいんですが、どう書くと読みやすくて理解しやすいか、どのような選択肢があって、どれを選んだのか、という視点で見て欲しいという話です。

- https://discord.com/channels/1084280443945353267/1217527351890546789/1218090301881581590
- https://discord.com/channels/1084280443945353267/1206101582861697046/1216454245004345465
- https://discord.com/channels/1084280443945353267/1200089668901937312/1204815592415301663

省略

## 3rd

```py
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        num_to_index = {}
        for i, num in enumerate(nums):
            rest = target - num
            if rest in num_to_index:
                return [num_to_index[rest], i]
            num_to_index[num] = i
        return [] # never reached
```
