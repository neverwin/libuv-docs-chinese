
.. _stream:

:c:type:`uv_stream_t` --- 流句柄
=======================================

流句柄提供对双工通信通道的一种抽象。
:c:type:`uv_stream_t` 是一个抽象类型，libuv提供了3种流的实现以
:c:type:`uv_tcp_t` 、 :c:type:`uv_pipe_t` 和 :c:type:`uv_tty_t` 的形式。


数据类型
----------

.. c:type:: uv_stream_t

    流句柄类型。

.. c:type:: uv_connect_t

    连接请求类型。

.. c:type:: uv_shutdown_t

    停机请求类型。

.. c:type:: uv_write_t

    写请求类型。 当复用这种对象时必须小心注意。
    当一个流处在非阻塞模式，用 ``uv_write`` 发送的写请求将被排队。
    在此刻复用对象是未定义的行为。
    仅在传递给 ``uv_write`` 的回调函数执行完毕后才能安全地复用 ``uv_write_t`` 对象。

.. c:type:: void (*uv_read_cb)(uv_stream_t* stream, ssize_t nread, const uv_buf_t* buf)

    数据在流上读取时的回调函数。

    `nread` 是 > 0 如果有可用的数据，或 < 0 当错误时。
    当我们已经到达EOF， `nread` 将被设置为 ``UV_EOF`` 。
    当 `nread` < 0 时 `buf` 参数可能并不指向一个合法的缓冲区；
    在那种情况下 `buf.len` 和 `buf.base` 都被设为0。

    .. note::
        `nread` 可能是0，并 *不* 预示着一个错误或EOF。
        这等同于在 ``read(2)`` 下的 ``EAGAIN`` 或 ``EWOULDBLOCK``。

    调用者负责在错误发生时通过调用 :c:func:`uv_read_stop` 或 :c:func:`uv_close` 停止/关闭流。
    尝试从流再次读取是未定义的。

    调用者负责释放缓冲区，libuv不会复用它。
    在错误时缓冲区可能是一个空缓冲区（这里 buf->base=NULL 且 buf->len=0）。

.. c:type:: void (*uv_write_cb)(uv_write_t* req, int status)

    数据已经在流上写后的回调函数。
    `status` 若成功将是0，否则 < 0。

.. c:type:: void (*uv_connect_cb)(uv_connect_t* req, int status)

    以 :c:func:`uv_connect` 开启连接完成后的回调函数。
    `status` 若成功将是0，否则 < 0。

.. c:type:: void (*uv_shutdown_cb)(uv_shutdown_t* req, int status)

    停机请求完成后的回调函数。
    `status` 若成功将是0，否则 < 0。

.. c:type:: void (*uv_connection_cb)(uv_stream_t* server, int status)

    当流服务器接收到新来的连接时的回调函数。
    用户能够通过调用 :c:func:`uv_accept` 来接受连接。
    `status` 若成功将是0，否则 < 0。


公共成员
^^^^^^^^^^^^^^

.. c:member:: size_t uv_stream_t.write_queue_size

    包含等待发送的排队字节的数量。 只读。

.. c:member:: uv_stream_t* uv_connect_t.handle

    指向此连接请求所运行于的流的指针。

.. c:member:: uv_stream_t* uv_shutdown_t.handle

    指向此停机请求所运行于的流的指针。

.. c:member:: uv_stream_t* uv_write_t.handle

    指向此写请求所运行于的流的指针。

.. c:member:: uv_stream_t* uv_write_t.send_handle

    指向使用此连接请求被发送的的流的指针。

.. seealso:: :c:type:`uv_handle_t` 的成员也适用。


API
---

.. c:function:: int uv_shutdown(uv_shutdown_t* req, uv_stream_t* handle, uv_shutdown_cb cb)

    停机双工流的向外（写）端。 它等待未处理的写请求完成。
    `handle` 应该指向已初始化的流。
    `req` 应该是一个未初始化的停机请求结构体。
    `cb` 在停机完成后被调用。

.. c:function:: int uv_listen(uv_stream_t* stream, int backlog, uv_connection_cb cb)

    开始侦听新来的连接。
    `backlog`指内核可能排队的连接数，与 :man:`listen(2)` 相同。
    当接受到新来的连接时，调用 :c:type:`uv_connection_cb` 回调函数。

