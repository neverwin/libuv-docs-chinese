
.. _poll:

:c:type:`uv_poll_t` --- 轮询句柄
===================================

轮询句柄用于监视文件描述符的可读性、可写性和连接断开，
和 :man:`poll(2)` 的目的类似。

轮询句柄的目的是允许集成的依赖于事件循环的外部库可以发出关于套接字状态变化的信号，
像 c-ares 或是 libssh2。 于任何其他目的使用 uv_poll_t 不被推荐；
:c:type:`uv_tcp_t` 、 :c:type:`uv_udp_t` 等等提供了更快的和比起 :c:type:`uv_poll_t` 更具伸缩性的实现，
尤其是在Windows上。

轮询句柄偶尔可能会发出文件描述符可读或可写的信号即使并不是的时候。
用户因此应该总是准备好处理 EAGAIN 或者同类错误，当尝试从文件描述符读取或写入的时候。

对同个套接字有多个活动的轮询句柄是不可以的，
这会导致libuv死循环或者别的故障。

用户不应该关闭文件描述符，当其被一个活动的轮询句柄轮询的时候。
这可能导致句柄报错，但是也可能开始轮询另外一个套接字。
然而在调用 :c:func:`uv_poll_stop` 或 :c:func:`uv_close` 之后，文件描述符立即能被安全地关闭。

.. note::
    在Windows上，只有套接字能被轮询句柄轮询。
    在Unix上，任何被 :man:`poll(2)` 接受的文件描述符都可以用。

.. note::
    在AIX上，不支持监视连接断开。

数据类型
----------

.. c:type:: uv_poll_t

    轮询句柄类型。

.. c:type:: void (*uv_poll_cb)(uv_poll_t* handle, int status, int events)

    传递给 :c:func:`uv_poll_start` 的回调函数的类型定义。

.. c:type:: uv_poll_event

    轮询事件类型

    ::

        enum uv_poll_event {
            UV_READABLE = 1,
            UV_WRITABLE = 2,
            UV_DISCONNECT = 4,
            UV_PRIORITIZED = 8
        };


公共成员
^^^^^^^^^^^^^^

N/A

.. seealso:: :c:type:`uv_handle_t` 的成员也适用。


API
---

.. c:function:: int uv_poll_init(uv_loop_t* loop, uv_poll_t* handle, int fd)

    使用一个文件描述符初始化句柄。

    .. versionchanged:: 1.2.2 文件描述符设为非阻塞模式。

.. c:function:: int uv_poll_init_socket(uv_loop_t* loop, uv_poll_t* handle, uv_os_sock_t socket)

    使用一个套接字描述符初始化句柄。 在Linux上这与
    :c:func:`uv_poll_init` 一样。在Windows上它使用一个SOCKET句柄。

    .. versionchanged:: 1.2.2 套接字设为非阻塞模式。

.. c:function:: int uv_poll_start(uv_poll_t* handle, int events, uv_poll_cb cb)

    开始轮询文件描述符。 `events` 是一个由
    UV_READABLE、UV_WRITABLE、UV_PRIORITIZED和UV_DISCONNECT组成的位掩码。
    当事件一被检测到，回调函数将以 `status` 为 0 被调用，
    并且检测的事件设置于 `events` 字段。

    UV_PRIORITIZED 事件用来监视sysfs中断或是TCP带外信息。

    UV_DISCONNECT 事件就某种意义而言也许不该被报告并且用户可以自由地忽略它，
    但是它可以帮助优化停机过程，因为可能避免额外的读或写调用。

    如果当轮询中发生了错误， `status` 将 < 0 并且对应于一种
    UV_E* 错误代码（详见 :ref:`errors` ）。
    用户不应该当句柄活动时关闭套接字。
    如果用户无论如何关闭了套接字，回调函数 *也许* 会被调用以报告错误状态，但这 **无法** 保证。

    .. note::
        在已经活动的句柄上调用 :c:func:`uv_poll_start` 是可以的。
        这么做将更新正监视中的事件掩码。

    .. note::
        虽然可以设置UV_DISCONNECT，它不支持AIX并且在这种情况下不会出现于回调函数的 `events` 字段。 

    .. versionchanged:: 1.9.0 新增 UV_DISCONNECT 事件。
    .. versionchanged:: 1.14.0 新增 UV_PRIORITIZED 事件。

.. c:function:: int uv_poll_stop(uv_poll_t* poll)

    停止轮询文件描述符，回调函数将不会再被调用。

.. seealso:: :c:type:`uv_handle_t` 的API函数也适用。
