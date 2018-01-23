---
title: "使用 Visual Studio 将 ASP.NET Core 应用发布到 Azure"
author: rick-anderson
description: "了解如何使用 Visual Studio 将 ASP.NET Core 应用发布到 Azure 应用服务。"
ms.author: riande
manager: wpickett
ms.date: 12/16/2017
ms.topic: get-started-article
ms.technology: aspnet
ms.prod: asp.net-core
uid: tutorials/publish-to-azure-webapp-using-vs
ms.openlocfilehash: dd31e3a9583a0c152e97ae7cf6b215389298a20c
ms.sourcegitcommit: 3e303620a125325bb9abd4b2d315c106fb8c47fd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/19/2018
---
<span data-ttu-id="8ac29-103">/zh-cn</span><span class="sxs-lookup"><span data-stu-id="8ac29-103">/en-us</span></span>

# <a name="publish-an-aspnet-core-web-app-to-azure-app-service-using-visual-studio"></a><span data-ttu-id="8ac29-104">使用 Visual Studio 将 ASP.NET Core Web 应用发布到 Azure App Service</span><span class="sxs-lookup"><span data-stu-id="8ac29-104">Publish an ASP.NET Core web app to Azure App Service using Visual Studio</span></span>

<span data-ttu-id="8ac29-105">作者：[Rick Anderson](https://twitter.com/RickAndMSFT) 、[Cesar Blum Silveira](https://github.com/cesarbs) 和 [Rachel Appel](https://twitter.com/rachelappel)</span><span class="sxs-lookup"><span data-stu-id="8ac29-105">By [Rick Anderson](https://twitter.com/RickAndMSFT), [Cesar Blum Silveira](https://github.com/cesarbs), and [Rachel Appel](https://twitter.com/rachelappel)</span></span>

<span data-ttu-id="8ac29-106">如果你在 Mac 上工作，请参阅[从 Visual Studio for Mac 发布到 Azure](https://blog.xamarin.com/publish-azure-visual-studio-mac/)。</span><span class="sxs-lookup"><span data-stu-id="8ac29-106">See [Publish to Azure from Visual Studio for Mac](https://blog.xamarin.com/publish-azure-visual-studio-mac/) if you are working on a Mac.</span></span>

## <a name="set-up"></a><span data-ttu-id="8ac29-107">设置</span><span class="sxs-lookup"><span data-stu-id="8ac29-107">Set up</span></span>

* <span data-ttu-id="8ac29-108">如果没有，请打开[免费 Azure 帐户](https://aka.ms/K5y5yh)。</span><span class="sxs-lookup"><span data-stu-id="8ac29-108">Open a [free Azure account](https://aka.ms/K5y5yh) if you do not have one.</span></span> 

## <a name="create-a-web-app"></a><span data-ttu-id="8ac29-109">创建 Web 应用</span><span class="sxs-lookup"><span data-stu-id="8ac29-109">Create a web app</span></span>

<span data-ttu-id="8ac29-110">在 Visual Studio 起始页中，选择“文件”>“新建”>“项目...”</span><span class="sxs-lookup"><span data-stu-id="8ac29-110">In the Visual Studio Start Page, select **File > New > Project...**</span></span>

![“文件”菜单](publish-to-azure-webapp-using-vs/_static/file_new_project.png)

<span data-ttu-id="8ac29-112">完成“新建项目”对话框：</span><span class="sxs-lookup"><span data-stu-id="8ac29-112">Complete the **New Project** dialog:</span></span>

* <span data-ttu-id="8ac29-113">在左侧窗格中，选择“.NET Core”。</span><span class="sxs-lookup"><span data-stu-id="8ac29-113">In the left pane, select **.NET Core**.</span></span>
* <span data-ttu-id="8ac29-114">在中间窗格中，选择“ASP.NET Core Web 应用程序”。</span><span class="sxs-lookup"><span data-stu-id="8ac29-114">In the center pane, select **ASP.NET Core Web Application**.</span></span>
* <span data-ttu-id="8ac29-115">选择“确定”。</span><span class="sxs-lookup"><span data-stu-id="8ac29-115">Select **OK**.</span></span>

![“新建项目”对话框](publish-to-azure-webapp-using-vs/_static/new_prj.png)

<span data-ttu-id="8ac29-117">在“新建 ASP.NET Core Web 应用程序”对话框中：</span><span class="sxs-lookup"><span data-stu-id="8ac29-117">In the **New ASP.NET Core Web Application** dialog:</span></span>

* <span data-ttu-id="8ac29-118">选择“Web 应用程序”。</span><span class="sxs-lookup"><span data-stu-id="8ac29-118">Select **Web Application**.</span></span>
* <span data-ttu-id="8ac29-119">选择“更改身份验证”。</span><span class="sxs-lookup"><span data-stu-id="8ac29-119">Select **Change Authentication**.</span></span>

![“新建项目”对话框](publish-to-azure-webapp-using-vs/_static/new_prj_2.png)

<span data-ttu-id="8ac29-121">“更改身份验证”对话框随即出现。</span><span class="sxs-lookup"><span data-stu-id="8ac29-121">The **Change Authentication** dialog appears.</span></span> 

* <span data-ttu-id="8ac29-122">选择“个人用户帐户”。</span><span class="sxs-lookup"><span data-stu-id="8ac29-122">Select **Individual User Accounts**.</span></span>
* <span data-ttu-id="8ac29-123">选择“确定”返回到“新建 ASP.NET Core Web 应用程序”，然后再次选择“确定”。</span><span class="sxs-lookup"><span data-stu-id="8ac29-123">Select **OK** to return to the **New ASP.NET Core Web Application**, then select **OK** again.</span></span>

![“新建 ASP.NET Core Web 身份验证”对话框](publish-to-azure-webapp-using-vs/_static/new_prj_auth.png) 

<span data-ttu-id="8ac29-125">Visual Studio 随即创建解决方案。</span><span class="sxs-lookup"><span data-stu-id="8ac29-125">Visual Studio creates the solution.</span></span>

## <a name="run-the-app"></a><span data-ttu-id="8ac29-126">运行应用</span><span class="sxs-lookup"><span data-stu-id="8ac29-126">Run the app</span></span>

* <span data-ttu-id="8ac29-127">按 Ctrl+F5 运行项目。</span><span class="sxs-lookup"><span data-stu-id="8ac29-127">Press CTRL+F5 to run the project.</span></span>
* <span data-ttu-id="8ac29-128">测试“关于”和“联系”链接。</span><span class="sxs-lookup"><span data-stu-id="8ac29-128">Test the **About** and **Contact** links.</span></span>

![localhost 上的 Microsoft Edge 中打开的 Web 应用程序](publish-to-azure-webapp-using-vs/_static/show.png)

### <a name="register-a-user"></a><span data-ttu-id="8ac29-130">注册用户</span><span class="sxs-lookup"><span data-stu-id="8ac29-130">Register a user</span></span>

* <span data-ttu-id="8ac29-131">选择“注册”并注册新用户。</span><span class="sxs-lookup"><span data-stu-id="8ac29-131">Select **Register** and register a new user.</span></span> <span data-ttu-id="8ac29-132">可使用虚构电子邮件地址。</span><span class="sxs-lookup"><span data-stu-id="8ac29-132">You can use a fictitious email address.</span></span> <span data-ttu-id="8ac29-133">提交时，页面上会显示以下错误：</span><span class="sxs-lookup"><span data-stu-id="8ac29-133">When you submit, the page displays the following error:</span></span>

    <span data-ttu-id="8ac29-134">“内部服务器错误: 处理请求时，数据库操作失败。SQL 异常: 无法打开数据库。可通过向应用程序数据库上下文应用现有迁移解决此问题。”\*</span><span class="sxs-lookup"><span data-stu-id="8ac29-134">*"Internal Server Error: A database operation failed while processing the request. SQL exception: Cannot open the database. Applying existing migrations for Application DB context may resolve this issue."*</span></span>
* <span data-ttu-id="8ac29-135">选择“应用迁移”，并在页面更新后刷新页面。</span><span class="sxs-lookup"><span data-stu-id="8ac29-135">Select **Apply Migrations** and, once the page updates, refresh the page.</span></span>

![内部服务器错误: 处理请求时，数据库操作失败。](publish-to-azure-webapp-using-vs/_static/mig.png)

<span data-ttu-id="8ac29-139">应用将显示用于注册新用户的电子邮件和一个“注销”链接。</span><span class="sxs-lookup"><span data-stu-id="8ac29-139">The app displays the email used to register the new user and a **Log out** link.</span></span>

![Microsoft Edge 中打开的 Web 应用程序。](publish-to-azure-webapp-using-vs/_static/hello.png)

## <a name="deploy-the-app-to-azure"></a><span data-ttu-id="8ac29-142">将应用部署到 Azure</span><span class="sxs-lookup"><span data-stu-id="8ac29-142">Deploy the app to Azure</span></span>

<span data-ttu-id="8ac29-143">在解决方案资源管理器中右键单击该项目，然后选择“发布...”。</span><span class="sxs-lookup"><span data-stu-id="8ac29-143">Right-click on the project in Solution Explorer and select **Publish...**.</span></span>

![打开且突出显示“发布”链接的上下文菜单](publish-to-azure-webapp-using-vs/_static/pub.png)

<span data-ttu-id="8ac29-145">在“发布”对话框中：</span><span class="sxs-lookup"><span data-stu-id="8ac29-145">In the **Publish** dialog:</span></span>

* <span data-ttu-id="8ac29-146">选择“Microsoft Azure 应用服务”。</span><span class="sxs-lookup"><span data-stu-id="8ac29-146">Select **Microsoft Azure App Service**.</span></span>
* <span data-ttu-id="8ac29-147">选择齿轮图标，然后选择“创建配置文件”。</span><span class="sxs-lookup"><span data-stu-id="8ac29-147">Select the gear icon and then select **Create Profile**.</span></span>
* <span data-ttu-id="8ac29-148">选择“创建配置文件”。</span><span class="sxs-lookup"><span data-stu-id="8ac29-148">Select **Create Profile**.</span></span>

![“发布”对话框](publish-to-azure-webapp-using-vs/_static/maas1.png)

### <a name="create-azure-resources"></a><span data-ttu-id="8ac29-150">创建 Azure 资源</span><span class="sxs-lookup"><span data-stu-id="8ac29-150">Create Azure resources</span></span>

<span data-ttu-id="8ac29-151">“创建应用服务”对话框随即显示：</span><span class="sxs-lookup"><span data-stu-id="8ac29-151">The **Create App Service** dialog appears:</span></span>

* <span data-ttu-id="8ac29-152">输入订阅。</span><span class="sxs-lookup"><span data-stu-id="8ac29-152">Enter your subscription.</span></span>
* <span data-ttu-id="8ac29-153">“应用名称”、“资源组”和“应用服务计划”输入字段已填充。</span><span class="sxs-lookup"><span data-stu-id="8ac29-153">The **App Name**, **Resource Group**, and **App Service Plan** entry fields are populated.</span></span> <span data-ttu-id="8ac29-154">可以保留这些名称，也可以进行更改。</span><span class="sxs-lookup"><span data-stu-id="8ac29-154">You can keep these names or change them.</span></span>

![“应用服务”对话框](publish-to-azure-webapp-using-vs/_static/newrg1.png)

* <span data-ttu-id="8ac29-156">选择“服务”选项卡以创建新的数据库。</span><span class="sxs-lookup"><span data-stu-id="8ac29-156">Select the **Services** tab to create a new database.</span></span>

* <span data-ttu-id="8ac29-157">选择绿色的 + 图标以创建新的 SQL 数据库</span><span class="sxs-lookup"><span data-stu-id="8ac29-157">Select the green **+** icon to create a new SQL Database</span></span>

![新建 SQL 数据库](publish-to-azure-webapp-using-vs/_static/sql.png)

* <span data-ttu-id="8ac29-159">在“配置 SQL 数据库”对话框中选择“新建...”以创建新的数据库。</span><span class="sxs-lookup"><span data-stu-id="8ac29-159">Select **New...** on the **Configure SQL Database** dialog to create a new database.</span></span>

![新建 SQL 数据库和服务器](publish-to-azure-webapp-using-vs/_static/conf.png)

<span data-ttu-id="8ac29-161">“配置 SQL Server”对话框随即出现。</span><span class="sxs-lookup"><span data-stu-id="8ac29-161">The **Configure SQL Server** dialog appears.</span></span>

* <span data-ttu-id="8ac29-162">输入管理员用户名和密码，然后选择“确定”。</span><span class="sxs-lookup"><span data-stu-id="8ac29-162">Enter an administrator user name and password, and then select **OK**.</span></span> <span data-ttu-id="8ac29-163">可保留默认的“服务器名称”。</span><span class="sxs-lookup"><span data-stu-id="8ac29-163">You can keep the default **Server Name**.</span></span> 

> [!NOTE]
> <span data-ttu-id="8ac29-164">不可使用“admin”作为管理员用户名。</span><span class="sxs-lookup"><span data-stu-id="8ac29-164">"admin" is not allowed as the administrator user name.</span></span>

![“配置 SQL Server”对话框](publish-to-azure-webapp-using-vs/_static/conf_servername.png)

* <span data-ttu-id="8ac29-166">选择“确定”。</span><span class="sxs-lookup"><span data-stu-id="8ac29-166">Select **OK**.</span></span>

<span data-ttu-id="8ac29-167">Visual Studio 将返回到“创建应用服务”对话框。</span><span class="sxs-lookup"><span data-stu-id="8ac29-167">Visual Studio returns to the **Create App Service** dialog.</span></span>

* <span data-ttu-id="8ac29-168">选择“创建应用服务”对话框上的“创建”。</span><span class="sxs-lookup"><span data-stu-id="8ac29-168">Select **Create** on the **Create App Service** dialog.</span></span>

![“配置 SQL 数据库”对话框](publish-to-azure-webapp-using-vs/_static/conf_final.png)

<span data-ttu-id="8ac29-170">Visual Studio 在 Azure 上创建 Web 应用和 SQL Server。</span><span class="sxs-lookup"><span data-stu-id="8ac29-170">Visual Studio creates the Web app and SQL Server on Azure.</span></span> <span data-ttu-id="8ac29-171">此步骤可能需要几分钟。</span><span class="sxs-lookup"><span data-stu-id="8ac29-171">This step can take a few minutes.</span></span> <span data-ttu-id="8ac29-172">有关创建的资源的信息，请参阅[其他资源](#additonal-resources)。</span><span class="sxs-lookup"><span data-stu-id="8ac29-172">For information on the resources created, see [Additonal resources](#additonal-resources).</span></span>

<span data-ttu-id="8ac29-173">部署完成时，选择“设置”：</span><span class="sxs-lookup"><span data-stu-id="8ac29-173">When deployment completes, select **Settings**:</span></span>

![“配置 SQL Server”对话框](publish-to-azure-webapp-using-vs/_static/set.png)

<span data-ttu-id="8ac29-175">在“发布”对话框的“设置”页面上：</span><span class="sxs-lookup"><span data-stu-id="8ac29-175">On the **Settings** page of the **Publish** dialog:</span></span>

  * <span data-ttu-id="8ac29-176">展开“数据库”并选中“在运行时使用此连接字符串”。</span><span class="sxs-lookup"><span data-stu-id="8ac29-176">Expand **Databases** and check **Use this connection string at runtime**.</span></span>
  * <span data-ttu-id="8ac29-177">展开“Entity Framework 迁移”并选中“在发布时应用此迁移”。</span><span class="sxs-lookup"><span data-stu-id="8ac29-177">Expand **Entity Framework Migrations** and check **Apply this migration on publish**.</span></span>

* <span data-ttu-id="8ac29-178">选择“保存”。</span><span class="sxs-lookup"><span data-stu-id="8ac29-178">Select **Save**.</span></span> <span data-ttu-id="8ac29-179">Visual Studio 将返回到“发布”对话框。</span><span class="sxs-lookup"><span data-stu-id="8ac29-179">Visual Studio returns to the **Publish** dialog.</span></span> 

![“发布”对话框：设置面板](publish-to-azure-webapp-using-vs/_static/pubs.png)

<span data-ttu-id="8ac29-181">单击“发布” 。</span><span class="sxs-lookup"><span data-stu-id="8ac29-181">Click **Publish**.</span></span> <span data-ttu-id="8ac29-182">Visual Studio 将应用发布到 Azure。</span><span class="sxs-lookup"><span data-stu-id="8ac29-182">Visual Studio publishs your app to Azure.</span></span> <span data-ttu-id="8ac29-183">部署完成时，应用在浏览器中打开。</span><span class="sxs-lookup"><span data-stu-id="8ac29-183">When the depoyment completes, the app is opened in a browser.</span></span>

### <a name="test-your-app-in-azure"></a><span data-ttu-id="8ac29-184">在 Azure 中测试应用</span><span class="sxs-lookup"><span data-stu-id="8ac29-184">Test your app in Azure</span></span>

* <span data-ttu-id="8ac29-185">测试“关于”和“联系”链接</span><span class="sxs-lookup"><span data-stu-id="8ac29-185">Test the **About** and **Contact** links</span></span>

* <span data-ttu-id="8ac29-186">注册新用户</span><span class="sxs-lookup"><span data-stu-id="8ac29-186">Register a new user</span></span>

![Microsoft Edge 中 Azure App Service 上打开的 Web 应用程序](publish-to-azure-webapp-using-vs/_static/register.png)

### <a name="update-the-app"></a><span data-ttu-id="8ac29-188">更新应用</span><span class="sxs-lookup"><span data-stu-id="8ac29-188">Update the app</span></span>

* <span data-ttu-id="8ac29-189">编辑“Pages/About.cshtml”Razor 页面并更改其内容。</span><span class="sxs-lookup"><span data-stu-id="8ac29-189">Edit the *Pages/About.cshtml* Razor page and change its contents.</span></span> <span data-ttu-id="8ac29-190">例如，可以将段落修改为显示“Hello ASP.NET Core!”：[!code-html[About](publish-to-azure-webapp-using-vs/sample/about.cshtml?highlight=9&range=1-9)]</span><span class="sxs-lookup"><span data-stu-id="8ac29-190">For example, you can modify the paragraph to say "Hello ASP.NET Core!": [!code-html[About](publish-to-azure-webapp-using-vs/sample/about.cshtml?highlight=9&range=1-9)]</span></span>

* <span data-ttu-id="8ac29-191">右键单击项目，然后再次选择“发布...”。</span><span class="sxs-lookup"><span data-stu-id="8ac29-191">Right-click on the project and select **Publish...** again.</span></span>

![打开且突出显示“发布”链接的上下文菜单](publish-to-azure-webapp-using-vs/_static/pub.png)

* <span data-ttu-id="8ac29-193">应用发布后，验证所做的更改在 Azure 上是否可用。</span><span class="sxs-lookup"><span data-stu-id="8ac29-193">After the app is published, verify the changes you made are available on Azure.</span></span>

![验证任务已完成](publish-to-azure-webapp-using-vs/_static/final.png)

### <a name="clean-up"></a><span data-ttu-id="8ac29-195">清理</span><span class="sxs-lookup"><span data-stu-id="8ac29-195">Clean up</span></span>

<span data-ttu-id="8ac29-196">完成应用测试后，转到 [Azure 门户](https://portal.azure.com/)并删除该应用。</span><span class="sxs-lookup"><span data-stu-id="8ac29-196">When you have finished testing the app, go to the [Azure portal](https://portal.azure.com/) and delete the app.</span></span>

* <span data-ttu-id="8ac29-197">选择“资源组”，然后选择所创建的资源组。</span><span class="sxs-lookup"><span data-stu-id="8ac29-197">Select **Resource groups**, then select the resource group you created.</span></span>

![Azure 门户：侧边栏菜单中的资源组](publish-to-azure-webapp-using-vs/_static/portalrg.png)

* <span data-ttu-id="8ac29-199">在“资源组”页面中，选择“删除”。</span><span class="sxs-lookup"><span data-stu-id="8ac29-199">In the **Resource groups** page, select **Delete**.</span></span>

![Azure 门户：“资源组”页面](publish-to-azure-webapp-using-vs/_static/rgd.png)

* <span data-ttu-id="8ac29-201">输入资源组的名称并选择“删除”。</span><span class="sxs-lookup"><span data-stu-id="8ac29-201">Enter the name of the resource group and select **Delete**.</span></span> <span data-ttu-id="8ac29-202">现已从 Azure 中删除了本教程中创建的应用和其他所有资源。</span><span class="sxs-lookup"><span data-stu-id="8ac29-202">Your app and all other resources created in this tutorial are now deleted from Azure.</span></span>

### <a name="next-steps"></a><span data-ttu-id="8ac29-203">后续步骤</span><span class="sxs-lookup"><span data-stu-id="8ac29-203">Next steps</span></span>

* [<span data-ttu-id="8ac29-204">使用 Visual Studio 和 Git 持续部署到 Azure</span><span class="sxs-lookup"><span data-stu-id="8ac29-204">Continuous Deployment to Azure with Visual Studio and Git</span></span>](xref:host-and-deploy/azure-apps/azure-continuous-deployment)

## <a name="additonal-resources"></a><span data-ttu-id="8ac29-205">其他资源</span><span class="sxs-lookup"><span data-stu-id="8ac29-205">Additonal resources</span></span>

* [<span data-ttu-id="8ac29-206">Azure 应用服务</span><span class="sxs-lookup"><span data-stu-id="8ac29-206">Azure App Service</span></span>](https://docs.microsoft.com/en-us/azure/app-service/app-service-web-overview)
* [<span data-ttu-id="8ac29-207">Azure 资源组</span><span class="sxs-lookup"><span data-stu-id="8ac29-207">Azure resource groups</span></span>](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview#resource-groups)
* [<span data-ttu-id="8ac29-208">Azure SQL 数据库</span><span class="sxs-lookup"><span data-stu-id="8ac29-208">Azure SQL Database</span></span>](https://docs.microsoft.com/en-us/azure/sql-database/)