---
lab:
    title: 'Lab B: Managing storage of a Storage Spaces Direct cluster by using Windows Admin Center and Windows PowerShell'
    module: 'Module 3: Planning for and implementing Azure Stack HCI Storage'
---
# Lab B: Managing storage of a Storage Spaces Direct cluster by using Windows Admin Center and Windows PowerShell

## Scenario

Now that you have provisioned a Storage Spaces Direct cluster in an automated manner by using Windows PowerShell, you want to determine whether you can minimize administrative effort associated with remediating disk failures in a Storage Spaces Direct cluster by leveraging its resiliency and self-healing capabilities.

## Objectives

After completing this lab, you'll be able to manage storage of a Storage Spaces Direct cluster by using Windows Admin Center and Windows PowerShell.

## Estimated time: 50 minutes

## Lab setup

To connect to the virtual machine (VM) for the lab, follow the steps provided to you by the lab hosting provider.

## Exercise 1: Managing storage of a Storage Spaces Direct cluster by using Windows Admin Center and Windows PowerShell

You want to determine whether you can minimize the administrative effort associated with remediating disk failures in a Storage Spaces Direct cluster. You will do this by examining its resiliency and self-healing capabilities using Windows Admin Center and Windows PowerShell.

The main tasks for this exercise are as follows:

1. Review the installation of the Storage Spaces Direct cluster on the lab VMs.
1. Create and manage volumes by using Windows Admin Center.
1. Review the health status of the Storage Spaces Direct cluster.
1. Simulate removing a disk from the Storage Spaces Direct cluster.
1. Simulate returning a disk to the Storage Spaces Direct cluster.
1. Simulate removing a disk and replacing it with a different one.
1. Deprovision the lab resources.

> **Note:** Ensure that the **Scenario.ps1** script you started in the previous lab has completed successfully before you start this exercise.

### Task 1: Review the installation of the Storage Spaces Direct cluster on the lab VMs

1. In the **WSLab-Management** VM console session, in the browser window displaying Windows Admin Center, connect to the `s2d-cluster.corp.contoso.com` cluster, and authenticate as **CORP\LabAdmin** with the password **LS1setup!**.
1. In the browser window displaying Windows Admin Center, on the `s2d-cluster.corp.contoso.com` page, navigate to the inventory of the Storage Spaces Direct cluster volumes.
1. Review the list of volumes and verify that each of them is listed with the **OK** status.
1. In the browser window displaying Windows Admin Center, on the `s2d-cluster.corp.contoso.com` page, navigate to the inventory of the Storage Spaces Direct cluster drives.
1. Review the list of drives and verify that each of them is listed with the **OK** status.
1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, review the virtual switch configuration. Note that each node is connected to an external switch named **SETSwitch**. Review the list of network adapters attached to the switch and verify that the load balancing algorithm is set to **Hyper-V port**.
1. In the browser window displaying Windows Admin Center, from the `s2d-cluster.corp.contoso.com` page, display the **Settings** panel, and verify that the cluster contains a single storage pool.

   > **Note:** You have the option of assigning an arbitrary name to the storage pool.

1. On the **Storage Spaces and pools** page review the cache settings.

   > **Note:** The **Cache mode for HDD** is set by default to **Read/Write** and **Cache mode for SSD** is set to **Write only**. You have the option of modifying these settings.

1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, on the **Settings** panel, in the **Cluster** section, review the following entries:

      - Access point. Verify that **Cluster name** is set to **S2D-Cluster**.
      - Node shutdown behavior. Verify that the setting **Move virtual machines on node shutdown** is enabled.
      - Cluster traffic encryption. Verify that **Core traffic** is set to **Sign** and **Storage traffic** to **Clear text**.
      - Virtual machine load balancing. Verify that **Balance virtual machines** is set to **Always** with **Aggressiveness** set to **Low**.
      - Witness. Verify that **Witness type** is set to **File share witness** with **File share path** set to **\\\\DC\\S2D-ClusterWitness**.

1. In the **WSLab-Management** VM console session, start **Failover Cluster Manager**.
1. In **Failover Cluster Manager**, connect to the `s2d-cluster.corp.contoso.com` cluster, review the list of disks, verify that the cluster contains a single pool named **Cluster Pool 1**, and examine the storage pool properties, including virtual and physical disks.
1. In **Failover Cluster Manager**, select the **Networks** node and note that it contains separate entries for **Management** and **SMB** networks, with two network adapters on each cluster node.

### Task 2: Create and manage volumes by using Windows Admin Center

