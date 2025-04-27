---
title: 7_days_golang学习
tags:
  - gin
  - 7-days-golang
---

通过学习项目 7_days_golang搭建各种自己的框架了解一些网络工作原理和知识
---

学习geektutu源代码：https://github.com/geektutu/7days-golang


跟着学习的项目代码已经放在: https://github.com/Whuichenggong/Study_Go



### 1.gee.go

关键代码

```go
package gee

import (
	"fmt"
	"net/http"
)

//首先定义了类型HandlerFunc，这是提供给框架用户的，用来定义路由映射的处理方法

type HandlerFunc func(http.ResponseWriter, *http.Request)

// 在Engine中，添加了一张路由映射表router
// key 由请求方法和静态路由地址构成，例如GET-/、GET-/hello、POST-/hello
// 如果请求方法不同,可以映射不同的处理方法(Handler)，value 是用户映射的处理方法

type Engine struct {
	router map[string]HandlerFunc
}

// New is the constructor of gee.Engine
func New() *Engine {
	return &Engine{router: make(map[string]HandlerFunc)}
}

func (engine *Engine) addRoute(method string, pattern string, handler HandlerFunc) {
	key := method + pattern
	engine.router[key] = handler
}

// 用户调用(*Engine).GET()方法时，会将路由和处理方法注册到映射表 router 中，(*Engine).Run()方法，是 ListenAndServe 的包装。
func (engine *Engine) GET(pattern string, handler HandlerFunc) {
	engine.addRoute("GET", pattern, handler)
}

func (engine *Engine) POST(pattern string, handler HandlerFunc) {
	engine.addRoute("POST", pattern, handler)
}

func (engine *Engine) Run(addr string) (err error) {
	return http.ListenAndServe(addr, engine)
}

// Engine实现的 ServeHTTP 方法的作用就是，解析请求的路径，查找路由映射表，如果查到，就执行注册的处理方法。如果查不到，就返回 404 NOT FOUND 。
func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	key := req.URL.Path
	if handler, ok := engine.router[key]; ok {
		handler(w, req)
	} else {
		fmt.Fprintf(w, "404 Not Found: %s\n", req.URL)
	}

}

```


### 2.go.mod代码
```
module github.com/Whuichenggong

go 1.22.1

require gee v0.0.0

replace gee => ./gee

```
replace gee => ./gee

这是一个替换指令，它告诉 Go 工具链用本地相对路径 ./gee 中的 gee 包替换远程需要的 gee 包。
这意味着，尽管 require 指令可能指向一个特定的远程版本或分支，
这个 replace 指令实际上将使用当前目录下的 gee 文件夹中的代码。

#### 2.1初始化 Go 模块：
如果你的项目还没有被初始化为 Go 模块，你需要先在项目的根目录下运行以下命令来初始化它：

`go mod init <module-name>`

替换 <module-name> 为你的模块名称。例如，如果你的项目名称是 example，你会运行：

`go mod init example`

### 3.main.go

```
package main

import (
	"fmt"
	"net/http"

	"gee"
)

func main() {
	r := gee.New()
	r.GET("/", func(w http.ResponseWriter, req *http.Request) {
		fmt.Fprintf(w, "URL.Path = %q\n", req.URL.Path)
	})

	r.GET("/hello", func(w http.ResponseWriter, req *http.Request) {
		for k, v := range req.Header {
			fmt.Fprintf(w, "Header[%q] = %q\n", k, v)
		}
	})

	r.Run(":9999")
}



```
##### 新增：
测试 POST 请求
启动服务器后，测试 POST 请求可以使用以下工具：

方法 1: 使用 curl
执行以下命令发送 POST 请求：

curl -X POST http://localhost:8080/submit

---


```go
package main

import (
"fmt"
"io/ioutil"
"net/http"
)

type HandlerFunc func(http.ResponseWriter, *http.Request)

type Engine struct {
router map[string]HandlerFunc
}

func New() *Engine {
return &Engine{router: make(map[string]HandlerFunc)}
}

func (engine *Engine) addRoute(method string, pattern string, handler HandlerFunc) {
key := method + pattern
engine.router[key] = handler
}

func (engine *Engine) POST(pattern string, handler HandlerFunc) {
engine.addRoute("POST", pattern, handler)
}

func (engine *Engine) Run(addr string) error {
return http.ListenAndServe(addr, engine)
}

func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
key := req.Method + req.URL.Path
if handler, ok := engine.router[key]; ok {
handler(w, req)
} else {
http.NotFound(w, req)
}
}

```


