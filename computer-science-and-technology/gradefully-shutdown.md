## 优雅重启

本文参考如下博客
> https://juejin.im/post/6867074626427502600
结合代码分析


## 如何实现

> 实现目的

> - 不关闭现有连接（正在运行中的程序）。
> - 新的进程启动并替代旧进程。
> - 新的进程接管新的连接。
> - 连接要随时响应用户的请求。当用户仍在请求旧进程时要保持连接，新用户应请求新进程，不可以出现拒绝请求的情况。

> 实现原理

> - 监听 SIGHUP 信号;
> - 收到信号后将服务监听的文件描述符传递给新的子进程;
> - 此时新老进程同时接收请求;
> - 父进程停止接收新请求, 等待旧请求完成(或超时);
> - 父进程退出.

## 代码分析

go http库shutdown

```go
// Shutdown gracefully shuts down the server without interrupting any
// active connections. Shutdown works by first closing all open
// listeners, then closing all idle connections, and then waiting
// indefinitely for connections to return to idle and then shut down.
// If the provided context expires before the shutdown is complete,
// Shutdown returns the context's error, otherwise it returns any
// error returned from closing the Server's underlying Listener(s).
//
// When Shutdown is called, Serve, ListenAndServe, and
// ListenAndServeTLS immediately return ErrServerClosed. Make sure the
// program doesn't exit and waits instead for Shutdown to return.
//
// Shutdown does not attempt to close nor wait for hijacked
// connections such as WebSockets. The caller of Shutdown should
// separately notify such long-lived connections of shutdown and wait
// for them to close, if desired. See RegisterOnShutdown for a way to
// register shutdown notification functions.
//
// Once Shutdown has been called on a server, it may not be reused;
// future calls to methods such as Serve will return ErrServerClosed.
func (srv *Server) Shutdown(ctx context.Context) error {
	// 在原子的存储某个值的过程中，任何cpu都不会进行针对进行同一个值的读或写操作。
	// 如果我们把所有针对此值的写操作都改为原子操作，那么就不会出现针对此值的读操作读操作因被并发的进行而读到修改了一半的情况。
	atomic.StoreInt32(&srv.inShutdown, 1)

	srv.mu.Lock()
	lnerr := srv.closeListenersLocked()
	srv.closeDoneChanLocked()
	for _, f := range srv.onShutdown {
		go f()
	}
	srv.mu.Unlock()

	ticker := time.NewTicker(shutdownPollInterval)
	defer ticker.Stop()
	for {
		if srv.closeIdleConns() {
			return lnerr
		}
		select {
		case <-ctx.Done():
			return ctx.Err()
		case <-ticker.C:
		}
	}
}
```

go在关闭http服务的流程如下：
- 先关闭所有的打开的listener
- 然后关闭空闲的connections
- 最后无限期等待连接转为空闲后再做关闭
- 也就是说，如果在此期间有在处理的连接是不会关闭的

对比直接close的方法：

```go
// Close immediately closes all active net.Listeners and any
// connections in state StateNew, StateActive, or StateIdle. For a
// graceful shutdown, use Shutdown.
//
// Close does not attempt to close (and does not even know about)
// any hijacked connections, such as WebSockets.
//
// Close returns any error returned from closing the Server's
// underlying Listener(s).
func (srv *Server) Close() error {
	atomic.StoreInt32(&srv.inShutdown, 1)
	srv.mu.Lock()
	defer srv.mu.Unlock()
	srv.closeDoneChanLocked()
	err := srv.closeListenersLocked()
	for c := range srv.activeConn {
		c.rwc.Close()
		delete(srv.activeConn, c)
	}
	return err
}
```

可以发现差异主要在于Close是直接遍历activeConn然后直接Close掉，而Shutdown则是设定定时器去关闭空闲链接


closeIdleConns遍历所有连接，尝试关闭，如果所有的连接都关闭了则返回true
```go
// closeIdleConns closes all idle connections and reports whether the
// server is quiescent.
func (s *Server) closeIdleConns() bool {
	s.mu.Lock()
	defer s.mu.Unlock()
	quiescent := true
	for c := range s.activeConn {
		st, unixSec := c.getState()
		// Issue 22682: treat StateNew connections as if
		// they're idle if we haven't read the first request's
		// header in over 5 seconds.
		if st == StateNew && unixSec < time.Now().Unix()-5 {
			st = StateIdle
		}
		if st != StateIdle || unixSec == 0 {
			// Assume unixSec == 0 means it's a very new
			// connection, without state set yet.
			quiescent = false
			continue
		}
		c.rwc.Close()
		delete(s.activeConn, c)
	}
	return quiescent
}
```

此处涉及到ConnState的概念

```go
var stateName = map[ConnState]string{
	StateNew:      "new",
	StateActive:   "active",
	StateIdle:     "idle",
	StateHijacked: "hijacked",
	StateClosed:   "closed",
}
```

```go
const (
	// StateNew represents a new connection that is expected to
	// send a request immediately. Connections begin at this
	// state and then transition to either StateActive or
	// StateClosed.
	StateNew ConnState = iota

	// StateActive represents a connection that has read 1 or more
	// bytes of a request. The Server.ConnState hook for
	// StateActive fires before the request has entered a handler
	// and doesn't fire again until the request has been
	// handled. After the request is handled, the state
	// transitions to StateClosed, StateHijacked, or StateIdle.
	// For HTTP/2, StateActive fires on the transition from zero
	// to one active request, and only transitions away once all
	// active requests are complete. That means that ConnState
	// cannot be used to do per-request work; ConnState only notes
	// the overall state of the connection.
	StateActive

	// StateIdle represents a connection that has finished
	// handling a request and is in the keep-alive state, waiting
	// for a new request. Connections transition from StateIdle
	// to either StateActive or StateClosed.
	StateIdle

	// StateHijacked represents a hijacked connection.
	// This is a terminal state. It does not transition to StateClosed.
	StateHijacked

	// StateClosed represents a closed connection.
	// This is a terminal state. Hijacked connections do not
	// transition to StateClosed.
	StateClosed
)
```

StateHijacked：被劫持，例如websocket
