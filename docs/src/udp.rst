
.. _udp:

:c:type:`uv_udp_t` --- UDP句柄
=================================

UDP句柄封装了对客户端和服务端的UDP通信。


数据类型
----------

.. c:type:: uv_udp_t

    UDP句柄类型。

.. c:type:: uv_udp_send_t

    UDP发送请求类型。

.. c:type:: uv_udp_flags

    用在 :c:func:`uv_udp_bind` 和 :c:type:`uv_udp_recv_cb` 的标志。

    ::

        enum uv_udp_flags {
            /* 禁用双栈模式。 */
            UV_UDP_IPV6ONLY = 1,
            /*
            * 指明消息被截断由于读缓冲区太小。剩余部分被操作系统丢弃。
            * 用于 uv_udp_recv_cb 。
            */
            UV_UDP_PARTIAL = 2,
            /*
            * 指明当以uv_udp_bind绑定句柄时是否 SO_REUSEADDR 将被设置。
            * 在BSDs和OS X上这设置SO_REUSEPORT套接字标志。
            * 在其他Unix平台上，它设置SO_REUSEADDR标志。
            * 这意味着多个线程或进程能够无错误地绑定到同一个地址（假如它们都设置了此标志）
            * 但是只有最后一个绑定的将会接收到任何流量，以从之前侦听器“偷走”端口的效果。
            */
            UV_UDP_REUSEADDR = 4
        };

.. c:type:: void (*uv_udp_send_cb)(uv_udp_send_t* req, int status)

    传递给 :c:func:`uv_udp_send` 的回调函数的类型定义，
    在数据发送之后被调用。

.. c:type:: void (*uv_udp_recv_cb)(uv_udp_t* handle, ssize_t nread, const uv_buf_t* buf, const struct sockaddr* addr, unsigned flags)

    传递给 :c:func:`uv_udp_recv_start` 的回调函数的类型定义，
    在终点接收了数据时被调用。

    * `handle` ： UDP句柄。
    * `nread` ： 已接收字节数。
      0 如果没有更多数据可读。 你可以丢弃读缓冲区或赋予其新的用途。
      注意 0 也意味着收到了空的数据报（在这种情况下 `addr` 是非空的）。
      < 0 如果检测到了一个传输错误。
    * `buf` ： :c:type:`uv_buf_t` 带有接收到的数据。
    * `addr` ： ``struct sockaddr*`` 包含发送者地址。
      可能为 NULL。 只在回调函数的期间内有效。
    * `flags`: 一个或更多被或的UV_UDP_* 常量。当前仅
      ``UV_UDP_PARTIAL`` 被使用。

    .. note::
        接收回调函数将被以 `nread` == 0 和 `addr` == NULL 调用当没有数据可读，
        且以 `nread` == 0 和 `addr` != NULL 当收到了一个空的UDP数据报。

.. c:type:: uv_membership

    对多播地址的成员类型。

    ::

        typedef enum {
            UV_LEAVE_GROUP = 0,
            UV_JOIN_GROUP
        } uv_membership;


公共成员
^^^^^^^^^^^^^^

.. c:member:: size_t uv_udp_t.send_queue_size

    排队发送的字节数目。
    这个字段严格显示当前排队的消息量。

.. c:member:: size_t uv_udp_t.send_queue_count

    当前排队等待处理的发送请求的数目。

.. c:member:: uv_udp_t* uv_udp_send_t.handle

    此发送请求所发生在的UDP句柄。

.. seealso:: :c:type:`uv_handle_t` 的成员也适用。


API
---

.. c:function:: int uv_udp_init(uv_loop_t* loop, uv_udp_t* handle)

    初始化一个新的UDP句柄。 实际的套接字是惰性创建的。
    返回 0 当成功时。

