# 141_linked-list-cycle

## 1st

所要時間: 6:50

n: ノード数
- 時間計算量: O(n)
- 空間計算量: O(1)

fast->next->next, slow->next のポインタの更新をhead->next->next, head->nextと書いており1ミス。

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    bool hasCycle(ListNode *head) {
        const ListNode* fast = head;
        const ListNode* slow = head;
        while (fast != nullptr && fast->next != nullptr) {
            fast = fast->next->next;
            slow = slow->next;
            if (fast == slow) return true;
        }
        return false;
    }
};
```

### setを用いた解法

所要時間: 7:00

n: ノード数
- 時間計算量: O(n)
- 空間計算量: O(n)

nodesという変数名は微妙か。visited_nodesくらい？

```cpp
class Solution {
public:
    bool hasCycle(ListNode *head) {
        unordered_set<const ListNode*> nodes;
        const ListNode* node = head;
        while (node != nullptr) {
            if (nodes.contains(node)) return true;
            nodes.insert(node);
            node = node->next;
        }
        return false;
    }
};
```


## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1231966485610758196/1233781489913692211
- https://discord.com/channels/1084280443945353267/1228896007279083653/1230164507670745210
- https://discord.com/channels/1084280443945353267/1206101582861697046/1230215117556285530
- 他の方のコードではconstが余り使われていないようなので[Google Style Guide](https://google.github.io/styleguide/cppguide.html#Use_of_const)で調べた。
  > Using const on local variables is neither encouraged nor discouraged.

  とのことで、ローカル変数についてはどちらでも良いという態度だった。個人的には認知負荷が下がるので不変であることはなるべく宣言したいと考えている (最適化の面もあるかもしれないが、いつ恩恵が受けられるのかは理解していない)。



他の方の回答を参考にしているうちに https://discord.com/channels/1084280443945353267/1206101582861697046/1231264494538330304 でinsertだけで判定できる話が上がっていたのを思い出し、そちらで実装してみた。

```cpp
class Solution {
public:
    bool hasCycle(ListNode *head) {
        unordered_set<const ListNode*> visited_nodes = {};
        const ListNode* node = head;
        while (node != nullptr) {
            const auto [it, inserted] = visited_nodes.insert(node);
            if (!inserted) return true;
            node = node->next;
        }
        return false;
    }
};
```


## 3rd

```cpp
class Solution {
public:
    bool hasCycle(ListNode *head) {
        const ListNode* fast = head;
        const ListNode* slow = head;
        while (fast != nullptr && fast->next != nullptr) {
            fast = fast->next->next;
            slow = slow->next;
            if (fast == slow) return true;
        }
        return false;
    }
};
```
