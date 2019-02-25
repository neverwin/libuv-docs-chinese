
.. _process:

:c:type:`uv_process_t` --- 进程句柄
=========================================

进程句柄将会生成新进程，并且允许用户控制它和使用流与它建立通信通道。


数据类型
----------

.. c:type:: uv_process_t

    进程句柄类型。

.. c:type:: uv_process_options_t

    生成进程的选项（传递给 :c:func:`uv_spawn` 的）

    ::

        typedef struct uv_process_options_s {
            uv_exit_cb exit_cb;
            const char* file;
            char** args;
            char** env;
            const char* cwd;
            unsigned int flags;
            int stdio_count;
            uv_stdio_container_t* stdio;
            uv_uid_t uid;
            uv_gid_t gid;
        } uv_process_options_t;

.. c:type:: void (*uv_exit_cb)(uv_process_t*, int64_t exit_status, int term_signal)

    传递给 :c:type:`uv_process_options_t` 的回调函数的类型定义，
    将指明退出状态和引发进程终止的信号，如果有的话。

.. c:type:: uv_process_flags

    设置在 :c:type:`uv_process_options_t` 标志字段的标志。

    ::

        enum uv_process_flags {
            /*
            * 设置子进程的用户ID。
            */
            UV_PROCESS_SETUID = (1 << 0),
            /*
            * 设置子进程的组ID。
            */
            UV_PROCESS_SETGID = (1 << 1),
            /*
            * 当将参数列表转换为一个命令行字符串时，不要用引号包围参数，抑或是任何其它转义。
            * 这个选项只在Windows系统上有效，在Unix上被忽略。
            */
            UV_PROCESS_WINDOWS_VERBATIM_ARGUMENTS = (1 << 2),
            /*
            * 以分离状态生成子进程——这将使得它成为进程组的组长，
            * 将有效地允许子进程在父进程退出之后继续运行。
            * 注意子进程将保持父进程的事件循环处于活动状态，
            * 除非父进程在子进程句柄上调用 uv_unref() 。
            */
            UV_PROCESS_DETACHED = (1 << 3),
            /*
            * 隐藏子进程默认被创建的窗口。
            * 这个选项只在Windows系统上有效，在Unix上被忽略。
            */
            UV_PROCESS_WINDOWS_HIDE = (1 << 4),
            /*
            * 隐藏子进程默认被创建的终端窗口。
            * 这个选项只在Windows系统上有效，在Unix上被忽略。
            */
            UV_PROCESS_WINDOWS_HIDE_CONSOLE = (1 << 5),
            /*
            * 隐藏子进程默认被创建的GUI窗口。
            * 这个选项只在Windows系统上有效，在Unix上被忽略。
            */
            UV_PROCESS_WINDOWS_HIDE_GUI = (1 << 6)
        };

.. c:type:: uv_stdio_container_t

    传递给子进程的每个stdio句柄或文件描述符的容器。

    ::

        typedef struct uv_stdio_container_s {
            uv_stdio_flags flags;
            union {
                uv_stream_t* stream;
                int fd;
            } data;
        } uv_stdio_container_t;

.. c:type:: uv_stdio_flags

    指定stdio如何被传送到子进程的标志。

    ::

        typedef enum {
            UV_IGNORE = 0x00,
            UV_CREATE_PIPE = 0x01,
            UV_INHERIT_FD = 0x02,
            UV_INHERIT_STREAM = 0x04,
            /*
            * 当指定UV_CREATE_PIPE时，UV_READABLE_PIPE和UV_WRITABLE_PIPE决定了从子进程视角的数据流方向，
            * 可以两个被同时指定，以创建双工数据流。
            */
            UV_READABLE_PIPE = 0x10,
            UV_WRITABLE_PIPE = 0x20
            /*
             * 在Windows上以重叠模式打开子进程管道句柄。
             * 在Unix上被忽略。
             */
            UV_OVERLAPPED_PIPE = 0x40
        } uv_stdio_flags;


公共成员
^^^^^^^^^^^^^^

