<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="69d3d-101">本教程指导你如何生成 Ruby on Rails Web 应用，该应用使用 Microsoft Graph API 检索用户的日历信息。</span><span class="sxs-lookup"><span data-stu-id="69d3d-101">This tutorial teaches you how to build a Ruby on Rails web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="69d3d-102">如果你更喜欢仅下载已完成的教程，可以通过两种方式下载它。</span><span class="sxs-lookup"><span data-stu-id="69d3d-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="69d3d-103">下载 [Ruby 快速入门](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) ，以在数分钟内获取工作代码。</span><span class="sxs-lookup"><span data-stu-id="69d3d-103">Download the [Ruby quick start](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) to get working code in minutes.</span></span>
> - <span data-ttu-id="69d3d-104">下载或克隆 [GitHub 存储库](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp)。</span><span class="sxs-lookup"><span data-stu-id="69d3d-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="69d3d-105">先决条件</span><span class="sxs-lookup"><span data-stu-id="69d3d-105">Prerequisites</span></span>

<span data-ttu-id="69d3d-106">在开始本教程之前，应在开发计算机上安装以下工具。</span><span class="sxs-lookup"><span data-stu-id="69d3d-106">Before you start this tutorial, you should have the following tools installed on your development machine.</span></span>

- [<span data-ttu-id="69d3d-107">Ruby</span><span class="sxs-lookup"><span data-stu-id="69d3d-107">Ruby</span></span>](https://www.ruby-lang.org/en/downloads/)
- [<span data-ttu-id="69d3d-108">SQLite3</span><span class="sxs-lookup"><span data-stu-id="69d3d-108">SQLite3</span></span>](https://sqlite.org/index.html)
- [<span data-ttu-id="69d3d-109">Node.js</span><span class="sxs-lookup"><span data-stu-id="69d3d-109">Node.js</span></span>](https://nodejs.org/en/)
- [<span data-ttu-id="69d3d-110">花</span><span class="sxs-lookup"><span data-stu-id="69d3d-110">Yarn</span></span>](https://yarnpkg.com/)

<span data-ttu-id="69d3d-111">你还应该拥有具有邮箱的个人 Microsoft 帐户Outlook.com或 Microsoft 工作或学校帐户。</span><span class="sxs-lookup"><span data-stu-id="69d3d-111">You should also have either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span> <span data-ttu-id="69d3d-112">如果你没有 Microsoft 帐户，有几个选项可以获取免费帐户：</span><span class="sxs-lookup"><span data-stu-id="69d3d-112">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="69d3d-113">你可以 [注册新的个人 Microsoft 帐户](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)。</span><span class="sxs-lookup"><span data-stu-id="69d3d-113">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="69d3d-114">你可以 [注册 Office 365 开发人员计划](https://developer.microsoft.com/office/dev-program) ，获取免费的 Office 365 订阅。</span><span class="sxs-lookup"><span data-stu-id="69d3d-114">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="69d3d-115">本教程使用所需工具的以下版本编写。</span><span class="sxs-lookup"><span data-stu-id="69d3d-115">This tutorial was written with the following versions of the required tools.</span></span> <span data-ttu-id="69d3d-116">本指南中的步骤可能与其他版本一起运行，但尚未经过测试。</span><span class="sxs-lookup"><span data-stu-id="69d3d-116">The steps in this guide may work with other versions, but that has not been tested.</span></span>
>
> - <span data-ttu-id="69d3d-117">Ruby 版本 2.6.6。</span><span class="sxs-lookup"><span data-stu-id="69d3d-117">Ruby version 2.6.6.</span></span>
> - <span data-ttu-id="69d3d-118">SQLite3 版本 3.31.1</span><span class="sxs-lookup"><span data-stu-id="69d3d-118">SQLite3 version 3.31.1</span></span>
> - <span data-ttu-id="69d3d-119">Node.js版本 14.15.0</span><span class="sxs-lookup"><span data-stu-id="69d3d-119">Node.js version 14.15.0</span></span>
> - <span data-ttu-id="69d3d-120">花色版本 1.22.0</span><span class="sxs-lookup"><span data-stu-id="69d3d-120">Yarn version 1.22.0</span></span>

## <a name="feedback"></a><span data-ttu-id="69d3d-121">反馈</span><span class="sxs-lookup"><span data-stu-id="69d3d-121">Feedback</span></span>

<span data-ttu-id="69d3d-122">请在 GitHub 存储库中提供有关本教程 [的任何反馈](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp)。</span><span class="sxs-lookup"><span data-stu-id="69d3d-122">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>
