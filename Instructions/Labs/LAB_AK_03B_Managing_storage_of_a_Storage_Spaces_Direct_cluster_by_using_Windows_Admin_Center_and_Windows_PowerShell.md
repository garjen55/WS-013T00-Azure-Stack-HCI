---
lab:
    title: 'Lab B: Managing storage of a Storage Spaces Direct cluster by using Windows Admin Center and Windows PowerShell'
    type: 'Answer Key'
    module: 'Module 3: Planning for and implementing Azure Stack HCI Storage'
---
# Lab B answer key: Managing storage of a Storage Spaces Direct cluster by using Windows Admin Center and Windows PowerShell

## Exercise 1: Managing storage of a Storage Spaces Direct cluster by using Windows Admin Center and Windows PowerShell

> **Note:** Ensure that the **Scenario.ps1** script you started in the previous lab has completed successfully before you start this exercise.

### Task 1: Review the installation of the Storage Spaces Direct cluster on the lab VMs

1. In the console session to the **WSLab-Management** VM, in the browser window displaying the Windows Admin Center interface, on the **All connections** page, select **+ Add**.
1. On the **Add or create resources** pane, in the **Server clusters** section, select **Add**.
1. In the **Cluster name** text box, enter `s2d-cluster.corp.contoso.com`, and then select the **Use another account for this connection** option.
1. In the **Username** text box, enter **CORP\LabAdmin**, in the **Password** text box, enter **LS1setup!**, select **Connect with account**, and then select **Add**.
1. In the console session to the **WSLab-Management** VM, in the browser window displaying the Windows Admin Center interface, on the **All connections** page, select the `s2d-cluster.corp.contoso.com` entry.
1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, in the **Storage** section, select **Volumes**, and then on the **Volumes** pane, select the **Inventory** tab.
1. Review the list of volumes and verify that each of them is listed with a status of **OK**.
1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, in the **Storage** section, select **Drives**.
1. On the **Drives** panel, select the **Inventory** tab, and then review the list of drives and verify that each of them is listed with a status of **OK**.
1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, in the **Networking** section, select **Virtual switches**, on the **Virtual switches** panel, note that each node is connected to an external switch named **SETSwitch**, select one of the **SETSwitch** entries, and then select **Settings**.
1. On the **Settings for SETSwitch** panel, review the list of attached network adapters and the load balancing algorithm (set to **Hyper-V port**), and then select **Close**.
1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, select **Settings**.
1. On the **Settings** panel, select **Storage Spaces and pools** and verify that the cluster contains a single storage pool.

   > **Note:** You have the option to assign an arbitrary name to the storage pool.

1. On the **Storage Spaces and pools** page review the cache settings.

   > **Note:** The **Cache mode for HDD** is set by default to **Read/Write** and the **Cache mode for SSD** is set to **Write only**. You have the option to modify these settings.

1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, on the **Settings** panel, in the **Cluster** section, select the following entries:

      - Access point. Verify that **Cluster name** is set to **S2D-Cluster**.
      - Node shutdown behavior. Verify that the setting **Move virtual machines on node shutdown** is enabled.
      - Cluster traffic encryption. Verify that **Core traffic** is set to **Sign** and **Storage traffic** is set to **Clear text**.
      - Virtual machine load balancing. Verify that **Balance virtual machines** is set to **Always** with **Aggressiveness** set to **Low**.
      - Witness. Verify that **Witness type** is set to **File share witness** with **File share path** set to **\\\\DC\\S2D-ClusterWitness**.

1. In the console session to the **WSLab-Management** VM, switch to the **Server Manager** window, select **Tools**, and in the **Tools** drop-down menu, select **Failover Cluster Manager**.
1. In the **Failover Cluster Manager** window, right-click or access the context menu for the **Failover Cluster Manager** node, and then in the context menu, select **Connect to cluster**.
1. In the **Select Cluster** dialog box, in the **Cluster name** text box, enter `s2d-cluster.corp.contoso.com`, and then select **OK**.
1. In the **Failover Cluster Manager** window, in the **Storage** node tree, select **Disks**, and then review the list of disks.
1. In the **Failover Cluster Manager** window, in the **Storage** node tree, select **Pools**, and then verify that it contains a single pool named **Cluster Pool 1**.
1. Select the **Cluster Pool 1** entry, and in the **Cluster Pool 1** pane, examine its properties by selecting the **Summary** tab, followed by the **Virtual Disks** and **Physical Disks** tabs.
1. In the **Failover Cluster Manager** window, select the **Networks** node and note that it contains separate entries for the **Management** and **SMB** networks.
1. Select the **SMB** entry, and then select the **Network connections** tab and note that it uses two network adapters on each cluster node.

