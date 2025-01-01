# 定义控制器

## Spring 的案例

在 web 开发的过程中，最重要的工作之一就是将一个请求映射到一个相关的处理函数当中，使用该函数处理http 请求，并返回相应的内容，我们一般把封装处理函数的类叫做控制器，意思就是控制相应的运行。

在java或者python 语言中可以使用很简洁的语言就可以完成这样的工作，并且文件中间没有耦合，降低文件的相关性，减低开发的工作量。下面是spring官方提供的代码：

```java linenums='1'
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}

	@GetMapping("/hello")
	public String hello(@RequestParam(value = "name", defaultValue = "World") 	String name) {
		return String.format("Hello %s!", name);
	}
}
```

我们可以看到在17-20行，简简单单的4行代码定义一个 处理 `/hello` url地址 的函数 `hello`它传入一个 request 行的参数 param,并返回相应的字符串。

## “$” 符号 与 宏注解

spring 的 `url-函数映射`如此简单清晰,以至于后来者一直在模仿。我在思考时也是如此。可惜的是在`c++` 当中并不支持注解，不支持`@`符号，也不支持反射。

但是我在实际操作的时候，发现`c++` 对于 `$` 这个符号并没有特殊的限制。于是我想使用 `$` 这个符号做一个类似 `@GetMapping` 这样清晰的标记，于是就有了 `$GetMapping` 这样的标记，这样写在`c++` 当中可以很清楚的把这些特殊标记符和功能代码分开。`$GetMapping` 内部实现是基于`宏`实现的（后面会细讲），那我简单的把这种由宏定义的注解称之为 `宏注解`具体内容请查看 [宏注解](../tips/macro_annotation.md) 。

在中文`c++`圈子中有一句话，`模板加宏，法力无穷`，可见宏在c++中的地位。当然仅仅有宏是不够的，还得需要另外两样东西，`反射`和`SIR`（静态初始化注入）。这两个内容和本章节的关系不是太密切，可以查看如下内容

- [Qt 反射与 Q_GADGET](../tips/reflection_and_q_gadget.md)
- [静态初始化注册](../tips/static_initialize_register.md)

## 定义 Controller

在上面的准备之后，我们也可以写出类似于`Spring`这样基于注解的代码。

在 [编写一个网络服务器](../guide/start_a_web_server.md)当中已经给出了一份代码，其中的controller 代码，重新复制在下面以作详细的讲解：

```cpp linenums='1'
// MyController.h
#pragma once

#include <IWebCore>

class MyController : public IControllerInterface<MyController>
{
    Q_GADGET
    $AsController(MyController)
public:
    MyController() = default;

    $GetMapping(index, /)
    QString index();
};


// MyController.cpp
#include "MyController.h"

QString MyController::index()
{
    return "hello world";
}
```

为了方便起见，我把声明和实现的代码放置在一起，实际上他们是分开的。

### 头文件包含

在代码第二行是防止头文件被多次包含， `#pragma once` 这个是 msvc式的声明，与之相对的还有gcc式的声明：

```cpp
#ifndef __MYCONTROLLER_H__
#define __MYCONTROLLER_H__
 	... ... // 声明、定义语句
#endif
```

两者没有明显的区别，但是如果定义了同名类，第二种方式有可能漏掉包含。而且 由于我们使用的c++版本比较高级，所以第一种在编译器支持上是没有问题的。

### IWebCore 头文件

在代码第4行，发现我们只包含了一个头文件, `IWebCore` 这个文件定义在 `src` 文件夹当中: 内容如下：

``` cpp linenums='1'
#pragma once

#include "assertion/assertion"
#include "base/base"
#include "bean/bean"
#include "log/log"
#include "configuration/configuration"
#include "controller/controller"
#include "common/common"
#include "server/IHttpServer.h"
#include "process/process"
#include "orm/orm"
#include "test/test"

$PackageWebCoreUsing
```

`IWebCore` 包含了所有有必要导出的头文件，包含这一个头文件之后，可以不用包含其他文件，当然，如果开发者觉得包含的头文件过多，编译起来有点困难，也可以把这个头文件设置成 预编译头文件:

```properties
PRECOMPILED_HEADER = IWebCore
```

当然你也可以不使用这个头文件。如果你熟悉代码，也可以包含独立的头文件,成为以下的格式：

```cpp linenums='1'
#include <controller/IControllerManage.h>
#include <controller/IControllerInterface.h>
#include <controller/pp/IControllerPreProcessor.h>
#include <controller/pp/IGetMappingPreProcessor.h>
#include <configuration/IConfigurationManage.h>
using namespace IWebCore;
```

