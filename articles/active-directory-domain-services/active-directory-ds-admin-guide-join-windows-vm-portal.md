---
title: "Azure Active Directory 域服务：将 Windows Server VM 加入托管域 | Microsoft 文档"
description: "将 Windows Server 虚拟机加入 Azure AD 域服务"
services: active-directory-ds
documentationcenter: 
author: mahesh-unnikrishnan
manager: mahesh-unnikrishnan
editor: curtand
ms.assetid: 29316313-c76c-4fb9-8954-5fa5ec82609e
ms.service: active-directory-ds
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/19/2017
ms.author: maheshu
ms.openlocfilehash: 5f661dba2e647ac905e7d84927fdbf6dbc76094f
ms.sourcegitcommit: 963e0a2171c32903617d883bb1130c7c9189d730
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/20/2017
---
# <a name="join-a-windows-server-virtual-machine-to-a-managed-domain"></a>将 Windows Server 虚拟机加入托管域
本文展示了如何使用 Azure 门户部署 Windows Server 虚拟机。 它还展示了如何将虚拟机加入 Azure AD 域服务托管域。

## <a name="step-1-create-the-windows-server-virtual-machine"></a>步骤 1：创建 Windows Server 虚拟机
执行以下步骤创建 Windows 虚拟机，然后将其加入已在其中启用了 Azure AD 域服务的虚拟网络。

