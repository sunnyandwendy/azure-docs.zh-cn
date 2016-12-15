---
title: "Operations Management Suite (OMS) 管理解决方案中的视图 | Microsoft 文档"
description: "Operations Management Suite (OMS) 中的管理解决方案中通常包括一个或多个用于可视化数据的视图。  本文介绍如何导出视图设计器所创建的视图，并将其包含在管理解决方案中。 "
services: operations-management-suite
documentationcenter: 
author: bwren
manager: jwhit
editor: tysonn
ms.assetid: 570b278c-2d47-4e5a-9828-7f01f31ddf8c
ms.service: operations-management-suite
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/17/2016
ms.author: bwren
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: ae4e75c7ac318a414672cf791bb35d1615f45dea


---
# <a name="views-in-operations-management-suite-oms-management-solutions-preview"></a>Operations Management Suite (OMS) 管理解决方案中的视图（预览版）
> [!NOTE]
> 这是在 OMS 中创建管理解决方案的初步文档，当前仅提供预览版。 如下所述的全部架构均会有变动。    
> 
> 

[Operations Management Suite (OMS) 中的管理解决方案](operations-management-suite-solutions.md)通常包括一个或多个用于可视化数据的视图。  本文介绍如何导出[视图设计器](../log-analytics/log-analytics-view-designer.md)所创建的视图，并将其包含在管理解决方案中。  

> [!NOTE]
> 本文中的示例使用管理解决方案需要或通用的参数和变量，[在 Operations Management Suite (OMS) 中创建管理解决方案](operations-management-suite-solutions-creating.md)对此进行了介绍 
> 
> 

## <a name="prerequisites"></a>先决条件
本文假设你已经熟悉如何[创建管理解决方案](operations-management-suite-solutions-creating.md)及解决方案文件的结构。

## <a name="overview"></a>概述
若要在管理解决方案中包含视图，则需要在[解决方案文件](operations-management-suite-solutions-creating.md)中为其创建**资源**。  描述视图详细配置的 JSON 通常很复杂，普通的解决方案作者无法手动进行创建。  最常见方法是使用 [视图设计器](../log-analytics/log-analytics-view-designer.md)创建视图，并将其导出，然后再将其详细配置添加到解决方案。 

将视图添加到解决方案的基本步骤如下所示。  每个步骤都在后续相应部分进行了详细介绍。

1. 将视图导出到文件。
2. 在解决方案中创建视图资源。
3. 添加视图详细信息。

## <a name="export-the-view-to-a-file"></a>将视图导出到文件
按照[Log Analytics 视图设计器](../log-analytics/log-analytics-view-designer.md)中的说明将视图导出到文件。  导出的文件是[与解决方案文件具有相同元素的 JSON 格式](operations-management-suite-solutions-creating.md#management-solution-files)。  

视图文件的 **resources** 元素将具有表示 OMS 工作区的 **Microsoft.OperationalInsights/workspaces** 类型的资源。  此元素将具有表示此视图的 **views** 类型的子元素，并包含其详细配置。  复制此元素的详细信息，然后将其复制到解决方案中。

## <a name="create-the-view-resource-in-the-solution"></a>在解决方案中创建视图资源
将以下视图将资源添加到解决方案文件的 **resources** 元素。  这将使用以下描述的必须同时添加的变量。  注意：**Dashboard** 和 **OverviewTile** 属性是占位符，你将使用导出的视图文件中的相应属性对其进行覆盖。

    {
        "apiVersion": "[variables('LogAnalyticsApiVersion')]",
        "name": "[concat(parameters('workspaceName'), '/', variables('ViewName'))]",
        "type": "Microsoft.OperationalInsights/workspaces/views",
        "location": "[parameters('workspaceregionId')]",
        "id": "[Concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.OperationalInsights/workspaces/', parameters('workspaceName'),'/views/', variables('ViewName'))]",
        dependson": [
            ],
        "properties": {
            "Id": "[variables('ViewName')]",
            "Name": "[variables('ViewName')]",
            "DisplayName": "[variables('ViewName')]",
            "Description": "",
            "Author": "[variables('ViewAuthor')]",
            "Source": "Local",
            "Dashboard": ,
            "OverviewTile": 
        }
    }

