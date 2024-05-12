# 206. Reverse Linked List

## 1st


- `if (!head || !head->next) return head;` の `!head->next` の条件は不要
- `node->next = nullptr;` の node は head にした方が意図が伝わりやすそう。reverseした後の末尾が何も指さなくするようにするための処理なので。

所要時間: 7:08

n: ノード数
- 時間計算量: O(n)
- 空間計算量: O(1)

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if (!head || !head->next) return head;
        ListNode* node = head;
        ListNode* next = head->next;
        node->next = nullptr;
        while (next) {
            ListNode* after_next = next->next;
            next->next = node;
            node = next;
            next = after_next;
        }
        return node;
    }
};
```

---

再帰バージョン

所要時間: 3:21

n: ノード数
- 時間計算量: O(n)
- 空間計算量: O(n)

基底条件で `!node->next` を入れておらず1ミス。reversed_headという名前だと伝わらないかも。reverse_list_headか

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* node) {
        if (!node || !node->next) return node;
        ListNode* const reversed_head = reverseList(node->next);
        node->next->next = node;
        node->next = nullptr;
        return reversed_head;
    }
};
```

---

上の再帰をなるべく忠実にスタック版に直したもの。息を吐くようにはまだまだできない。

- `if (!head || !head->next) return head;` の `!head->next` の条件は不要
- `go` は StackFrame ごとに持たなくても関数内で一つあれば良い
- `reversed_head` も関数内で一つで良いが、再帰版の再現のためにあえてこうした

所要時間: 12:59

n: ノード数
- 時間計算量: O(n)
- 空間計算量: O(n)

```cpp
struct StackFrame {
    bool go;
    ListNode* node;
    ListNode*& reversed_head;
};

class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if (!head || !head->next) return head;
        ListNode* reversed_head = nullptr;
        stack<StackFrame> node_info_stack = {};
        node_info_stack.push({true, head, reversed_head});
        while (!node_info_stack.empty()) {
            auto& [go, node, reversed_head] = node_info_stack.top();
            if (!node->next) {
                reversed_head = node;
                node_info_stack.pop();
                continue;
            }
            if (go) {
                node_info_stack.push({true, node->next, reversed_head});
                go = false;
                continue;
            }
            node_info_stack.pop();
            node->next->next = node;
            node->next = nullptr;
        }
        return reversed_head;
    }
};
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1200089668901937312/1204756236973776916
- https://discord.com/channels/1084280443945353267/1195700948786491403/1198919632908730488
    - スタックを使って解いたバージョンを見て、行きか帰りかのフラグを持たずとも先に全部スタックに入れてからpopしていけば十分だったと気づいた


```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if (!head) return nullptr;
        stack<ListNode*> nodes = {};
        ListNode* node = head;
        while (node) {
            nodes.push(node);
            node = node->next;
        }
        ListNode* const reversed_head = nodes.top();
        nodes.pop();
        while (!nodes.empty()) {
            ListNode * const node = nodes.top();
            nodes.pop();
            node->next->next = node;
            node->next = nullptr;
        }
        return reversed_head;
    }
};
```

## 3rd

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if (!head) return nullptr;
        ListNode* node = head;
        ListNode* next = head->next;
        head->next = nullptr;
        while (next) {
            ListNode* const after_next = next->next;
            next->next = node;
            node = next;
            next = after_next;
        }
        return node;
    }
};
```
