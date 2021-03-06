
## <a name="view-vms-scheduled-for-maintenance-in-the-portal"></a>在门户中查看计划用于维护的虚拟机

安排了计划的大量维护并发送通知后，可以观察受即将到来的大量维护影响的虚拟机列表。 

可以使用 Azure 门户来查找计划进行维护的 VM。

1. 登录到 [Azure 门户](https://portal.azure.com)。

2. 在左侧导航栏中，单击“虚拟机”。

3. 在“虚拟机”窗格中，单击“列”按钮，打开可用列的列表。

4. 选择并添加以下列：

   维护 - 显示虚拟机的维护状态。 下面是可能的值：
      
      | 值 | 说明 |
      |-------|-------------|
      | 立即启动 | 虚拟机位于自助维护窗口中，用户可以自行启动维护。 请参阅以下内容，了解如何在虚拟机上启动维护 | 
      | 计划 | 已安排虚拟机进行维护，无需用户启动维护。 可以在此视图中选择“自动计划”窗口或单击虚拟机来查看维护窗口 | 
      | 已完成 | 已成功启动并完成虚拟机维护。 | 
      | 已跳过| 已经选择启动维护，但没有成功。 将无法使用自助式维护选项。 你的 VM 必须由 Azure 在计划性维护阶段重启。 | 

   主动维护 - 显示可以自行启动虚拟机维护的时间范围。
   
   计划的维护 - 显示 Azure 重新启动虚拟机以完成维护的时间范围。 




## <a name="notification-and-alerts-in-the-portal"></a>门户中的通知和警报

Azure 通过向订阅所有者和共有者组发送电子邮件来传达计划维护的安排。 可以通过创建 Azure 活动日志警报，为此通信添加其他收件人和频道。 有关详细信息，请参阅 [通过 Azure 活动日志监视订阅活动] (../articles/monitoring-and-diagnostics/monitoring-overview-activity-logs.md)

1. 登录到 [Azure 门户](https://portal.azure.com)。
2. 在左侧菜单中选择“监视”。 
3. 在“监视 -活 动日志”窗格中，选择“警报”。
4. 在“监视 - 警报”窗格中，单击“+ 添加活动日志警报”。
5. 填写“添加活动日志警报”页中的信息，请务必在“条件”中设置以下内容：类型：维护；状态：全部（请勿将状态设置为“活动”或“已解决”）；级别：全部
    
若要详细了解如何配置活动日志警报，请参阅[创建活动日志警报](../articles/monitoring-and-diagnostics/monitoring-activity-log-alerts.md)
    
    
## <a name="start-maintenance-on-your-vm-from-the-portal"></a>从门户启动虚拟机维护

在查看虚拟机详细信息时，将能够看到更多维护相关的详细信息。  
如果虚拟机包含在计划的大量维护中，则会在在虚拟机详细信息视图的顶部添加新的通知功能区。 此外，如有必要，可以添加一个新选项来启动维护。 


单击维护通知以查看维护页面，其中包含计划维护的更多详细信息。 从这里将能够开始维护虚拟机。

开始维护后，虚拟机将重新启动，维护状态得以更新，在几分钟内反映结果。

如果错过了可以开始维护的窗口，当 Azure 重新启动虚拟机时，仍然可以看到该窗口。 
