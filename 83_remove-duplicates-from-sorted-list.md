# 83_remove-duplicates-from-sorted-list

## 1st

所要時間: 3:51

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
    ListNode* deleteDuplicates(ListNode* head) {
        ListNode* node = head;
        while (node != nullptr && node->next != nullptr) {
            if (node->val == node->next->val) {
                node->next = node->next->next;
            } else {
                node = node->next;
            }
        }
        return head;
    }
};
```

---

再帰で解いた。返り値がvoidの関数でもほぼ反射で明示的にreturnを書いてしまうが、微妙だろうか。
早期returnしないでif-elseで分岐した方が見やすいかもしれない。

所要時間: 8:02

n: ノード数
- 時間計算量: O(n)
- 空間計算量: O(n) (再帰の深さがn)

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        deleteDuplicatesInternal(head);
        return head;
    }

private:
    void deleteDuplicatesInternal(ListNode* node) {
        if (node == nullptr || node->next == nullptr) return;
        if (node->val == node->next->val) {
            node->next = node->next->next;
            deleteDuplicatesInternal(node);
            return;
        }
        deleteDuplicatesInternal(node->next);
        return;
    }
};
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1231966485610758196/1234171208400896000
    - PR: https://github.com/TORUS0818/leetcode/pull/5

↑を参考にして、再帰のヘルパー関数いらないなと思い書き直した。
1stの解法とは違い手前に現れた重複するノードがスキップされる。

所要時間: 8:19 (どう書けばいいのかしばらく分からず、時間の割に苦戦していた)

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* node) {
        if (node == nullptr || node->next == nullptr) return node;
        if (node->val == node->next->val) {
            node = deleteDuplicates(node->next);
        } else {
            node->next = deleteDuplicates(node->next);
        }
        return node;
    }
};
```

- https://discord.com/channels/1084280443945353267/1227073733844406343/1233666371368390657
    - PR: https://github.com/sakupan102/arai60-practice/pull/3/files

↑を見て実装。今注目しているノードの値と次のノードの値が等しければ次のノードをスキップする。
こちらも紙に書きながらどうやるんだ...と悩んで時間をかけていた

所要時間: 14:11

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        if (head == nullptr) return nullptr;
        head->next = deleteDuplicatesInternal(head->next, head->val);
        return head;
    }

private:
    ListNode* deleteDuplicatesInternal(ListNode* const node, const int original_value) const {
        if (node == nullptr) return node;
        if (node->val == original_value) {
            return deleteDuplicatesInternal(node->next, original_value);
        }
        node->next = deleteDuplicatesInternal(node->next, node->val);
        return node;
    }
};
```


## 3rd

再帰の方が苦手だったのでそちらで実装。

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* node) {
        if (node == nullptr || node->next == nullptr) return node;
        if (node->val == node->next->val) {
            node = deleteDuplicates(node->next);
        } else {
            node->next = deleteDuplicates(node->next);
        }
        return node;
    }
};
```

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        if (head == nullptr) return nullptr;
        head->next = deleteDuplicatesInternal(head->next, head->val);
        return head;
    }

private:
    ListNode* deleteDuplicatesInternal(ListNode* node, const int original_val) const {
        if (node == nullptr) return node;
        if (node->val == original_val) {
            return deleteDuplicatesInternal(node->next, original_val);
        }
        node->next = deleteDuplicatesInternal(node->next, node->val);
        return node;
    }
};
```
