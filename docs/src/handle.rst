
.. _handle:

:c:type:`uv_handle_t` --- 基础句柄
=====================================

`uv_handle_t` 是所有libuv句柄类型的基类型。

结构体是对齐的以便任何libuv句柄能转化成 `uv_handle_t`。
这里定义的所有API函数适用于任意句柄类型。 

libuv句柄无法移动。
传递给函数的句柄结构体指针在请求的操作期间必须保持有效。
当使用栈分配的句柄时请小心。

数据类型
----------

.. c:type:: uv_handle_t

    基础的libuv句柄类型。

.. c:type:: uv_handle_type

    libuv句柄的种类。

    ::

        typedef enum {
          UV_UNKNOWN_HANDLE = 0,
          UV_ASYNC,
          UV_CHECK,
          UV_FS_EVENT,
          UV_FS_POLL,
          UV_HANDLE,
          UV_IDLE,
          UV_NAMED_PIPE,
          UV_POLL,
          UV_PREPARE,
          UV_PROCESS,
          UV_STREAM,
          UV_TCP,
          UV_TIMER,
          UV_TTY,
          UV_UDP,
          UV_SIGNAL,
          UV_FILE,
          UV_HANDLE_TYPE_MAX
        } uv_handle_type;

.. c:type:: uv_any_handle

    所有句柄类型的并集。

.. c:type:: void (*uv_alloc_cb)(uv_handle_t* handle, size_t suggested_size, uv_buf_t* buf)

    传递给 :c:func:`uv_read_start` 和 :c:func:`uv_udp_recv_start` 的回调函数的类型定义。
    用户必须分配内存并填充 :c:type:`uv_buf_t` 结构体。 如果赋值NULL作为缓冲区的基址或0作为缓冲区长度，
    将会在 :c:type:`uv_udp_recv_cb` 或 :c:type:`uv_read_cb` 的回调函数里触发一个 ``UV_ENOBUFS`` 错误。

    提供了建议的大小（目前大多数情况是65536），但这只是个标示，
    与待读取的数据没有任何关系。 用户自行决定分配多少内存。

    例如，使用了像freelists、分配池或slab分配器这类自定义分配模式的程序\
    可能决定使用一个不同的大小以匹配于它们所用的memory chunks。

    例子：

    ::

        static void my_alloc_cb(uv_handle_t* handle, size_t suggested_size, uv_buf_t* buf) {
          buf->base = malloc(suggested_size);
          buf->len = suggested_size;
        }

.. c:type:: void (*uv_close_cb)(uv_handle_t* handle)

    传递给 :c:func:`uv_close` 的回调函数的类型定义。


公共成员
^^^^^^^^^^^^^^

.. c:member:: uv_loop_t* uv_handle_t.loop

    指向句柄所运行在的 :c:type:`uv_loop_t` 。 只读。

.. c:member:: uv_handle_type uv_handle_t.type

    :c:type:`uv_handle_type` ，指向潜在句柄的类型。 只读。

.. c:member:: void* uv_handle_t.data

    用户定义的任意数据的空间。 libuv不使用且不触及这个字段。


API
---

.. c:function:: UV_HANDLE_TYPE_MAP(iter_macro)

    对每个句柄类型扩展出一系列的 `iter_macro` 调用的宏。
    `iter_macro` 以两个参数调用：不带 `UV_` 前缀的 `uv_handle_type` 元素名，
    和不带 `uv_` 前缀和 `_t` 后缀的对应的结构体类型名。

.. c:function:: int uv_is_active(const uv_handle_t* handle)

    如果句柄活动返回非零值，如果不活动返回零。
    "活动" 的意思依赖于句柄类型：

    - uv_async_t 句柄总是活动的且无法不活动，
      除非以 uv_close() 关闭它。

    - uv_pipe_t、uv_tcp_t、uv_udp_t 等句柄
      —— 基本上任何处理I/O的句柄
      —— 当做牵涉到I/O的事情时是活动的，
      像读、写、连接、接受新连接等等。

    - uv_check_t, uv_idle_t, uv_timer_t 等句柄是活动的当以
      uv_check_start(), uv_idle_start() 等调用开始时。

    经验法则：如果一个 `uv_foo_t` 类型的句柄有个 `uv_foo_start()` 函数，
    则它从那个函数调用起是活动的。
    同样地， `uv_foo_stop()` 使句柄再次不活动。

.. c:function:: int uv_is_closing(const uv_handle_t* handle)

    如果句柄正在关闭或已经关闭返回非零值，否则零。

    .. note::
        这个函数只应该在句柄初始化和关闭回调函数到来前这段时间内被使用。

