---
lab:
    title: 'Lab: Using Windows Admin Center in hybrid scenarios'
    type: 'Answer Key'
    module: 'Module 2: Operating and maintaining Azure Stack HCI'
---
# Lab answer key: Using Windows Admin Center in hybrid scenarios

## Exercise 1: Provisioning the lab environment by using PowerShell

### Task 1: Prepare the lab artifacts

1. From the lab virtual machine (VM), start Windows PowerShell ISE as Administrator.
1. In the Administrator: Windows PowerShell ISE window, from the console pane, run the following to remove the **Zone.Identifier** alternate data stream, which has a value of **3**, indicating that it was downloaded from the Internet:

   ```powershell
   Get-ChildItem -Path F:\WSLab-master\ -File -Recurse | Unblock-File
   ```

### Task 2: Deploy the lab infrastructure

1. On the lab VM, in the console pane of the PowerShell ISE window, run the following to set the current directory:

   ```powershell
   Set-Location -Path F:\WSLab-master\Scripts
   ```

1. In the script pane of the PowerShell ISE window, run the following to rename **Scenario.ps1** and **LabConfig.ps1**:

   ```powershell
   Move-Item -Path '.\LabConfig.ps1' -Destination '.\LabConfig.m2l0.ps1' -Force -ErrorAction SilentlyContinue
   Move-Item -Path '.\Scenario.ps1' -Destination '.\Scenario.m2l0.ps1' -Force -ErrorAction SilentlyContinue
   ```

1. In the script pane of the PowerShell ISE window, run the following to copy **Scenario.ps1** and **Labconfig.ps1** files from **F:\\WSLab-master\\Scenarios\\S2D and Cloud Services Onboarding** to the current directory:

   ```powershell
   Copy-Item -Path 'F:\WSLab-master\Scenarios\S2D and Cloud Services Onboarding\Scenario.ps1' -Destination '.\'
   Copy-Item -Path 'F:\WSLab-master\Scenarios\S2D and Cloud Services Onboarding\Labconfig.ps1' -Destination '.\'
   ```

1. In the script pane of the PowerShell ISE window, open the **F:\\WSLab-master\\Scripts\\LabConfig.ps1** file, in the first line, replace `Prefix = 'WSLab-'` with `Prefix = 'WSLabOnboard-'`, save the changes, and then close the file.

1. In the PowerShell ISE window, open and run the **F:\\WSLab-master\\Scripts\\3_Deploy.ps1** script to VMs for the lab environment.

   >**Note**: For the Telemetry Level prompt, select the default setting of None. The script should complete in about seven minutes. For the prompt to start the VMs, select All to Start the VMs. When prompted with Press enter to continue, select **Enter**.

## Exercise 2: Integrating hyperconverged infrastructure with Azure services

### Task 1: Prepare the lab infrastructure VMs for integration with Azure services

1. On the lab VM, from the console pane of the PowerShell ISE window, run the following to start all of the lab infrastructure VMs:

   ```powershell
   Get-VM | Where-Object Name -ne 'WSLabOnboard-DC' | Start-VM
   ```

1. On the lab VM, start the Hyper-V Manager console and select the node representing the lab VM; in the list of VMs, right-click or access the context menu on the **WSLabOnboard-DC** entry; and then select **Connect**. When you receive a prompt, select **Connect**, and then sign in by using **CORP\\LabAdmin** as the username and **LS1setup!** as the password.
1. Within the console session to the **WSLabOnboard-DC** VM, start File Explorer, and then create a folder **C:\\Library**.
1. Switch back to the lab VM and use the copy and paste functionality of the Hyper-V console session to copy **F:\\WSLab-master\\Scripts\\Scenario.ps1** on the lab VM to **C:\\Library** on the **WSLabOnboard-DC** VM.
1. Within the console session to the **WSLabOnboard-DC** VM, start Windows PowerShell ISE as Administrator.
1. Within the console session to the **WSLabOnboard-DC** VM, in the PowerShell ISE window, open the **C:\\Library\\Scenario.ps1** file in the script pane, select the first part of the script between the lines **1** and **25** marked as **#region Prereqs**, and then run that part of the script either by selecting the **F8** key or selecting the **Run selection** button in the toolbar of the PowerShell ISE window.

   >**Note**: This part of the script installs prerequisites that allow subsequent parts of the script to run, including Remote Server Administration Tools and Azure PowerShell modules.

   >**Note**: Wait for the script to complete. Ignore any errors regarding **Login-AZaccount**.