.. c:function:: int uv_udp_init_ex(uv_loop_t* loop, uv_udp_t* handle, unsigned int flags)

    以指定的标志初始化句柄。
    在此刻只有 `flags` 参数的低8位用于套接字域。一个套接字将为给定的域创建。
    如果指定的域是 ``AF_UNSPEC`` 则没有套接字被创建，就像 :c:func:`uv_udp_init` 一样。

    .. versionadded:: 1.7.0

.. c:function:: int uv_udp_open(uv_udp_t* handle, uv_os_sock_t sock)

    打开一个已存在的文件描述符或者Windows套接字作为一个UDP句柄。

    仅对Unix：
    `sock` 参数的唯一需求是遵循数据报合约（工作在无连接模式、支持sendmsg()/recvmsg()、等等）。
    换句话说，其他数据报类型的套接字像原始套接字或者Netlink套接字也能被传递给这个函数。

    .. versionchanged:: 1.2.1 文件描述符设为非阻塞模式。

    .. note::
        被传递的文件描述符或套接字没有类型检查，但是需要它代表一个合法的数据报套接字。

.. c:function:: int uv_udp_bind(uv_udp_t* handle, const struct sockaddr* addr, unsigned int flags)

    绑定UDP句柄到一个IP地址和端口。

    :param handle: UDP句柄。 应该以 :c:func:`uv_udp_init` 被初始化。

    :param addr: 带地址和端口绑定的 `struct sockaddr_in` 或 `struct sockaddr_in6` 。

    :param flags: 指明套接字将被如何绑定，
        ``UV_UDP_IPV6ONLY`` 和 ``UV_UDP_REUSEADDR`` 被支持。

    :returns: 0 若成功，或一个 < 0 的错误代码若失败。

.. c:function:: int uv_udp_getsockname(const uv_udp_t* handle, struct sockaddr* name, int* namelen)

    获取UDP句柄的本地的IP和端口。

    :param handle: UDP句柄。 应该以 :c:func:`uv_udp_init` 被初始化并且被绑定。

    :param name: 被地址数据填充的结构体的指针。
        为了支持IPv4和IPv6 `struct sockaddr_storage` 应被使用。

    :param namelen: 在输入上它指明 `name` 字段的数据。 在输出上它指明它被填充了多少。

    :returns: 0 若成功，或一个 < 0 的错误代码若失败。

.. c:function:: int uv_udp_set_membership(uv_udp_t* handle, const char* multicast_addr, const char* interface_addr, uv_membership membership)

    对一个多播地址设置成员。

    :param handle: UDP句柄。 应该以 :c:func:`uv_udp_init` 被初始化。

    :param multicast_addr: 设置成员的多播地址。

    :param interface_addr: 接口地址。

    :param membership: 应该为 ``UV_JOIN_GROUP`` 或 ``UV_LEAVE_GROUP`` 。

    :returns: 0 若成功，或一个 < 0 的错误代码若失败。

.. c:function:: int uv_udp_set_multicast_loop(uv_udp_t* handle, int on)

    设置IP多播循环标志。 使得多播包循环回本地套接字。

    :param handle: UDP句柄。 应该以 :c:func:`uv_udp_init` 被初始化。 

    :param on: 1 为开，0 为关。

    :returns: 0 若成功，或一个 < 0 的错误代码若失败。

.. c:function:: int uv_udp_set_multicast_ttl(uv_udp_t* handle, int ttl)

    设置多播TTL。

    :param handle: UDP句柄。 应该以 :c:func:`uv_udp_init` 被初始化。 

    :param ttl: 1 到 255 。

    :returns: 0 若成功，或一个 < 0 的错误代码若失败。

.. c:function:: int uv_udp_set_multicast_interface(uv_udp_t* handle, const char* interface_addr)

    设置发送和接收数据所在的多播接口。

    :param handle: UDP句柄。 应该以 :c:func:`uv_udp_init` 被初始化。 

    :param interface_addr: 接口地址。

    :returns: 0 若成功，或一个 < 0 的错误代码若失败。

