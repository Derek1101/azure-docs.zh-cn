---
title: "Azure 中的 IP 地址类型 | Microsoft 文档"
description: "了解如何在 Azure 中使用公共和专用 IP 地址"
services: virtual-network
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: 
tags: azure-resource-manager
ms.assetid: 610b911c-f358-4cfe-ad82-8b61b87c3b7e
ms.service: virtual-network
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/18/2017
ms.author: jdial
ms.openlocfilehash: 95f2b57b2012df816c76a1b6ec55ca9f92e134a3
ms.sourcegitcommit: afc78e4fdef08e4ef75e3456fdfe3709d3c3680b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/16/2017
---
# <a name="ip-address-types-and-allocation-methods-in-azure"></a>Azure 中的 IP 地址类型和分配方法

可以将 IP 地址分配到与其他 Azure 资源通信的 Azure 资源，也可以将其分配到本地网络和 Internet。 可以在 Azure 中使用两种类型的 IP 地址：

* **公共 IP 地址**：用来与 Internet 通信，包括与面向公众的 Azure 服务通信。
* **专用 IP 地址**：用于在 Azure 虚拟网络 (VNet) 中通信，以及在本地网络中通信（使用 VPN 网关或 ExpressRoute 线路将网络扩展到 Azure 时）。

> [!NOTE]
> Azure 具有用于创建和处理资源的两个不同的部署模型：[Resource Manager 和经典](../azure-resource-manager/resource-manager-deployment-model.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。  本文介绍如何使用 Resource Manager 部署模型。Microsoft 建议对大多数新的部署使用该模型，而不是[经典部署模型](virtual-network-ip-addresses-overview-classic.md)。
> 

如果熟悉经典部署模型，请参阅[经典部署与 Resource Manager 之间的 IP 寻址差异](virtual-network-ip-addresses-overview-classic.md#differences-between-resource-manager-and-classic-deployments)。

## <a name="public-ip-addresses"></a>公共 IP 地址

公共 IP 地址可让 Azure 资源与 Internet 以及面向公众的 Azure 服务（例如 [Azure Redis 缓存](https://azure.microsoft.com/services/cache)、[Azure 事件中心](https://azure.microsoft.com/services/event-hubs)、[SQL 数据库](../sql-database/sql-database-technical-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)和 [Azure 存储](../storage/common/storage-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json)）通信。

在 Azure 资源管理器中，[公共 IP](virtual-network-public-ip-address.md) 地址是具有其自身属性的资源。 可与公共 IP 地址资源关联的部分资源包括：

* 虚拟机网络接口
* 面向 Internet 的负载均衡器
* VPN 网关
* 应用程序网关数

### <a name="ip-address-version"></a>IP 地址版本

公共 IP 地址是使用 IPv4 或 IPv6 地址创建的。 公共 IPv6 地址只能分配到面向 Internet 的负载均衡器。

### <a name="sku"></a>SKU

使用以下 SKU 之一创建公共 IP 地址：

#### <a name="basic"></a>基本

推出 SKU 之前创建的所有公共 IP 地址为基本 SKU 公共 IP 地址。 随着 SKU 的推出，可以选择指定公共 IP 地址要采用的 SKU。 基本 SKU 地址为：

- 使用静态或动态分配方法分配。
- 分配到可以采用公共 IP 地址的任何 Azure 资源，例如网络接口、VPN 网关、应用程序网关和面向 Internet 的负载均衡器。
- 可以分配到特定的区域。
- 非区域冗余。 若要详细了解可用性区域，请参阅[可用性区域概述](../availability-zones/az-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

#### <a name="standard"></a>标准

标准 SKU 公共 IP 地址为：

- 只能使用静态分配方法分配。
- 分配到网络接口或面向 Internet 的标准负载均衡器。 有关 Azure 负载均衡器 SKU 的详细信息，请参阅 [Azure 负载均衡器标准 SKU](../load-balancer/load-balancer-standard-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。
- 默认为区域冗余。 可以按区域创建，在特定的可用性区域中有保障。  若要详细了解可用性区域，请参阅[可用性区域概述](../availability-zones/az-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。
 
> [!NOTE]
> 将标准 SKU 公共 IP 地址分配到虚拟机的网络接口时，必须使用[网络安全组](security-overview.md#network-security-groups)显式允许预期流量。  创建并关联网络安全组以及显式允许所需流量后与资源的通信故障才会结束。

标准 SKU 以预览版提供。 创建标准 SKU 公共 IP 地址之前，必须先注册预览版，并在支持的位置创建该地址。 若要注册预览版，请参阅[注册标准 SKU 预览版](virtual-network-public-ip-address.md#register-for-the-standard-sku-preview)。 有关支持的位置（区域）的列表，请参阅[区域可用性](../load-balancer/load-balancer-standard-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json#region-availability)；有关其他区域支持，请密切关注 [Azure 虚拟网络更新](https://azure.microsoft.com/updates/?product=virtual-network)页。


### <a name="allocation-method"></a>分配方法

将 IP 地址分配到公共 IP 地址资源有两种方法 - 动态或静态。 默认分配方法为*动态*，即**不是**在创建时分配的 IP 地址。 公共 IP 地址是在启动（或创建）关联的资源（例如 VM 或负载均衡器）时分配的。 停止（或删除）该资源时，就会释放该 IP 地址。 例如，从资源 A 中释放后，可将该 IP 地址分配到不同的资源。 如果在停止资源 A 的情况下将 IP 地址分配到不同的资源，则重启资源 A 时，会分配一个不同的 IP 地址。

要确保所关联资源的 IP 地址保持不变，可将分配方法显式设置为*静态*。 静态 IP 地址是立即分配的。 只有在删除该资源或将其分配方法更改为“动态”时，才会释放该地址。

> [!NOTE]
> 即使将分配方法设置为“静态”，也无法通过指定方式将实际 IP 地址分配到公共 IP 地址资源。 Azure 会从创建资源时所在的 Azure 位置的可用 IP 地址池中分配 IP 地址。
>

以下情况通常使用静态公共 IP 地址：

* 必须更新防火墙规则才能与 Azure 资源通信。
* 对 DNS 名称进行解析时，如果更改了 IP 地址，则需更新 A 记录。
* Azure 资源与其他使用 IP 地址型安全模型的应用或服务通信。
* 使用链接到 IP 地址的 SSL 证书。

> [!NOTE]
> Azure 会从每个 Azure 区域的唯一地址范围中分配公共 IP 地址。 有关详细信息，请参阅 [Azure 数据中心 IP 范围](https://www.microsoft.com/download/details.aspx?id=41653)。
>

### <a name="dns-hostname-resolution"></a>DNS 主机名解析
可以为公共 IP 资源指定一个 DNS 域名标签，以便在 Azure 托管的 DNS 服务器中为 *domainnamelabel*.*location*.cloudapp.azure.com 创建目标为公共 IP 地址的映射。 例如，如果在创建公共 IP 资源时将 *domainnamelabel* 指定为 **contoso**，将 Azure 的“位置”指定为“美国西部”，则会将完全限定域名 (FQDN) **contoso.westus.cloudapp.azure.com** 解析成该资源的公共 IP 地址。 可以使用 FQDN 创建指向 Azure 中的公共 IP 地址的自定义域 CNAME 记录。

> [!IMPORTANT]
> 所创建的每个域名标签在其 Azure 位置中必须是唯一的。  
>

### <a name="virtual-machines"></a>虚拟机

将公共 IP 地址分配到其**网络接口**可将其与 [Windows](../virtual-machines/windows/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 或 [Linux](../virtual-machines/linux/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 虚拟机相关联。 可以向虚拟机分配动态或静态公共 IP 地址。 详细了解如何[将 IP 地址分配到网络接口](virtual-network-network-interface-addresses.md)。

### <a name="internet-facing-load-balancers"></a>面向 Internet 的负载均衡器

可将通过任一 [SKU](#SKU) 创建的公共 IP 地址与 [Azure 负载均衡器](../load-balancer/load-balancer-overview.md)相关联，只需将其分配给负载均衡器**前端**配置即可。 此公共 IP 地址充当负载均衡型虚拟 IP 地址 (VIP)。 可以向负载均衡器前端分配动态或静态公共 IP 地址。 还可以向负载均衡器前端分配多个公共 IP 地址，这会启用[多 VIP](../load-balancer/load-balancer-multivip-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 方案，如包含基于 SSL 的网站的多租户环境。 有关 Azure 负载均衡器 SKU 的详细信息，请参阅 [Azure 负载均衡器标准 SKU](../load-balancer/load-balancer-standard-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

### <a name="vpn-gateways"></a>VPN 网关

[Azure VPN 网关](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%2ftoc.json)将 Azure 虚拟网络连接到其他 Azure 虚拟网络或本地网络。 需将公共 IP 地址分配到 VPN 网关才能与远程网络通信。 只能向 VPN 网关分配动态公共 IP 地址。

### <a name="application-gateways"></a>应用程序网关数

将公共 IP 地址分配给网关的**前端**配置可以将其与 Azure [应用程序网关](../application-gateway/application-gateway-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json)相关联。 此公共 IP 地址充当负载均衡型 VIP。 只能将动态公共 IP 地址分配给应用程序网关前端配置。

### <a name="at-a-glance"></a>概览
下表显示了将公共 IP 地址关联到顶级资源时所依据的特定属性，以及能够使用的可能分配方法（动态或静态）。

| 顶级资源 | IP 地址关联 | 动态 | 静态 |
| --- | --- | --- | --- |
| 虚拟机 |网络接口 |是 |是 |
| 面向 Internet 的负载均衡器 |前端配置 |是 |是 |
| VPN 网关 |网关 IP 配置 |是 |否 |
| 应用程序网关 |前端配置 |是 |否 |

## <a name="private-ip-addresses"></a>专用 IP 地址
专用 IP 地址能够让 Azure 资源在不使用可访问 Internet 的 IP 地址的情况下，与[虚拟网络](virtual-networks-overview.md)或本地网络中的其他资源（通过 VPN 网关或 ExpressRoute 线路）通信。

在 Azure 资源管理器部署模型中，可将专用 IP 地址关联到以下类型的 Azure 资源：

* 虚拟机网络接口
* 内部负载均衡器 (ILB)
* 应用程序网关数

### <a name="ip-address-version"></a>IP 地址版本

专用 IP 地址是使用 IPv4 或 IPv6 地址创建的。 只能使用动态分配方法分配专用 IPv6 地址。 无法在虚拟网络上的专用 IPv6 地址之间通信。 可以通过面向 Internet 的负载均衡器，从 Internet 与专用 IPv6 地址进行入站通信。 有关详细信息，请参阅[使用 IPv6 创建面向 Internet 的负载均衡器](../load-balancer/load-balancer-ipv6-internet-ps.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

### <a name="allocation-method"></a>分配方法

可以根据资源所部署到的虚拟网络子网的地址范围来分配专用 IP 地址。 分配专用 IP 地址有两种方法：

- **动态**：Azure 保留每个子网地址范围中的前四个地址，不分配这些地址。 Azure 将下一个可用的地址分配给子网地址范围中的资源。 例如，如果子网的地址范围为 10.0.0.0/16，且地址 10.0.0.0.4-10.0.0.14 已分配（.0-.3 为保留地址），则 Azure 会将 10.0.0.15 分配给资源。 动态方法是默认的分配方法。 动态 IP 地址在分配后，仅在以下情况下才会释放：网络接口已删除、已分配到同一虚拟网络中的另一子网，或者分配方法已更改为静态，这种情况下会指定另一 IP 地址。 默认情况下，当分配方法从动态更改为静态时，Azure 会将以前的动态分配的地址作为静态地址分配。
- **静态**：由你选择并分配子网地址范围中的地址。 分配的地址可以是子网地址范围中的任何地址，前提是该地址不是子网地址范围中的头四个地址，也不是当前已分配给子网中任何其他资源的地址。 只有在删除网络接口之后，静态地址才会释放。 如果将分配方法更改为动态，Azure 会动态地将以前分配的静态 IP 地址作为动态地址分配，即使该地址是子网地址范围中的下一个可用地址。 如果将网络接口分配给同一虚拟网络中的另一子网，则该地址也会更改。但是，若要将网络接口分配给另一子网，必须先将分配方法从静态更改为动态。 将网络接口分配给另一子网以后，即可将分配方法改回为静态，并根据新子网的地址范围分配 IP 地址。

### <a name="virtual-machines"></a>虚拟机

可将一个或多个专用 IP 地址分配给 [Windows](../virtual-machines/windows/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 或 [Linux](../virtual-machines/linux/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 虚拟机的一个或多个**网络接口**。 可将每个专用 IP 地址的分配方法指定为动态或静态。

#### <a name="internal-dns-hostname-resolution-for-virtual-machines"></a>内部 DNS 主机名解析（针对虚拟机）

所有 Azure 虚拟机都默认配置了 [Azure 托管的 DNS 服务器](virtual-networks-name-resolution-for-vms-and-role-instances.md#azure-provided-name-resolution)，除非显式配置了自定义 DNS 服务器。 这些 DNS 服务器为驻留在同一个虚拟网络内的虚拟机提供内部名称解析。

创建虚拟机时，主机名到其专用 IP 地址的映射将添加到 Azure 托管的 DNS 服务器。 如果虚拟机有多个网络接口，或者一个网络接口有多个 IP 配置，主机名会映射到主要网络接口的主要 IP 配置的专用 IP 地址。

使用 Azure 托管的 DNS 服务器配置的虚拟机可将同一虚拟网络中的所有虚拟机的主机名解析为其专用 IP 地址。 若要在连接的虚拟网络中解析虚拟机的主机名，必须使用自定义 DNS 服务器。

### <a name="internal-load-balancers-ilb--application-gateways"></a>内部负载均衡器 (ILB) 和应用程序网关

可以将专用 IP 地址分配到 [Azure 内部负载均衡器](../load-balancer/load-balancer-internal-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (ILB) 或 [Azure 应用程序网关](../application-gateway/application-gateway-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json)的**前端**配置。 此专用 IP 地址将用作内部终结点，仅供其虚拟网络和连接到该虚拟网络的远程网络中的资源访问。 可将动态或静态专用 IP 地址分配到前端配置。

### <a name="at-a-glance"></a>概览
下表显示了将专用 IP 地址关联到顶级资源时所依据的特定属性，以及能够使用的可能分配方法（动态或静态）。

| 顶级资源 | IP 地址关联 | 动态 | 静态 |
| --- | --- | --- | --- |
| 虚拟机 |网络接口 |是 |是 |
| 负载均衡 |前端配置 |是 |是 |
| 应用程序网关 |前端配置 |是 |是 |

## <a name="limits"></a>限制
Azure 中的[网络限制](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits)全面阐述了对 IP 寻址施加的限制。 这些限制根据区域和订阅设置。 可以[与支持人员联系](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade)，根据业务需求将默认限制提高到最大限制。

## <a name="pricing"></a>定价
公共 IP 地址可能会产生少许费用。 有关 Azure 中 IP 地址定价的详细信息，请阅读 [IP 地址定价](https://azure.microsoft.com/pricing/details/ip-addresses)页。

## <a name="next-steps"></a>后续步骤
* [使用 Azure 门户通过静态公共 IP 部署 VM](virtual-network-deploy-static-pip-arm-portal.md)
* [使用模板通过静态公共 IP 部署 VM](virtual-network-deploy-static-pip-arm-template.md)
* [通过 Azure 门户使用静态专用 IP 地址部署 VM](virtual-networks-static-private-ip-arm-pportal.md)