将以下变量添加到解决方案文件的 [variables](operations-management-suite-solutions-creating.md#variables) 元素，并将值替换为你的解决方案的值。

    "LogAnalyticsApiVersion": "2015-11-01-preview",
    "ViewAuthor": "Your name."
    "ViewDescription": "Optional description of the view."
    "ViewName": "Provide a name for the view here."


注意：你可以从导出的视图文件复制整个视图资源，但需要对其进行以下更改才能在解决方案中生效。  

* 视图资源的**类型**需要从**视图**更改为 **Microsoft.OperationalInsights/workspaces**。
* 视图资源的**名称**属性需要更改为包括工作区名称。
* 工作区中的依赖关系需要删除，因为并未在解决方案中定义工作区资源。
* 需要将 **DisplayName** 属性添加到视图。  **Id**、**名称** 和 **DisplayName** 必须完全匹配。
* 必须更改参数名称，以匹配所需的参数集。
* 变量应在解决方案中进行定义，并在适当的属性中使用。

## <a name="add-the-view-details"></a>添加视图详细信息
导出的视图文件中的视图资源将在 **properties** 属性中包含两个元素，名称分别为 **Dashboard** 和 **OverviewTile**，它们包含视图的详细配置。  将这两个元素及其内容复制解决方案文件中的视图资源的 **properties** 元素。 

## <a name="example"></a>示例
例如，下面的示例演示包含视图的简单解决方案文件。  由于空间原因，将显示省略号 (...) 以表示 **Dashboard** 和 **OverviewTile** 内容。

    {
        "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "workspaceName": {
                "type": "string"
            },
            "accountName": {
                "type": "string"
            },
            "workspaceRegionId": {
                "type": "string"
            },
            "regionId": {
                "type": "string"
            },
            "pricingTier": {
                "type": "string"
            }
        },
        "variables": {
            "SolutionVersion": "1.1",
            "SolutionPublisher": "Contoso",
            "SolutionName": "Contoso Solution",
            "LogAnalyticsApiVersion": "2015-11-01-preview",
            "ViewAuthor":  "user@contoso.com",
            "ViewDescription":  "This is a sample view.",
            "ViewName":  "Contoso View"
        },
        "resources": [
            {
                "name": "[concat(variables('SolutionName'), '(' ,parameters('workspacename'), ')')]",
                "location": "[parameters('workspaceRegionId')]",
                "tags": { },
                "type": "Microsoft.OperationsManagement/solutions",
                "apiVersion": "[variables('LogAnalyticsApiVersion')]",
                "dependsOn": [
                    "[concat('Microsoft.OperationalInsights/workspaces/', parameters('workspacename'), '/views/', variables('ViewName'))]"
                ],
                "properties": {
                    "workspaceResourceId": "[concat(resourceGroup().id, '/providers/Microsoft.OperationalInsights/workspaces/', parameters('workspacename'))]",
                    "referencedResources": [
                    ],
                    "containedResources": [
                        "[concat('Microsoft.OperationalInsights/workspaces/', parameters('workspaceName'), '/views/', variables('ViewName'))]"
                    ]
                },
                "plan": {
                    "name": "[concat(variables('SolutionName'), '(' ,parameters('workspaceName'), ')')]",
                    "Version": "[variables('SolutionVersion')]",
                    "product": "ContosoSolution",
                    "publisher": "[variables('SolutionPublisher')]",
                    "promotionCode": ""
                }
            },
            {
                "apiVersion": "[variables('LogAnalyticsApiVersion')]",
                "name": "[concat(parameters('workspaceName'), '/', variables('ViewName'))]",
                "type": "Microsoft.OperationalInsights/workspaces/views",
                "location": "[parameters('workspaceregionId')]",
                "id": "[Concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.OperationalInsights/workspaces/', parameters('workspaceName'),'/views/', variables('ViewName'))]",
                "dependson": [
                ],
                "properties": {
                    "Id": "[variables('ViewName')]",
                    "Name": "[variables('ViewName')]",
                    "DisplayName": "[variables('ViewName')]",
                    "Description": "[variables('ViewDescription')]",
                    "Author": "[variables('ViewAuthor')]",
                    "Source": "Local",
                    "Dashboard": ...,
                    "OverviewTile": ...
                }
            }
          ]
    }




## <a name="next-steps"></a>后续步骤
* 了解创建[管理解决方案](operations-management-suite-solutions-creating.md)的完整详细信息。
* 包括[管理解决方案中的自动化 runbook](operations-management-suite-solutions-resources-automation.md)。




<!--HONumber=Nov16_HO3-->

