### 命名规范

[TOC]

#### 宏定义命名规范

- $AsXXXX 
- $UseXXX
- 不能有 \$RegisterXXX 和 \$ConstructXXX

#### 类定义命名规范

- XXXInterface

- XXXWare

    - IOrmTableWare
    - IOrmViewWare 等一些不再用 Interface 标记的内容
    - IOrmDialectWare

- XXXManage

    - 这个作为基类和静态类,存储一些编译运行过程中存储的东西

- XXXResponseInterface

- XXXPreProcessor

    - 预编译选项

- XXXUnit

    - 可以嵌入其他类中做辅助的子类.

    - // TODO: 这里需要思考是叫 interface 还是叫 Unit, 按照道理是 interface, 但一般实现都是 Unit
        // 思考出来了， 还是按照 底层 Unit 再封装上面一层 interface.
        // 用户自然也可以使用 Unit, 不过 Api 提供的是 interface. 两者的作用不一样

        - > 关于 ArgumentParserInterface 和 ArgumentParserUnit 的思考

- TestXXX

    - 测试文件

- XXXTable

    - 表名

- XXXBean

    - bean

- XXXModel

    - model

- XXXController

    - controller

- XXXInterceptor

- XXXProcessor

- XXXImpl

    - pimpl 组件

- XXXHelper

    - 主类的帮助类，需要static 调用的， 但是和 Util 不太像， 和 Impl 也不太像
    - 同样的 namespace   xxxHelper 表示 也是主类的帮助 名字空间， 用于帮助主类，实现功能，优化 Api 的布局。

- XXXRaw

    - 单纯存储数据的类 可以视为 Impl 的一个子集
    - 

- XXXXImplProxy

    - 这个必须为namespace ，目的是代理 Impl 在 template 中的调用，在头文件中 隐藏 Impl.h

    