.. c:function:: int uv_udp_set_broadcast(uv_udp_t* handle, int on)

    设置多播开关。

    :param handle: UDP句柄。 应该以 :c:func:`uv_udp_init` 被初始化。 

    :param on: 1 为开，0 为关。

    :returns: 0 若成功，或一个 < 0 的错误代码若失败。

.. c:function:: int uv_udp_set_ttl(uv_udp_t* handle, int ttl)

    设置生存时间（TTL）。

    :param handle: UDP句柄。 应该以 :c:func:`uv_udp_init` 被初始化。 

    :param ttl: 1 到 255 。

    :returns: 0 若成功，或一个 < 0 的错误代码若失败。

.. c:function:: int uv_udp_send(uv_udp_send_t* req, uv_udp_t* handle, const uv_buf_t bufs[], unsigned int nbufs, const struct sockaddr* addr, uv_udp_send_cb send_cb)

    通过UDP套接字发送数据。 如果套接字之前没有用
    :c:func:`uv_udp_bind` 绑定，它将绑定到 0.0.0.0
    （IPv4地址“所有接口”）和一个随机的端口号。

    在Windows上如果 `addr` 被初始化指向一个未指定的地址
    （ ``0.0.0.0`` 或者 ``::`` ），它将被改变以指向 ``localhost`` 。
    这么做是为了符合Linux系统的行为。

    :param req: UDP请求句柄。 不需要初始化。

    :param handle: UDP句柄。 应该以 :c:func:`uv_udp_init` 被初始化。 

    :param bufs: 发送缓冲区的列表。

    :param nbufs: `bufs` 中的缓冲区个数。

    :param addr: 带远端地址和端口的 `struct sockaddr_in` 或 `struct sockaddr_in6` 。

    :param send_cb: 当数据已发出时调用的回调函数。

    :returns: 0 若成功，或一个 < 0 的错误代码若失败。

    .. versionchanged:: 1.19.0 新增 ``0.0.0.0`` 和 ``::`` 到 ``localhost`` 的映射

.. c:function:: int uv_udp_try_send(uv_udp_t* handle, const uv_buf_t bufs[], unsigned int nbufs, const struct sockaddr* addr)

    与 :c:func:`uv_udp_send` 相同，但是如果无法立刻完成不会排队一个发送请求。

    :returns: >= 0 ： 已发送的字节数（匹配给定缓冲区的大小）。
        < 0 ： 负的错误代码（返回 ``UV_EAGAIN`` 当无法立刻发送消息时）。

.. c:function:: int uv_udp_recv_start(uv_udp_t* handle, uv_alloc_cb alloc_cb, uv_udp_recv_cb recv_cb)

    准备接受数据。 如果套接字之前没有用
    :c:func:`uv_udp_bind` 绑定，它将绑定到 0.0.0.0
    （IPv4地址“所有接口”）和一个随机的端口号。

    :param handle: UDP句柄。 应该以 :c:func:`uv_udp_init` 被初始化。 

    :param alloc_cb: 当需要临时存储时调用的回调函数。

    :param recv_cb: 接收数据调用的回调函数。

    :returns: 0 若成功，或一个 < 0 的错误代码若失败。

.. c:function:: int uv_udp_recv_stop(uv_udp_t* handle)

    停止侦听新来的数据报。

    :param handle: UDP句柄。 应该以 :c:func:`uv_udp_init` 被初始化。

    :returns: 0 若成功，或一个 < 0 的错误代码若失败。

.. c:function:: size_t uv_udp_get_send_queue_size(const uv_udp_t* handle)

    返回 `handle->send_queue_size` 。

    .. versionadded:: 1.19.0

.. c:function:: size_t uv_udp_get_send_queue_count(const uv_udp_t* handle)

    返回 `handle->send_queue_count` 。

    .. versionadded:: 1.19.0

.. seealso:: :c:type:`uv_handle_t` 的API函数也适用。
