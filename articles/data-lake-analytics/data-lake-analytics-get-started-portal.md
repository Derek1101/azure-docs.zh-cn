---
title: "通过 Azure 门户开始使用 Azure Data Lake Analytics | Microsoft Docs"
description: "了解如何使用 Azure 门户创建 Data Lake Analytics 帐户，使用 U-SQL 创建 Data Lake Analytics 作业，并提交该作业。 "
services: data-lake-analytics
documentationcenter: 
author: edmacauley
manager: jhubbard
editor: cgronlun
ms.assetid: b1584d16-e0d2-4019-ad1f-f04be8c5b430
ms.service: data-lake-analytics
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 03/21/2017
ms.author: edmaca
ms.openlocfilehash: 2722a2d72ed90ea0005362563ecaee30750c040a
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/11/2017
---
# <a name="get-started-with-azure-data-lake-analytics-using-azure-portal"></a>通过 Azure 门户开始使用 Azure Data Lake Analytics
[!INCLUDE [get-started-selector](../../includes/data-lake-analytics-selector-get-started.md)]

了解如何使用 Azure 门户创建 Azure Data Lake Analytics 帐户，在 [U-SQL](data-lake-analytics-u-sql-get-started.md)中定义作业，并将作业提交到 Data Lake Analytics 服务。 有关 Data Lake Analytics 的详细信息，请参阅 [Azure Data Lake Analytics 概述](data-lake-analytics-overview.md)。

## <a name="prerequisites"></a>先决条件

在开始学习本教程之前，必须有一个 Azure 订阅。 请参阅 [获取 Azure 免费试用版](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="create-a-data-lake-analytics-account"></a>创建 Data Lake Analytics 帐户

现在可同时创建 Data Lake Analytics 帐户和 Data Lake Store 帐户。  此步骤很简单，只需大约 60 秒即可完成。

1. 登录到 [Azure 门户](https://portal.azure.com)。
2. 单击“新建” >  “数据 + 分析” > “Data Lake Analytics”。
3. 为以下项选择值：
   * **名称**：为 Data Lake Analytics 帐户命名（只允许小写字母和数字）。
   * **订阅**：选择用于 Analytics 帐户的 Azure 订阅。
   * **资源组**。 选择现有的 Azure 资源组或创建新的资源组。
   * **位置**。 为 Data Lake Analytics 帐户选择 Azure 数据中心。
   * Data Lake Store：按照说明创建新的 Data Lake Store 帐户，或选择现有帐户。 
4. （可选）为 Data Lake Analytics 帐户选择定价层。
5. 单击“创建” 。 


## <a name="your-first-u-sql-script"></a>第一个 U-SQL 脚本

以下文本是一个非常简单的 U-SQL 脚本。 它仅是在脚本中定义一个小型数据集，然后将该数据集写出到默认 Data Lake Store 中，并写为名为 `/data.csv` 的文件。

```
@a  = 
    SELECT * FROM 
        (VALUES
            ("Contoso", 1500.0),
            ("Woodgrove", 2700.0)
        ) AS 
              D( customer, amount );
OUTPUT @a
    TO "/data.csv"
    USING Outputters.Csv();
```

## <a name="submit-a-u-sql-job"></a>提交 U-SQL 作业

1. 从 Data Lake Analytics 帐户单击“新建作业”。
2. 粘贴在上面所示的 U-SQL 脚本文本中。 
3. 单击“提交”。   
4. 请等到作业状态变为“成功”。
5. 如果作业失败，请参阅[对 Data Lake Analytics 作业进行监视和故障排除](data-lake-analytics-monitor-and-troubleshoot-jobs-tutorial.md)。
6. 单击“输出”选项卡，然后单击 `data.csv`。 

## <a name="see-also"></a>另请参阅

* 若要着手开发 U-SQL 应用程序，请参阅 [使用 Data Lake Tools for Visual Studio 开发 U-SQL 脚本](data-lake-analytics-data-lake-tools-get-started.md)。
* 若要了解 U-SQL，请参阅 [Azure Data Lake Analytics U-SQL 语言入门](data-lake-analytics-u-sql-get-started.md)。
* 有关管理任务，请参阅 [Manage Azure Data Lake Analytics using Azure Portal](data-lake-analytics-manage-use-portal.md)（使用 Azure 门户管理 Azure Data Lake Analytics）。
