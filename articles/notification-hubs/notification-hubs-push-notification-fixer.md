---
title: "Azure 通知中心 - 诊断指南"
description: "有关如何在 Azure 通知中心诊断常见问题的指南。"
services: notification-hubs
documentationcenter: Mobile
author: ysxu
manager: erikre
editor: 
ms.assetid: b5c89a2a-63b8-46d2-bbed-924f5a4cce61
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: NA
ms.devlang: multiple
ms.topic: article
ms.date: 10/03/2016
ms.author: yuaxu
ms.openlocfilehash: 32e3a2e6f840afd865375a622cfae0d33ba65090
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/11/2017
---
# <a name="azure-notification-hubs---diagnosis-guidelines"></a>Azure 通知中心 - 诊断指南
## <a name="overview"></a>概述
我们从 Azure 通知中心客户处收到的最常见问题之一是如何找出以下问题的原因：他们看不到从应用程序后端发送的通知显示在客户端设备上，删除通知的位置和原因以及如何修复此类问题。 在本文中，我们将查看为什么通知被删除或没有在设备上终止的各种原因。 我们还将浏览可以用来分析和找出根本原因的方法。 

首先，理解 Azure 通知中心如何将通知推送到设备很重要。
![][0]

在典型的发送通知流中，消息是从应用程序后端发送到 Azure 通知中心 (NH)，这反过来会对考虑配置的标记和标记表达式来确定“目标”的所有注册（即需要接收推送通知的所有注册）进行一些处理。 这些注册可以横跨各种受支持的平台 - iOS、Google、Windows、Windows Phone、Kindle 和 Baidu for China Android。 建立目标之后，NH 将推送出通知，将通知拆分为多个批量发送到设备平台专用**推送通知服务 (PNS)**（例如，APNS for Apple、GCM for Google 等）。NH 使用各自的 PNS（基于在配置通知中心页面上的 Azure 经典门户中设置的凭据）进行身份验证。 然后，PNS 会将通知转发到各自的**客户端设备**。 这是平台推荐的方式，用以传递推送通知，并且注意通知传递的最后 Leg 在平台 PNS 和设备之间发生。 因此，我们有四个主要组件（*客户端*、*应用程序后端*、*Azure 通知中心 (NH)* 和*推送通知服务 (PNS)*），并且这些组件的任意一个都有可能导致通知被删除。 有关此体系结构的更多详细信息，请参阅[通知中心概述]。

在可能指示配置问题的初始测试/暂存阶段中，可能出现无法传递通知的情况，或者可能在生产中发生这种情况，这可能导致所有或部分通知被删除，同时指明一些更深层次的应用程序或消息模式问题。 在本节中，我们会在下面查看各种已删除通知场景，从常见类型到更加稀有的类型一应俱全，其中一些你可能发现很常见，其中一些并不常见。 

## <a name="azure-notifications-hub-mis-configuration"></a>Azure 通知中心配置错误
Azure 通知中心需要在开发人员的应用程序的环境中对自身进行身份验证，以成功将通知发送到各自的 PNS。 这种情况是可能的，方法是开发人员在各自的平台（Google、Apple、Windows 等）中创建开发人员帐户，并注册可在其中获取凭据（需要在通知中心配置部分下的门户中进行配置）的应用程序。 如果没有通过任何通知，第一步应该是确保在通知中心中配置正确的凭据，并且要与在平台专用开发人员帐户下创建的应用程序相匹配。 会发现[入门教程]非常有用，可以逐步完成此过程。 下面是一些常见的错误配置：

1. **常规**
   
    a) 确保通知中心名称（不含错字）相同：
   
   * 从客户端注册的具体位置； 
   * 其中从后端发送通知，  
   * 其中已配置 PNS 凭据并且 
   * 已在客户端和后端配置它的 SAS 凭据。 
     
     b) 确保使用的是客户端和应用程序后端上的正确 SAS 配置字符串。 一般说来，必须在客户端上使用**DefaultListenSharedAccessSignature** 并在应用程序后端（它可赋予向 NH 发送通知的权限）上使用 **DefaultFullSharedAccessSignature**
2. **Apple Push Notification 服务 (APNS) 配置**
   
    必须维护两个不同的中心 - 一个用于生产目的，另一个用于测试目的。 这意味着将要在沙箱环境中使用的证书上传到一个中心，并将要在生产中使用的证书上传到另一个中心。 请勿尝试将不同类型的证书上传到相同的中心，因为它可能造成通知完全失败。 如果发现自己无意中将不同类型的证书上传到相同的中心，我们建议删除该中心并重新开始。 由于某种原因，如果无法删除中心，则最起码必须从该中心中删除所有现有注册。 
