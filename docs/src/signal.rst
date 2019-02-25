
.. _signal:

:c:type:`uv_signal_t` --- 信号句柄
=======================================

信号句柄在按事件循环的基础上实现了Unix风格的信号处理。

在Windows上模拟了一些信号的接收：

* SIGINT 通常在用户按 CTRL+C 时被发送。
  然而，如同在Unix上，当终端原始模式开启时这无法保证。

* SIGBREAK 在用户按 CTRL + BREAK 时被发送。

* SIGHUP 在用户关闭终端窗口时被生成。
  在SIGHUP时给予程序大约10秒执行清理。
  在那之后Windows将会无条件地终止程序。

* 对其他类型的信号的监视器能被成功地创建，但是这些信号永远不会被接收到。
  这些信号是： `SIGILL` 、 `SIGABRT` 、 `SIGFPE` 、 `SIGSEGV` 、
  `SIGTERM` 和 `SIGKILL` 。

* 通过编程方式调用 raise() 或 abort() 发出一个信号不会被libuv检测到；
  这些信号将不会触发信号监视器。

.. note::
    在Linux上 SIGRT0 和 SIGRT1 （信号32和33）用于NPTL线程库来管理线程。
    对这些信号安装监视器将会导致无法预测的行为并且强烈不推荐。
    libuv未来的版本中可能会直接拒绝它们。

.. versionchanged:: 1.15.0 在Windows上改进对 SIGWINCH 的支持。

数据类型
----------

.. c:type:: uv_signal_t

    信号句柄类型。

.. c:type:: void (*uv_signal_cb)(uv_signal_t* handle, int signum)

    传递给 :c:func:`uv_signal_start` 的回调函数的类型定义。


公共成员
^^^^^^^^^^^^^^

.. c:member:: int uv_signal_t.signum

    被这个句柄监视的信号。 只读。

.. seealso:: :c:type:`uv_handle_t` 的成员也适用。


API
---

.. c:function:: int uv_signal_init(uv_loop_t* loop, uv_signal_t* signal)

    初始化句柄。

.. c:function:: int uv_signal_start(uv_signal_t* signal, uv_signal_cb cb, int signum)

    以给定的回调函数开始句柄，监视给定的信号。

.. c:function:: int uv_signal_start_oneshot(uv_signal_t* signal, uv_signal_cb cb, int signum)

    .. versionadded:: 1.12.0

    与 :c:func:`uv_signal_start` 同样的功能，但是信号句柄在接受到信号的时刻被重置。

.. c:function:: int uv_signal_stop(uv_signal_t* signal)

    停止句柄，回调函数将不会再被调用。

.. seealso:: :c:type:`uv_handle_t` 的API函数也适用。
