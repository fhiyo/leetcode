# 20. Valid Parentheses

## 1st

所要時間: 7:22

n: 文字列長
- 時間計算量: O(n)
- 空間計算量: O(n)

- if(isLeftParen(ch)) {... continue; } よりも if ... else ... の方が対比が分かりやすかったかもしれない


```cpp
class Solution {
public:
    bool isValid(string s) {
        stack<char> left_parens = {};
        for (const char ch : s) {
            if (isLeftParen(ch)) {
                left_parens.push(ch);
                continue;
            }
            if (left_parens.empty()) return false;
            const char left_paren = left_parens.top();
            left_parens.pop();
            if (!matchParenPair(left_paren, ch)) return false;
        }
        return left_parens.empty();
    }

private:
    static bool isLeftParen(const char ch) {
        return ch == '(' || ch == '{' || ch == '[';
    }

    static bool matchParenPair(const char left, const char right) {
        return left == '(' && right == ')' || left == '{' && right == '}' || left == '[' && right == ']';
    }
};
```

再帰下降構文解析でも解いてみる。

所要時間: 17:22

n: 文字列長
- 時間計算量: O(n)
- 空間計算量: O(1)

- resultという変数に入れなくても直接 `if(!parseParens(s, index)) return false;` で良かったか。副作用感が強い気もするが...

```cpp
class Solution {
public:
    /*
        parens := ( '(' parens ')' | '{' parens '}' | '[' parens ']' )*
    */
    bool isValid(string s) {
        int index = 0;
        while (index < s.length()) {
            const bool result = parseParens(s, index);
            if (!result) return false;
        }
        return true;
    }

private:
    static bool parseParens(const string& s, int& index) {
        if (index >= s.length()) return false;
        const char left_paren = consume(s, index);
        if (!isLeftParen(left_paren) || index >= s.length()) return false;
        while (!matchParenPair(left_paren, s[index])) {
            const bool result = parseParens(s, index);
            if (!result) return false;
        }
        consume(s, index); // right paren
        return true;
    }

    static char consume(const string& s, int& index) {
        return s[index++];
    }

    static bool isLeftParen(const char ch) {
        return ch == '(' || ch == '{' || ch == '[';
    }

    static bool matchParenPair(const char left, const char right) {
        return left == '(' && right == ')' || left == '{' && right == '}' || left == '[' && right == ']';
    }
};
```


## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1235971495696662578/1238099951725051974
- https://discord.com/channels/1084280443945353267/1233603535862628432/1235821988296003666
- https://discord.com/channels/1084280443945353267/1225849404037009609/1231643513385521212
    - 括弧以外の文字が来たときにどう動いてほしいか、という話題。自分はfalseが返ってくればいいかなと思って実装した。
- https://discord.com/channels/1084280443945353267/1201211204547383386/1202007495846150175
    - プッシュダウンオートマトンの話

```cpp
class Solution {
public:
    bool isValid(string s) {
        stack<char> left_parens = {};
        for (const char ch : s) {
            if (isLeftParen(ch)) {
                left_parens.push(ch);
            } else {
                if (left_parens.empty()) return false;
                const char left = left_parens.top();
                left_parens.pop();
                if (!matchPairParen(left, ch)) return false;
            }
        }
        return left_parens.empty();
    }

private:
    static bool isLeftParen(const char ch) {
        return ch == '(' || ch == '{' || ch == '[';
    }

    static bool matchPairParen(const char left, const char right) {
        return left == '(' && right == ')' || left == '{' && right == '}' || left == '[' && right == ']';
    }
};
```

## 3rd

省略