1. In the browser window displaying Windows Admin Center, navigate to the panel listing inventory of volumes on the **s2d-cluster** Storage Spaces Direct cluster.

   > **Note:** The inventory at this point includes only pre-created **ClusterPerformanceHistory** volume.

1. From the panel listing inventory of volumes on the **s2d-cluster** Storage Spaces Direct cluster, create a new volume with the settings listed in the following table:

   *Table 1: Three-way volume settings*

   |Setting|Value|
   |---|---|
   |Name|Volume01-3wm|
   |Resiliency|Three-way mirror|
   |Size on HDD|100|
   |Size units|GB|

   > **Note:** Review the resulting estimated footprint and the total available storage space.

1. From the panel listing inventory of volumes on the **s2d-cluster** Storage Spaces Direct cluster, repeat the same sequence of steps to configure the settings of a new volume as listed in the following table:

   *Table 2: Mirror-accelerated parity volume settings*

   |Setting|Value|
   |---|---|
   |Name|Volume02-map70|
   |Resiliency|Mirror-accelerated parity|
   |Parity percentage|70% parity, 30% mirror|
   |Size on HDD|100|
   |Size units|GB|

1. Review the resulting estimated footprint and the total available storage space and then select **More options**.
1. In the **More options** section, note the message indicating that to use deduplication and compression, it's necessary to install the **Data Deduplication** role on every server.
1. Create the volume without enabling deduplication and compression.
1. In the **WSLab-Management** VM console session, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to install the **Data Deduplication** Windows Server role service on each cluster node:

   ```powershell
   $servers = @('S2D1','S2D2','S2D3','S2D4')
   Invoke-Command -ComputerName $servers -ScriptBlock {Install-WindowsFeature -Name FS-Data-Deduplication}
   ```

   > **Note:** Wait for the installation to complete before you proceed to the next step. This should take about three minutes.

1. Switch to the browser window displaying the Windows Admin Center interface and enable deduplication on the **Volume02-map70** volume with the **Hyper-V** **Deduplication mode**.

   > **Note:** You might need to close and re-open the browser page displaying the Windows Admin Center interface to account for the installation of the **Data Deduplication** role service. 

1. From the panel displaying configuration of the **Volume02-map70** volume, expand its size to **200 GB**.
1. Review the settings of the volume **Volume02-map70**, including **Optional features**, and verify that it contains **Dual parity** and **Three-way mirror** under **Storage Tiers**.

   > **Note:** You have the option of enabling or disabling encryption and compression, but it's not possible to modify integrity checksum or resiliency settings after the volume is created.

1. From the **Volume02-map70** pane in Windows Admin Center, navigate to the content of **C: > ClusterStorage > Volume02-map70**.

   > **Note:** The Windows Admin Center automatically displays the page with the content of **C: > ClusterStorage > Volume02-map70** on the Hyper-V cluster node, which serves as the owner of the corresponding volume.

1. Copy the **tools.vhdx** file from the **F:\\WSLab-master\\Scripts\\ParentDisks** folder on the lab VM to the **Downloads** folder on the **WSLab-Management** VM.
1. Switch to the console session connected to the **WSLab-Management** VM, and in the browser window displaying Windows Admin Center, create a new folder **C:\\ClusterStorage\\Volume02-map70\\vhdFiles**, and then upload the **tools.vhdx** file into it.

### Task 3: Review the health status of the Storage Spaces Direct cluster

1. In the **WSLab-Management** VM console session, from the **Administrator: Windows PowerShell ISE** window, run the following command to identify the health status of the Storage Spaces Direct cluster:

   ```powershell
   $storagesubsystem = Get-StorageSubSystem -CimSession s2d-cluster -FriendlyName Cl*
   $storagesubsystem
   ```

   > **Note:** Ensure that the **HealthStatus** is listed as **Healthy** and **OperationalStatus** is listed as **OK**.

1. In the **WSLab-Management** VM console session, switch to the browser window displaying Windows Admin Center, and on the `s2d-cluster.corp.contoso.com` page, review the content of **Dashboard** and verify that it reports **Healthy** status for all cluster components.
1. In the **WSLab-Management** VM console session, switch to the **Administrator: Windows PowerShell ISE** window and then run the following command to identify any health faults of the Storage Spaces Direct cluster:

   ```powershell
   Get-HealthFault -CimSession s2d-cluster
   ```

   > **Note:** If there are no health faults, the cmdlet should return **WARNING: s2d-cluster: There aren't any faults right now**.

