---
title: "创建 Azure Service Fabric Windows 容器应用程序 | Microsoft Docs"
description: "在本快速入门中，请在 Azure Service Fabric 上创建第一个 Windows 容器应用程序。"
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: vturecek
ms.assetid: 
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: quickstart
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 02/27/18
ms.author: ryanwi
ms.custom: mvc
ms.openlocfilehash: 7a8d28ef842ba77355628c79c20fa7fd3c693380
ms.sourcegitcommit: c765cbd9c379ed00f1e2394374efa8e1915321b9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/28/2018
---
# <a name="quickstart-deploy-a-service-fabric-windows-container-application-on-azure"></a>快速入门：在 Azure 上部署 Service Fabric Windows 容器应用程序
Azure Service Fabric 是一款分布式系统平台，可用于部署和管理可缩放的可靠微服务和容器。 

在 Service Fabric 群集上运行 Windows 容器中的现有应用程序不需要对应用程序进行任何更改。 本快速入门介绍如何在 Service Fabric 应用程序中部署预建的 Docker 容器映像。 完成后，你会有一个正在运行的 Windows Server 2016 Nano Server 和 IIS 容器。 本快速入门介绍如何部署 Windows 容器。若要部署 Linux 容器，请阅读[此快速入门](service-fabric-quickstart-containers-linux.md)。

![IIS default web page][iis-default]

可以通过此快速入门了解如何：
> [!div class="checklist"]
> * 打包 Docker 映像容器
> * 配置通信
> * 生成并打包 Service Fabric 应用程序
> * 将容器应用程序部署到 Azure

## <a name="prerequisites"></a>先决条件
* 一个 Azure 订阅（可以创建[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)）。
* 一台运行以下软件的开发计算机：
  * Visual Studio 2015 或 Visual Studio 2017。
  * [Service Fabric SDK 和工具](service-fabric-get-started.md)。

## <a name="package-a-docker-image-container-with-visual-studio"></a>使用 Visual Studio 打包 Docker 映像容器
Service Fabric SDK 和工具提供服务模板，用于将容器部署到 Service Fabric 群集。

以“管理员”身份启动 Visual Studio。  选择“文件” > “新建” > “项目”。

选择“Service Fabric 应用程序”，将其命名为“MyFirstContainer”，并单击“确定”。

从“托管的容器和应用程序”模板中选择“容器”。

