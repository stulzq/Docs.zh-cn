---
title: ASP.NET Core 中的过滤器
author: ardalis
description: 了解过滤器的工作原理以及如何在 ASP.NET Core MVC 中使用它们。
ms.author: riande
ms.custom: mvc
ms.date: 10/15/2018
uid: mvc/controllers/filters
ms.openlocfilehash: d25976af860ed8733c0548a4102e0095969b2ba6
ms.sourcegitcommit: 9bb58d7c8dad4bbd03419bcc183d027667fefa20
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2018
ms.locfileid: "52861844"
---
# <a name="filters-in-aspnet-core"></a>ASP.NET Core 中的过滤器

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)、[Tom Dykstra](https://github.com/tdykstra/) 和 [Steve Smith](https://ardalis.com/)

ASP.NET Core MVC 中的过滤器允许在请求处理管道中的特定阶段之前或之后运行代码。

> [!IMPORTANT]
> 本主题不适用于 Razor 页面。 ASP.NET Core 2.1 及更高版本支持适用于 Razor 页面的 [IPageFilter](/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter?view=aspnetcore-2.0) 和 [IAsyncPageFilter](/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncpagefilter?view=aspnetcore-2.0)。 有关详细信息，请参阅 [Razor 页面的筛选方法](xref:razor-pages/filter)。

 内置过滤器处理任务，例如：

 * 授权（防止用户访问未获授权的资源）。
 * 确保所有请求都使用 HTTPS。
 * 响应缓存（对请求管道进行短路出路，以便返回缓存的响应）。 

可以创建自定义过滤器来处理横切关注点。 过滤器可以避免跨操作复制代码。 例如，错误处理异常过滤器可以合并错误处理。

[查看或下载 GitHub 中的示例](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/controllers/filters/sample)。

## <a name="how-do-filters-work"></a>过滤器的工作原理

过滤器在 *MVC 操作调用管道*（有时称为*过滤器管道*）内运行。  过滤器管道在 MVC 选择要执行的操作之后运行。

![请求通过其他中间件、路由中间件、操作选择和 MVC 操作调用管道进行处理。 请求处理继续往回通过操作选择、路由中间件和各种其他中间件，变成发送到客户端的响应。](filters/_static/filter-pipeline-1.png)

### <a name="filter-types"></a>过滤器类型

每种过滤器类型都在过滤器管道中的不同阶段执行。

* [授权过滤器](#authorization-filters)最先运行，用于确定是否已针对当前请求为当前用户授权。 如果请求未获授权，它们可以让管道短路。 

* [资源过滤器](#resource-filters)是授权后最先处理请求的过滤器。  它们可以在过滤器管道的其余阶段运行之前以及管道的其余阶段完成之后运行代码。 出于性能方面的考虑，可以使用它们来实现缓存或以其他方式让过滤器管道短路。 它们在模型绑定之前运行，所以可以影响模型绑定。

* [操作过滤器](#action-filters)可以在调用单个操作方法之前和之后立即运行代码。 它们可用于处理传入某个操作的参数以及从该操作返回的结果。

* [异常过滤器](#exception-filters)用于在向响应正文写入任何内容之前，对未经处理的异常应用全局策略。

* [结果过滤器](#result-filters)可以在执行单个操作结果之前和之后立即运行代码。 仅当操作方法成功执行时，它们才会运行。 对于必须围绕视图或格式化程序的执行的逻辑，它们很有用。

下图展示了这些过滤器类型在过滤器管道中的交互方式。

![请求通过授权过滤器、资源过滤器、模型绑定、操作过滤器、操作执行和操作结果转换、异常过滤器、结果过滤器和结果执行进行处理。 返回时，请求仅由结果过滤器和资源过滤器进行处理，变成发送到客户端的响应。](filters/_static/filter-pipeline-2.png)

## <a name="implementation"></a>实现

过滤器通过不同的接口定义支持同步和异步实现。 

可在其管道阶段之前和之后运行代码的同步过滤器定义 On*Stage*Executing 方法和 On*Stage*Executed 方法。 例如，在调用操作方法之前调用 `OnActionExecuting`，在操作方法返回之后调用 `OnActionExecuted`。

[!code-csharp[](./filters/sample/src/FiltersSample/Filters/SampleActionFilter.cs?name=snippet1)]

异步过滤器定义单一的 On*Stage*ExecutionAsync 方法。 此方法采用 *FilterType*ExecutionDelegate 委托来执行过滤器的管道阶段。 例如，`ActionExecutionDelegate` 调用该操作方法或下一个操作过滤器，用户可以在调用它之前和之后执行代码。

[!code-csharp[](./filters/sample/src/FiltersSample/Filters/SampleAsyncActionFilter.cs?highlight=6,8-10,13)]

可以在单个类中为多个过滤器阶段实现接口。 例如，[ActionFilterAttribute](/dotnet/api/microsoft.aspnetcore.mvc.filters.actionfilterattribute?view=aspnetcore-2.0) 类实现 `IActionFilter` 和 `IResultFilter`，以及它们的异步等效接口。

> [!NOTE]
> 过滤器接口的同步和异步版本**任意**实现一个，而不是同时实现。 该框架会先查看过滤器是否实现了异步接口，如果是，则调用该接口。 如果不是，则调用同步接口的方法。 如果在一个类中同时实现了这两种接口，则仅调用异步方法。 使用抽象类时，比如 [ActionFilterAttribute](/dotnet/api/microsoft.aspnetcore.mvc.filters.actionfilterattribute?view=aspnetcore-2.0)，将为每种过滤器类型仅重写同步方法或仅重写异步方法。

### <a name="ifilterfactory"></a>IFilterFactory

[IFilterFactory](/dotnet/api/microsoft.aspnetcore.mvc.filters.ifilterfactory) 实现 [IFilterMetadata](/dotnet/api/microsoft.aspnetcore.mvc.filters.ifiltermetadata)。 因此，`IFilterFactory` 实例可在过滤器管道中的任意位置用作 `IFilterMetadata` 实例。 当该框架准备调用过滤器时，它会尝试将其转换为 `IFilterFactory`。 如果强制转换成功，则调用 [CreateInstance](/dotnet/api/microsoft.aspnetcore.mvc.filters.ifilterfactory.createinstance) 方法来创建将调用的 `IFilterMetadata` 实例。 这提供了一种很灵活的设计，因为无需在应用启动时显式设置精确的过滤器管道。

用户可以在自己的属性实现上实现 `IFilterFactory` 作为另一种创建过滤器的方法：

[!code-csharp[](./filters/sample/src/FiltersSample/Filters/AddHeaderWithFactoryAttribute.cs?name=snippet_IFilterFactory&highlight=1,4,5,6,7)]

### <a name="built-in-filter-attributes"></a>内置过滤器属性

该框架包含许多可子类化和自定义的基于属性的内置过滤器。 例如，以下结果过滤器会向响应添加标头。

<a name="add-header-attribute"></a>

[!code-csharp[](./filters/sample/src/FiltersSample/Filters/AddHeaderAttribute.cs?highlight=5,16)]

属性允许过滤器采用参数，如上面的示例所示。 可将此属性添加到控制器或操作方法，并指定 HTTP 标头的名称和值：

[!code-csharp[](./filters/sample/src/FiltersSample/Controllers/SampleController.cs?name=snippet_AddHeader&highlight=1)]

`Index` 操作的结果如下所示：响应标头显示在右下角。

![显示响应标头（包括 Author Steve Smith @ardalis）的 Microsoft Edge 开发人员工具](filters/_static/add-header.png)

多种过滤器接口具有相应属性，这些属性可用作自定义实现的基类。

过滤器属性：

* `ActionFilterAttribute`
* `ExceptionFilterAttribute`
* `ResultFilterAttribute`
* `FormatFilterAttribute`
* `ServiceFilterAttribute`
* `TypeFilterAttribute`

[本文稍后](#dependency-injection)会对 `TypeFilterAttribute` 和 `ServiceFilterAttribute` 进行介绍。

## <a name="filter-scopes-and-order-of-execution"></a>过滤器作用域和执行顺序

可以将过滤器添加到管道中的三个*作用域*之一。 可以使用属性将过滤器添加到特定的操作方法或控制器类。 或者，也可以注册所有控制器和操作的全局过滤器。 通过将过滤器添加到 `ConfigureServices` 中的 `MvcOptions.Filters` 集合，可以将其添加为全局过滤器：

[!code-csharp[](./filters/sample/src/FiltersSample/Startup.cs?name=snippet_ConfigureServices&highlight=5-8)]

### <a name="default-order-of-execution"></a>默认执行顺序

当管道的某个特定阶段有多个过滤器时，作用域可确定过滤器执行的默认顺序。  全局过滤器涵盖类过滤器，类过滤器又涵盖方法过滤器。 这种模式有时称为“俄罗斯套娃”嵌套，因为增加的每个作用域都包装在前一个作用域中，就像[套娃](https://wikipedia.org/wiki/Matryoshka_doll)一样。 通常情况下，无需显式确定排序便可获得所需的重写行为。

在这种嵌套模式下，过滤器的 *after* 代码会按照与 *before* 代码相反的顺序运行。 其序列如下所示：

* 过滤器的 *before* 代码应用于全局
  * 过滤器的 *before* 代码应用于控制器
    * 过滤器的 *before* 代码应用于操作方法
    * 过滤器的 *after* 代码应用于操作方法
  * 过滤器的 *after* 代码应用于控制器
* 过滤器的 *after* 代码应用于全局
  
下面的示例阐释了为同步操作过滤器调用过滤器方法的顺序。

| 序列 | 过滤器作用域 | 过滤器方法 |
|:--------:|:------------:|:-------------:|
| 1 | Global | `OnActionExecuting` |
| 2 | 控制器 | `OnActionExecuting` |
| 3 | 方法 | `OnActionExecuting` |
| 4 | 方法 | `OnActionExecuted` |
| 5 | 控制器 | `OnActionExecuted` |
| 6 | Global | `OnActionExecuted` |

此序列显示：

* 方法过滤器已嵌套在控制器过滤器中。
* 控制器过滤器已嵌套在全局过滤器中。 

换句话说，如果处于异步过滤器的 On*Stage*ExecutionAsync 方法内，则当代码位于堆栈上时，所有过滤器都在更严格的作用域中运行。

> [!NOTE]
> 继承自 `Controller` 基类的每个控制器都包括 `OnActionExecuting` 和 `OnActionExecuted` 方法。 这些方法包装针对某项给定操作运行的过滤器：`OnActionExecuting` 在所有过滤器之前调用，`OnActionExecuted` 在所有过滤器之后调用。

### <a name="overriding-the-default-order"></a>重写默认顺序

可以通过实现 `IOrderedFilter` 来重写默认执行序列。 此接口公开了一个 `Order` 属性来确定执行顺序，该属性优先于作用域。 具有较低 `Order` 值的过滤器会在具有较高 `Order` 值的过滤器之前执行其 *before* 代码。 具有较低 `Order` 值的过滤器会在具有较高 `Order` 值的过滤器之后执行其 *after* 代码。 可使用构造函数参数来设置 `Order` 属性：

```csharp
[MyFilter(Name = "Controller Level Attribute", Order=1)]
```

如果具有上述示例中所示的 3 个相同的操作过滤器，但将控制器和全局过滤器的 `Order` 属性分别设置为 1 和 2，则会反转执行顺序。

| 序列 | 过滤器作用域 | `Order` 属性 | 过滤器方法 |
|:--------:|:------------:|:-----------------:|:-------------:|
| 1 | 方法 | 0 | `OnActionExecuting` |
| 2 | 控制器 | 1  | `OnActionExecuting` |
| 3 | Global | 2  | `OnActionExecuting` |
| 4 | Global | 2  | `OnActionExecuted` |
| 5 | 控制器 | 1  | `OnActionExecuted` |
| 6 | 方法 | 0  | `OnActionExecuted` |

在确定过滤器的运行顺序时，`Order` 属性优先于作用域。 先按顺序对过滤器排序，然后使用作用域消除并列问题。 所有内置过滤器实现 `IOrderedFilter` 并将默认 `Order` 值设为 0。 对于内置过滤器，作用域会确定顺序，除非将 `Order` 设为非零值。

## <a name="cancellation-and-short-circuiting"></a>取消和设置短路

通过设置提供给过滤器方法的 `context` 参数上的 `Result` 属性，可以在过滤器管道的任意位置设置短路。 例如，以下资源过滤器将阻止执行管道的其余阶段。

<a name="short-circuiting-resource-filter"></a>

[!code-csharp[](./filters/sample/src/FiltersSample/Filters/ShortCircuitingResourceFilterAttribute.cs?highlight=12,13,14,15)]

在下面的代码中，`ShortCircuitingResourceFilter` 和 `AddHeader` 过滤器都以 `SomeResource` 操作方法为目标。 `ShortCircuitingResourceFilter`：

* 先运行，因为它是资源过滤器且 `AddHeader` 是操作过滤器。
* 对管道的其余部分进行短路处理。

这样 `AddHeader` 过滤器就不会为 `SomeResource` 操作运行。 如果这两个过滤器都应用于操作方法级别，只要 `ShortCircuitingResourceFilter` 先运行，此行为就不会变。 先运行 `ShortCircuitingResourceFilter`（考虑到它的过滤器类型），或显式使用 `Order` 属性。

[!code-csharp[](./filters/sample/src/FiltersSample/Controllers/SampleController.cs?name=snippet_AddHeader&highlight=1,9)]

## <a name="dependency-injection"></a>依赖关系注入

可按类型或实例添加过滤器。 如果添加实例，该实例将用于每个请求。 如果添加类型，则将激活该类型，这意味着将为每个请求创建一个实例，并且[依赖关系注入](../../fundamentals/dependency-injection.md) (DI) 将填充所有构造函数依赖项。 按类型添加过滤器等效于 `filters.Add(new TypeFilterAttribute(typeof(MyFilter)))`。

如果将过滤器作为属性实现并直接添加到控制器类或操作方法中，则该过滤器不能由[依赖关系注入](../../fundamentals/dependency-injection.md) (DI) 提供构造函数依赖项。 这是因为属性在应用时必须提供自己的构造函数参数。 这是属性工作原理上的限制。

如果过滤器具有一些需要从 DI 访问的依赖项，有几种受支持的方法可用。 可以使用以下接口之一，将过滤器应用于类或操作方法：

* `ServiceFilterAttribute`
* `TypeFilterAttribute`
* 在属性上实现的 `IFilterFactory`

> [!NOTE]
> 记录器就是一种可能需要从 DI 获取的依赖项。 但是，应避免单纯为进行日志记录而创建和使用过滤器，因为[内置的框架日志记录功能](xref:fundamentals/logging/index)可能已经提供用户所需。 如果要将日志记录功能添加到过滤器，它应重点关注业务领域问题或特定于过滤器的行为，而非 MVC 操作或其他框架事件。

### <a name="servicefilterattribute"></a>ServiceFilterAttribute

在 DI 中注册服务过滤器实现类型。 `ServiceFilterAttribute` 可从 DI 检索过滤器实例。 将 `ServiceFilterAttribute` 添加到 `Startup.ConfigureServices` 中的容器中，并在 `[ServiceFilter]` 属性中引用它：

[!code-csharp[](./filters/sample/src/FiltersSample/Startup.cs?name=snippet_ConfigureServices&highlight=11)]

[!code-csharp[](../../mvc/controllers/filters/sample/src/FiltersSample/Controllers/HomeController.cs?name=snippet_ServiceFilter&highlight=1)]

使用 `ServiceFilterAttribute` 时，`IsReusable` 设置会提示：过滤器实例可能在其创建的请求范围之外被重用。 该框架不保证在稍后的某个时刻将创建过滤器的单个实例，或不会从 DI 容器重新请求过滤器。 如果使用的过滤器依赖于具有除单一实例以外的生命周期的服务，请避免使用 `IsReusable`。

使用 `ServiceFilterAttribute` 时不注册过滤器类型会引发异常：

```
System.InvalidOperationException: No service for type
'FiltersSample.Filters.AddHeaderFilterWithDI' has been registered.
```

`ServiceFilterAttribute` 可实现 `IFilterFactory`。 `IFilterFactory` 公开用于创建 `IFilterMetadata` 实例的 `CreateInstance` 方法。 `CreateInstance` 方法从服务容器 (DI) 中加载指定的类型。

### <a name="typefilterattribute"></a>TypeFilterAttribute

`TypeFilterAttribute` 与 `ServiceFilterAttribute` 类似，但不会直接从 DI 容器解析其类型。 它使用 `Microsoft.Extensions.DependencyInjection.ObjectFactory` 对类型进行实例化。

由于存在这种差异，所以存在以下情况：

* 使用 `TypeFilterAttribute` 引用的类型不需要先注册在容器中。  它们具备由容器实现的依赖项。 
* `TypeFilterAttribute` 可以选择为类型接受构造函数参数。

使用 `TypeFilterAttribute` 时，`IsReusable` 设置会提示：过滤器实例可能在其创建的请求范围之外被重用。 该框架不保证将创建过滤器的单一实例。 如果使用的过滤器依赖于具有除单一实例以外的生命周期的服务，请避免使用 `IsReusable`。

下面的示例演示如何使用 `TypeFilterAttribute` 将参数传递到类型：

[!code-csharp[](../../mvc/controllers/filters/sample/src/FiltersSample/Controllers/HomeController.cs?name=snippet_TypeFilter&highlight=1,2)]

### <a name="ifilterfactory-implemented-on-your-attribute"></a>在属性上实现 IFilterFactory

如果你的过滤器符合以下描述：

* 不需要任何参数。
* 具备需要由 DI 填充的构造函数依赖项。

在类和方法上可以不使用 `[TypeFilter(typeof(FilterType))]`改用自己命名的属性。 下面的过滤器展示了如何实现此操作：

[!code-csharp[](./filters/sample/src/FiltersSample/Filters/SampleActionFilterAttribute.cs?name=snippet_TypeFilterAttribute&highlight=1,3,7)]

可以使用 `[SampleActionFilter]` 语法将此过滤器应用于类或方法，而不必使用 `[TypeFilter]` 或 `[ServiceFilter]`。

## <a name="authorization-filters"></a>授权过滤器

授权过滤器：

* 控制对操作方法的访问。
* 是过滤器管道中要执行的第一个过滤器。 
* 具有在它之前的执行的方法，但没有之后执行的方法。 

用户只有在编写自己的授权框架时，才应编写自定义授权过滤器。 建议配置授权策略或编写自定义授权策略，而不是编写自定义过滤器。 内置过滤器实现只负责调用授权系统。

切勿在授权过滤器内引发异常，因为没有任何能处理该异常的组件（异常过滤器不会进行处理）。 在出现异常时请小心应对。

详细了解[授权](xref:security/authorization/introduction)。

## <a name="resource-filters"></a>资源过滤器

* 实现 `IResourceFilter` 或 `IAsyncResourceFilter` 接口，
* 它们的执行会覆盖过滤器管道的绝大部分。 
* 只有[授权过滤器](#authorization-filters)在资源过滤器之前运行。

如果需要使某个请求正在执行的大部分工作短路，资源过滤器会很有用。 例如，如果响应在缓存中，则缓存过滤器可以绕开管道的其余阶段。

前面所示的[短路资源过滤器](#short-circuiting-resource-filter)便是一种资源过滤器。 另一个示例是 [DisableFormValueModelBindingAttribute](https://github.com/aspnet/Entropy/blob/rel/1.1.1/samples/Mvc.FileUpload/Filters/DisableFormValueModelBindingAttribute.cs)：

* 可以防止模型绑定访问表单数据。 
* 如果要上传大型文件，同时想防止表单被读入内存，那么此过滤器会很有用。

## <a name="action-filters"></a>操作过滤器

操作过滤器：

* 实现 `IActionFilter` 或 `IAsyncActionFilter` 接口。
* 它们的执行围绕着操作方法的执行。

下面是一个操作过滤器示例：

[!code-csharp[](./filters/sample/src/FiltersSample/Filters/SampleActionFilter.cs?name=snippet_ActionFilter)]

[ActionExecutingContext](/dotnet/api/microsoft.aspnetcore.mvc.filters.actionexecutingcontext) 提供以下属性：

* `ActionArguments`：用于处理对操作的输入。
* `Controller`：用于处理控制器实例。 
* `Result`：设置此属性会使操作方法和后续操作过滤器的执行短路。 引发异常也会阻止操作方法和后续过滤器的执行，但会被视为失败，而不是一个成功的结果。

[ActionExecutedContext](/dotnet/api/microsoft.aspnetcore.mvc.filters.actionexecutedcontext) 提供 `Controller` 和 `Result` 以及下列属性：

* `Canceled`：如果操作执行已被另一个过滤器设置短路，则为 true。
* `Exception`：如果操作或后续操作过滤器引发了异常，则为非 NULL 值。 将此属性设置为 NULL 可有效地“处理”异常，并且将执行 `Result`，就像它是从操作方法正常返回的一样。

对于 `IAsyncActionFilter`，一个向 `ActionExecutionDelegate` 的调用可以达到以下目的：

* 执行所有后续操作过滤器和操作方法。
* 返回 `ActionExecutedContext`。 

若要设置短路，可将 `ActionExecutingContext.Result` 分配到某个结果实例，并且不调用 `ActionExecutionDelegate`。

该框架提供一个可子类化的抽象 `ActionFilterAttribute`。 

操作过滤器可用于验证模型状态，并在状态为无效时返回任何错误：

[!code-csharp[](./filters/sample/src/FiltersSample/Filters/ValidateModelAttribute.cs)]

`OnActionExecuted` 方法在操作方法之后运行，可通过 `ActionExecutedContext.Result` 属性查看和处理操作结果。 如果操作执行已被另一个过滤器设置短路，则 `ActionExecutedContext.Canceled` 设置为 true。 如果操作或后续操作过滤器引发了异常，则 `ActionExecutedContext.Exception` 设置为非 NULL 值。 将 `ActionExecutedContext.Exception` 设置为 null：

* 有效地“处理”异常。
* 执行 `ActionExectedContext.Result`，从操作方法中将它正常返回。

## <a name="exception-filters"></a>异常过滤器

*异常过滤器*可实现 `IExceptionFilter` 或 `IAsyncExceptionFilter` 接口。 它们可用于为应用实现常见的错误处理策略。 

下面的异常过滤器示例使用自定义开发人员错误视图，显示在开发应用时发生的异常的相关详细信息：

[!code-csharp[](./filters/sample/src/FiltersSample/Filters/CustomExceptionFilterAttribute.cs?name=snippet_ExceptionFilter&highlight=1,14)]

异常过滤器：

* 没有之前和之后的事件。 
* 实现 `OnException` 或 `OnExceptionAsync`。 
* 处理控制器创建、[模型绑定](../models/model-binding.md)、操作过滤器或操作方法中发生的未经处理的异常。 
* 请不要捕获资源过滤器、结果过滤器或 MVC 结果执行中发生的异常。

若要处理异常，请将 `ExceptionContext.ExceptionHandled` 属性设置为 true，或编写响应。 这将停止传播异常。 异常过滤器无法将异常转变为“成功”。 只有操作过滤器才能执行该转变。

> [!NOTE]
> 在 ASP.NET Core 1.1 中，如果将 `ExceptionHandled` 设置为 true 并编写响应，则不会发送响应。 在这种情况下，ASP.NET Core 1.0 不发送响应，ASP.NET Core 1.1.2 则恢复为 1.0 的行为。 有关详细信息，请参阅 GitHub 存储库中的[问题编号 5594](https://github.com/aspnet/Mvc/issues/5594)。 

异常过滤器：

* 非常适合捕获发生在 MVC 操作中的异常。
* 并不像错误处理中间件那么灵活。 

建议使用中间件处理异常。 仅在需要根据所选 MVC 操作以不同方式执行错误处理时，才使用异常过滤器。 例如，应用可能具有用于 API 终结点和视图/HTML 的操作方法。 API 终结点可能返回 JSON 形式的错误信息，而基于视图的操作可能返回 HTML 形式的错误页。

`ExceptionFilterAttribute` 可以子类化。 

## <a name="result-filters"></a>结果过滤器

* 实现 `IResultFilter` 或 `IAsyncResultFilter` 接口。
* 它们的执行围绕着操作结果的执行。 

下面是一个添加 HTTP 标头的结果过滤器示例。

[!code-csharp[](./filters/sample/src/FiltersSample/Filters/LoggingAddHeaderFilter.cs?name=snippet_ResultFilter)]

要执行的结果类型取决于所执行的操作。 返回视图的 MVC 操作会将所有 Razor 处理作为要执行的 `ViewResult` 的一部分。 API 方法可能会将某些序列化操作作为结果执行的一部分。 详细了解[操作结果](actions.md)

当操作或操作过滤器生成操作结果时，仅针对成功的结果执行结果过滤器。 当异常过滤器处理异常时，不执行结果过滤器。

`OnResultExecuting` 方法可以将 `ResultExecutingContext.Cancel` 设置为 true，使操作结果和后续结果过滤器的执行短路。 设置短路时，通常应写入响应对象，以免生成空响应。 如果引发异常，则会导致：

* 阻止操作结果和后续过滤器的执行。
* 结果被视为失败而不是成功。

当 `OnResultExecuted` 方法运行时，响应可能已发送到客户端，而且不能再更改（除非引发了异常）。 如果操作结果执行已被另一个过滤器设置短路，则 `ResultExecutedContext.Canceled` 设置为 true。

如果操作结果或后续结果过滤器引发了异常，则 `ResultExecutedContext.Exception` 设置为非 NULL 值。 将 `Exception` 设置为 NULL 可有效地“处理”异常，并防止 MVC 在管道的后续阶段重新引发该异常。 在处理结果过滤器中的异常时，可能无法向响应写入任何数据。 如果操作结果在其执行过程中引发异常，并且标头已刷新到客户端，则没有任何可靠的机制可用于发送失败代码。

对于 `IAsyncResultFilter`，通过调用 `ResultExecutionDelegate` 上的 `await next` 可执行所有后续结果过滤器和操作结果。 若要设置短路，可将 `ResultExecutingContext.Cancel` 设置为 true，并且不调用 `ResultExectionDelegate`。

该框架提供一个可子类化的抽象 `ResultFilterAttribute`。 前面所示的 [AddHeaderAttribute](#add-header-attribute) 类是一种结果过滤器属性。

## <a name="using-middleware-in-the-filter-pipeline"></a>在过滤器管道中使用中间件

资源过滤器的工作方式与[中间件](xref:fundamentals/middleware/index)类似，即涵盖管道中的所有后续执行。 但过滤器又不同于中间件，它们是 MVC 的一部分，这意味着它们有权访问 MVC 上下文和构造。

在 ASP.NET Core 1.1 中，可以在过滤器管道中使用中间件。 如果有一个中间件组件，该组件需要访问 MVC 路由数据，或者只能针对特定控制器或操作运行，则可能需要这样做。

若要将中间件用作过滤器，可创建一个具有 `Configure` 方法的类型，该方法可指定要注入到过滤器管道的中间件。 下面的示例使用本地化中间件为请求建立当前区域性：

[!code-csharp[](./filters/sample/src/FiltersSample/Filters/LocalizationPipeline.cs?name=snippet_MiddlewareFilter&highlight=3,21)]

然后，可以使用 `MiddlewareFilterAttribute` 为所选控制器或操作或者在全局范围内运行中间件：

[!code-csharp[](./filters/sample/src/FiltersSample/Controllers/HomeController.cs?name=snippet_MiddlewareFilter&highlight=2)]

中间件过滤器与资源过滤器在过滤器管道的相同阶段运行，即，在模型绑定之前以及管道的其余阶段之后。

## <a name="next-actions"></a>后续操作

若要尝试使用过滤器，请[下载、测试并修改该示例](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/controllers/filters/sample)。