3. **Google Cloud Messaging (GCM) 配置** 
   
    a) 确保在云项目下启用“Google Cloud Messaging for Android”。 
   
    ![][2]
   
    b) 确保创建“服务器密钥”，同时获取 NH 进行 GCM 身份验证时使用的凭据。 
   
    ![][3]
   
    c) 确保已在客户端上配置“项目 ID”，其中该客户端是可以从仪表板中获取的完全数字实体：
   
    ![][1]

## <a name="application-issues"></a>应用程序问题
1) **标记/标记表达式**

如果使用标记或标记表达式来细分受众，发送通知时这总是有可能的，根据你在发送调用中指定的标记/标记表达式，未发现目标。 发送通知并验证仅从含有这些注册的客户端中收到通知时，最好查看注册以确保存在匹配的标记。 例如 如果向 NH 进行的所有注册都是通过“政治”标记（比如说）完成，但要使用“体育”标记发送通知，那么通知将不会发送到任何设备。 复杂用例可能涉及下面的标记表达式：只使用“标记 A”或“标记 B”进行注册，但在发送通知时，定位的是“标记 A”和“标记 B”。 在下文的自我诊断提示部分中，有几种方法可以用于查看注册以及它们含有的标记。 

2) **模板问题**

如果使用的是模板，则确保遵循[模板指南]中描述的以下指南。 

3) **注册无效**

假设通知中心配置正确，而且任意标记/标记表达式使用正确（可以找到需要向其发送通知的有效目标），NH 会关闭并行的几个批处理，每个批处理都会向一组注册发送消息。 

> [!NOTE]
> 由于我们执行并行处理，因此不保证传递通知的顺序。 
> 
> 

现在，已为“最多一次”消息传递模型优化 Azure 通知中心。 这表示我们尝试执行重复数据消除，从而不向设备发送一次以上的通知。 为了确保这一点，在实际将消息发送到 PNS 之前，我们浏览注册并确保每个设备标识符仅发送一条消息。 当每个批处理被发送到 PNS 之后（这反过来会接受和验证注册），PNS 有可能在批处理的一个或多个注册中检测到错误，将错误返回到 Azure NH，并停止处理，从而完全删除该批处理。 对于使用 TCP 流协议的 APNS 也是如此。 尽管我们已针对“最多一次”传送做了优化，但在这种情况下，我们将从数据库中删除出错的注册，然后针对该批中的其他设备重试通知传送。