.. c:function:: void uv_close(uv_handle_t* handle, uv_close_cb close_cb)

    请求句柄关闭。 `close_cb` 将在这个调用之后被异步调用。
    这个函数必须在每个句柄释放内存前调用。
    此外，内存只能在 `close_cb` 里或者返回后释放。

    封装文件描述符的句柄立即关闭，
    但是 `close_cb` 将会被推迟到下次事件循环迭代。
    给你释放关联于句柄的任何资源的机会。

    进行中请求，像 uv_connect_t 或 uv_write_t，
    被取消并且它们的回调函数以 status=UV_ECANCELED 被异步调用。

.. c:function:: void uv_ref(uv_handle_t* handle)

    引用给定的句柄。 引用是幂等的，也就是说，
    如果已经被引用的句柄再调用这个函数无效果。

    See :ref:`refcount`.

.. c:function:: void uv_unref(uv_handle_t* handle)

    反引用给定的句柄。 引用是幂等的，也就是说，
    如果没有被引用的句柄再调用这个函数无效果。

    See :ref:`refcount`.

.. c:function:: int uv_has_ref(const uv_handle_t* handle)

    如果句柄被引用了，返回非零，否则是零。

    See :ref:`refcount`.

.. c:function:: size_t uv_handle_size(uv_handle_type type)

    返回给定句柄类型的大小。
    对不想知道结构体布局的FFI绑定作者有用。


杂项 API函数
---------------------------

下面的API函数用到一个 :c:type:`uv_handle_t` 类型的参数，但是它们只适用于某些句柄类型。

.. c:function:: int uv_send_buffer_size(uv_handle_t* handle, int* value)

    获取或是设置操作系统用于套接字的发送缓存的大小。

    如果 `*value` == 0 ，将返回当前发送缓存大小，
    否则它将用 `*value` 设置新的发送缓存大小。

    此函数在Unix上对TCP、管道和UDP句柄有效，
    在Windows上对TCP和UDP句柄有效。

    .. note::
        Linux将会设置双倍的大小，并且返回的是原先设置值的双倍大小。 

.. c:function:: int uv_recv_buffer_size(uv_handle_t* handle, int* value)

    获取或是设置操作系统用于套接字的接收缓存的大小。

    如果 `*value` == 0 ，将返回当前接收缓存大小，
    否则它将用 `*value` 设置新的发送接收大小。

    此函数在Unix上对TCP、管道和UDP句柄有效，
    在Windows上对TCP和UDP句柄有效。

    .. note::
        Linux将会设置双倍的大小，并且返回的是原先设置值的双倍大小。 

.. c:function:: int uv_fileno(const uv_handle_t* handle, uv_os_fd_t* fd)

    获取平台相关的等效文件描述符。

    以下句柄被支持：TCP、管道、TTY、UDP和轮询。
    传递其他任何句柄类型将会以 `UV_EINVAL` 失败。

    如果一个句柄尚未有依附的文件描述符或句柄被关闭了，
    这个返回将返回 `UV_EBADF` 。

    .. warning::
        使用这个函数时请小心。
        libuv假定它控制着文件描述符，所以对文件描述符的任何改变可能引发失灵。

.. c:function:: uv_loop_t* uv_handle_get_loop(const uv_handle_t* handle)

    返回 `handle->loop` 。

    .. versionadded:: 1.19.0

.. c:function:: void* uv_handle_get_data(const uv_handle_t* handle)

    返回 `handle->data` 。

    .. versionadded:: 1.19.0

.. c:function:: void* uv_handle_set_data(uv_handle_t* handle, void* data)

    设置 `handle->data` 为 `data` 。

    .. versionadded:: 1.19.0

.. c:function:: uv_handle_type uv_handle_get_type(const uv_handle_t* handle)

    返回 `handle->type` 。

    .. versionadded:: 1.19.0

.. c:function:: const char* uv_handle_type_name(uv_handle_type type)

    返回给定句柄类型等效的结构体名称，
    例如对 `UV_NAMED_PIPE` 是 `"pipe"` （即 :c:type:`uv_pipe_t` ）。

    如果不存在这样的句柄类型，它返回 `NULL` 。

    .. versionadded:: 1.19.0

.. _refcount:

引用计数
------------------

libuv事件循环（如果运行在默认模式）运行直至没有剩下活动的
`和` 被引用的句柄。用户能够通过对活动句柄解引用来强制循环提前退出，
例如调用 :c:func:`uv_unref`
在调用 :c:func:`uv_timer_start` 之后。

句柄可以被引用和解引用，引用计数模式没有用到计数器，
所以两种操作是幂等的。

所有活动句柄默认是被引用的，参见 :c:func:`uv_is_active`
获取在 `活动的` 关联上更详尽的解释。
