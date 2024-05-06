# 82_remove-duplicates-from-sorted-list-ii

## 1st

所要時間: 12:01

n: ノード数
- 時間計算量: O(n)
- 空間計算量: O(1)

deleteDuplicates内の `current != nullptr && current->next != nullptr` のandをorと間違えて書き1ミス。


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
        ListNode* const sentinel = new ListNode(0, head);
        ListNode* previous = sentinel;
        ListNode* current = head;
        while (current != nullptr && current->next != nullptr) {
            if (current->val == current->next->val) {
                current = skipDuplicateNodes(current);
                previous->next = current;
            } else {
                previous = current;
                current = current->next;
            }
        }
        ListNode* const new_head = sentinel->next;
        delete sentinel;
        return new_head;
    }

private:
    ListNode* skipDuplicateNodes(ListNode* node) const {
        // precondition: node != nullptr
        while (node->next != nullptr && node->val == node->next->val) {
            node = node->next;
        }
        return node->next;
    }
};
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1231966485610758196/1236735903729188874

こちらを参考に再帰バージョンで書く。

所要時間: 23:30

n: ノード数
- 時間計算量: O(n)
- 空間計算量: O(n)

適当にそれっぽいものを書いて、頭の中でデバッグして帳尻を合わせてしまった。もっと自然言語でどういう作業をするかを説明するという意識を持つべきかと感じた。あとwhileの部分とか、どこまで再帰でやるかという問題はあるかも

- deleteDuplicatesの `if (node == nullptr || node->next == nullptr) return node;` は無くても動く
- isDuplicateValueNodeはhasDuplicateValueの方がよいかも

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* node) {
        if (node == nullptr || node->next == nullptr) return node;
        while (isDuplicateValueNode(node)) {
            node = skipDuplicateNodes(node);
        }
        if (node != nullptr) {
            node->next = deleteDuplicates(node->next);
        }
        return node;
    }

private:
    bool isDuplicateValueNode(const ListNode* const node) const {
        return node != nullptr && node->next != nullptr && node->val == node->next->val;
    }

    ListNode* skipDuplicateNodes(ListNode* node) const {
        while (node->next != nullptr && node->val == node->next->val) {
            node = node->next;
        }
        return node->next;
    }
};
```

- https://discord.com/channels/1084280443945353267/1227073733844406343/1228549424351940668
- https://discord.com/channels/1084280443945353267/1217527351890546789/1221668820557496370
- https://discord.com/channels/1084280443945353267/1192728121644945439/1204829923705884702


## 3rd


```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        ListNode* const sentinel = new ListNode(0, head);
        ListNode* previous = sentinel;
        ListNode* current = head;
        while (current && current->next) {
            if (current->val == current->next->val) {
                current = skipDuplicateNodes(current);
                previous->next = current;
            } else {
                previous = current;
                current = current->next;
            }
        }
        ListNode* const new_head = sentinel->next;
        delete sentinel;
        return new_head;
    }

private:
    ListNode* skipDuplicateNodes(ListNode* node) const {
        // precondition: node != nullptr
        while (node->next && node->val == node->next->val) {
            node = node->next;
        }
        return node->next;
    }
};
```

## 4th

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        ListNode sentinel = ListNode(0, head);
        ListNode* node = &sentinel;
        while (node->next != nullptr && node->next->next != nullptr) {
            if (node->next->val == node->next->next->val) {
                node->next = skipDuplicateNodes(node->next);
            } else {
                node = node->next;
            }
        }
        return sentinel.next;
    }

private:
    ListNode* skipDuplicateNodes(ListNode* node) const {
        // precondition: node != nullptr
        while (node->next != nullptr && node->val == node->next->val) {
            node = node->next;
        }
        return node->next;
    }
};
```