在“映像名称”中输入“microsoft/iis:nanoserver”，即 [Windows Server Nano Server 和 IIS 基映像](https://hub.docker.com/r/microsoft/iis/)。 

将服务命名为“MyContainerService”，然后单击“确定”。

## <a name="configure-communication-and-container-port-to-host-port-mapping"></a>配置通信和容器端口到主机端口映射
此服务需要使用终结点进行通信。  就本快速入门来说，容器化服务会在端口 80 上进行侦听。  在解决方案资源管理器中，打开 *MyFirstContainer/ApplicationPackageRoot/MyContainerServicePkg/ServiceManifest.xml*。  在 ServiceManifest.xml 文件中更新现有的 `Endpoint`，然后添加协议、端口和 URI 方案： 

```xml
<Resources>
    <Endpoints>
        <Endpoint Name="MyContainerServiceTypeEndpoint" UriScheme="http" Port="80" Protocol="http"/>
   </Endpoints>
</Resources>
```
提供 `UriScheme` 即可向 Service Fabric 命名服务自动注册容器终结点，确保其可以被发现。 本文末尾提供完整的 ServiceManifest.xml 示例文件。 

配置容器的“端口到主机”端口映射，使端口 80 上针对服务的传入请求映射到容器上的端口 80。  在解决方案资源管理器中打开 *MyFirstContainer/ApplicationPackageRoot/ApplicationManifest.xml*，然后在 `ContainerHostPolicies` 中指定 `PortBinding` 策略。  就本快速入门来说，`ContainerPort` 为 80，`EndpointRef` 为“MyContainerServiceTypeEndpoint”（在服务清单中定义的终结点）。    

```xml
<ServiceManifestImport>
...
  <ConfigOverrides />
  <Policies>
    <ContainerHostPolicies CodePackageRef="Code">
      <PortBinding ContainerPort="80" EndpointRef="MyContainerServiceTypeEndpoint"/>
    </ContainerHostPolicies>
  </Policies>
</ServiceManifestImport>
```

本文末尾提供完整的 ApplicationManifest.xml 示例文件。

## <a name="create-a-cluster"></a>创建群集
若要将应用程序部署到 Azure 中的群集，可以加入合作群集。 合作群集是在 Azure 上托管的、由 Service Fabric 团队运行的免费限时 Service Fabric 群集，任何人都可以在其中部署应用程序及了解平台的情况。 该群集使用单个自签名证书来确保节点到节点和客户端到节点的安全。 

登录并[加入 Windows 群集](http://aka.ms/tryservicefabric)。 通过单击 **PFX** 链接，将 PFX 证书下载到计算机。 证书和**连接终结点**值在以下步骤中使用。

![PFX 和连接终结点](./media/service-fabric-quickstart-containers/party-cluster-cert.png)

在 Windows 计算机上，将 PFX 安装到 *CurrentUser\My* 证书存储中。

```powershell
PS C:\mycertificates> Import-PfxCertificate -FilePath .\party-cluster-873689604-client-cert.pfx -CertStoreLocation Cert:
\CurrentUser\My


  PSParentPath: Microsoft.PowerShell.Security\Certificate::CurrentUser\My

Thumbprint                                Subject
----------                                -------
3B138D84C077C292579BA35E4410634E164075CD  CN=zwin7fh14scd.westus.cloudapp.azure.com
```

请记住以下步骤的指纹。  

## <a name="deploy-the-application-to-azure-using-visual-studio"></a>使用 Visual Studio 将应用程序部署到 Azure
至此，应用程序已准备就绪，可以直接通过 Visual Studio 将它部署到群集了。

在解决方案资源管理器中右键单击“MyFirstContainer”，选择“发布”。 此时，“发布”对话框显示。

将 Party 群集页面中的“连接终结点”复制到“连接终结点”字段。 例如，`zwin7fh14scd.westus.cloudapp.azure.com:19000`。 单击“高级连接参数”，验证连接参数信息。  *FindValue* 和 *ServerCertThumbprint* 值必须与前一步骤中安装的证书的指纹匹配。 

![发布对话框](./media/service-fabric-quickstart-containers/publish-app.png)

单击“发布” 。

群集中的每个应用程序都必须具有唯一名称。  Party 群集是一个公共、共享的环境，但是可能与现有应用程序存在冲突。  如果存在名称冲突，请重命名 Visual Studio 项目并重新部署。

打开浏览器并导航到 http://zwin7fh14scd.westus.cloudapp.azure.com:80。 此时会看到 IIS 默认网页：![IIS 默认网页][iis-default]

## <a name="complete-example-service-fabric-application-and-service-manifests"></a>Service Fabric 应用程序和服务清单的完整示例
以下是用在本快速入门中的完整服务和应用程序清单。

### <a name="servicemanifestxml"></a>ServiceManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="MyContainerServicePkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType.
         The UseImplicitHost attribute indicates this is a guest service. -->
    <StatelessServiceType ServiceTypeName="MyContainerServiceType" UseImplicitHost="true" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <!-- Follow this link for more information about deploying Windows containers to Service Fabric: https://aka.ms/sfguestcontainers -->
      <ContainerHost>
        <ImageName>microsoft/iis:nanoserver</ImageName>
      </ContainerHost>
    </EntryPoint>
    <!-- Pass environment variables to your container: -->
    <!--
    <EnvironmentVariables>
      <EnvironmentVariable Name="VariableName" Value="VariableValue"/>
    </EnvironmentVariables>
    -->
  </CodePackage>

  <!-- Config package is the contents of the Config directoy under PackageRoot that contains an 
       independently-updateable and versioned set of custom configuration settings for your service. -->
  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="MyContainerServiceTypeEndpoint" UriScheme="http" Port="80" Protocol="http"/>
    </Endpoints>
  </Resources>
</ServiceManifest>
```
### <a name="applicationmanifestxml"></a>ApplicationManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest ApplicationTypeName="MyFirstContainerType"
                     ApplicationTypeVersion="1.0.0"
                     xmlns="http://schemas.microsoft.com/2011/01/fabric"
                     xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <Parameters>
    <Parameter Name="MyContainerService_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <!-- Import the ServiceManifest from the ServicePackage. The ServiceManifestName and ServiceManifestVersion 
       should match the Name and Version attributes of the ServiceManifest element defined in the 
       ServiceManifest.xml file. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="MyContainerServicePkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <ContainerHostPolicies CodePackageRef="Code">
        <PortBinding ContainerPort="80" EndpointRef="MyContainerServiceTypeEndpoint"/>
      </ContainerHostPolicies>
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <!-- The section below creates instances of service types, when an instance of this 
         application type is created. You can also create one or more instances of service type using the 
         ServiceFabric PowerShell module.
         
         The attribute ServiceTypeName below must match the name defined in the imported ServiceManifest.xml file. -->
    <Service Name="MyContainerService" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="MyContainerServiceType" InstanceCount="[MyContainerService_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
</ApplicationManifest>
```

## <a name="next-steps"></a>后续步骤
在此快速入门中，读者学习了如何：
> [!div class="checklist"]
> * 打包 Docker 映像容器
> * 配置通信
> * 生成并打包 Service Fabric 应用程序
> * 将容器应用程序部署到 Azure

* 详细了解如何运行 [Service Fabric 上的容器](service-fabric-containers-overview.md)。
* 阅读[在容器中部署 .NET 应用程序](service-fabric-host-app-in-a-container.md)教程。
* 了解 Service Fabric [应用程序生命周期](service-fabric-application-lifecycle.md)。
* 查看 GitHub 上的 [Service Fabric 容器代码示例](https://github.com/Azure-Samples/service-fabric-containers)。

[iis-default]: ./media/service-fabric-quickstart-containers/iis-default.png
[publish-dialog]: ./media/service-fabric-quickstart-containers/publish-dialog.png