func main() {
engine := New()

    // 注册一个 POST 路由
    engine.POST("/submit", func(w http.ResponseWriter, req *http.Request) {
        // 读取请求体数据
        body, err := ioutil.ReadAll(req.Body)
        if err != nil {
            http.Error(w, "Failed to read request body", http.StatusInternalServerError)
            return
        }
        // 响应请求体内容
        fmt.Fprintf(w, "Received: %s", string(body))
    })

    // 启动服务器
    engine.Run(":8080")
}
测试：
启动程序后，用 curl 发送 POST 请求并附带数据：

b
curl -X POST -d "data=HelloWorld" http://localhost:8080/submit
服务器返回：

kotlin

Received: data=HelloWorld



#### ServeHTTP好像有点问题

main.go
附带了对代码的理解

```go
package gee

import (
	"fmt"
	"net/http"
)

//首先定义了类型HandlerFunc，这是提供给框架用户的，用来定义路由映射的处理方法

type HandlerFunc func(http.ResponseWriter, *http.Request)

// 在Engine中，添加了一张路由映射表router
// key 由请求方法和静态路由地址构成，例如GET-/、GET-/hello、POST-/hello
// 如果请求方法不同,可以映射不同的处理方法(Handler)，value 是用户映射的处理方法

type Engine struct {
	router map[string]HandlerFunc
}

// New is the constructor of gee.Engine
func New() *Engine {
	return &Engine{router: make(map[string]HandlerFunc)}
}

// 这段代码的作用将HTTP请求的路由和对应的处理函数注册到路由表中的核心方法
// pattern路由路径
func (engine *Engine) addRoute(method string, pattern string, handler HandlerFunc) {
	//将HTTp方法和路径拼接成唯一一个键 作为路由表的router的键
	key := method + pattern
	//将处理函数 handler 存入路由表中，关联到对应的路由键。
	engine.router[key] = handler
}

// 用户调用 addRoute("GET", "/home", someHandlerFunc) 在 engine.router 映射表中，会存储一个键值对：
// 调用engine.GET("/home", someHandlerFunc)： 实际是 等价 engine.addRoute("GET", "/home", someHandlerFunc)
func (engine *Engine) GET(pattern string, handler HandlerFunc) {
	engine.addRoute("GET", pattern, handler)
}

func (engine *Engine) POST(pattern string, handler HandlerFunc) {
	engine.addRoute("POST", pattern, handler)
}

// 这段代码隐藏了调用ServeHTTP
func (engine *Engine) Run(addr string) (err error) {
	return http.ListenAndServe(addr, engine)
}

// Engine实现的 ServeHTTP 方法的作用就是，解析请求的路径，查找路由映射表，如果查到，就执行注册的处理方法。如果查不到，就返回 404 NOT FOUND 。
// 不需要显式调用 ServeHTTP
// 在 Go 的 HTTP 框架中，ServeHTTP 是 http.Handler 接口的约定方法。当你把 Engine 作为服务器的处理器传递时，它会被 ListenAndServe 自动调用。

func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	key := req.URL.Path
	if handler, ok := engine.router[key]; ok {
		handler(w, req)
	} else {
		fmt.Fprintf(w, "404 Not Found: %s\n", req.URL)
	}

}

```
##### 问题
在 ServeHTTP 中，当前只从 req.URL.Path 获取路径，
而没有结合 req.Method，会导致不同的 HTTP 方法（如 GET 和 POST）冲突或无法正确匹配。
addRoute 方法仅使用了路径（pattern）和方法（method）拼接为路由键，例如：GET/home。
```go
func (engine *Engine) addRoute(method string, pattern string, handler HandlerFunc) {
    key := method + "-" + pattern // 区分 HTTP 方法和路径
    engine.router[key] = handler
}
```
修改的这段代码
原来只使用了路径（req.URL.Path）作为路由键。例如：

请求路径 /hello 的键为 /hello。
不区分 GET /hello 和 POST /hello，它们会共用同一个路由键 /hello。
建议的代码
使用 HTTP 方法和路径 拼接成路由键。例如：

GET /hello 的键为 GET-/hello。
POST /hello 的键为 POST-/hello。
这样可以区分不同方法对应的路由处理函数。





与gin框架启动很相似

