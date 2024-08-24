# 8. String to Integer (atoi)

## 1st

### ①

言われたとおりにやる。int32の範囲外になったときの処理がたどたどしい感じだった。

https://docs.python.org/3/reference/expressions.html#operator-precedence
シフト演算子よりも+, -の方が優先順位が高いので注意

[str.isdigit()](https://docs.python.org/3.11/library/stdtypes.html#str.isdigit)があるのは知っていた (isの後にアンスコ入れるのかうろ覚えだったが) が、[0-9]以外もTrueにするやつがいくつかあったと思うので使うのを止めておいた (`'12۳۴5'` とかを普通に12345に解釈するの何か微妙な気もしたので。とはいえ`int('12۳۴5') == 12345`ではあるのだが...)。
`ord('0') <= ord(c) <= ord('9')` とする方が列挙せず済むので目に優しいか (単純に `'0' <= c <= '9'` でもいいが)。

digitの変換も、一文字だから `ord(s[i]) - ord('0')` の方が自然かもしれない

`while is_digit(i)` 内はbreakせずその場で返せばよかった...


所要時間: 11:45

n: len(s)
- 時間計算量: O(n)
- 空間計算量 (auxiliary): O(1)


```py
class Solution:
    INT_MAX = (1 << 31) - 1

    def myAtoi(self, s: str) -> int:
        def is_whitespace(index: int) -> bool:
            return index < len(s) and s[index] == ' '

        def is_sign(index: int) -> bool:
            return index < len(s) and (is_plus(index) or is_minus(index))

        def is_plus(index: int) -> bool:
            return index < len(s) and s[index] == '+'

        def is_minus(index: int) -> bool:
            return index < len(s) and s[index] == '-'

        def is_digit(index: int) -> bool:
            return index < len(s) and (
                s[index] == '0' or \
                s[index] == '1' or \
                s[index] == '2' or \
                s[index] == '3' or \
                s[index] == '4' or \
                s[index] == '5' or \
                s[index] == '6' or \
                s[index] == '7' or \
                s[index] == '8' or \
                s[index] == '9'
            )

        i = 0
        while is_whitespace(i):
            i += 1
        sign = 1
        if is_sign(i):
            if is_minus(i):
                sign = -1
            i += 1
        integer = 0
        while is_digit(i):
            integer *= 10
            integer += int(s[i])
            if integer >= Solution.INT_MAX + 1:
                integer = Solution.INT_MAX + 1
                break
            i += 1
        if sign > 0 and integer > Solution.INT_MAX:
            return Solution.INT_MAX
        return sign * integer
```

---
昔C++で書いたもの。こっちの方がきれいな気がする...

```cpp
class Solution {
public:
    int myAtoi(string s) {
        int abs_value = 0;
        int index = 0;
        skipWhitespace(s, index);
        int sign = parseSignIfExists(s, index);
        while (isdigit(s[index])) {
            const int digit = parseDigit(s, index);
            if (abs_value > INT_MAX / 10 || abs_value == INT_MAX / 10 && digit > INT_MAX % 10) {
                // 10 * abs_value + digit >= INT_MAX + 1 (if ignore overflow)
                if (sign == -1) return INT_MIN;
                return INT_MAX;
            }
            abs_value = 10 * abs_value + digit;
        }
        return sign * abs_value;
    }

private:
    void skipWhitespace(const string& s, int& index) const {
        while (s[index] == ' ') {
            index++;
        }
        return;
    }

    int parseSignIfExists(const string& s, int& index) const {
        int sign = 1;
        if (s[index] == '+' || s[index] == '-') {
            if (s[index] == '-') {
                sign = -1;
            }
            index++;
        }
        return sign;
    }

    int parseDigit(const string& s, int& index) const {
        // precondition: s[index] is digit
        return s[index++] - '0';
    }
};
```

### ②

符号付き32bit整数のオーバーフローを気にした場合の実装。

処理時間: 23:40

n: len(s)
- 時間計算量: O(n)
- 空間計算量 (auxiliary): O(1)


```py
class Solution:
    INT_MAX = (1 << 30) - 1 + (1 << 30) # avoid overflow

    def myAtoi(self, s: str) -> int:
        def is_whitespace(index: int) -> bool:
            return index < len(s) and s[i] == ' '

        def is_sign(index: int) -> bool:
            return is_plus(index) or is_minus(index)

        def is_plus(index: int) -> bool:
            return index < len(s) and s[index] == '+'

        def is_minus(index: int) -> bool:
            return index < len(s) and s[index] == '-'

        def is_digit(index: int) -> bool:
            return index < len(s) and ord('0') <= ord(s[index]) <= ord('9')

        i = 0
        while is_whitespace(i):
            i += 1
        sign = 1
        if is_sign(i):
            if is_minus(i):
                sign = -1
            i += 1
        abs_value = 0
        while is_digit(i):
            digit = ord(s[i]) - ord('0')
            i += 1
            if abs_value > Solution.INT_MAX // 10 or abs_value == Solution.INT_MAX // 10 and digit > Solution.INT_MAX % 10:
                if sign > 0:
                    return Solution.INT_MAX
                return -Solution.INT_MAX - 1 # INT_MIN
            abs_value *= 10
            abs_value += digit
        return sign * abs_value
```


## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1253694251271852095/1270671133452140555
  - https://github.com/rihib/leetcode/pull/10
- https://discord.com/channels/1084280443945353267/1196472827457589338/1247562936453562420
  - https://github.com/Mike0121/LeetCode/pull/23

digits部分をまとめてから一気に変換している。先に `s = s.lstrip()` するのもなるほど。
超巨大な数になる場合、最後までparseを止めないため不利になる実装。


```py
class Solution:
    INT_MIN = - (1 << 31)
    INT_MAX = (1 << 31) - 1

    def myAtoi(self, s: str) -> int:
        def is_sign(index: int) -> bool:
            return index < len(s) and (is_plus(index) or is_minus(index))

        def is_plus(index: int) -> bool:
            return index < len(s) and s[index] == '+'

        def is_minus(index: int) -> bool:
            return index < len(s) and s[index] == '-'

        def is_digit(index: int) -> bool:
            return index < len(s) and ord('0') <= ord(s[index]) <= ord('9')

        s = s.lstrip()
        i = 0
        negative = False
        if is_sign(i):
            negative = is_minus(i)
            i += 1
        digits = ['0']
        while is_digit(i):
            digits.append(s[i])
            i += 1
        num = int(''.join(digits))
        if negative:
            num *= -1
        if num <= Solution.INT_MIN:
            return Solution.INT_MIN
        if num >= Solution.INT_MAX:
            return Solution.INT_MAX
        return num
```

- https://discord.com/channels/1084280443945353267/1233603535862628432/1235228260363931658
  - https://github.com/goto-untrapped/Arai60/pull/9
- https://discord.com/channels/1084280443945353267/1233295449985650688/1234515624575762463
  - https://github.com/Exzrgs/LeetCode/pull/4
- https://discord.com/channels/1084280443945353267/1201211204547383386/1232378622027890688
  - https://github.com/shining-ai/leetcode/pull/59
- https://discord.com/channels/1084280443945353267/1226508154833993788/1231996199020789760
  - https://github.com/nittoco/leetcode/pull/6
- https://discord.com/channels/1084280443945353267/1225849404037009609/1230578528328880250
  - https://github.com/SuperHotDogCat/coding-interview/pull/5
- https://discord.com/channels/1084280443945353267/1200089668901937312/1224698913282457610
  - https://github.com/hayashi-ay/leetcode/pull/69
- https://discord.com/channels/1084280443945353267/1217527351890546789/1219950919928647750
  - https://github.com/cheeseNA/leetcode/pull/5

```py
digits = '0123456789'
s[index] in digits
```

という判定も分かりやすくていいなぁと思う。


## 3rd

is_sign()とはその辺りの関数は `s[i] in foo` のやり方で十分見やすいなと思い消した。

全体を通して、広めのスコープでiという短い名前の変数を取り回しているが、そんなに見にくくなってないような気がしてるので長い名前にはしていない (indexとしても読みやすさ変わらない気がする)。


```py
class Solution:
    MAX_INT = (1 << 31) - 1
    MIN_INT = -(1 << 31)
    SIGNS = '+-'
    DIGITS = '0123456789'

    def myAtoi(self, s: str) -> int:
        s = s.lstrip()
        i = 0
        sign = 1
        if i < len(s) and s[i] in Solution.SIGNS:
            if s[i] == '-':
                sign = -1
            i += 1
        abs_value = 0
        while i < len(s) and s[i] in Solution.DIGITS:
            digit = ord(s[i]) - ord('0')
            i += 1
            abs_value *= 10
            abs_value += digit
            if sign > 0 and abs_value > Solution.MAX_INT:
                return Solution.MAX_INT
            if sign < 0 and abs_value > abs(Solution.MIN_INT):
                return Solution.MIN_INT
        return sign * abs_value
```