可以使用 Azure 通知中心 REST API 来获取有关注册的失败传送尝试的错误信息：[按照消息遥测数据：获取通知消息遥测数据](https://msdn.microsoft.com/library/azure/mt608135.aspx)和 [PNS 反馈](https://msdn.microsoft.com/library/azure/mt705560.aspx)。 有关示例代码，请参阅 [SendRESTExample](https://github.com/Azure/azure-notificationhubs-samples/tree/master/dotnet/SendRestExample)。

## <a name="pns-issues"></a>PNS 问题
各自的 PNS 收到通知消息之后，那么它的责任就是将通知传递到设备。 此时，Azure 通知中心是不相关的，而且不会控制何时会通知传递到设备或是否将通知传递到设备。 由于平台通知服务非常强大，这些通知会在几秒钟时间从 PNS 到达很多设备。 但是，如果 PNS 进行限制的话，那么 Azure 通知中心会应用指数让步策略；如果 PNS 在 30 分钟之内都无法联系，则我们会准备一个策略以宣布这些消息过期并永久删除它们。 

如果 PNS 尝试传递通知，但设备处于脱机状态，则通知被 PNS 短暂存储，在设备可用时传递到该设备。 只存储了特定应用的一个最近通知。 如果在设备处于脱机状态时发送了多个通知，则每个新通知将导致前一个通知被放弃。 只保留最新通知的这类行为在 APNS 中被称为合并通知，在 GCM（它使用折叠密钥）中被称为折叠通知。 如果设备长时间处于脱机状态，则放弃所有为它存储的通知。 来源 - [APNS 指南] & [GCM 指南]

在 Azure 通知中心中，可以使用泛型 `SendNotification` API（例如，对于 .NET SDK – `SendNotificationAsync`）通过 HTTP 标头来传递合并密钥，其还会将按原样传递的 HTTP 标头传递到各自的 PNS。 

## <a name="self-diagnose-tips"></a>自我诊断提示
此处我们会检查用于诊断的各种途径以及所有通知中心问题的根本原因：

### <a name="verify-credentials"></a>验证凭据
1. **PNS 开发人员门户**
   
    使用[入门教程]在各自的 PNS 开发人员门户（APNS、GCM、WNS 等）中验证它们。
2. **Azure 经典门户**
   
    转到“配置”选项卡，查看并将凭据与从 PNS 开发人员门户中获取的凭据进行匹配。 
   
    ![][4]

### <a name="verify-registrations"></a>验证注册
1. **Visual Studio**
   
    如果使用 Visual Studio 进行开发，则可以连接到 Microsoft Azure，并查看和管理大量的 Azure 服务，包括“服务器资源管理器”中的通知中心。 这主用于开发/测试环境。 
   
    ![][9]
   
    可以在中心中查看和管理所有注册，其中该中心已经针对平台、本机或模板注册、所有标记、PNS 标识符、注册 ID 以及过期日期进行很好地分类。 可以动态编辑注册；假如要编辑所有标记，这非常有用。 
   
    ![][8]
   
   > [!NOTE]
   > 编辑注册的 Visual Studio 功能应该只能在开发/测试有限的注册时使用。 如果需要批量修复注册，可以考虑使用[导入/导出注册](https://msdn.microsoft.com/library/dn790624.aspx)中所述的导出/导入注册功能
   > 
   > 
2. **服务总线资源管理器**
   
    很多客户使用[服务总线资源管理器]中描述的服务总线资源管理器，查看并管理自己的通知中心。 [服务总线资源管理器代码]是开源项目，可从 code.microsoft.com 中获取

### <a name="verify-message-notifications"></a>验证消息通知
1. **Azure 经典门户**
   
    可以转到“调试”选项卡向客户端发送测试通知，无需启动和运行任何服务后端。 
   
    ![][7]
2. **Visual Studio**
   
    还可以从 Visual Studio 的 comforts 发送测试通知：
   
    ![][10]
   
    可以在此处阅读有关 Visual Studio 通知中心 Azure Resource Manage 功能的更多信息 
   
   * [VS 服务器资源管理器概述]
   * [VS 服务器资源管理器博客文章 - 1]
   * [VS 服务器资源管理器博客文章 - 2]

### <a name="debug-failed-notifications-review-notification-outcome"></a>调试失败的通知/查看通知结果
**EnableTestSend 属性**

通过通知中心发送通知时，起初只要对 NH 排队以进行处理，从而找到它的所有目标，然后最终 NH 将它发送到 PNS。 这意味着，使用 REST API 或任意客户端 SDK 时，发送调用的成功返回只表示消息已成功在通知中心中排队。 当 NH 最终准备将消息发送到 PNS 时，它不会深入探索发生了什么情况。 如果通知没有到达客户端设备，则可能在 NH 尝试将消息传递到 PNS 时出现错误。例如，负载大小超出了 PNS 允许的上限，或者在 NH 中配置的凭据无效等。若要深入分析 PNS 错误，我们引入了一个名为 [EnableTestSend 功能]的属性。 从门户或 Visual Studio 客户端中发送测试消息时，系统会自动启用此属性，从而允许查看详细的调试信息。 根据 .NET SDK 的示例，可以通过 API 使用此属性，其现在可用，并且最终会被添加到所有客户端 SDK。 要和 REST 调用一起使用此属性，直接在发送调用的末尾附加名为“test”的查询字符串参数。例如： 

    https://mynamespace.servicebus.windows.net/mynotificationhub/messages?api-version=2013-10&test

*示例 (.NET SDK)*

假设要使用 .NET SDK 发送原生 toast 通知：

    NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString(connString, hubName);
    var result = await hub.SendWindowsNativeNotificationAsync(toast);
    Console.WriteLine(result.State);

`result.State` 将只在执行结束时陈述 `Enqueued`，而不深入分析推送发生了什么情况。 现在，可以使用 `EnableTestSend` 布尔值属性，同时初始化 `NotificationHubClient`，并获取有关发送通知时遇到的 PNS 错误的详细状态。 此处发送调用需要更多时间进行返回，因为它只在 NH 已将通知传递到 PNS 之后返回以确定结果。 

    bool enableTestSend = true;
    NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString(connString, hubName, enableTestSend);

    var outcome = await hub.SendWindowsNativeNotificationAsync(toast);
    Console.WriteLine(outcome.State);

    foreach (RegistrationResult result in outcome.Results)
    {
        Console.WriteLine(result.ApplicationPlatform + "\n" + result.RegistrationId + "\n" + result.Outcome);
    }

*示例输出*

    DetailedStateAvailable
    windows
    7619785862101227384-7840974832647865618-3
    The Token obtained from the Token Provider is wrong

此消息表示在通知中心配置的凭据无效，或在中心注册方面存在问题，推荐的方案是删除此注册并让客户端重新创建注册，并发送消息。 

> [!NOTE]
> 请注意，此属性的使用受到极大限制，因此只能在开发/测试环境中将它与一组有限的注册结合使用。 我们仅向 10 台设备发送调试通知。 此外，我们每分钟向 10 台设备发送有限的处理调试。 
> 
> 

### <a name="review-telemetry"></a>查看遥测
1. **使用 Azure 经典门户**
   
    通过该门户可以获取有关通知中心上所有活动的快速概述。 
   
    a) 从“仪表板”选项卡上，可以查看注册、通知以及每个平台的错误的汇总视图。 
   
    ![][5]
   
    b) 从“监视器”选项卡中，还可以添加其他很多平台专用指标，以便针对在NH 尝试发送通知给 PNS 时返回的所有 PNS 特有错误进行深入查看。 
   
    ![][6]
   
    c) 首先，应查看**传入消息**、**注册操作**、**成功通知**，并转到每个平台选项卡查看 PNS 具体的错误。 
   
    d) 如果在身份验证设置中错误配置了通知中心，那么会看到 PNS 身份验证错误。 这表示要检查 PNS 凭据。 

