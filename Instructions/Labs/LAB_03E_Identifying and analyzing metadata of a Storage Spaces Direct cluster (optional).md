---
lab:
    title: 'Lab E: Identifying and analyzing metadata of a Storage Spaces Direct cluster (optional)'
    module: 'Module 3: Planning for and implementing Azure Stack HCI Storage'
---
# Lab E: Identifying and analyzing metadata of a Storage Spaces Direct cluster (optional)

## Scenario

Another resiliency consideration you want to explore is metadata of the storage pool and its components. You want to ensure that you understand resiliency provisions that must be taken into account to protect cluster stability and integrity. You also want to be able to identify how a Storage Spaces Direct cluster maintains information about its data.

## Objectives

After completing this lab, you'll be able to identify and analyze the metadata of a Storage Spaces Direct cluster.

## Estimated time: 35 minutes

## Lab setup

To connect to the VM for the lab, follow the steps provided to you by the lab hosting provider.

## Exercise 1: Identifying and analyzing metadata of a Storage Spaces Direct cluster

### Scenario

To ensure that you understand resiliency provisions that must be taken into account to protect cluster stability and integrity, and to be able to identify how the Storage Spaces Direct cluster maintains information about its data, you need to identify and analyze the metadata of the storage pool and its components.

The main tasks for this exercise are as follows:

1. Provision the lab environment VMs.
1. Deploy a Storage Spaces Direct cluster.
1. Examine physical disk owners.
1. Explore storage pool metadata.
1. Explore metadata of a volume.
1. Explore metadata of a scoped volume.
1. Deprovision the lab resources.

### Task 1: Provision the lab environment VMs

1. From the lab VM, in the **Administrator: Windows PowerShell ISE** window, run the following command to rename **LabConfig.ps1** and **Scenario.ps1**:

   ```powershell
   Set-Location -Path 'F:\WSLab-master\Scripts'
   Move-Item -Path '.\LabConfig.ps1' -Destination '.\LabConfig.m3l4.ps1' -Force -ErrorAction SilentlyContinue
   Move-Item -Path '.\Scenario.ps1' -Destination '.\Scenario.m3l4.ps1' -Force -ErrorAction SilentlyContinue
   ```

1. In the **Administrator: Windows PowerShell ISE** window, from the **script** pane, save the following command as **F:\\WSLab-master\\Scripts\\LabConfig.ps1**:

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

   > **Note:** The script should complete in about 15 minutes.

1. When the script completes, in the **Administrator: Windows PowerShell ISE** window, run the following command to start the newly provisioned VMs that will host the Storage Spaces Direct environment:

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