### Task 2: Create and manage volumes by using Windows Admin Center

1. In the browser window displaying the Windows Admin Center interface, on the **s2d-cluster** page, in the **Storage** section, select **Volumes**.

1. On the **Volumes** pane, select **Inventory**, and then select **Create**.

   > **Note:** At this point, the inventory includes only the pre-created **ClusterPerformanceHistory** volume.

1. On the **Create volume** pane, specify the settings listed in the following table:

   *Table 1: Three-way volume settings*

   |Setting|Value|
   |---|---|
   |Name|Volume01-3wm|
   |Resiliency|Three-way mirror|
   |Size on HDD|100|
   |Size units|GB|

1. Review the resulting estimated footprint and the total available storage space, and then select **Create** twice.

1. On the **Create volume** pane, specify the settings listed in the following table:

   *Table 2: Mirror-accelerated parity volume settings*

   |Setting|Value|
   |---|---|
   |Name|Volume02-map70|
   |Resiliency|Mirror-accelerated parity|
   |Parity percentage|70% parity, 30% mirror|
   |Size on HDD|100|
   |Size units|GB|

1. Review the resulting estimated footprint and the total available storage space, and then select **More options**.

1. In the **More options** section, note the message indicating that to use deduplication and compression, it's necessary to install the **Data Deduplication** role on every server.

1. On the **Create volume** pane, select **Create**.

1. To install the **Data Deduplication** Windows Server role service on each cluster node, in the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   $servers = @('S2D1','S2D2','S2D3','S2D4')
   Invoke-Command -ComputerName $servers -ScriptBlock {Install-WindowsFeature -Name FS-Data-Deduplication}
   ```

   > **Note:** Wait for the installation to complete before you proceed to the next step. This should take about three minutes.

1. Switch to the browser window displaying the Windows Admin Center interface.

1. On the `s2d-cluster.corp.contoso.com` page, on the **Volumes** panel, select the **Volume02-map70** entry, on the **Volumes > Volume Volume02-map70** pane, in the **Optional features** section, set the **Deduplication and compression** switch to **On**, and when prompted for confirmation, in the **Deduplication and compression** message box, select **Start**.

1. On the **Enable deduplication** pane, in the **Deduplication mode** drop-down list, select **Hyper-V** and then select **Enable deduplication**.

    > **Note:** You might need to close and re-open the browser page displaying the Windows Admin Center interface to account for the installation of the **Data Deduplication** role service.

1. On the **Volumes > Volume Volume02-map70** pane, select **Expand**.

1. On the **Expand volume Volume02-map70**, in the **Size on HDD (Current size 99.9 GB)** text box, enter **200**, and then select **Expand**.

1. On the **Volumes** pane, on the **Inventory** tab, in the list of volumes, select the **Volume02-map70** entry.

1. On the **Volume02-map70** pane, review the existing settings, including **Optional features**. 

1. In the **Related** section, select **Storage tiers**, and verify that it contains **Dual parity** and **Three-way mirror**.

   > **Note:** You have the option to enable or disable encryption and compression, but it's not possible to modify integrity checksum or resiliency settings after the volume is created.

1. On the **Volume02-map70** pane, select **Open**.

    > **Note:** The Windows Admin Center automatically displays the page with the content of **C: > ClusterStorage > Volume02-map70** on the Hyper-V cluster node, which serves as the owner of the corresponding volume.

1. Switch to the lab VM and use the copy and paste functionality of the Hyper-V console session to copy the **tools.vhdx** file from the **F:\\WSLab-master\\Scripts\\ParentDisks** folder to the **Downloads** folder on the **WSLab-Management** VM.

1. Switch to the console session connected to the **WSLab-Management** VM, in the browser window displaying the Windows Admin Center interface.

1. On the **C: > ClusterStorage > Volume02-map70** pane, select **New Folder**, and on the **Create New Folder** pane, in the **New folder name** text box, enter **vhdFiles**, and then select **Submit**.

1. On the **Files** pane, select the newly created folder, and on the **C: > ClusterStorage > Volume02-map70 > vhdFiles** pane, select **More**, and then select **Upload**.

1. On the **Upload** pane, select **Select files**, and in the **Open** dialog box, navigate to the **Downloads** folder, select **tools.vhdx**, and then select **Open**.

1. On the **Upload** pane, select **Submit**, and then verify that the upload completed successfully.

### Task 3: Review the health status of the Storage Spaces Direct cluster

1. To identify the health status of the Storage Spaces Direct cluster, in the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   $storagesubsystem = Get-StorageSubSystem -CimSession s2d-cluster -FriendlyName Cl*
   $storagesubsystem
   ```

   > **Note:** Ensure that **HealthStatus** is listed as **Healthy** and **OperationalStatus** as **OK**.

