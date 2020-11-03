## channel关闭

> Always close a channel on the producer side

永远在生产方关闭一个channel

参考：
> 如何优雅地关闭 Go 中的工作 goroutine https://ictar.xyz/2019/10/19/trans-go-worker-cancellation/
> Go并发编程模型：主动停止goroutine https://zhuanlan.zhihu.com/p/66659719

## 如何解包

```go
//处理客户端请求
go func(conn net.Conn) {
    //创建封包拆包对象dp
    dp := znet.NewDataPack()
    for {
        //1 先读出流中的head部分
        headData := make([]byte, dp.GetHeadLen())
        _, err := io.ReadFull(conn, headData) //ReadFull 会把msg填充满为止
        if err != nil {
            fmt.Println("read head error")
            break
        }
        //将headData字节流 拆包到msg中
        msgHead, err := dp.Unpack(headData)
        if err != nil {
            fmt.Println("server unpack err:", err)
            return
        }

        if msgHead.GetDataLen() > 0 {
            //msg 是有data数据的，需要再次读取data数据
            msg := msgHead.(*znet.Message)
            msg.Data = make([]byte, msg.GetDataLen())

            //根据dataLen从io中读取字节流
            _, err := io.ReadFull(conn, msg.Data)
            if err != nil {
                fmt.Println("server unpack data err:", err)
                return
            }

            fmt.Println("==> Recv Msg: ID=", msg.Id, ", len=", msg.DataLen, ", data=", string(msg.Data))
        }
    }
}(conn)
```