---
对Web服务来说，无非是根据请求*http.Request，构造响应http.ResponseWriter。
但是这两个对象提供的接口粒度太细，比如我们要构造一个完整的响应，需要考虑消息头(Header)和消息体(Body)，而 Header 包含了状态码(StatusCode)，
消息类型(ContentType)等几乎每次请求都需要设置的信息。因此，如果不进行有效的封装， 那么框架的用户将需要写大量重复，繁杂的代码
且容易出错。针对常用场景，能够高效地构造出 HTTP 响应是一个好的框架必须考虑的点。

代码要学会封装 否则代码整洁度看起来还是会差很多的 对于别人理解一会更方便


为什么要添加context  对于框架来说，还需要支撑额外的功能。例如，将来解析动态路由/hello/:name，参数:name的值放在哪呢？
再比如，框架需要支持中间件，那中间件产生的信息放在哪呢？

contxet保留了你想寻找的一些东西
拓展性和复杂性留在内部
对外简化了接口。

Context 的作用是为每个 HTTP 请求提供一个上下文对象，
方便操作请求和响应，并提供了一些简化开发的工具方法。
通过 Context 统一管理 HTTP 请求和响应的逻辑。


可以把 Context 看作是：

一个请求的容器： 它封装了与 HTTP 请求相关的所有信息，并提供了一些方法让你更轻松地操作这些信息。

开发者和 HTTP 请求的桥梁： 开发者通过 Context 与客户端通信，包括读取请求信息和发送响应。
```go
func handler(c *Context) {
// 获取查询参数
name := c.Query("name")

    // 构造 JSON 响应
    if name != "" {
        c.JSON(http.StatusOK, H{"message": "Hello " + name})
    } else {
        c.String(http.StatusBadRequest, "Name is required")
    }
}

```
1.
深入框架原理：
阅读 Gin、Echo 等框架的源码，了解它们如何设计和扩展 Context。

尝试扩展功能：
在 Context 上添加自定义方法，比如记录日志、追踪请求 ID 等。

2.
http.ResponseWriter 和 *http.Request 的实际意义
http.ResponseWriter

作用：
代表服务端用来写入 HTTP 响应的接口。开发者通过它向客户端返回数据（如响应头、响应状态码、响应体等）。
实际应用：
在服务端，http.ResponseWriter 将生成的 HTTP 响应数据写入 TCP 连接的输出流，客户端会接收到这些数据并解析呈现。
*http.Request

作用：
表示客户端发来的 HTTP 请求，包含了所有请求相关的信息（如 URL、方法、头部、表单数据、Cookie、Body 等）。
实际应用：
服务端根据 *http.Request 的内容（路径、方法等），判断客户端的需求并生成相应的响应。

```go
package main

import (
	"fmt"
	"net/http"
)

func handler(w http.ResponseWriter, req *http.Request) {
	// 设置响应头
	w.Header().Set("Content-Type", "text/plain")

	// 设置状态码
	w.WriteHeader(http.StatusOK)

	// 写入响应体
	fmt.Fprintf(w, "Hello, %s!\n", req.URL.Query().Get("name"))
}

func main() {
	http.HandleFunc("/", handler)
	http.ListenAndServe(":8080", nil)
}
```
客户端请求示例：

浏览器访问 http://localhost:8080/?name=zhaozhonghe

服务端响应：

HTTP/1.1 200 OK //设置的状态码 200
Content-Type: text/plain //设置的请求头 响应过来了 并且返回到了 客户端页面
Content-Length: 12

Hello, zhaozhonghe! // 读取 HTTP 请求 将数据写入响应体，通过 w 发送给客户端。

---

### 第二天

