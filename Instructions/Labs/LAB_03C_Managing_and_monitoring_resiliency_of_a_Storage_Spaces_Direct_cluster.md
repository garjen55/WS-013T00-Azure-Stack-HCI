---
lab:
    title: 'Lab C: Managing and monitoring resiliency of a Storage Spaces Direct cluster'
    module: 'Module 3: Planning for and implementing Azure Stack HCI Storage'
---
# Lab C: Managing and monitoring resiliency of a Storage Spaces Direct cluster

## Scenario

You want to examine resiliency in situations when there are simultaneous cluster node and drive failures. You want to understand how resiliency can protect cluster stability and integrity. To start, you will create tiered volumes and test volume, disk, and cluster resiliency.

## Objectives

After completing this lab, you'll be able to manage and monitor resiliency of a Storage Spaces Direct cluster.

## Estimated time: 55 minutes

## Lab setup

To connect to the VM for the lab, follow the steps provided to you by the lab hosting provider.

## Exercise 1: Managing and monitoring resiliency of a Storage Spaces Direct cluster

### Scenario

You have evaluated the self-healing capabilities of Storage Spaces Direct clusters after removing individual disks. Now you want to examine resiliency in situations when there are simultaneous cluster node and drive failures.

The main tasks for this exercise are as follows:

1. Provision the lab environment VMs.
1. Configure the management server.
1. Create and configure a failover cluster.
1. Configure fault domains on the failover cluster.
1. Enable Storage Spaces Direct on the failover cluster.
1. Review fault domain configuration on the Storage Spaces Direct cluster.
1. Create tiered volumes on a Storage Spaces Direct failover cluster.
1. Test resiliency of the Storage Spaces Direct cluster.
1. Restore failed disks and nodes on the Storage Spaces Direct cluster.
1. Deprovision the lab resources.

### Task 1: Provision the lab environment VMs

1. On the lab VM, in the **Administrator: Windows PowerShell ISE** window, run the following command to rename **LabConfig.ps1** and **Scenario.ps1**:

   ```powershell
   Set-Location -Path 'F:\WSLab-master\Scripts'
   Move-Item -Path '.\LabConfig.ps1' -Destination '.\LabConfig.m3l3.ps1' -Force -ErrorAction SilentlyContinue
   Move-Item -Path '.\Scenario.ps1' -Destination '.\Scenario.m3l3.ps1' -Force -ErrorAction SilentlyContinue
   ```

1. In the **Administrator: Windows PowerShell ISE** window, from the **script** pane, save the following content as **F:\\WSLab-master\\Scripts\\LabConfig.ps1**:

   ```powershell
   $LabConfig=@{ DomainAdminName='LabAdmin'; AdminPassword='LS1setup!'; Prefix = 'WSLab-'; SwitchName = 'LabSwitch'; DCEdition='4'; Internet=$true ; AdditionalNetworksConfig=@(); VMs=@()}
   1..6 | % {
        $VMNames="S2D";
        $LABConfig.VMs += @{
        VMName = "$VMNames$_" ;
        Configuration = 'S2D' ;
        ParentVHD = 'Win2019Core_G2.vhdx';
        SSDNumber = 0;
        SSDSize=800GB ;
        HDDNumber = 12;
        HDDSize= 4TB ;
        MemoryStartupBytes= 1GB;
        NestedVirt=$false
       }
   }
   $LabConfig.VMs += @{
        VMName = 'Management' ;
        Configuration = 'Simple';
        ParentVHD = 'Win2019_G2.vhdx';
        StaticMemory = $true;
        MemoryStartupBytes = 8GB;
        AddToolsVHD = $True;
        DisableWCF = $True;
        VMProcessorCount = 4
   }
   ```

1. In the **Administrator: Windows PowerShell ISE** window, open and run the **F:\\WSLab-master\\Scripts\\3_Deploy.ps1** script to provision VMs for the Storage Spaces Direct environment.

   > **Note:** Select **None** at the Telemetry prompt. The script should complete in about 10 minutes.

1. When the script completes, in the **Administrator: Windows PowerShell ISE** window, run the following command to start the newly provisioned VMs that will host the Storage Spaces Direct environment:

   ```powershell
   Get-VM -Name 'WSLab-Management' | Start-VM
   Start-Sleep 150
   Get-VM | Where-Object Name -like 'WSLab-S2D*' | Start-VM -AsJob
   ```

