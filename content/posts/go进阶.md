---
date: "2019-08-26T06:53:32.176Z"
title: go进阶
---
# Diagnostics

go提供了一系列诊断逻辑和性能问题的工具。

- profiling分析
- tracing跟踪
- debuging调试
- 运行时统计信息和事件

## Profiling

profiling信息可以在go test或者net/http/pprof包的时候使用。

runtime/pprof包有：

- cpu
  - 主动消费cpu周期所花费的时间，不包括睡眠或者io等待
- heap
  - 报告内存分配采样；
  - 当前或历史内存使用状况
  - 检测内存泄露
- threadcreate
  - 报告创建新的系统线程
- goroutine
  - 当前所有协程的堆栈跟踪
- block
  - 显示goroutine阻塞等待同步原语的位置。
  - 默认不开启，使用runtime.SetBlockProfileRate启用
- mutex
  - 报告锁竞争。
  - 如果认为自己的程序因为互斥锁导致cpu不能充分利用的时候，使用这个。
  - 默认也是不开启，使用 runtime.SetMutexProfileFraction 启用。

其他可用的的性能分析工具

Linux使用https://perf.wiki.kernel.org/index.php/Tutorial，perf可以分析cgo/SWIG代码和系统内核。

mac上使用 https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/InstrumentsUserGuide/ 就足够了。

分析线上处于生产状态服务

在生产上分析程序也是没问题的，但是开启某些指标会增加成本。

可视化分析数据

go 提供了很多可视化的工具，参考https://blog.golang.org/profiling-go-programs

也可以创建自定义的profil文件：参考https://golang.org/pkg/runtime/pprof/#Profile

也可以自定义修改pprof程序监听的端口和路径，参考：

```go
package main

import (
	"log"
	"net/http"
	"net/http/pprof"
)

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/custom_debug_path/profile", pprof.Profile)
	log.Fatal(http.ListenAndServe(":7777", mux))
}
```