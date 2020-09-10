---
lab:
    title: 'Lab F: Identifying and analyzing metadata of a Storage Spaces Direct cluster (optional)'
    type: 'Answer Key'
    module: 'Module 3: Planning for and implementing Azure Stack HCI Storage'
---
# Lab F answer key: Identifying and analyzing metadata of a Storage Spaces Direct cluster (optional)

## Exercise 1: Identifying and analyzing metadata of a Storage Spaces Direct cluster

### Task 1: Provision the lab environment VMs

1. From the lab VM, in the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to rename **LabConfig.ps1** and **Scenario.ps1**:

   ```powershell
   Set-Location -Path 'F:\WSLab-master\Scripts'
   Move-Item -Path '.\LabConfig.ps1' -Destination '.\LabConfig.m3l5.ps1' -Force -ErrorAction SilentlyContinue
   Move-Item -Path '.\Scenario.ps1' -Destination '.\Scenario.m3l5.ps1' -Force -ErrorAction SilentlyContinue
   ```

1. In the **Administrator: Windows PowerShell ISE** window, open a new tab, and in the **script** pane, paste the following command, and then save it as **F:\\WSLab-master\\Scripts\\LabConfig.ps1**:

   ```powershell
   $LabConfig=@{ DomainAdminName = 'LabAdmin'; AdminPassword = 'LS1setup!'; Prefix = 'WSLab-'; SwitchName = 'LabSwitch'; DCEdition = '4' ; Internet = $true ;AdditionalNetworksConfig = @(); VMs = @()}
   1..6 | % {$VMNames = "6node"; $LABConfig.VMs += @{ VMName = "$VMNames$_" ; Configuration = 'S2D' ; ParentVHD = 'Win2019Core_G2.vhdx'; SSDNumber = 0; SSDSize=800GB ; HDDNumber = 4; HDDSize = 4TB ; MemoryStartupBytes = 2GB ; MemoryMinimumBytes=512MB }}
   $LABConfig.VMs += @{
        VMName = "Management" ;
        Configuration = 'S2D' ;
        ParentVHD = 'Win2019_G2.vhdx';
        SSDNumber = 1;
        SSDSize = 50GB ;
        MemoryStartupBytes = 8GB;
        NestedVirt = $false;
        StaticMemory = $true;
        VMProcessorCount = 4
   }
   ```

1. On the lab VM, in the **Administrator: Windows PowerShell ISE** window, open and run the **F:\\WSLab-master\\Scripts\\3_Deploy.ps1** script to provision VMs for the Storage Spaces Direct environment.

   > **Note:** The script should complete in about 15 minutes. When prompted **Press enter to continue**, select the **Enter** key.

1. When the script completes, in the **Administrator: Windows PowerShell ISE** window, from the **console** pane, run the following command to start the newly provisioned VMs that will host the Storage Spaces Direct environment:

   ```powershell
   Get-VM -Name 'WSLab-Management' | Start-VM
   Start-Sleep 150
   Get-VM | Where-Object Name -like 'WSLab-*node*' | Start-VM -AsJob
   ```

1. On the lab VM, start **Hyper-V Manager** and connect via a console session to **WSLab-DC**. When prompted to sign in, provide the username **CORP\\LabAdmin** and the password **LS1setup!**.
1. In the **WSLab-DC** VM console session, start **Windows PowerShell ISE** as an administrator.
1. From the **Administrator: Windows PowerShell ISE** window, run `slmgr -rearm` and then select **OK**.
1. From the **Administrator: Windows PowerShell ISE** window, run `Restart-Computer -Force`.

 > **Note**: Make sure that the **WSLab-DC** VM is running before you proceed to the next task.

### Task 2: Deploy a Storage Spaces Direct cluster

1. On the lab VM, in the **Server Manager** window, select **Tools**, and then in the drop-down list, select **Hyper-V Manager**.
1. On the lab VM, in the **Hyper-V Manager** console, in the list of virtual machines, right-click or access the context menu for the **WSLab-Management** entry, and then select **Connect** to establish a console session to the **WSLab-Management** VM. When prompted to sign in, provide the username **CORP\\LabAdmin** and the password **LS1setup!**.
1. In the console session to the **WSLab-Management** VM, start Windows PowerShell ISE as Administrator.
1. In the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to provision a six-node Storage Spaces Direct cluster:

   ```powershell
   $clusters = @()
   $clusters += @{Nodes=1..6 | % {"6node$_"} ; Name="6nodeCluster" ; IP="10.0.0.116" }

   Install-WindowsFeature -Name RSAT-Clustering,RSAT-Clustering-Mgmt,RSAT-Clustering-PowerShell,RSAT-Hyper-V-Tools,RSAT-AD-PowerShell, RSAT-ADDS

   # Install features on servers
   Invoke-Command -computername $clusters.nodes -ScriptBlock {
      Install-WindowsFeature -Name "Failover-Clustering","Hyper-V-PowerShell"
   }

   # Restart all servers to finalize installation of Failover Clustering
   Restart-Computer -ComputerName $clusters.nodes -Protocol WSMan -Wait -For PowerShell

   # Create clusters
   foreach ($cluster in $clusters){
      New-Cluster -Name $cluster.Name -Node $cluster.Nodes -StaticAddress $cluster.IP
      Start-Sleep 5
      Clear-DNSClientCache
   }

   # Add file share witness
   foreach ($cluster in $clusters){
      $clusterName = $cluster.Name
      # Create new directory
      $witnessName = $clusterName+"Witness"
      Invoke-Command -ComputerName DC -ScriptBlock {New-Item -Path c:\Shares -Name $using:witnessName -ItemType Directory}
      $accounts = @()
      $accounts += "CORP\$($clusterName)$"
      $accounts += "CORP\Domain Admins"
      New-SmbShare -Name $witnessName -Path "c:\Shares\$witnessName" -FullAccess $accounts -CimSession DC
      # Set NTFS permissions
      Invoke-Command -ComputerName DC -ScriptBlock {(Get-SmbShare $using:witnessName).PresetPathAcl | Set-Acl}
      # Set Quorum
      Set-ClusterQuorum -Cluster $clusterName -FileShareWitness "\\DC\$WitnessName"
   }

   # Enable S2D
   Enable-ClusterS2D -CimSession $clusters.Name -Verbose -Confirm:0
   ```

   > **Note:** Wait for the script to complete. This should take about five minutes.

