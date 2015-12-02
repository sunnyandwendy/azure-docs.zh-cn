<properties
   pageTitle="Service Fabric 运行状况监视简介"
   description="本文说明了 Azure Service Fabric 运行状况监视模型，包括运行状况实体、报告和评估。"
   services="service-fabric"
   documentationCenter=".net"
   authors="oanapl"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="09/03/2015"
   wacn.date=""/>

# Service Fabric 运行状况监视简介
Service Fabric 引入了一个运行状况模型，该模型提供丰富、灵活且可扩展的运行状况评估和报告。这包括对群集的状态和在其中运行的服务进行附近实时监视。可以轻松地获取运行状况信息，并在潜在问题级联和造成大规模停机之前采取措施对其予以更正。典型的模型是服务基于其本地视图发送报告，并聚合信息，以提供整体的群集级别视图。

Service Fabric 组件使用此运行状况模型报告其当前状态。此外，你可以使用相同的机制报告应用程序中的运行状况。特定于你的自定义条件的运行状况报告的质量和丰富程度将决定你能够针对运行的应用程序检测和修复问题的轻松程度。

> [AZURE.NOTE]我们根据需要为监视的升级启动运行状况子系统。Service Fabric 提供监视的升级，该升级了解如何在没有停机时间、无用户干预最小化，并具有完整的群集和应用程序可用性的情况下升级群集或应用程序。若要执行此操作，升级会基于配置的升级策略检查运行状况，并且仅当运行状况遵从所需的阈值时允许升级继续。否则，升级会自动回滚或暂停，以便让管理员有机会修复问题。若要了解有关应用程序升级的详细信息，请参阅[本文](service-fabric-application-upgrade)。

## 运行状况存储
运行状况存储保留群集中关于实体的运行状况相关信息，以进行轻松地检索和评估。它作为 Service Fabric 保留的有状态服务进行实现，以确保高度可用性和可缩放性。它是 fabric:/System 应用程序的一部分，并且只要群集已启动并正在运行，即可使用。

## 运行状况实体和层次结构
运行状况实体采用逻辑层次结构进行组织，该结构会捕获不同实体之间的交互和依赖项。基于通过 Service Fabric 组件接收的报告，运行状况存储自动构建实体和层次结构。

运行状况实体镜像 Service Fabric 实体（例如：运行状况应用程序实体匹配群集中部署的应用程序实例，运行状况节点实体匹配 Service Fabric 群集节点）。运行状况层次结构捕获系统实体的交互并且是进行高级运行状况评估的基础。你可以通过 [Service Fabric 技术概述](service-fabric-technical-overview.md)了解 Service Fabric 的关键概念。有关应用程序的详细信息，请转到 [Service Fabric 应用程序模型](service-fabric-application-model.md)。

利用运行状况实体和层次结构，你能够有效地报告、调试和监视群集和应用程序。利用运行状况模型，你能够对群集中的许多移动片段的运行状况进行准确而**精细**的表示。

![运行状况实体。][1]
运行状况实体基于父-子关系在层次结构中进行组织。

[1]: ./media/service-fabric-health-introduction/servicefabric-health-hierarchy.png

运行状况实体是：

- **群集**。表示 Service Fabric 群集的运行状况。群集运行状况报告说明影响整个群集并且不能缩小到一个或多个不正常子项的条件。示例：因网络分区或通信问题而导致的群集裂脑。

- **节点**。表示 Service Fabric 节点的运行状况。节点运行状况报告说明影响节点功能并且通常影响在其上运行的所有已部署实体的条件。示例：节点磁盘空间不足（或其他计算机范围属性，如内存、连接等）或节点关闭。节点实体由节点名称（字符串）标识。

- **应用程序**。表示在群集中运行的应用程序实例的运行状况。应用程序运行状况报告说明影响应用程序的整体运行状况并且不能缩小到单个子项（服务或已部署应用程序）的条件。示例：应用程序中不同服务之间的端到端交互。应用程序实体由应用程序名称 (Uri) 标识。