2) **以编程方式访问**

在此处了解更多详情 - 

* [以编程方式遥测访问]
* [通过 API 示例遥测访问] 

> [!NOTE]
> 与功能（例如，**导出/导入注册**、**通过 API 进行遥测访问**等）相关的若干个遥测只可在标准层中使用。 如果使用免费层或基本层并尝试使用这些功能，那么在从 REST API 中直接使用它们时使用 SDK 和 HTTP 403（禁止），将收到这方面的异常消息。 请确保已通过 Azure 经典门户向上移到标准层。  
> 
> 

<!-- IMAGES -->
[0]: ./media/notification-hubs-diagnosing/Architecture.png
[1]: ./media/notification-hubs-diagnosing/GCMConfigure.png
[2]: ./media/notification-hubs-diagnosing/GCMEnable.png
[3]: ./media/notification-hubs-diagnosing/GCMServerKey.png
[4]: ./media/notification-hubs-diagnosing/PortalConfigure.png
[5]: ./media/notification-hubs-diagnosing/PortalDashboard.png
[6]: ./media/notification-hubs-diagnosing/PortalMonitoring.png
[7]: ./media/notification-hubs-diagnosing/PortalTestNotification.png
[8]: ./media/notification-hubs-diagnosing/VSRegistrations.png
[9]: ./media/notification-hubs-diagnosing/VSServerExplorer.png
[10]: ./media/notification-hubs-diagnosing/VSTestNotification.png

<!-- LINKS -->
[通知中心概述]: notification-hubs-push-notification-overview.md
[入门教程]: notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md
[模板指南]: https://msdn.microsoft.com/library/dn530748.aspx 
[APNS 指南]: https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/ApplePushService.html#//apple_ref/doc/uid/TP40008194-CH100-SW4
[GCM 指南]: http://developer.android.com/google/gcm/adv.html
[Export/Import Registrations]: http://msdn.microsoft.com/library/dn790624.aspx
[服务总线资源管理器]: http://msdn.microsoft.com/library/dn530751.aspx
[服务总线资源管理器代码]: https://code.msdn.microsoft.com/windowsazure/Service-Bus-Explorer-f2abca5a
[VS 服务器资源管理器概述]: http://msdn.microsoft.com/library/windows/apps/xaml/dn792122.aspx 
[VS 服务器资源管理器博客文章 - 1]: http://azure.microsoft.com/blog/2014/04/09/deep-dive-visual-studio-2013-update-2-rc-and-azure-sdk-2-3/#NotificationHubs 
[VS 服务器资源管理器博客文章 - 2]: http://azure.microsoft.com/blog/2014/08/04/announcing-release-of-visual-studio-2013-update-3-and-azure-sdk-2-4/ 
[EnableTestSend 功能]: http://msdn.microsoft.com/library/microsoft.servicebus.notifications.notificationhubclient.enabletestsend.aspx
[以编程方式遥测访问]: http://msdn.microsoft.com/library/azure/dn458823.aspx
[通过 API 示例遥测访问]: https://github.com/Azure/azure-notificationhubs-samples/tree/master/FetchNHTelemetryInExcel