1. On the lab VM, start **Hyper-V Manager** and connect via a console session to **WSLab-DC**. When prompted to sign in, provide the username **CORP\\LabAdmin** and the password **LS1setup!**.
1. In the **WSLab-DC** VM console session, start **Windows PowerShell ISE** as an administrator.
1. From the **Administrator: Windows PowerShell ISE** window, run `slmgr -rearm` and then select **OK**.
1. From the **Administrator: Windows PowerShell ISE** window, run `Restart-Computer -Force`.

 > **Note**: Make sure that the **WSLab-DC** VM is running before you proceed to the next task.

### Task 2: Configure the management server

1. On the lab VM, from **Hyper-V Manager**, and connect with a console session to **WSLab-Management**. When prompted to sign in, provide the username **CORP\\LabAdmin** and the password **LS1setup!**.

1. In the **WSLab-Management** VM console session, start **Windows PowerShell ISE** as **Administrator**.

1. In the **WSLab-Management** VM console session, in the **Administrator: Windows PowerShell ISE** window, run the following command to install RSAT:

   ```powershell
   Install-WindowsFeature -Name RSAT-Clustering,RSAT-Clustering-Mgmt,RSAT-Clustering-PowerShell,RSAT-Hyper-V-Tools,RSAT-AD-PowerShell,RSAT-ADDS
   ```

   > **Note:** Proceed to the next step without waiting for the installation to complete.

1. In the **WSLab-Management** VM console session, start another instance of **Windows PowerShell ISE** as **Administrator**.

1. From the newly started **Administrator: Windows PowerShell ISE** window, run the following command to download and install the Microsoft Edge (Chromium) browser:

   ```powershell
   $progressPreference='SilentlyContinue'
   Invoke-WebRequest -Uri "https://go.microsoft.com/fwlink/?linkid=2069324&language=en-us&Consent=1" -UseBasicParsing -OutFile "$env:USERPROFILE\Downloads\MicrosoftEdgeSetup.exe"
   Start-Process -FilePath "$env:USERPROFILE\Downloads\MicrosoftEdgeSetup.exe" -Wait
   ```

   > **Note:** Proceed to the next step without waiting for the installation to complete.

1. In the **WSLab-Management** VM console session, start another instance of **Windows PowerShell ISE** as **Administrator**.

1. From the newly started **Administrator: Windows PowerShell ISE** window, run the following command to download and install Windows Admin Center:

   ```powershell
   Invoke-WebRequest -UseBasicParsing -Uri https://aka.ms/WACDownload -OutFile "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v waclog.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > **Note:** Proceed to the next step without waiting for the installation to complete.

1. Switch back to the first **Administrator: Windows PowerShell ISE** window where you initiated the installation of RSAT, and wait for the installation to complete.

1. To configure Kerberos-constrained delegation so as to minimize prompts for credentials when using Windows Admin Center, from the **script** pane, run the following command:

   ```powershell
   $gateway = "Management"
   $nodes = Get-ADComputer -Filter * -SearchBase "ou=workshop,DC=corp,dc=contoso,DC=com"
   $gatewayObject = Get-ADComputer -Identity $gateway
   foreach ($node in $nodes){
    Set-ADComputer -Identity $node -PrincipalsAllowedToDelegateToAccount $gatewayObject
   }
   ```

   > **Note:** Before you proceed to the next step, verify that the Microsoft Edge and Windows Admin Center installations completed.

1. Close the other two instances of the **Administrator: Windows PowerShell ISE** window you opened earlier in this task.

1. Switch to the **Microsoft Edge** browser window, navigate to `https://management.corp.contoso.com`, and when prompted to authenticate, sign in as **CORP\\LabAdmin** with the password **LS1setup!**.

### Task 3: Create and configure a failover cluster

