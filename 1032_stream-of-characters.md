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

## 2nd

ローリングハッシュを用いた解法

所要時間: 13:12

コンストラクタ: wordsの長さをnとして
- 時間計算量: $O(\sum_{i=1}^n len(word_i))$
- 空間計算量: $O(\sum_{i=1}^n len(word_i))$

query() 1回辺りの評価。wordのうち最大長のものの長さを$l$として
- 時間計算量: $O(l)$
- 空間計算量: $O(l)$

```cpp
class StreamChecker {
public:
    StreamChecker(vector<string>& words) : stream_suffix_(), max_word_length_(0), word_hashes_({}), end_of_word_hashes_({}) {
        for (const string& word : words) {
            uint64_t hash = 0;
            uint64_t prime_pow = 1;
            for (auto it = word.crbegin(); it != word.crend(); it++) {
                nextRollingHash(*it, hash, prime_pow);
                word_hashes_.insert(hash);
            }
            end_of_word_hashes_.insert(hash);
            max_word_length_ = max(max_word_length_, word.length());
        }
    }

    bool query(char letter) {
        stream_suffix_.push_back(letter);
        if (stream_suffix_.size() > max_word_length_) {
            stream_suffix_.pop_front();
        }
        uint64_t hash = 0;
        uint64_t prime_pow = 1;
        for (auto it = stream_suffix_.crbegin(); it != stream_suffix_.crend(); it++) {
            nextRollingHash(*it, hash, prime_pow);
            if (!word_hashes_.contains(hash)) return false;
            if (end_of_word_hashes_.contains(hash)) return true;
        }
        return false;
    }

private:
    void nextRollingHash(const char ch, uint64_t& hash, uint64_t& prime_pow) const {
        const uint64_t prime = 29;
        const uint64_t mod = (1 << 61) - 1;
        hash = (hash + prime_pow * (ch - 'a' + 1)) % mod;
        prime_pow *= prime;
    }

    deque<char> stream_suffix_;
    size_t max_word_length_;
    set<uint64_t> word_hashes_;
    set<uint64_t> end_of_word_hashes_;
};
```

## 3rd

所要時間: 16:07

```cpp
class Trie {
public:
    Trie() {}

    // insert a word in reverse order
    void insert(const string& word) {
        if (!word.length()) return;
        insert(word, word.length() - 1, &root_);
    }

    bool search(const deque<char>& s) const {
        if (s.empty()) return true;
        return search(s, s.size() - 1, &root_);
    }

private:
    struct Node {
        bool is_end;
        map<char, unique_ptr<Node>> next_nodes;
        Node() : is_end(false), next_nodes() {}
    };

    void insert(const string& word, const int index, Node* const node) {
        if (index == -1) {
            node->is_end = true;
            return;
        }
        node->next_nodes.insert({word[index], make_unique<Node>()});
        insert(word, index - 1, node->next_nodes.at(word[index]).get());
    }

    bool search(const deque<char>& s, const int index, const Node* const node) const {
        if (node->is_end) return true;
        if (index == -1) return false;
        if (!node->next_nodes.contains(s[index])) return false;
        return search(s, index - 1, node->next_nodes.at(s[index]).get());
    }

    Node root_;
};

class StreamChecker {
public:
    StreamChecker(vector<string>& words) : checker_(), stream_suffix_({}), max_word_length_(0) {
        for (const string& word : words) {
            checker_.insert(word);
            max_word_length_ = max(max_word_length_, word.length());
        }
    }

    bool query(char letter) {
        stream_suffix_.push_back(letter);
        if (stream_suffix_.size() > max_word_length_) {
            stream_suffix_.pop_front();
        }
        return checker_.search(stream_suffix_);
    }

private:
    Trie checker_;
    deque<char> stream_suffix_;
    size_t max_word_length_;
};
```


## 4th

Aho-Corasick

```cpp
class DFA {
public:
    DFA() : num_(0), root_(genId()), current_node_(&root_), known_chars_({}) {}

    ~DFA() {
        for (const auto [ch, node] : root_.go_to) {
            if (node == &root_) continue;
            freeNodes(node);
        }
    }

    void build(const vector<string>& words) {
        for (const string& word : words) {
            insert(word, 0, &root_);
        }
        for (const char ch : known_chars_) {
            root_.go_to.try_emplace(ch, &root_);
        }
        buildFailureLink();
    }

    bool transit(const char ch) {
        if (!known_chars_.contains(ch)) {
            root_.go_to[ch] = &root_;
            known_chars_.insert(ch);
        }
        while (!current_node_->go_to.contains(ch)) {
            current_node_ = current_node_->failure;
        }
        current_node_ = current_node_->go_to.at(ch);
        return current_node_->active;
    }

private:
    struct Node {
        int id; // for debug
        map<char, Node*> go_to;
        Node* failure;
        bool active;
        Node(int id) : id(id), go_to({}), failure(nullptr), active(false) {}
    };

    void insert(const string& word, const size_t index, Node* const node) {
        if (index == word.length()) {
            node->active = true;
            return;
        }
        known_chars_.insert(word[index]);
        if (!node->go_to.contains(word[index])) {
            node->go_to[word[index]] = new Node(genId());
        }
        insert(word, index + 1, node->go_to.at(word[index]));
    }

    void buildFailureLink() {
        queue<Node*> nodes = {};
        for (auto [ch, node] : root_.go_to) {
            if (node == &root_) continue;
            node->failure = &root_;
            nodes.push(node);
        }
        while (!nodes.empty()) {
            Node* const node = nodes.front();
            nodes.pop();
            for (auto [ch, next_node] : node->go_to) {
                Node* failure = node->failure;
                while (!failure->go_to.contains(ch)) {
                    failure = failure->failure;
                }
                next_node->failure = failure->go_to.at(ch);
                next_node->active = next_node->active || next_node->failure->active;
                nodes.push(next_node);
            }
        }
    }

    // free memory allocated for nodes
    void freeNodes(Node* const node) {
        for (auto [ch, next_node] : node->go_to) {
            freeNodes(next_node);
        }
        delete node;
    }

    // for debug
    int genId() {
        return num_++;
    }

    // for debug
    static void printNode(const Node* const node) {
        printf("---------------\n");
        printf("node id: %d\n", node->id);
        printf("-- go_to --\n");
        for (const auto [ch, next_node] : node->go_to) {
            printf("ch: %c, next_node: %d\n", ch, next_node->id);
        }
        printf("failure: %d\n", node->failure ? node->failure->id : -1);
        printf("---------------\n");
    }

    int num_;
    Node root_;
    Node* current_node_;
    set<char> known_chars_;
};

class StreamChecker {
public:
    StreamChecker(vector<string>& words) : checker_({}) {
        checker_.build(words);
    }

    bool query(char letter) {
        return checker_.transit(letter);
    }

private:
    DFA checker_;
};
```
