
.. _prepare:

:c:type:`uv_prepare_t` --- 准备句柄
=========================================

准备句柄将在每次循环迭代时运行给定的回调函数，
在I/O轮询前一刻。


数据类型
----------

.. c:type:: uv_prepare_t

    准备句柄类型。

.. c:type:: void (*uv_prepare_cb)(uv_prepare_t* handle)

    传递给 :c:func:`uv_prepare_start` 的回调函数的类型定义。


公共成员
^^^^^^^^^^^^^^

N/A

.. seealso:: :c:type:`uv_handle_t` 的成员也适用。


API
---

.. c:function:: int uv_prepare_init(uv_loop_t* loop, uv_prepare_t* prepare)

    初始化句柄。

.. c:function:: int uv_prepare_start(uv_prepare_t* prepare, uv_prepare_cb cb)

    以给定的回调函数开始句柄。

.. c:function:: int uv_prepare_stop(uv_prepare_t* prepare)

    停止句柄，回调函数将不会再被调用。

.. seealso:: :c:type:`uv_handle_t` 的API函数也适用。
