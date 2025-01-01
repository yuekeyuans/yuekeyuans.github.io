# IWebCore 简介

<pre float="center">
 _____  _    _        _      _____
|_   _|| |  | |      | |    /  __ \
  | |  | |  | |  ___ | |__  | /  \/  ___   _ __  ___
  | |  | |/\| | / _ \| '_ \ | |     / _ \ | '__|/ _ \
 _| |_ \  /\  /|  __/| |_) || \__/\| (_) || |  |  __/
 \___/  \/  \/  \___||_.__/  \____/ \___/ |_|   \___|
</pre>


`IWebCore` 是基于`c++11`的服务器开发框架，它使用户更加专注于开发本身，降低开发难度，提高开发效率。

基于`Qt`反射技术，`IWebCore`提供了`controller`自动注册，类对象的序列化和反序列化，数据库自动注册和默认操作等一一系列方便你有用的技术支持。 

---

## 地址

- github地址 [yuekeyuans/IWebCore (github.com)](https://github.com/yuekeyuans/IWebCore)
- 文档地址：[IWebCore文档](http://www.iwebcore.com/)

## 优点

### c++11 / Qt 平台支持。

- **大量的c++11库和 Qt 库支持**

- **成熟的编程语言**

    目前主流的c++开发是基于c++11， 而Qt开发也是 c++ 开发的主流之一。

- **跨平台开发和运行**

    软件目前支持在所有Qt能够运行的环境上编译开发，实现软件的跨平台支持。所以开发者可以在windows 上面进行快速开发编译，而在完成开发之后部署到linux主机上面再次编译并运行。 

### 高效

- **降低 内存， cpu 等资源的消耗**

    c++ 拥有很高的运行效率，对比与其他的主流网络框架，它能够节省更多内存的使用，cpu使用，进而带动服务器数量的减少和能源消耗的减少，降低碳排放。(参见文章: [Why C++ ? 王者归来 | 酷 壳 - CoolShell](https://coolshell.cn/articles/6548.html) )

- 软件运行速度快。
- 秒速启动运行。

### 快速开发。

- 易于上手

    c++ 开发很多人觉得困难是因为c++本身带来的复杂性。`IWebCore` 对于新手非常友好，开发人员初期并不需要了解C++复杂的特性也可以 使用controller，并进行数据库的增删改查等操作。

- 类似Spring和 FastApi 等主流web开发框架

    降低了开发者的学习难度，使用了 `宏注解` 如 `$Controller` 等开发者熟悉的方式。

### Controller

- controller 自动注册
- url灵活配置，动态url
- 函数参数从请求中动态解析和注入绑定
- 灵活的返回方式，可以返回 text,json, xml, status等内容。
- 严格的参数检查。

### 强大的orm 支持

- 内置 database，table，view， model支持
- table sql 自动生成
- json 原生支持
- 内置各种Util转换，可以将 bean 或 bean 的集合转换成 json/string 类型。

### 内置 http server

- 支持 `GET`， `POST`，`PUT`，`HEAD` ,`DELETE`, `PATCH`, `OPTION`等方法。
- HttpMime, HttpStatus支持。
- **TODO:** 在将来的版本中优化tcp请求模型，使用`epoll`和 `IOCP` 等方式。

### 方便的配置方式。

- 支持 System和 Application两种级别的配置
- 提供包括 代码，`json`, `yaml`,`宏注解` 等多种配置方式

### 运行期错误检查。

- 友好的运行前检查，

    在实例运行之前，会对软件的逻辑进行详细的检查，在有问题的地方生成有好的 warning 或者 fatal 信息

---

## 代码样式

下面提供一个最简单的项目代码，只有两个文件`main.cpp` 和 `MyFirstController.h`

```cpp linenums='1'
// main.cpp
#include <IWebCore>

int main(int argc, char *argv[])
{
    IWebApplication app(argc, argv);
    IHttpServer server;
    server.listen();
    return app.exec();
} 
```

``` cpp linenums='1'
// MyFirstController.h
#pragma once

#include <IWebCore>

class MyFirstController : public IControllerInterface<MyFirstController>
{
    Q_GADGET
    $AsController(MyFirstController)
public:
    $GetMapping(index, /)
    QString index()
    {
        return "hello world";
    }
};
```

当运行项目的时候，一个webserver 已经创建并运行完成。

我们可以访问 http://127.0.0.1:8088, 并看到有 hello world 返回。

![image-20220226193138143](./img/index/image-20220226193138143.png)



第一个 `IWebCore`应用已经开发出来了，想了解更多，可以查看

- [安装 IWebCore](./guide/install_iwebcore.md)
- [编写一个网络服务器](./guide/start_a_web_server.md)
- [使用 orm](./guide/using_orm.md)





 