1. In the console session to the **WSLab-Management** VM, switch to the browser window displaying the Windows Admin Center interface and navigate to the **s2d-cluster** page.

1. On the **s2d-cluster** page, select **Dashboard** and review its contents, verifying that it reports a status of **Healthy** for all cluster components.

1. In the console session to the **WSLab-Management** VM, switch to the **Administrator: Windows PowerShell ISE** window.

1. To identify any health faults of the Storage Spaces Direct cluster, from the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   Get-HealthFault -CimSession s2d-cluster
   ```

   > **Note:** If there are no health faults, the cmdlet should return **WARNING: Storage Spaces Direct-cluster: There aren't any faults right now**.

1. In the console session to the **WSLab-Management** VM, from the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to identify the health actions of the Storage Spaces Direct cluster:

   ```powershell
   $storagesubsystem | Get-StorageHealthAction -CimSession s2d-cluster
   ```

   > **Note:** Verify that there are no pending health actions.

1. In the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to identify the status of virtual disks of the Storage Spaces Direct cluster:

   ```powershell
   Get-VirtualDisk -CimSession s2d-cluster | Sort-Object FriendlyName
   ```

   > **Note:** Ensure that the **HealthStatus** is listed as **Healthy** and **OperationalStatus** as **OK**.

### Task 4: Simulate removing a disk from the Storage Spaces Direct cluster

1. Switch to the lab VM and, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to pick a random disk in one of the nodes of the Storage Spaces Direct cluster:

   ```powershell
   $diskToPull = Get-VM -Name WSLab-s2d* | Get-VMHardDiskDrive | Where-Object ControllerLocation -ge 1 | Get-Random
   $diskToPull
   ```

1. On the lab VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to simulate removal of the disk you identified in the previous step:

   ```powershell
   $pulledDiskPath = $diskToPull.Path
   $diskToPull | Remove-VMHardDiskDrive
   ```

1. Switch to the console session connected to the **WSLab-Management** VM and, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, rerun the following command to identify the status of the virtual disks of the Storage Spaces Direct cluster:

   ```powershell
   Get-VirtualDisk -CimSession s2d-cluster | Sort-Object FriendlyName
   ```

   > **Note:** Verify that **HealthStatus** is listed as **Warning** and **OperationalStatus** as **Incomplete** for virtual disks.

1. In the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to identify the health status of the Storage Spaces Direct disks:

   ```powershell
   Get-PhysicalDisk -CimSession s2d-cluster
   ```

   > **Note:** Review the output of the cmdlet and note that the **Operational Status** of one of the disks is listed as **Lost Communication**.

1. In the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, rerun the following command to identify the health status of the Storage Spaces Direct cluster:

   ```powershell
   Get-HealthFault -CimSession s2d-cluster
   ```

   > **Note:** Review the faults displayed by the Health service. You might have to wait a few minutes for the faults to display.

1. In the console session to the **WSLab-Management** VM, switch to the browser window displaying the Windows Admin Center interface, navigate back to the `s2d-cluster.corp.contoso.com` page, and in the **Storage** section, select **Volumes**, and then on the **Volumes** pane, review the content of the **Summary** tab, focusing on the list of **Alerts**.
1. On the **Volumes** pane, select the **Inventory** tab.
1. Review the list of volumes and identify the ones which are listed with the **Needs repair** status.
1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, in the **Storage** section, select **Drives** and then on the **Drives** panel, on the **Summary** tab, review the **Alerts** section.
1. On the **Drives** panel, select the **Inventory** tab and identify the drive with the **Lost communication** status.
1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, select **Dashboard**, and review its contents, verifying that it includes alerts indicating a drive issue.

### Task 5: Simulate returning a disk to the Storage Spaces Direct cluster

1. Switch to the lab VM, and from the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to simulate returning the disk removed in the previous task back to the same node of the Storage Spaces Direct cluster:

   ```powershell
   Add-VMHardDiskDrive -VMName $disktopull.VMName -Path $pulledDiskPath
   ```

1. Switch to the console session connected to the **WSLab-Management** VM, and from the **console** pane of the **Administrator: Windows PowerShell ISE** window, rerun the following command to identify the status of the virtual disks of the Storage Spaces Direct cluster:

   ```powershell
   Get-VirtualDisk -CimSession s2d-cluster | Sort-Object FriendlyName
   ```

   > **Note:** The **HealthStatus** of virtual disks should be listed again as **Healthy**, with **OperationalStatus** listed as **OK**. If you observe one of the virtual disks listed as **InService**, wait for about a minute and repeat this step.

1. In the console session to the **WSLab-Management** VM, from the **console** pane of the **Administrator: Windows PowerShell ISE** window, rerun the following command to identify the health status of the Storage Spaces Direct cluster:

   ```powershell
   Get-HealthFault -CimSession s2d-cluster
   ```

   > **Note:** The storage subsystem should return to the healthy status in about five minutes, so you might need to wait and rerun the cmdlet if you are still observing messages that indicate faults.

1. In the console session to the **WSLab-Management** VM, switch to the browser window displaying the Windows Admin Center interface.
1. On the `s2d-cluster.corp.contoso.com` page, in the **Storage** section, select **Volumes**, and then on the **Volumes** pane, review the content of the **Summary** tab, focusing on the list of **Alerts**.
1. On the **Volumes** pane, select the **Inventory** tab, and then review the list of volumes and verify that each of them is listed with a status of **OK**.
1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, in the **Storage** section, select **Drives**.
1. On the **Drives** panel, on the **Summary** tab, verify that there are no entries in the **Alerts** section.
1. On the **Drives** panel, select the **Inventory** tab and then verify that all drives are listed with a status of **OK**.
1. In the console session to the **WSLab-Management** VM, switch to the browser window displaying the Windows Admin Center interface.
1. On the `s2d-cluster.corp.contoso.com` page, select **Dashboard** and review its contents, verifying that it reports a status of **Healthy** for all cluster components and a single alert indicating the sync operation.
1. In the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to identify the health status of the Storage Spaces Direct cluster:

   ```powershell
   $storagesubsystem = Get-StorageSubSystem -CimSession s2d-cluster -FriendlyName Cl*
   $storagesubsystem
   ```

   > **Note:**  Before you proceed to the next task, verify that the **HealthStatus** is listed as **Healthy** and that the **OperationalStatus** is listed as **OK**.

### Task 6: Simulate removing a disk and replacing it with a different one

1. Switch to the lab VM.
1. To choose a random disk from one of the nodes of the Storage Spaces Direct cluster, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   $diskToPull = Get-VM -Name WSLab-s2d* | Get-VMHardDiskDrive | Where-Object ControllerLocation -ge 1 | Get-Random
   $diskToPull
   ```

