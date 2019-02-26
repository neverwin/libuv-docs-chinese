
.. _pipe:

:c:type:`uv_pipe_t` --- 管道句柄
===================================

管道句柄对Unix上的本地域套接字和Windows上的有名管道提供一个抽象。

:c:type:`uv_pipe_t` 是 :c:type:`uv_stream_t` 的一个 '子类型' 。


数据类型
----------

.. c:type:: uv_pipe_t

    管道句柄类型。


公共成员
^^^^^^^^^^^^^^

.. c:member:: int uv_pipe_t.ipc

    是否这个管道适合在不同进程间传递句柄。

.. seealso:: :c:type:`uv_stream_t` 的成员也适用。


API
---

.. c:function:: int uv_pipe_init(uv_loop_t* loop, uv_pipe_t* handle, int ipc)

    初始化一个管道句柄。 `ipc` 参数是一个布尔值指明是否管道将用于在不同进程间传递句柄。

.. c:function:: int uv_pipe_open(uv_pipe_t* handle, uv_file file)

    打开一个已存在的文件描述符或者句柄作为一个管道。

    .. versionchanged:: 1.2.1 文件描述符设为非阻塞模式。

    .. note::
        被传递的文件描述符或句柄没有类型检查，但是需要它代表一个合法的管道。

.. c:function:: int uv_pipe_bind(uv_pipe_t* handle, const char* name)

    绑定管道到一个文件路径（Unix）或者名字（Windows）。

    .. note::
        Unix上的路径被截取到 ``sizeof(sockaddr_un.sun_path)`` 字节，通常在
        92 到 108 字节之间。

.. c:function:: void uv_pipe_connect(uv_connect_t* req, uv_pipe_t* handle, const char* name, uv_connect_cb cb)

    连接到Unix域套接字或者有名管道。

    .. note::
        Unix上的路径被截取到 ``sizeof(sockaddr_un.sun_path)`` 字节，通常在
        92 到 108 字节之间。

.. c:function:: int uv_pipe_getsockname(const uv_pipe_t* handle, char* buffer, size_t* size)

    获取Unix域套接字或者有名管道的名字。

    必须提供一个预分配的缓冲区。 size参数存有缓冲区的大小并设为在输出上写到缓冲区的字节数。
    如果缓冲区不够大，将返回 ``UV_ENOBUFS`` 并且这个参数将包含需要的大小。

    .. versionchanged:: 1.3.0 返回的长度不再包括终止的空字节，且缓冲区不以空字节终止。

.. c:function:: int uv_pipe_getpeername(const uv_pipe_t* handle, char* buffer, size_t* size)

    获取被句柄连接的Unix域套接字或者有名管道的名字。

    必须提供一个预分配的缓冲区。 size参数存有缓冲区的大小并设为在输出上写到缓冲区的字节数。
    如果缓冲区不够大，将返回 ``UV_ENOBUFS`` 并且这个参数将包含需要的大小。

    .. versionadded:: 1.3.0

.. c:function:: void uv_pipe_pending_instances(uv_pipe_t* handle, int count)

    设置当管道服务器等待连接时未处理的管道实例句柄的数目。

    .. note::
        这个设置只应用于Windows。

.. c:function:: int uv_pipe_pending_count(uv_pipe_t* handle)
.. c:function:: uv_handle_type uv_pipe_pending_type(uv_pipe_t* handle)

    用来通过IPC管道接收句柄。

    首先——调用 :c:func:`uv_pipe_pending_count` ，
    如果 > 0 则初始化给定 `type` 的一个句柄，
    通过 :c:func:`uv_pipe_pending_type` 返回再调用
    ``uv_accept(pipe, handle)`` 。

.. seealso:: :c:type:`uv_stream_t` 的API函数也适用。

.. c:function:: int uv_pipe_chmod(uv_pipe_t* handle, int flags)

    修改管道的权限，允许它被不同用户运行的进程访问。
    使得管道被所有用户可写和可读。
    模式可以是 ``UV_WRITABLE`` 、 ``UV_READABLE`` 或 ``UV_WRITABLE | UV_READABLE`` 。
    这个函数是阻塞的。

    .. versionadded:: 1.16.0