- **服务**。表示在群集中运行的服务的运行状况。服务运行状况报告说明影响服务的整体运行状况并且不能缩小到分区或副本的条件。示例：导致全部分区问题的服务配置（如端口或外部文件共享）。服务实体由服务名称 (Uri) 标识。

- **分区**。表示服务分区的运行状况。分区运行状况报告说明影响整个副本集的条件。示例：副本数目低于目标计数或分区在仲裁丢失中。分区实体由分区 id (Guid) 标识。

- **副本**。表示有状态服务副本或无状态服务实例的运行状况。这是最小的单元监视器并且系统组件可以针对应用程序对其进行报告。示例：对于有状态服务，如果它不能将操作复制到辅助副本或者复制不是按照预期进度进行操作，主要副本则可以报告。如果它耗尽了资源或存在连接问题，无状态实例则可以报告。副本实体由分区 id (Guid) 和副本或实例 id（长型值）标识。

- **DeployedApplication**。表示*在节点上运行的应用程序*的运行状况。已部署应用程序运行状况报告说明特定于节点上的应用程序的条件，该条件不能缩小到部署在同一个节点上的服务包。示例：不能在该节点上下载应用程序包或在节点上设置应用程序安全主体时出现问题。已部署应用程序由应用程序名称 (Uri) 和节点名称（字符串）标识。

- **DeployedServicePackage**。表示在群集节点中运行的应用程序的服务包运行状况。它说明特定于服务包的条件，该条件不会影响同一个应用程序的同一节点上的其他服务包。示例：服务包中的代码包无法启动或配置包无法读取。已部署服务包由应用程序名称 (Uri)、节点名称（字符串）和服务清单名称（字符串）标识。

利用运行状况模型的粒度可以轻松地检测和更正问题。例如，如果服务未响应，则可能报告应用程序实例不正常；但是，由于问题可能不会影响该应用程序中的所有服务，所以它不太理想。应针对不正常的服务申请报告，或者，如果详细信息指向特定子分区，则应针对该分区申请报告。数据将通过层次结构自动呈现：不正常的分区将在服务和应用程序级别显示。这将有助于找出并更快地解决问题的根本原因。

运行状况层次结构由父-子关系组成。群集由节点和应用程序组成；应用程序具有服务和已部署应用程序；已部署应用程序已部署服务包。服务具有分区，并且每个分区都有一个或多个副本。节点和已部署实体之间具有特殊关系。如果其机构系统组件（故障转移管理器服务）报告某一节点不正常，则它会影响已部署应用程序、服务包和在其上部署的副本。

运行状况层次结构基于最新的运行状况报告表示系统的最新状态，该报告几乎是实时信息。基于应用程序特定的逻辑或监视的自定义条件，内部和外部的监视器可以针对相同实体进行报告。用户报告与系统报告共存。

当通过设计服务更容易对大型云服务进行调试、监视和随后操作时，花时间规划如何报告并对运行状况作出响应。

## 运行状况状态
Service Fabric 使用三种运行状况状态来说明实体是否正常：“确定”、“警告”和“错误”。发送到运行状况存储的任何报告都必须指定其中一种状态。运行状况评估结果是其中一种状态。

可能的运行状况状态如下：

- “确定”：实体正常。没有针对它或其子项（如果适用）报告已知问题。

- “警告”：实体遇到一些问题，但并非不正常（即：不会导致任何功能问题的意外延迟）。在某些情况下，警告条件可能会在不需要任何特殊干预的情况下修复本身，并可用于提供后续事项的可见性。在其他情况下，警告条件可能会在无需用户干预的情况下降级到严重问题。

- “错误”：实体不正常。应采取行动修复实体的状态，因为它无法正常运行。

- “未知”：运行状况存储中不存在实体。可以从分布式查询（例如获取 Service Fabric 节点或应用程序）中获取此结果。这些查询会合并来自多个系统组件的结果。如果另一个系统组件有一个实体，该实体尚未达到运行状况存储或已从运行状况存储中清除，则合并后的查询会填充具有“未知”运行状况状态的运行状况结果。

## 运行状况策略
运行状况存储应用运行状况策略基于其报告与其子项确定实体是否正常。