1. To simulate removal of the disk you identified in the previous step, in the lab VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   $pulledDiskPath = $diskToPull.Path
   $diskToPull | Remove-VMHardDiskDrive
   ```

1. To simulate replacing the removed disk with another one, in the lab VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   $newDiskPath = "$(($pulledDiskPath).Substring(0,$pulledDiskPath.Length-5))_NEW.vhdx"
   New-VHD -Path $newDiskPath -SizeBytes 4TB
   Add-VMHardDiskDrive -VMName $diskToPull.VMName -Path $newDiskPath
   ```

1. Switch to the console session connected to the **WSLab-Management** VM.
1. To identify the status of the virtual disks of the Storage Spaces Direct cluster, from the **console** pane of the **Administrator: Windows PowerShell ISE** window, rerun the following command:

   ```powershell
   Get-VirtualDisk -CimSession s2d-cluster | Sort-Object FriendlyName
   ```

   > **Note:** Verify that the **OperationalStatus** for virtual disks is listed as **Incomplete** and the **HealthStatus** is listed as **Warning**.

1. To verify that the repair and regeneration jobs are in progress on the Storage Spaces Direct cluster, in the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, rerun the following command:

   ```powershell
   $storagesubsystem = Get-StorageSubSystem -CimSession s2d-cluster -FriendlyName Cl*
   $storagesubsystem | Get-StorageJob
   ```

   > **Note:** If you don't observe any jobs, wait one minute and then rerun the cmdlets.