1. In the **WSLab-Management** VM console session, from the **Administrator: Windows PowerShell ISE** window, run the following command to identify the health actions of the Storage Spaces Direct cluster.

   ```powershell
   $storagesubsystem | Get-StorageHealthAction -CimSession s2d-cluster
   ```

   > **Note:** Verify that there are no pending health actions.

1. In the **WSLab-Management** VM console session, from the **Administrator: Windows PowerShell ISE** window, run the following command to identify the status of virtual disks of the Storage Spaces Direct cluster:

   ```powershell
   Get-VirtualDisk -CimSession s2d-cluster | Sort-Object FriendlyName
   ```

   > **Note:** Ensure that the **HealthStatus** is listed as **Healthy** and **OperationalStatus** as **OK**.

### Task 4: Simulate removing a disk from the Storage Spaces Direct cluster

1. Switch to the lab VM, and from the **Administrator: Windows PowerShell ISE** window, run the following command to choose a random disk in one of the nodes of the S2D cluster:

   ```powershell
   $diskToPull = Get-VM -Name WSLab-s2d* | Get-VMHardDiskDrive | Where-Object ControllerLocation -ge 1 | Get-Random
   $diskToPull
   ```

1. On the lab VM, from the **Administrator: Windows PowerShell ISE** window, run the following command to simulate removal of the disk you identified in the previous step:

   ```powershell
   $pulledDiskPath = $diskToPull.Path
   $diskToPull | Remove-VMHardDiskDrive
   ```

1. Switch to the console session connected to the **WSLab-Management** VM, and from the **Administrator: Windows PowerShell ISE** window, rerun the following command to identify the status of the virtual disks of the Storage Spaces Direct cluster:

   ```powershell
   Get-VirtualDisk -CimSession s2d-cluster | Sort-Object FriendlyName
   ```

   > **Note:** Verify that **HealthStatus** is listed as **Warning** and **OperationalStatus** as **Incomplete** for all virtual disks.

1. In the **WSLab-Management** VM console session, from the **Administrator: Windows PowerShell ISE** window, run the following command to identify the health status of the Storage Spaces Direct disks:

   ```powershell
   Get-PhysicalDisk -CimSession s2d-cluster
   ```

   > **Note:** Review the output of the cmdlet and note that the **Operational Status** of one of the disks is listed as **Lost Communication**.

1. In the **WSLab-Management** VM console session, from the **Administrator: Windows PowerShell ISE** window, rerun the following command to identify the health status of the Storage Spaces Direct cluster:

   ```powershell
   Get-HealthFault -CimSession s2d-cluster
   ```

   > **Note:** Review the faults displayed by the Health service.

1. In the **WSLab-Management** VM console session, switch to the browser window displaying Windows Admin Center, navigate to the volume summary pane of the `s2d-cluster.corp.contoso.com` Storage Spaces Direct cluster and review the listing of alerts.
1. In the **WSLab-Management** VM console session, in the browser window displaying Windows Admin Center, navigate to the volume inventory pane of the `s2d-cluster.corp.contoso.com` Storage Spaces Direct cluster.
1. Review the list of volumes and identify the ones which are listed with the **Needs repair** status.
1. In the **WSLab-Management** VM console session, in the browser window displaying Windows Admin Center, navigate to the drive summary pane of the `s2d-cluster.corp.contoso.com` Storage Spaces Direct cluster and review the listing of alerts.
1. In the **WSLab-Management** VM console session, in the browser window displaying Windows Admin Center, navigate to the drive inventory pane of the `s2d-cluster.corp.contoso.com` Storage Spaces Direct cluster.
1. In the drive inventory, identify the drive with the **Lost communication** status.
1. In the browser window displaying Windows Admin Center, on the `s2d-cluster.corp.contoso.com` page, navigate to the **Dashboard** pane and review its contents, verifying that it includes alerts indicating a drive issue.

### Task 5: Simulate returning a disk to the Storage Spaces Direct cluster

1. Switch to the lab VM, and from the **Administrator: Windows PowerShell ISE** window, run the following command to simulate returning the disk removed in the previous task to the same node of the Storage Spaces Direct cluster:

   ```powershell
   Add-VMHardDiskDrive -VMName $disktopull.VMName -Path $pulledDiskPath
   ```

1. Switch to the console session connected to the **WSLab-Management** VM, and from the **Administrator: Windows PowerShell ISE** window, rerun the following command to identify the status of the virtual disks of the Storage Spaces Direct cluster:

   ```powershell
   Get-VirtualDisk -CimSession s2d-cluster | Sort-Object FriendlyName
   ```

   > **Note:** The **HealthStatus** of virtual disks should be listed again as **Healthy**, with **OperationalStatus** listed as **OK**. If you observe one of the virtual disks listed as **InService**, wait for about one minute and repeat this step.

