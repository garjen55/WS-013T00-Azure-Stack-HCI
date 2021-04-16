---
lab:
    title: 'Lab: Using Windows Admin Center in hybrid scenarios'
    module: 'Module 2: Operating and maintaining Azure Stack HCI'
---

# Lab: Using Windows Admin Center in hybrid scenarios

## Scenario

Contoso, Ltd. is a medium-size financial services company with its headquarters in London, England. It currently operates almost entirely on-premises, with most of its compute environment running on the Windows Server platform, including virtualized workloads on Windows Server 2012 R2 and Microsoft Hyper-V hosts in Windows Server 2016. Its internal IT staff is well versed in Microsoft technologies, including its virtualization and software-defined datacenter offerings.

In recent months, as part of datacenter consolidation and modernization initiatives, Contoso IT migrated some of its applications to a range of Azure infrastructure as a service (IaaS) and platform as a service (PaaS) services. However, several highly regulated workloads must remain in the on-premises datacenters.

Two of these workloads present a challenge due to their performance and resiliency requirements. The first workload is a group of heavily utilized Microsoft SQL Server instances hosting transactional databases for Contoso's loan origination department. The second workload is an isolated Virtual Desktop Infrastructure (VDI) farm for users in Contoso's securities research department, which is supposed to replace an aging Windows Server 2012 R2-based Remote Desktop Services (RDS) deployment.

Contoso's Chief Information Officer (CIO) realizes that implementing these workloads will require additional investment in hardware. Before making the investment, she wants to verify that the extra expense will help the IT organization deliver a modern technological solution and accelerate the datacenter consolidation initiative. She also wants to make sure that it helps ensure a consistent management approach that leverages existing IT skills and, if possible, integrates with some of the cloud services that Contoso is already benefiting from, such as Azure Monitor. It's also critical that the new solution provide multiple levels of high availability and resiliency, thereby protecting them from localized failures and facilitating disaster recovery to another on-premises location.

IT management has started its search for solutions that would satisfy these requirements. As lead system engineer, they have asked you to assist with searching and implementing a proof-of-concept environment that would help identify the most viable candidate.

To address the requirements for deployments of highly regulated workloads, you'll provision the core compute and networking components of the lab environment and then test integration of hyperconverged infrastructure with Azure services, including Azure Monitor and Azure Automation. You'll also test Cluster-Aware updating.

## Objectives

After completing this lab, you'll be able to:

- Provision the lab environment by using PowerShell.
- Integrate hyperconverged infrastructure with Azure services.
- Review Azure integration functionality.
- Manage updates to hyperconverged infrastructure.
- Deprovision the lab environment.

## Estimated time: 180 minutes

## Lab setup

To connect to the lab virtual machine (VM), follow the steps the lab hosting provider provides you.

## Exercise 1: Provisioning the lab environment by using PowerShell

### Scenario

To evaluate integration of hyperconverged infrastructure with Azure, you must first provision the core compute and networking components of the lab environment.

The main tasks for this exercise are as follows:

1. Prepare the lab artifacts.
1. Deploy the lab infrastructure.

### Task 1: Prepare the lab artifacts

1. From the lab VM, start Windows PowerShell ISE as Administrator.
1. In the Administrator: Windows PowerShell ISE window, from the console pane, run the following to remove the **Zone.Identifier** alternate data stream, which has a value of **3** indicating that it was downloaded from the Internet:

   ```powershell
   Get-ChildItem -Path F:\WSLab-master\ -File -Recurse | Unblock-File
   ```

### Task 2: Deploy the lab infrastructure

