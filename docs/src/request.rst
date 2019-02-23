
.. _request:

:c:type:`uv_req_t` --- 基础请求
===================================

`uv_req_t` 是所有libuv请求类型的基类型。

结构体是对齐的以便任何libuv请求能转化成 `uv_req_t`。
这里定义的所有API函数适用于任意请求类型。


数据类型
----------

.. c:type:: uv_req_t

    基础libuv请求结构体。

.. c:type:: uv_any_req

    所有请求类型的并集。


公共成员
^^^^^^^^^^^^^^

.. c:member:: void* uv_req_t.data

    用户定义的任意数据的空间。 libuv不使用且不触及这个字段。

.. c:member:: uv_req_type uv_req_t.type

    指向请求的类型。 只读。

    ::

        typedef enum {
            UV_UNKNOWN_REQ = 0,
            UV_REQ,
            UV_CONNECT,
            UV_WRITE,
            UV_SHUTDOWN,
            UV_UDP_SEND,
            UV_FS,
            UV_WORK,
            UV_GETADDRINFO,
            UV_GETNAMEINFO,
            UV_REQ_TYPE_MAX,
        } uv_req_type;


API
---

.. c:function:: UV_REQ_TYPE_MAP(iter_macro)

    对每个请求类型扩展出一系列的 `iter_macro` 调用的宏。
    `iter_macro` 以两个参数调用：不带 `UV_` 前缀的 `uv_req_type` 元素名，
    和不带 `uv_` 前缀和 `_t` 后缀的对应的结构体类型名。

.. c:function:: int uv_cancel(uv_req_t* req)

    取消待处理的请求。 如果请求在执行中或已经执行完毕时失败。

    返回 0 当成功时，或者一个 < 0 的错误代码当失败时。

    当前仅支持取消 :c:type:`uv_fs_t` 、 :c:type:`uv_getaddrinfo_t` 、
    :c:type:`uv_getnameinfo_t` 和 :c:type:`uv_work_t` 请求。 

    取消的请求的回调函数在未来某时被调用。
    释放关联于请求的内存是 **不** 安全的直到回调函数调用之后。

    这是取消如何报告给回调函数的方式：

    * 一个 :c:type:`uv_fs_t` 请求的 req->result 字段设为 `UV_ECANCELED` 。

    * 一个 :c:type:`uv_work_t` 、 :c:type:`uv_getaddrinfo_t` 或 :c:type:`uv_getnameinfo_t`
      请求的回调函数以 status == `UV_ECANCELED` 被调用。

.. c:function:: size_t uv_req_size(uv_req_type type)

    返回给定请求类型的大小。
    对不想知道结构体布局的FFI绑定作者有用。

.. c:function:: void* uv_req_get_data(const uv_req_t* req)

    返回 `req->data`.

    .. versionadded:: 1.19.0

.. c:function:: void* uv_req_set_data(uv_req_t* req, void* data)

    设置 `req->data` 为 `data`.

    .. versionadded:: 1.19.0

.. c:function:: uv_req_type uv_req_get_type(const uv_req_t* req)

    返回 `req->type`.

    .. versionadded:: 1.19.0

.. c:function:: const char* uv_req_type_name(uv_req_type type)

    返回给定请求类型等效的结构体名称，
    例如对 `UV_CONNECT` 是 `"connect"` （即 :c:type:`uv_connect_t` ）。

    如果不存在这样的请求类型，它返回 `NULL` 。

    .. versionadded:: 1.19.0
