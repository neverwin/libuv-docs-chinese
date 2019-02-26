
.. _tcp:

:c:type:`uv_tcp_t` --- TCP句柄
=================================

TCP句柄用于表示TCP流和服务器。

:c:type:`uv_tcp_t` 是 :c:type:`uv_stream_t` 的一个 '子类型' 。


数据类型
----------

.. c:type:: uv_tcp_t

    TCP句柄类型。


公共成员
^^^^^^^^^^^^^^

N/A

.. seealso:: :c:type:`uv_stream_t` 的成员也适用。


API
---

.. c:function:: int uv_tcp_init(uv_loop_t* loop, uv_tcp_t* handle)

    初始化句柄。 迄今为止没有创建套接字。

.. c:function:: int uv_tcp_init_ex(uv_loop_t* loop, uv_tcp_t* handle, unsigned int flags)

    以指定的标志来初始化句柄。
    在此刻只有 `flags` 参数的低8位用于套接字域。一个套接字将为给定的域创建。
    如果指定的域是 ``AF_UNSPEC`` 则没有套接字被创建，就像 :c:func:`uv_tcp_init` 一样。

    .. versionadded:: 1.7.0

.. c:function:: int uv_tcp_open(uv_tcp_t* handle, uv_os_sock_t sock)

    打开一个已存在的文件描述符或者套接字作为一个TCP句柄。

    .. versionchanged:: 1.2.1 文件描述符设为非阻塞模式。

    .. note::
        被传递的文件描述符或套接字没有类型检查，但是需要它代表一个合法的流套接字。

.. c:function:: int uv_tcp_nodelay(uv_tcp_t* handle, int enable)

    开启 `TCP_NODELAY`，禁用Nagle算法。

.. c:function:: int uv_tcp_keepalive(uv_tcp_t* handle, int enable, unsigned int delay)

    开启/禁用TCP keep-alive。 `delay` 是以秒计的起始延迟，当 `enable` 是零时被忽略。

.. c:function:: int uv_tcp_simultaneous_accepts(uv_tcp_t* handle, int enable)

    开启/禁用同时异步的接受请求被操作系统排队当侦听新来的TCP连接时。

    这个设置用来调整TCP服务器达到满意的性能。
    拥有同时接受能显著地提高接受连接的速率（这就是为什么它默认开启），
    但是可能导致在多进程设置下不均衡的负载。

.. c:function:: int uv_tcp_bind(uv_tcp_t* handle, const struct sockaddr* addr, unsigned int flags)

    绑定句柄到一个地址和端口。 `addr` 应该指向一个初始化了的
    ``struct sockaddr_in`` 或者 ``struct sockaddr_in6`` 。

    当端口已经被占用时，你会预期见到一个 ``UV_EADDRINUSE``
    错误来自于 :c:func:`uv_tcp_bind` 、 :c:func:`uv_listen` 或
    :c:func:`uv_tcp_connect` 之一。 那就是说，一个对此函数成功的调用并不保证对
    :c:func:`uv_listen` 或 :c:func:`uv_tcp_connect`
    的调用也会成功。

    `flags` 能包括 ``UV_TCP_IPV6ONLY`` ，在这种情况下禁用双栈支持且只使用IPv6。

.. c:function:: int uv_tcp_getsockname(const uv_tcp_t* handle, struct sockaddr* name, int* namelen)

    获取句柄当前绑定的地址。 `name` 必须指向一个合法且足够大的内存块，
    推荐使用 ``struct sockaddr_storage`` 获得IPv4和IPv6支持。

.. c:function:: int uv_tcp_getpeername(const uv_tcp_t* handle, struct sockaddr* name, int* namelen)

    连接到句柄的远端的地址。 `name` 必须指向一个合法且足够大的内存块，
    推荐使用 ``struct sockaddr_storage`` 获得IPv4和IPv6支持。

.. c:function:: int uv_tcp_connect(uv_connect_t* req, uv_tcp_t* handle, const struct sockaddr* addr, uv_connect_cb cb)

    建立一个IPv4或IPv6的TCP连接。 提供一个已初始化的TCP句柄和一个未初始化的
    :c:type:`uv_connect_t` 。 `addr` 应该指向一个已初始化的
    ``struct sockaddr_in`` 或 ``struct sockaddr_in6`` 。

    在Windows上如果 `addr` 被初始化指向一个未指定的地址
    （ ``0.0.0.0`` 或者 ``::`` ），它将被改变以指向 ``localhost`` 。
    这么做是为了符合Linux系统的行为。

    这个回调函数当连接已建立或当连接发生错误时被调用。

    .. versionchanged:: 1.19.0 新增 ``0.0.0.0`` 和 ``::`` 到 ``localhost`` 的映射

.. seealso:: :c:type:`uv_stream_t` 的API函数也适用。
