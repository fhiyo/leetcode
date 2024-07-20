# 3. Longest Substring Without Repeating Characters

## 1st

注目している文字列に含まれる文字の集合を管理し、その集合に次の文字をいれるときに重複が出るなら重複が出なくなるまで左側を進めて集合から削除をしていく。

文字->indexの辞書じゃなくてsetで良かった...

left, rightだと閉区間っぽい気がして気になるが、左側を開区間にしたいときに伝わりやすい変数名はあるだろうか？もちろん閉区間にしてright - left + 1と区間の長さを計算してもいいのだが。

`left = -1 # exclusive` みたいにとりあえずコメントを書くか。

所要時間: 8:16

n: len(s)
- 時間計算量: O(n)
- 空間計算量: O(n)

```py
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        char_to_index = {}
        max_length = 0
        left = -1
        for right in range(len(s)):
            if s[right] in char_to_index:
                while s[right] in char_to_index:
                    left += 1
                    del char_to_index[s[left]]
            max_length = max(max_length, right - left)
            char_to_index[s[right]] = right
        return max_length
```

setで書き直した。

```py
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        chars = set()
        left = -1
        max_length = 0
        for right in range(len(s)):
            while s[right] in chars:
                left += 1
                chars.remove(s[left])
            max_length = max(max_length, right - left)
            chars.add(s[right])
        return max_length
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1196472827457589338/1246368719148548117
  - https://github.com/Mike0121/LeetCode/pull/21

setってremove(), clear()だけじゃなく、discard()やpop()もあるのか。[Built-in Types — Python 3.12.4 documentation](https://docs.python.org/3.12/library/stdtypes.html#set)
discard(elem)はelemが無くてもエラーにならない、pop()は要素があるならelemを返す。

辞書にすると、管理にかかる計算コストが少し少なくなるのか。setにする方が素直だとは思うが。

```py
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        max_length = 0
        left = 0
        char_to_index = {}
        for right in range(len(s)):
            if s[right] in char_to_index and char_to_index[s[right]] >= left:
                left = char_to_index[s[right]] + 1
            max_length = max(max_length, right - left + 1)
            char_to_index[s[right]] = right
        return max_length
```

- https://discord.com/channels/1084280443945353267/1235971495696662578/1236898802703663208
  - https://github.com/sakzk/leetcode/pull/3

左端をloopで回す方法。

```py
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        max_length = 0
        chars = set()
        end = 0
        for begin in range(len(s)):
            while end < len(s) and s[end] not in chars:
                chars.add(s[end])
                end += 1
            max_length = max(max_length, end - begin)
            if end == len(s):
                break
            chars.remove(s[begin])
        return max_length
```

全探索するならこうするだろうか。

```py
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        max_length = 0
        for begin in range(len(s)):
            for end in range(begin + 1, len(s) + 1):
                if len(set(s[begin:end])) == end - begin:
                    max_length = max(max_length, end - begin)
        return max_length
```

endの範囲がlen(s) + 1までにしなくてはいけないことに気づかず1ミス。
TLEした。

毎回setを作るのがさすがに無駄かと思い、左端ごとにsetを作るようにする。こちらは通った。

```py
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        max_length = 0
        for begin in range(len(s)):
            chars = set()
            end = begin
            while end < len(s) and s[end] not in chars:
                chars.add(s[end])
                end += 1
            max_length = max(max_length, end - begin)
        return max_length
```

- https://discord.com/channels/1084280443945353267/1233295449985650688/1233598424780116039
  - https://github.com/Exzrgs/LeetCode/pull/2
- https://discord.com/channels/1084280443945353267/1230079550923341835/1233450465560363070
  - https://github.com/thonda28/leetcode/pull/6
- https://discord.com/channels/1084280443945353267/1226536545020809226/1228387150291144775
  - https://github.com/t0d4/leetcode/pull/2
- https://discord.com/channels/1084280443945353267/1225849404037009609/1229038363613331498
  - https://github.com/SuperHotDogCat/coding-interview/pull/3
- https://discord.com/channels/1084280443945353267/1226508154833993788/1228744387937177650
  - https://github.com/nittoco/leetcode/pull/3
- https://discord.com/channels/1084280443945353267/1201211204547383386/1228003299651879044
  - https://github.com/shining-ai/leetcode/pull/48
- https://discord.com/channels/1084280443945353267/1217527351890546789/1218817308328464405
  - https://github.com/cheeseNA/leetcode/pull/3
- https://discord.com/channels/1084280443945353267/1200089668901937312/1215986234812403752
  - https://github.com/hayashi-ay/leetcode/pull/47


## 3rd

```py
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        max_length = 0
        chars = set()
        left = -1 # exclusive
        for right in range(len(s)):
            while s[right] in chars:
                left += 1
                chars.remove(s[left])
            max_length = max(max_length, right - left)
            chars.add(s[right])
        return max_length
```
