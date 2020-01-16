# 常量

*   **SWOOLE_VERSION** 当前Swoole的版本号，字符串类型，如1.6.0

## 构造函数参数

*   **SWOOLE_BASE** 使用Base模式，业务代码在Reactor进程中直接执行
*   **SWOOLE_PROCESS** 使用进程模式，业务代码在Worker进程中执行

## Socket 类型

*   **SWOOLE_SOCK_TCP** 创建tcp socket
*   **SWOOLE_SOCK_TCP6** 创建tcp ipv6 socket
*   **SWOOLE_SOCK_UDP** 创建udp socket
*   **SWOOLE_SOCK_UDP6** 创建udp ipv6 socket
*   **SWOOLE_SOCK_UNIX_DGRAM** 创建unix dgram socket
*   **SWOOLE_SOCK_UNIX_STREAM** 创建unix stream socket
*   **SWOOLE_SOCK_SYNC** 同步客户端
*   **SWOOLE_SOCK_ASYNC** 异步客户端

## SSL 加密方法

*   SWOOLE_SSLv3_METHOD
*   SWOOLE_SSLv3_SERVER_METHOD
*   SWOOLE_SSLv3_CLIENT_METHOD
*   SWOOLE_SSLv23_METHOD（默认加密方法）
*   SWOOLE_SSLv23_SERVER_METHOD
*   SWOOLE_SSLv23_CLIENT_METHOD
*   SWOOLE_TLSv1_METHOD
*   SWOOLE_TLSv1_SERVER_METHOD
*   SWOOLE_TLSv1_CLIENT_METHOD
*   SWOOLE_TLSv1_1_METHOD
*   SWOOLE_TLSv1_1_SERVER_METHOD
*   SWOOLE_TLSv1_1_CLIENT_METHOD
*   SWOOLE_TLSv1_2_METHOD
*   SWOOLE_TLSv1_2_SERVER_METHOD
*   SWOOLE_TLSv1_2_CLIENT_METHOD
*   SWOOLE_DTLSv1_METHOD
*   SWOOLE_DTLSv1_SERVER_METHOD
*   SWOOLE_DTLSv1_CLIENT_METHOD