#### 1.添加context
```go
package gee

import (
	"encoding/json"
	"fmt"
	"net/http"
)

//对Web服务来说，无非是根据请求*http.Request，构造响应http.ResponseWriter

// 给map[string]interface{}起了一个别名gee.H，构建JSON数据时，显得更简洁。

type H map[string]interface{}

// Context目前只包含了http.ResponseWriter和*http.Request，另外提供了对 Method 和 Path 这两个常用属性的直接访问。

type Context struct {
	Writer http.ResponseWriter
	Req    *http.Request

	Path   string
	Method string

	StatusCode int
}

func newContext(w http.ResponseWriter, req *http.Request) *Context {
	return &Context{
		Writer: w,
		Req:    req,
		Path:   req.URL.Path,
		Method: req.Method,
	}
}

// 提供了访问Query和PostForm参数的方法。
func (c *Context) PostForm(key string) string {
	return c.Req.FormValue(key)
}

func (c *Context) Query(key string) string {
	return c.Req.URL.Query().Get(key)
}

func (c *Context) Status(code int) {
	c.StatusCode = code
	c.Writer.WriteHeader(code)
}

func (c *Context) SetHeader(key string, value string) {
	c.Writer.Header().Set(key, value)
}

// 提供了快速构造String/Data/JSON/HTML响应的方法。
func (c *Context) String(code int, format string, values ...interface{}) {
	c.SetHeader("Content-Type", "text/plain")
	c.Status(code)
	c.Writer.Write([]byte(fmt.Sprintf(format, values...)))
}

func (c *Context) JSON(code int, obj interface{}) {
	c.SetHeader("Content-Type", "application/json")
	c.Status(code)
	encoder := json.NewEncoder(c.Writer)
	if err := encoder.Encode(obj); err != nil {
		http.Error(c.Writer, err.Error(), 500)
	}
}

func (c *Context) Data(code int, data []byte) {
	c.Status(code)
	c.Writer.Write(data)
}

func (c *Context) HTML(code int, html string) {
	c.SetHeader("Content-Type", "text/html")
	c.Status(code)
	c.Writer.Write([]byte(html))
}

```



#### 2.添加router

想 路由需要的参数 路径 方法 处理函数

```go
package gee

import (
	"log"
	"net/http"
)

type router struct {
	handlers map[string]HandlerFunc
}

func newRouter() *router {
	return &router{handlers: make(map[string]HandlerFunc)}
}

func (r *router) addRoute(method string, pattern string, handler HandlerFunc) {
	log.Printf("Route %4s - %s", method, pattern)
	key := method + "-" + pattern
	r.handlers[key] = handler
}

func (r *router) handle(c *Context) {
	key := c.Method + "-" + c.Path
	if handler, ok := r.handlers[key]; ok {
		handler(c)
	} else {
		c.String(http.StatusNotFound, "404 NOT FOUND: %s\n", c.Path)
	}
}


```
r.handlers[key] = handler 这段代码将key也就是路径 和 处理函数关连到了一起

post用终端请求
1.
Invoke-WebRequest -Uri "http://localhost:9999/login" -Method POST -Body "username=zhaozhonghe&password=zzh123456"

2.
curl.exe -X POST -d "username=zhaozhonghe&password=zzh123456" http://localhost:9999/login 
返回结果
```shell
{"password":"zzh123456","username":"zhaozhonghe"}
```

测试第二天的gee
第一种返回结果
```shell
StatusCode        : 200
StatusDescription : OK
Content           : {"password":"zzh123456","username":"zhaozhonghe"}

RawContent        : HTTP/1.1 200 OK
Content-Length: 50
Content-Type: application/json
Date: Tue, 26 Nov 2024 13:43:37 GMT

                    {"password":"zzh123456","username":"zhaozhonghe"}

Forms             : {}
Headers           : {[Content-Length, 50], [Content-Type, application/json], [Date, Tue, 26 Nov 2024 13:43:37 GMT]}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 50
```


#### gee.go

```go
package gee

import "net/http"

// HandlerFunc defines the request handler used by gee
type HandlerFunc func(*Context)

// Engine implement the interface of ServeHTTP
type Engine struct {
	router *router
}

// New is the constructor of gee.Engine
func New() *Engine {
	return &Engine{router: newRouter()}
}

func (engine *Engine) addRoute(method string, pattern string, handler HandlerFunc) {
	engine.router.addRoute(method, pattern, handler)
}

// GET defines the method to add GET request
func (engine *Engine) GET(pattern string, handler HandlerFunc) {
	engine.addRoute("GET", pattern, handler)
}

// POST defines the method to add POST request
func (engine *Engine) POST(pattern string, handler HandlerFunc) {
	engine.addRoute("POST", pattern, handler)
}

// Run defines the method to start a http server
func (engine *Engine) Run(addr string) (err error) {
	return http.ListenAndServe(addr, engine)
}

func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	c := newContext(w, req)
	engine.router.handle(c)
}

```




## 第三天


我们用了一个非常简单的map结构存储了路由表，使用map存储键值对，索引非常高效，但是有一个弊端，键值对的存储的方式，只能用来索引静态路由。
那如果我们想支持类似于/hello/:name这样的动态路由怎么办呢？
所谓动态路由，即一条路由规则可以匹配某一类型而非某一条固定的路由。
例如/hello/:name，可以匹配/hello/geektutu、hello/jack等。
请等待~~~

