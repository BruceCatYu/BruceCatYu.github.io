---
title: "go pprof"
tags: ["中文","pprof"]
categories: ["golang","进阶"]
cover: https://random.imagecdn.app/376/252
top_img: https://random.imagecdn.app/1920/400
date: 2022-09-22T16:22:23+08:00
---

## 简介  
Go pprof是官方工具，可以用来查找内存泄漏  

安装方法:  
```zsh
go install github.com/google/pprof@latest
```


## 内存泄漏分析  

在项目中穿插一点点代码
```go
	import _ "net/http/pprof"
```

```go
	go func() {
		http.ListenAndServe(":6060", nil)
	}()
```

重新编译启动后即可打开web界面查看各项数据(web界面略)  

可以先记录下初始的内存情况:
```zsh
curl -s http://ip:6060/debug/pprof/heap\?debug\=1 > base.heap
```

运行一段时间后，再次记录当前内存情况:  
```zsh
curl -s http://ip:6060/debug/pprof/heap\?debug\=1 > current.heap
```

此时可以拿两份文进行对比  
```zsh
go tool pprof --base base.heap current.heap
```
在里面输入:
```zsh
top
```
即可查看内存差异,效果:  
```zsh
Type: inuse_space
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) top
Showing nodes accounting for 42.08MB, 100% of 42.08MB total
      flat  flat%   sum%        cum   cum%
   42.08MB   100%   100%    42.08MB   100%  <unknown>
```

也可通过web服务查看:  
```zsh
go tool pprof --http :9090 --base base.heap current.heap
``` 
画面略
