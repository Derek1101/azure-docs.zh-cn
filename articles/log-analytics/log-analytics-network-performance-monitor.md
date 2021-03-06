---
title: "Azure Log Analytics 中的网络性能监视器解决方案 | Microsoft 文档"
description: "借助 Azure Log Analytics 中的网络性能监视器可以监视网络的性能，即时检测出网络性能瓶颈。"
services: log-analytics
documentationcenter: 
author: bandersmsft
manager: carmonm
editor: 
ms.assetid: 5b9c9c83-3435-488c-b4f6-7653003ae18a
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/18/2017
ms.author: banders
ms.openlocfilehash: 10e8eeaade5d51b1a15c30802b28600bcf6c72d9
ms.sourcegitcommit: d6ad3203ecc54ab267f40649d3903584ac4db60b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/19/2017
---
# <a name="network-performance-monitor-solution-in-log-analytics"></a>Log Analytics 中的网络性能监视器解决方案

![网络性能监视器符号](./media/log-analytics-network-performance-monitor/npm-symbol.png)

本文档介绍了如何设置和使用 Log Analytics 中的网络性能监视器解决方案，它可以帮助你监视网络的性能，即时检测出网络性能瓶颈。 通过使用网络性能监视器解决方案，可以监视两个网络、子网或服务器之间的丢失和延迟。 网络性能监视器会检测到诸如流量黑洞、路由错误和常规网络监视方法无法检测到的问题之类的网络问题。 只要突破网络链接的阈值，网络性能监视器就会生成警报并进行通知。 系统可以自动获知这些阈值，也可以配置这些阈值以使用自定义警报规则。 网络性能监视器确保及时检测到网络性能问题，然后将问题的根源本地化为特定网络段或设备。

可以使用解决方案仪表板检测网络问题，该仪表板会显示与网络相关的汇总信息，包括最近的网络运行状况事件、不正常的网络链接以及面临大量数据包丢失和延迟的子网链接。 可以深入网络链接，查看子网链接的当前运行状况状态以及节点到节点链接。 还可以查看网络、子网和节点到节点级别的丢失和延迟的历史走向。 可以通过查看历史走向图检测到瞬时网络问题，并在拓扑图上找到网络瓶颈。 交互式拓扑图使你可以可视化逐跳网络路由，并确定问题的根源。 如同任何其他解决方案一样，可以针对各种分析需求使用日志搜索，以基于网络性能监视器收集的数据创建自定义报告。

该解决方案使用综合事务作为检测网络故障的主要机制。 因此，无需考虑特定网络设备的供应商或型号，就可以使用它。 可跨本地、云 (IaaS) 和混合环境进行工作。 该解决方案会自动发现网络拓扑和你网络中的各种路由。

典型的网络监视产品重点监视网络设备（路由器、交换机等）运行状况，并不会就两个点之间网络连接实际质量提供见解（网络性能监视器会提供这种见解）。

### <a name="using-the-solution-standalone"></a>单独使用解决方案
如果想要监视其关键工作负荷、网络、数据中心或办公地点之间的网络连接质量，可以使用网络性能监视器解决方案本身来监视以下各项之间的连接状况：

* 使用公共网络或专用网络连接的多个数据中心或办公地点
* 正在运行业务线应用程序的关键工作负荷
* 诸如 Microsoft Azure 或 Amazon Web Services (AWS) 之类的公有云服务和本地网络（如果提供 IaaS (VM) 并且将网关配置为允许本地网络和云网络之间进行通信）
* Azure 和本地网络（使用 Express Route 时）

### <a name="using-the-solution-with-other-networking-tools"></a>结合其他网络工具使用解决方案
如果想要监视业务线应用程序，可以将网络性能监视器解决方案用作其他网络工具的配套解决方案。 网速较慢会导致应用程序运行缓慢，网络性能监视器可以帮助你调查因潜在网络问题引起的应用程序性能问题。 由于该解决方案不需要对网络设备进行任何访问，因此应用程序管理员不需要依靠网络团队提供有关网络如何影响应用程序的信息。

此外，如果已购买其他网络监视工具，那么该解决方案可以补充这些工具，因为大多数常规网络监视解决方案不会就端到端网络性能指标（如丢失和延迟）提供见解。  网络性能监视器解决方案可帮助填补这一空白。