11.21日看到了字节的课 是关于动态路由的设计 前缀匹配树




#### router.go
前缀树路由： 重点学习这个数据结构

bilibili: https://www.bilibili.com/video/BV1wT4y1x7xm?t=45.6


```go
package gee

import (
	"net/http"
	"strings"
)

type router struct {
	roots    map[string]*node //增加的
	handlers map[string]HandlerFunc
}

func newRouter() *router {
	return &router{
		roots:    make(map[string]*node),
		handlers: make(map[string]HandlerFunc),
	}
}

// Only one * is allowed
func parsePattern(pattern string) []string {
	vs := strings.Split(pattern, "/")

	parts := make([]string, 0)
	for _, item := range vs {
		if item != "" {
			parts = append(parts, item)
			if item[0] == '*' {
				break
			}
		}
	}
	return parts
}

func (r *router) addRoute(method string, pattern string, handler HandlerFunc) {
	parts := parsePattern(pattern)

	key := method + "-" + pattern
	_, ok := r.roots[method]
	if !ok {
		r.roots[method] = &node{}
	}
	r.roots[method].insert(pattern, parts, 0)
	r.handlers[key] = handler
}

func (r *router) getRoute(method string, path string) (*node, map[string]string) {
	searchParts := parsePattern(path)
	params := make(map[string]string)
	root, ok := r.roots[method]

	if !ok {
		return nil, nil
	}

	n := root.search(searchParts, 0)

	if n != nil {
		parts := parsePattern(n.pattern)
		for index, part := range parts {
			if part[0] == ':' {
				params[part[1:]] = searchParts[index]
			}
			if part[0] == '*' && len(part) > 1 {
				params[part[1:]] = strings.Join(searchParts[index:], "/")
				break
			}
		}
		return n, params
	}

	return nil, nil
}

func (r *router) getRoutes(method string) []*node {
	root, ok := r.roots[method]
	if !ok {
		return nil
	}
	nodes := make([]*node, 0)
	root.travel(&nodes)
	return nodes
}

func (r *router) handle(c *Context) {
	n, params := r.getRoute(c.Method, c.Path)
	if n != nil {
		c.Params = params
		key := c.Method + "-" + n.pattern
		r.handlers[key](c)
	} else {
		c.String(http.StatusNotFound, "404 NOT FOUND: %s\n", c.Path)
	}
}


```

