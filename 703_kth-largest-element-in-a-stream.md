# 703. Kth Largest Element in a Stream

## 1st

所要時間: 9:23

n: 入力長
- 時間計算量: O(n * log(k))
- 空間計算量: O(k)

実装方針はすぐに立ったがprioirty_queueの変数名が思いつかず悩んでいた。
k_が1以上であることはコンストラクタでチェックすべきだった気がする。
昇順と降順のどちらが最小ヒープになるかいつも分からなくなる。

```cpp
class KthLargest {
public:
    KthLargest(int k, vector<int>& nums) : k_(k), largest_k_()  {
        for (const int num : nums) {
            add(num);
        }
    }

    int add(int val) {
        largest_k_.push(val);
        if (largest_k_.size() > k_) {
            largest_k_.pop();
        }
        return largest_k_.top();
    }

private:
    int k_;
    priority_queue<int, vector<int>, decltype([](int a, int b){ return a > b; })> largest_k_;
};

/**
 * Your KthLargest object will be instantiated and called as such:
 * KthLargest* obj = new KthLargest(k, nums);
 * int param_1 = obj->add(val);
 */
```

---

二分ヒープを自作したバージョン

所要時間: 29:40

n: 入力長
- 時間計算量: O(n * log(k))
- 空間計算量: O(k)

以前練習したので思い出しながらではあるが何とか書けた。
pop()の頭にはarr_.size() > 0であることをassertするなどした方が良さそう。
pop()のif文内の条件を間違えており1ミス。

追記: templateの後のangleとの間は一文字空ける方が主流か？ [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html) では空けている記述の方が多いが一部 (悪い例ではなく) 空けていないものも見られる。

```cpp
template<typename T>
class MinHeap {
public:
    void push(const T element) {
        arr_.push_back(element);
        size_t index = arr_.size() - 1;
        while (index > 0 && arr_[parentIndex(index)] > arr_[index]) {
            swap(arr_[parentIndex(index)], arr_[index]);
            index = parentIndex(index);
        }
    }

    void pop() {
        arr_[0] = arr_[arr_.size() - 1];
        arr_.pop_back();
        size_t index = 0;
        while (hasChildren(index)) {
            const size_t smaller_child_index = getSmallerChildIndex(index);
            if (arr_[index] <= arr_[smaller_child_index]) break;
            swap(arr_[index], arr_[smaller_child_index]);
            index = smaller_child_index;
        }
    }

    T top() const {
        assert(arr_.size() > 0);
        return arr_[0];
    }

    size_t size() const {
        return arr_.size();
    }

private:
    size_t parentIndex(const size_t index) const {
        assert(index > 0);
        return (index - 1) / 2;
    }

    bool hasChildren(const size_t index) const {
        return 2 * index + 1 < arr_.size();
    }

    size_t getSmallerChildIndex(const size_t index) const {
        assert(hasChildren(index));
        if (2 * index + 1 == arr_.size() - 1) {
            // has only left child
            return 2 * index + 1;
        }
        if (arr_[2 * index + 1] <= arr_[2 * index + 2]) {
            return 2 * index + 1;
        }
        return 2 * index + 2;
    }

    vector<T> arr_;
};

class KthLargest {
public:
    KthLargest(int k, vector<int>& nums) : k_largest_nums_(), k_(k) {
        for (const int num : nums) {
            add(num);
        }
    }

    int add(int val) {
        k_largest_nums_.push(val);
        if (k_largest_nums_.size() > k_) {
            k_largest_nums_.pop();
        }
        return k_largest_nums_.top();
    }

private:
    MinHeap<int> k_largest_nums_;
    int k_;
};
```

---

ヒープを使わず、やってきた数字を降順にソートしながら配列に入れたバージョン。
こちらもk_が1以上かをチェックしていない。

所要時間: 11:12

n: 入力長
- 時間計算量: O(n * k)
- 空間計算量: O(k)


```cpp
class KthLargest {
public:
    KthLargest(int k, vector<int>& nums) : k_largest_nums_(), k_(k) {
        for (const int num : nums) {
            insert(num);
        }
    }

    int add(int val) {
        insert(val);
        return k_largest_nums_[k_ - 1];
    }

private:
    void insert(const int num) {
        int position = lower_bound(k_largest_nums_.begin(), k_largest_nums_.end(), num, greater<int>()) - k_largest_nums_.begin();
        k_largest_nums_.push_back(num);
        for (int i = k_largest_nums_.size() - 1; i > position; i--) {
            swap(k_largest_nums_[i - 1], k_largest_nums_[i]);
        }
        if (k_largest_nums_.size() > k_) {
            k_largest_nums_.pop_back();
        }
    }

    vector<int> k_largest_nums_;
    int k_;
};
```


## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1227073733844406343/1240248383961305128
- https://discord.com/channels/1084280443945353267/1217527351890546789/1222093064399749180
    - コメントにあったcpythonのヒープ実装を読んだ https://github.com/python/cpython/blob/31a28cbae0989f57ad01b428c007dade24d9593a/Lib/heapq.py#L260
        - \_siftupでふるい上げるときは注目するノードの値に関わらず葉まで到達させている。heapの条件を満たすような位置に持ってくるのはふるい落とす\_siftdownに任せている
- https://discord.com/channels/1084280443945353267/1200089668901937312/1219685867409510670


## 3rd

省略

```cpp
class KthLargest {
public:
    KthLargest(int k, vector<int>& nums) : k_(k), k_largest_nums_() {
        assert(k > 0);
        for (const int num : nums) {
            add(num);
        }
    }

    int add(int val) {
        k_largest_nums_.push(val);
        if (k_largest_nums_.size() > k_) {
            k_largest_nums_.pop();
        }
        return k_largest_nums_.top();
    }

private:
    int k_;
    priority_queue<int, vector<int>, greater<int>> k_largest_nums_;
};
```
