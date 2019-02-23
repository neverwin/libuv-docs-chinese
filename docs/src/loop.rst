
.. _loop:

:c:type:`uv_loop_t` --- 事件循环
==================================

事件循环是libuv功能的核心。
它负责处理I/O轮询，和基于不同事件源的将被运行的回调函数的调度。


数据类型
----------

.. c:type:: uv_loop_t

    循环 数据类型。

.. c:type:: uv_run_mode

    用在以 :c:func:`uv_run` 运行循环的模式。

    ::

        typedef enum {
            UV_RUN_DEFAULT = 0,
            UV_RUN_ONCE,
            UV_RUN_NOWAIT
        } uv_run_mode;

.. c:type:: void (*uv_walk_cb)(uv_handle_t* handle, void* arg)

    传递给 :c:func:`uv_walk` 的回调函数的类型定义。


公共成员
^^^^^^^^^^^^^^

.. c:member:: void* uv_loop_t.data

    用户定义的任意数据的空间。
    libuv不使用且不触及这个字段。


API
---

.. c:function:: int uv_loop_init(uv_loop_t* loop)

    初始化给定的 `uv_loop_t` 结构体。

.. c:function:: int uv_loop_configure(uv_loop_t* loop, uv_loop_option option, ...)

    .. versionadded:: 1.0.2

    设置额外的循环选项。 你通常应该在第一次调用
    :c:func:`uv_run` 之前先调用这个，除非另行说明。

    返回 0 若成功， 或一个UV_E* 错误代码若失败。
    准备好处理 UV_ENOSYS ；它意味着循环选项不被平台所支持。

    支持的选项：

    - UV_LOOP_BLOCK_SIGNAL：当轮询新事件时阻塞信号。
      给 :c:func:`uv_loop_configure` 的第二个参数是信号编号。

      这个选项当前仅实现了SIGPROF信号，
      用于当使用抽样性能分析器时压制不必要的唤醒。
      请求其他的信号将会以 UV_EINVAL 失败。

.. c:function:: int uv_loop_close(uv_loop_t* loop)

    释放所有的内部循环资源。
    调用这个函数仅当循环已经结束执行并且所有开放的句柄和请求已经被关闭，
    否则的话它将返回 UV_EBUSY 。 在这个函数返回后，
    用户可以释放为循环分配的内存。

.. c:function:: uv_loop_t* uv_default_loop(void)

    返回初始化过的默认循环。
    它可能返回 NULL 如若内存分配失败。

    这个函数就是对于需要一个贯穿应用程序的全局循环的一个简单方式，
    默认循环与
    :c:func:`uv_loop_init` 初始化的循环并无差异。 就其本身而言，
    默认循环可以（并且应该）被 :c:func:`uv_loop_close` 关闭，
    以便与它关联的资源被释放。

    .. warning::
        这个函数不是线程安全。

.. c:function:: int uv_run(uv_loop_t* loop, uv_run_mode mode)

    这个函数运行事件循环。 它将依指定的模式而采取不同的行为：

    - UV_RUN_DEFAULT：运行事件循环直到没有更多的活动的和被引用到的句柄或请求。
      返回非零如果 :c:func:`uv_stop`
      被调用且仍有活动的句柄或请求。
      在所有其他情况下返回零。
    - UV_RUN_ONCE：轮询I/O一次。 注意如若没有待处理的回调函数这个函数阻塞。
      返回零当完成时（没有剩余的活动的句柄或请求），
      或者非零值如果期望更多回调函数时
      （意味着你应该在未来某时再次运行这个事件循环）。
    - UV_RUN_NOWAIT：轮询I/O一次但不会阻塞，如若没有待处理的回调函数时。
      返回零当完成时（没有剩余的活动的句柄或请求），
      或者非零值如果期望更多回调函数时
      （意味着你应该在未来某时再次运行这个事件循环）。

.. c:function:: int uv_loop_alive(const uv_loop_t* loop)

    如果有被引用的活动句柄、活动请求或者循环里的关闭句柄时返回非零值。

.. c:function:: void uv_stop(uv_loop_t* loop)

    停止事件循环，致使 :c:func:`uv_run` 尽快结束。
    这不快于下次循环迭代。
    如果这个函数在I/O阻塞前被调用，
    这次循环将不会为I/O阻塞。

