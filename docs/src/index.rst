
欢迎来到libuv中文文档
==================================

摘要
--------

libuv是一个强调异步I/O的多平台支持库。 开发它
主要是用于 `Node.js`_ ，但它也被用在 `Luvit`_ 、
`Julia`_ 、 `pyuv`_ 和 `其他项目`_ 。

.. note::
    如果你在这份翻译中发现问题你可以发送
    `pull requests <https://github.com/neverwin/libuv-docs-chinese>`_ 来帮助！

.. _Node.js: http://nodejs.org
.. _Luvit: http://luvit.io
.. _Julia: http://julialang.org
.. _pyuv: https://github.com/saghul/pyuv
.. _其他项目: https://github.com/libuv/libuv/wiki/Projects-that-use-libuv


特点
--------

* 全功能的事件循环基于epoll、kqueue、IOCP、event ports
* 异步的TCP和UDP套接字
* 异步的DNS解析
* 异步的文件和文件系统操作
* 文件系统事件
* ANSI转义代码控制的TTY
* IPC包括套接字共享，使用Unix域套接字或有名管道（Windows）
* 子进程
* 线程池
* 信号处理
* 高分辨率时钟
* 线程和同步原语


文档
-------------

.. toctree::
   :maxdepth: 1

   design
   api
   guide
   upgrading


下载
---------

libuv 可在 `这儿 <http://dist.libuv.org/dist/>`_ 下载。


安装
------------

安装步骤详见 `README <https://github.com/libuv/libuv/blob/master/README.md>`_ 。

