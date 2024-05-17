# 146. LRU Cache

## 1st

所要時間: 57:11

n: 入力長
- 時間計算量: O(n)
- 空間計算量: O(capacity)

分からなかったので答え見た＆unique_ptrのことを調べながらやってしまったので時間がかかっている。
cacheのvalueに生ポインタを使うならputでリストとcacheから削除した後にdeleteしないとメモリリークするはず。
removeFromListでnode->prevとnode->nextにnullptrを代入するのは不要だが念の為の気持ちで書いている。

```cpp
struct Node {
    int key;
    int value;
    Node* prev;
    Node* next;

    Node(int key, int value) : key(key), value(value), prev(nullptr), next(nullptr) {}
};

class LRUCache {
    unordered_map<int, unique_ptr<Node>> cache;
    Node head;
    Node tail;
    int capacity;

public:
    LRUCache(int capacity) : cache(), head(0, 0), tail(0, 0), capacity(capacity) {
        head.next = &tail;
        tail.prev = &head;
    }

    int get(int key) {
        if (!cache.contains(key)) return -1;
        Node* const node = cache.at(key).get();
        toMRU(node);
        return node->value;
    }

    void put(int key, int value) {
        if (cache.contains(key)) {
            cache.at(key)->value = value;
            toMRU(cache.at(key).get());
            return;
        }
        cache[key] = make_unique<Node>(key, value);
        addToList(cache.at(key).get());
        if (cache.size() > capacity) {
            Node* const lru = tail.prev;
            removeFromList(lru);
            cache.erase(lru->key);
        }
    }

private:
    // MRU: Most Recently Used
    void toMRU(Node* const node) {
        removeFromList(node);
        addToList(node);
    }

    void removeFromList(Node* const node) {
        node->prev->next = node->next;
        node->next->prev = node->prev;
        node->prev = nullptr;
        node->next = nullptr;
    }

    // add to next to head
    void addToList(Node* const node) {
        node->prev = &head;
        node->next = head.next;
        head.next->prev = node;
        head.next = node;
    }
};

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache* obj = new LRUCache(capacity);
 * int param_1 = obj->get(key);
 * obj->put(key,value);
 */
```


## 2nd

省略。↓を1stのときに調べていた

- [std::unique_ptr - cppreference.com](https://en.cppreference.com/w/cpp/memory/unique_ptr)
- [std::unordered_map<Key,T,Hash,KeyEqual,Allocator>::unordered_map - cppreference.com](https://en.cppreference.com/w/cpp/container/unordered_map/unordered_map)
- [List-initialization (since C++11) - cppreference.com](https://en.cppreference.com/w/cpp/language/list_initialization)
- [Value categories - cppreference.com](https://en.cppreference.com/w/cpp/language/value_category)


## 3rd

getLRU()はcacheが空のときにnullptrを返すようにしてみた。各種ポインタを引数に受け取るメソッドは受け取ったポインタがnullptrでないことをassertとかで保証すべきなのかもしれない。

```cpp
struct Node {
    int key;
    int value;
    Node* prev;
    Node* next;

    Node(int key, int value) : key(key), value(value), prev(nullptr), next(nullptr) {}
};

class LRUCache {
    unordered_map<int, unique_ptr<Node>> cache;
    int capacity;
    Node head;
    Node tail;

public:
    LRUCache(int capacity) : cache(), capacity(capacity), head(0, 0), tail(0, 0) {
        head.next = &tail;
        tail.prev = &head;
    }

    int get(int key) {
        if (!cache.contains(key)) return -1;
        toMRU(cache.at(key).get());
        return cache.at(key)->value;
    }

    void put(int key, int value) {
        if (cache.contains(key)) {
            cache.at(key)->value = value;
            toMRU(cache.at(key).get());
            return;
        }
        cache[key] = make_unique<Node>(key, value);
        addToList(cache.at(key).get());
        if (cache.size() > capacity) {
            Node* const lru = getLRU();
            removeFromList(lru);
            cache.erase(lru->key);
        }
    }

private:
    // MRU: Most Recently Used
    void toMRU(Node* const node) {
        removeFromList(node);
        addToList(node);
    }

    void removeFromList(Node* const node) {
        node->prev->next = node->next;
        node->next->prev = node->prev;
        node->next = nullptr;
        node->prev = nullptr;
    }

    void addToList(Node* const node) {
        node->prev = &head;
        node->next = head.next;
        head.next->prev = node;
        head.next = node;
    }

    Node* getLRU() {
        if (tail.prev == &head) return nullptr;
        return tail.prev;
    }
};
```

## 4th

std::listを使って書いてみた。iteratorを管理する手間が増えるので、書く前に思ったよりも楽にはならなかった。
key_to_list_やrecently_used_order_list_など名前が微妙。

```cpp
class LRUCache {
public:
    LRUCache(int capacity) : capacity_(capacity), key_to_node_(), key_to_list_(), recently_used_order_list_() {}

    int get(int key) {
        if (!key_to_node_.contains(key)) return -1;
        toMRU(key_to_node_.at(key).get());
        return key_to_node_.at(key)->value;
    }

    void put(int key, int value) {
        auto [it, inserted] = key_to_node_.try_emplace(key, make_unique<Node>(key, value));
        Node* const node = it->second.get();
        if (!inserted) {
            node->value = value;
            toMRU(node);
            return;
        }
        recently_used_order_list_.push_front(node);
        key_to_list_.emplace(key, recently_used_order_list_.begin());
        if (recently_used_order_list_.size() > capacity_) {
            Node* const lru = recently_used_order_list_.back();
            recently_used_order_list_.pop_back();
            key_to_list_.erase(lru->key);
            key_to_node_.erase(lru->key);
        }
    }

private:
    struct Node {
        int key = 0;
        int value = 0;
        Node(int key, int value) : key(key), value(value) {}
    };

    void toMRU(Node* const node) {
        recently_used_order_list_.erase(key_to_list_.at(node->key));
        key_to_list_.erase(node->key);
        recently_used_order_list_.push_front(node);
        key_to_list_.emplace(node->key, recently_used_order_list_.begin());
    }

    int capacity_;
    unordered_map<int, unique_ptr<Node>> key_to_node_;
    unordered_map<int, list<Node*>::iterator> key_to_list_;
    list<Node*> recently_used_order_list_;
};
```

## 5th

```cpp
class LRUCache {
public:
    LRUCache(int capacity) : capacity_(capacity), nodes_(), key_to_node_() {}

    int get(int key) {
        auto it = key_to_node_.find(key);
        if (it == key_to_node_.end()) return -1;
        moveToFront(it->second);
        return it->second->value;
    }

    void put(int key, int value) {
        auto it = key_to_node_.find(key);
        if (it != key_to_node_.end()) {
            it->second->value = value;
            moveToFront(it->second);
            return;
        }
        nodes_.emplace_front(key, value);
        key_to_node_[key] = nodes_.begin();
        if (key_to_node_.size() > capacity_) {
            Node* const lru = &nodes_.back();
            key_to_node_.erase(lru->key);
            nodes_.pop_back();
        }
    }

private:
    struct Node {
        int key;
        int value;
        Node(int key, int value) : key(key), value(value) {}
    };

    void moveToFront(list<Node>::iterator it) {
        nodes_.splice(nodes_.begin(), nodes_, it);
    }

    int capacity_;
    list<Node> nodes_;
    unordered_map<int, list<Node>::iterator> key_to_node_;
};
```