.. c:function:: size_t uv_loop_size(void)

    返回 `uv_loop_t` 结构体的大小。
    对不想知道结构体布局的FFI绑定作者有用。

.. c:function:: int uv_backend_fd(const uv_loop_t* loop)

    获取后端文件描述符。
    仅支持kqueue、epoll和event ports。

    这可以跟 `uv_run(loop, UV_RUN_NOWAIT)` 协同使用，
    来在一个线程内轮询同时在另一个线程内运行事件循环的回调函数。
    详见 test/test-embed.c 获取示例。

    .. note::
        在另一个kqueue轮询集合里嵌入一个kqueue文件描述符不在所有平台有效。
        不是一个添加描述符导致的错误，而是不会产生事件。

.. c:function:: int uv_backend_timeout(const uv_loop_t* loop)

    获取轮询时限。 返回值是微秒，或者 -1 当没有时限的时候。

.. c:function:: uint64_t uv_now(const uv_loop_t* loop)

    以微秒返回当前的时间戳。
    时间戳在事件循环计时开始的时候缓存，
    详见 :c:func:`uv_update_time` 获取细节和原理。

    这个时间戳在一些任意的时间点单调增加。
    不要对开始点作出假设，你将只会感到沮丧。

    .. note::
        用 :c:func:`uv_hrtime` 若你需要亚毫秒粒度。

.. c:function:: void uv_update_time(uv_loop_t* loop)

    更新事件循环概念 "now" 。 Libuv在事件循环计时开始时缓存当前时间，
    以减少时间相关的系统调用数目。

    你通常将不会需要调用这个函数，
    除非你有在更长时间周期内阻塞事件循环的回调函数，
    此处 "更长" 有点主观但大概一毫秒或更多。

.. c:function:: void uv_walk(uv_loop_t* loop, uv_walk_cb walk_cb, void* arg)

    遍历句柄列表： `walk_cb` 将以给定的 `arg` 被执行。

.. c:function:: int uv_loop_fork(uv_loop_t* loop)

    .. versionadded:: 1.12.0

    :man:`fork(2)` 系统调用后在子进程内重新初始化任何必要的内核状态。

    先前开启的监视器将继续在子进程内开启。

    有必要在每次在父线程里建立事件循环时显式调用这个函数，
    如若你计划在子进程里继续使用这个循环，
    包括默认循环（即便你不在父线程里继续使用它）。
    这个函数必须在调用
    :c:func:`uv_run` 或任何其他使用到子进程里的循环的API函数之前调用。
    这么做失败了将导致未定义的行为，
    可能包括发给父子进程重复的事件或是中止子进程。

    如果可能，优先在子进程建立一个新循环而不是复用父进程创建的循环。
    fork后在子进程新建的循环不应该使用这个函数。

    这个函数未在 Windows 上实现，这里返回 ``UV_ENOSYS`` 。

    .. caution::

       这个函数是实验性的。 它可能包含bug，且可能修改或删除。
       无法保证 API 和 ABI 稳定性。

    .. note::

        在 Mac OS X 上，如果文件夹FS事件句柄用在父进程的 *任何事件循环* 中，
        子进程将不再能够使用最有效率的FSEvent实现。
        相反，在子进程使用文件夹FS事件句柄将回退到用于文件和其他基于kqueue系统的同等实现。

    .. caution::

       在 AIX 和 SunOS 上，在fork时已经在父进程开启的FS事件句柄将 *不会*
       在子进程里分发事件；
       它们必须被关闭且重启。
       在所有其他系统上，它们继续正常工作无需任何进一步介入。

    .. caution::

       任何之前从 :c:func:`uv_backend_fd` 返回的值现在无效了。
       那个函数必须再次调用以确定正确的后端文件描述符。

.. c:function:: void* uv_loop_get_data(const uv_loop_t* loop)

    返回 `loop->data` 。

    .. versionadded:: 1.19.0

.. c:function:: void* uv_loop_set_data(uv_loop_t* loop, void* data)

    设置 `loop->data` 为 `data` 。

    .. versionadded:: 1.19.0