## <a name="installing-and-configuring-agents-for-the-solution"></a>安装和配置适用于解决方案的代理
使用[将 Windows 计算机连接到 Log Analytics](log-analytics-windows-agents.md) 和[将 Operations Manager 连接到 Log Analytics](log-analytics-om-agents.md) 中的基本过程安装代理。

> [!NOTE]
> 要有足够数据发现并监视网络资源，需要安装至少 2 个代理。 否则，该解决方案会一直保持为配置状态，直到安装并配置其他代理。
>
>

### <a name="where-to-install-the-agents"></a>代理安装位置
在安装代理之前，请考虑网络的拓扑结构以及要监视网络的哪些部分。 我们建议为想要监视的每个子网安装一个以上的代理。 也就是说，对于要监视的每一个子网，请选择至少两个服务器或 VM，并在其上安装代理。

如果不确定你网络的拓扑结构，请在具有关键工作负荷、想要监视网络性能的服务器上安装代理。 例如，可能想要跟踪 Web 服务器与运行 SQL Server 的服务器之间的网络连接。 在该示例中，应在这两台服务器上都安装代理。

代理会监视主机之间的网络连接（链接） - 而不是主机本身。 因此，若要监视某个网络链接，必须在该链接的两个终结点上都安装代理。

### <a name="configure-agents"></a>配置代理

如果打算为综合事务使用 ICMP 协议，则需要启用以下防火墙规则以便可靠地利用 ICMP：

```
netsh advfirewall firewall add rule name="NPMDICMPV4Echo" protocol="icmpv4:8,any" dir=in action=allow
netsh advfirewall firewall add rule name="NPMDICMPV6Echo" protocol="icmpv6:128,any" dir=in action=allow
netsh advfirewall firewall add rule name="NPMDICMPV4DestinationUnreachable" protocol="icmpv4:3,any" dir=in action=allow
netsh advfirewall firewall add rule name="NPMDICMPV6DestinationUnreachable" protocol="icmpv6:1,any" dir=in action=allow
netsh advfirewall firewall add rule name="NPMDICMPV4TimeExceeded" protocol="icmpv4:11,any" dir=in action=allow
netsh advfirewall firewall add rule name="NPMDICMPV6TimeExceeded" protocol="icmpv6:3,any" dir=in action=allow
```


