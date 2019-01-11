---
title: ASP.NET Core 中的 Razor 页面的筛选方法
author: Rick-Anderson
description: 了解如何为 ASP.NET Core 中的 Razor 页面创建筛选方法。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.date: 04/05/2018
uid: razor-pages/filter
ms.openlocfilehash: 5b233d95c9fbab09c64072377b85b40b127df7b7
ms.sourcegitcommit: 375e9a67f5e1f7b0faaa056b4b46294cc70f55b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/29/2018
ms.locfileid: "50205933"
---
# <a name="filter-methods-for-razor-pages-in-aspnet-core"></a>ASP.NET Core 中的 Razor 页面的筛选方法

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

Razor 页面过滤器 [IPageFilter](/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter?view=aspnetcore-2.0) 和 [IAsyncPageFilter](/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncpagefilter?view=aspnetcore-2.0) 允许 Razor 页面在运行 Razor 页面处理程序的前后运行代码。 Razor 页面过滤器与 [ASP.NET Core MVC 操作过滤器](xref:mvc/controllers/filters#action-filters)类似，但它们不能应用于单个页面处理程序方法。 

Razor 页面过滤器：

* 在选择处理程序方法后但在模型绑定发生前运行代码。
* 在模型绑定完成后，执行处理程序方法前运行代码。
* 在执行处理程序方法后运行代码。
* 可在页面或全局范围内实现。
* 无法应用于特定的页面处理程序方法。

代码可在使用页面构造函数或中间件执行处理程序方法前运行，但仅 Razor 页面过滤器可访问 [HttpContext](/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel.httpcontext?view=aspnetcore-2.0#Microsoft_AspNetCore_Mvc_RazorPages_PageModel_HttpContext)。 过滤器具有 [FilterContext](/dotnet/api/microsoft.aspnetcore.mvc.filters.filtercontext?view=aspnetcore-2.0) 派生参数，可用于访问 `HttpContext`。 例如，[实现过滤器属性](#ifa)示例将标头添加到响应中，而使用构造函数或中间件则无法做到这点。

[查看或下载示例代码](https://github.com/aspnet/Docs/tree/master/aspnetcore/razor-pages/filter/sample/PageFilter)（[如何下载](xref:index#how-to-download-a-sample)）

Razor 页面过滤器提供的以下方法可在全局或页面级应用：

* 同步方法：

    * [OnPageHandlerSelected](/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter.onpagehandlerselected?view=aspnetcore-2.0)：在选择处理程序方法后但在模型绑定发生前调用。
    * [OnPageHandlerExecuting](/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter.onpagehandlerexecuting?view=aspnetcore-2.0)：在执行处理器方法前，模型绑定完成后调用。
    * [OnPageHandlerExecuted](/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter.onpagehandlerexecuted?view=aspnetcore-2.0)：在执行处理器方法后，生成操作结果前调用。

* 异步方法：

    * [OnPageHandlerSelectionAsync](/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncpagefilter.onpagehandlerselectionasync?view=aspnetcore-2.0)：在选择处理程序方法后，但在模型绑定发生前，进行异步调用。
    * [OnPageHandlerExecutionAsync](/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncpagefilter.onpagehandlerexecutionasync?view=aspnetcore-2.0)：在调用处理程序方法前，但在模型绑定结束后，进行异步调用。

> [!NOTE]
> 过滤器接口的同步和异步版本**任意**实现一个，而不是同时实现。 该框架会先查看过滤器是否实现了异步接口，如果是，则调用该接口。 如果不是，则调用同步接口的方法。 如果两个接口都已实现，则只会调用异步方法。 对页面中的替代应用相同的规则，同步替代或异步替代只能任选其一实现，不可二者皆选。

## <a name="implement-razor-page-filters-globally"></a>全局实现 Razor 页面过滤器

以下代码实现了 `IAsyncPageFilter`：

[!code-csharp[Main](filter/sample/PageFilter/Filters/SampleAsyncPageFilter.cs?name=snippet1)]

在前面的代码中，[ILogger](/dotnet/api/microsoft.extensions.logging.ilogger?view=aspnetcore-2.0) 不是必需的。 它在示例中用于提供应用程序的跟踪信息。

以下代码启用 `Startup` 类中的 `SampleAsyncPageFilter`：

[!code-csharp[Main](filter/sample/PageFilter/Startup.cs?name=snippet2&highlight=11)]

以下代码显示完整的 `Startup` 类：

[!code-csharp[Main](filter/sample/PageFilter/Startup.cs?name=snippet1)]

以下代码调用 `AddFolderApplicationModelConvention` 将 `SampleAsyncPageFilter` 仅应用于 /subFolder 中的页面：

[!code-csharp[Main](filter/sample/PageFilter/Startup2.cs?name=snippet2)]

以下代码实现同步的 `IPageFilter`：

[!code-csharp[Main](filter/sample/PageFilter/Filters/SamplePageFilter.cs?name=snippet1)]

以下代码启用 `SamplePageFilter`：

[!code-csharp[Main](filter/sample/PageFilter/StartupSync.cs?name=snippet2&highlight=11)]

::: moniker range=">= aspnetcore-2.1"

## <a name="implement-razor-page-filters-by-overriding-filter-methods"></a>通过重写过滤器方法实现 Razor 页面过滤器

以下代码替代同步的 Razor 页面过滤器：

[!code-csharp[Main](filter/sample/PageFilter/Pages/Index.cshtml.cs)]

::: moniker-end

<a name="ifa"></a>
## <a name="implement-a-filter-attribute"></a>实现过滤器属性

基于属性的内置过滤器 [OnResultExecutionAsync](/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncresultfilter.onresultexecutionasync?view=aspnetcore-2.0#Microsoft_AspNetCore_Mvc_Filters_IAsyncResultFilter_OnResultExecutionAsync_Microsoft_AspNetCore_Mvc_Filters_ResultExecutingContext_Microsoft_AspNetCore_Mvc_Filters_ResultExecutionDelegate_) 可以进行子类化。 以下过滤器向响应添加标头：

[!code-csharp[Main](filter/sample/PageFilter/Filters/AddHeaderAttribute.cs)]

以下代码应用 `AddHeader` 属性：

[!code-csharp[Main](filter/sample/PageFilter/Pages/Contact.cshtml.cs?name=snippet1)]

有关重写顺序的说明，请参阅[重写默认顺序](xref:mvc/controllers/filters#overriding-the-default-order)。

有关将过滤器管道与过滤器短路的说明，请参阅[取消和设置短路](xref:mvc/controllers/filters#cancellation-and-short-circuiting)。 

<a name="auth"></a>
## <a name="authorize-filter-attribute"></a>授权过滤器属性

[Authorize](/dotnet/api/microsoft.aspnetcore.authorization.authorizeattribute?view=aspnetcore-2.0) 属性可应用于 `PageModel`：

[!code-csharp[Main](filter/sample/PageFilter/Pages/ModelWithAuthFilter.cshtml.cs?highlight=7)]