1. On the lab VM, from **Hyper-V Manager**, and connect via a console session to **WSLab-Management**. When prompted to sign in, provide the username **CORP\\LabAdmin** and the password **LS1setup!**.
1. In the **WSLab-Management** VM console session, start **Windows PowerShell ISE** as **Administrator**.
1. In the **Administrator: Windows PowerShell ISE** window, run the following command to provision a Storage Spaces Direct cluster:

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

   # Enable Storage Spaces Direct
   Enable-ClusterS2D -CimSession $clusters.Name -Verbose -Confirm:0
   ```

   > **Note:** Wait for the script to complete. This might take about 5 minutes.

### Task 3: Examine physical disk owners

1. In the **WSLab-Management** VM console session, in the **Administrator: Windows PowerShell ISE** window, run the following command to display the physical disks of **6nodecluster** Storage Spaces Direct cluster:

   ```powershell
   Get-PhysicalDisk -CimSession 6nodecluster |ft FriendlyName,Size,Description
   ```

1. In the **Administrator: Windows PowerShell ISE** window, run the following command to set the **Description** attribute of each physical disk to the name of the cluster node to which the disk is attached for all Storage Spaces Direct clusters:

   ```powershell
   $clusters = @()
   $clusters += @{Nodes=1..6 | % {"6node$_"} ; Name="6nodeCluster" ; IP="10.0.0.116" }
   foreach ($clusterName in ($clusters.Name | select -Unique)){
      $storageNodes=Get-StorageSubSystem -CimSession $clusterName -FriendlyName Clus* | Get-StorageNode
      foreach ($storageNode in $storageNodes){$storageNode | Get-PhysicalDisk -PhysicallyConnected -CimSession $storageNode.Name | Set-PhysicalDisk -Description $storageNode.Name -CimSession $storageNode.Name}
   }
   ```

1. In the **Administrator: Windows PowerShell ISE** window, re-run the following command to display the physical disks of the **6nodecluster** Storage Spaces Direct cluster, this time with the **Description** attribute containing the name of the owner node:

   ```powershell
   Get-PhysicalDisk -CimSession 6nodecluster |ft FriendlyName,Size,Description
   ```

### Task 4: Explore storage pool metadata

1. In the **WSLab-Management** VM console session, in the **Administrator: Windows PowerShell ISE** window, run the following command to display the number of disks with metadata for the 6-node cluster:

   ```powershell
   foreach ($clusterName in ($clusters.Name | select -Unique)){
     Get-StoragePool -CimSession $clusterName |
     Get-PhysicalDisk -HasMetadata -CimSession $clusterName |
     Sort-Object Description |
     Format-Table DeviceId,FriendlyName,SerialNumber,MediaType,Description
   }
   ```

   > **Note:** The number of disks with metadata depends on size of the cluster, as per the following table:

   *Table 1: Number of disks with metadata*

   |Number of nodes (fault domains)|Number of disks with metadata|
   |---|---|
   |2|6|
   |3|6|
   |4|8|
   |5|5|
   |6|5|

   > **Note:** In a 6-node cluster, metadata is located on five of the six nodes. This means that if you lose random half nodes, 50 percent of the storage pool will go offline.

1. In the **WSLab-Management** VM console session, start **Failover Cluster Manager**, and then connect to `6nodecluster.corp.contoso.com`.
1. In **Failover Cluster Manager**, review the list of nodes.
1. Locate the storage pool named **Cluster Pool 1**, and examine its properties, including virtual disks and physical disks.
1. In the **WSLab-Management** VM console session, switch to the **Administrator: Windows PowerShell ISE** window and run the following command to capture the list of three nodes hosting the storage pool metadata of the 6-node cluster:

   ```powershell
   $nodesToShutDown = (Get-StoragePool -CimSession 6nodecluster |
   Get-PhysicalDisk -HasMetadata -CimSession $clusterName | Select-Object -First 3).Description
   ```

1. In the **Administrator: Windows PowerShell ISE** window, run the following command to shut down the three nodes hosting metadata of the 6-node cluster:

   ```powershell
   Stop-Computer -ComputerName $nodesToShutDown -Force
   ```

1. In the **WSLab-Management** VM console session, switch to **Failover Cluster Manager** and verify that the **Cluster Pool 1** storage pool has a status of **Failed**.

   > **Note:** It might take a few minutes before the storage pool reaches the **Failed** status.

1. In **Failover Cluster Manager**, review the cluster's critical events, and locate the most recent critical event that references the storage pool failure resulting from a lack of quorum of healthy disks.
1. Switch to the lab VM, and then in the Hyper-V Manager console, start the VM that you shut down earlier in this exercise.
1. Switch to the **WSLab-Management** VM, and in **Failover Cluster Manager**, bring the **Cluster Pool 1** online.

### Task 5: Explore metadata of a volume

1. In the **WSLab-Management** VM console session, in the **Administrator: Windows PowerShell ISE** window, run the following command to create a volume on the 6-node cluster:

   ```powershell
   Invoke-Command -ComputerName ($clusters.Name | select -Unique) -ScriptBlock {New-Volume -FriendlyName labVolume -Size 100GB}
   ```

1. In the **Administrator: Windows PowerShell ISE** window, run the following command to display metadata of the newly created volume on the 6-node cluster:

   ```powershell
   foreach ($clusterName in ($clusters.Name | select -Unique)){
      Get-VirtualDisk -FriendlyName labVolume -CimSession $clusterName |
      Get-PhysicalDisk -HasMetadata -CimSession $clusterName |
      Sort-Object Description |
      Format-table DeviceId,FriendlyName,SerialNumber,MediaType,Description
   }
   ```

   > **Note:** Based on the output, you can determine whether the number of disks with metadata matches the one used by the storage pool metadata.

### Task 6: Explore metadata of a scoped volume

1. In the **Administrator: Windows PowerShell ISE** window, run the following command to create scoped volumes on the 6-node cluster:

   ```powershell
   $faultDomains = Get-StorageFaultDomain -Type StorageScaleUnit -CimSession 6nodecluster | Sort FriendlyName
   New-Volume -FriendlyName "2Scope-Volume" -Size 100GB -StorageFaultDomainsToUse ($faultDomains | Get-Random -Count 2) -CimSession 6nodecluster -StoragePoolFriendlyName S2D* -NumberOfDataCopies 2
   New-Volume -FriendlyName "3Scope-Volume" -Size 100GB -StorageFaultDomainsToUse ($faultDomains | Get-Random -Count 3) -CimSession 6nodecluster -StoragePoolFriendlyName S2D*
   New-Volume -FriendlyName "4Scope-Volume" -Size 100GB -StorageFaultDomainsToUse ($faultDomains | Get-Random -Count 4) -CimSession 6nodecluster -StoragePoolFriendlyName S2D*
   New-Volume -FriendlyName "5Scope-Volume" -Size 100GB -StorageFaultDomainsToUse ($faultDomains | Get-Random -Count 5) -CimSession 6nodecluster -StoragePoolFriendlyName S2D*
   New-Volume -FriendlyName "6Scope-Volume" -Size 100GB -StorageFaultDomainsToUse ($faultDomains | Get-Random -Count 6) -CimSession 6nodecluster -StoragePoolFriendlyName S2D*
   ```

   > **Note:** For a 6-node cluster, set the number of a volume’s scopes to four. The additional volumes in this exercise aren’t used as an example of their practical use, but rather as an illustration about how different scope values affect volume distribution. 

1. In the **Administrator: Windows PowerShell ISE** window, run the following command to display metadata of the newly created volumes on the 6-node cluster:

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

1. Switch to the lab VM.
1. On the lab VM, in the **Administrator: Windows PowerShell ISE** window, run the **F:\\WSLab-master\\Scripts\\Cleanup.ps1** script to remove all VMs provisioned in this lab.
1. In the **Administrator: Windows PowerShell ISE** window, close the tab displaying the **F:\\WSLab-master\\Scripts\\Cleanup.ps1** script.

## Results

After completing this lab, you will have identified and analyzed the metadata of a Storage Spaces Direct cluster.
