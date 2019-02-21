
.. _errors:

错误处理
==============

在libuv中，错误是负数常量。 根据经验，
不管何时有一个状态参数，或是一个API函数返回一个整数的时候，
一个负数意味着一个错误。

当一个使用回调函数的函数返回了一个错误，
这个回调函数将永远不会被调用。

.. note::
    实现的细节：在 Unix 中错误代码是负的 `errno` （或者说 `-errno`），当在
    Windows 上它们由libuv定义为任意的负数。


错误常量
---------------

.. c:macro:: UV_E2BIG

    参数列表太长了

.. c:macro:: UV_EACCES

    权限被拒绝

.. c:macro:: UV_EADDRINUSE

    地址已经被使用

.. c:macro:: UV_EADDRNOTAVAIL

    地址不可用

.. c:macro:: UV_EAFNOSUPPORT

    不支持的地址族

.. c:macro:: UV_EAGAIN

    资源临时不可用

.. c:macro:: UV_EAI_ADDRFAMILY

    不支持的地址族

.. c:macro:: UV_EAI_AGAIN

    临时性失败

.. c:macro:: UV_EAI_BADFLAGS

    错的 ai_flags 值

.. c:macro:: UV_EAI_BADHINTS

    无效的指示值

.. c:macro:: UV_EAI_CANCELED

    请求取消了

.. c:macro:: UV_EAI_FAIL

    永久性失败

.. c:macro:: UV_EAI_FAMILY

    ai_family 不支持

.. c:macro:: UV_EAI_MEMORY

    内存用完

.. c:macro:: UV_EAI_NODATA

    没有地址

.. c:macro:: UV_EAI_NONAME

    未知的代码或服务

.. c:macro:: UV_EAI_OVERFLOW

    参数缓存越界

.. c:macro:: UV_EAI_PROTOCOL

    解析的协议未知

.. c:macro:: UV_EAI_SERVICE

    对套接字类型，服务不可用

.. c:macro:: UV_EAI_SOCKTYPE

    套接字类型不支持

.. c:macro:: UV_EALREADY

    连接已经在进行中

.. c:macro:: UV_EBADF

    错的文件描述符

.. c:macro:: UV_EBUSY

    资源忙或是锁定了

.. c:macro:: UV_ECANCELED

    操作取消了

.. c:macro:: UV_ECHARSET

    非法的 Unicode 字符

.. c:macro:: UV_ECONNABORTED

    软件导致的连接中止

.. c:macro:: UV_ECONNREFUSED

    连接被拒绝

.. c:macro:: UV_ECONNRESET

    连接被远端重置

.. c:macro:: UV_EDESTADDRREQ

    需要目的地址

.. c:macro:: UV_EEXIST

    文件已经存在

.. c:macro:: UV_EFAULT

    在系统调用参数里有错的地址

.. c:macro:: UV_EFBIG

    文件太大了

.. c:macro:: UV_EHOSTUNREACH

    主机不可达

.. c:macro:: UV_EINTR

    中断的系统调用

.. c:macro:: UV_EINVAL

    非法参数

.. c:macro:: UV_EIO

    i/o 错误

.. c:macro:: UV_EISCONN

    套接字已连接

.. c:macro:: UV_EISDIR

    在文件夹上的非法操作

.. c:macro:: UV_ELOOP

    遇到了太多符号链接

.. c:macro:: UV_EMFILE

    打开的文件太多了

.. c:macro:: UV_EMSGSIZE

    消息太长了

.. c:macro:: UV_ENAMETOOLONG

    名字太长了

.. c:macro:: UV_ENETDOWN

    网络停机

.. c:macro:: UV_ENETUNREACH

    网络不可达

.. c:macro:: UV_ENFILE

    文件表溢出

.. c:macro:: UV_ENOBUFS

    没有可用的缓存空间

.. c:macro:: UV_ENODEV

    没有这样的设备

.. c:macro:: UV_ENOENT

    没哟这样的文件或文件夹

.. c:macro:: UV_ENOMEM

    内存不够

.. c:macro:: UV_ENONET

    机器不在网络上

.. c:macro:: UV_ENOPROTOOPT

    协议不可用

.. c:macro:: UV_ENOSPC

    设备上没有剩余空间

.. c:macro:: UV_ENOSYS

    未被实现的函数

.. c:macro:: UV_ENOTCONN

    套接字未连接

.. c:macro:: UV_ENOTDIR

    不是一个文件夹

.. c:macro:: UV_ENOTEMPTY

    文件夹非空

.. c:macro:: UV_ENOTSOCK

    在非套接字上进行套接字操作

.. c:macro:: UV_ENOTSUP

    套接字不支持的操作

.. c:macro:: UV_EPERM

    不允许的操作

.. c:macro:: UV_EPIPE

    破碎的管道

.. c:macro:: UV_EPROTO

    协议错误

.. c:macro:: UV_EPROTONOSUPPORT

    协议不支持

.. c:macro:: UV_EPROTOTYPE

    对套接字的错误的协议类型

.. c:macro:: UV_ERANGE

    结果太大了

.. c:macro:: UV_EROFS

    只读的文件系统

.. c:macro:: UV_ESHUTDOWN

    不能在传输终点关机后发送

.. c:macro:: UV_ESPIPE

    非法查寻

.. c:macro:: UV_ESRCH

    没有这样的进程

.. c:macro:: UV_ETIMEDOUT

    连接超时

.. c:macro:: UV_ETXTBSY

    文本文件忙

.. c:macro:: UV_EXDEV

    不允许跨设备链接

.. c:macro:: UV_UNKNOWN

    未知错误

.. c:macro:: UV_EOF

    文件结尾

.. c:macro:: UV_ENXIO

    没有这样的设备或地址

.. c:macro:: UV_EMLINK

    太多的链接


API
---

.. c:function:: UV_ERRNO_MAP(iter_macro)

    对以上每个错误常量扩展出一系列的 `iter_macro` 调用的宏。
    `iter_macro` 以两个参数调用：不带 `UV_` 前缀的错误常量名，
    和错误信息字符串字面量。

.. c:function:: const char* uv_strerror(int err)

    返回对应给定错误代码的错误信息。
    泄漏一些字节的内存，当你以未知的错误代码调用它时。

.. c:function:: char* uv_strerror_r(int err, char* buf, size_t buflen)

    返回对应给定错误代码的错误信息。
    以零结尾的信息存储在用户提供的缓冲区 `buf` 里，不超过 `buflen` 字节。

    .. versionadded:: 1.22.0

.. c:function:: const char* uv_err_name(int err)

    返回对应给定错误代码的错误名。
    泄漏一些字节的内存，当你以未知的错误代码调用它时。

.. c:function:: char* uv_err_name_r(int err, char* buf, size_t buflen)

    返回对应给定错误代码的错误名。
    以零结尾的名称存储在用户提供的缓冲区 `buf` 里，不超过 `buflen` 字节。

    .. versionadded:: 1.22.0

.. c:function:: int uv_translate_sys_error(int sys_errno)

   返回等同于给定平台相关错误代码的libuv错误代码：
   POSIX 错误代码在 Unix 上（存储于 `errno` ），
   和Win32错误代码在Windows上（ `GetLastError()` 或 `WSAGetLastError()` 返回的）。

   如果 `sys_errno` 已经是一个libuv错误，则直接返回。

   .. versionchanged:: 1.10.0 function declared public.
