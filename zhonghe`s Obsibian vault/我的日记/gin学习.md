
api 最先创建的文件


## server.go

Validation 防止用户传入一些无用的数据


```go
type LoginRequest struct {
    Username string `json:"username" binding:"required"`
    Password string `json:"password" binding:"required,min=6"`
}

func LoginHandler(c *gin.Context) {
    var req LoginRequest
    
    // 关键操作：绑定并验证
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }
    
    // 绑定成功后使用 req.Username 和 req.Password
}
```


```go
  
func (server *Server) createAccount(ctx *gin.Context) {  
    var req CreateAccountRequest  
    //解析请求体中的 JSON 并尝试将其映射到 req 变量  
    if err := ctx.ShouldBindJSON(&req); err != nil {  
       ctx.JSON(http.StatusBadRequest, errorResponse(err))  
       return  
    }  
  
    authPayload := ctx.MustGet(authorizationPayloadKey).(*token.Payload)  
    arg := db.CreateAccountParams{  
       Owner:    authPayload.Username, ///疑问为什么能返回username  
       Currency: req.Currency,  
       Balance:  0,  
    }  
    //如果省下以下这段store调用CreateAccount，则在测试的时候会提示缺少调用方法  
    account, err := server.store.CreateAccount(ctx, arg)  
    if err != nil {  
  
       errCode := db.ErrorCode(err)  
       if errCode == db.ForeignKeyViolation || errCode == db.UniqueViolation {  
          ctx.JSON(http.StatusForbidden, errorResponse(err))  
          return  
       }  
       ctx.JSON(http.StatusInternalServerError, errorResponse(err))  
       return  
    }  
    ctx.JSON(http.StatusOK, account)  
}
```

常用的  ` errorResponse` 函数


## 为什么不直接返回err？ 而是要创建一个 errorResponse函数




理解 `err` 和 `errorResponse(err)` 的区别，关键在于 **错误的处理与展示方式**。

### 1. **`err` 是 Go 的原始错误对象**

`err` 是 Go 中的一个 **错误对象（`error` 类型）**。当函数返回错误时，它是 `error` 接口类型的实例，包含了一个描述错误的消息。这个错误对象本身是 **系统内部的**，主要是给开发人员用来追踪和调试的。

例如，假设你有一个错误：

```go
err := errors.New("数据库连接失败")
```

这个错误本身不会暴露详细信息给前端，它仅仅是一个字符串，描述了问题的原因（例如 `"数据库连接失败"`），但并不包含更多的上下文信息。

### 2. **`errorResponse(err)` 是一个统一的错误响应格式**

`errorResponse(err)` 这个方法的作用是 **将 Go 错误（`err`）转换成一种更适合 API 返回的格式**。通常，我们希望在 API 返回错误时，返回给客户端一个 **标准化且安全的格式**，避免暴露过多内部细节。

`errorResponse(err)` 是把原始的 `err` 错误对象的消息提取出来，包装成一个结构化的 JSON 格式（`gin.H`）：

```go
func errorResponse(err error) gin.H {
	return gin.H{"error": err.Error()}
}
```

其中：

- `err.Error()` 提取了错误对象中的描述性消息（通常是一个字符串），而不是错误本身。
    
- 返回的是一个 **JSON 格式**，包含错误信息：`{"error": "数据库连接失败"}`，这样前端就能看到“数据库连接失败”的消息。
    

### 3. **为什么不能直接返回 `err`？**

- **Go 的 `err` 不是一个可以直接返回给前端的对象**。它是一个 **接口类型**，并没有直接包含 JSON 格式化的信息。
    
- 如果你直接返回 `err`，例如：
    
    ```go
    ctx.JSON(http.StatusInternalServerError, err)
    ```
    
    **Go 会尝试把 `err` 转换为 JSON 格式**，这通常会返回类似下面的结构：
    
    ```json
    {
      "Op": "query",
      "Path": "SELECT * FROM users",
      "Err": "database connection failed"
    }
    ```
    
    这个结构显然对前端没有什么帮助，甚至还泄露了系统的底层实现（如 SQL 查询语句、数据库连接等信息），这在生产环境中是非常危险的。
    

### 4. **总结区别**

- `err` 是 Go 内部的 **错误对象**，它包含了关于错误的详细信息，但并不适合直接返回给客户端。
    
- `errorResponse(err)` 是 **一个格式化后的错误响应**，将错误对象的消息提取并转化成一个适合 API 返回的格式，保证前端能够看到清晰的错误信息，同时避免泄露内部细节。
    

### 举个例子：

假设在你的项目中，数据库发生了错误，`err` 可能是：

```go
err := errors.New("database connection failed")
```

如果你直接返回 `err`：

```go
ctx.JSON(http.StatusInternalServerError, err)
```

返回给前端的响应可能是：

```json
{
  "Op": "query",
  "Path": "SELECT * FROM users",
  "Err": "database connection failed"
}
```

但如果你使用 `errorResponse(err)`：

```go
ctx.JSON(http.StatusInternalServerError, errorResponse(err))
```

返回给前端的响应是：

```json
{
  "error": "database connection failed"
}
```

### 为什么使用 `errorResponse`？

1. **简洁明了**：给前端返回一个简洁的错误消息，易于处理和展示。
    
2. **安全**：避免暴露数据库操作、系统内部路径、SQL 查询等敏感信息。
    
3. **一致性**：统一所有错误的返回格式，使得前端和开发人员在错误处理上达成一致。
    

---

希望这次解释能让你更加清楚 `err` 和 `errorResponse(err)` 的区别！



## 对于router.Run的封装


这段代码的核心思想：**将服务启动逻辑封装起来，提供统一入口。**

```go
func (server *Server) Start(address string) error {
    return server.router.Run(address)
}
```

### **目的简述**

这个方法的作用就是：**启动 HTTP 服务器，让服务运行在指定地址上。**

### **为什么要这样封装？挑重点讲：**

---

#### ✅ 1. **隐藏细节，统一调用方式**

你不需要在 `main.go` 里关心 `gin` 具体怎么启动，只需要：

```go
server.Start(":8080")
```

清晰明了，一看就知道是“启动服务”。

---

#### ✅ 2. **以后可以扩展**

未来你想加点功能，比如：

- 启动前打印日志
    
- 做端口检查
    
- 做预加载数据、定时任务
    

只要在 `Start()` 里改一行代码，外部调用 `Start()` 的方式完全不变。

```go
func (server *Server) Start(address string) error {
    log.Println("Starting server at", address)
    preloadCache()
    return server.router.Run(address)
}
```

---

#### ✅ 3. **符合面向对象思想：职责单一**

`Server` 是你的服务对象，它应该负责启动自己。

不应该让外部函数（比如 `main()`）去操作内部组件（比如 `router`），否则耦合性太高。

---

### 一句话总结：

> `Start()` 是对服务启动逻辑的封装，使代码更清晰、更容易维护、扩展性更强。

你现在是在写结构化、可维护的大型项目，这种封装是必须的。需要我顺带解释 `Server` 是怎么设计的吗？



## 主main.go




## error.go

这个 `error.go` 文件的目的是：

> **对底层数据库的错误进行封装，让你在业务代码中写得更清晰、更安全、更易维护。**


## Viper

从文件或环境变量中读取加载配置

取消代码中的硬编码例如

const （
		
）


### app.env




