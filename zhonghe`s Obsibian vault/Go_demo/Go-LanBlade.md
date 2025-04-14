
背景： 

单播： 用户的增加导致信息员负载加重 

❌ **扩展性差**：若需发送给多个目标，需建立多次连接（如视频会议中每个用户单独传输）。  
❌ **带宽浪费**：相同数据需多次传输（如直播场景下服务器需为每个用户发送一份数据流）。

广播： 造成网络资源浪费 

❌ **网络拥堵**：所有设备必须处理广播包，无论是否需要（尤其在大型网络中）。  
❌ **无法跨子网**：广播通常被路由器隔离，仅限本地局域网。  
❌ **安全性低**：任何设备均可接收广播数据，可能泄露信息。

组播： 只有加入我一组的设备 

- 实时音视频流（如直播、视频会议）。
    
- 分布式系统状态同步（如金融行情推送）。
    
- 局域网服务发现（如 mDNS）。


目的 想在局域网下 发现局域网下的设备 并 实现共享 在局域网内的设备间快速、无需配置地传输文件（可以提供一个简单的 Web 界面或命令行接口）。

**mDNS 使用组播（`224.0.0.251:5353`）** 正是因为它平衡了效率和范围，避免了广播的泛滥问题！






设想

```bash
mytool list             # 查看局域网内的设备列表
mytool send file.txt 192.168.1.8
mytool broadcast "吃饭啦！"
```


使用库

`github.com/hashicorp/mdns`


```go
package discover

  

import (

    "log"

    "net"

    "os"

    "strconv"

    "time"

  

    "github.com/hashicorp/mdns"

)

  

func main() {

    // 自定义 logger 以控制输出格式

    customLogger := log.New(os.Stdout, "[mDNS] ", log.LstdFlags|log.Lmicroseconds)

  

    entriesCh := make(chan *mdns.ServiceEntry, 32) // 更大的缓冲区

    defer close(entriesCh)

  

    // 高级发现协程

    go func() {

        devices := make(map[string]*mdns.ServiceEntry)

        timeout := time.After(10 * time.Second)

  

        for {

            select {

            case entry := <-entriesCh:

                if entry == nil || entry.AddrV4 == nil {

                    continue

                }

  

                // 复合键去重 (IP+Port+Name)

                key := entry.AddrV4.String() + ":" + strconv.Itoa(entry.Port) + "|" + entry.Name

                if _, exists := devices[key]; !exists {

                    devices[key] = entry

                    customLogger.Printf("发现 %-40s IP: %-15s 端口: %-5d 主机: %s",

                        truncateString(entry.Name, 35),

                        entry.AddrV4,

                        entry.Port,

                        truncateString(entry.Host, 25),

                    )

                }

  

            case <-timeout:

                if len(devices) == 0 {

                    customLogger.Println("\n⚠️ 未发现设备，诊断建议：")

                    customLogger.Println("1. 确保目标设备支持 mDNS:")

                    customLogger.Println("   - Windows: 安装 Bonjour (https://support.apple.com/kb/DL999)")

                    customLogger.Println("   - Linux:   sudo systemctl start avahi-daemon")

                    customLogger.Println("2. 网络检查:")

                    customLogger.Println("   - 执行 ping 224.0.0.251 (mDNS 组播地址)")

                    customLogger.Println("   - 检查防火墙: sudo ufw allow 5353/udp")

                    customLogger.Println("3. 高级测试:")

                    customLogger.Println("   - tcpdump -i eth0 udp port 5353")

                } else {

                    customLogger.Printf("\n✅ 扫描完成! 共发现 %d 个设备", len(devices))

                }

                return

            }

        }

    }()

  

    // 完整服务类型列表 (40+ 常见类型)

    services := []string{

        "_workstation._tcp", // 通用工作站

        "_smb._tcp",         // SMB文件共享

        "_SSH._tcp",         // SSH远程登录

    }

  

    // 获取所有网络接口

    interfaces, err := net.Interfaces()

    if err != nil {

        customLogger.Fatalf("获取网络接口失败: %v", err)

    }

  

    // 并行多接口查询

    for _, iface := range interfaces {

        // 跳过无效接口

        if iface.Flags&net.FlagUp == 0 || iface.Flags&net.FlagMulticast == 0 {

            continue

        }

  

        for _, service := range services {

            go func(svc string, iface net.Interface) {

                param := &mdns.QueryParam{

                    Service:             svc,

                    Domain:              "local",

                    Timeout:             2 * time.Second,

                    Entries:             entriesCh,

                    Interface:           &iface, // 指定网络接口

                    WantUnicastResponse: true,   // 请求单播响应

                    DisableIPv4:         false,

                    DisableIPv6:         true, // 完全禁用IPv6

                    Logger:              customLogger,

                }

  

                if err := mdns.Query(param); err != nil {

                    customLogger.Printf("接口 %s 查询 %s 失败: %v", iface.Name, svc, err)

                }

            }(service, iface)

        }

    }

  

    // 等待发现协程结束

    time.Sleep(12 * time.Second)

}

  

// 辅助函数：截断长字符串

func truncateString(s string, max int) string {

    if len(s) <= max {

        return s

    }

    return s[:max-3] + "..."

}
```

运行代码


使main.go调用！

**更好的选择：利用 Go 的交叉编译**

考虑到你的应用是一个本地网络工具，并且 Go 语言本身非常擅长**交叉编译**生成**单个静态链接的可执行文件**，这通常是分发此类工具**更简单、更方便**的方式：

1. **无需 Docker:** 用户不需要安装 Docker。
2. **平台原生:** 你可以为 Windows (`.exe`)、macOS (universal binary) 和 Linux (amd64, arm64 等) 分别编译出原生可执行文件。
    
    Bash
    
    ```
    # 编译 Linux amd64 版本
    GOOS=linux GOARCH=amd64 go build -o lan-toolkit-linux-amd64 .
    # 编译 Windows amd64 版本
    GOOS=windows GOARCH=amd64 go build -o lan-toolkit-windows-amd64.exe .
    # 编译 macOS amd64 版本
    GOOS=darwin GOARCH=amd64 go build -o lan-toolkit-macos-amd64 .
     # 编译 macOS arm64 (Apple Silicon) 版本
    GOOS=darwin GOARCH=arm64 go build -o lan-toolkit-macos-arm64 .
    # 编译 Linux arm64 (比如树莓派 64 位系统)
    GOOS=linux GOARCH=arm64 go build -o lan-toolkit-linux-arm64 .
    ```
    
3. **直接运行:** 用户下载对应其操作系统的文件，（可能需要给予执行权限 `chmod +x`），然后直接运行即可。
4. **无网络复杂性:** 程序直接运行在宿主机上，自然可以访问本地网络和 mDNS，没有 Docker 网络模式的麻烦。



使用scp 将打包好的文件 传入到我的服务器上 

`scp .\pi_device_scanner root@8.222.186.212:/root/`

