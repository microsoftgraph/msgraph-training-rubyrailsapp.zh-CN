<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="4b439-101">本教程向您介绍如何构建使用 Microsoft Graph API 检索用户的日历信息的 Ruby on Rails web 应用程序。</span><span class="sxs-lookup"><span data-stu-id="4b439-101">This tutorial teaches you how to build a Ruby on Rails web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="4b439-102">如果您只想下载已完成的教程, 可以通过两种方式下载它。</span><span class="sxs-lookup"><span data-stu-id="4b439-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="4b439-103">下载[Ruby 快速入门](https://developer.microsoft.com/graph/quick-start?platform=option-ruby), 以分钟为单位获取工作代码。</span><span class="sxs-lookup"><span data-stu-id="4b439-103">Download the [Ruby quick start](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) to get working code in minutes.</span></span>
> - <span data-ttu-id="4b439-104">下载或克隆[GitHub 存储库](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp)。</span><span class="sxs-lookup"><span data-stu-id="4b439-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="4b439-105">先决条件</span><span class="sxs-lookup"><span data-stu-id="4b439-105">Prerequisites</span></span>

<span data-ttu-id="4b439-106">在开始本教程之前, 您应在开发计算机上安装[Ruby](https://www.ruby-lang.org/en/downloads/) 。</span><span class="sxs-lookup"><span data-stu-id="4b439-106">Before you start this tutorial, you should have [Ruby](https://www.ruby-lang.org/en/downloads/) installed on your development machine.</span></span> <span data-ttu-id="4b439-107">如果没有 Ruby, 请访问 "下载选项" 的上一个链接。</span><span class="sxs-lookup"><span data-stu-id="4b439-107">If you do not have Ruby, visit the previous link for download options.</span></span>

> [!NOTE]
> <span data-ttu-id="4b439-108">本教程是使用 Ruby 版本2.4.4 编写的。</span><span class="sxs-lookup"><span data-stu-id="4b439-108">This tutorial was written with Ruby version 2.4.4.</span></span> <span data-ttu-id="4b439-109">本指南中的步骤可能适用于其他版本, 但尚未经过测试。</span><span class="sxs-lookup"><span data-stu-id="4b439-109">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="feedback"></a><span data-ttu-id="4b439-110">反馈</span><span class="sxs-lookup"><span data-stu-id="4b439-110">Feedback</span></span>

<span data-ttu-id="4b439-111">请在[GitHub 存储库](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp)中提供有关本教程的任何反馈。</span><span class="sxs-lookup"><span data-stu-id="4b439-111">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>