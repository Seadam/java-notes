# Java Socket

#### 内容摘要

概述，网络模型，三要素，IP 地址，端口号，传输协议，Socket，InetAddress，UDP传输

---

#### 概述

* 资源共享和信息传递
* 不同程序间的信息交换

#### 网络模型

* 协议栈


* OSI （*Open System Interconnection*）参考模型
  * 应用层
    * 应用程序
  * 表示层
    * 加密解密，压缩解压缩
  * 会话层
    * 管理不同的会话，同一个程序不同的窗口
  * 传输层
    * 端口号：标识计算机中的进行通信的进程
  * 网络层
    * IP：从网络层面标识一台计算机
    * IP 地址的封装与解封装
  * 数据链路层
    * MAC 地址：唯一标识一台计算机的标志
  * 物理层
    * 流传输
* TCP/IP 参考模型
  * 应用层
    * OSI ：应用层，表示层，回话层
  * 传输层
    * OSI：传输层
    * 通信协议：TCP、UDP
  * 网际层
    * OSI：网络层
  * 接入层
    * OSI：数据链路层，物理层

#### 三要素

* IP 地址：网路中设备标识
* 端口号：标识进程的逻辑地址，不同进程的标识
* 通信协议：TCP、UDP

#### IP 地址

* 每个连接上互联网的主机分配 32 bit 地址 4 个字节，`00001010 00000000 00000000 00000001`
* 为了方便实用，点分十进制来表示，用 `.` 分隔，`10.0.0.1`
* IP 地址 = 网络号码 + 主机地址
* A类 ：第一个字节表示网络编码，后三个字节表示主机编号   2^24^（数量级）
  * `xxx.0.0.0`   表示网络本身，网络地址
  * `xxx.255.255.255`   表示该A类网络中所有主机，广播地址
  * 范围：`1.0.0.1 ~~ 127.255.255.254`
  * 私有地址：`10.x.x.x`
* B类：前两个字节表示网络编号，后两个字节表示主机编号    2^16^（数量级）
  * `xxx.xxx.0.0`
  * `xxx.xxx.255.255`
  * 范围：`128.0.0.1 ~~ 191.255.255.254`
  * 私有地址：`172.16.0.0 ~~ 172.31.255.255`
* C类：前三个字节表示网络编号，后一个字节表示主机编号    2^8^  （数量级）
  * `xxx.xxx.xxx.0`
  * `xxx.xxx.xxx.255`
  * 范围：`192.0.0.1 ~~ 223.255.255.254`
  * 私有地址：`192.168.x.x`
* 每一种类型的网络 IP 地址中，不能用的 ：
  * `0 网络地址` 
  *  `255 广播地址`  对该地址发送数据包，网络中所有主机都会收到
* 特殊地址
  * `127.0.0.1` 回环测试地址，用于测试本机网络，不会出现在公网中
* 子网掩码 *subnet mask*
  * 用来标识到底是几类地址
  * 原理：掩码与 IP 按位与
  * A 类 `225.0.0.0`
  * B 类 `255.255.0.0`
  * C 类 `255.255.255.0`
* 私有地址：公网中不能使用，用在局域网中
* 保留地址
  * `169.254.x.x`
  * D 类 `224.0.0.1 ~~ 239.255.255.254`
  * E 类 `240.0.0.1 ~~ 247.255.254`

#### 端口号

* 物理端口 网卡口（MAC地址）
* 逻辑端口：标识进程的逻辑地址
* 有效端口：`0 ~ 65535`
* 系统使用或保留：`0 ~ 1024`
* HTTP 协议使用 `80` 端口，`HTTPS` 协议使用 `443` 端口

#### 传输协议

* UDP：每个数据报大小限制为 64k，不需要建立连接，不可靠，速度快
* TCP：建立连接，传输数据的通道，三次握手，可靠，效率低

#### Socket

* 套接字 ` Socket = ip地址 + 端口号`
* 屏蔽了协议的具体细节
* 通信的两端都有 Socket
* 网络通信就是 Socket 间通信
* 逻辑上的通信信道，底层**用流来传输**

#### InetAddress

* 操作 IP 地址的类，没有构造方法
* `INetAddress getByName(String host)`
  * 传入主机名或者 IP地址 字符串，返回 IP地址对象
  * 传入 null 则返回 localhost
* `String get HostName()`
  * 返回主机名
* `String getHostAddress()`
  * 返回 IP地址，字符串

#### UDP 

* 数据源地址和目的地址 以及 数据 封装


* 只管发送数据，不管接收方是否接收


* `DatagramSocket`  指定发送端与接受端
  * 一个端口同时只能被一个 Socket 绑定，不然会异常 `AddressAlreadyUseException`
  * `send(DatagramPacket packet)`
  * `receive(DatagramPacket packet)`
    * 阻塞方法，等着接收
* `DatagramPacket `  数据包
  * `getData()`
  * `getLength()`


* 发送端与接受端是独立运行的进程

* 流程

  * 发送
    1. 启动 socket 服务，并监听端口，处理 `SocketException`
    2. 创建数据包，指定目的 data, ip, port，处理 `UnknownHostException`
    3. 发送，处理 `IOException`
    4. 关闭 Socket

  ```Java
  public class UdpSendTest {
      public static void main(String[] args) throws IOException, SocketException {
          // 1. 启动 socket 服务，并监听端口，此处处理 SocketException
          DatagramSocket sender = new DatagramSocket(8888);
          // 2. 创建数据包，指定目的 data, ip, port
          // data
          String message = "Hello receiver.";
          byte[] data = message.getBytes();
          // ip 此处处理 UnknownHostException
          InetAddress ip = InetAddress.getByName("192.168.3.42");
          // port
          int port = 9999;
          DatagramPacket packet = new DatagramPacket(data, data.length, ip, port);
          // 3. 发送，此处处理 IOException
          sender.send(packet);
          // 4. 关闭
          sender.close();
      }
  }
  ```

  * 接收
    1. 启动 Socket 服务，并监听发送方端口，此处处理 `SocketException`
    2. 创建数据包，用来接收数据
    3. 接收，阻塞方法，此处处理 `IOException`
    4. 解析数据包
    5. 关闭 Socket

  ```java
  public class UdpReceiveTest {
      public static void main(String[] args) throws SocketException, IOException {
          // 1. 启动 Socket 服务，并监听发送方端口，此处处理 SocketException
          DatagramSocket receiver = new DatagramSocket(9999);
          // 2. 创建数据包，用来接收数据
          byte[] data = new byte[1024];
          DatagramPacket packet = new DatagramPacket(data, data.length);
          // 3. 接收，阻塞方法，此处处理 IOException
          receiver.receive(packet);
          // 4. 解析数据包
          byte[] reveiveData = packet.getData();
          int length = packet.getLength();
          String result = new String(reveiveData, 0, length);
          System.out.println("Received: " + result);
          // 5. 关闭
          receiver.close();
      }
  }
  ```

