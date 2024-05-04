# 142_linked-list-cycle-ii

## 1st

所要時間: 16:29

空間計算量O(1)の方針は覚えていたものの、証明に時間がかかる。キャストしないとシグネチャが合わないことに気づかず1ミス。
findMeetingPointの返り値の型はポインタの先がconstなしの`ListNode*`の方が、ポインタが指すオブジェクトを変更しやすいのでいいかもしれない。
meetingPointだとサイクルの合流地点と思うだろうか？

n: ノード数
- 時間計算量: O(n)
- 空間計算量: O(1)

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
    ListNode *detectCycle(ListNode *head) {
        const ListNode* const meeting_point = findMeetingPoint(head);
        if (meeting_point == nullptr) return nullptr;
        const ListNode* from_head = head;
        const ListNode* from_meeting_point = meeting_point;
        while (from_head != from_meeting_point) {
            from_head = from_head->next;
            from_meeting_point = from_meeting_point->next;
        }
        return const_cast<ListNode*>(from_head);
    }

private:
    const ListNode* findMeetingPoint(const ListNode* const head) const {
        const ListNode* fast = head;
        const ListNode* slow = head;
        while (fast != nullptr && fast->next != nullptr) {
            fast = fast->next->next;
            slow = slow->next;
            if (fast == slow) return fast;
        }
        return nullptr;
    }
};

/*
  x   y
a---b---c
    |   |
    -----
      z

x, y, z: 距離
c: fastとslowの合流地点
[1]fast==slowの時点
fastの移動距離: x + (y+z)*m (m: 整数)
slowの移動距離: x + y
x+(y+z)*m = 2(x+y)
(y+z)*(m-1)+z = x
z === x (mod y+z)
*/
```

### setを用いた解法

所要時間: 2:17

n: ノード数
- 時間計算量: O(n)
- 空間計算量: O(n)

```cpp
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        unordered_set<const ListNode*> visited_nodes = {};
        const ListNode* node = head;
        while (node != nullptr) {
            if (visited_nodes.contains(node)) return const_cast<ListNode*>(node);
            visited_nodes.insert(node);
            node = node->next;
        }
        return nullptr;
    }
};
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1231966485610758196/1233764823943090236
- https://discord.com/channels/1084280443945353267/1228896007279083653/1231229102535479346
- https://discord.com/channels/1084280443945353267/1221030192609493053/1225103398962204686

所要時間: 4:39

```cpp
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        const ListNode* const meeting_point = findMeetingPoint(head);
        if (!meeting_point) return nullptr;
        const ListNode* from_head = head;
        const ListNode* from_meeting_point = meeting_point;
        while (from_head != from_meeting_point) {
            from_head = from_head->next;
            from_meeting_point = from_meeting_point->next;
        }
        return const_cast<ListNode*>(from_head);
    }

private:
    ListNode* findMeetingPoint(const ListNode* const head) const {
        const ListNode* fast = head;
        const ListNode* slow = head;
        while (fast && fast->next) {
            fast = fast->next->next;
            slow = slow->next;
            if (fast == slow) return const_cast<ListNode*>(fast);
        }
        return nullptr;
    }
};
```


## 3rd

省略