> [AZURE.NOTE]可以在群集清单（适用于群集和节点运行状况评估）或应用程序清单（适用于应用程序评估和任何其子项）中指定运行状况策略。运行状况评估请求也可以在自定义运行状况评估策略中传递，并且仅用于该评估。

默认情况下，Service Fabric 针对父-子层次结构关系应用严格的规则（所有内容都必须正常）；只要其中一个子项具有一个不正常事件，父项则被视为不正常。

### 群集运行状况策略
群集运行状况策略用于评估群集运行状况状态和节点运行状况状态。可以在群集清单中对它进行定义。如果不存在，则默认为默认策略（0 容忍失败）。
包含：

- **ConsiderWarningAsError**。指定是否在运行状况评估期间将“警告”运行状况报告视为错误。默认值：False。

- **MaxPercentUnhealthyApplications**。群集被视为“错误”之前可以保留不正常的应用程序的最大容忍百分比。

- **MaxPercentUnhealthyNodes**。群集被视为“错误”之前可以保留不正常的节点的最大容忍百分比。在大型群集中，始终会有关闭或故障进行修复的节点，因此应配置此百分比以便容忍该节点。

下面是群集清单的摘录：

```xml
<FabricSettings>
  <Section Name="HealthManager/ClusterHealthPolicy">
    <Parameter Name="ConsiderWarningAsError" Value="False" />
    <Parameter Name="MaxPercentUnhealthyApplications" Value="0" />
    <Parameter Name="MaxPercentUnhealthyNodes" Value="20" />
  </Section>
</FabricSettings>
```

### 应用程序运行状况策略
应用程序运行状况策略说明如何对应用程序及其子项进行事件和子项状态聚合评估。它可以在应用程序清单（应用程序包中的 ApplicationManifest.xml）中定义。如果未指定，当它具有“警告”或“错误”运行状况状态的运行状况报告或子项时，则 Service Fabric 假定实体不正常。
可配置的策略是：

- **ConsiderWarningAsError**。指定是否在运行状况评估期间将“警告”运行状况报告视为错误。默认值：False。

- **MaxPercentUnhealthyDeployedApplications**。应用程序被视为“错误”之前可以保留不正常的已部署应用程序的最大容忍百分比。此值通过用不正常的已部署应用程序的数目除以目前在群集中部署的应用程序的节点数目计算得出。计算结果调高为整数，以便容忍少量节点上出现一次失败。默认值：0%。

- **DefaultServiceTypeHealthPolicy**。指定默认服务类型运行状况策略，该策略会替换应用程序中所有服务类型的默认运行状况策略。

- **ServiceTypeHealthPolicyMap**。按照服务类型映射服务运行状况策略，该策略会替换指定服务类型的默认服务类型运行状况策略。例如，在包含无状态网关服务类型和有状态引擎服务类型的应用程序中，可以对无状态和有状态服务的运行状况策略进行不同的配置。通过按照服务类型指定策略，你能够更精细地控制服务的运行状况。

### 服务类型运行状况策略
服务类型运行状况策略指定如何评估和聚合服务的子项。包含：

- **MaxPercentUnhealthyPartitionsPerService**。服务被视为不正常之前不正常分区的最大容忍百分比。默认值：0%。

- **MaxPercentUnhealthyReplicasPerPartition**。分区被视为不正常之前不正常副本的最大容忍百分比。默认值：0%。

- **MaxPercentUnhealthyServices**。应用程序被视为不正常之前不正常服务的最大容忍百分比。默认值：0%

下面是应用程序清单的摘录：

```xml
    <Policies>
        <HealthPolicy ConsiderWarningAsError="true" MaxPercentUnhealthyDeployedApplications="20">
            <DefaultServiceTypeHealthPolicy
                   MaxPercentUnhealthyServices="0"
                   MaxPercentUnhealthyPartitionsPerService="10"
                   MaxPercentUnhealthyReplicasPerPartition="0"/>
            <ServiceTypeHealthPolicy ServiceTypeName="FrontEndServiceType"
                   MaxPercentUnhealthyServices="0"
                   MaxPercentUnhealthyPartitionsPerService="20"
                   MaxPercentUnhealthyReplicasPerPartition="0"/>
            <ServiceTypeHealthPolicy ServiceTypeName="BackEndServiceType"
                   MaxPercentUnhealthyServices="20"
                   MaxPercentUnhealthyPartitionsPerService="0"
                   MaxPercentUnhealthyReplicasPerPartition="0">
            </ServiceTypeHealthPolicy>
        </HealthPolicy>
    </Policies>
```

