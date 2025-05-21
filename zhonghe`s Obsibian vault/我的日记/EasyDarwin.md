
#  Building on Windows


## make build/windows

首先make build/windows出现错误

![error](https://github.com/Zhonghe-zhao/My-Note/blob/main/zhonghe%60s%20Obsibian%20vault/%E5%9B%BE%E7%89%87%E5%AD%98%E6%94%BE/EasyDarwin/Pasted%20image%2020250518170245.png)


项目中提供了一个 `GetBuildTime()` 函数，会解析 `buildTimeAt` 为时间戳，作为构建时间展示。

在这过程中，我遇到两个问题：

1. **Git 解析失败**：由于 Git 命令没有添加 `--` 分隔，出现了 `ambiguous argument 'Try'` 的错误，后通过添加 `--` 或使用 `symbolic-ref` 修复。
    
2. **构建时报 undefined 错误**：main 函数调用了 `GetBuildTime()`，但该函数定义未包含在同一构建包中，修复方式是将定义放入相同的 `main` 包下，并确保 Makefile 构建时包含相关源码文件。


查找报错位置 main.go 的第52行！ 结合内容

```go
-ldflags="-s -w \  
    -X main.buildVersion=$(VERSION) \  
    -X main.gitBranch=$(BRANCH_NAME) \  
    -X main.gitHash=$(HASH_AND_DATE) \  
    -X main.buildTimeAt=$(shell date +%s) \  
    -X main.release=true \  
    " -o=$(dir)/EasyDarwin.exe ./cmd/server
```


使用命令 `go list -f '{{.GoFiles}}' ./cmd/server [main.go service.go wire.go]`发现`buildtime.go`并没有被编译  !!!

查看 文件发现 

`import "C"`

CGO 模式!  我进行了注释处理！

编译成功！

![succeed](https://github.com/Zhonghe-zhao/My-Note/blob/main/zhonghe%60s%20Obsibian%20vault/%E5%9B%BE%E7%89%87%E5%AD%98%E6%94%BE/EasyDarwin/Pasted%20image%2020250518185803.png)




## Getting Started Guide

![3](https://github.com/Zhonghe-zhao/My-Note/blob/main/zhonghe%60s%20Obsibian%20vault/%E5%9B%BE%E7%89%87%E5%AD%98%E6%94%BE/EasyDarwin/Pasted%20image%2020250518190201.png)


安装 FFmpeg 打开EasyDarwin


![4](https://github.com/Zhonghe-zhao/My-Note/blob/main/zhonghe%60s%20Obsibian%20vault/%E5%9B%BE%E7%89%87%E5%AD%98%E6%94%BE/EasyDarwin/Pasted%20image%2020250518192950.png)


## 创建推流

安装测试视频

`D:\vedio> curl -o oceans.mp4 http://vjs.zencdn.net/v/oceans.mp4`

在` D:\vedio>`下

使用命令

`ffmpeg -re -i ./oceans.mp4 -c copy -f flv -y rtmp://localhost:21935/live/stream_1?sign=ulSFVQGoTU


![5](https://github.com/Zhonghe-zhao/My-Note/blob/main/zhonghe%60s%20Obsibian%20vault/%E5%9B%BE%E7%89%87%E5%AD%98%E6%94%BE/EasyDarwin/Pasted%20image%2020250518193850.png)


![6](https://github.com/Zhonghe-zhao/My-Note/blob/main/zhonghe%60s%20Obsibian%20vault/%E5%9B%BE%E7%89%87%E5%AD%98%E6%94%BE/EasyDarwin/Pasted%20image%2020250518194415.png)


推流成功

## 拉流

![7](https://github.com/Zhonghe-zhao/My-Note/blob/main/zhonghe%60s%20Obsibian%20vault/%E5%9B%BE%E7%89%87%E5%AD%98%E6%94%BE/EasyDarwin/Pasted%20image%2020250518194859.png)