.. c:function:: int uv_accept(uv_stream_t* server, uv_stream_t* client)

    调用用来配合 :c:func:`uv_listen` 接受新来的连接。
    在接收到 :c:type:`uv_connection_cb` 后调用这个函数以接受连接。
    在调用这个函数前，客户端句柄必须被初始化。
    < 0 返回值表示错误。

    当 :c:type:`uv_connection_cb` 回调函数被调用时，保证这个函数将会成功第一次。
    如果你尝试使用超过一次，它可能失败。
    建议每个 :c:type:`uv_connection_cb` 调用只调用这个函数一次。

    .. note::
        `server` 和 `client` 必须是运行在同一个循环之上的句柄。、

.. c:function:: int uv_read_start(uv_stream_t* stream, uv_alloc_cb alloc_cb, uv_read_cb read_cb)

    从内向的流读取数据。
    将会调用 :c:type:`uv_read_cb` 回调函数几次直到没有更多数据可读或是调用了 :c:func:`uv_read_stop` 。

.. c:function:: int uv_read_stop(uv_stream_t*)

    停止从流读取数据。
    :c:type:`uv_read_cb` 回调函数将不再被调用。

    这个函数是幂等的且可以在已停止的流上安全地被调用。

.. c:function:: int uv_write(uv_write_t* req, uv_stream_t* handle, const uv_buf_t bufs[], unsigned int nbufs, uv_write_cb cb)

    写数据到流。 缓冲区按序写入。例如：

    ::

        void cb(uv_write_t* req, int status) {
            /* 处理写结果的逻辑 */
        }

        uv_buf_t a[] = {
            { .base = "1", .len = 1 },
            { .base = "2", .len = 1 }
        };

        uv_buf_t b[] = {
            { .base = "3", .len = 1 },
            { .base = "4", .len = 1 }
        };

        uv_write_t req1;
        uv_write_t req2;

        /* 写 "1234" */
        uv_write(&req1, stream, a, 2, cb);
        uv_write(&req2, stream, b, 2, cb);

    .. note::
        被缓冲区指向的内存必须保持有效直到回调函数执行完。
        这也适用于 :c:func:`uv_write2` 。

.. c:function:: int uv_write2(uv_write_t* req, uv_stream_t* handle, const uv_buf_t bufs[], unsigned int nbufs, uv_stream_t* send_handle, uv_write_cb cb)

    扩展的写函数用于在管道上发送句柄。
    管道必须以 `ipc` == 1 初始化。

    .. note::
        `send_handle` 必须是一个TCP套接字或者管道，且为一个服务器或一个连接（侦听或已连接状态）。
        绑定的套接字或管道将被假设是服务器。

.. c:function:: int uv_try_write(uv_stream_t* handle, const uv_buf_t bufs[], unsigned int nbufs)

    与 :c:func:`uv_write` 相同，但是如果无法立刻完成时不会排队写请求。

    将返回以下之一：

    * > 0: 已写字节数（可能小于提供的缓冲区大小）。
    * < 0: 负的错误代码（返回 ``UV_EAGAIN`` 如果没有数据能立刻发送）。

.. c:function:: int uv_is_readable(const uv_stream_t* handle)

    如果流可读返回1，否则0。

.. c:function:: int uv_is_writable(const uv_stream_t* handle)

    如果流可写返回1，否则0。

.. c:function:: int uv_stream_set_blocking(uv_stream_t* handle, int blocking)

    启用或禁用流的阻塞模式。

    当阻塞模式开启时所有的写都是同步完成的。
    别的界面保持不变，比如操作完成或失败将仍然通过回调函数异步被报告。

    .. warning::
        太依赖于这个API是不推荐的。
        它可能在未来明显地变化。

        当前在Windows上只工作于 :c:type:`uv_pipe_t` 句柄。
        在UNIX平台，所有的 :c:type:`uv_stream_t` 句柄都被支持。

        另外当前libuv当阻塞模式在已经提交写请求之后改变时没有作顺序保证。
        因此推荐在打开或创建流之后立即设置阻塞模式。

    .. versionchanged:: 1.4.0 新增UNIX实现。

.. c:function:: size_t uv_stream_get_write_queue_size(const uv_stream_t* stream)

    返回 `stream->write_queue_size` 。

    .. versionadded:: 1.19.0

.. seealso:: :c:type:`uv_handle_t` 的API函数也适用。