1. Within the console session to the **WSLabOnboard-DC** VM, in the PowerShell ISE window, in the **C:\\Library\\Scenario.ps1** file in the script pane, select the second part of the script between the lines **27** and **61** marked as **#region Install Windows Admin Center in a GW mode**, and then run that part of the script either by selecting the **F8** key or selecting the **Run selection** button in the toolbar of the PowerShell ISE window.

   >**Note**: This part of the script installs Windows Admin Center in the gateway mode on the **WACGW** VM.

   >**Note**: Ignore error messages regarding aborted I/O operation.

1. Install the Microsoft Edge based on Chromium browser.

1. Within the console session to the **WSLabOnboard-DC** VM, complete the Microsoft Edge installation by selecting **Get started**, following subsequent prompts, and then closing the browser window.

### Task 2: Provision a Storage Spaces Direct cluster within the lab environment

1. Switch to the lab VM and in the PowerShell ISE window, from the console pane, run the following to shut down the VMs that will serve as nodes of the Storage Spaces Direct cluster in this lab:

   ```powershell
   $VMs = @('WSLabOnboard-S2D1','WSLabOnboard-S2D2','WSLabOnboard-S2D3','WSLabOnboard-S2D4')
   Stop-VM -VMName $VMs -Force
   ```

1. On the lab VM, in the PowerShell ISE window, from the console pane, run the following to configure nested virtualization for the VMs that will serve as nodes of the Storage Spaces Direct cluster in this lab:

   ```powershell
   Set-VMProcessor -VMName $VMs -ExposeVirtualizationExtensions $true
   ```

1. On the lab VM, in the PowerShell ISE window, from the console pane, run the following to configure static memory for the VMs that will serve as nodes of the Storage Spaces Direct cluster in this lab:

   ```powershell
   Set-VM $VMs -ProcessorCount 2 -StaticMemory -MemoryStartupBytes 4GB
   ```

   >**Note**: This is not required for nested virtualization but mitigates problems with memory during startup.

1. On the lab VM, in the PowerShell ISE window, from the console pane, run the following to start the VMs that will serve as nodes of the Storage Spaces Direct cluster in this lab:

   ```powershell
   Start-VM -VMName $VMs
   ```

1. Switch to the console session to the **WSLabOnboard-DC** VM. In the PowerShell ISE window, open a new tab in the script pane, paste the following script, and then run it to install Windows Server 2019 roles and features necessary to provision Storage Spaces Direct cluster on the four VMs in the lab environment (**S2D1**, **S2D2**, **S2D3**, and **S2D4**):

   ```powershell
   $servers = @('S2D1','S2D2','S2D3','S2D4')
   $features = 'Hyper-V', 'Failover-Clustering', 'Data-Center-Bridging', 'RSAT-Clustering-PowerShell', 'Hyper-V-PowerShell', 'FS-FileServer'
   Invoke-Command ($servers) {
     Install-WindowsFeature -Name $using:features
   }
   ```

1. Within the console session to the **WSLabOnboard-DC** VM, in the PowerShell ISE window, open a new tab in the script pane, paste the following script, and then run it to complete installation of server roles and features by restarting the four VMs in the lab environment (**S2D1**, **S2D2**, **S2D3**, and **S2D4**):

   ```powershell
   Invoke-Command ($servers) {
     Restart-Computer -Force
   }
   ```

   >**Note**: Wait a few minutes until the operating system in all four VMs is running.

