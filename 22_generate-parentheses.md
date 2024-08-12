# 22. Generate Parentheses

## 1st

### ①

使った括弧をリストに順番に入れて、使い切ったら文字列にして出力する。今までの括弧の列からあり得る出力を探索し終えたらその括弧をリストから除去するbacktracking。

所要時間: 13:30

- 時間計算量: O(n * 2^(2n)) (文字列の任意の位置で開き括弧/閉じ括弧が許される仮定の甘い見積もり。厳密には n x カタラン数でO(4^n/(sqrt(n)))？ (ref: https://en.wikipedia.org/wiki/Catalan_number))
  - 手元の環境で試すとn=15で約12s, n=16で約46sかかった。CPUの周波数1G/sとして1回の計算が10ステップ、言語定数50とすると
    $(4^{16})/\sqrt{16} * 10 * 50 / 10^9 = 536 \text{秒}$
    10倍以上計測とずれてるけど、まあ許容か...？
- 空間計算量: O(n) (出力を除く)

```py
class Solution:
    def generateParenthesis(self, n: int) -> List[str]:
        parens = []

        def generate(left: int, right: int) -> Iterator[str]:
            if left == 0 and right == 0:
                yield ''.join(parens)
                return
            if left > 0:
                parens.append('(')
                yield from generate(left - 1, right)
                parens.pop()
            if right - left > 0:
                parens.append(')')
                yield from generate(left, right - 1)
                parens.pop()

        return list(generate(n, n))
```

### ②

対応する閉じ括弧がない開き括弧の数と、残りの閉じ括弧の数を管理する方法。

たとえばgenerate(2,3)は `"))()", ")())", "()))"` を順に返す。

(後でleft_open_count => left_unclosed_count とすればいいかなと思った)

所要時間: 9:40

- 計算量: 省略

```py
class Solution:
    def generateParenthesis(self, n: int) -> list[str]:
        def generate(left_open_count: int, right_rest_count: int) -> Iterator[str]:
            if left_open_count == right_rest_count:
                yield ')' * right_rest_count
                return
            if left_open_count > 0:
                yield from map(lambda s: ')' + s, generate(left_open_count - 1, right_rest_count - 1))
            yield from map(lambda s: '(' + s, generate(left_open_count + 1, right_rest_count))

        return list(generate(0, n))
```

### ③

こっちの方がもう少し分かりやすいか？あまり変わらないかも。

所要時間: 未計測

```py
class Solution:
    def generateParenthesis(self, n: int) -> list[str]:
        def generate(left_open_count: int, right_rest_count: int) -> Iterator[str]:
            if right_rest_count == 0:
                yield ''
                return
            if left_open_count > 0:
                yield from map(lambda s: ')' + s, generate(left_open_count - 1, right_rest_count - 1))
            if left_open_count < right_rest_count:
                yield from map(lambda s: '(' + s, generate(left_open_count + 1, right_rest_count))

        return list(generate(0, n))
```

### ④

②のようにgeneratorを返すようにすると関数が重複したものを返すときにcacheを使いにくい。下のようにlistを返す方が良いかもしれない。

所要時間: 6:04

- 計算量: 省略

```py
class Solution:
    def generateParenthesis(self, n: int) -> List[str]:
        @cache
        def create_parens(left_unclosed_count: int, right_rest_count: int) -> list[str]:
            if left_unclosed_count == right_rest_count:
                return [')' * right_rest_count]
            parens_list = []
            if left_unclosed_count > 0:
                for parens in create_parens(left_unclosed_count - 1, right_rest_count - 1):
                    parens_list.append(')' + parens)
            for parens in create_parens(left_unclosed_count + 1, right_rest_count):
                parens_list.append('(' + parens)
            return parens_list

        return create_parens(0, n)
```

### ⑤

雑にgeneratorをcacheするならこんな感じ？

参考
- [Can I memoize a Python generator? - Stack Overflow](https://stackoverflow.com/questions/4566769/can-i-memoize-a-python-generator)
- [python - How to hash *args **kwargs for function cache? - Stack Overflow](https://stackoverflow.com/questions/10220599/how-to-hash-args-kwargs-for-function-cache)
- https://github.com/python/cpython/blob/53ebb6232a8ebc03827cf2251bfc67f1886ffd70/Lib/functools.py#L464


```py
def generator_cache(f):
    cache = {}

    def ret(*args, **kwargs):
        key = (*args, tuple(sorted(kwargs.items())))
        if not key in cache:
            cache[key] = f(*args, **kwargs)
        cache[key], g = tee(cache[key])
        return g

    return ret


class Solution:
    def generateParenthesis(self, n: int) -> list[str]:
        @generator_cache
        def generate(left_open_count: int, right_rest_count: int) -> Iterator[str]:
            if right_rest_count == 0:
                yield ''
                return
            if left_open_count > 0:
                yield from map(lambda s: ')' + s, generate(left_open_count - 1, right_rest_count - 1))
            if left_open_count < right_rest_count:
                yield from map(lambda s: '(' + s, generate(left_open_count + 1, right_rest_count))

        return list(generate(0, n))
```


## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1253694251271852095/1271108009354985473
  - https://github.com/rihib/leetcode/pull/11

    - https://discord.com/channels/1084280443945353267/1218823830743547914/1231546400714788864

    > 他に、開き+閉じ*(0~ここまでの開きの数までのどれか)までを一つの仕事とみるという仕事の分担方法もあるように思います。

```py
class Solution:
    def generateParenthesis(self, n: int) -> List[str]:
        def generate(s: str, left_unclosed_count: int) -> Iterator[str]:
            if len(s) == 2 * n:
                yield s
            for i in range(left_unclosed_count + 2):
                new_length = len(s) + i + 1
                if new_length > 2 * n:
                    break
                new_left_unclosed_count = left_unclosed_count - i + 1
                if new_length == 2 * n and new_left_unclosed_count > 0:
                    break
                yield from generate(s + '(' + ')' * i, new_left_unclosed_count)

        return list(generate('', 0))
```

- https://discord.com/channels/1084280443945353267/1239148130679783424/1267471436801507410
  - https://github.com/rossy0213/leetcode/pull/27/files
- https://discord.com/channels/1084280443945353267/1252267683731345438/1252591437485441024
  - https://github.com/wf9a5m75/leetcode3/pull/2

動きを見ると下のような感じ。一つ対応する括弧を作り、その中と右の外側に再帰的に括弧を作っていく感じのようだ。なるほど。

```txt
gen[0] -> ""
gen[1] -> "(gen[0])gen[0]" -> "()"
gen[2] -> "(gen[0])gen[1]", "(gen[1])gen[0]" -> "()()", "(())"
gen[3] -> "(gen[0])gen[2]", "(gen[1])gen[1]", "(gen[2])gen[0]" -> "()()()", "()(())", "(())()", "(()())", "((()))"
```

inner functionでcacheしているので外側で出力を変更してもcacheの中身は消えている (generateParenthesis()が呼び出されている間のみ有効なキャッシュになっている)。

```py
class Solution:
    def generateParenthesis(self, n: int) -> List[str]:
        @cache
        def generate(n: int) -> list[str]:
            if n == 0:
                return ['']
            all_parens = []
            for num_inner_parens in range(n):
                num_outer_parens = n - num_inner_parens - 1
                parens = []
                for inner_parens in generate(num_inner_parens):
                    for outer_parens in generate(num_outer_parens):
                        parens.append('(' + inner_parens + ')' + outer_parens)
                all_parens.extend(parens)
            return all_parens

        return generate(n)
```

- https://discord.com/channels/1084280443945353267/1233603535862628432/1235821988296003666
  - https://github.com/goto-untrapped/Arai60/pull/11
- https://discord.com/channels/1084280443945353267/1233295449985650688/1235438504625049640
  - https://github.com/Exzrgs/LeetCode/pull/6
- https://discord.com/channels/1084280443945353267/1225849404037009609/1232393910643589200
  - https://github.com/SuperHotDogCat/coding-interview/pull/7
- https://discord.com/channels/1084280443945353267/1218823830743547914/1231112614835388486
  - https://github.com/ryoooooory/LeetCode/pull/6

validな括弧列に対して"()"を開き括弧と閉じ括弧の位置を変えながら挿入する。重複が発生するので、既に見た途中状態が入ってきたらスキップする。

```py
class Solution:
    def generateParenthesis(self, n: int) -> List[str]:
        seen = set()

        def generate(n: int, parens: str) -> Iterator[str]:
            if parens in seen:
                return
            seen.add(parens)
            if n == 0:
                yield parens
                return
            for i in range(len(parens) + 1):
                new_parens = parens[:i] + '(' + parens[i:]
                for j in range(i + 1, len(new_parens) + 1):
                    yield from generate(n - 1, new_parens[:j] + ')' + new_parens[j:])

        return list(generate(n, ''))
```

- https://discord.com/channels/1084280443945353267/1201211204547383386/1230218098573836318
  - https://github.com/shining-ai/leetcode/pull/53
- https://discord.com/channels/1084280443945353267/1200089668901937312/1224712663523790980
  - https://github.com/hayashi-ay/leetcode/pull/70


## 3rd

分かりやすいかなと思うのでこれにした。

```py
class Solution:
    def generateParenthesis(self, n: int) -> List[str]:
        parens = []

        def generate(left: int, right: int) -> Iterator[str]:
            if left == n and right == n:
                yield ''.join(parens)
                return
            if left < n:
                parens.append('(')
                yield from generate(left + 1, right)
                parens.pop()
            if left > right:
                parens.append(')')
                yield from generate(left, right + 1)
                parens.pop()

        return list(generate(0, 0))
```
