# 127. Word Ladder

## 1st

### ①

DFSでやろうとして時間制限を超えてやり直すなどでかなり時間がかかった。エッジの重みがすべて1のグラフ上の最短距離を求めるのだからBFSがまず候補に入るはず。
DFSを書くのにも動く状態になるまで40分以上かけている。問題を見たときに自然な発想ができないのと思いついた方法を検討したときに上手くいかない or 計算量などで問題があることを速く思いつければいいのだが...

他の方のコードを見て思ったが、幅優先探索している意識が強いのでqueueを使っているが、このコードだとword_queue, next_word_queueはキューじゃなくてもリストなどでも良い。

snake caseとcamel caseが混じってるのは少し気になるが、問題と自分の書き方の都合。何なら与えられたコードの引数を変えればいい。

所要時間: 1:01:27

n: len(wordList), m: wordListに含まれるwordとbeginWordの長さの平均, l: 出現しうる文字の数 (今回はa-zで定数)
- 時間計算量: O(nml)
- 空間計算量: O(nm)

```py
class Solution:
    def ladderLength(self, beginWord: str, endWord: str, wordList: List[str]) -> int:
        words = set(wordList)
        num_words = 1
        word_queue = deque([beginWord])
        while word_queue:
            next_word_queue = deque()
            while word_queue:
                word = word_queue.popleft()
                if word == endWord:
                    return num_words
                for next_word in self._getNextWords(word, words):
                    next_word_queue.append(next_word)
            word_queue = next_word_queue
            num_words += 1
        return 0

    def _getNextWords(self, word: str, words: set[str]) -> Generator[str, None, None]:
        for i, original_ch in enumerate(word):
            for ch in ascii_lowercase:
                if ch == original_ch:
                    continue
                new_word = f'{word[:i]}{ch}{word[i + 1:]}'
                if new_word in words:
                    words.remove(new_word)
                    yield new_word
```

### ②

次のwordを見つけるのにハミング距離を用いた解法。問題の制約上、len(wordList)<=5000なのでwordListでループを回すと時間がかかる。①はlen(word)でループを回しており、こちらは <=10なので速い。
この解法で通ったときは8278msだったので、たまたま通っただけだろう。
get_distanceは `return sum(a[i] != b[i] for i in range(len(a)))` や `return sum(ai != bi for ai, bi in zip(a, b))` のようにsumを使っても良いと思う。

所要時間: 14:56

n: len(wordList), m: wordListに含まれるwordとbeginWordの長さの平均,
- 時間計算量: O(n^2 * m)
- 空間計算量: O(nm)

```py
class Solution:
    def ladderLength(self, beginWord: str, endWord: str, wordList: List[str]) -> int:
        words = set(wordList)
        word_queue = deque([beginWord])
        num_words = 1
        while word_queue:
            next_word_queue = deque()
            while word_queue:
                word = word_queue.popleft()
                if word == endWord:
                    return num_words
                for next_word in self._getNextWords(word, words):
                    next_word_queue.append(next_word)
            word_queue = next_word_queue
            num_words += 1
        return 0

    def _getNextWords(self, word: str, words: set[str]) -> Generator[str, None, None]:
        def get_distance(a: str, b: str) -> int:
            assert len(a) == len(b)
            distance = 0
            for i in range(len(a)):
                if a[i] != b[i]:
                    distance += 1
            return distance

        used_words = set()
        for candidate in words:
            if get_distance(word, candidate) == 1:
                used_words.add(candidate)
                yield candidate
        words.difference_update(used_words)
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1227073733844406343/1235880072238334022
  - https://github.com/sakupan102/arai60-practice/pull/20

先に隣接リストを作ってから幅優先探索。隣接リストにはindexを入れたが、そのまま文字列を入れても良い。usedはset()を使って `in` で判定しても良い。


```py
class Solution:
    def ladderLength(self, beginWord: str, endWord: str, wordList: List[str]) -> int:
        wordList.append(beginWord)
        adjacentList = self._makeAdjacentList(wordList)
        numWords = 1
        wordIndexes = deque([len(adjacentList) - 1]) # beginWord index in wordList
        used = [False] * len(wordList)
        while wordIndexes:
            nextWordIndexes = deque()
            while wordIndexes:
                wordIndex = wordIndexes.popleft()
                used[wordIndex] = True
                if wordList[wordIndex] == endWord:
                    return numWords
                for nextWordIndex in adjacentList[wordIndex]:
                    if used[nextWordIndex]:
                        continue
                    nextWordIndexes.append(nextWordIndex)
            wordIndexes = nextWordIndexes
            numWords += 1
        return 0

    def _makeAdjacentList(self, wordList: List[str]) -> List[List[int]]:
        def areAdjacentPair(x: str, y: str) -> int:
            assert len(x) == len(y)
            distance = 0
            for i in range(len(x)):
                if distance > 1:
                    return False
                if x[i] != y[i]:
                    distance += 1
            return distance == 1

        adjacentList = [[] for _ in range(len(wordList))]
        for i in range(len(wordList)):
            for j in range(i + 1, len(wordList)):
                if areAdjacentPair(wordList[i], wordList[j]):
                    adjacentList[i].append(j)
                    adjacentList[j].append(i)
        return adjacentList
```

- https://discord.com/channels/1084280443945353267/1200089668901937312/1215117909450424410
  - https://github.com/hayashi-ay/leetcode/pull/42

  endWordがwordListになかったら0を返すようにしている。高速化したくなったらやるかも、くらいの感覚。

i番目の文字が任意となることを表すパターンをkeyとして、そのパターンに合致するwordの集合をvalueとする辞書を作る。
ちょっとネストが深い。

```py
class Solution:
    def ladderLength(self, beginWord: str, endWord: str, wordList: List[str]) -> int:
        def make_patterns(word: str) -> Generator[str, None, None]:
            for i in range(len(word)):
                yield f'{word[:i]}.{word[i + 1:]}'

        pattern_to_words = defaultdict(list)
        for word in [beginWord] + wordList:
            for pattern in make_patterns(word):
                pattern_to_words[pattern].append(word)
        words = [beginWord]
        seen_words = set()
        num_words = 1
        while words:
            next_words = []
            for word in words:
                seen_words.add(word)
                if word == endWord:
                    return num_words
                for pattern in make_patterns(word):
                    for next_word in pattern_to_words[pattern]:
                        if next_word in seen_words:
                            continue
                        next_words.append(next_word)
            words = next_words
            num_words += 1
        return 0
```

## 3rd

```py
class Solution:
    def ladderLength(self, beginWord: str, endWord: str, wordList: List[str]) -> int:
        unused_words = set(wordList)

        def generate_adjacent_words(word: str) -> Generator[str, None, None]:
            for i in range(len(word)):
                for ch in ascii_lowercase:
                    adjacent_word = f'{word[:i]}{ch}{word[i + 1:]}'
                    if adjacent_word in unused_words:
                        unused_words.remove(adjacent_word)
                        yield adjacent_word

        num_words = 1
        words = [beginWord]
        while words:
            next_words = []
            for word in words:
                if word == endWord:
                    return num_words
                for next_word in generate_adjacent_words(word):
                    next_words.append(next_word)
            words = next_words
            num_words += 1
        return 0
```
