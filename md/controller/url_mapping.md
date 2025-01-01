# Url 规则

## 一个案例

我们先写一个已经写过的案例

```cpp linenums='1'
#pragma once

#include <IWebCore>

class MyController : public IControllerInterface<MyController>
{
    Q_GADGET
    $AsController(MyController)
public:
    MyController() = default;

    $IgnoreParamWarn(hello)
    $GetMapping(hello, /<name>)
    QString hello(QString name);
};
```

这个案例在 [编写一个网络服务器](../guide/start_a_web_server.md) 当中使用过，意思是 将 `/<name>` 映射到`hello` 函数上面。我们重新整理代码如下:

```cpp linenums='1'
#pragma once

#include <IWebCore>

class MyController : public IControllerInterface<MyController>
{
    Q_GADGET
    $AsController(MyController)
public:
    MyController() = default;

    $GetMapping(hello, /<name>)
    QString hello($Url(QString, name));
};
```

看见这里的代码变化是 删除了 `$IgnoreParamWarn(hello)`， 并且将参数`QString name` 改成 `$Url(QString, name)` 的形式。

`$Url(QString, name)` 是告诉 controller 要从 `Url` 这个里面去查找 name 参数的值，并且参数值的类型转换成 `QString` 类型。 关于 `$Url`， 请参考 [函数参数位置](./controller_request_params.md)

`$IgnoreParamWarn(hello)` 的出现是因为 `hello` 中查找 `name` 不知道从哪里查找，所以它会从 request 的每一项参数中间查找。但是这里会有一个问题，就是 request 当中可能会有多个叫做 name 的参数，匹配的不一定是开发者想要的那个，所以在第一处就报一个错误，但是使用`$IgnoreParamWarn(hello)` 则可以消除这个警报。

在第二处的代码的地方，我们直接指定了name 的位置是 `Url` 里面，这时就没有歧义，也就不用屏蔽警报。

__在这个地方，我想说的是所有在 url 中拦截出来的数据可以在 函数中以参数的形式被使用。__当然，还有其他使用方式，参见  [函数参数位置](./controller_request_params.md)。 下面我们来讲解几种动态映射url 的参数的方式。

##  <name\> 的形式

这种形式，结果不名自喻。意思就是不管是什么样的形式，都给拦截掉，并且把拦截的url 片段以 name 的名称存储起来。

这里有一个省略式，就是中间的 `name` 可以不写， 直接写成  `<> ` 的形式。注意中间不要有空格之类的东西。这个表示这个url 片段可以为任何非空有效的 url 片段。

举例如下:

```cpp linenums='1'
class MyController : public IControllerInterface<MyController>
{
    Q_GADGET
    $AsController(MyController)
public:
    MyController() = default;

    $GetMapping(hello, /<name>/<age>)
    QString hello($Url(QString, name), $Url(int, age));
    
    $GetMapping(world, /<>/<>/<lastName>)
    void world(IResponse& response, QString lastName);
};
```

根据8-9 行的定义，所有有两个片段的 url 请求 都会映射到 `hello` 函数，

根据11-12 行的定义， 所有有三个片段 的url 请求都会映射到 world 的函数。

细心的开发者可能会发现几个问题：

- 比如说 `/yue/keyuan` , 这个请求会映射到 hello 函数， 那么如果我们定义了映射 `/yue/keyuan` 的其他函数请求怎么办？

    > 答案式这次查找会失败，在 `IControllerManage` 内部找到多于一个的函数映射，那么这个请求会失败，返回 `404` 错误

- 类似 url `/yue/keyuan`如果映射成功，但是age的信息不匹配怎么办?

    > 在解析的过程中 `keyuan` 会以QByteArray 的形式存储在 map 中， 生成的map 为 {"name":"yue", "age":"keyuan"}。
    >
    > 在找到函数时，会通过反射生成函数`hello`内部的两个参数，分别为 `QString*` 和 `int*` 。之后会把相应的参数转型成当前的类型， 把"yue" 转换成 `QString` 类型，把 "keyuan" 转换成 `int` 类型。
    >
    > 很显然，"keyuan"转换不成 `int` 类型，所以在这里虽然映射成功，但是由于解析不成功，将会返回 `400 Bad Request`

