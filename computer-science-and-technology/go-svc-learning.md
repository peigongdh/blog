## go-svc源码分析

### 核心代码

```go
type Service interface {
	// Init is called before the program/service is started and after it's
	// determined if the program is running as a Windows Service. This method must
	// be non-blocking.
	Init(Environment) error

	// Start is called after Init. This method must be non-blocking.
	Start() error

	// Stop is called in response to syscall.SIGINT, syscall.SIGTERM, or when a
	// Windows Service is stopped.
	Stop() error
}
```

```go
// Run runs your Service.
//
// Run will block until one of the signals specified in sig is received.
// If sig is empty syscall.SIGINT and syscall.SIGTERM are used by default.
func Run(service Service, sig ...os.Signal) error {
	env := environment{}
	if err := service.Init(env); err != nil {
		return err
	}

	if err := service.Start(); err != nil {
		return err
	}

	if len(sig) == 0 {
		sig = []os.Signal{syscall.SIGINT, syscall.SIGTERM}
	}

	signalChan := make(chan os.Signal, 1)
	signalNotify(signalChan, sig...)
	<-signalChan

	return service.Stop()
}
```

### 信号

- 2) SIGINT
- 程序终止(interrupt)信号, 在用户键入INTR字符(通常是Ctrl-C)时发出，用于通知前台进程组终止进程。
- 15) SIGTERM
- 程序结束(terminate)信号, 与SIGKILL不同的是该信号可以被阻塞和处理。通常用来要求程序自己正常退出，shell命令kill缺省产生这个信号。如果进程终止不了，我们才会尝试SIGKILL。


### 分析

```go
signalChan := make(chan os.Signal, 1)
signalNotify(signalChan, sig...)
<-signalChan
```

- <-ch用来从channel ch中接收数据，这个表达式会一直被block，直到有数据可以接收。

## 扩展阅读

- go channel 详解

> https://colobu.com/2016/04/14/Golang-Channels/