1. To provision a failover cluster, In the **WSLab-Management** VM console session, in the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   $servers = 1..6 | % {"S2D$_"}
   $clusterName = "S2D-Cluster"
   $clusterIP = "10.0.0.111"

   # Install features on servers
   Invoke-Command -computername $servers -ScriptBlock {
       Install-WindowsFeature -Name "Failover-Clustering","Hyper-V-PowerShell","RSAT-Clustering-PowerShell"
   }

   # Restart servers since failover clustering in Windows Server 2019 requires reboot
   Restart-Computer -ComputerName $servers -Protocol WSMan -Wait -For PowerShell

   # Create cluster
   New-Cluster -Name $clusterName -Node $servers -StaticAddress $clusterIP
   Start-Sleep 5
   Clear-DNSClientCache

   # Add File Share Witness
   # Create a new directory
   $witnessName = $clusterName+"Witness"
   Invoke-Command -ComputerName DC -ScriptBlock {New-Item -Path c:\Shares -Name $using:WitnessName -ItemType Directory}
   $accounts = @()
   $accounts += "CORP\$($clusterName)$"
   $accounts += "CORP\Domain Admins"
   New-SmbShare -Name $witnessName -Path "c:\Shares\$witnessName" -FullAccess $accounts -CimSession DC -ErrorAction SilentlyContinue
   # Set NTFS permissions
   Invoke-Command -ComputerName DC -ScriptBlock {(Get-SmbShare $using:witnessName).PresetPathAcl | Set-Acl}
   # Set Quorum
   Set-ClusterQuorum -Cluster $clusterName -FileShareWitness "\\DC\$witnessName"
   ```

   > **Note:** Wait until the script completes before you proceed to the next task. This should take about 10 minutes.

### Task 4: Configure fault domains on the failover cluster

1. To configure fault domains on the Storage Spaces Direct cluster, in the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   $clusterName="S2D-Cluster"

   # Create Fault domains with PowerShell (note: Enable-ClusterS2D will fail in Windows Server 2016. Fixed in Windows Server 2019)
   New-ClusterFaultDomain -Name "Rack01" -FaultDomainType Rack -Location "Contoso HQ, Room 4010, Aisle A, Rack 01" -CimSession $clusterName
   New-ClusterFaultDomain -Name "Rack02" -FaultDomainType Rack -Location "Contoso HQ, Room 4010, Aisle A, Rack 02" -CimSession $clusterName
   New-ClusterFaultDomain -Name "Rack03" -FaultDomainType Rack -Location "Contoso HQ, Room 4010, Aisle A, Rack 03" -CimSession $clusterName

   # Assign fault domains
   # Assign nodes to racks
   1..2 |ForEach-Object {Set-ClusterFaultDomain -Name "S2D$_" -Parent "Rack01" -CimSession $clusterName}
   3..4 |ForEach-Object {Set-ClusterFaultDomain -Name "S2D$_" -Parent "Rack02" -CimSession $clusterName}
   5..6 |ForEach-Object {Set-ClusterFaultDomain -Name "S2D$_" -Parent "Rack03" -CimSession $clusterName}
   ```

1. To display the newly configured fault domains, in the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   $clusterName="S2D-Cluster"
   Get-ClusterFaultDomain -CimSession $clusterName
   Get-ClusterFaultDomainxml -CimSession $clusterName
   ```

1. within the Windows Admin Center browser window, connect to the `s2d-cluster.corp.contoso.com` cluster.
1. Review the rack information for each node of the `s2d-cluster.corp.contoso.com` cluster.

   > **Note:** If necessary, select **Install** to install **RSAT-Clustering-PowerShell**, which is required by Windows Admin Center.

### Task 5: Enable Storage Spaces Direct on the failover cluster

1. To enable Storage Spaces Direct on the cluster, In the **WSLab-Management** VM console session, in the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   $clusterName="S2D-Cluster"
   Enable-ClusterS2D -CimSession $clusterName -Verbose
   ```

   > **Note:** Wait for the installation to complete before you proceed to the next task. This should take about 3 minutes.

1. Review the provisioning steps and note that the Storage Spaces Direct cluster setup has automatically set the default fault domain awareness on the clustered storage subsystem.

### Task 6: Review the fault domain configuration on the Storage Spaces Direct cluster

1. To list storage pool properties, In the **WSLab-Management** VM console session, in the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   $clusterName="S2D-Cluster"
   Get-StoragePool -CimSession $clusterName -FriendlyName S2D* | fl *
   ```

   > **Note:** Verify that **FaultDomainAwarenessDefault** is automatically set to **StorageRack**.

1. To list properties of storage tiers, in the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   Get-StorageTier -CimSession s2d-cluster | fl *
   ```

   > **Note:** Verify that the two tiers named **MirrorOnHDD** and **Capacity** both have **FaultDomainAwarenessDefault** set to **StorageRack**.