1. To identify the health status of the Storage Spaces Direct disks, in the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command:

       ```powershell
       Get-PhysicalDisk -CimSession s2d-cluster
       ```

      > **Note:** Review the output of the cmdlet and note that **Operational Status** of one of the disks is listed as **{Removing From Pool, Lost Communication}** and **Usage** is listed as **Retired**.
   
1. To identify the retired disk, in the console session to the **WSLab-Management** VM, from the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   Get-PhysicalDisk -CimSession s2d-cluster -Usage retired
   ```

1. To identify the health status of the Storage Spaces Direct cluster, in the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   Get-HealthFault -CimSession s2d-cluster
   ```

1. Review the output of the cmdlet and note that the drive will be automatically retired after 15 minutes of lost communication.
1. In the console session to the **WSLab-Management** VM, switch to the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, in the **Storage** section, select **Volumes**, and on the **Volumes** pane, select the **Inventory** tab.
1. Review the list of volumes and verify that they are online but listed with a status of **Needs repair**.
1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, in the **Storage** section, select **Drives**, and on the **Drives** panel, on the **Summary** tab, review the entry in the **Alerts** section.
1. On the **Drives** panel, select the **Inventory** tab and identify the drive listed with the status of **Retired, Removing from pool, Lost communication**.
1. To verify whether the retired disk has been automatically removed from the Storage Spaces Direct cluster, in the console session to the **WSLab-Management** VM, switch to the **Administrator: Windows PowerShell ISE** window, and then from the **console** pane, run the following command:

   ```powershell
   Get-PhysicalDisk -CimSession s2d-cluster -Usage retired
   ```

   > **Note:** Verify that the command does not return any output. If that's not the case, wait a few minutes and rerun the command.

1. In the console session to the **WSLab-Management** VM, switch to the browser window displaying the Windows Admin Center interface.
1. On the `s2d-cluster.corp.contoso.com` page, in the **Storage** section, select **Volumes**, and then on the **Summary** tab of the **Volumes** pane, verify that all volumes are healthy.
1. On the **Volumes** pane, select the **Inventory** tab and then review the list of volumes, verifying that each of them is listed with a status of **OK**.
1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, in the **Storage** section, select **Drives**, and then on the **Drives** panel, on the **Summary** tab, verify that there are no entries in the **Alerts** section.
1. On the **Drives** panel, select the **Inventory** tab and verify that all drives are listed with a status of **OK**.
1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, select **Dashboard**, and review its contents, verifying that it reports a status of **Healthy** for all cluster components, and that there are no alerts listed.

### Task 7: Deprovision the lab resources

1. Switch to the lab VM, and in the **Administrator: Windows PowerShell ISE** window, open and run the **F:\\WSLab-master\\Scripts\\Cleanup.ps1** script to remove all VMs provisioned in this lab. When prompted, on the **console** pane, enter **Y**, and then select the **Enter** key.
1. After the script completes, select any key.
1. In the **Administrator: Windows PowerShell ISE** window, close the tab displaying the **F:\\WSLab-master\\Scripts\\Cleanup.ps1** script.