如果打算使用 TCP 协议，则需要打开这些计算机的防火墙端口，确保代理可以通信。 需要下载 [EnableRules.ps1 PowerShell 脚本](https://gallery.technet.microsoft.com/OMS-Network-Performance-04a66634)，并使用管理员权限在 PowerShell 窗口中在不使用任何参数的情况下运行该脚本。

该脚本会创建网络性能监视器所需的注册表项，还会创建用于允许代理创建相互之间的 TCP 连接的 Windows 防火墙规则。 该脚本创建的注册表项还指定是否记录调试日志和该日志文件的路径。 还会定义用于通信的代理 TCP 端口。 该脚本会自动设置这些注册表项的值，因此不应手动更改这些注册表项。

默认打开的端口为 8084。 通过向该脚本提供参数 `portNumber` 即可使用自定义端口。 不过，应在运行该脚本的所有计算机上使用相同端口。

> [!NOTE]
> EnableRules.ps1 脚本仅在运行该脚本的计算机上配置 Windows 防火墙规则。 如果有网络防火墙，应确保该防火墙允许流量去往网络性能监视器正在使用的 TCP 端口。
>
>

## <a name="configuring-the-solution"></a>配置解决方案
使用以下信息安装和配置解决方案。

1. 网络性能监视器解决方案将从运行 Windows Server 2008 SP1 或更高版本或者 Windows 7 SP1 或更高版本的计算机获取数据，这与 Microsoft Monitoring Agent (MMA) 的要求是相同的。 NPM 代理也可以在 Windows 桌面/客户端操作系统（Windows 10、Windows 8.1、Windows 8 和 Windows 7）上运行。
    >[!NOTE]
    >Windows 服务器操作系统的代理同时支持使用 TCP 和 ICMP 作为综合事务的协议。 不过，Windows 客户端操作系统的代理仅支持使用 ICMP 作为综合事务的协议。

2. 从 [Azure 应用商店](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.NetworkMonitoringOMS?tab=Overview)或使用[从解决方案库中添加 Log Analytics 解决方案](log-analytics-add-solutions.md)中所述的过程，将网络性能监视器解决方案添加到工作区。<br><br> ![网络性能监视器符号](./media/log-analytics-network-performance-monitor/npm-symbol.png)  
3. 在 OMS 门户中，将看到一个标题为**网络性能监视器**的新磁贴以及*解决方案需要进行额外配置*消息。 单击磁贴导航到“部署”选项卡，选择用来创建综合事务以监视网络的协议。  查看[选择正确的协议 - ICMP 或 TCP](#choose-the-right-protocol-icmp-or-tcp)，帮助自己选择适合网络的适当协议。<br><br> ![解决方案要求选择协议](media/log-analytics-network-performance-monitor/log-analytics-netmon-perf-welcome.png)<br><br>

4. 选择协议后，将重定向到“OMS 概述”页。 当解决方案从网络聚合数据时，网络性能监视器概述磁贴会显示一条消息，指出“数据聚合正在进行”。<br><br> ![解决方案正在聚合数据](media/log-analytics-network-performance-monitor/log-analytics-netmon-tile-status-01.png)<br><br>
5. 收集数据并为其编制索引后，概述磁贴会发生变化，指示需要执行其他配置。<br><br> ![解决方案磁贴中要求执行其他配置](media/log-analytics-network-performance-monitor/log-analytics-netmon-tile-status-02.png)<br><br>
6. 单击该磁贴，并遵循以下步骤开始配置解决方案。

### <a name="create-new-networks"></a>新建网络
网络性能监视器中的网络是子网的逻辑容器。 可以创建一个具有友好名称的网络，并根据业务逻辑在其中添加子网。 例如，可以创建名为 *London* 的网络并添加伦敦数据中心内的所有子网，或创建名为 *ContosoFrontEnd* 的网络，并将为 Contoso 应用的前端提供服务的所有子网添加到此网络。
在配置页上的“网络”选项卡中，可以看到名为“默认”的网络。如果尚未创建任何网络，自动发现的所有子网将会放入“默认”网络中。
每当创建网络时，会向创建的网络中添加子网，并从默认网络中删除该子网。 删除某个网络时，会自动将其中的所有子网返回到默认网络。
因此，“默认”网络充当包含在用户定义的任何网络中的所有子网的容器。 无法编辑或删除默认网络。 它始终保留在系统中。 不过，可以根据需要创建任意数量的自定义网络。
在大多数情况下，会将组织中的子网排列在多个网络中，应根据业务逻辑创建一个或多个网络，以便对子网进行分组

#### <a name="to-create-a-new-network"></a>新建网络
1. 单击“添加网络”，并键入网络名称和描述。
2. 选择一个或多个子网，并单击“添加”。
3. 单击“保存”以保存配置。<br><br> ![添加网络](./media/log-analytics-network-performance-monitor/npm-add-network.png)

### <a name="wait-for-data-aggregation"></a>等待数据聚合
首次保存配置之后，该解决方案会开始收集已安装代理的节点之间的网络数据包丢失和延迟信息。 此过程可能需要一些时间，有时会超过 30 分钟。 在此状态下，概述页中的网络性能监视器磁贴会显示一条指示“正在进行数据聚合”的消息。

![正在进行数据聚合](./media/log-analytics-network-performance-monitor/npm-aggregation.png)

当数据完成上传后，会看到网络性能监视器磁贴更新为正在显示数据。

![网络性能监视器磁贴](./media/log-analytics-network-performance-monitor/npm-tile.png)

单击该磁贴即可查看网络性能监视器仪表板。

![网络性能监视器仪表板](./media/log-analytics-network-performance-monitor/npm-dash01.png)

### <a name="edit-monitoring-settings-for-subnets"></a>编辑子网的监视设置
配置页上的“子网”选项卡中将列出至少安装一个代理的所有子网。

#### <a name="to-enable-or-disable-monitoring-for-particular-subnetworks"></a>启用或禁用对特定子网的监视
1. 选中或清除“子网 ID”旁边的框，并确保已选中或已清除“用于监视”（视情况而定）。 可以选择或清除多个子网。 禁用时，不会监视子网，因为会更新代理以停止 ping 其他代理。
2. 通过以下方式选择想要监视的特定子网的节点：从列表中选择子网，并在包含未监视节点的列表与包含已监视节点的列表之间移动所需节点。
   可以根据需要向子网添加自定义“描述”。
3. 单击“保存”以保存配置。<br><br> ![编辑子网](./media/log-analytics-network-performance-monitor/npm-edit-subnet.png)

### <a name="choose-nodes-to-monitor"></a>选择要监视的节点
所有已安装代理的节点会列在“节点”选项卡上。

#### <a name="to-enable-or-disable-monitoring-for-nodes"></a>启用或禁用对节点的监视
1. 选择要监视的节点或清除要停止监视的节点。
2. 单击“用于监视”，或清除它（视情况而定）。
3. 单击“保存” 。<br><br> ![启用节点监视](./media/log-analytics-network-performance-monitor/npm-enable-node-monitor.png)

### <a name="set-monitoring-rules"></a>设置监视规则
突破 2 个子网或 2 个网络之间网络连接的性能阈值时，网络性能监视器会生成运行状况事件。 系统可以自动获知这些阈值，或者，你也可以提供自定义警报。
系统会自动创建默认规则，每当任何一对网络/子网链接之间的丢失或延迟突破系统获知的阈值时，该规则就会生成运行状况事件。 在尚未显式创建任何监视规则之前，这可以帮助解决方案监视网络基础结构。 如果启用了默认规则，所有节点都会将综合事务发送到已启用监视的其他所有节点。 如果网络规模较小，例如，有少量的服务器在运行微服务，而你想要确保所有服务器相互连接，则默认规则就非常有用。

>[!NOTE]
>我们强烈建议禁用默认规则并创建自定义监视规则，尤其网络规模较大，需要使用大量的节点进行监视时。 这可以减少解决方案生成的流量，并有助于对网络监视进行组织。

根据业务逻辑创建自定义监视规则。 例如，如果想要监视 2 个办公地点到总部的网络连接性能，可将办公地点 1 中的所有子网分组到网络 O1 中，将办公地点 2 中的所有子网分组到网络 O2 中，将总部中的所有子网分组到网络 H 中。创建 2 个监视规则 - 一个在 O1 与 H 之间创建，另一个在 O2 与 H 之间创建。


#### <a name="to-create-custom-monitoring-rules"></a>创建自定义监视规则
1. 在“监视器”选项卡中，单击“添加规则”，并输入规则名称和描述。
2. 从列表中选择要监视的网络或子网链接对。
3. 首先从网络下拉列表中选择含有第一个感兴趣子网的网络，然后从相应的子网下拉列表中选择子网。
   如果想要监视网络链接中的所有子网，请选择“全部子网”。 同样，选择其他感兴趣的子网。 还可以单击“添加例外”从已选项中排除对特定子网链接的监视。
4. [选择 ICMP 或 TCP 协议](#choose-the-right-protocol-icmp-or-tcp)来执行综合事务。
5. 如果不希望针对所选项生成运行状况事件，请清除“启用对此规则覆盖的链接进行运行状况监视”。
6. 选择监视条件。
   可以通过键入阈值，针对运行状况事件生成设置自定义阈值。 只要条件值超出其针对所选网络/子网对选择的阈值，就会生成运行状况事件。
7. 单击“保存”以保存配置。<br><br> ![创建自定义监视规则](./media/log-analytics-network-performance-monitor/npm-monitor-rule.png)

保存监视规则后，可以单击“创建警报”，将该规则与警报管理集成。 系统将使用搜索查询与其他自动填入的所需参数自动创建警报规则。 使用警报规则可以收到基于电子邮件的警报，以及 NPM 中的现有警报。 警报还能配合 Runbook 触发补救措施，或者，可以使用 Webhook 将警报与现有服务管理解决方案集成。 可以单击“管理警报”来编辑警报设置。

### <a name="choose-the-right-protocol-icmp-or-tcp"></a>选择正确的协议 - ICMP 或 TCP

网络性能监视器 (NPM) 使用综合事务来计算数据包丢失和链路延迟等网络性能指标。 为了更好地理解这一点，请考虑一个已连接到网络链路一端的 NPM 代理。 此 NPM 代理将探测数据包发送至已连接到网络另一端的第二个 NPM 代理。 第二个代理使用响应数据包答复。 此过程将重复多次。 通过测量答复数和接收每个答复所花费的时间，第一个 NPM 代理将评估链路延迟和数据包丢弃情况。

这些数据包的格式、大小和序列由在创建监视规则时选择的协议决定。 根据数据包协议，中间网络设备（路由器、交换机等）可能以不同方式处理这些数据包。 因此，协议选择将影响结果的准确性。 此外，协议选择还决定是否在部署 NPM 解决方案后必须执行任何手动步骤。

NPM 提供了 ICMP 与 TCP 协议之间的选择，以便于执行综合事务。
如果在创建综合事务规则时选择 ICMP，NPM 代理将使用 ICMP ECHO 消息来计算与网络延迟和数据包丢失相关的数据。 ICMP ECHO 使用传统 Ping 实用程序发送的同一消息。 当使用 TCP 作为协议时，NPM 代理通过网络发送 TCP SYN 数据包。 此后，TCP 握手完成，然后将使用 RST 数据包删除连接。

#### <a name="points-to-consider-before-choosing-the-protocol"></a>选择协议之前需考虑的要点
选择要使用的协议之前，请考虑以下信息：

##### <a name="discovering-multiple-network-routes"></a>发现多个网络路由
发现多个路由时，TCP 更为准确，并且在每个子网中需要使用的代理更少。 例如，一个或两个使用 TCP 的代理可以发现子网之间的所有冗余路径。 但是，需要多个使用 ICMP 的代理才能达到类似结果。 使用 ICMP，如果两个子网之间有 *N* 个路由，则源或目标子网中需要 5*N* 个以上的代理。

##### <a name="accuracy-of-results"></a>结果的准确性
路由器和交换机往往将较低的优先级分配给 ICMP ECHO 数据包（与 TCP 数据包相比）。 在某些情况下，当网络设备负载很重时，TCP 获取的数据将更准确地反映应用程序遇到的丢失和延迟情况。 这是因为大部分应用程序流量都是通过 TCP 传送的。 在这种情况下，与 TCP 相比，ICMP 提供的结果准确性较低。

##### <a name="firewall-configuration"></a>防火墙配置
TCP 协议要求 TCP 数据包发送到目标端口。 NPM 代理使用的默认端口为 8084，但可以在配置代理时更改此端口。 因此，需要确保网络防火墙或 NSG 规则（在 Azure 中）允许该端口上的流量。 还需要确保安装代理的计算机上的本地防火墙已配置为允许此端口上的流量。

可以使用 PowerShell 脚本配置运行 Windows 的计算机上的防火墙规则，但需要手动配置网络防火墙。

相反，ICMP 不使用端口运行。 多数企业方案中都允许 ICMP 流量通过防火墙，以便于使用 Ping 实用程序等网络诊断工具。 因此，如果可以从一台计算机 Ping 另一台计算机，则可以使用 ICMP 协议而无需手动配置防火墙。

> [!NOTE]
> 某些防火墙可能会阻止 ICMP，导致重新传输，因而在安全信息和事件管理系统中生成大量的事件。 请确保选择的协议未被网络防火墙/NSG 阻止，否则 NPM 无法监视网段。  因此，我们建议使用 TCP 进行监视。 如果存在如下所述的情况，因而无法使用 TCP，则应使用 ICMP：
> * 使用基于 Windows 客户端的节点，因为 Windows 客户端中不允许 TCP 原始套接字
> * 网络防火墙/NSG 会阻止 TCP


#### <a name="how-to-switch-the-protocol"></a>如何切换协议

如果在部署期间选择使用 ICMP，可以随时通过编辑默认监视规则来切换到 TCP。

##### <a name="to-edit-the-default-monitoring-rule"></a>编辑默认监视规则
1.  导航到“网络性能” > “监视” > “配置” > “监视”，并单击“默认规则” 。
2.  滚动到“协议”部分，并选择要使用的协议。
3.  单击“保存”以应用设置。

即使默认规则使用特定协议，也可以使用其他协议创建新规则。 甚至可以创建混合规则，其中一些规则使用 ICMP，另一些规则使用 TCP。


## <a name="data-collection-details"></a>数据收集详细信息
选择 TCP 作为协议来收集丢失与延迟信息时，网络性能监视器使用 TCP SYN-SYNACK-ACK 握手数据包；如果选择 ICMP 作为协议来收集此类信息，则它会使用 ICMP ECHO ICMP ECHO REPLY。 此外，还使用跟踪路由来获取拓扑信息。

下表显示了数据收集方法，以及有关如何为网络性能监视器收集数据的其他详细信息。

| 平台 | 直接代理 | SCOM 代理 | Azure 存储 | 是否需要 SCOM？ | 通过管理组发送的 SCOM 代理数据 | 收集频率 |
| --- | --- | --- | --- | --- | --- | --- |
| Windows | &#8226; | &#8226; |  |  |  |每隔 5 秒发送 TCP 握手/ICMP ECHO 消息，每隔 3 分钟发送数据 |

解决方案使用综合事务来评估网络的运行状况。 网络中各个点安装的 OMS 代理会相互交换 TCP 数据包或 ICMP Echo（取决于选择用于监视的协议）。 在此过程中，代理将了解往返时间和丢包情况（如果有）。 每个代理还会定期对其他代理执行跟踪路由，以全部找出网络中必须测试的各种路由。 使用此数据，代理就可以推断出网络延迟和丢包数字。 代理每隔 5 秒钟重复执行测试，聚合数据 3 分钟，然后将数据上传到 Log Analytics 服务。

> [!NOTE]
> 尽管代理相互频繁地通信，但它们在进行测试时并不会产生大量网络流量。 代理仅仅依靠 TCP SYN-SYNACK-ACK 握手数据包确定丢失和延迟 - 不会交换任何数据包。 在此过程中，代理只在需要时才会进行相互通信，并且会对代理通信拓扑进行优化以减少网络流量。
>
>

## <a name="using-the-solution"></a>使用解决方案
本部分介绍了所有仪表板函数及其使用方法。

### <a name="solution-overview-tile"></a>解决方案概述磁贴
在启用了网络性能监视器解决方案后，OMS 概述页中的解决方案磁贴将提供网络运行状况的快速浏览。 将显示一张环形图，呈现正常和不正常子网链接的数目。 单击该磁贴时，会打开解决方案仪表板。

![网络性能监视器磁贴](./media/log-analytics-network-performance-monitor/npm-tile.png)

### <a name="network-performance-monitor-solution-dashboard"></a>网络性能监视器解决方案仪表板
“网络摘要”边栏选项卡中会显示网络的摘要及其相对大小。 磁贴随后会显示系统中的网络链接、子网链接和路径（路径由代理的两个主机的 IP 地址以及这两个主机之间的跃点组成）的总数。

“排名靠前的网络运行状况事件”边栏选项卡中会提供系统中最新运行状况事件和警报列表以及事件发生后所经过的时间。 只要网络或子网链接的数据包丢失或延迟超过阈值，就会生成运行状况事件或警报。

“排名靠前的不正常的网络链接”边栏选项卡中会显示不正常的网络链接列表。 此类网络链接定义如下：当时这些网络链接中存在一个或多个不良运行状况事件。

“丢失最多、排名靠前的子网链接”和“延迟最多的子网链接”边栏选项卡中会分别显示按数据包丢失排列的排名靠前的子网链接和按延迟排列的排名靠前的子网链接。 某些网络链接中可能会发生高延迟或一定量的数据包丢失。 此类链接会显示在前十个列表中，但不会标记为不正常。

“常见查询”边栏选项卡中包含一组用于直接提取网络监视原始数据的搜索查询。 可以基于这些查询创建用于生成自定义报表的查询。

![网络性能监视器仪表板](./media/log-analytics-network-performance-monitor/npm-dash01.png)

### <a name="drill-down-for-depth"></a>深入了解
可以单击解决方案仪表板中的各个链接来进一步了解任何感兴趣部分。 例如，当看到警报或不正常的网络链接出现在仪表板上时，可以单击它以进一步进行调查。 会你将转到列出该特定网络链接的所有子网链接的页面。 将能够查看每个子网链接的丢失、延迟和运行状况状态，快速找出哪个子网链接导致了该问题。 可以单击“查看节点链接”查看不正常的子网链接的所有节点链接。 然后，可以查看个别节点到节点链接，找到不正常的节点链接。

可以单击“查看拓扑”查看源节点和目标节点之间路由的逐跳拓扑。 不正常的路由或跃点会显示为红色，以便你能够快速确定网络特定部分的问题。

![数据挖掘](./media/log-analytics-network-performance-monitor/npm-drill.png)

### <a name="network-state-recorder"></a>网络状态记录器

每个视图显示特定时间点的网络运行状况快照。 默认会显示最新的状态。 页面顶部栏显示该状态所处的时间点。 可以选择后退到某个时间，并在栏中单击“操作”查看网络运行状况的快照。 查看最新状态时，还可以选择启用或禁用任何页面的自动刷新。

![网络状态](./media/log-analytics-network-performance-monitor/network-state.png)

#### <a name="trend-charts"></a>趋势图
在进行挖掘的每个级别，都可以查看某个网络链接的丢失和延迟趋势。 趋势图也适用于子网和节点链接。 可以通过使用图表顶部的时间控制，更改图绘制的时间间隔。

趋势图从历史角度向你呈现了网络链接的性能。 某些网络问题本质上是瞬时发生的，仅通过查看该网络的当前状态难以捕获。 这是因为问题可能会快速浮现并在有人注意到之前就消失，只在之后的某个时间点才会再次出现。 此类瞬时问题对于应用程序管理员而言也比较棘手，因为这些问题常常表现为应用程序响应时间原因不明的增加，即使所有应用程序组件看起来都平稳地运行时也会如此。

可以通过查看趋势图（其中，问题会呈现为网络延迟或数据包丢失出现激增情况）轻松地检测到这些类型的问题。

![趋势图](./media/log-analytics-network-performance-monitor/npm-trend.png)

#### <a name="hop-by-hop-topology-map"></a>逐跳拓扑图
网络性能监视器会向你呈现交互式拓扑图中两个节点之间路由的逐跳拓扑图。 可以通过以下方式查看拓扑图：选择某个节点链接，并单击“查看拓扑”。 此外，也可以通过单击仪表板上的“路径”磁贴来查看拓扑图。 单击仪表板上的“路径”时，需要从左侧面板中选择源节点和目标节点，单击“绘图”即可绘制两个节点之间的路由。

拓扑图会显示两个节点之间存在多少个路由，以及数据包会采用哪条路径。 在拓扑图上，网络性能瓶颈会以红色标记。 可以通过查看拓扑图上标为红色的元素，定位到发生问题的网络连接或网络设备。

当在拓扑图上单击某个节点或将光标悬停在其上方时，会看到诸如 FQDN 和 IP 地址之类的节点属性。 单击跃点查看其 IP 地址。 可以使用可折叠的操作窗格中的筛选器来选择筛选特定的路由。 还可以通过使用操作窗格中的滑块隐藏中间跃点来简化网络拓扑。 可以使用鼠标滚轮来放大或缩小拓扑图。

请注意，图中所示的拓扑为第 3 层拓扑，不包含第 2 层的设备和连接。

![逐跳拓扑图](./media/log-analytics-network-performance-monitor/npm-topology.png)

#### <a name="fault-localization"></a>错误本地化
网络性能监视器无需连接到网络设备即可找到网络瓶颈。 基于从网络收集的数据以及通过在网络图上应用高级算法，网络性能监视器可大概估计最有可能是问题根源的网络部分。

当不可访问跃点时，此方法可用于确定网络瓶颈，因为不需要从网络设备（例如，路由器或交换机）收集任何数据。 当两个节点之间的跃点不在管理控制下时，此方法也非常有用。 例如，跃点可能是 ISP 路由器。

### <a name="log-analytics-search"></a>Log Analytics 搜索
通过网络性能监视器仪表板和挖掘页面以图形方式显示的所有数据也可以在 Log Analytics 搜索中以本地方式使用。 可以使用搜索查询语言查询数据，然后通过将数据导出到 Excel 或 PowerBI 创建自定义报表。 仪表板的“常见查询”边栏选项卡中有一些查询非常有用，可以基于这些查询创建自己的查询和报表。

![搜索查询](./media/log-analytics-network-performance-monitor/npm-queries.png)

## <a name="investigate-the-root-cause-of-a-health-alert"></a>调查运行状况警报的根本原因
现在，已对网络性能监视器有所了解，下面我们简单分析一下某个运行状况事件的根本原因。

1. 在“概述”页上，通过观察“网络性能监视器”磁贴即可快速获取网络的运行状况快照。 可以看到，在 6 个受监视的子网链接中，有 2 个是不正常的。 这需要调查。 单击该磁贴以查看解决方案仪表板。<br><br> ![网络性能监视器磁贴](./media/log-analytics-network-performance-monitor/npm-investigation01.png)  
2. 在以下示例图中，可以看到有某个运行状况事件指出网络链接不正常。 若要调查该问题，请单击“DMZ2-DMZ1”网络链接找出问题的根源。<br><br> ![不正常的网络链接示例](./media/log-analytics-network-performance-monitor/npm-investigation02.png)  
3. 深化页显示“DMZ2-DMZ1”网络链接中的所有子网链接。 你会注意到，这两个子网链接的延迟已超过阈值，使网络链接不正常。 还会看到这两个子网链接的延迟趋势。 图表中的时间选择控制可用于重点关注所需时间范围。 可以查看延迟达到其峰值时的当天时间。 若要调查问题，稍后也可以搜索日志来获取此时间段。 单击“查看节点链接”以进一步进行挖掘。<br><br> ![不正常的子网链接示例](./media/log-analytics-network-performance-monitor/npm-investigation03.png) 
4. 与上一页类似，特定子网链接的挖掘页面会列出其构成节点链接。 可以在此处执行与上一步类似的操作。 单击“查看拓扑”即可查看 2 个节点之间的拓扑。<br><br> ![不正常的节点链接示例](./media/log-analytics-network-performance-monitor/npm-investigation04.png)  
5. 2 个所选节点之间的所有路径绘制在拓扑图中。 可以在拓扑图上可视化两个节点之间路由的逐跳拓扑。 它清晰地呈现两个节点之间存在多少个路由，以及数据包会采用哪条路径。 网络性能瓶颈会标记为红色。 可以通过查看拓扑图上标为红色的元素，定位到发生问题的网络连接或网络设备。<br><br> ![不正常的拓扑视图示例](./media/log-analytics-network-performance-monitor/npm-investigation05.png)  
6. 可在“操作”窗格中查看每个路径中的丢失情况、延迟和跃点数。 滚动条可用于查看这些不正常的路径的详细信息。  使用筛选器可以选择包含不正常跃点的路径，以便只绘制所选路径的拓扑。 可以使用鼠标滚轮放大或缩小拓扑图。

   在下图中，通过查看红色标记的路径和跃点，就可以清楚地了解到网络特定部分的问题区域的根本原因。 单击拓扑图中的节点会呈现该节点的属性，包括 FQDN 和 IP 地址。 单击跃点会显示该跃点的 IP 地址。<br><br> ![不正常的拓扑 - 路径详细信息示例](./media/log-analytics-network-performance-monitor/npm-investigation06.png)

## <a name="provide-feedback"></a>提供反馈

- **UserVoice** - 可以发表有关希望我们开发的网络性能监视器功能的想法。 请访问我们的 [UserVoice 页](https://feedback.azure.com/forums/267889-log-analytics/category/188146-network-monitoring)。
- **加入我们的队伍** - 我们总是希望一直有新客户不断加入我们的队伍。 那样，能够在早期接触到新功能并帮助我们改进网络性能监视器。 如果有兴趣加入，请填写此[快速调查](https://aka.ms/npmcohort)。

## <a name="next-steps"></a>后续步骤
* [搜索日志](log-analytics-log-searches.md)以查看详细的网络性能数据记录。