1. Within the console session to the **WSLabOnboard-DC** VM, in the PowerShell ISE window, open a new tab in the script pane, paste the following script, and then run it to configure storage to prepare for provisioning of a Storage Spaces Direct cluster on the four VMs in the lab environment (**S2D1**, **S2D2**, **S2D3**, and **S2D4**):

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

1. Within the console session to the **WSLabOnboard-DC** VM, in the PowerShell ISE window, from the console pane, run the following cmdlet to perform cluster validation for the four VMs in the lab environment (**S2D1**, **S2D2**, **S2D3**, and **S2D4**):

   ```powershell
   Test-Cluster -Node 'S2D1','S2D2','S2D3','S2D4' -Include 'Storage Spaces Direct', 'Inventory', 'Network', 'System Configuration'
   ```

   >**Note**: In order to run Test-Cluster from the **WSLabOnboard-DC** VM, you will need to install the Failover Clustering feature and restart the **WSLabOnboard-DC** VM.  Ignore cluster validation errors. That's expected.

1. Within the console session to the **WSLabOnboard-DC** VM, in the PowerShell ISE window, from the console pane, run the following cmdlet to create a new cluster consisting of the four VMs in the lab environment (**S2D1**, **S2D2**, **S2D3**, and **S2D4**):

   ```powershell
   New-Cluster -Name 'S2DCL1' -Node 'S2D1','S2D2','S2D3','S2D4' -NoStorage
   ```

   >**Note**: Wait for the cluster to be provisioned.

### Task 3: Configure Cloud Witness quorum for the Storage Spaces Direct cluster

