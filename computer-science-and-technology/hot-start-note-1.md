## 优雅重启

> https://github.com/peigongdh/hotstart

## BUG修复

> https://github.com/peigongdh/hotstart/commit/c0a5de73693a08fe573a991e66af2aa8a12c81ab

```go
func (srv *HotServer) getNetListener(addr string) (ln net.Listener, err error) {
	if srv.isChild {
		// TODO: name not effect
		file := os.NewFile(LISTENER_FD, "")
		defer func() {
			_ = file.Close()
		}()
		ln, err = net.FileListener(file)
		if err != nil {
			err = fmt.Errorf("net.FileListener error: %v", err)
			return nil, err
		}
	} else {
		ln, err = net.Listen("tcp", addr)
		if err != nil {
			err = fmt.Errorf("net.Listen error: %v", err)
			return nil, err
		}
	}
	return ln, nil
}

// NewFile returns a new File with the given file descriptor and
// name. The returned value will be nil if fd is not a valid file
// descriptor. On Unix systems, if the file descriptor is in
// non-blocking mode, NewFile will attempt to return a pollable File
// (one for which the SetDeadline methods work).
func NewFile(fd uintptr, name string) *File {
	// ...
}
```

- 修复了os.NewFile后没有Close导致子进程下有未关闭的fd

```
~ lsof -i:9999
COMMAND   PID     USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
demo.bin 3539 zhangpei    3u  IPv6 0x2ba97131f7043101      0t0  TCP *:distinct (LISTEN)
demo.bin 3539 zhangpei    6u  IPv6 0x2ba97131f7043101      0t0  TCP *:distinct (LISTEN)
```

- TODO: 在修复close前，设置NewFile的name为什么不生效？

同理，在fork时，如果失败，则应该关闭getTCPListenerFile返回的file

```go
func (srv *HotServer) fork() (err error) {
	listener, err := srv.getTCPListenerFile()
	if err != nil {
		return fmt.Errorf("failed to get socket file descriptor: %v", err)
	}

	// set hotstart restart env flag
	env := append(
		os.Environ(),
		"HOT_CONTINUE=1",
	)

	execSpec := &syscall.ProcAttr{
		Env:   env,
		Files: []uintptr{os.Stdin.Fd(), os.Stdout.Fd(), os.Stderr.Fd(), listener.Fd()},
	}

	srv.logf("HotServer do fork")
	_, err = syscall.ForkExec(os.Args[0], os.Args, execSpec)
	if err != nil {
		_ = listener.Close()
		return fmt.Errorf("Restart: Failed to launch, error: %v", err)
	}

	return
}

func (srv *HotServer) getTCPListenerFile() (*os.File, error) {
	file, err := srv.listener.(*net.TCPListener).File()
	if err != nil {
		return file, err
	}
	return file, nil
}

// File returns a copy of the underlying os.File.
// It is the caller's responsibility to close f when finished.
// Closing l does not affect f, and closing f does not affect l.
//
// The returned os.File's file descriptor is different from the
// connection's. Attempting to change properties of the original
// using this duplicate may or may not have the desired effect.
func (l *TCPListener) File() (f *os.File, err error) {
    // ...
}
```