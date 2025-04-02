
# 修改 Trie 类实现写时复制功能

根据提供的头文件，我将指导你如何实现写时复制(COW) Trie。我们需要重点关注 `Put` 和 `Remove` 方法的实现。

## 1. 实现 `Put` 方法 (插入/更新)

```cpp
template <class T>
auto Trie::Put(std::string_view key, T value) const -> Trie {
    if (key.empty()) {
        return *this; // 空键不处理
    }
    
    // 从根节点开始递归处理
    auto new_root = PutImpl<T>(root_, key, std::make_shared<T>(std::move(value)), 0);
    return Trie(new_root); // 返回包含新根的新Trie
}

template <class T>
std::shared_ptr<const TrieNode> Trie::PutImpl(
    const std::shared_ptr<const TrieNode> &node,
    std::string_view key,
    std::shared_ptr<T> value,
    size_t depth) {
    
    // 1. 创建当前节点的新副本
    std::shared_ptr<TrieNode> new_node;
    if (!node) {
        new_node = std::make_shared<TrieNode>();
    } else {
        new_node = node->Clone();
    }
    
    // 2. 如果是最后一个字符
    if (depth == key.length() - 1) {
        // 创建带值的新节点
        return std::make_shared<TrieNodeWithValue<T>>(
            new_node->children_, std::move(value));
    }
    
    // 3. 递归处理下一个字符
    char current_char = key[depth];
    auto child = new_node->children_.count(current_char) 
        ? new_node->children_.at(current_char) 
        : nullptr;
    
    auto new_child = PutImpl<T>(child, key, std::move(value), depth + 1);
    
    // 4. 更新子节点映射(创建新map)
    auto new_children = new_node->children_;
    new_children[current_char] = new_child;
    
    // 5. 返回更新后的节点
    return std::make_shared<TrieNode>(new_children);
}
```

## 2. 实现 `Remove` 方法

```cpp
auto Trie::Remove(std::string_view key) const -> Trie {
    if (!root_ || key.empty()) {
        return *this; // 空树或空key直接返回原Trie
    }
    
    auto new_root = RemoveImpl(root_, key, 0);
    return Trie(new_root ? new_root : std::make_shared<TrieNode>());
}

std::shared_ptr<const TrieNode> Trie::RemoveImpl(
    const std::shared_ptr<const TrieNode> &node,
    std::string_view key,
    size_t depth) {
    
    // 1. 如果节点不存在，直接返回
    if (!node) {
        return nullptr;
    }
    
    // 2. 创建当前节点副本
    auto new_node = node->Clone();
    
    // 3. 到达key末尾
    if (depth == key.length()) {
        if (!new_node->is_value_node_) {
            return node; // 原节点不是值节点，无需修改
        }
        // 创建无值的新节点
        return std::make_shared<TrieNode>(new_node->children_);
    }
    
    // 4. 处理下一个字符
    char c = key[depth];
    if (!new_node->children_.count(c)) {
        return node; // 键不存在，返回原节点
    }
    
    // 5. 递归处理子节点
    auto child = new_node->children_.at(c);
    auto new_child = RemoveImpl(child, key, depth + 1);
    
    // 6. 更新子节点
    auto new_children = new_node->children_;
    if (new_child) {
        new_children[c] = new_child;
    } else {
        new_children.erase(c);
        // 如果当前节点无值且无子节点，可返回nullptr
        if (!new_node->is_value_node_ && new_children.empty()) {
            return nullptr;
        }
    }
    
    return std::make_shared<TrieNode>(new_children);
}
```

## 3. 实现 `Get` 方法

```cpp
template <class T>
auto Trie::Get(std::string_view key) const -> const T * {
    auto node = root_;
    for (char c : key) {
        if (!node || !node->children_.count(c)) {
            return nullptr;
        }
        node = node->children_.at(c);
    }
    
    // 检查是否是带值的节点
    if (node && node->is_value_node_) {
        auto value_node = dynamic_cast<const TrieNodeWithValue<T> *>(node.get());
        if (value_node) {
            return value_node->value_.get();
        }
    }
    return nullptr;
}
```

## 4. 关键实现要点

1. **写时复制原则**：
   - 每次修改都通过 `Clone()` 创建新节点
   - 未修改的子节点保持共享
   - 返回包含新根的新 `Trie` 对象

2. **递归处理**：
   - 沿 key 的字符路径递归处理
   - 每层都创建新节点并更新子节点映射

3. **内存管理**：
   - 使用 `shared_ptr` 自动管理节点生命周期
   - 通过 `const` 保证节点不可变

4. **边界条件处理**：
   - 空 key 处理
   - 不存在的 key 处理
   - 删除后空节点的清理

## 5. 测试建议

1. **基本功能测试**：
   - 插入新键值对
   - 更新已有键的值
   - 删除键值对
   - 查询不存在的键

2. **共享验证**：
   - 修改后检查原 Trie 是否保持不变
   - 验证未修改的节点是否共享

3. **性能测试**：
   - 大规模数据插入
   - 深层嵌套键的操作

这种实现方式完全符合写时复制的特性，保证了线程安全和高效的内存使用。每个操作都会返回一个新的 Trie 对象，而原始 Trie 保持不变。