## 运行状况评估
用户或自动化服务可以随时评估任何实体的运行状况。若要评估实体运行状况，运行状况存储聚合实体上的所有运行状况报告，并评估其所有子项（如果适用）。运行状况聚合算法使用运行状况策略，该策略指定如何评估运行状况报告以及如何聚合子项运行状况状态（如果适用）。

### 运行状况报告聚合
一个实体可以具有由不同属性上的不同报告器（系统组件或监视器）发送的多个运行状况报告。聚合使用关联的运行状况策略，尤其是应用程序或群集运行状况策略的 ConsiderWarningAsError 成员，该策略指定如何评估警告。

已聚合运行状况状态由实体上**最差**的运行状况报告触发。如果至少有一个“错误”运行状况报告，已聚合运行状况状态则为“错误”。

![运行状况报告与错误报告聚合。][2]

“错误”运行状况报告触发将处于“错误”状态的运行状况实体。

[2]: ./media/service-fabric-health-introduction/servicefabric-health-report-eval-error.png

如果没有任何“错误”报告并有一个或多个“警告”，已聚合运行状况状态则为“警告”或“错误”，具体取决于 ConsiderWarningAsError 策略标志。

![运行状况报告和警告报告聚合，ConsiderWarningAsError 为 false。][3]

运行状况报告与警告报告聚合，ConsiderWarningAsError 为 false（默认值）。

[3]: ./media/service-fabric-health-introduction/servicefabric-health-report-eval-warning.png

### 子项运行状况聚合
实体的已聚合运行状况状态反映子项运行状况状态（如果适用）。用于聚合子项运行状况状态的算法基于实体类型使用适用的运行状况策略。

![子项实体运行状况聚合。][4]

基于运行状况策略的子项聚合。

[4]: ./media/service-fabric-health-introduction/servicefabric-health-hierarchy-eval.png

评估所有子项后，根据取自基于实体和子项类型的策略的已配置最大百分比的不正常情况，运行状况存储会聚合运行状况状态。

- 如果所有子项都具有“确定”状态，该子项已聚合运行状况状态则为“确定”。

- 如果子项具有“确定”状态和“警告”状态，该子项已聚合运行状况状态则为“警告”。

- 如果具有“错误”状态的子项不遵从不正常子项的最大允许百分比，已聚合运行状况状态则为“错误”。

- 如果具有“错误”状态的子项遵从不正常子项的最大允许百分比，已聚合运行状况状态则为“警告”。

## 运行状况报告
系统组件和内部/外部监视器可以告针对 Service Fabric 实体进行报告。*报告器*基于它们正在监视的一些条件对监视的实体的运行状况进行**本地**判断。它们无需查看任何全局状态和聚合数据。之所以不需要，原因是它会使报告器成为复杂的有机体，届时需要查看很多内容，以便推断要发送的信息。

若要将运行状况数据发送到运行状况存储，报告器需要标识受影响的实体并创建运行状况报告。然后，报告可以通过具有 FabricClient.HealthManager.ReportHealth 的 API、通过 Powershell 或通过 REST 进行发送。

### 运行状况报告
群集中每个实体的运行状况报告都包含以下信息：

- SourceId。唯一标识运行状况事件的报告器的字符串。

