# 1032. Stream of Characters

## 1st

オートマトンで、Trieとかでできることはすぐ気付いたが、残念ながら実装する力が無かった。
15分ほど考えて一度止める。後でナイーブにでも実装しようと思い直して実装した。memcmpの引数を忘れたので調べた。素直に考えればstrcmpを使えばよかったか。
wordの最大長以上はストリームに来た文字を保持する必要はないが、高々40Kしか来ないので削除のコストを省いた

所要時間: 8:58

query() 1回辺りの評価。今まで来た入力文字数 n, wordsのサイズ m, wordsの平均文字数 l とすると
- 時間計算量: O(ml)
- 空間計算量: O(n + ml)

```cpp
class StreamChecker {
public:
    StreamChecker(vector<string>& words) : words_(words), stream_() {}

    bool query(char letter) {
        stream_ += letter;
        for (const string& word : words_) {
            if (stream_.length() < word.length()) continue;
            const int start = stream_.length() - word.length();
            if (memcmp(stream_.data() + start, word.data(), word.length()) == 0) return true;
        }
        return false;
    }

private:
    vector<string>& words_;
    string stream_;
};

/**
 * Your StreamChecker object will be instantiated and called as such:
 * StreamChecker* obj = new StreamChecker(words);
 * bool param_1 = obj->query(letter);
 */
```

---

Trie木は [スペルチェッカで使われがちなデータ構造 Trie の概要と実装 - A Memorandum](https://blog1.mammb.com/entry/2023/02/01/195037) を参考にした、があまり似てないかも。
Trie木の構築時に終了ノードのフラグ管理を間違えて1ミス。

自分の実装はアクティブなノードへのポインタを抱えるようにしているので、入力長をいくら長くしても時間もメモリも現実的な範囲で収まるはず。今回の問題設定 (At most 4*10^4 calls will be made to query.) だとアクティブなノードを状態として持たず、ストリームで来た文字列を全部保存して毎回尻尾から検索にかける方が速い気はする。ストリームを全部持つのは少し抵抗があるが。
wordの最大長以上はストリームに来た文字を保持する必要がないので、文字列の先頭削除にコストはかかるもののある程度ストリームからのデータが多くなったら適当に先頭を捨てる方が良いのかもしれない。
扱う文字種が増えたらvectorじゃなくmapで子を管理する。

std::basic_string::erase()の計算量がドキュメントに無かったので調べた。SOの記事によるといろんな戦略の実装があったので計算量を決めるのを諦めたぽい？
- [std::basic_string<CharT,Traits,Allocator>::erase - cppreference.com](https://en.cppreference.com/w/cpp/string/basic_string/erase)
- [C++ Time and Space Complexity of string class erase member function - Stack Overflow](https://stackoverflow.com/questions/52866539/c-time-and-space-complexity-of-string-class-erase-member-function)

getCharIndex()で使ったinline指定子の使い方が妥当なのか調べた。cppreference.comによると

> A function defined entirely inside a class/struct/union definition, whether it's a member function or a non-member friend function, is implicitly an inline function unless it is attached to a named module(since C++20).

とあるので、自信はないが付ける意味はないんじゃなかろうか。

- [inline specifier - cppreference.com](https://en.cppreference.com/w/cpp/language/inline)
- [c++ - Why is "inline" required on static inline variables? - Stack Overflow](https://stackoverflow.com/questions/46874055/why-is-inline-required-on-static-inline-variables)
- [c++ - Can you inline static member functions? - Stack Overflow](https://stackoverflow.com/questions/9244609/can-you-inline-static-member-functions)
- [C++におけるinline指定子の誤解 - よーる](https://lpha-z.hatenablog.com/entry/2019/02/17/231500)

所要時間: 1:04:10

query() 1回辺りの評価。今まで来た入力文字数 n, wordsのサイズ m, wordsの平均文字数 l とすると
- 時間計算量: O(ml)
- 空間計算量: O(n + ml)

```cpp
class TrieNode {
public:
    TrieNode() : next_nodes_(CHAR_COUNT), is_end_(false) {}

    void insert(const char* const ch) {
        if (*ch == '\0') {
            is_end_ = true;
            return;
        }
        const int char_index = getCharIndex(*ch);
        if (!next_nodes_[char_index]) {
            next_nodes_[char_index] = make_unique<TrieNode>();
        }
        next_nodes_[char_index]->insert(ch + 1);
    }

    TrieNode* transit(const char ch) {
        return next_nodes_[getCharIndex(ch)].get();
    }

    bool isEnd() const {
        return is_end_;
    }

private:
    static const size_t CHAR_COUNT = 26; // a-z
    vector<unique_ptr<TrieNode>> next_nodes_;
    bool is_end_;

    static inline int getCharIndex(const char ch) {
        assert('a' <= ch && ch <= 'z');
        return ch - 'a';
    }
};

class Trie {
public:
    Trie() : root_(), active_nodes_({&root_}) {}

    void insert(const string& s) {
        root_.insert(s.data());
    }

    bool searchStream(const char ch) {
        vector<TrieNode*> next_active_nodes = {};
        bool match = false;
        for (TrieNode* const node : active_nodes_) {
            TrieNode* const next_node = node->transit(ch);
            if (next_node) {
                next_active_nodes.push_back(next_node);
                match = match || next_node->isEnd();
            }
        }
        active_nodes_ = next_active_nodes;
        active_nodes_.push_back(&root_);
        return match;
    }

private:
    TrieNode root_;
    vector<TrieNode*> active_nodes_;
};

class StreamChecker {
public:
    StreamChecker(vector<string>& words) : stream_matcher_() {
        for (const string& word : words) {
            stream_matcher_.insert(word);
        }
    }

    bool query(char letter) {
        return stream_matcher_.searchStream(letter);
    }

private:
    Trie stream_matcher_;
};
```
