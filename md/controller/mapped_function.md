# 处理函数

处理函数是直接处理request请求的函数，该函数写在`Controller`类中，并且被 `$xxxMapping` 的宏注解所注解。其中`$xxxMapping` 请参考 [定义控制器](./define_controller.md)  和 [Url 规则](./url_mapping.md) 两个部分。这一部分专门讲说处理函数。

处理函数这里只讲说一部分，开一个头，其中另外的部分在如下两个章节描述。

- [函数返回类型](./controller_return_value.md)
- [函数参数s](./controller_request_params.md)
- [请求与响应类](./request_and_response.md)

我们先写一个处理函数:

```cpp linenums='1'
#pragma once

#include <IWebCore>
#include "UserTable.h"

class MyController : public IControllerInterface<MyController>
{
public:
    MyController() = default;

    $GetMapping(fun1, /fun1)
    QString fun1();

    $GetMapping(fun2, /fun2)
    QString fun2();

    $GetMapping(fun3, /fun3)
    IJsonResponse fun3($Param(UserTable, userTable));

    $GetMapping(fun4, /fun5)
    IJsonResponse fun4();

    $GetMapping(fun5, /fun5)
    IJsonResponse fun5();

    $GetMapping(fun6, /fun6)
    void fun6(IRequest& request, IResponse& response);
};
```

我们暂时定义了六个映射函数,fun1-fun6。他们的实现如下:

```cpp linenums='1'
#include "MyController.h"

QString MyController::fun1()
{
    return "hello world";
}

QString MyController::fun2()
{
    return "$json:[1,2,3,4,5,6]";
}

IJsonResponse MyController::fun3($Param (UserTable, userTable))
{
    return userTable_param;
}

IJsonResponse MyController::fun4()
{
    QJsonObject obj;
    obj["name"] = "yuekeyuan";
    obj["masterPiece"] = "IWebCore";
    obj["expactation"] = "world famouse";
    return obj;
}

IJsonResponse MyController::fun5()
{
    QList<UserTable> tables;
    UserTable table1;
    tables.append(table1);
    tables.append(UserTable{});

    return tables;
}

void MyController::fun6(IRequest &request, IResponse &response)
{
    Q_UNUSED(request)

    response.setStatus(IHttpStatus::OK_200);
    response.setMime(IHttpMime::TEXT_PLAIN_UTF8);
    response << "hello world";
}
```

希望写这多的代码读者不要觉得厌烦。我们用这几个函数展示以下如何接收，处理，并返回一个request。中间可能会让读者翻看这里的代码。

## 返回值和返回值类型

在上面的代码展示中，我们看到了返回值类型。`QString`, `IJsonResponse`, 和 `void`。function允许的返回值类型并不止这么多，下面详细列出了返回值类型， 这些在[函数返回类型](./controller_return_value.md) 还会再提到。

- 基础类型: void， int
- Qt内置类型， QString, QJsonValue, QJsonArray, QJsonObject, QJsonValue, QByteArray。
- 内置自定义类型, IByteArrayResponse, IHtmlResponse, IJsonResponse, IPlainTextResponse, IRedirectResponse, IStatusCodeResponse.

当然，开发者也可以定义自己的返回值类型， 比如，定义一个 `IFileResponse`, 返回一个文件等。

`IWebCore`这里对前两种类型做了特殊的解释和规定

- 当返回值为 void时，他的参数当中必须含有` IResponse&` 的参数类型，目的是为了使函数能够向外有输出。 例如 `fun6`。如果没有这个类型的参数，程序在运行初期检查的时候会主动报错并给出提示信息。
- 当返回值 为 int 时， 这个int 值是指状态码，详见[Http状态码](../biscuit/http_status_code.md)。当返回值没有在代码的状态码中定义的， 这个状态值将被转换成 `1024 UNKNOWN`
- 当 类型是 QJsonXXX 系列的时, 会被转换成 `IJsonResponse` 类型。
- QByteArray 会被转换成 `IByteArrayResponse`类型进行返回。
- 当





