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

## lsof

lsof分析

> https://zhuanlan.zhihu.com/p/36672944

lsof（list open files）是一个查看当前系统文件的工具。在linux环境下，任何事物都以文件的形式存在，用户通过文件不仅可以访问常规数据，还可以访问网络连接和硬件；如传输控制协议 (TCP) 和用户数据报协议 (UDP)套接字等，系统在后台都为该应用程序分配了一个文件描述符，该文件描述符提供了大量关于此应用程序的信息。

### 参数

-a：列出打开文件存在的进程；
-c<进程名>：列出指定进程所打开的文件；
-g：列出GID号进程详情；
-d<文件号>：列出占用该文件号的进程；
+d<目录>：列出目录下被打开的文件；
+D<目录>：递归列出目录下被打开的文件；
-n<目录>：列出使用NFS的文件；
-i<条件>：列出符合条件的进程(4、6、协议、:端口、 @ip )；
-p<进程号>：列出指定进程号所打开的文件；
-u：列出UID号进程详情；

### 表头含义

COMMAND：进程的名称；
PID：进程标识符；
PPID：父进程标识符(需要指定-R参数)；
USER：进程所有者；
PGID：进程所属组；
FD：文件描述符，应用程序通过文件描述符识别该文件。

### 文件描述符

#### FD类型：

①. cwd：表示current work dirctory，即：应用程序的当前工作目录，这是该应用程序启动的目录，除非它本身对这个目录进行更改；
②. txt：该类型的文件是程序代码，如应用程序二进制文件本身或共享库，如上列表中显示的 /sbin/init 程序；
③. lnn：library references (AIX)；
④. er：FD information error (see NAME column)；
⑤. jld：jail directory (FreeBSD)；
⑥. ltx：shared library text (code and data)；
⑦. mxx ：hex memory-mapped type number xx.
⑧. m86：DOS Merge mapped file；
⑨. mem：memory-mapped file；
⑩. mmap：memory-mapped device；
⑪. pd：parent directory；
⑫. rtd：root directory；
⑬. tr：kernel trace file (OpenBSD)；
⑭. v86 VP/ix mapped file；
⑮. 0：表示标准输出；
⑯. 1：表示标准输入；
⑰. 2：表示标准错误。

#### 文件状态模式：

①.u：表示该文件被打开并处于读取/写入模式；
②.r：表示该文件被打开并处于只读模式；
③.w：表示该文件被打开并处于只写模式；
④.空格：表示该文件的状态模式为unknow，且没有锁定；
⑤.-：表示该文件的状态模式为unknow，且被锁定。

#### 文件锁：

①. N：for a Solaris NFS lock of unknown type；
②. r：for read lock on part of the file；
③. R：for a read lock on the entire file；
④. w：for a write lock on part of the file；(文件的部分写锁)
⑤. W：for a write lock on the entire file；(整个文件的写锁)
⑥. u：for a read and write lock of any length；
⑦. U：for a lock of unknown type；
⑧. x：for an SCO OpenServer Xenix lock on part of the file；
⑨. X：for an SCO OpenServer Xenix lock on the entire file；
⑩. space：if there is no lock。

### 文件类型

①. DIR：表示目录；
②. CHR：表示字符类型；
③. BLK：块设备类型；
④. UNIX：UNIX 域套接字；
⑤. FIFO：先进先出 (FIFO) 队列；
⑥. IPv4：网际协议 (IP) 套接字；
⑦. REG: 表示文件

### DEVICE：指定磁盘的名称；
### SIZE：文件的大小；
### NODE：索引节点(文件在磁盘上的标识)；
### NAME：打开文件的确切名称。

### 例子

以hotStart项目为例：

```
lsof -c demo.bin
COMMAND   PID     USER   FD     TYPE             DEVICE SIZE/OFF     NODE NAME
demo.bin 4129 zhangpei  cwd      DIR                1,4      320 31521754 /Users/zhangpei/mine/hotstart
demo.bin 4129 zhangpei  txt      REG                1,4  7430060 32081277 /Users/zhangpei/mine/hotstart/demo.bin
demo.bin 4129 zhangpei  txt      REG                1,4   973872 22900055 /usr/lib/dyld
demo.bin 4129 zhangpei    0u     CHR               16,3  0t56758      641 /dev/ttys003
demo.bin 4129 zhangpei    1u     CHR               16,3  0t56758      641 /dev/ttys003
demo.bin 4129 zhangpei    2u     CHR               16,3  0t56758      641 /dev/ttys003
demo.bin 4129 zhangpei    3     PIPE 0x2ba97131f4331a01    16384          ->0x2ba97131f4331d01
demo.bin 4129 zhangpei    4     PIPE 0x2ba97131f4331d01    16384          ->0x2ba97131f4331a01
demo.bin 4129 zhangpei    5u    IPv6 0x2ba97131f7042581      0t0      TCP *:distinct (LISTEN)
demo.bin 4129 zhangpei    6u  KQUEUE                                      count=0, state=0xa
```

