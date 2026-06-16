# D-Bus 学习笔记 01


## 基础概念

- D-Bus 是一个消息总线系统，其功能已涵盖进程间通信的所有需求，并具备一些特殊的用途
- D-Bus 是三层架构的进程间通信系统：接口层 + 总线层 + 包装层
  - **接口层**：由函数库 `libdbus` 提供，进程可通过该库使用 D-Bus 的能力
  - **总线层**：由 D-Bus 总线守护进程提供，在 Linux 系统启动时运行，负责进程间的消息路由和传递，其中包括 Linux 内核和 Linux 桌面环境的消息传递
  - **包装层**：一系列基于特定应用程序框架的 Wrapper 库
- D-Bus 具备自身的协议，协议基于二进制数据设计，与数据结构和编码方式无关
  - `libdbus` / D-Bus 总线守护进程均不需要太大的系统开销

### 总线类型

总线是 D-Bus 的进程间通信机制，一个系统中通常存在多条总线，这些总线由 D-Bus 总线守护进程管理：

- **系统总线（System Bus）**：只有 Linux 内核、Linux 桌面环境和权限较高的程序才能向该总线写入消息，以此保障系统安全性，防止有恶意进程假冒 Linux 发送消息
- **会话总线（Session Buses）**：由普通进程创建，可同时存在多条。会话总线属于某个进程私有，它用于进程间传递消息

### 重要概念

- **对象**：对象是封装后的匹配器与回调函数，它以对等（peer-to-peer）协议使每个消息都有一个源地址和一个目的地址。这些地址又称为对象路径，或者称之为总线名称。对象的接口是回调函数，它以类似 C++ 的虚拟函数实现。当一个进程注册到某个总线时，都要创建相应的消息对象
- **消息**：D-Bus 的消息分为信号（signals）、方法调用（method calls）、方法返回（method returns）和错误（errors）
  - **信号**：最基本的消息，注册的进程可简单地发送信号到总线上，其他进程通过总线读取消息
  - **方法调用**：通过总线传递参数，执行另一个进程接口函数的机制，用于某个进程控制另一个进程
  - **方法返回**：注册的进程在收到相关信息后，自动做出反应的机制，由回调函数实现
  - **错误**：信号的一种，是注册进程错误处理机制之一
- **服务**：服务（Services）是进程注册的抽象。进程注册某个地址后，即可获得对应总线的服务。D-Bus 提供了服务查询接口，进程可通过该接口查询某个服务是否存在，或者在服务结束时自动收到来自系统的消息

## 典型流程

### 建立服务

1. 建立 dbus 连接（`dbus_bus_get()`）
2. 为这个 dbus 连接（`DbusConnection`）起名（`dbus_bus_request_name()`），此名即为后续远程调用的服务名
3. 进入监听循环（`dbus_connection_read_write()`）
4. 循环中，从总线中取出信息（`dbus_connection_pop_message()`）
5. 通过比对信息中的方法接口名和方法名（`dbus_message_is_method_call()`），如果一致，跳转到相应的处理中，从消息中取出远程调用的参数，并建立回传[^1]结果的通路（`reply_to_method_call()`）

[^1]: 回传动作本身等同于一次不需要等待结果的远程调用。

### 发送信号

1. 建立 dbus 连接，并为这个连接起名，建立发送信号的通道
2. 在建立通道的函数中，需要填写该信号的接口名及信号名（`dbus_message_new_signal()`）
3. 将相关的参数压到对应的信号中去（`dbus_message_iter_init_append()`、`dbus_message_iter_append_basic()`）
4. 启动发送（`dbus_connection_send()`、`dbus_connection_flush()`）

### 进行远程调用

1. 建立 dbus 连接，并为 dbus 连接命名
2. 申请一个远程调用通道（`dbus_message_new_method_call()`），注意填写服务器名，及本次调用的接口名 + 方法名，压入本次调用的参数（`dbus_message_iter_init_append()`、`dbus_message_iter_append_basic()`）
3. 把真正要传的参数写入消息结构（送完之后一般都会判断是否越界）
4. 启动发送调用并释放发送相关的消息结构（`dbus_connection_send_with_reply()`），启动函数中带有一个句柄，之后阻塞等待句柄带回总线上回传的信息，当句柄回传信息之后，从信息结构中分离出参数（`dbus_message_iter_init()`、`dbus_message_iter_next()`、`dbus_message_iter_get_arg_type()`、`dbus_message_iter_get_basic()`）

### 信号接收

1. 建立 dbus 连接并为 dbus 连接起名，为要进行的消息循环添加匹配条件（通过信号名和信号接口名来进行匹配控制）（`dbus_bus_add_match()`）。进入等待循环后，只需要对信号名、信号接口名进行判断就可以分别处理各种信号了

## 常用 API 参考

| 函数 | 说明 |
|------|------|
| `dbus_connection_read_write()` | 在连接打开时阻塞，直到可以读或写，然后执行读写操作 |
| `dbus_connection_pop_message()` | 从入站消息队列中取出第一条消息并移除；队列为空时返回 `NULL` |
| `dbus_connection_send()` | 将消息加入出站队列，异步写入网络；要强制写出可调用 `dbus_connection_flush()` |
| `dbus_connection_send_with_reply()` | 发送消息并返回 `DBusPendingCall`，用于接收回复；超时则生成合成错误回复 |
| `dbus_message_is_signal()` | 检查消息是否为具有给定 interface 和 member 字段的信号 |
| `dbus_message_iter_init()` | 初始化 `DBusMessageIter` 以读取消息参数 |
| `dbus_message_iter_next()` | 移动迭代器到下一个字段；无下一个字段时返回 `FALSE` |
| `dbus_message_iter_get_arg_type()` | 返回迭代器指向的参数类型；迭代器在消息末尾时返回 `DBUS_TYPE_INVALID` |
| `dbus_message_iter_get_basic()` | 从消息迭代器读取基本类型值（整数、字符串等非容器类型） |
| `dbus_message_new_signal()` | 构造表示信号发射的新消息 |
| `dbus_message_iter_init_append()` | 初始化 `DBusMessageIter` 以向消息末尾追加参数 |
| `dbus_message_iter_append_basic()` | 向消息追加基本类型值 |
| `dbus_message_new_method_call()` | 构造调用远程对象方法的新消息 |
| `dbus_bus_get()` | 连接到总线守护进程并注册客户端；若连接已存在则返回已有连接 |
| `dbus_bus_request_name()` | 请求总线将给定名称分配给此连接 |
| `dbus_bus_add_match()` | 添加匹配规则以匹配经过消息总线的消息 |
| `dbus_pending_call_block()` | 阻塞直到 pending call 完成 |
| `dbus_pending_call_steal_reply()` | 获取回复消息；每条 pending call 只能调用一次 |

### 命名规则

给 `DBusConnection` 起名字（命名）时，两个相互通信的连接（connection）不能同名。

命名规则：`xxx.xxx`（例如 `zeng.xiaolong`）


---

> 作者: [Charles Y](https://github.com/devCharl/devcharl.github.io)  
> URL: https://devcharl.github.io/develop/dbus-learning-notes-01/  

