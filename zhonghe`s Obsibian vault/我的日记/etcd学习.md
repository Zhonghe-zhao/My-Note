
## 在linux中如何在go.mod中快速查找某一个模块路径


`root@xiaoxinxiaohao:/etcd_source_code/etcd# grep raft go.mod
        go.etcd.io/raft/v3 v3.6.0-beta.0`


`root@xiaoxinxiaohao:/etcd_source_code/etcd# go list -m all | grep raft

go.etcd.io/raft/v3 v3.6.0-beta.0

root@xiaoxinxiaohao:/etcd_source_code/etcd# go list -m -f '{{.Dir}}' go.etcd.io/raft/v3

'/root/go/pkg/mod/go.etcd.io/raft/v3@v3.6.0-beta.0`



```shell
dlv exec ./bin/etcd -- --name s1 --listen-client-urls http://0.0.0.0:2379 \
           --advertise-client-urls http://0.0.0.0:2379 \
           --listen-peer-urls http://0.0.0.0:2380 \
           --initial-advertise-peer-urls http://0.0.0.0:2380 \
           --initial-cluster s1=http://0.0.0.0:2380,s2=http://0.0.0.0:2381,s3=http://0.0.0.0:2382 \
           --initial-cluster-token tkn \
           --initial-cluster-state new

```


这个错误表明，调试器无法确定 `raft.go:30` 的确切位置，因为它在多个包中都有定义：一个是在 `go.etcd.io/etcd/server/v3/etcdserver/raft.go` 中，另一个是在 `go.etcd.io/raft/v3@v3.6.0-beta.0/raft.go` 中。

为了让 `dlv` 明确地知道你想调试哪个文件，应该指定完整路径。

### 解决方法

#### 1. **明确指定完整路径**

你可以通过完整路径来指明你要断点的文件，确保调试器知道你要设置断点的位置。

如果你想在 `go.etcd.io/etcd/server/v3/etcdserver/raft.go` 中设置断点，可以这样：

```bash
(dlv) break go.etcd.io/etcd/server/v3/etcdserver/raft.go:30
```

同样，如果你想在另一个版本的 `raft.go` 设置断点（例如，`go.etcd.io/raft/v3@v3.6.0-beta.0/raft.go`），可以指定相应的路径：

```bash
(dlv) break go.etcd.io/raft/v3@v3.6.0-beta.0/raft.go:30
```

#### 2. **确认源代码位置**

如果你不确定要调试的文件位置，可以通过列出调试器加载的源代码来确认。例如，使用 `list` 命令查看文件内容：

```bash
(dlv) list go.etcd.io/etcd/server/v3/etcdserver/raft.go
```

或者：

```bash
(dlv) list go.etcd.io/raft/v3@v3.6.0-beta.0/raft.go
```

这将帮助你确认文件是否已正确加载，并确定你要调试的位置。

### 总结

由于有多个版本的 `raft.go` 文件，`dlv` 无法确定设置断点的位置。通过指定文件的完整路径，你可以明确告诉调试器在哪个文件设置断点。