### Task 7: Create tiered volumes on a Storage Spaces Direct failover cluster

1. To create a volume referencing the **Capacity** tier, In the **WSLab-Management** VM console session, in the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   New-Volume -StoragePoolFriendlyName s2d* -FriendlyName WithTier -FileSystem CSVFS_ReFS -StorageTierFriendlyNames Capacity -StorageTierSizes 1TB -CimSession $clusterName
   ```

1. To create a volume not referencing any specific tier, in the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   New-Volume -StoragePoolFriendlyName s2d* -FriendlyName WithoutTier -FileSystem CSVFS_ReFS -Size 1TB -ResiliencySettingName Mirror -CimSession $clusterName
   ```

1. In the **WSLab-Management** VM console session, refresh the Windows Admin Center browser window and navigate to the pane displaying the **WithoutTier** volume properties.
1. On the **WithoutTier** volume pane, note that **Fault domain awareness** is set to **Rack**.

   > **Note:** The actual **FaultDomainAwareness** property is defined on the virtual disk level.

1. Within the Windows Admin Center browser window, navigate to the pane displaying the **WithTier** volume properties.
1. On the **WithTier** volume pane, note that **Fault domain awareness** is also set to **Rack**.

   > **Note:** The actual **FaultDomainAwareness** property is defined on the storage tier associated with the tiered disk.

### Task 8: Test resiliency of the Storage Spaces Direct cluster

1. To simulate a failure of an entire rack containing two cluster nodes, switch to the lab VM and in the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   Get-VM -Name "WSLab-S2D1","WSLab-S2D2" | Stop-VM -TurnOff
   ```

1. Switch to the console session connected to the **WSLab-Management** VM, and in the browser window displaying Windows Admin Center, navigate to the server inventory of the `s2d-cluster.corp.contoso.com` cluster.

1. Verify that the **S2D1** and **S2D2** nodes in **Rack01** are down, but that the cluster remains online.

   > **Note:** You might need to refresh the page displaying the Windows Admin Center interface to review the updated status of the cluster nodes.

1. Navigate to the volume inventory of the `s2d-cluster.corp.contoso.com` cluster, and then verify that all volumes are healthy.

1. To simulate a storage failure caused by removing one capacity disk from both the **S2D3** and **S2D4** cluster nodes, switch to the lab VM, and in the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   Get-VM -Name "WSLab-S2D3","WSLab-S2D4" | Get-VMHardDiskDrive | Where-Object controllerlocation -eq 1 | Remove-VMHardDiskDrive
   ```

1. Switch to the console session connected to the **WSLab-Management** VM, and in the browser window displaying Windows Admin Center, review the volume inventory of the `s2d-cluster.corp.contoso.com` cluster, and verify that all volumes are still healthy.

1. In the **WSLab-Management** VM console session, in the browser window displaying Windows Admin Center, review the summary of drives, and verify that **26** out of **72** drives are listed as **Critical**.

1. To simulate a failure of all disks attached to both **S2D3** and **S2D4** cluster nodes, switch to the lab VM, and in the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   Get-VM -Name "WSLab-S2D3","WSLab-S2D4" | Get-VMHardDiskDrive | Where-Object controllerlocation -ne 0 | Remove-VMHardDiskDrive
   ```

1. Switch to the console session connected to the **WSLab-Management** VM, and in the browser window displaying Windows Admin Center, review the summary of drives, and verify that **48** out of **72** drives are listed as **Critical**.

1. Navigate to the volume inventory of the `s2d-cluster.corp.contoso.com` cluster, and verify that all volumes are healthy.

    > **Note:** The storage pool with all volumes might transition to the offline state if you trigger subsequent faults too quickly. If this happens, consider repeating this exercise and pausing between the steps of this task.

   > **Note:** The cluster, pool, and virtual disks are all capable of surviving another node failure providing their respective resources reside in the surviving rack.

1. In the **WSLab-Management** VM console session, start **Failover Cluster Manager** and connect to the `s2d-cluster.corp.contoso.com` cluster.

1. In **Failover Cluster Manager**, identify the owner node of both the **With Tier** and **Without Tier** virtual disks. If the owner node is listed as **S2D3** or **S2D4**, move it to either **S2D5** or **S2D6**.

1. Identify the owner node of the **Cluster Pool 1** storage pool. If the owner node is listed as **S2D3** or **S2D4**, move it to either **S2D5** or **S2D6**.

1. To simulate a **S2D3** node failure, switch to the lab VM, and in the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   Stop-VM -Name "WSLab-S2D3" -TurnOff
   ```

