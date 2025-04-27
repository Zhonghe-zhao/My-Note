
### 运算符重载



### 智能指针

1. 智能指针时类模板 在栈上创建智能指针对象

2. 把普通指针交给智能指针对象

3. 智能指针对象过期时 调用析构函数释放普通指针的内存



相当于智能指针是一个对象 里面的成员包含原始指针




 *unique_ptr*

1. **`std::unique_ptr`**
    
    - 独占所有权指针
        
    - 同一时间只能有一个`unique_ptr`指向特定对象
        
    - 当指针离开作用域时，自动删除其所指对象
        
    - 不可复制，但可以移动

 资源管理方法

- **get()** `pointer get() const noexcept`
    
    - 返回存储的原始指针，但不释放所有权
        
    
    int* raw_ptr = ptr.get();
    
- **release()** `pointer release() noexcept`
    
    - 释放所有权，返回原始指针（调用者需负责删除）
        
    
    int* raw_ptr = ptr.release(); // ptr现在为空
    delete raw_ptr; // 需要手动删除
    
- **reset()** `void reset(pointer p = pointer()) noexcept`
    
    - 重置指针，删除当前管理的对象（如果有）




2. **`std::shared_ptr`**
    
    - 共享所有权指针
        
    - 多个`shared_ptr`可以指向同一对象
        
    - 使用引用计数跟踪有多少指针指向对象
        
    - 当最后一个`shared_ptr`被销毁时，对象被删除


shared_ptr <AA> p0 = make_shared<AA>("xishi")



3. **`std::weak_ptr`**
    
    - 弱引用指针
        
    - 不增加引用计数
        
    - 用于解决`shared_ptr`的循环引用问题
        
    - 必须转换为`shared_ptr`才能访问所指对象


make_unique 

![[shared_ptr.png]]


[[unique_ptr.png]]