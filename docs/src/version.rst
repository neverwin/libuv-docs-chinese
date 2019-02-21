
.. _version:

用于版本检测的宏和函数
=====================================

始于版本1.0.0，libuv遵循 `semantic versioning`_
模式。这意味着新的API能在一个主版本发布的任何时期引入。
在这个部分你将会了解所有允许你有条件地编写或编译代码的宏和函数，
用于跟多个libuv版本打交道。

.. _semantic versioning: http://semver.org


宏
------

.. c:macro:: UV_VERSION_MAJOR

    libuv版本中的主编号。

.. c:macro:: UV_VERSION_MINOR

    libuv版本中的次编号。

.. c:macro:: UV_VERSION_PATCH

    libuv版本中的补丁编号。

.. c:macro:: UV_VERSION_IS_RELEASE

    设置成 1 代表着一个libuv的发布版，0 表示开发版快照。

.. c:macro:: UV_VERSION_SUFFIX

    libuv版本后缀。特定的开发发布版本例如 Release Candidates
    可能有个后缀比如 "rc" 。

.. c:macro:: UV_VERSION_HEX

    返回libuv的版本信息打包进单个整数中。每个部分占8位，
    补丁版本在最低8位。
    比方说，libuv 1.2.3 就是 0x010203 。

    .. versionadded:: 1.7.0


函数
---------

.. c:function:: unsigned int uv_version(void)

    返回 :c:macro:`UV_VERSION_HEX`.

.. c:function:: const char* uv_version_string(void)

    以字符串形式返回libuv版本号。
    对于非发布版本，版本后缀也包括在内。