1. In the **WSLab-Management** VM console session, from the **Administrator: Windows PowerShell ISE** window, rerun the following command to identify the health status of the Storage Spaces Direct cluster:

   ```powershell
   Get-HealthFault -CimSession s2d-cluster
   ```

   > **Note:** The storage subsystem should return to the healthy status in about five minutes, so you might need to wait and rerun the cmdlet if you are still observing messages indicating faults.

1. In the **WSLab-Management** VM console session, switch to the browser window displaying Windows Admin Center, navigate to the **volume summary** pane of the `s2d-cluster.corp.contoso.com` Storage Spaces Direct cluster and review listing of alerts.
1. In the **WSLab-Management** VM console session, in the browser window displaying Windows Admin Center, navigate to the **volume inventory** pane of the `s2d-cluster.corp.contoso.com` Storage Spaces Direct cluster.
1. Review the list of volumes and verify that each of them is listed with the **OK** status.
1. In the **WSLab-Management** VM console session, in the browser window displaying Windows Admin Center, navigate to the **drive summary** pane of the `s2d-cluster.corp.contoso.com` Storage Spaces Direct cluster and verify that there are no alerts.
1. In the **WSLab-Management** VM console session, in the browser window displaying Windows Admin Center, navigate to the **drive inventory** pane of the `s2d-cluster.corp.contoso.com` Storage Spaces Direct cluster.
1. In the drive inventory, verify that all drives are listed with the **OK** status.
1. In the browser window displaying Windows Admin Center, on the `s2d-cluster.corp.contoso.com` page, navigate to the **Dashboard** pane and review its contents, verifying that it reports **Healthy** status for all cluster components and displays a single alert indicating the sync operation.
1. In the **WSLab-Management** VM console session, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to identify the health status of the Storage Spaces Direct cluster:

   ```powershell
   $storagesubsystem = Get-StorageSubSystem -CimSession s2d-cluster -FriendlyName Cl*
   $storagesubsystem
   ```

   > **Note:** Before you proceed to the next task, verify that the **HealthStatus** is listed as **Healthy** and that the **OperationalStatus** is listed as **OK**.

### Task 6: Simulate removing a disk and replacing it with a different one

1. Switch to the lab VM, and from the **Administrator: Windows PowerShell ISE** window, run the following command to choose a random disk from one of the nodes of the Storage Spaces Direct cluster:

   ```powershell
   $diskToPull = Get-VM -Name WSLab-s2d* | Get-VMHardDiskDrive | Where-Object ControllerLocation -ge 1 | Get-Random
   $diskToPull
   ```

1. On the lab VM, from the **Administrator: Windows PowerShell ISE** window, run the following command to simulate removal of the disk you identified in the previous step:

   ```powershell
   $pulledDiskPath = $diskToPull.Path
   $diskToPull | Remove-VMHardDiskDrive
   ```

1. On the lab VM, from the **Administrator: Windows PowerShell ISE** window, run the following command to simulate replacing the removed disk with another one:

   ```powershell
   $newDiskPath = "$(($pulledDiskPath).Substring(0,$pulledDiskPath.Length-5))_NEW.vhdx"
   New-VHD -Path $newDiskPath -SizeBytes 4TB
   Add-VMHardDiskDrive -VMName $diskToPull.VMName -Path $newDiskPath
   ```

1. Switch to the console session connected to the **WSLab-Management** VM, and from the **Administrator: Windows PowerShell ISE** window, rerun the following command to identify the status of the virtual disks of the Storage Spaces Direct cluster:

   ```powershell
   Get-VirtualDisk -CimSession s2d-cluster | Sort-Object FriendlyName
   ```

   > **Note:** Verify that the **OperationalStatus** for virtual disks is listed **Incomplete** and **HealthStatus** is listed as **Warning**.

1. In the **WSLab-Management** VM console session, from the **Administrator: Windows PowerShell ISE** window, rerun the following command to verify that the repair and regeneration jobs are in progress on the Storage Spaces Direct cluster:

   ```powershell
   $storagesubsystem = Get-StorageSubSystem -CimSession s2d-cluster -FriendlyName Cl*
   $storagesubsystem | Get-StorageJob
   ```

   > **Note:** If you don't observe any jobs, wait one minute and rerun the cmdlets.