1. On the lab VM, in the console pane of the Administrator: Windows PowerShell ISE window, set the current directory to **F:\\WSLab-master\\Scripts**.
1. Rename **F:\\WSLab-master\\Scripts\\LabConfig.ps1** to **LabConfig.m2l0.ps1**.
1. Rename **F:\\WSLab-master\\Scripts\\Scenario.ps1** to **Scenario.m2l0.ps1**.
1. Copy the **Scenario.ps1** and **Labconfig.ps1** files from **F:\\WSLab-master\\Scenarios\\S2D and Cloud Services Onboarding** to **F:\\WSLab-master\\Scripts**.
1. Open the **F:\\WSLab-master\\Scripts\\LabConfig.ps1** file, in the first line, replace **Prefix = 'WSLab-'** with **Prefix = 'WSLabOnboard-'**, and save the change.
1. From the PowerShell ISE window, run the **F:\\WSLab-master\\Scripts\\3_Deploy.ps1** script to provision VMs for the lab environment.

   >**Note**: For the Telemetry Level prompt, select the default setting of **None**. The script should complete in about seven minutes. For the prompt to start the VMs, select All to Start the VMs. When prompted with **Press enter to continue**, select **Enter**.

## Exercise 2: Integrating hyperconverged infrastructure with Azure services

### Scenario

Now that you have the core components of the lab environment provisioned, it is time to implement hyperconverged infrastructure and integrate it with Azure services.

The main tasks for this exercise are as follows:

1. Prepare the lab infrastructure VMs for integration with Azure services.
1. Provision a Storage Spaces Direct cluster within the lab environment.
1. Configure Cloud Witness quorum for the Storage Spaces Direct cluster.
1. Enable Storage Spaces Direct on the cluster.
1. Provision Azure Log Analytics workspace and Azure Log Analytics gateway.
1. Configure Azure Log Analytics workspace.
1. Integrate hyperconverged infrastructure with Azure Automation.
1. Integrate Storage Spaces Direct cluster nodes with Azure Monitor.

### Task 1: Prepare the lab infrastructure VMs for integration with Azure services

1. On the lab VM, use PowerShell to start all the lab infrastructure VMs.
1. From the Hyper-V Manager console, connect to the **WSLabOnboard-DC** VM, and then sign in by using **CORP\\LabAdmin** as the username and **LS1setup!** as the password.
1. Create a new folder **C:\\Library** on the **WSLabOnboard-DC** VM.
1. Copy **F:\\WSLab-master\\Scripts\\Scenario.ps1** from the lab VM to the folder **C:\\Library** on the **WSLabOnboard-DC** VM.
1. Within the console session to the **WSLabOnboard-DC** VM, open the **C:\\Library\\Scenario.ps1** file in **Windows PowerShell ISE**, and then run the first part of the script marked as **#region Prereqs**.

   >**Note**: This part of the script installs prerequisites that allow subsequent parts of the script to run, including Remote Server Administration Tools and Azure PowerShell modules.

   >**Note**: Wait for the script to complete. Ignore any errors regarding **Login-AZaccount**.

1. Within the console session to the **WSLabOnboard-DC** VM, from the **Windows PowerShell ISE** window, run the second part of the script **C:\\Library\\Scenario.ps1** marked as **#region Install Windows Admin Center in a GW mode**.

   >**Note**: This part of the script installs Windows Admin Center in the gateway mode on the **WACGW** VM.

   >**Note**: Ignore error messages regarding aborted I/O operation.

1. Install the Microsoft Edge based on Chromium browser.

### Task 2: Provision a Storage Spaces Direct cluster within the lab environment

1. Switch to the lab VM and from the PowerShell ISE window, shut down the VMs that will serve as nodes of the Storage Spaces Direct cluster in this lab (**WSLabOnboard-S2D1**, **WSLabOnboard-S2D2**, **WSLabOnboard-S2D3**, and **WSLabOnboard-S2D4**).
1. On the lab VM, from the PowerShell ISE window, enable nested virtualization for the VMs that will serve as nodes of the Storage Spaces Direct cluster in this lab.
1. On the lab VM, from the PowerShell ISE window, configure static memory for the VMs that will serve as nodes of the Storage Spaces Direct cluster in this lab, leaving **ProcessorCount** at **2** and setting **MemoryStartupBytes** to **4GB**.
1. On the lab VM, from the PowerShell ISE window, start the VMs that will serve as nodes of the Storage Spaces Direct cluster in this lab.
1. Switch to the console session to the **WSLabOnboard-DC** VM. From the PowerShell ISE window, install Windows Server 2019 roles and features **Hyper-V**, **Failover-Clustering**, **Data-Center-Bridging**, **RSAT-Clustering-PowerShell**, **Hyper-V-PowerShell**, and **FS-FileServer** as necessary to provision Storage Spaces Direct cluster on the four VMs in the lab environment (**S2D1**, **S2D2**, **S2D3**, and **S2D4**).
1. Within the console session to the **WSLabOnboard-DC** VM, from the PowerShell ISE window, restart **S2D1**, **S2D2**, **S2D3**, and **S2D4** VMs.

   >**Note**: Verify that the operating system in all VMs is running before you proceed to the next step.

