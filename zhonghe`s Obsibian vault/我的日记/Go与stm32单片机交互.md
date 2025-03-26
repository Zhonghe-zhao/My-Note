

```c
package main

  

import (

    "bufio"

    "fmt"

    "log"

    "strings"

  

    "github.com/jmoiron/sqlx"

    _ "github.com/lib/pq"

    "go.bug.st/serial"

)

  

const (

    SERIAL_PORT = "COM5" // 修改为你的实际COM口

    BAUD_RATE   = 9600   // 需与STM32程序一致

    DB_DSN      = "postgres://root:secret@localhost:5432/buttondb?sslmode=disable"

)

  

func main() {

    // 1. 检测可用串口（调试用）

    ports, err := serial.GetPortsList()

    if err != nil {

        log.Printf("警告：无法获取串口列表: %v", err)

    } else {

        fmt.Println("可用串口:", ports)

    }

  

    // 2. 打开串口

    port, err := serial.Open(SERIAL_PORT, &serial.Mode{

        BaudRate: BAUD_RATE,

        DataBits: 8,

        Parity:   serial.NoParity,

        StopBits: serial.OneStopBit,

    })

    if err != nil {

        log.Fatalf("无法打开串口 %s: %v", SERIAL_PORT, err)

    }

    defer port.Close()

    log.Printf("已连接串口: %s @ %d baud", SERIAL_PORT, BAUD_RATE)

  

    // 3. 连接数据库

    db, err := sqlx.Connect("postgres", DB_DSN)

    if err != nil {

        log.Fatalf("数据库连接失败: %v", err)

    }

    defer db.Close()

  

    // 4. 验证表结构（适配你现有的表）

    _, err = db.Exec("SELECT id, message, press_time FROM button_presses LIMIT 1")

    if err != nil {

        log.Fatalf("表结构验证失败: %v (请确认表名和字段是否正确)", err)

    }

  

    log.Println("系统就绪，等待数据...")

  

    // 5. 读取串口数据

    scanner := bufio.NewScanner(port)

    for scanner.Scan() {

        raw := strings.TrimSpace(scanner.Text())

        log.Printf("收到数据: %q", raw)

  

        // 处理数据并存入数据库

        switch {

        case raw == "0" || raw == "1":

            // 按钮状态（0/1）

            _, err := db.Exec(

                "INSERT INTO button_presses (message) VALUES ($1)",

                fmt.Sprintf("按钮状态: %s", raw),

            )

            if err != nil {

                log.Printf("写入失败: %v", err)

            } else {

                log.Printf("已记录按钮状态: %s", raw)

            }

  

        case strings.EqualFold(raw, "hello"):

            // hello消息

            _, err := db.Exec(

                "INSERT INTO button_presses (message) VALUES ($1)",

                "hello",

            )

            if err != nil {

                log.Printf("写入失败: %v", err)

            } else {

                log.Println("已记录hello消息")

            }

  

        default:

            log.Printf("忽略未知数据: %q", raw)

        }

    }

  

    if err := scanner.Err(); err != nil {

        log.Fatalf("串口读取错误: %v", err)

    }

}
```


这段程序 与单片机进行交互并且 接收到了单片机发送的数据hello