1. In the **WSLab-Management** VM console session, from the **Administrator: Windows PowerShell ISE** window, run the following command to identify the health status of the Storage Spaces Direct disks:

   ```powershell
   Get-PhysicalDisk -CimSession s2d-cluster
   ```

   > **Note:** Review the output of the cmdlet and note that **Operational Status** of one of the disks is listed as **{Removing From Pool, Lost Communication}** and **Usage** is listed as **Retired**.

1. In the **WSLab-Management** VM console session, from the **Administrator: Windows PowerShell ISE** window, run the following command to identify the retired disk:

   ```powershell
   Get-PhysicalDisk -CimSession s2d-cluster -Usage retired
   ```

1. In the **WSLab-Management** VM console session, from the **Administrator: Windows PowerShell ISE** window, run the following command to identify the health status of the Storage Spaces Direct cluster.

   ```powershell
   Get-HealthFault -CimSession s2d-cluster
   ```

1. Review the output of the cmdlet and note that the drive will be automatically retired after 15 minutes of lost communication.

1. In the **WSLab-Management** VM console session, switch to the browser window displaying Windows Admin Center, navigate to the **volume summary** pane of the `s2d-cluster.corp.contoso.com` Storage Spaces Direct cluster, and review the listing of alerts.
1. In the **WSLab-Management** VM console session, in the browser window displaying Windows Admin Center, navigate to the volume inventory pane of the `s2d-cluster.corp.contoso.com` Storage Spaces Direct cluster.
1. Review the list of volumes and verify that each of them is listed with the **Needs repair** status.
1. In the **WSLab-Management** VM console session, in the browser window displaying Windows Admin Center, navigate to the **drive summary** pane of the `s2d-cluster.corp.contoso.com` Storage Spaces Direct cluster, and review the listing of alerts.
1. In the **WSLab-Management** VM console session, in the browser window displaying Windows Admin Center, navigate to the **drive inventory** pane of the `s2d-cluster.corp.contoso.com` Storage Spaces Direct cluster.
1. In the drive inventory, identify the drive with the **Retired, Removing from pool, Lost communication** status.
1. In the browser window displaying Windows Admin Center, on the `s2d-cluster.corp.contoso.com` page, navigate to the **Dashboard** pane and review its contents, verifying that it includes alerts that indicate a drive issue.
1. In the **WSLab-Management** VM console session, switch to the **Administrator: Windows PowerShell ISE** window and run the following command to verify whether the retired disk has been automatically removed from the Storage Spaces Direct cluster:

   ```powershell
   Get-PhysicalDisk -CimSession s2d-cluster -Usage retired
   ```

   > **Note:** Verify that the command doesn't return any output. If that's not the case, wait a few minutes and rerun the command.

1. In the **WSLab-Management** VM console session, switch to the browser window displaying Windows Admin Center, navigate to the **volume summary** pane of the `s2d-cluster.corp.contoso.com` Storage Spaces Direct cluster, and review the listing of alerts.
1. In the **WSLab-Management** VM console session, in the browser window displaying Windows Admin Center, navigate to the **volume inventory** pane of the `s2d-cluster.corp.contoso.com` Storage Spaces Direct cluster.
1. Review the list of volumes and verify that each of them is listed with the **OK** status.
1. In the **WSLab-Management** VM console session, in the browser window displaying Windows Admin Center, navigate to the **drive summary** pane of the `s2d-cluster.corp.contoso.com` Storage Spaces Direct cluster, and verify that there are no alerts.
1. In the **WSLab-Management** VM console session, in the browser window displaying Windows Admin Center, navigate to the **drive inventory** pane of the `s2d-cluster.corp.contoso.com` Storage Spaces Direct cluster.
1. In the drive inventory, verify that all drives are listed with the **OK** status.
1. In the browser window displaying Windows Admin Center, on the `s2d-cluster.corp.contoso.com` page, navigate to the **Dashboard** pane and review its contents, verifying that it reports **Healthy** status for all cluster components and a single alert indicating the sync operation.

### Task 7: Deprovision the lab resources

1. Switch to the lab VM.
1. On the lab VM, from the **Administrator: Windows PowerShell ISE** window, run the **F:\\WSLab-master\\Scripts\\Cleanup.ps1** script to remove all VMs provisioned in this lab.
1. In the **Administrator: Windows PowerShell ISE** window, close the tab displaying the **F:\\WSLab-master\\Scripts\\Cleanup.ps1** script.

## Results

After completing this lab, you will have managed storage in a Storage Spaces Direct cluster by using Windows Admin Center and Windows PowerShell.