特别要提到的一点就是 `IWebCore`定义第15行`$PackageWebCoreUsing`  这是一个宏。 展开后就是上面代码的第6行

```c++
using namespace IWebCore;
```

实际上整个 IWebCore代码 都是 在`IWebCore` 名称空间下面的。所以，如果当你定义的类或者函数与IWebCore 的有冲突，可以为相应的名称加上 名字空间`IWebCore`

### IControllerInterface

在代码第6行，我们声明了一个类名称 `MyController`, 并且以 `public`的形式继承`IControllerInterface`模板基类， 模板参数就是我们定义的controller 名称。

#### 定义

模板基类对于新手有点绕，不明白直接用也没有问题。

IControllerInterface的定义如下:

```cpp linenums='1'
template<typename T, bool enabled = true>
class IControllerInterface : public IControllerTaskUnit<T, enabled>
{
public:
    IControllerInterface() = default;
    virtual ~IControllerInterface() = default;
    virtual void task() final;

protected:
    virtual void registerControllerFun(void *handler, const QMap<QString, QString> &clsInfo,
                                const QVector<QMetaMethod> &methods) = 0;
};

template<typename T, bool enabled>
void IControllerInterface<T, enabled>::task(){
    auto inst = T::instance();
    auto clsInfo = IMetaUtil::getMetaClassInfoMap(T::staticMetaObject);
    auto methods = IMetaUtil::getMetaMethods(T::staticMetaObject);
    registerControllerFun(inst, clsInfo, methods);
}
```

这个东西简单来说就是将 controller 类 T 通过 task 函数注册到一个地方，IControllerManage 这样的一个池子里,等待请求来临时可以查找 IControllerManage 池子里面匹配相应url的函数，再调用这个函数。具体如何注册，可以查看[静态初始化注册](../tips/static_initialize_register.md) 以及 [Task](../task/task.md)

#### 停用模板

还有一点需要注意的是： IControllerInterface 基类有两个模板参数：

```cpp
template<typename T, bool enabled = true>
class IControllerInterface : public IControllerTaskUnit<T, enabled>
```

第一个 `typename T` 是要注册的 controller 类， 而第二个 `bool enabled = true` 这个bool 类型的默认模板参数的作用是是否向 IControllerManage注册 这个controller，并使这个controller 生效。

所以，当你已经定义了一个 controller,但是暂时不需要这个controller, 但是又不能删除，可以将第二个模板参数设置为 false，如下：

``` cpp linenums='1'
// MyController.h
#pragma once

#include <IWebCore>

class MyController : public IControllerInterface<MyController, false>
{
    Q_GADGET
    $AsController(MyController)
public:
    MyController() = default;

    $GetMapping(index, /)
    QString index();
};
```

这样，MyController就不会生效。

#### 关于 IxxxInterface 命名规则的问题

在这里插入一个话题，就是基类的命名问题。

在`IWebCore` 的编写过程中，有很多不同功能的类，宏注解，函数，命名空间等等内容，不同的类型的内容有自己独特的前缀或后缀方式，详情请参考[命名规则](../others/name_regulation.md)

对于类型接口，接口的名称都是`IxxxInterface` 这种格式的，比如这里的`IControllerInterface` 以后会接触到的`IOrmTableInterface` , `IConfigurationInterface` 等等类型。所以在编写代码的时候，IDE 会给出好多中提示，我们一般是选择以 Interface 结尾的名称作为基类。

### Q_GADGET

在`MyController.h` 第8行有 `Q_GADGET` 声明。 关于 `Q_GADGET` 的内容可以查看[Qt 反射与 Q_GADGET](../tips/reflection_and_q_gadget.md),这里简要作为说明。

Q_GADGET 有两点作用

- 第一,是Qt实现反射的基础。

    有熟悉Qt 的开发者接触最多的应该是 Q_OBJECT。 而Q_GADGET 是一个轻量级的Q_OBJECT，它包含Qt反射所需要的最小集合。

    ```cpp linenums='1'
    #define Q_GADGET \
    public: \
        static const QMetaObject staticMetaObject; \
        void qt_check_for_QGADGET_macro(); \
        typedef void QtGadgetHelper; \
    private: \
        QT_WARNING_PUSH \
        Q_OBJECT_NO_ATTRIBUTES_WARNING \
        Q_DECL_HIDDEN_STATIC_METACALL static void qt_static_metacall(QObject *, QMetaObject::Call, int, void **); \
        QT_WARNING_POP \
        QT_ANNOTATE_CLASS(qt_qgadget, "")
    ```

    可以看见它里面包含了一个 staticMetaObject对象。这个对象里面能够反射出类当中被标记出来的 classinfo,method,property,enum, className 等信息，使用这些信息就可以操纵这个类的对象。