### Task 3: Examine physical disk owners

1. In the console session to the **WSLab-Management** VM, from the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to display the physical disks of the **6nodecluster** Storage Spaces Direct cluster:

   ```powershell
   Get-PhysicalDisk -CimSession 6nodecluster |ft FriendlyName,Size,Description
   ```

1. In the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to set the **Description** attribute of each physical disk to the name of the cluster node to which the disk is attached for the Storage Spaces Direct cluster:

   ```powershell
   $clusters = @()
   $clusters += @{Nodes=1..6 | % {"6node$_"} ; Name="6nodeCluster" ; IP="10.0.0.116" }
   foreach ($clusterName in ($clusters.Name | select -Unique)){
      $storageNodes=Get-StorageSubSystem -CimSession $clusterName -FriendlyName Clus* | Get-StorageNode
      foreach ($storageNode in $storageNodes){$storageNode | Get-PhysicalDisk -PhysicallyConnected -CimSession $storageNode.Name | Set-PhysicalDisk -Description $storageNode.Name -CimSession $storageNode.Name}
   }
   ```

1. In the console session to the **WSLab-Management** VM, from the **console** pane of the **Administrator: Windows PowerShell ISE** window, rerun the following command to display physical disks of **6nodecluster** Storage Spaces Direct cluster, this time with the **Description** attribute containing the name of the owner node:

   ```powershell
   Get-PhysicalDisk -CimSession 6nodecluster |ft FriendlyName,Size,Description
   ```

### Task 4: Explore metadata of the storage pool

1. In the console session to the **WSLab-Management** VM, from the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to display the number of disks with metadata for the six-node cluster:

   ```powershell
   foreach ($clusterName in ($clusters.Name | select -Unique)){
     Get-StoragePool -CimSession $clusterName |
     Get-PhysicalDisk -HasMetadata -CimSession $clusterName |
     Sort-Object Description |
     Format-Table DeviceId,FriendlyName,SerialNumber,MediaType,Description
   }
   ```

   > **Note:** The number of disks with metadata depends on the size of the cluster, as the following table displays:

   *Table 1: The number of disks with metadata*

   |Number of nodes (fault domains)|Number of disks with metadata|
   |---|---|
   |2|6|
   |3|6|
   |4|8|
   |5|5|
   |6|5|

   > **Note:** In a six-node cluster, metadata is located on five of the six nodes. This means that if you lose random half nodes, 50 percent of the storage pool will go offline.

1. In the console session to the **WSLab-Management** VM, switch to the **Server Manager** window, select **Tools**, and then in the **Tools** drop-down list, select **Failover Cluster Manager**.
1. In the **Failover Cluster Manager** window, right-click or access the context menu for the **Failover Cluster Manager** node, and then in the context menu, select **Connect to cluster**.
1. In the **Select Cluster** dialog box, in the **Cluster name** text box, enter `6nodecluster.corp.contoso.com`, and then select **OK**.
1. In the **Failover Cluster Manager** window, select **Nodes**, and then review the list of nodes.
1. In the **Failover Cluster Manager** window, in the **Storage** node tree, select **Pools**, and then verify that it contains a single pool named **Cluster Pool 1**.
1. Select the **Cluster Pool 1** entry, and on the **Cluster Pool 1** pane, examine its properties by selecting the **Summary** tab, followed by the **Virtual Disks** and **Physical Disks** tabs.
1. In the console session to the **WSLab-Management** VM, switch to the **Administrator: Windows PowerShell ISE** window, and then from the **script** pane, run the following command to capture the list of three nodes hosting the storage pool metadata of the six-node cluster:

   ```powershell
   $nodesToShutDown = (Get-StoragePool -CimSession 6nodecluster |
   Get-PhysicalDisk -HasMetadata -CimSession $clusterName | Select-Object -First 3).Description
   ```

