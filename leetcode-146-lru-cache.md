## LRU Cache
题目来源于[LeetCode 146](https://leetcode.com/problems/lru-cache/#/description)
设计一个LRU cache, 要求get, put方法都是O(1)时间复杂度.

## 思路分析:

根据key, O(1)复杂度找到value, 只能用hash索引, 所以数据通过map存储. 我们要设计的是cache, 就要有淘汰机制, 我们把value设计成一个双向链表节点, 这样, 每次访问一个节点, 都把它移动到头结点. 当size > capacity时, 直接将tail节点删除.

另一个需注意的点是, 每次put前, 需要检查要插入的key是否已经存在, 如果已经存在, 只需要更新value和将节点前移到头结点.

#include <memory>
#include <map>
#include <iostream>

class Entry {
public:
    int key;
    int value;
    std::shared_ptr<Entry> next;
    std::shared_ptr<Entry> prev;

    Entry(int key, int value) : key(key), value(value), next(nullptr), prev(nullptr) {

    }
};

class LRUCache {
private:
    int capacity;
    int size;
    std::shared_ptr<Entry> head;
    std::shared_ptr<Entry> tail;
    std::map<int, std::shared_ptr<Entry>> data;

public:
    LRUCache(int capacity) : capacity(capacity), size(0), head(nullptr), tail(nullptr) {

    }

    int get(int key) {
        std::map<int, std::shared_ptr<Entry>>::iterator it = data.find(key);
        if (it == data.end()) {
            return -1;
        } else {
            if (head != it->second) {
                if (tail == it->second) {
                    tail = tail->prev;
                    tail->next = nullptr;

                    head->prev = it->second;
                    it->second->next = head;
                    head = it->second;
                    head->prev = nullptr;
                } else {
                    std::shared_ptr<Entry> pre = it->second->prev;
                    std::shared_ptr<Entry> nxt = it->second->next;
                    pre->next = nxt;
                    nxt->prev = pre;

                    it->second->next = head;
                    head->prev = it->second;

                    head = it->second;
                    head->prev = nullptr;
                }
            }
            return it->second->value;
        }
    }

    void put(int key, int value) {
        std::shared_ptr<Entry> node = std::make_shared<Entry>(key, value);
        bool checkSize = false;
        if (!head) {
            head = node;
            tail = node;
            data[key] = node;
            checkSize = true;
        } else {
            auto it = data.find(key);
            if (it == data.end()) {
                node->next = head;
                head->prev = node;
                head = node;
                data[key] = node;
                checkSize = true;
            } else {
                it->second->value = value;
                get(key);
            }
        }

        if (checkSize && (++size > capacity)) {
            std::shared_ptr<Entry> pre = tail->prev;
            pre->next = nullptr;
            data.erase(tail->key);
            tail = pre;
            --size;
        }
    }
};

int main(int argc, char** argv) {
    LRUCache cache(2);
    cache.put(1, 1);
    cache.put(2, 2);
    std::cout << cache.get(1) << std::endl;
    cache.put(3, 3);
    std::cout << cache.get(2) << std::endl;
    cache.put(4, 4);
    std::cout << cache.get(1) << std::endl;
    std::cout << cache.get(3) << std::endl;
    std::cout << cache.get(4) << std::endl;

    std::cout << "It works" << std::endl;
}