parsePattern 函数的作用是解析路由路径，将路径按 / 分隔成各个部分。比如 /user/:id 会被分解成 ["user", ":id"]。
如果路径中出现了 *（通常用于匹配任意多的路径部分），解析会在遇到 * 时停止。比如 /files/*filepath 会解析成 ["files", "*filepath"]。
parts 数组存储了路由路径的各个部分（如静态部分、动态部分、通配符部分）


##### tire.go
```go
package gee

import (
	"fmt"
	"strings"
)

type node struct {
	pattern  string
	part     string
	children []*node
	isWild   bool
}

func (n *node) String() string {
	return fmt.Sprintf("node{pattern=%s, part=%s, isWild=%t}", n.pattern, n.part, n.isWild)
}

func (n *node) insert(pattern string, parts []string, height int) {
	if len(parts) == height {
		n.pattern = pattern
		return
	}
	part := parts[height]
	children := n.matchChildren(part)
	var child *node
	if len(children) == 0 {
		child = &node{part: part, isWild: part[0] == ':' || part[0] == '*'}
		n.children = append(n.children, child)
	} else {
		child = children[0] // 假设我们总是取第一个匹配的子节点
	}
	child.insert(pattern, parts, height+1)
}

func (n *node) search(parts []string, height int) *node {
	if len(parts) == height || strings.HasPrefix(n.part, "*") {
		if n.pattern == "" {
			return nil
		}
		return n
	}

	part := parts[height]
	children := n.matchChildren(part)

	for _, child := range children {
		result := child.search(parts, height+1)
		if result != nil {
			return result
		}
	}

	return nil
}

func (n *node) travel(list *([]*node)) {
	if n.pattern != "" {
		*list = append(*list, n)
	}
	for _, child := range n.children {
		child.travel(list)
	}
}

func (n *node) matchChild(part string) *node {
	for _, child := range n.children {
		if child.part == part || child.isWild {
			return child
		}
	}
	return nil
}

func (n *node) matchChildren(part string) []*node {
	nodes := make([]*node, 0)
	for _, child := range n.children {
		if child.part == part || child.isWild {
			nodes = append(nodes, child)
		}
	}
	return nodes
}

```

先学习一下前缀树

##### 定义树结点结构体
```go
type trieNode struct {
    nexts [26]*trieNode
    PassCnt int //用来记录中途是否有途径某个节点的个数
    end bool //匹配某个单词是否是结尾 比如seat的结尾是t
}
```
树
```go
type Trie struct {
    root *trieNode
}
```

```go
func Newtrie *Trie{
    return &Trie{
    root: &trieNode{},
}
}
```
##### 查询


```go
    func (t *Trie) Search(word string) bool {
    //查找目标节点，使根节点开始抵达目标节点沿路跟字符串恰好等于word
    node := t.search(word)
    return node != nil && node.end
}
```
##### Tire.search 方法源码


字符➖a
如果返回的单词是 前缀树中的别的单词的前缀判断
```go
func (t *Trie)search(target string)*trieNode{
//移动指针从根节点出发
move :t.root
/依次追历target中的每个字符
for_, ch:range target{
//倘若nexts中不存在对应于这个字符的节点，说明该单词没插入过，返回ni1
if move.nexts[ch-'a']==nil{

return nil
}
//指针向着子节点移动
movemove.nexts [ch-'a']
}

//来到末尾，说明已经完全匹配好单词，直接返回这个节点
//需要注意，找到目标节点不一定代表单词存在，因为该节点的end标识未必为true
//比如我们之前往trie中插入了apple这个单词，但是查找app这个单词时，预期的返回
return move


}

}

```

##### 前缀匹配
```go
//前缀树做前缀匹配很简单
    func (t *Trie) StartWith(prefix(string)) bool {
    return t.search(prefix) != nil
}
```
##### 前缀统计
```go
func (t *Trie) PassCnt(prefix string) int{
    node := t.search(prefix)
    if node == nil {
    return 0  
}
    return node.PassCnt
}


```
##### 插入单词
例子： 要插入apple 树中app可以复用 
    则插入 l e

```go
    func (t *Trie) Insert(word string) {
//如果单词存在直接返回
    if t.Search(word){
    return
}
        
        move := t.root


    for _,ch := range word {
//如果不存在创建出来
    if move.nexts[ch-'a'] == nil {
    move.nexts[ch-'a'] = &trieNode{}
}
    move.nexts[ch-'a'].passCnt++
    move = move.nexts[ch-'a']

move.end =true
}

```

##### 删除流程

```go
func (t *Trie) Erase(word string) bool {
    if !t.Search(word){
return false

}
  move := t.root
    for _, ch := range word {
    move.nexts[ch-'a'].passCnt --
if move.nexts[ch-'a'].passCnt == 0 {
    move.nexts[ch-'a'] = nil
    return true
}
    move = move.nexts[ch-'a']
}

    move.end = false
    return true
}

```

整段代码下来还是有点看不懂啊呜呜

11.25日
拖了几天 
感谢tutu
#### 分组控制

分组控制是Web框架的基础功能之一，路由的分组，往往某一组路由需要相似的处理

以/post开头的路由匿名可访问。
以/admin开头的路由需要鉴权。
以/api开头的路由是 RESTful 接口，可以对接第三方平台，需要三方平台鉴权。

/post是一个分组
/post/a和/post/b可以是该分组下的子分组
作用在/post分组上的中间件(middleware)，也都会作用在子分组，子分组还可以应用自己特有的中间件。

中间件可以给框架提供无限的扩展能力
用在分组上的效果也更明显
/admin的分组，可以应用鉴权中间件；/分组应用日志中间件，
/是默认的最顶层的分组，也就意味着给所有的路由，即整个框架增加了记录日志的能力。

一个 Group 对象需要具备哪些属性呢？首先是前缀(prefix)，
比如/，或者/api；要支持分组嵌套，那么需要知道当前分组的父亲(parent)是谁；
中间件是应用在分组上的，那还需要存储应用在该分组上的中间件(middlewares)。



```go
r := gee.New()
v1 := r.Group("/v1")
v1.GET("/", func(c *gee.Context) {
	c.HTML(http.StatusOK, "<h1>Hello Gee</h1>")
})
```
好好看看仓库中的代码 梳理思路 感觉好有意思但是看不懂哈哈哈哈

11.26日 回看前三天的代码 增加一些自己的理解和修改 再继续向下学习！ 