- 第二， Qt 当中有 moc 编译器，这个moc 编译器的作用就是根据当前被标记有Q_GADGET 或者Q_OBJECT的类声明，生成一个带有类信息的moc_xxx.cpp文件。供 staticMetaObject 来解析。

所以这里我们使用 Q_GADGET 这个宏很明显的作用就是要使 MyController 这个类能够被反射，其内部结构能够被掌握。比如说，我们在这里使用的 `$GetMapping` 所标记的 url 和函数 `index`之间的映射信息；函数`index` 的信息等。

总而言之，我们这里使用Q_GADGET的作用使这个类能够被反射到。

### $AsController

在`MyController.h` 中间第9 行出现`$AsController(MyController)`。这同样使一个宏注解，和最上面的`@RestController` 大概意思是相同的，都是用来标记这是一个controller。不过 `$AsController` 需要至少一个参数。

#### $AsController 的两种形式。

`AsController` 宏注解有两种重载形式，分别为：

- `#define $AsController_1(klassName)`

    只有一个参数的 $AsController 传入的是类的名称。

    klassName 名称的传入目的是在`宏注解`当中实现一些`controller`当中共同的函数，比如说定义单例，重载基类的相关函数等。

- `#define $AsController_2(klassName, path)`

    有两个参数的 $AsController 第一个参数传入的是类的名称，第二个参数则是这个 controller 所有的url 映射的前缀。

    比如说当path 的参数是 `abc`, 那么在这个函数中间映射的所有url 的前面都要加上 `abc`。比如说当`$GetMapping` 的url 参数是 `hello`, 那么其实际映射的url 就是 `/abc/hello`

这里使用了 `宏重载` 的技术，有不了解而想了解的开发者可以自行学习。

同样，`$AsXXX` 也是一种命名规范，请参考 [命名规范](../others/name_regulation.md)

### 声明映射函数

在`MyController.h` 的13-14行声明如下

```cpp linenums='1'
    $GetMapping(index, /)
    QString index();
```

这里的作用是声明一个index函数并将 index 函数映射到 `/` 的url 上面， index 函数返回一个字符串。

#### $XxxMapping

$GetMapping 是映射到 `GET` 请求当中。除此以外还有另外集中 mapping 宏注解

- $PostMapping 用于映射POST 请求
- $PutMapping 用于映射PUT 请求
- $DeleteMapping 用于映射DELETE 请求
- PatchMapping 用于映射PATCH请求

除这些常规请求以外，还有一种 `OPTION` 请求和 `HEAD`请求。

根据 rfc, `HEAD` 请求会被解析成`GET` 请求，返回首行和headers,但是不反回body 内容。所以这种请求不需要单独使用宏注解

`OPTIONS` 请求会返回当前url所支持的请求类型，同样不需要宏注解。

另外，在另外一种Controller 类型, `IStatusController` 类型当中，有 $StatusMapping注解，这里只列不表。

这些 $xxxMapping至少需要一个参数。我们以 `$GetMapping` 为例，删减了部分代码，其定义如下

```cpp linenums='1'
#define $GetMapping_1(funName)  \
    $GetMappingDeclare(funName) \
    Q_INVOKABLE

#define $GetMapping_2(funName, url1)  \
    $GetMappingDeclare(funName, url1) \
    Q_INVOKABLE

#define $GetMapping_3(funName, url1, url2)  \
    $GetMappingDeclare(funName, url1, url2) \
    Q_INVOKABLE

#define $GetMapping_4(funName, url1, url2, url3)  \
    $GetMappingDeclare(funName, url1, url2, url3) \
    Q_INVOKABLE

#define $GetMapping_5(funName, url1, url2, url3, url4)  \
    $GetMappingDeclare(funName, url1, url2, url3, url4) \
    Q_INVOKABLE

#define $GetMapping_6(funName, url1, url2, url3, url4, url5)  \
    $GetMappingDeclare(funName, url1, url2, url3, url4, url5) \
    Q_INVOKABLE

#define $GetMapping_7(funName, url1, url2, url3, url4, url5, url6)  \
    $GetMappingDeclare(funName, url1, url2, url3, url4, url5, url6) \
    Q_INVOKABLE

#define $GetMapping_8(funName, url1, url2, url3, url4, url5, url6, url7)  \
    $GetMappingDeclare(funName, url1, url2, url3, url4, url5, url6, url7) \
    Q_INVOKABLE

#define $GetMapping_9(funName, url1, url2, url3, url4, url5, url6, url7, url8)  \
    $GetMappingDeclare(funName, url1, url2, url3, url4, url5, url6, url7, url8) \
    Q_INVOKABLE

#define $GetMapping_(N) $GetMapping_##N
#define $GetMapping_EVAL(N) $GetMapping_(N)
#define $GetMapping(...) PP_EXPAND( $GetMapping_EVAL(PP_EXPAND( PP_NARG(__VA_ARGS__) ))(__VA_ARGS__) )


```

