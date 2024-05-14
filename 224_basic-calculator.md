# 224. Basic Calculator

## 1st

所要時間: 28:03

n: 文字列長
- 時間計算量: O(n)
- 空間計算量: O(1)

何回か書いたことがあるので再帰下降構文解析ではスムーズに手が進んだ。
factorとtermは逆だったかもしれない。

↑掛け算割り算が来ないことを見逃していた。 [227. Basic Calculator II](https://leetcode.com/problems/basic-calculator-ii/description/) にsubmitしたら通ったので合っている...のだと思うが、単項マイナスが含まれてなかったので良くわからない


```cpp
class Calculator {
public:
    /*
        expr   := factor (('+' | '-') factor)*
        factor := term (('*' | '/') term)*
        term   := ws ('-' ws)? (number | '(' ws expr ws ')') ws
        number := [0-9]+
        ws     := ' '*
    */
    static int eval(const string& input) {
        int index = 0;
        return expr(input, index);
    }

private:
    static int expr(const string& s, int& i) {
        int lfactor = factor(s, i);
        while (true) {
            if (s[i] == '+') {
                consume('+', s, i);
                int rfactor = factor(s, i);
                lfactor += rfactor;
            } else if (s[i] == '-') {
                consume('-', s, i);
                int rfactor = factor(s, i);
                lfactor -= rfactor;
            } else break;
        }
        return lfactor;
    }

    static int factor(const string& s, int& i) {
        int lterm = term(s, i);
        while (true) {
            if (s[i] == '*') {
                consume('*', s, i);
                int rterm = term(s, i);
                lterm *= rterm;
            } else if (s[i] == '/') {
                consume('/', s, i);
                int rterm = term(s, i);
                lterm /= rterm;
            } else break;
        }
        return lterm;
    }

    static int term(const string& s, int& i) {
        int sign = 1;
        int result = 0;
        ws(s, i);
        if (s[i] == '-') {
            consume('-', s, i);
            sign = -1;
            ws(s, i);
        }
        if (isdigit(s[i])) {
            result = number(s, i);
        } else {
            if (!consume('(', s, i)) fail(s, i);
            result = expr(s, i);
            if (!consume(')', s, i)) fail(s, i);
        }
        ws(s, i);
        return sign * result;
    }

    // signed 32-bit integer の範囲に収まることは条件から保証されている
    static int number(const string& s, int& i) {
        int num = 0;
        while (isdigit(s[i])) {
            num = 10 * num + (s[i] - '0');
            i++;
        }
        return num;
    }

    static void ws(const string& s, int& i) {
        while (s[i] == ' ') {
            i++;
        }
    }

    static bool consume(const char ch, const string& s, int& i) {
        if (i < s.length() && s[i] == ch) {
            i++;
            return true;
        }
        return false;
    }

    static void fail(const string& s, const int i) {
        cerr << "interpretation is failed at " << i << ". input: " << s << endl;
        exit(1);
    }
};

class Solution {
public:
    int calculate(string s) {
        return Calculator::eval(s);
    }
};
```

---

プッシュダウンオートマトンで解く。
逆ポーランド記法に直せばいいかと思ったが単項マイナスの扱いをどうするのかよく分からなかったので https://leetcode.com/problems/basic-calculator/solutions/62361/iterative-java-solution-with-stack を参考にして書いた。
書けるようにはしたがいまいちピンと来ていない。

所要時間: 12:43

n: 文字列長
- 時間計算量: O(n)
- 空間計算量: O(n)

```cpp
class Solution {
public:
    int calculate(string s) {
        stack<int> sign_and_nums = {};
        int result = 0;
        int sign = 1;
        int num = 0;
        int index = 0;
        while (index < s.length()) {
            skipWhiteSpace(s, index);
            if (isdigit(s[index])) {
                num = parseInt(s, index);
            } else {
                if (s[index] == '+' || s[index] == '-') {
                    result += sign * num;
                    if (s[index] == '+') {
                        sign = 1;
                    } else {
                        // '-'
                        sign = -1;
                    }
                    num = 0;
                }
                if (s[index] == '(') {
                    sign_and_nums.push(result);
                    result = 0;
                    sign_and_nums.push(sign);
                    sign = 1;
                }
                if (s[index] == ')') {
                    result += sign * num;
                    sign = 1;
                    num = 0;
                    result *= sign_and_nums.top(); // sign
                    sign_and_nums.pop();
                    result += sign_and_nums.top(); // num
                    sign_and_nums.pop();
                }
                index++;
            }
        }
        if (num) {
            result += sign * num;
        }
        return result;
    }

private:
    static void skipWhiteSpace(const string& s, int& i) {
        while (s[i] == ' ') {
            i++;
        }
    }

    static int parseInt(const string& s, int& i) {
        int result = 0;
        while (isdigit(s[i])) {
            result = 10 * result + (s[i] - '0');
            i++;
        }
        return result;
    }
};
```


## 2nd

プッシュダウンオートマトン

```cpp
class Solution {
public:
    int calculate(string s) {
        stack<int> sign_and_nums = {};
        int result = 0;
        int sign = 1;
        int num = 0;
        int index = 0;
        while (index < s.length()) {
            skipWhiteSpace(s, index);
            if (isdigit(s[index])) {
                num = parseInt(s, index);
            } else {
                if (s[index] == '+' || s[index] == '-') {
                    result += sign * num;
                    if (s[index] == '+') {
                        sign = 1;
                    } else {
                        sign = -1; // '-'
                    }
                    num = 0;
                }
                if (s[index] == '(') {
                    sign_and_nums.push(result);
                    result = 0;
                    sign_and_nums.push(sign);
                    sign = 1;
                }
                if (s[index] == ')') {
                    result += sign * num;
                    sign = 1;
                    num = 0;
                    result *= sign_and_nums.top(); // sign
                    sign_and_nums.pop();
                    result += sign_and_nums.top(); // num
                    sign_and_nums.pop();
                }
                index++;
            }
        }
        if (num) {
            result += sign * num;
        }
        return result;
    }

private:
    static void skipWhiteSpace(const string& s, int& i) {
        while (s[i] == ' ') {
            i++;
        }
    }

    static int parseInt(const string& s, int& i) {
        int num = 0;
        while (isdigit(s[i])) {
            num = 10 * num + (s[i] - '0');
            i++;
        }
        return num;
    }
};
```

---

再帰下降構文解析

```cpp
class Solution {
public:
    /*
        expr   := term (('+' | '-') term)*
        term   := ws ('-' ws)? (number | '(' ws expr ws ')') ws
        number := [0-9]+
        ws   := ' '*
    */
    int calculate(string s) {
        int index = 0;
        return expr(s, index);
    }

private:
    static int expr(const string& s, int& i) {
        int lterm = term(s, i);
        while (true) {
            if (s[i] != '+' && s[i] != '-') break;
            const char op = s[i];
            i++;
            int rterm = term(s, i);
            switch (op) {
                case '+':
                    lterm += rterm;
                    break;
                case '-':
                    lterm -= rterm;
                    break;
                default:
                    fail(s, i);
            }
        }
        return lterm;
    }

    static int term(const string& s, int& i) {
        int sign = 1;
        int result = 0;
        ws(s, i);
        if (s[i] == '-') {
            sign = -1;
            i++;
            ws(s, i);
        }
        if (isdigit(s[i])) {
            result = number(s, i);
        } else {
            if (!consume('(', s, i)) fail(s, i);
            ws(s, i);
            result = expr(s, i);
            ws(s, i);
            if (!consume(')', s, i)) fail(s, i);
        }
        ws(s, i);
        result *= sign;
        return result;
    }

    static int number(const string& s, int& i) {
        int num = 0;
        while (isdigit(s[i])) {
            num = 10 * num + (s[i] - '0');
            i++;
        }
        return num;
    }

    static void ws(const string& s, int& i) {
        while (s[i] == ' ') {
            i++;
        }
    }

    static bool consume(const char ch, const string& s, int& i) {
        if (i < s.length() && s[i] == ch) {
            i++;
            return true;
        }
        return false;
    }

    static void fail(const string& s, const int i) {
        cerr << "parse failed at " << i << ". input string: " << s << endl;
        exit(1);
    }
};
```

## 3rd

省略