1. Within the console session to the **WSLabOnboard-DC** VM, from the PowerShell ISE window, run the following script to configure storage to prepare for provisioning of a Storage Spaces Direct cluster on the four VMs in the lab environment (**S2D1**, **S2D2**, **S2D3**, and **S2D4**):

   ```powershell
   Invoke-Command ($servers) {
     Update-StorageProviderCache
     Get-StoragePool | ? IsPrimordial -eq $false | Set-StoragePool -IsReadOnly:$false -ErrorAction SilentlyContinue
     Get-StoragePool | ? IsPrimordial -eq $false | Get-VirtualDisk | Remove-VirtualDisk -Confirm:$false -ErrorAction SilentlyContinue
     Get-StoragePool | ? IsPrimordial -eq $false | Remove-StoragePool -Confirm:$false -ErrorAction SilentlyContinue
     Get-PhysicalDisk | Reset-PhysicalDisk -ErrorAction SilentlyContinue
     Get-Disk | ? Number -ne $null | ? IsBoot -ne $true | ? IsSystem -ne $true | ? PartitionStyle -ne RAW | % {
        $_ | Set-Disk -isoffline:$false
        $_ | Set-Disk -isreadonly:$false
        $_ | Clear-Disk -RemoveData -RemoveOEM -Confirm:$false
        $_ | Set-Disk -isreadonly:$true
        $_ | Set-Disk -isoffline:$true
    }
    Get-Disk | Where Number -Ne $Null | Where IsBoot -Ne $True | Where IsSystem -Ne $True | Where PartitionStyle -Eq RAW | Group -NoElement -Property FriendlyName
   } | Sort -Property PsComputerName, Count
   ```

1. Within the console session to the **WSLabOnboard-DC VM**, from the PowerShell ISE window, run cluster validation tests for **Storage Spaces Direct**, **Inventory**, **Network**, and **System Configuration** for the four VMs in the lab environment (**S2D1**, **S2D2**, **S2D3**, and **S2D4**).


   >**Note**: In order to run Test-Cluster from the **WSLabOnboard-DC** VM, you will need to install the Failover Clustering feature and restart the **WSLabOnboard-DC** VM. Ignore cluster validation errors. That's expected.

1. Within the console session to the **WSLabOnboard-DC** VM, from the PowerShell ISE window, create a new cluster named **S2DCL1** consisting of the four VMs in the lab environment (**S2D1**, **S2D2**, **S2D3**, and **S2D4**).

   >**Note**: Wait for the cluster to be provisioned.

### Task 3: Configure Cloud Witness quorum for the Storage Spaces Direct cluster

1. Within the console session to the **WSLabOnboard-DC** VM, start the Microsoft Edge based on Chromium browser and sign in to the Azure portal using a user account with the Owner or Contributor role in the Azure subscription you will be using in this lab.
1. In the Azure portal, create a storage account with the following settings (leave others with their default values):

   *Table 1: Storage account settings*
   
   |Setting|Value|
   |---|---|
   |Subscription|the name of the Azure subscription you are using in this lab|
   |Resource group|WS013-02-RG|
   |Storage account name|any globally unique name between 3 and 24 in length consisting of letters and digits|
   |Location|the name of an Azure region in proximity to the location of the lab environment|
   |Performance|Standard|
   |Account kind|Storage (general purpose v1)|
   |Replication|Locally redundant storage (LRS)|

   >**Note**: Wait for the Storage account to be created. This should take about two minutes.