1. 通过 [http://portal.azure.com](http://portal.azure.com) 登录到 Azure 门户。
2. 单击 Azure 门户左上角的“新建”按钮。
3. 选择“计算”，然后选择“Windows Server 2016 Datacenter”。

    ![选择映像](./media/active-directory-domain-services-admin-guide/create-windows-vm-select-image.png)
4. 在向导的“基本信息”页面上配置虚拟机的基本设置。

    ![配置 VM 的基本设置](./media/active-directory-domain-services-admin-guide/create-windows-vm-basics.png)

    > [!TIP]
    > 此处输入的用户名和密码是用来登录到虚拟机的本地管理员帐户的。 请选取一个强密码来保护虚拟机，使其免受密码暴力攻击的破坏。 请不要输入域用户帐户的凭据。
    >

5. 选择虚拟机的大小。 若要查看更多的大小，请选择“全部查看”或更改“支持的磁盘类型”筛选器。

    ![选择 VM 大小](./media/active-directory-domain-services-admin-guide/create-windows-vm-size.png)

6. 在向导的“设置”页面上，选择在其中部署 Azure AD 域服务托管域的虚拟网络。 请选取与你的托管域部署到的子网不同的一个子网。 对于其他设置，保留默认值，然后单击“确定”。

    ![为虚拟机选择虚拟网络](./media/active-directory-domain-services-admin-guide/create-windows-vm-select-vnet.png)

    > [!TIP]
    > **选取正确的虚拟网络和子网。**
    > 选择在其中部署你的托管域的虚拟网络或通过虚拟网络对等互连连接到它的虚拟网络。 如果选择未连接的虚拟网络，则无法将虚拟机加入到托管域。
    > 建议将托管域部署到一个专用子网中。 因此，不要选取已在其中启用了托管域的子网。

7. 在“购买”页面上，复查设置并单击“确定”以部署虚拟网络。
8. VM 部署将固定到 Azure 门户仪表板。

    ![完成](./media/active-directory-domain-services-admin-guide/create-windows-vm-done.png)
9. 部署完成后，可以在“概述”页面中看到关于 VM 的信息。


## <a name="step-2-connect-to-the-windows-server-virtual-machine-using-the-local-administrator-account"></a>步骤 2：使用本地管理员帐户连接到 Windows Server 虚拟机
现在，连接到新建的 Windows Server 虚拟机，以便将其加入域。 请使用在创建虚拟机时指定的本地管理员凭据。

执行以下步骤连接到虚拟机。

1. 单击“概述”页面上的“连接”按钮。 此时会创建并下载远程桌面协议 (.rdp) 文件。

    ![连接到 Windows 虚拟机](./media/active-directory-domain-services-admin-guide/connect-windows-vm.png)
2. 若要连接到 VM，请打开下载的 RDP 文件。 出现提示时，请单击“连接”。
3. 在登录提示中，输入创建虚拟机时指定的**本地管理员凭据**。 例如，本示例使用的是“localhost\mahesh”。
4. 你可能会在登录过程中收到证书警告。 单击“是”或“继续”以继续连接。

此时，应已使用本地管理员凭据登录到新建的 Windows 虚拟机。 下一步是将该虚拟机加入域。


## <a name="step-3-join-the-windows-server-virtual-machine-to-the-aad-ds-managed-domain"></a>步骤 3：将 Windows Server 虚拟机加入 AAD-DS 托管域
执行以下步骤，将 Windows Server 虚拟机加入 AAD-DS 托管域。

1. 如步骤 2 中所述连接到 Windows Server。 在“开始”屏幕中打开“服务器管理器”。
2. 在“服务器管理器”窗口的左窗格中单击“本地服务器”。

    ![在虚拟机上启动服务器管理器](./media/active-directory-domain-services-admin-guide/join-domain-server-manager.png)
3. 单击“属性”部分下面的“工作组”。 在“系统属性”属性页中，单击“更改”加入域。

    ![“系统属性”页](./media/active-directory-domain-services-admin-guide/join-domain-system-properties.png)
4. 在“域”文本框中指定 Azure AD 域服务托管域的域名，并单击“确定”。

    ![指定要加入的域](./media/active-directory-domain-services-admin-guide/join-domain-system-properties-specify-domain.png)
5. 系统会提示输入用于加入域的凭据。 请务必**指定属于 AAD DC 管理员组的用户的凭据**。 只有此组的成员才有权将计算机加入托管域。

    ![指定用于加入域的凭据](./media/active-directory-domain-services-admin-guide/join-domain-system-properties-specify-credentials.png)
6. 可以通过以下方式之一指定凭据：

   * **UPN 格式：（推荐）**指定在 Azure AD 中为用户帐户配置的 UPN 后缀。 在本示例中，用户“bob”的 UPN 后缀为“bob@domainservicespreview.onmicrosoft.com”。
   * **SAMAccountName 格式：**可以使用 SAMAccountName 格式指定帐户名。 在本示例中，用户“bob”需要输入“CONTOSO100\bob”。

     > [!TIP]
     > **建议使用 UPN 格式指定凭据。**
     > 如果用户的 UPN 前缀过长（例如“joehasareallylongname”），系统可能会自动生成 SAMAccountName。 如果 Azure AD 租户中有多个用户具有相同的 UPN 前缀（例如“bob”），服务可能会自动生成其 SAMAccountName 格式。 在这种情况下，使用 UPN 格式能够可靠地登录域。
     >

7. 成功加入域后，将看到以下欢迎加入域的消息。 请重新启动虚拟机完成域加入操作。

    ![欢迎加入域](./media/active-directory-domain-services-admin-guide/join-domain-done.png)


## <a name="troubleshooting-domain-join"></a>排查域加入问题
### <a name="connectivity-issues"></a>连接问题
如果虚拟机找不到域，请参阅以下故障排除步骤：

* 确保虚拟机已连接到已在其中启用域服务的同一个虚拟网络。 否则，虚拟机无法连接到域，因此也就无法加入该域。
* 请确保虚拟机位于某个虚拟网络中而后者又连接到已在其中启用了域服务的虚拟网络。
* 尝试使用托管域的域名来 ping 域（例如“ping contoso100.com”）。 如果无法执行此操作，请尝试 ping 页面中显示的、已在其中启用 Azure AD 域服务的域的 IP 地址（例如“ping 10.0.0.4”）。 如果能够 ping 通该 IP 地址，但无法 ping 通域，则表示 DNS 的配置可能不正确。 检查是否已将域的 IP 地址配置为虚拟网络的 DNS 服务器。
* 请尝试刷新虚拟机上的 DNS 解析程序缓存（“ipconfig /flushdns”）。

如果出现请求提供凭据来加入域的对话框，则表示没有连接问题。

### <a name="credentials-related-issues"></a>与凭据相关的问题
如果凭据有问题，因而无法加入域，请参考以下步骤。

* 尝试使用 UPN 格式指定凭据。 如果租户中有多个用户具有相同的 UPN 前缀，或者 UPN 前缀过长，系统可能会自动生成帐户的 SAMAccountName。 因此，帐户的 SAMAccountName 格式可能不同于所需的格式或者在本地域中使用的格式。
* 请尝试使用属于“AAD DC 管理员”组的用户帐户的凭据。
* 确保根据《入门指南》中所述的步骤[启用密码同步](active-directory-ds-getting-started-password-sync.md)。
* 确保使用 Azure AD 中配置的用户的 UPN（例如，“bob@domainservicespreview.onmicrosoft.com”）登录。
* 确保根据《入门指南》中所述，等待足够长的时间来完成密码同步。

## <a name="related-content"></a>相关内容
* [Azure AD 域服务 - 入门指南](active-directory-ds-getting-started.md)
* [管理受 Azure AD 域服务管理的域](active-directory-ds-admin-guide-administer-domain.md)
