---
title: Go语言聊天系统
tags:
  - Go管道
  - Angular_CLI
---



后端已经放到github仓库

## 12.30日
学习使用 Angular_CLI

1. `npm install -g @angular/cli`
2. `>ng new my-app`
3. `>cd my-app`
4. `ng serve` 启动服务

如果你想修改页面内容，可以编辑项目中的 `src/app/app.component.html` 文件，保存后浏览器会自动刷新显示更新。

5. `ng g service socket`

  
此提供程序将是可注入的，并根据某些事件发出数据。在构造函数方法中，建立了与 Golang 应用程序的 websocket 连接，并创建了三个事件侦听器。每个套接字创建和销毁都有一个事件侦听器，以及一个消息传入时的侦听器。