

Go 语言的错误处理方式有很多，每种方式都有其适用场景，主要是为了**灵活处理不同类型的错误**，提高代码的**可读性、可维护性和健壮性**。以下是 Go 语言主要的错误处理方法及其作用：

---

## **1. `error` 接口（最基础的错误处理方式）**

### **作用**：

- Go 语言的 `error` 是一个接口，所有错误类型都实现了它。
- 通过 `errors.New()` 或 `fmt.Errorf()` 创建错误对象，并返回给调用者。

### **示例**：

```go
import (
	"errors"
	"fmt"
)

func divide(a, b int) (int, error) {
	if b == 0 {
		return 0, errors.New("cannot divide by zero")
	}
	return a / b, nil
}

func main() {
	res, err := divide(10, 0)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	fmt.Println("Result:", res)
}
```

✅ **适用场景**：简单的错误处理，比如参数校验、基本错误返回。

---

## **2. `fmt.Errorf`（格式化错误信息）**

### **作用**：

- 允许通过 `fmt.Errorf` 添加额外的上下文信息。
- 结合 `%w` 让错误支持**嵌套（wrapped errors）**，方便后续错误判断。

### **示例**：

```go
import "fmt"

func openFile(filename string) error {
	return fmt.Errorf("failed to open file %s: %w", filename, errors.New("file not found"))
}

func main() {
	err := openFile("data.txt")
	if err != nil {
		fmt.Println("Error:", err)
	}
}
```

✅ **适用场景**：当错误需要带上**额外的上下文信息**时，避免错误信息过于模糊。

---

## **3. `errors.Is` & `errors.As`（错误类型判断 & 解包）**

### **作用**：

- **`errors.Is`**：判断错误是否是某个特定的错误（即便错误被包装）。
- **`errors.As`**：将错误转换为某个具体的错误类型。

### **示例**：

```go
import (
	"errors"
	"fmt"
)

var ErrNotFound = errors.New("resource not found")

func findResource(id int) error {
	return fmt.Errorf("query failed: %w", ErrNotFound)
}

func main() {
	err := findResource(42)

	// 判断是否是特定错误
	if errors.Is(err, ErrNotFound) {
		fmt.Println("The resource was not found")
	}

	// 解析错误类型
	var targetErr *ErrCustom
	if errors.As(err, &targetErr) {
		fmt.Println("Custom error:", targetErr)
	}
}
```

✅ **适用场景**：

- `errors.Is` 适用于检查是否是**特定类型的错误**（比如 `os.ErrNotExist`）。
- `errors.As` 适用于**从通用错误中提取具体错误类型**（比如 `net.OpError`）。

---

## **4. `panic` 和 `recover`（极端情况下的错误处理）**

### **作用**：

- `panic` 会导致程序立即终止，除非被 `recover` 捕获。
- `recover` 只能在 `defer` 语句中使用，用于**防止程序崩溃**，一般用于**全局错误处理**。

### **示例**：

```go
func riskyOperation() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("Recovered from panic:", r)
		}
	}()
	
	panic("Something went terribly wrong!")
}

func main() {
	riskyOperation()
	fmt.Println("Program continues after panic recovery")
}
```

✅ **适用场景**：

- `panic` 适用于**不可恢复的错误**（如数组越界）。
- `recover` 用于**拦截 panic，防止程序崩溃**（如 Web 服务器避免单个请求崩溃整个服务）。

⚠️ **一般情况下，不推荐使用 `panic/recover` 进行业务错误处理**，应优先返回 `error`。

---

## **5. 自定义错误类型（更丰富的错误信息）**

### **作用**：

- 自定义 `error` 结构体，让错误携带**更多信息**。
- 适用于**复杂系统**，提供更好的错误追踪。

### **示例**：

```go
import "fmt"

// 自定义错误类型
type MyError struct {
	Code    int
	Message string
}

func (e *MyError) Error() string {
	return fmt.Sprintf("Error %d: %s", e.Code, e.Message)
}

func doSomething() error {
	return &MyError{Code: 404, Message: "Not Found"}
}

func main() {
	err := doSomething()
	if err != nil {
		fmt.Println(err)
	}
}
```

✅ **适用场景**：适用于**API、数据库、网络请求等**需要携带额外信息的错误。

---

## **6. `log` 包（记录错误日志）**

### **作用**：

- 记录错误信息，方便**调试和运维**。
- `log.Fatal` 会终止程序，`log.Println` 只是打印日志。

### **示例**：

```go
import (
	"log"
	"os"
)

func main() {
	_, err := os.Open("nonexistent.txt")
	if err != nil {
		log.Println("Error:", err)  // 记录错误
	}
}
```

✅ **适用场景**：适用于**生产环境**，可以记录错误信息进行排查。

---

## **7. `defer`（确保资源释放）**

### **作用**：

- `defer` 确保资源（如文件、数据库连接）**一定会被释放**，避免资源泄露。

### **示例**：

```go
import (
	"fmt"
	"os"
)

func readFile(filename string) error {
	file, err := os.Open(filename)
	if err != nil {
		return err
	}
	// 确保文件被关闭
	defer file.Close()

	// 读取文件内容...
	fmt.Println("File opened successfully")
	return nil
}
```

✅ **适用场景**：适用于**文件、数据库连接、锁等资源管理**。

---

## **总结**

|方法|适用场景|备注|
|---|---|---|
|`error` 接口|最基础的错误处理|返回 `error`，由调用方决定如何处理|
|`fmt.Errorf("%w")`|添加上下文信息|方便追踪错误来源|
|`errors.Is`|判断错误类型|适用于嵌套错误|
|`errors.As`|提取具体错误类型|适用于复杂错误类型|
|`panic/recover`|处理不可恢复的错误|适用于全局错误拦截|
|自定义错误类型|需要携带更多信息|适用于 API、数据库、业务逻辑|
|`log` 记录错误|记录日志，方便调试|`log.Fatal` 会终止程序|
|`defer`|资源管理|确保 `Close()`、`Unlock()` 被调用|

---

## **为什么 Go 需要这么多错误处理方式？**

1. **不同场景适用不同方式**（比如 `error` 适用于一般业务逻辑，`panic/recover` 适用于极端情况）。
2. **灵活性和可读性**（可以选择最适合的方式处理错误，而不是一种方式适用于所有情况）。
3. **提高可维护性**（例如 `errors.Is`、`errors.As` 让错误处理更清晰，避免硬编码字符串比较）。
4. **增强错误追踪能力**（如 `fmt.Errorf("%w")` 提供更详细的错误信息）。

---

### **最佳实践**

- **优先返回 `error`，避免 `panic`，除非是不可恢复的错误**。
- **使用 `errors.Is` / `errors.As` 进行错误分类**。
- **日志记录 (`log.Println`) 只打印，不影响程序流程**。
- **资源管理使用 `defer`，避免手动释放导致的资源泄露**。

你现在遇到的错误处理问题是哪种？如果有具体代码，可以一起优化！