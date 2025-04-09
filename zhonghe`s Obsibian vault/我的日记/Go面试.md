## **uintptr 和 unsafe.Pointer的区别是什么**

 什么时候用哪个？

- **`unsafe.Pointer`**：用于在不同类型的 `*T` 之间强制转换。
    
- **`uintptr`**：用于做指针地址运算（偏移、比较等），但必须小心避免 GC 期间悬挂。