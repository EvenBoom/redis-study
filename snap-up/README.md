# 秒杀功能
## FAQ
Q:为什么redis能实现秒杀功能?   
A:因为redis是单进程单线程（epoll）   
    
Q:redis能否使用GET,SET来实现秒杀功能?		
A:不能！高并发下会出现超买超卖现象，所以使用redis的队列实现   
    
Q:如何进行测试?		
A:可以使用jmeter测试工具模拟高并发测试   
```
package main

import (
	"fmt"
	"net/http"

	"github.com/gomodule/redigo/redis"
)

func main() {
	http.HandleFunc("/buy", buy)
	http.ListenAndServe(":80", nil)
}

//需提前使用lpush命令将数据插入队列，数据可以是1，也可以是订单号（如果是订单号请把reply改回string）
func buy(w http.ResponseWriter, r *http.Request) {
	conn, _ := redis.Dial("tcp", "127.0.0.1:6379")//连接redis
	reply, err := conn.Do("rpop", "iphone")//执行出列命令
	isPop, _ := redis.Int(reply, err)
	if isPop > 0 {//如果队列没有数据，出列的是0
		w.Write([]byte("make it!"))
	} else {
		w.Write([]byte("failed!"))
	}

}

```