1. Copy the storage account name and its primary access key into Notepad.
1. Within the console session to the **WSLabOnboard-DC** VM, start another browser instance, navigate to the Windows Admin Center installation on the **WACGW** VM, and sign in by using **CORP\LabAdmin** as the user name and **LS1setup!** as the password.

    >**Note**: Select **Continue** if you receive an error that the connection is not secure.

1. From the **Windows Admin Center** interface, connect to the `S2DCL1.corp.contoso.com` cluster, and then authenticate by using the **CORP\\LabAdmin** credentials.
1. Add **Cloud witness** quorum to the `S2DCL1.corp.contoso.com` cluster.
1. When you receive a prompt, enable CredSSP.

### Task 4: Enable Storage Spaces Direct on the cluster

1. Within the console session to the **WSLabOnboard-DC** VM, from the PowerShell ISE window, enable Storage Spaces Direct on the newly created cluster.

    >**Note**: Disregard any error messages regarding **No disks found to be used for cache**.

1. Review the **Storage Spaces and Pools** settings of the `S2DCL1.corp.contoso.com` cluster from the **Windows Admin Center** interface.

   >**Note**: You might need to refresh the browser page to connect to the cluster.

### Task 5: Provision Azure Log Analytics workspace and Azure Log Analytics gateway

1. Within the console session to the **WSLabOnboard-DC** VM, switch to the browser window displaying the Azure portal, start a PowerShell session in **Cloud Shell**, and use it to register the **Microsoft.Insights** and **Microsoft.AlertsManagement** resource providers.
1. Use Cloud Shell to verify that the registration was successful.

    >**Note**: Wait until the registration completes.

1. Within the console session to the **WSLabOnboard-DC** VM, in the **Windows PowerShell ISE** window, in the **C:\\Library\\Scenario.ps1** file in the script pane, in line **77**, replace **OutpuMode** with **OutputMode**.

   >**Note**: Run **Install-Module AZ** on the **WSLabOnboard-DC** VM prior to performing the next step.

1. Within the console session to the **WSLabOnboard-DC** VM, from the **Windows PowerShell ISE** window, run the fourth part of the script marked **#region Connect to Azure and create Log Analytics workspace if needed**.

   >**Note**: This part of the script creates the Log Analytics workspace.

1. Follow prompts to authenticate to an Azure subscription.

   >**Note**: If the user account is associated with multiple Azure subscriptions, the script will automatically display a grid with the list of your subscriptions. Select the one you want to use in this lab and then select **OK**.

   >**Note**: If you have existing Azure Log Analytics workspaces in the Azure subscription that you select, the script will automatically display a grid with the list of available Log Analytics workspaces in the Azure subscription you selected. Select **Cancel**, and the script will automatically provision one with a name that consists of the **WSLabWinAnalytics** prefix followed by the Azure subscription ID.

