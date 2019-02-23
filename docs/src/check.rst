
.. _check:

:c:type:`uv_check_t` --- 检查句柄
=====================================

检查句柄将在每次循环迭代时运行给定的回调函数一次，
在I/O轮询后一刻。


数据类型
----------

.. c:type:: uv_check_t

    检查句柄类型。

.. c:type:: void (*uv_check_cb)(uv_check_t* handle)

    传递给 :c:func:`uv_check_start` 的回调函数的类型定义。


公共成员
^^^^^^^^^^^^^^

N/A

.. seealso:: :c:type:`uv_handle_t` 的成员也适用。


API
---

.. c:function:: int uv_check_init(uv_loop_t* loop, uv_check_t* check)

    初始化句柄。

.. c:function:: int uv_check_start(uv_check_t* check, uv_check_cb cb)

    以给定的回调函数开始句柄。

.. c:function:: int uv_check_stop(uv_check_t* check)

    停止句柄，回调函数将不会再被调用。

.. seealso:: :c:type:`uv_handle_t` 的API函数也适用。
