# socket

## 示例
```python
import socket
import threading
import logging

FORMAT = "[%(threadName)s] --- %(message)s"
logging.basicConfig(format=FORMAT, level=logging.INFO)


class ChatServer:
    def __init__(self, ip='127.0.0.1', port=9999):
        self.sock = socket.socket()
        self.addr = (ip, port)
        self.clients = []

    def start(self):
        self.sock.bind(self.addr)
        self.sock.listen()

        threading.Thread(target=self.accept, name='accept').start()

    def accept(self):        
        n = 0
        while True:
            n += 1
            s, _ = self.sock.accept()
            self.clients.append(s)
            logging.info('from: {}'.format(s))
            threading.Thread(target=self.revc, name='recv-{}'.format(n), args=(s,)).start()

    def revc(self, sock):
        while True:
            data = sock.recv(1024)
            logging.info('data: {}, from: {}'.format(data.decode(), sock))
            msg = 'Your msg from {} = {}'.format(sock.getpeername(), data.decode())
            for s in self.clients:
                s.send(msg.encode())

    def stop(self):
        self.sock.close()


if __name__ == '__main__':
    cs = ChatServer()
    cs.start()
    logging.info("I FREE...")

```
## 方法

* `socket.socket()`:      创建 socket 对象
* `socket.bind(address)`: 将地址绑定在 socket 对象上
* `socket.listen()`:      开始监听在某个socket接口上
* `socket.accept()`: 接受请求，从listen的队列中拿请求
* `socket.recv(size)`: 接受数据
* `socket.send(data)`: 发送数据
* `sockekt.close()`: 关闭 socket 对象

# socketserver
socket 函数高级封装

## 示例：

```python
from socketserver import BaseServer, BaseRequestHandler, ThreadingTCPServer, TCPServer


# 继承 BaseRequestHandler 基类
class MyHandler(BaseRequestHandler):
    # 重写基类 handle 方法
    def handle(self):
        print('~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~')
        print(self.__dict__)
        print(self.request)
        print(self.client_address)
        print(self.server)
        print('~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~')

        # 接受用户数据
        while True:
            data = self.request.recv(1024)
            msg = 'Your msg: {}'.format(data.decode())
            self.request.send(msg.encode())

        print('bye ~~~~~~~~~~~~~~~~~~~')

# 多线程法式启动一个 TCPServer
server = ThreadingTCPServer(('127.0.0.1', 9999), MyHandler)
# 启动 TCPServer， 使其一直处于监听状态
server.serve_forever()

# 只处理一次请求
# server.handle_request()

# 关闭 TCPServer
server.shutdown()  # close
```


### socketserver 类继承关系

```
BasaeServer     <-      TCPServer   <-      UDPServer
```

```
            +------------+
            | BaseServer |
            +------------+
                   |
                   v
+-----------+           +------------------+
| TCPServer |  -------> | UnixStreamServer |
+-----------+           +------------------+
                   |
                   v
+-----------+           +--------------------+
| UDPServer |  -------> | UnixDatagramServer |
+-----------+           +--------------------+
```

#### 同步类
* TCPServer
* UDPServer
* UnixStreamServer
* UnixDatagramServer

#### 异步类
通过两个Minxin类来支持异步： ForkingMinIn 和 ThreadingMinIn


##### 多进程
* class ForkingUDPServer(ForkingMixIn, UDPServer): pass
* class ForkingTCPServer(ForkingMixIn, TCPServer): pass

##### 多线程
* class ThreadingUDPServer(ThreadingMixIn, UDPServer): pass
* class ThreadingTCPServer(ThreadingMixIn, TCPServer): pass