.. c:member:: uv_process_t.pid

    生成的进程的PID。 在 :c:func:`uv_spawn` 调用后被设置。

.. note::
    :c:type:`uv_handle_t` 的成员也适用。

.. c:member:: uv_process_options_t.exit_cb

    进程退出时调用的回调函数。

.. c:member:: uv_process_options_t.file

    指向被执行程序的路径。

.. c:member:: uv_process_options_t.args

    命令行参数。 args[0] 应该是程序路径。
    在Windows上这使用了 `CreateProcess` 连接参数为一个字符串，可能导致一些奇怪的错误。
    详见 :c:type:`uv_process_flags` 上的 ``UV_PROCESS_WINDOWS_VERBATIM_ARGUMENTS`` 标志。

.. c:member:: uv_process_options_t.env

    新进程的环境。 如果为空使用父进程的环境。

.. c:member:: uv_process_options_t.cwd

    子进程的当前工作目录。

.. c:member:: uv_process_options_t.flags

    控制 :c:func:`uv_spawn` 行为的各种标志。
    详见 :c:type:`uv_process_flags` 。

.. c:member:: uv_process_options_t.stdio_count
.. c:member:: uv_process_options_t.stdio

    指向 :c:type:`uv_stdio_container_t` 结构体数组的 `stdio` 指针字段，描述了将对子进程生效的文件描述符。
    惯例是文件描述符0（stdio[0]指向的）用于stdin，文件描述符1用于stdout，文件描述符2用于stderr。

    .. note::
        在Windows上文件描述符大于2只对使用MSVCRT运行时的子进程有效。

.. c:member:: uv_process_options_t.uid
.. c:member:: uv_process_options_t.gid

    libuv能够改变子进程的用户/组ID。
    这只发生于在标志字段设置恰当的比特时。

    .. note::
        在Windows上这不被支持， :c:func:`uv_spawn` 将会失败并且设置错误为 ``UV_ENOTSUP`` 。

.. c:member:: uv_stdio_container_t.flags

    标志位指明stdio容器应该怎样被传递给子进程。
    详见 :c:type:`uv_stdio_flags` 。

.. c:member:: uv_stdio_container_t.data

    包括传递给子进程的流或文件描述符的共同体。


API
---

.. c:function:: void uv_disable_stdio_inheritance(void)

    禁用继承来自父进程的文件描述符/句柄。
    效果是此进程生成的子进程不会意外地继承这些句柄。

    在继承的文件描述符能被关闭和复制前，推荐在你的程序中尽早调用这个函数。

    .. note::
        这个函数工作在尽力而为的基础上：不保证libuv能够发现所有继承的文件描述符。
        通常这在Windows上比在Unix上工作地更好。

.. c:function:: int uv_spawn(uv_loop_t* loop, uv_process_t* handle, const uv_process_options_t* options)

    初始化进程描述符并且启动进程。
    如果进程成功生成，这个函数返回0。
    否则，返回对应于无法生成原因的负数错误代码。

    生成失败的可能原因包括（但不限于）
    要执行的文件不存在、没权限使用指定的setuid或setgid
    或者没有足够的内存分配给新进程。

    .. versionchanged:: 1.24.0 新增 `UV_PROCESS_WINDOWS_HIDE_CONSOLE` 和
                        `UV_PROCESS_WINDOWS_HIDE_GUI` 标志。

.. c:function:: int uv_process_kill(uv_process_t* handle, int signum)

    发送指定的信号到给定的进程句柄。
    检查 :c:ref:`signal` 上的文档对于信号的支持，特别是在Windows上。

.. c:function:: int uv_kill(int pid, int signum)

    发送指定的信号到给定的PID。
    检查 :c:ref:`signal` 上的文档对于信号的支持，特别是在Windows上。

.. c:function:: uv_pid_t uv_process_get_pid(const uv_process_t* handle)

    返回 `handle->pid` 。

    .. versionadded:: 1.19.0

.. seealso:: :c:type:`uv_handle_t` 的API函数也适用。