- 实体标识符。标识对其申请报告的实体。它因[实体类型](service-fabric-health-introduction.md#health-entities-and-hierarchy)而异：

  - 群集：无

  - 节点：节点名称（字符串）。

  - 应用程序：应用程序名称 (URI)。表示群集中部署的应用程序实例的名称。

  - 服务：服务名称 (URI)。表示群集中部署的服务实例的名称。

  - 分区：分区 id (GUID)。表示分区唯一标识符。

  - 副本：有状态服务副本 id 或无状态服务实例 id (Int64)。

  - DeployedApplication：应用程序名称 (URI) 和节点名称（字符串）。

  - DeployedServicePackage：应用程序名称 (URI)、节点名称（字符串）和服务清单名称（字符串）。

- 属性。允许报告器对实体的特定属性的运行状况事件进行分类的*字符串*（不是固定的枚举）。例如，报告器 A 可以报告 Node01“存储”属性的运行状况，报告器 B 可以报告 Node01“连接”属性的运行状况。这两个报告均被视为 Node01 实体的运行状况存储中的单独运行状况事件。

- 说明。报告器用于提供有关运行状况事件的详细信息的字符串。SourceId、属性和 HealthState 应完整说明报告。说明中添加了关于报告的用户可读信息，以便让管理员和用户更容易理解。

- HealthState。说明报告的运行状况状态的[枚举](service-fabric-health-introduction.md#health-states)。已接受值为“确定”、“警告”和“错误”。

- TimeToLive。指示运行状况报告的有效时间的时间跨度。结合 RemoveWhenExpired 时，它能够使 HealthStore 知道如何评估过期的事件。默认情况下，值为无穷大，且报告始终有效。

- RemoveWhenExpired。布尔值。如果设置为 true，过期的运行状况报告会自动从运行状况存储中删除，并且它不会影响实体运行状况评估。仅当报告在一段时间内有效且报告器不需要显式清除它时使用。此外可用于删除运行状况存储中的报告。例如：监视器通过以往源和属性进行更改并停止发送报告。因此它可以发送带有小型 TTL 和 RemoveWhenExpired 的报告，以清除运行状况存储中的所有以往状态。如果设置为 false，过期的报告在运行状况评估中则被视为错误。它向运行状况存储发出指示，源应该对此属性进行定期报告；如果没有报告，则一定是监视器出现了一些问题。通过将事件视为错误捕获监视器运行状况。

- SequenceNumber。需要不断增加的正整数，因为它表示报告的顺序。运行状况存储使用它来检测过时的报告，并且由于网络延迟或其他问题导致接收推迟。如果序列号小于或等于相同实体、源和属性的最新应用的序列号，报告将被拒绝。如果未指定，序列号会自动生成。仅当报告状态转换时才需要指定序列号：源需要记住它发送的报告并保留在故障转移时恢复的信息。

每个运行状况报告都需要 SourceId、实体标识符、属性和 HealthState。不允许 SourceId 字符串以前缀“System.”开头，该字符串是为系统报告保留的。对于相同实体，相同的源和属性只有一个报告；如果为相同的源和属性生成多个报告，它们则会在运行状况客户端（如果按批处理）或在运行状况存储端覆盖彼此。根据序列号进行这种替换操作：（具有更高的序列号）的较新报告替换较旧的报告。

### 运行状况事件
在内部，运行状况存储保留运行状况事件，包含来自报告以及其他元数据的所有信息，例如报告提供给运行状况客户端的时间以及在服务器端修改报告的时间。运行状况事件通过[运行状况查询](service-fabric-view-entities-aggregated-health.md#health-queries)返回。

已添加元数据包含：

- SourceUtcTimestamp：报告提供给运行状况客户端的时间 (Utc)

- LastModifiedUtcTimestamp：上次在服务器端修改报告的时间 (Utc)

- IsExpired：用于指示运行状况存储执行查询时报告是否过期的标志。仅当 RemoveWhenExpired 为 false 时事件才可能过期；否则，事件不会由查询返回，会从存储中删除。

- LastOkTransitionAt、LastWarningTransitionAt、LastErrorTransitionAt：确定/警告/错误转换的最后时间。这些字段提供关于事件的运行状况状态的转换的历史记录。

状态转换字段可用于更智能的警报或“历史”运行状况事件信息。它们实现类似下列方案：

- 当属性已处于警告/错误状态超过 X 分钟时发出警报。这样可避免针对临时情况发出警报。例如：如果运行状况状态已处于警告状态超过 5 分钟，则发出警报，可以转化为（HealthState == 警告并且立即 - LastWarningTransitionTime
> 5 分钟）。

- 仅针对在最后 X 分钟内更改的条件发出警报。如果在此之前，报告处于“错误”状态，则可以忽略它（因为之前已对它发出指示)。

- 如果属性在“警告”和“错误”之间切换，则确定它处于不正常状态的时间（即不确定）。例如：如果属性处于不正常状态超过 5 分钟，则发出警报，可以转化为：（HealthState != 确定并且立即 - LastOkTransitionTime > 5 分钟）。

## 示例：报告和评估应用程序运行状况
下列示例在源 MyWatchdog 中名为 fabric:/WordCount 的应用程序上通过 Powershell 发送运行状况报告。运行状况报告包含有关处于“错误”运行状况状态中的运行状况属性可用性的信息，含无限 TTL。然后，它会查询应用程序运行状况，此查询会返回已聚合运行状况状态错误和作为运行状况事件列表一部分的已报告运行状况事件。

```powershell
PS C:\> Send-ServiceFabricApplicationHealthReport –ApplicationName fabric:/WordCount –SourceId "MyWatchdog" –HealthProperty "Availability" –HealthState Error

PS C:\> Get-ServiceFabricApplicationHealth fabric:/WordCount

ApplicationName                 : fabric:/WordCount
AggregatedHealthState           : Error
UnhealthyEvaluations            :
                                  Error event: SourceId='MyWatchdog', Property='Availability'.

ServiceHealthStates             :
                                  ServiceName           : fabric:/WordCount/WordCount.Service
                                  AggregatedHealthState : Warning

                                  ServiceName           : fabric:/WordCount/WordCount.WebService
                                  AggregatedHealthState : Ok

DeployedApplicationHealthStates :
                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : Node.4
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : Node.1
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : Node.5
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : Node.2
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : Node.3
                                  AggregatedHealthState : Ok

HealthEvents                    :
                                  SourceId              : System.CM
                                  Property              : State
                                  HealthState           : Ok
                                  SequenceNumber        : 5102
                                  SentAt                : 4/15/2015 5:29:15 PM
                                  ReceivedAt            : 4/15/2015 5:29:15 PM
                                  TTL                   : Infinite
                                  Description           : Application has been created.
                                  RemoveWhenExpired     : False
                                  IsExpired             : False
                                  Transitions           : ->Ok = 4/15/2015 5:29:15 PM

                                  SourceId              : MyWatchdog
                                  Property              : Availability
                                  HealthState           : Error
                                  SequenceNumber        : 130736794527105907
                                  SentAt                : 4/16/2015 5:37:32 PM
                                  ReceivedAt            : 4/16/2015 5:37:32 PM
                                  TTL                   : Infinite
                                  Description           :
                                  RemoveWhenExpired     : False
                                  IsExpired             : False
                                  Transitions           : ->Error = 4/16/2015 5:37:32 PM

```

## 运行状况模型用法
利用运行状况模型，你能够通过云服务和基础 Service Fabric 平台进行缩放，因为监视和运行状况判断分布在群集内不同监视器中。
其他系统在群集级别具有单个集中式服务，该服务分析服务发出的所有*可能*有用的信息。这会妨碍其可缩放性并且它不允许使用它们收集非常具体的信息来帮助确定问题和尽可能接近根本原因的潜在问题。

运行状况模型大量用于监视和诊断、评估群集和应用程序运行状况以及监视的升级。其他服务使用运行状况数据执行自动修复、构建群集运行状况历史记录以及对某些条件发出警报。

## 后续步骤
[如何查看 Service Fabric 运行状况报告](service-fabric-view-entities-aggregated-health)

[使用系统运行状况报告进行故障排除](service-fabric-understand-and-troubleshoot-with-system-health-reports)

[添加自定义 Service Fabric 运行状况报告](service-fabric-report-health)

[如何在本地监视和诊断服务](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally)

[Service Fabric 应用程序升级](service-fabric-application-upgrade.md)
 

<!---HONumber=74-->