
.. _idle:

:c:type:`uv_idle_t` --- 空转句柄
===================================

空转句柄将在每次循环迭代时运行给定的回调函数一次，
在 :c:type:`uv_prepare_t` 句柄前一刻。

.. note::
    与准备句柄的显著差异在于当有活动空转句柄时，
    循环将以零时限轮询而不是为I/O阻塞。

.. warning::
    虽有不恰当的名字，空转句柄的回调函数在每次循环迭代时都会被调用，
    而不仅仅在循环确实 "空转的" 时候。


数据类型
----------

.. c:type:: uv_idle_t

    空转句柄类型。

.. c:type:: void (*uv_idle_cb)(uv_idle_t* handle)

    传递给 :c:func:`uv_idle_start` 的回调函数的类型定义。


公共成员
^^^^^^^^^^^^^^

N/A

.. seealso:: :c:type:`uv_handle_t` 的成员也适用。


API
---

.. c:function:: int uv_idle_init(uv_loop_t* loop, uv_idle_t* idle)

    初始化句柄。

.. c:function:: int uv_idle_start(uv_idle_t* idle, uv_idle_cb cb)

    以给定的回调函数开始句柄。

.. c:function:: int uv_idle_stop(uv_idle_t* idle)

    停止句柄，回调函数将不会再被调用。

.. seealso:: :c:type:`uv_handle_t` 的API函数也适用。
