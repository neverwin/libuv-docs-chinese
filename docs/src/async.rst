
.. _async:

:c:type:`uv_async_t` --- 异步句柄
=====================================

异步句柄允许用户 "唤醒" 事件循环并且从另一个线程调用回调函数。


数据类型
----------

.. c:type:: uv_async_t

    异步句柄类型。

.. c:type:: void (*uv_async_cb)(uv_async_t* handle)

    传递给 :c:func:`uv_async_init` 的回调函数的类型定义。


公共成员
^^^^^^^^^^^^^^

N/A

.. seealso:: :c:type:`uv_handle_t` 的成员也适用。


API
---

.. c:function:: int uv_async_init(uv_loop_t* loop, uv_async_t* async, uv_async_cb async_cb)

    初始化句柄。 允许回调函数为NULL。

    :returns: 0 当成功时，或者一个 < 0 的错误代码当失败时。

    .. note::
        不同于其他句柄初始化函数，句柄立刻开始。

.. c:function:: int uv_async_send(uv_async_t* async)

    唤醒事件循环并且调用异步句柄的回调函数。

    :returns: 0 当成功时，或者一个 < 0 的错误代码当失败时。

    .. note::
        从任何线程调用这个函数都是安全的。
        回调函数将从循环的线程上被调用。

    .. warning::
        libuv将会合并对 :c:func:`uv_async_send` 的调用，那就是说，不是对它的每个调用会 yield 回调函数的执行。
        例如：如果在回调函数被调用前一连调用 :c:func:`uv_async_send` 5 次，回调函数将只会调用一次。
        如果在回调函数被调用后再次调用 :c:func:`uv_async_send` ，回调函数将会再次被调用。

.. seealso::
    :c:type:`uv_handle_t` 的API函数也适用。