1. Switch to the console session connected to the **WSLab-Management** VM, and then within the console session, in **Failover Cluster Manager**, verify that the **S2D1**, **S2D2**, and **S2D3** nodes are down but that the cluster remains online.

1. In **Failover Cluster Manager**, verify that all virtual disks are online.

   > **Note:** The disks might transition to the offline state if you trigger subsequent faults too quickly. 

1. In the **WSLab-Management** VM console session, switch to the browser window displaying Windows Admin Center and verify that all volumes of the `s2d-cluster.corp.contoso.com` cluster are healthy.

1. In the browser window displaying Windows Admin Center, navigate to the `s2d-cluster.corp.contoso.com` cluster's drive inventory, and verify that **48** out of **72** drives are listed as **Critical**.

1. Review the results and verify that the cluster and its virtual disks remain online with only **24** out of **72** physical disks.

> **Note:** The storage pool with all volumes might transition to the offline state if you trigger subsequent faults too quickly. If this happens, consider repeating this exercise and pausing between the steps of this task.

### Task 9: Restore failed disks and nodes on the Storage Spaces Direct cluster

1. To return all disks to the **S2D3** and **S2D4** cluster nodes, switch to the lab VM, and in the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   $vmNames="WSLab-S2D3","WSLab-S2D4"
   foreach ($vmName in $vmNames){
     $vhds = (Get-ChildItem -Path "$((get-vm $vmName).ConfigurationLocation)\Virtual Hard Disks" | Where-Object name -like HDD*).FullName
     foreach ($vhd in $vhds){
        Add-VMHardDiskDrive -VMName $VMName -Path $VHD
     }
   }
   ```

1. On the lab VM, in the **Administrator: Windows PowerShell ISE** window, run the following command to start the **S2D1**, **S2D2**, and **S2D3** cluster nodes:

   ```powershell
   Start-VM -Name "WSLab-S2D1","WSLab-S2D2","WSLab-S2D3"
   ```

1. Switch to the console session connected to the **WSLab-Management** VM, and in the **Administrator: Windows PowerShell ISE** window, run the following command to identify the repair and regeneration jobs in progress:

   ```powershell
   Get-StorageSubSystem -CimSession s2d-cluster -FriendlyName CL* | Get-StorageJob
   ```

1. In the **WSLab-Management** VM console session, switch to the browser window displaying Windows Admin Center, and then review the status of the drives of the `s2d-cluster.corp.contoso.com` cluster.
1. Within the Windows Admin Center browser window, navigate to the **Dashboard** page of the `s2d-cluster.corp.contoso.com` cluster, and then review its contents to verify that it reports a status of **Healthy** for all cluster components, and that there are no alerts listed.

   > **Note:** The storage subsystem should return to the **Healthy** status in about 5 minutes. So, if you are still getting messages indicating storage faults, you might need to wait and rerun the cmdlet.

1. To identify the health status of the Storage Spaces Direct cluster, In the **WSLab-Management** VM console session, switch to the **Administrator: Windows PowerShell ISE** window and run the following command:

   ```powershell
   Get-HealthFault -CimSession s2d-cluster
   ```

   > **Note:** Ignore faults regarding memory consumption on cluster nodes; these are expected.

### Task 10: Deprovision the lab resources

1. Switch to the lab VM.
1. On the lab VM, in the **Administrator: Windows PowerShell ISE** window, run the **F:\\WSLab-master\\Scripts\\Cleanup.ps1** script to remove all VMs provisioned in this lab.
1. In the **Administrator: Windows PowerShell ISE** window, close the tab displaying the **F:\\WSLab-master\\Scripts\\Cleanup.ps1** script.

## Results

After completing this lab, you will have managed and monitored resiliency of a Storage Spaces Direct cluster.
