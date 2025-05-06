

### **可变参数用法**

1. **只能有一个可变参数**，且必须是最后一个参数：

```go


    func foo(a int, b ...string) {}  // 正确
    func bar(a ...int, b string) {}   // 错误：可变参数必须在最后
    
```


```go
2. **可变参数的类型必须相同**：
    func baz(a ...int) {}    // 正确
    func qux(a ...any) {}    // 正确（Go 1.18+ 泛型）
    func quux(a ...int, b ...string) {}  // 错误：只能有一个可变参数
```



# `context.WithoutCancel` 解析

`ctx, cancel := context.WithoutCancel(context.Background())` 是 Go 1.21 版本引入的一个新 Context 函数，它的作用是**创建一个不受父 Context 取消影响的新 Context**。