1. Within the console session to the **WSLabOnboard-DC** VM, start the Microsoft Edge based on Chromium browser, navigate to the [Azure portal](https://portal.azure.com) and, when you receive a prompt, sign in with the Owner or Contributor role in the Azure subscription you will be using in this lab.
1. In the Azure portal, in the **Search resources, services, and docs** text box at the top of the Azure portal page, enter **Storage accounts**, and then select the **Enter** key.
1. On the **Storage accounts** blade, select **+ Add**.
1. On the **Basics** tab of the **Create storage account** blade, specify the following settings (leave others with their default values):

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

1. On the **Basics** tab of the **Create storage account** blade, select **Review + Create**, wait for the validation process to complete, and then select **Create**.

   >**Note**: Wait for the Storage account to be created. This should take about two minutes.

1. In the Azure portal, in the **Search resources, services, and docs** text box at the top of the Azure portal page, enter **Resource groups**, and then select the **Enter** key.
1. On the **Resource groups** blade, in the list of resource group, select the **WS013-02-RG** entry.
1. On the **WS013-02-RG** resource group blade, in the list of resources, select the entry representing the newly created storage account.
1. On the storage account blade, in the **Settings** section, select **Access keys**.
1. From the blade displaying access keys, copy the values of **Storage account name** and **key1** into Notepad.

   >**Note**: You will need both values later in this task.

1. Within the console session to the **WSLabOnboard-DC** VM, start another instance of the Microsoft Edge based on Chromium browser, and then navigate to ```https://wacgw.corp.contoso.com```. When you receive a prompt, sign in by using **CORP\LabAdmin** as the user name and **LS1setup!** as the password.

   >**Note**: Select **Continue** if you receive an error that the connection is not secure.

1. In the **Windows Admin Center** interface, on the **All connections** page, select **+ Add**. On the **Add resources** panel, in the **Server clusters** tile, select **Add**. In the **Cluster name** text box, enter `S2DCL1.corp.contoso.com`and select **Use another account for this connection**. In the **Username** text box, enter **CORP\\LabAdmin**, in the **Password** text box, enter **LS1setup!**; select **Connect with account**, and then select **Add**.

1. Back on the **All connections** page, select the `S2DCL1.corp.contoso.com` entry.
1. On the `S2DCL1.corp.contoso.com` page, examine the **Overview** panel, and then select **Settings**.
1. On the **Settings** panel, in the **Cluster** section, select **Witness**, and then in the **Witness type** drop down list, select **Cloud witness**.
1. In the **Azure storage account name** text box, paste the value of the **Storage account name** you copied earlier in this task.
1. In the **Azure storage account key** text box, paste the value of the **key1** you copied earlier in this task.
1. Select **Save** and when you receive a prompt to enable CredSSP, select **Yes**.

### Task 4: Enable Storage Spaces Direct on the cluster

1. Within the console session to the **WSLabOnboard-DC** VM, switch to the PowerShell ISE window and from the console pane, run the following cmdlet to enable Storage Spaces Direct on the newly created cluster (when you receive a prompt whether to proceed, select **Yes to All**).

   ```powershell
   Enable-ClusterStorageSpacesDirect -CimSession 'S2DCL1'
   ```

   >**Note**: Disregard an error message regarding **No disks found to be used for cache**.

1. Switch back to the browser window displaying the **Windows Admin Center** interface; on the **Settings** panel of `S2DCL1.corp.contoso.com` page, select **Storage Spaces and Pools**; and then examine its settings.

   >**Note**: You might need to refresh the browser page to connect to the cluster.

### Task 5: Provision Azure Log Analytics workspace and Azure Log Analytics gateway

1. Within the console session to the **WSLabOnboard-DC** VM, switch to the browser window displaying the Azure portal, and then select the **Cloud Shell** icon in the toolbar of the portal interface.
1. If you receive a prompt to select either **Bash** or **PowerShell**, select **PowerShell**.

   >**Note**: If this is the first time you are starting **Cloud Shell** and you are presented with the **You have no storage mounted** message, select the subscription you are using in this lab, and then select **Create storage**.

1. From the Cloud Shell pane, run the following to register the **Microsoft.Insights** and **Microsoft.AlertsManagement** resource providers:

   ```powershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.Insights  
   Register-AzResourceProvider -ProviderNamespace Microsoft.AlertsManagement
   ```

1. From the Cloud Shell pane, run the following to verify that the registration was successful:

   ```powershell
   Get-AzResourceProvider -ProviderNamespace Microsoft.Insights  
   Get-AzResourceProvider -ProviderNamespace Microsoft.AlertsManagement
   ```

   >**Note**: Wait until the **RegistrationState** is listed as **Registered**.

1. Within the console session to the **WSLabOnboard-DC** VM, switch to the PowerShell ISE window. In the **C:\\Library\\Scenario.ps1** file in the script pane, in line **77**, replace **OutpuMode** with **OutputMode**, and then save the change.

   >**Note**: Run **Install-Module AZ** on the **WSLabOnboard-DC** VM prior to performing the next step.

1. Within the console session to the **WSLabOnboard-DC** VM, in the PowerShell ISE window, in the **C:\\Library\\Scenario.ps1** file in the script pane, select the fourth part of the script between the lines **74** and **105** marked as **#region Connect to Azure and create Log Analytics workspace if needed**, and then run that part of the script either by selecting the **F8** key or selecting the **Run selection** button in the toolbar of the PowerShell ISE window.

   >**Note**: This part of the script creates the Log Analytics workspace.

1. The script will display instructions to follow to authenticate to an Azure subscription. Start another instance of the Microsoft Edge based on Chromium browser, navigate to [https://microsoft.com/devicelogin](https://aka.ms/device-login), and then enter the code provided in the instructions. When you receive a prompt, authenticate by using a user account with the Owner or Contributor role in the Azure subscription you are using in this lab, and then close the browser window.

   >**Note**: If the user account is associated with multiple Azure subscriptions, the script will automatically display a grid with the list of your subscriptions. Select the one you want to use in this lab, and then select **OK**.

   >**Note**: If you have existing Azure Log Analytics workspaces in the Azure subscription that you select, the script will automatically display a grid with the list of available Log Analytics workspaces in the Azure subscription you selected. Select **Cancel**, and the script will automatically provision one with a name that consists of the **WSLabWinAnalytics** prefix followed by the Azure subscription ID.

1. The script will automatically display a grid containing the list of Azure regions. Select **eastus**, and then select **OK**.

   >**Note**: Make sure to select **eastus** as the target Azure region. The Azure Log Analytics location and the corresponding Azure Automation account locations must follow mappings documented in [Supported regions for linked Log Analytics workspace](https://aka.ms/region-mappings).

1. Within the console session to the **WSLabOnboard-DC** VM, in the PowerShell ISE window, in the **C:\\Library\\Scenario.ps1** file in the script pane, select the fifth part of the script between the lines **107** and **151** marked as **#region setup Log Analytics Gateway**, and then run that part of the script either by selecting the **F8** key or selecting the **Run selection** button in the toolbar of the PowerShell ISE window.

   >**Note**: This part of the script installs Log Analytics Gateway.

   >**Note**: Disregard the warning about breaking changes to the cmdlet **Get-AzOperationalInsightsWorkspaceSharedKey**.

### Task 6: Configure Azure Log Analytics workspace

1. Within the console session to the **WSLabOnboard-DC** VM, switch back to the browser window displaying the Azure portal interface.
1. In the Azure portal, in the **Search resources, services, and docs** text box at the top of the Azure portal page, enter **Log Analytics workspaces**, and then select the **Enter** key.
1. On the **Log Analytics workspaces** blade, select the workspace you created in the previous task.

   >**Note**: The workspace name has the **WSLabWorkspace** prefix.

1. On the Log Analytics workspace blade, in the **Settings** section, select **Agents Configuration**.
1. In the **Windows event logs** tab, enter **System**, and then select **+ Add windows event log**.
1. Use the procedure described in the previous step to add the **Application** log.
1. On the **Agents Configuration** blade, select **Windows performance counters** tab, select **+ Add performance counter**, enter **Processor(*)\\\% Processor Time** and select **Apply**.

### Task 7: Integrate hyperconverged infrastructure with Azure Automation

1. Within the console session to the **WSLabOnboard-DC** VM, switch to the PowerShell ISE window and, in the **C:\\Library\\Scenario.ps1** file in the script pane, replace line **163**

   ```powershell
   $location=(Get-AzOperationalInsightsWorkspace -Name $WorkspaceName -ResourceGroupName $ResourceGroupName).Location
   ```

   with the following code:

   ```powershell
   $location = 'eastus2'
   ```

   >**Note**: This ensures that the location of the Azure Automation account maps to the location of the Azure Log Analytics workspace, as documented in [Supported regions for linked Log Analytics workspace](https://aka.ms/region-mappings).

1. Within the console session to the **WSLabOnboard-DC** VM, in the PowerShell ISE window, in the **C:\\Library\\Scenario.ps1** file in the script pane, select the sixth part of the script between the lines **153** and **287** marked as **#region deploy a Windows Hybrid Runbook Worker**, and then run that part of the script either by selecting the **F8** key or selecting the **Run selection** button in the toolbar of the PowerShell ISE window.

   >**Note**: This part of the script creates an Azure Automation Account and configures Hybrid Runbook Worker on the **HRWorker01** VM.

   >**Note**: Disregard the warning about breaking changes to the cmdlet **Get-AzOperationalInsightsWorkspaceSharedKey**.

   >**Note**: Disregard error messages during registration of the Hybrid Runbook Worker. You can verify that the registration was successful by switching to the browser displaying the Azure portal interface, navigating to the **WSLabAutomationAccount** Azure Automation account you created in this task, selecting **Hybrid worker groups**, and then finally selecting the **System hybrid worker groups**. You will find the ```HRWorker01.Corp.contoso.com``` entry there, representing the newly registered Hybrid Runbook worker.

1. Within the console session to the **WSLabOnboard-DC** VM, in the PowerShell ISE window, in the **C:\\Library\\Scenario.ps1** file in the script pane, replace line **286**

   ```powershell
   $location=(Get-AzOperationalInsightsWorkspace -Name $WorkspaceName -ResourceGroupName $ResourceGroupName).Location
   ```

   with the following code:

   ```powershell
   $location = 'eastus2'
   ```

1. Within the console session to the **WSLabOnboard-DC** VM, in the PowerShell ISE window, in the **C:\\Library\\Scenario.ps1** file in the script pane, select the seventh part of the script between the lines **289** and **328** marked as **#region configure Hybrid Runbook Worker Addresses and Azure Automation Agent Service URL on Log Analytics Gateway**, and then run that part of the script either by selecting the **F8** key or selecting the **Run selection** button in the toolbar of the PowerShell ISE window.

   >**Note**: This part configures the Log Analytics Gateway to connect to the Azure Automation endpoints.

### Task 8: Integrate Storage Spaces Direct cluster nodes with with Azure Monitor

1. Within the console session to the **WSLabOnboard-DC** VM, in the PowerShell ISE window, in the **F:\\WSLab-master\\Scripts\\Scenario.ps1** file in the script pane, select the eighth part of the script between the lines **330** and **372** marked as **#region download and deploy MMA Agent to S2D cluster nodes**, and then run that part of the script either by selecting the **F8** key or selecting the **Run selection** button in the toolbar of the PowerShell ISE window.

   >**Note**: This part installs the Log Analytics agent to Storage Spaces Direct cluster nodes.

   >**Note**: Disregard the warning about breaking changes to the cmdlet **Get-AzOperationalInsightsWorkspaceSharedKey**.

1. Within the console session to the **WSLabOnboard-DC** VM, in the PowerShell ISE window, in the **F:\\WSLab-master\\Scripts\\Scenario.ps1** file in the script pane, select the ninth part of the script between the lines **374** and **401** marked as **#region download and install dependency agent (for service map solution)**, and then run that part of the script either by selecting the **F8** key or selecting the **Run selection** button in the toolbar of the PowerShell ISE window.

   >**Note**: This part installs the Dependency agent, which provides the Service Map functionality.

## Exercise 3: Reviewing Azure integration functionality

### Task 1: Review Log Analytics functionality

1. Within the console session to the **WSLabOnboard-DC** VM, switch back to the browser window displaying the Azure portal interface.
1. In the Azure portal, in the **Search resources, services, and docs** text box at the top of the Azure portal page, enter **Log Analytics workspaces**, and then select the **Enter** key.
1. On the **Log Analytics workspaces** blade, select the workspace you created earlier in this lab (its name starts with the **WSLabWorkspace** prefix).
1. On the Log Analytics workspace blade, in the **General** section, select **Logs**, and then select **Get Started**.
1. In the **Example queries** pane, in the list of **All Queries**, select **Virtual machines**.
In the list of sample queries, select **Top 10 Virtual Machines by CPU utilization in the last 7 days**, and then select **Run**. This will automatically display the corresponding query and its results.
   >**Note**: Review the query and the results.

   >**Note**: The query might result in the syntax error message if the data has not been collected yet. If so, wait for a few minutes and try again or return to this task once you complete the rest of the lab.

### Task 2: Review Azure Automation functionality

1. Within the console session to the **WSLabOnboard-DC** VM, in the browser window displaying the Azure portal, navigate back to the blade displaying the Log Analytics workspace you were reviewing in the previous task.
1. On the Log Analytics workspace blade, in the **Related Resources** section, select **Automation Account**.
1. Note the information regarding the linked Automation account, and then select **Go to account**. You will be redirected to the **WSLabAutomationAccount** blade.
1. On the **WSLabAutomationAccount** blade, in the **Configuration Management** section, select **Inventory**.
1. On the **Inventory** blade, note that you have the option to **Enable** the Inventory solution.
1. Without making any changes, on the **WSLabAutomationAccount** blade, in the **Configuration Management** section, select **Change tracking**.
1. On the **Change tracking** blade, note that you have the option to **Enable** the Change tracking solution.
1. Without making any changes, on the **WSLabAutomationAccount** blade, in the **Process automation** section, select **Hybrid worker groups**.
1. On the **Hybrid worker groups** blade, select the **System hybrid worker groups** tab and note that it contains a separate group for each server that was registered with Azure Automation, with a single worker per group.

   >**Note**: Verify that last seen time for each worker group is within one hour of the current time.

### Task 3: Review Service Map functionality

1. Within the console session to the **WSLabOnboard-DC** VM, in the browser window displaying the Azure portal, navigate back to the blade displaying the Log Analytics workspace you were reviewing in the first task of this exercise.
1. On the Log Analytics workspace blade, in the **General** section, select **Workspace summary**.
1. On the **Overview** blade, review the list of solutions that you implemented in the previous exercise, and then select the **Service Map** tile.

   >**Note**: It may take several minutes for the **Service Map** blade to appear.

1. On the **Service Map** blade, on the **Machines** tab, in the list of monitored servers, select **S2D1** (one of the nodes of the Storage Spaces Direct cluster), select the **+** icon in the diagram to the right of the server list to zoom into the diagram in the center of the blade, and then review the **Summary** pane on the right-hand side of the blade.
1. With the **S2D1** server selected, display each of the sections on the right-hand side of the pane, including **Summary**, **Properties**, **Alerts**, **Log Events**, **Performance**, **Security**, and **Updates**.

   >**Note**: In the **Security** section, if you find the **Logons with a clear text password** entry, select it. You will be automatically redirected to the Log Analytics workspace blade displaying the corresponding Kusto Query Language (KQL) query.

1. With the **S2D1** server selected, zoom in further on the diagram, and then expand the rectangle representing the **S2D2** cluster node by selecting the inverted caret character (**^**).

   >**Note**: Review the list of connections and verify that they involve multiple processes (such as **clussvc** and **System**).

1. Review the diagram and note that it includes connections to `DC.corp.contoso.com` over ports 53 (dns), 67 (bootps), 88, 123, 135, 389, and 445 (there might be others).

    **Note**: These connections are listed even though the **DC** server does not have the Log Analytics and Dependency agents installed.

## Exercise 4: Managing updates to hyperconverged infrastructure

### Task 1: Implement Cluster Aware Updating by using Windows Admin Center

1. Within the console session to the **WSLabOnboard-DC** VM, switch back to the **Windows Admin Center** interface, and in the list of **Tools** of the `S2DCL1.corp.contoso.com` page, select **Updates**.
1. Select **Add Cluster-Aware Updating role**.
1. On the **Updates** panel, select **Check for updates**.
1. Review the list of available updates without making any changes.

   >**Note**: You have the option to **Apply All Updates**. Do not select it.

   >**Note**: You can monitor the status of applying the updates directly from the **Cluster Aware Updating** panel.

### Task 2: Use Azure Automation update management

1. Within the console session to the **WSLabOnboard-DC** VM, switch back to the browser window displaying the **WSLabAutomationAccount** blade in the Azure portal.
1. On the **WSLabAutomationAccount** blade, in the **Update Management** section, select **Update Management**.
1. On the **Update Management** blade, review the list of machines, and identify noncompliant ones.

   >**Note**: To schedule an update deployment, you must first create a computer group.

1. Within the console session to the **WSLabOnboard-DC** VM, in the browser displaying the Azure portal, navigate back to the blade displaying the Log Analytics workspace you were reviewing in the previous exercise.

1. On the Log Analytics workspace blade, in the **General** section, select **Logs**.
1. On the **Example queries** pane, scroll down to the **Virtual machines** section, select it, in the listing of queries, locate the **Missing security or critical updates** from the **Virtual Machine** tile, hover over the **Run** button, and select **Load to editor**.
1. In the editor window, remove the line '| summarize count() by Classification', select **Run** and review results of the query.

   >**Note**: the query lists all of missing security or critical updates.

1. In the editor window, replace the query with the following one:

   ```kql
   Update
   | where UpdateState == 'Needed' and Optional == false and Classification == 'Security Updates' and Approved != false and Computer hassuffix "corp.contoso.com" and Computer !contains "S2D"
   | distinct Computer
   ```

   >**Note**: Computer group queries must use the `distinct Computer` clause.

   >**Note**: The query excludes servers which are members of the Storage Spaces Direct cluster because these are updated by using Cluster Aware Updating.

1. Select **Run** to verify that the query returns the list of noncompliant servers in the `Corp.contoso.com` domain that are not part of the Storage Spaces Direct cluster.
1. Select **Save**; in the drop down list, select **Save**; in the **Save** pane, specify the following settings; and then select **Save**:

   *Table 2: Group query settings*

   |Setting|Value|
   |---|---|
   |Name|`corp.contoso.com non-compliant non S2D servers`|
   |Save as|Function|
   |Function Alias|corp_non_s2d_non_compliant|
   |Save this query as a computer group|enabled|
   |Category|Updates|

1. In the Azure portal, navigate back to the **WSLabAutomationAccount** Automation Account blade, and in the **Update Management** section, select **Update Management**.
1. On the **Update Management** blade, select **Schedule update deployment**.
1. On the **New update deployment** blade, in the **Name** text box, enter **ws01302 update deployment** and ensure that the **Operating system** switch is set to **Windows**.
1. On the **New update deployment** blade, in the **Items to update** section, select **Groups to update**.
1. On the **Select groups** blade, select the **Non-Azure** tab; in the **Available Items** section, in the `corp.contoso.com non-compliant non S2D servers` row, select **add**; and then select **OK**.
1. On the **New update deployment** blade, in the **Items to update** section, select **Machines to update**.
1. On the **Select machines** blade, select the `corp.contoso.com non-compliant non S2D servers` entry, and then select **OK**.
1. Leave the settings within the **Update classifications** drop down list with the default values.

   >**Note**: You can use the **Include/exclude updates** setting to include or exclude individual updates.

1. On the **New update deployment** blade, in the **Items to update** section, select **Schedule settings**, specify the date and time at least 5 minutes ahead of the current date and time, select your current time zone, ensure that **Recurrence** switch is set to **Once**, and then select **OK**.

   >**Note**: You can use the **Pre-scripts + Post-scripts** setting to specify scripts to run before and after patch deployment.

1. Leave the **Maintenance window (minutes)** and **Reboot options** with their default values (**120** and **Reboot if required**, respectively) and select **Create**.
1. Back on the **WSLabAutomationAccount \| Update management** blade, select the **Deployment schedules** tab and ensure that the deployment has been successfully scheduled.

## Exercise 5: Deprovisioning the lab environment

### Task 1: Deprovision the Azure resources

1. Switch to the lab VM.
1. Start a browser, navigate to the Azure portal, and then sign in with the Owner or Contributor role in the Azure subscription you will be using in this lab.
1. Within the Azure portal, select the **Cloud Shell** icon in the toolbar of the portal interface.
1. If you receive a prompt to select either **Bash** or **PowerShell**, select **PowerShell**.
1. From the Cloud Shell pane, run the following to remove all Azure resources you provisioned in this lab:

   ```powershell
   Get-AzResourceGroup -Name 'WS013-02-RG' | Remove-AzResourceGroup -Force -AsJob
   Get-AzResourceGroup -Name 'WSLabWinAnalytics' | Remove-AzResourceGroup -Force -AsJob
   ```

### Task 2: Deprovision the lab resources

1. On the lab VM, start Windows PowerShell ISE as Administrator.
1. In the PowerShell ISE window, open and run the **F:\\WSLab-master\\Scripts\\Cleanup.ps1** script to remove all VMs provisioned in this lab.