由于这种映射类型会有漏洞，所以就有如下两种映射类型:

## <name:type\>  的形式

这里的意思也是很清晰，`name`表示片段将要赋予的名称， `type` 表示对片段的约束。

这里也可以有省略式，

- 既可以省略` name`, 成为 `<:type>` 这种形式，表示只有约束，不用采集 url里的参数。
- 也可以省略 `type`, 成为 `<name:>` 这种形式，表示采集参数，但不加约束。这个和 `<name>` 这种形式是没有区别的。
- 也可以 `name` 和 `type` 两者都省略， 成为 `<:>` 这种形式。这个和 `<>` 这种形式没有区别。

这里的 type 约束分为两种: 一种是内置类型，一种是用户自定义注册类型。

### 内置类型

目前内置的类型关键字如下：

- number系列 ：short, ushort, int, uint, long, ulong, longlong, ulonglong, float, double
- 时间类型: data, QDate, time, QTime, datetime, QDateTime
- 字符串类型: string, QString
- 其他类型: uuid, base64

这里只要以上面定义的类型名称将type 替换掉即可。

注意，这里的定义的类型可以不和 参数的类型一致，但是，建议和参数类型一致，否则不排除出问题的可能。比如说我在url中使用 `int` 类型作为约束，但是参数中使用 `short`, 那么就会出现转换错误的问题，而导致请求无法继续。

### 自定义类型及注册。

除了这些内置类型之外，用户也可以自定义类型，用在约束url 片段。

 自定义类型的注册也分为两种，一种是注册函数，一种则是注册正则表达式。这两个注册函数定义在 IControllerManage 当中， 如下：

```cpp linenums='1'
class IControllerManage
{
    ... ...
    using ValidatorFun = bool (*)(const QString&);
    ... ... 
    static void registerPathValidator(const QString& name, const QString& regexp); 
    static void registerPathValidator(const QString& name, ValidatorFun fun);
	... ...
}
```

注册 validator 需要发生在对 Controller 中的函数解析之前。而Controller中的函数实际解析则是发生在 ITaskManage的 静态 run 函数之内执行的。所以我们在此之前解析就可以。

为了方便起见，`IWebCore` 定义了 `IControllerPathValidatorInterface`。这是一个很长的类名，但是不常用，所以能够接受。当然，你也可以实现自己的注册方式，参考 [SIR](../tips/static_initialize_register.md)。

我们就实现一个自定义的 注册器, 如下:

```cpp linenums='1'
// FiveCharaterUrlValidator.h
#pragma once

#include <IWebCore>

class FiveCharaterUrlValidator : public IControllerPathValidatorInterface<FiveCharaterUrlValidator>
{
    $UseInstance(FiveCharaterUrlValidator)
public:
    FiveCharaterUrlValidator() = default;
    virtual void task() override;
};


// FiveCharaterUrlValidator.cpp
#include "FiveCharaterUrlValidator.h"

void FiveCharaterUrlValidator::task()
{
    auto fiveCharatorValidator = [](const QString& piece) -> bool{
        return piece.length() == 5;
    };
    registerValidator("5c", fiveCharatorValidator);
}
```

如上所示我们定义了一个叫做 `5c` 的规则，意思是字符数量只能是5个。之后我们就可以在代码中使用如 `<name:5c>` 这种类型的规则了。

上面演示的是注册一个函数，如果是注册一个正则式，也可以直接调用如下:

```cpp
registerValidator("5c", "[.]{5}");
```

这样同样能达到规则约束的目的。

## <reg:name:exp\>   的形式

这一次有三个参数，两个冒号。

第一个参数就是`reg` (其实什么都可以，这个现在没有强制规定名称。但是考虑到扩展，所以用户就直接使用`reg` 吧)。第二个参数是`name` 这个和前面一样。

主要是第三个参数，第三个参数是 正则表达式。如果正则式写错了，运行期检查的时候会报错的，所有上面的 `5c`匹配其实也可以写成 "<reg:name:[.]{5c}>"

第三种形式也有省略式。

- `<:name:exp>` 省略前面的`reg`
- `<::exp>` 省略前面的reg 和 name
- `<:name:>` 这种形式 等于 `<name>`
- <::> 嗯，所有的都省略了，其实就等于 `<>`  这种格式了。
