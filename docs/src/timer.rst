
.. _timer:

:c:type:`uv_timer_t` --- 计时器句柄
=====================================

计时器句柄用于调度回调函数在将来被调用。


数据类型
----------

.. c:type:: uv_timer_t

    计时器句柄类型。

.. c:type:: void (*uv_timer_cb)(uv_timer_t* handle)

    传递给 :c:func:`uv_timer_start` 的回调函数的类型定义。


公共成员
^^^^^^^^^^^^^^

N/A

.. seealso:: :c:type:`uv_handle_t` 的成员也适用。


API
---

.. c:function:: int uv_timer_init(uv_loop_t* loop, uv_timer_t* handle)

    初始化句柄。

.. c:function:: int uv_timer_start(uv_timer_t* handle, uv_timer_cb cb, uint64_t timeout, uint64_t repeat)

    开启计时器。 `timeout` 和 `repeat` 以微秒计。

    如果 `timeout` 是零，回调函数在下一次事件循环迭代时运行。
    如果 `repeat` 是非零值，回调函数首次在 `timeout` 毫秒后运行，之后每隔 `repeat` 毫秒重复运行。

    .. note::
        不要更新事件循环概念"now"。 详见 :c:func:`uv_update_time` 获取更多信息。

        如果计时器已经活动，它简单地被更新。

.. c:function:: int uv_timer_stop(uv_timer_t* handle)

    停止计时器，回调函数将不会再被调用。

.. c:function:: int uv_timer_again(uv_timer_t* handle)

    停止计时器，并且如果它是重复的则用repeat值作为timeout重启它。
    如果计时器之前从未被启用，返回 UV_EINVAL 。

.. c:function:: void uv_timer_set_repeat(uv_timer_t* handle, uint64_t repeat)

    以微秒设置重复间隔值。 计时器将以给定的间隔被调度运行，
    不管回调函数的执行周期，并且在时间片超出时遵循正常计时器语义。

    比方说，如果一个50ms重复的计时器首次运行于17ms，
    它将在33ms之后再次运行。如果在首次回调函数之后其他的任务花费了超过33ms，
    则回调函数将会尽可能快运行。

    .. note::
        如果repeat值从一个计时器回调函数里被设置，它不会立即生效。
        如果计时器之前不是重复的，之前的计时将被停止。
        如果它是重复的，则旧的repeat值将被用来调度下一次时限。

.. c:function:: uint64_t uv_timer_get_repeat(const uv_timer_t* handle)

    获取计时器 repeat 值。

.. seealso:: :c:type:`uv_handle_t` 的API函数也适用。