可以看见第一个参数永远时 对应的函数名称， 否则就不正确。

当只有一个参数时，对应的函数映射到当前controller 的根目录。也就是`/`目录。当controller 没有指定目录时，

第2个到第9个参数是 不同的url,最多一共可以有8个不同的url映射到同一个函数当中。

#### url pattern

写在 `$XxxMapping` 中的url 不仅是静态的，而且是可以动态映射的。比如当 url 是`/<name:string>/<age:int>`这种格式时，它表示url 的第一个参数时`string` 类型的，而映射系统会保留这个参数以 name 的形式传给函数。第二个参数 `<age:int>`则表示这个参数必须能够转换成 int 类型，如果转换不成，这个url就不会被匹配。第二个参数会被赋值到 name, 传给 函数使用。

关于具体的url如何使用，请参考 [Url 定义](./url_mapping.md)

#### Q_INVOKABLE

在 Qt 的元对象系统当中，如何让一个内部的函数被对象系统了解，并且能够做出反射？ 答案就是在这个函数面前添加 `Q_INVOKABLE`。

当函数添加`Q_INVOKABLE`之后，`moc`工具会记录当前函数的所有信息，生成在 moc_xxx.cpp 文件当中。记录的信息包括但不限于

- 函数名称，函数签名
- 函数返回值和返回值类型和 对象返回值类型的 MetaType
- 函数各个参数类型，参数名称和参数类型的 MetaType
- 函数的引用本身，是函数可以通过 invokeOnGadget 被调用。

这个也是 `Qt` signal/slots的基础之一。不过在实现 `IWebCore` 时，并没有使用 signal/slots，而是使用了原始的反射系统，signal/slots 使用起来反而不方便。

所以我们可以看见，所有的 `$XxxMapping`宏注解后面都有一个 `Q_INVOKABLE` 跟着。这个也就意味着宏注解后面得跟上一个函数，否则编译器在宏处理这个阶段就会报错，同样，`$XxxMapping` 不能堆叠使用。这个也会报错。

凡是都有例外，我们也可以使用 `$MappingDeclare` 这种形式来进行定义 。比如我们想让一个函数有多个 能够匹配多个类型的调用，我们可以写代码如下

```cpp linenums='1'
    $GetMappingDeclare(index, /)
    $PostMappingDeclare(index, /)
    $PutMappingDeclare(index, /)
    $DeleteMappingDeclare(index, /)
    $PatchMappingDeclare(index, /)
    Q_INVOKABLE IJsonResponse index();
```

这样，所有的请求都 mapping 到  `index` 函数。

#### 映射函数

映射函数是指被 url 映射的函数。它有特定的要求，比如说不能是重载函数，函数参数不能有默认值等等的要求。这里不做详细的讲解，具体请看 [映射函数](./mapped_function.md)。这一章节。

在 `MyController` 案例中，函数返回值是 `QString` 类型。在解析的过程中， QString 类型的一般将被解析成 `plain/text` 这样的 `mime` 类型， 表示这是一个普通的字符串类型。

剧透一点：如果 返回的字符串 是 "$json:[1,2,3,4]", 那么返回的类型是 json 类型的。返回json类型的数据也可以使用 `IJsonResponse` 这样的类型，返回 QJsonObject,等， 也可以使用 "[1,2,3]"_json  这种格式返回。 

## 总结

关于Controller的详细讲解到此为止。之后的内容，如 `Q_GADGET`这种东西会默认开发者已经了解。如果有不清楚的可以回头看看这一章节，或者看看 [Qt Documentation | Home](https://doc.qt.io/)。当然把它当作装饰性的东西也并不影响开发。
