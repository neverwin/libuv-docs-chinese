
.. _migration_010_100:

libuv 0.10 -> 1.0.0 迁移指南
===================================

一些API在1.0.0开发过程当中发生了很大的变化。
这是一份迁移指南，关于在0.10发布后发生的最重要的变化。


循环初始化和关闭
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

在libuv 0.10（和任何之前的版本中），循环通过 `uv_loop_new` 创建，
为新的循环分配内存且初始化它；同时通过 `uv_loop_delete`，
销毁循环且释放内存。 从1.0开始，这些被废弃了
并且用户负责分配内存和初始化循环。

libuv 0.10

::

    uv_loop_t* loop = uv_loop_new();
    ...
    uv_loop_delete(loop);

libuv 1.0

::

    uv_loop_t* loop = malloc(sizeof *loop);
    uv_loop_init(loop);
    ...
    uv_loop_close(loop);
    free(loop);

.. note::
    错误处理为了简洁被省略了。 查看文档 :c:func:`uv_loop_init`
    和 :c:func:`uv_loop_close`.


错误处理
~~~~~~~~~~~~~~

错误处理在libuv 1.0中有巨大的翻修。 总的来说，在libuv 0.10中，函数和状态参数里
0 代表成功和 -1 代表失败，用户不得不使用 `uv_last_error`
获取错误代码，这是个正数。

在1.0中，函数和状态参数包括了确切的错误代码，0 代表成功，
错误的时候为一个负数。

libuv 0.10

::

    ... 假如 'server' 是一个已经在侦听中的TCP服务器
    r = uv_listen((uv_stream_t*) server, 511, NULL);
    if (r == -1) {
      uv_err_t err = uv_last_error(uv_default_loop());
      /* err.code 包括 UV_EADDRINUSE */
    }

libuv 1.0

::

    ... 假如 'server' 是一个已经在侦听中的TCP服务器
    r = uv_listen((uv_stream_t*) server, 511, NULL);
    if (r < 0) {
      /* r 包括 UV_EADDRINUSE */
    }


线程池变化
~~~~~~~~~~~~~~~~~~

在libuv 0.10当中，Unix下用了一个默认是4个线程的线程池，而Windows下用了
`QueueUserWorkItem` API，这使用了一个Windows内部的线程池，默认是每个进程 512 个线程。

在1.0中，我们统一了两种实现，这样Windows现在使用和Unix同样的实现。
线程池的大小可以被外部环境变量 ``UV_THREADPOOL_SIZE``
设置。详见 :c:ref:`threadpool`。


分配回调函数 API变化
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

在libuv 0.10中，回调函数必须返回一个 :c:type:`uv_buf_t` 类型的填充值：

::

    uv_buf_t alloc_cb(uv_handle_t* handle, size_t size) {
        return uv_buf_init(malloc(size), size);
    }

在libuv 1.0中，一个指向一个buffer的指针传递给回调函数，用户需要填充它：

::

    void alloc_cb(uv_handle_t* handle, size_t size, uv_buf_t* buf) {
        buf->base = malloc(size);
        buf->len = size;
    }


IPv4 / IPv6 API的统一
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

libuv 1.0统一了IPv4和IPv6 API。 再也没有 `uv_tcp_bind` 和 `uv_tcp_bind6` 这样的二元性，
现在只有 :c:func:`uv_tcp_bind` 。

IPv4函数采用 ``struct sockaddr_in`` 结构体的值，IPv6函数采用
``struct sockaddr_in6``。现在，函数采用 ``struct sockaddr*`` （注意，它是一个指针）。
它能在栈上分配。

libuv 0.10

::

    struct sockaddr_in addr = uv_ip4_addr("0.0.0.0", 1234);
    ...
    uv_tcp_bind(&server, addr)

libuv 1.0

::

    struct sockaddr_in addr;
    uv_ip4_addr("0.0.0.0", 1234, &addr)
    ...
    uv_tcp_bind(&server, (const struct sockaddr*) &addr, 0);

IPv4和IPv6结构创建函数（ :c:func:`uv_ip4_addr` 和 :c:func:`uv_ip6_addr` ）
也变了，确保使用前你查看了文档。

.. note::
    这些变化适用于所有区分IPv4和IPv6地址的函数。


流 / UDP数据接收回调函数 API变化
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

流和UDP数据接收回调函数现在接受一个指向 :c:type:`uv_buf_t` 缓冲区的指针，
而不是一个结构体值。

libuv 0.10

::

    void on_read(uv_stream_t* handle,
                 ssize_t nread,
                 uv_buf_t buf) {
        ...
    }

    void recv_cb(uv_udp_t* handle,
                 ssize_t nread,
                 uv_buf_t buf,
                 struct sockaddr* addr,
                 unsigned flags) {
        ...
    }

libuv 1.0

::

    void on_read(uv_stream_t* handle,
                 ssize_t nread,
                 const uv_buf_t* buf) {
        ...
    }

    void recv_cb(uv_udp_t* handle,
                 ssize_t nread,
                 const uv_buf_t* buf,
                 const struct sockaddr* addr,
                 unsigned flags) {
        ...
    }


通过管道的接收句柄 API变化
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

在libuv 0.10（和之前版本）中， `uv_read2_start` 函数被用来开始在管道上读数据，
可能致使在其上接收句柄。 这类函数的回调函数看起来像这样：

::

    void on_read(uv_pipe_t* pipe,
                 ssize_t nread,
                 uv_buf_t buf,
                 uv_handle_type pending) {
        ...
    }

在libuv 1.0中， `uv_read2_start` 被移除了，并且在读回调函数当中，
用户需要检查是否有待处理的句柄，使用
:c:func:`uv_pipe_pending_count` 和 :c:func:`uv_pipe_pending_type` ：

::

    void on_read(uv_stream_t* handle,
                 ssize_t nread,
                 const uv_buf_t* buf) {
        ...
        while (uv_pipe_pending_count((uv_pipe_t*) handle) != 0) {
            pending = uv_pipe_pending_type((uv_pipe_t*) handle);
            ...
        }
        ...
    }


从句柄里提取文件描述符
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

当它还不被API支持的时候，用户通常访问libuv内部变量，
以获取例如TCP句柄的文件访问符。

::

    fd = handle->io_watcher.fd;

这现在已经正式地暴露于 :c:func:`uv_fileno` 函数。


uv_fs_readdir重命名和API变化
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

在libuv 0.10中，当完成时 `uv_fs_readdir`
返回一个字符串列表于 `req->ptr` 字段。
在1.0中，这个函数被重命名为 :c:func:`uv_fs_scandir` ，
因为这确实通过 ``scandir(3)`` 实现。

另外，用户可以使用 :c:func:`uv_fs_scandir_next`
函数一次获取一个结果，而不是分配完整的列表字符串。
这个函数不需要往返于线程池，因为libuv将保持
``scandir(3)`` 返回的 *凹齿* 列表在一旁。
