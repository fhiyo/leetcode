# 139. Word Break

## 1st

### ①

s[:i]までが分割出来ているとき、s[i:j]がwordDictに含まれていればs[:j]まで分割できる。
find_end_indexs_of_wordはもう少し良い関数名が欲しかった。
[startswith](https://docs.python.org/ja/3.12/library/stdtypes.html#str.startswith) には第二引数としてstart_indexがあったので、suffixのような変数を作らなくて良かった。

breakableはその変数名だけだと何を表しているか分からない。処理を見ればそこまで読み解くのは苦労しないとは思うが。
`# breakable[i]: s[:i] can be broken into the subset of wordDict` みたいなコメントを書くのがいいかなと後で思った。

所要時間: 13:13

n: len(s), m: len(wordDict), l: mean(len(wordDict[i]))
- 時間計算量: O(n^2 * ml)
- 空間計算量: O(n)

```py
class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        def find_end_indexs_of_word(start_index: int) -> Iterator[int]:
            suffix = s[start_index:]
            for word in wordDict:
                if suffix.startswith(word):
                    yield start_index + len(word)

        breakable = [False] * (len(s) + 1)
        breakable[0] = True
        for i in range(len(s)):
            if not breakable[i]:
                continue
            for end_index in find_end_indexs_of_word(i):
                breakable[end_index] = True
        return breakable[-1]
```

### ②

メモ化再帰。やってること考えると `breakable_from(index: int)` の方がいいだろうか

所要時間: 5:17

n: len(s), m: len(wordDict), l: mean(len(wordDict[i]))
- 時間計算量: O(nml)
- 空間計算量: O(n) (高々sの長さ分しか再帰しない)

```py
class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        @cache
        def breakable(start_index: int) -> bool:
            if start_index == len(s):
                return True
            result = False
            for word in wordDict:
                if s.startswith(word, start_index):
                    result |= breakable(start_index + len(word))
            return result

        return breakable(0)
```

### ③

非再帰のDFS.breakable_indexesは使用済みのindexたちなので、usedで十分表せそう。stにしてしまったスタックの方をbreakable_indexesにすればよいか。

もっと分かりやすい変数名 or コメントを書く があるかも。

所要時間: 7:37

n: len(s), m: len(wordDict), l: mean(len(wordDict[i]))
- 時間計算量: O(nml)
- 空間計算量: O(n)

```py
class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        st = [0]
        breakable_indexes = set()
        while st:
            breakable_index = st.pop()
            if breakable_index == len(s):
                return True
            if breakable_index in breakable_indexes:
                continue
            breakable_indexes.add(breakable_index)
            for word in wordDict:
                if s.startswith(word, breakable_index):
                    st.append(breakable_index + len(word))
        return False
```

### ④

BFS。

```py
class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        breakable_indexes = deque([0])
        used = set()
        while breakable_indexes:
            breakable_index = breakable_indexes.popleft()
            if breakable_index == len(s):
                return True
            if breakable_index in used:
                continue
            used.add(breakable_index)
            for word in wordDict:
                if s.startswith(word, breakable_index):
                    breakable_indexes.append(breakable_index + len(word))
        return False
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1227073733844406343/1249332985199726663
- https://discord.com/channels/1084280443945353267/1225849404037009609/1245020539479916596
- https://discord.com/channels/1084280443945353267/1233603535862628432/1244667824564080741
- https://discord.com/channels/1084280443945353267/1233295449985650688/1239804259973857400
- https://discord.com/channels/1084280443945353267/1201211204547383386/1224731898622771371

正規表現やローリングハッシュでもTrieでもできるという話。まあそうだろうという気はする。

問題の制約的にはwordDictでloopを回すよりもsの部分文字列がwordDictの中に含まれているかを調べる方が良さそう。ただし文字列のコピーが発生する。
min_word_length, max_word_lengthは自前でループを回して同時に求めてもよいが、組み込みの関数の方が2回wordDictを舐めてもまだ速いだろうと思いこうしている。
range_start, range_endは一応無駄なループを回さないように計算している。

```py
class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        min_word_length = len(min(wordDict, key=len))
        max_word_length = len(max(wordDict, key=len))
        words = set(wordDict)
        # breakable[i]: s[:i+1] can be broken into the subset of words
        breakable = [False] * (len(s) + 1)
        breakable[0] = True
        for end in range(1, len(s) + 1):
            range_start = max(0, end - max_word_length)
            range_end = max(0, end - min_word_length + 1)
            for start in range(range_start, range_end):
                if not breakable[start]:
                    continue
                if s[start:end] in words:
                    breakable[end] = True
        return breakable[-1]
```

- https://discord.com/channels/1084280443945353267/1200089668901937312/1221441289254342666

②の別バージョン

```py
class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        @cache
        def breakable_from(start_index: int) -> bool:
            if start_index == len(s):
                return True
            for word in wordDict:
                if (s.startswith(word, start_index)
                    and breakable_from(start_index + len(word))):
                    return True
            return False

        return breakable_from(0)
```

参考にしてTrieで書いてみた。

```py
class TrieNode:
    def __init__(self):
        self.children: dict[str, TrieNode] = {}
        self.active: bool = False


class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str):
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.active = True

    def enumerate_prefix_words(self, s: str) -> Iterator[str]:
        # enumerate the words in the Trie that are prefixes of s
        node = self.root
        word_as_list = []
        for c in s:
            if node.active:
                yield ''.join(word_as_list)
            if c not in node.children:
                return
            word_as_list.append(c)
            node = node.children[c]
        if node.active:
            yield ''.join(word_as_list)


class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        trie = self._build_trie(wordDict)

        @cache
        def breakable(s: str) -> bool:
            if not s:
                return True
            for word in trie.enumerate_prefix_words(s):
                if breakable(s[len(word):]):
                    return True
            return False

        return breakable(s)

    def _build_trie(self, words: Iterable[str]) -> TrieNode:
        trie = Trie()
        for word in words:
            trie.insert(word)
        return trie
```


## 3rd

```py
class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        # breakable[i]: whether s[:i] can be broken into words that are included in wordDict
        breakable = [False] * (len(s) + 1)
        breakable[0] = True
        for i in range(len(s)):
            if not breakable[i]:
                continue
            for word in wordDict:
                if s.startswith(word, i):
                    breakable[i + len(word)] = True
        return breakable[-1]
```