1. When you receive a prompt to select the Azure regions where the Log Analytics workspace will reside, select **eastus**.

   >**Note**: Make sure to select **eastus** as the target Azure region. The Azure Log Analytics location and the corresponding Azure Automation account locations must follow mappings documented in [Supported regions for linked Log Analytics workspace](https://aka.ms/region-mappings).

1. Within the console session to the **WSLabOnboard-DC** VM, from the **Windows PowerShell ISE** window, run the fifth part of the script marked as **#region setup Log Analytics Gateway**.

   >**Note**: This part of the script installs Log Analytics Gateway.

   >**Note**: Disregard the warning about breaking changes to the cmdlet **Get-AzOperationalInsightsWorkspaceSharedKey**.

### Task 6: Configure Azure Log Analytics workspace

1. Within the console session to the **WSLabOnboard-DC** VM, switch back to the browser window displaying the Azure portal.
1. In the Azure portal, navigate to the blade displaying the newly created Log Analytics workspace.

   >**Note**: The workspace name has the **WSLabWorkspace** prefix.

1. From the Log Analytics workspace blade, enable collecting data from the **System** and **Application** Windows event logs as well as the **Processor(*)\\\% Processor Time** Windows performance counters.

### Task 7: Integrate hyperconverged infrastructure with Azure Automation

1. Within the console session to the **WSLabOnboard-DC** VM, in the PowerShell ISE window, in the **C:\\Library\\Scenario.ps1** file in the script pane, replace line **163** with the following code:

   ```powershell
   $location = 'eastus2'
   ```

   >**Note**: This ensures that the location of the Azure Automation account maps to the location of the Azure Log Analytics workspace, as documented in [Supported regions for linked Log Analytics workspace](https://aka.ms/region-mappings).

1. Within the console session to the **WSLabOnboard-DC** VM, from the **Windows PowerShell ISE** window, run the sixth part of the script marked as **#region deploy a Windows Hybrid Runbook Worker**.

   >**Note**: This part of the script creates an Azure Automation Account and configures Hybrid Runbook Worker on the **HRWorker01** VM.

   >**Note**: Disregard the warning about breaking changes to the cmdlet **Get-AzOperationalInsightsWorkspaceSharedKey**.

   >**Note**: Disregard error messages during registration of the Hybrid Runbook Worker. You can verify that the registration was successful by switching to the browser displaying the Azure portal interface, navigating to the **WSLabAutomationAccount** Azure Automation account you created in this task, selecting **Hybrid worker groups**, and finally selecting the **System hybrid worker groups**. There you will find the ```HRWorker01.Corp.contoso.com``` entry, representing the newly registered Hybrid Runbook worker.

1. Within the console session to the **WSLabOnboard-DC** VM, in the PowerShell ISE window, in the **C:\\Library\\Scenario.ps1** file, replace line **286** with the following code:

   ```powershell
   $location = 'eastus2'
   ```

1. Within the console session to the **WSLabOnboard-DC** VM, from the **Windows PowerShell ISE** window, run the seventh part of the script marked as **#region configure Hybrid Runbook Worker Addresses and Azure Automation Agent Service URL on Log Analytics Gateway**.

   >**Note**: This part configures the Log Analytics Gateway to connect to the Azure Automation endpoints.

### Task 8: Integrate Storage Spaces Direct cluster nodes with with Azure Monitor

1. Within the console session to the **WSLabOnboard-DC** VM, from the **Windows PowerShell ISE** window, run the eighth part of the script marked as **#region download and deploy MMA Agent to S2D cluster nodes**.

   >**Note**: This part installs the Log Analytics agent to Storage Spaces Direct cluster nodes.

   >**Note**: Disregard the warning about breaking changes to the cmdlet **Get-AzOperationalInsightsWorkspaceSharedKey**.

1. Within the console session to the **WSLabOnboard-DC** VM, from the **Windows PowerShell ISE** window, run the ninth part of the script marked as **#region download and install dependency agent (for service map solution)**.

   >**Note**: This part installs the Dependency agent which provides the Service Map functionality.

## Exercise 3: Reviewing Azure integration functionality

### Scenario

With the integration in place, you now must validate its capabilities by reviewing references to hyperconverged infrastructure in Azure Monitor, Log Analytics, and Azure Automation.

The main tasks for this exercise are as follows:

1. Review Log Analytics functionality.
1. Review Azure Automation functionality.
1. Review Service Map functionality.

### Task 1: Review Log Analytics functionality

1. Within the console session to the **WSLabOnboard-DC** VM, switch back to the browser window displaying the Azure portal.
1. In the Azure portal, navigate to the blade displaying the newly created Log Analytics workspace.

   >**Note**: The workspace name has the **WSLabWorkspace** prefix.

1. From the Log Analytics workspace blade, run the **Top 10 Virtual Machines by CPU utilization** sample query.

   >**Note**: Review the query and the results.

   >**Note**: The query might result in the syntax error message if the data has not been collected yet. If so, wait for a few minutes and try again or return to this task once you complete the rest of the lab.

### Task 2: Review Azure Automation functionality

1. Within the console session to the **WSLabOnboard-DC** VM, in the browser window displaying the Azure portal, navigate back to the blade displaying the Log Analytics workspace you were reviewing in the previous task.
1. On the Log Analytics workspace blade, in the **Related Resources** section, select **Automation Account**.
1. Note the information regarding the linked Automation account, and then select **Go to account**.
1. On the **WSLabAutomationAccount** blade, in the **Configuration Management** section, select **Inventory** and note that you have the option to **Enable** the Inventory solution.
1. Without making any changes, on the **WSLabAutomationAccount** blade, in the **Configuration Management** section, select **Change tracking**.
1. On the **Change tracking** blade, note that you have the option to **Enable** the Change tracking solution.
1. Without making any changes, on the **WSLabAutomationAccount** blade, in the **Process automation** section, select **Hybrid worker groups**.
1. On the **Hybrid worker groups** blade, select the **System hybrid worker groups** tab and note that it contains a separate group for each server that was registered with Azure Automation, with a single worker per group.

   >**Note**: Verify that last seen time for each worker group is within one hour of the current time.

### Task 3: Review Service Map functionality

1. Within the console session to the **WSLabOnboard-DC** VM, in the browser window displaying the Azure portal, navigate back to the blade displaying the Log Analytics workspace you were reviewing in the first task of this exercise.
1. On the Log Analytics workspace blade, in the **General** section, select **Workspace summary**.
1. On the **Overview** blade, review the list of solutions that you implemented in the previous exercise and navigate to the **Service Map** blade.

   >**Note**: It may take several minutes for the **Service Map** blade to appear.

1. On the **Service Map** blade, on the **Machines** tab, in the list of monitored servers, select **S2D1** (one of the nodes of the Storage Spaces Direct cluster), zoom into the diagram in the center of the blade, and then review the **Summary** pane on the right-hand side of the blade.
1. With the **S2D1** server selected, display each of the sections on the right-hand side of the pane, including **Summary**, **Properties**, **Alerts**, **Log Events**, **Performance**, **Security**, and **Updates**.

   >**Note**: In the **Security** section, if you find the **Logons with a clear text password** entry, select it. You will be automatically redirected to the Log Analytics workspace blade displaying the corresponding Kusto Query Language (KQL) query.

1. With the **S2D1** server selected, zoom in further on the diagram, and then expand the rectangle representing the **S2D2** cluster node.

   >**Note**: Review the list of connections, and verify that they involve multiple processes (such as **clussvc** and **System**).

1. Review the diagram and note that it includes connections to `DC.corp.contoso.com` over ports 53 (dns), 67 (bootps), 88, 123, 135, 389, and 445 (there might be others).

   >**Note**: These connections are listed even though the **DC** server does not have the Log Analytics and Dependency agents installed.

## Exercise 4: Managing updates to hyperconverged infrastructure

### Scenario

One of your objectives is to determine the most efficient approach to deploying updates to servers that are part of your hyperconverged environment. You decided to evaluate the use of Cluster Aware Updating and Azure Automation Update Management.

The main tasks for this exercise are as follows:

1. Implement Cluster Aware Updating by using Windows Admin Center.
1. Use Azure Automation update management.

### Task 1: Implement Cluster Aware Updating by using Windows Admin Center

1. Within the console session to the **WSLabOnboard-DC** VM, switch back to the **Windows Admin Center** interface, and then in the list of **Tools** of the `S2DCL1.corp.contoso.com` page, select **Updates**.
1. From the **Windows Admin Center** interface, enable **Cluster Aware Updating**, and then check for available updates.
1. Review the list of available updates without making any changes.

   >**Note**: You have the option to **Apply All Updates**. Do not select it.

   >**Note**: You can monitor the status of applying the updates directly from the **Cluster Aware Updating** panel.

### Task 2: Use Azure Automation update management

1. Within the console session to the **WSLabOnboard-DC** VM, switch back to the browser window displaying the **WSLabAutomationAccount** blade in the Azure portal.
1. From the **WSLabAutomationAccount** blade, navigate to the **Update Management** blade.
1. From the **Update Management** blade, review the list of machines and identify noncompliant ones.

   >**Note**: To schedule an update deployment, you must first create a computer group.

1. Within the console session to the **WSLabOnboard-DC** VM, in the browser displaying the Azure portal, navigate back to the blade displaying the Log Analytics workspace you were reviewing in the previous exercise.
1. On the Log Analytics workspace blade, navigate to the list of example queries.
1. From the list of example queries, load the **Missing security or critical updates** from the **Virtual Machine** section into the editor window.
1. In the editor window, remove the line '| summarize count() by Classification', select **Run** and review results of the query.

   >**Note**: the query lists all of missing security or critical updates.

1. In the editor window, replace the query with the following one:

   ```kql
   Update
   | where UpdateState == 'Needed' and Optional == false and Classification == 'Security Updates' and Approved != false and Computer hassuffix "corp.contoso.com" and Computer !contains "S2D"
   | distinct Computer
   ```

   >**Note**: Computer group queries must use the `distinct Computer` clause.

   >**Note**: The query excludes servers which are members of the Storage Spaces Direct cluster since these are updated by using Cluster Aware Updating.

1. Run the query to verify that the query returns the list of noncompliant servers in the `Corp.contoso.com` domain that are not part of the Storage Spaces Direct cluster.
1. Save the query with the following settings:

   *Table 2: Group query settings*

   |Setting|Value|
   |---|---|
   |Name|`corp.contoso.com non-compliant non S2D servers`|
   |Save as|Function|
   |Function Alias|`corp_non_s2d_non_compliant`|
   |Save this query as a computer group|Enabled|
   |Category|Updates|

1. In the Azure portal, navigate back to the **Update Management** blade of the **WSLabAutomationAccount** Automation Account.
1. From the **WSLabAutomationAccount** Automation Account blade, schedule an update deployment with the following settings (leave others with their default values):

   *Table 3: Update deployment settings*

   |Setting|Value|
   |---|---|
   |Name|ws01302 update deployment|
   |Operating system|Windows|
   |Groups to update|`corp.contoso.com non-compliant non S2D servers`|
   |Machines to update|`corp.contoso.com non-compliant non S2D servers`|
   |Schedule settings|date and time at least 5 minutes ahead of the current date and time|

   >**Note**: You can use the **Include/exclude updates** setting to include or exclude individual updates.

   >**Note**: You can use the **Pre-scripts + Post-scripts** setting to specify scripts to run before and after patch deployment.

1. Back on the **Update management** blade, navigate to the **Deployment schedules** tab, and ensure that the deployment has been successfully scheduled.

## Exercise 5: Deprovisioning the lab environment

### Scenario

To minimize Azure-related charges, you'll deprovision the Azure resources provisioned throughout this lab. You'll also revert the state of the lab environment to its original state in preparation for further testing.

The main tasks for this exercise are as follows:

1. Deprovision the Azure resources.
1. Deprovision the lab resources.

### Task 1: Deprovision the Azure resources

1. Switch to the lab VM.
1. Start a browser, navigate to the Azure portal, and then sign in with the Owner or Contributor role in the Azure subscription you will be using in this lab.
1. Within the Azure portal, start a PowerShell session in Cloud Shell.
1. From the Cloud Shell pane, run the following to remove all Azure resources you provisioned in this lab:

   ```powershell
   Get-AzResourceGroup -Name 'WS013-02-RG' | Remove-AzResourceGroup -Force -AsJob
   Get-AzResourceGroup -Name 'WSLabWinAnalytics' | Remove-AzResourceGroup -Force -AsJob
   ```

### Task 2: Deprovision the lab resources

1. On the lab VM, start Windows PowerShell ISE as Administrator.
1. From the PowerShell ISE window, run the **F:\\WSLab-master\\Scripts\\Cleanup.ps1** script to remove all VMs provisioned in this lab.

### Results

After completing this lab, you will have provisioned the lab environment by using PowerShell, integrated hyperconverged infrastructure with Azure services, reviewed Azure integration functionality, managed updates to hyperconverged infrastructure, and deprovisioned the lab environment.