1. In the console session to the **WSLab-Management** VM, from the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to shut down the three nodes hosting the metadata of the six-node cluster:

   ```powershell
   Stop-Computer -ComputerName $nodesToShutDown -Force
   ```

1. In the console session to the **WSLab-Management** VM, switch to the **Failover Cluster Manager** window, and then in the **Storage** node tree, with the **Pools** node selected, verify that the **Cluster Pool 1** storage pool has a status of **Failed**.

   > **Note:** It might take a few minutes before the storage pool reaches the **Failed** status.

1. In the **Failover Cluster Manager** window, on the **Actions** pane, select **Show Critical Events**, and in the list of events, locate the most recent event that references the storage pool failure because of the lack of quorum of healthy disks.
1. In the **Critical events for Cluster Pool 1** window, review the event, and then select **Close**.
1. Switch to the lab VM, and then in the **Hyper-V Manager** console, in the list of virtual machines, select the VM you shut down earlier in this task, and then in the **Actions** pane, in the **Selected Virtual Machines** section, select **Start**.
1. Switch to the **WSLab-Management** VM, in the **Failover Cluster Manager** window, right-click or access the context menu for the **Cluster Pool 1** entry, and in the context menu, select **Bring Online**.

### Task 5: Explore metadata of a volume

1. In the console session to the **WSLab-Management** VM, from the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to create a volume on the six-node cluster:

   ```powershell
   Invoke-Command -ComputerName ($clusters.Name | select -Unique) -ScriptBlock {New-Volume -FriendlyName labVolume -Size 100GB}
   ```

1. In the console session to the **WSLab-Management** VM, from the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to display the metadata of the newly created volume on the six-node cluster:

   ```powershell
   foreach ($clusterName in ($clusters.Name | select -Unique)){
      Get-VirtualDisk -FriendlyName labVolume -CimSession $clusterName |
      Get-PhysicalDisk -HasMetadata -CimSession $clusterName |
      Sort-Object Description |
      Format-table DeviceId,FriendlyName,SerialNumber,MediaType,Description
   }
   ```

   > **Note:** Based on the output, you can determine whether the number of disks with metadata matches the number of disks used by the storage pool metadata.

### Task 6: Explore metadata of a scoped volume

1. In the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to create scoped volumes on the six-node cluster:

   ```powershell
   $faultDomains = Get-StorageFaultDomain -Type StorageScaleUnit -CimSession 6nodecluster | Sort FriendlyName
   New-Volume -FriendlyName "2Scope-Volume" -Size 100GB -StorageFaultDomainsToUse ($faultDomains | Get-Random -Count 2) -CimSession 6nodecluster -StoragePoolFriendlyName S2D* -NumberOfDataCopies 2
   New-Volume -FriendlyName "3Scope-Volume" -Size 100GB -StorageFaultDomainsToUse ($faultDomains | Get-Random -Count 3) -CimSession 6nodecluster -StoragePoolFriendlyName S2D*
   New-Volume -FriendlyName "4Scope-Volume" -Size 100GB -StorageFaultDomainsToUse ($faultDomains | Get-Random -Count 4) -CimSession 6nodecluster -StoragePoolFriendlyName S2D*
   New-Volume -FriendlyName "5Scope-Volume" -Size 100GB -StorageFaultDomainsToUse ($faultDomains | Get-Random -Count 5) -CimSession 6nodecluster -StoragePoolFriendlyName S2D*
   New-Volume -FriendlyName "6Scope-Volume" -Size 100GB -StorageFaultDomainsToUse ($faultDomains | Get-Random -Count 6) -CimSession 6nodecluster -StoragePoolFriendlyName S2D*
   ```

   > **Note:** For a six-node cluster, set the number of a volume’s scopes to four. The additional volumes in this exercise aren’t used as an example of their practical use, but rather as an illustration about how different scope values affect volume distribution.

1. In the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to display the metadata of the newly created volumes on the six-node cluster:

   ```powershell
   $friendlyNames=2..6 | % {"$($_)Scope-Volume"}
   foreach ($friendlyName in $friendlyNames){
      Write-Host -Object "$friendlyName" -ForeGroundColor Cyan
      Get-VirtualDisk -FriendlyName $friendlyName -CimSession 6nodecluster |
      Get-PhysicalDisk -HasMetadata -CimSession 6nodecluster |
      Sort-Object Description |
      Format-Table DeviceId,FriendlyName,SerialNumber,MediaType,Description
   }
   ```

   > **Note:** Review the output and note that the number of cluster nodes containing metadata of each volume matches the scope determined by the value of the **StorageFaultDomainsToUse** parameter of the **New-Volume** cmdlet.

### Task 7: Deprovision the lab resources

1. Switch to the lab VM, and in the **Administrator: Windows PowerShell ISE** window, open and run the **F:\\WSLab-master\\Scripts\\Cleanup.ps1** script to remove all VMs provisioned in this lab. When prompted, on the **console** pane, enter **Y**, and then select **Enter**.
1. After the script completes, select any key.
1. In the **Administrator: Windows PowerShell ISE** window, close the tab displaying the **F:\\WSLab-master\\Scripts\\Cleanup.ps1** script.
