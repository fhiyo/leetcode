# 2_add-two-numbers.md

## 1st

所要時間: 17:12

n: l1, l2それぞれのノード数の大きい方
- 時間計算量: O(n)
- 空間計算量: O(n)

一度解いたことがあるにしては時間がかかりすぎている。 `l1 || l2 || curry` のorをandと間違えて1ミス、解消に手間取る。

- 変数名addedはadded_value_nodeか。単にnodeでも十分かも。先頭がdummyで最初はdummyを指すので名前がつけづらい
- while内のvalという変数名はtotalくらいか。個人的にはvalを算出する計算が分かりやすいので許容範囲かという気はする

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
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode* dummy_head = new ListNode(0);
        ListNode* added = dummy_head;
        int curry = 0;
        while (l1 || l2 || curry) {
            const int val = getValueOrDefault(l1) + getValueOrDefault(l2) + curry;
            const int digit = val % 10;
            curry = val / 10;
            added->next = new ListNode(digit);
            added = added->next;
            l1 = getNextIfExists(l1);
            l2 = getNextIfExists(l2);
        }
        return dummy_head->next;
    }

private:
    static int getValueOrDefault(const ListNode* const node) {
        if (!node) return 0;
        return node->val;
    }

    static ListNode* getNextIfExists(const ListNode* const node) {
        if (!node) return nullptr;
        return node->next;
    }
};
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1231966485610758196/1237404231225577592
- https://discord.com/channels/1084280443945353267/1225849404037009609/1233844967777243318
- https://discord.com/channels/1084280443945353267/1233603535862628432/1233768573718106253
- https://discord.com/channels/1084280443945353267/1200089668901937312/1206949445359640576

再帰バージョンで書いた。

所要時間: 10:12

n: l1, l2それぞれのノード数の大きい方
- 時間計算量: O(n)
- 空間計算量: O(n)

リストのサイズが高々100なのでスタックオーバーフローすることはないだろう。
スタックサイズ8MBとして、スタックフレーム辺りの引数、ローカル変数でポインタ3つ、intが2つ + ベースポインタとリターンアドレスで48byte+αくらいだろうか？とすると 8*2**20/48 ≒ 170K回くらいは深くできそう。
(ポインタやレジスタのサイズを64bitとして計算している)

```cpp
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        return addTwoNumbersHelper(l1, l2, 0);
    }

private:
    static ListNode* addTwoNumbersHelper(
        const ListNode* const digit_node1,
        const ListNode* const digit_node2,
        const int curry
    ) {
        if (!digit_node1 && !digit_node2 && !curry) return nullptr;
        ListNode* const digit_node = new ListNode(0);
        const int total = getValOrDefault(digit_node1) + getValOrDefault(digit_node2) + curry;
        digit_node->val = total % 10; // digit
        digit_node->next = addTwoNumbersHelper(getNextIfExists(digit_node1), getNextIfExists(digit_node2), total / 10);
        return digit_node;
    }

    static int getValOrDefault(const ListNode* const node) {
        if (!node) return 0;
        return node->val;
    }

    static ListNode* getNextIfExists(const ListNode* const node) {
        if (!node) return nullptr;
        return node->next;
    }
};
```


## 3rd


```cpp
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode dummy_head = ListNode();
        ListNode* node = &dummy_head;
        int curry = 0;
        while (l1 || l2 || curry) {
            const int total = getValOrDefault(l1) + getValOrDefault(l2) + curry;
            curry = total / 10;
            node->next = new ListNode(total % 10); // digit
            node = node->next;
            l1 = getNextIfExists(l1);
            l2 = getNextIfExists(l2);
        }
        return dummy_head.next;
    }

private:
    static int getValOrDefault(const ListNode* const node) {
        if (!node) return 0;
        return node->val;
    }

    static ListNode* getNextIfExists(const ListNode* const node) {
        if (!node) return nullptr;
        return node->next;
    }
};
```

## 4th

```cpp
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode dummy_head = ListNode(-1);
        ListNode* node = &dummy_head;
        int carry = 0;
        while (l1 || l2 || carry) {
            const int total = getValOrDefault(l1, 0) + getValOrDefault(l2, 0) + carry;
            carry = total / 10;
            node->next = new ListNode(total % 10); // digit
            node = node->next;
            l1 = getNextIfExists(l1);
            l2 = getNextIfExists(l2);
        }
        return dummy_head.next;
    }

private:
    static int getValOrDefault(const ListNode* const node, const int default_value) {
        if (!node) return default_value;
        return node->val;
    }

    static ListNode* getNextIfExists(const ListNode* const node) {
        if (!node) return nullptr;
        return node->next;
    }
};
```
