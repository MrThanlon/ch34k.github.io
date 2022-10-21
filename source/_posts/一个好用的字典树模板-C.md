---
title: 一个好用的字典树模板(C++)
date: 2021-06-21 16:46:13
tags:
  - 字典树
  - 数据结构
  - ICPC
  - 刷题
categories:
  - 数据结构
  - 字典树
---

因为大多数题目的字符串都是小写字母，所以就按照这个设计了，共26个英文字母，每个节点占用26个char。

```c++
class TrieNode {
public:
    array<TrieNode *, 26> node;
    TrieNode() : node({nullptr}) {}
    void insert(string &s) {
        TrieNode *ptr = this;
        for (auto c: s) {
            c -= 'a';
            if (ptr->node[c] == nullptr) {
                ptr->node[c] = new TrieNode;
            }
            ptr = ptr->node[c];
        }
    }
    bool find(string &s) {
        TrieNode* ptr = this;
        for (auto c: s) {
            c -= 'a';
            if (ptr->node[c] == nullptr) {
                return false;
            } else {
                ptr = ptr->node[c];
            }
        }
        return true;
    }
};
```

