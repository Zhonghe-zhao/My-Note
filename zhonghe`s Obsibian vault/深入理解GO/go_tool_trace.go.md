
```go
package main

import (

    "fmt"

    "os"

    "runtime/trace"

)


func main() {

    f, err := os.Create("trace.out")

    if err != nil {

        panic(err)

    }


    defer f.Close()

    err = trace.Start(f)

    if err != nil {

        panic(err)

    }

    defer trace.Stop()

  

    fmt.Println("Hello, world!")

}
```

~~~

go run go_tool_trace.go
Hello, world!

go tool trace trace.out
2025/03/30 21:03:58 Preparing trace for viewer...
2025/03/30 21:03:58 Splitting trace for viewer...
2025/03/30 21:03:58 Opening browser. Trace viewer is listening on http://127.0.0.1:63561

~~~

###  trace1


|            |           |                     |     |
| ---------- | --------- | ------------------- | --- |
| Goroutines | GCWaiting | 0.43050400000000005 | 0   |
| Goroutines | Runnable  | 0.43050400000000005 | 0   |
| Goroutines | Running   | 0.43050400000000005 | 2   |
为什么是两个核心在运行？


#### trace2

```go
package main

  

import "time"

  

func main() {

    for i := 0; i < 5; i++ {

        time.Sleep(1 * time.Second)

        println("Hello, World")

    }

  

}
```


~~~
$env:GODEBUG="schedtrace=1000"
PS E:\Go_Advanced\chaper1\cmd\trace2> go run .\deBug_trace.go
~~~

