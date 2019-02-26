
.. _tty:

:c:type:`uv_tty_t` --- TTY句柄
=================================

TTY句柄代表对终端的一个流。

:c:type:`uv_tty_t` 是 :c:type:`uv_stream_t` 的一个 '子类型' 。


数据类型
----------

.. c:type:: uv_tty_t

    TTY句柄类型。

.. c:type:: uv_tty_mode_t

    .. versionadded:: 1.2.0

    TTY模式类型：

    ::

      typedef enum {
          /* 初始/正常终端模式 */
          UV_TTY_MODE_NORMAL,
          /* 原始输入模式（在Windows上，也启用了ENABLE_WINDOW_INPUT） */
          UV_TTY_MODE_RAW,
          /* 用于IPC的二进制安全的I/O模式（仅Unix） */
          UV_TTY_MODE_IO
      } uv_tty_mode_t;



公共成员
^^^^^^^^^^^^^^

N/A

.. seealso:: :c:type:`uv_stream_t` 的成员也适用。


API
---

.. c:function:: int uv_tty_init(uv_loop_t* loop, uv_tty_t* handle, uv_file fd, int unused)

    以给定的文件描述符初始化一个新的TTY流。
    通常文件描述符是：

    * 0 = stdin
    * 1 = stdout
    * 2 = stderr

    在Unix上这个函数将会使用 :man:`ttyname_r(3)` 决定终端文件描述符的路径，
    打开它，如果被传递的文件描述符指向一个TTY再使用它。
    这允许libuv将TTY放入非阻塞模式而不影响共享这个TTY的其他进程。

    这个函数在不支持ioctl的TIOCGPTN或TIOCPTYGNAME的系统上不是线程安全的，
    例如OpenBSD和Solaris。

    .. note::
        如果重新打开TTY失败，libuv回退到阻塞写。

    .. versionchanged:: 1.23.1: `readable` 参数现在没用且被忽略。
                        正确的值现在将由内核自动检测。

    .. versionchanged:: 1.9.0: TTY的路径由
                        :man:`ttyname_r(3)` 决定。而在之前的版本中libuv打开
                        `/dev/tty`。

    .. versionchanged:: 1.5.0: 尝试以一个指向一个文件的文件描述符初始化一个TTY流在UNIX上返回
                        `UV_EINVAL` 。

.. c:function:: int uv_tty_set_mode(uv_tty_t* handle, uv_tty_mode_t mode)

    .. versionchanged:: 1.2.0: 模式由 :c:type:`uv_tty_mode_t` 值指定。

    使用指定的终端模式设置TTY。

.. c:function:: int uv_tty_reset_mode(void)

    当程序退出时将被调用。 重设TTY设置到默认值以便被接下来的进程接管。

    这个函数在Unix平台上是异步线程安全的，但是可能以错误代码 ``UV_EBUSY`` 而失败，
    如果你当执行于 :c:func:`uv_tty_set_mode` 中间调用它的时候。

.. c:function:: int uv_tty_get_winsize(uv_tty_t* handle, int* width, int* height)

    获取当前的窗口大小。 成功时返回0。

.. seealso:: :c:type:`uv_stream_t` 的API函数也适用。
