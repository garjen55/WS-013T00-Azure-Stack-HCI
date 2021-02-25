---
lab:
    title: 'Lab C: Managing and monitoring resiliency of a Storage Spaces Direct cluster'
    type: 'Answer Key'
    module: 'Module 3: Planning for and implementing Azure Stack HCI Storage'
---
# Lab C answer key: Managing and monitoring resiliency of a Storage Spaces Direct cluster

## Exercise 1: Managing and monitoring resiliency of a Storage Spaces Direct cluster

### Task 1: Provision the lab environment VMs

1. On the lab VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to rename **LabConfig.ps1** and **Scenario.ps1**:

   ```powershell
   Set-Location -Path 'F:\WSLab-master\Scripts'
   Move-Item -Path '.\LabConfig.ps1' -Destination '.\LabConfig.m3l3.ps1' -Force -ErrorAction SilentlyContinue
   Move-Item -Path '.\Scenario.ps1' -Destination '.\Scenario.m3l3.ps1' -Force -ErrorAction SilentlyContinue
   ```

1. In the **Administrator: Windows PowerShell ISE** window, open a new tab on the **script** pane, paste the following command, and save it as **F:\\WSLab-master\\Scripts\\LabConfig.ps1**:

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

1. On the lab VM, in the **Administrator: Windows PowerShell ISE** window, open and run the **F:\\WSLab-master\\Scripts\\3_Deploy.ps1** script to provision VMs for the Storage Spaces Direct environment.

   > **Note:** Select **None** at the Telemetry prompt. The script should complete in about 10 minutes. When prompted **Press enter to continue**, select the **Enter** key.

1. When the script completes, in the **Administrator: Windows PowerShell ISE** window, open a new tab, and run the following command to start the newly provisioned VMs that will host the Storage Spaces Direct environment:

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

1. On the lab VM, in the **Server Manager** window, select **Tools**, and then in the drop-down list, select **Hyper-V Manager**.

1. On the lab VM, in the **Hyper-V Manager** console, in the list of virtual machines, right-click or access the context menu for **WSLab-Management** entry, and then select **Connect** to establish a console session to the **WSLab-Management** VM. When prompted to sign in, provide the **CORP\\LabAdmin** username and **LS1setup!** password.

1. In the console session to the **WSLab-Management** VM, start Windows PowerShell ISE as Administrator.

1. In the console session to the **WSLab-Management** VM, from the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to install Remote Server Administration Tools:

   ```powershell
   Install-WindowsFeature -Name RSAT-Clustering,RSAT-Clustering-Mgmt,RSAT-Clustering-PowerShell,RSAT-Hyper-V-Tools,RSAT-AD-PowerShell,RSAT-ADDS
   ```

   > **Note:** Proceed to the next step without waiting for the installation to complete.

1. In the console session to the **WSLab-Management** VM, start another instance of Windows PowerShell ISE as Administrator.

1. In the console session to the **WSLab-Management** VM, from the **script** pane of the newly started **Administrator: Windows PowerShell ISE** window, run the following command to download and install the Microsoft Edge (Chromium) browser:

   ```powershell
   $progressPreference='SilentlyContinue'
   Invoke-WebRequest -Uri "https://go.microsoft.com/fwlink/?linkid=2069324&language=en-us&Consent=1" -UseBasicParsing -OutFile "$env:USERPROFILE\Downloads\MicrosoftEdgeSetup.exe"
   Start-Process -FilePath "$env:USERPROFILE\Downloads\MicrosoftEdgeSetup.exe" -Wait
   ```

   > **Note:** Proceed to the next step without waiting for the installation to complete.

1. In the console session to the **WSLab-Management** VM, start another instance of Windows PowerShell ISE as Administrator.

1. In the console session to the **WSLab-Management** VM, from the **script** pane of the newly started **Administrator: Windows PowerShell ISE** window, run the following command to download and install Windows Admin Center:

   ```powershell
   Invoke-WebRequest -UseBasicParsing -Uri https://aka.ms/WACDownload -OutFile "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v waclog.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > **Note:** Proceed to the next step without waiting for the installation to complete.

1. Switch to the first **Administrator: Windows PowerShell ISE** window where you initiated installation of the Remote Server Administration Tools and wait for the installation to complete.

1. To configure Kerberos constrained delegation to minimize prompts for credentials when using Windows Admin Center, from the **script** pane, run the following command:

   ```powershell
   $gateway = "Management"
   $nodes = Get-ADComputer -Filter * -SearchBase "ou=workshop,DC=corp,dc=contoso,DC=com"
   $gatewayObject = Get-ADComputer -Identity $gateway
   foreach ($node in $nodes){
    Set-ADComputer -Identity $node -PrincipalsAllowedToDelegateToAccount $gatewayObject
   }
   ```

   > **Note:** Before you proceed to the next step, verify that the installation of Microsoft Edge and Windows Admin Center completed.

1. Close the other two instances of the **Administrator: Windows PowerShell ISE** window you opened earlier in this task without saving the scripts you ran from each.

1. Switch to the Microsoft Edge browser window, select **Get started**, accept the default tab page settings, and select the **Continue without Signing-in** link, use the Microsoft Edge browser to navigate to `https://management.corp.contoso.com`, and then when prompted to authenticate, sign in as **CORP\\LabAdmin** with the password **LS1setup!**.

### Task 3: Create and configure a failover cluster

1. To provision a failover cluster, in the console session to the **WSLab-Management** VM, from the **Administrator: Windows PowerShell ISE** window, run the following command:

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

   > **Note:** Wait until the script completes before you proceed to the next task. This might take about 10 minutes.

### Task 4: Configure fault domains on the failover cluster

1. To configure fault domains on the Storage Spaces Direct cluster, in the console session to the **WSLab-Management** VM, from the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   $clusterName = "S2D-Cluster"

   # Create Fault domains with PowerShell
   New-ClusterFaultDomain -Name "Rack01" -FaultDomainType Rack -Location "Contoso HQ, Room 4010, Aisle A, Rack 01" -CimSession $clusterName
   New-ClusterFaultDomain -Name "Rack02" -FaultDomainType Rack -Location "Contoso HQ, Room 4010, Aisle A, Rack 02" -CimSession $clusterName
   New-ClusterFaultDomain -Name "Rack03" -FaultDomainType Rack -Location "Contoso HQ, Room 4010, Aisle A, Rack 03" -CimSession $clusterName

   # Assign fault domains
   # Assign nodes to racks
   1..2 |ForEach-Object {Set-ClusterFaultDomain -Name "S2D$_" -Parent "Rack01" -CimSession $clusterName}
   3..4 |ForEach-Object {Set-ClusterFaultDomain -Name "S2D$_" -Parent "Rack02" -CimSession $clusterName}
   5..6 |ForEach-Object {Set-ClusterFaultDomain -Name "S2D$_" -Parent "Rack03" -CimSession $clusterName}
   ```

1. To display the newly configured fault domains, in the console session to the **WSLab-Management** VM, from the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   $clusterName = "S2D-Cluster"
   Get-ClusterFaultDomain -CimSession $clusterName
   Get-ClusterFaultDomainxml -CimSession $clusterName
   ```

1. In the console session to the **WSLab-Management** VM, switch to the browser window displaying the Windows Admin Center interface, navigate to the **All connections** page, and then select **+ Add**.

1. On the **Add or create resources** panel, on the **Server clusters** tile, select **Add**.

1. In the **Cluster name** text box, enter `s2d-cluster.corp.contoso.com`. If prompted, select the **Use another account for this connection** option.

1. In the **Username** text box, enter **CORP\LabAdmin**, in the **Password** text box, enter **LS1setup!**, select **Connect with account**, and then select **Add**.

1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, in the **Compute** section, select **Nodes**, and on the **Nodes** pane, review the rack information for each node.

   > **Note:** If necessary, select **Install** to install **RSAT-Clustering-PowerShell**, which is required by Windows Admin Center.

### Task 5: Enable Storage Spaces Direct on the failover cluster

1. To enable Storage Spaces Direct on the cluster, in the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command, selecting **Yes to All** when prompted for confirmation:

   ```powershell
   $clusterName = "S2D-Cluster"
   Enable-ClusterS2D -CimSession $clusterName -Verbose
   ```

   > **Note:** Wait for the installation to complete before you proceed to the next task. This should take about three minutes.

1. Review the provisioning steps and note that the Storage Spaces Direct cluster setup has automatically set the default fault domain awareness on the clustered storage subsystem.

### Task 6: Review fault domain configuration on the Storage Spaces Direct cluster

1. In the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to list storage pool properties:

   ```powershell
   $clusterName = "S2D-Cluster"
   Get-StoragePool -CimSession $clusterName -FriendlyName S2D* | fl *
   ```

   > **Note:** Verify that **FaultDomainAwarenessDefault** is automatically set to **StorageRack**.

1. In the console session to the **WSLab-Management** VM, from the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to list properties of storage tiers:

   ```powershell
   Get-StorageTier -CimSession s2d-cluster | fl *
   ```

   > **Note:** Verify that the two tiers named **MirrorOnHDD** and **Capacity** have **FaultDomainAwarenessDefault** set to **StorageRack**.

### Task 7: Create tiered volumes on a Storage Spaces Direct failover cluster

1. In the console session to the **WSLab-Management** VM, from the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to create a volume referencing the **Capacity** tier:

   ```powershell
   New-Volume -StoragePoolFriendlyName s2d* -FriendlyName WithTier -FileSystem CSVFS_ReFS -StorageTierFriendlyNames Capacity -StorageTierSizes 1TB -CimSession $clusterName
   ```

1. In the console session to the **WSLab-Management** VM, from the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to create a volume not referencing any specific tier:

   ```powershell
   New-Volume -StoragePoolFriendlyName s2d* -FriendlyName WithoutTier -FileSystem CSVFS_ReFS -Size 1TB -ResiliencySettingName Mirror -CimSession $clusterName
   ```

1. In the console session to the **WSLab-Management** VM, refresh the browser window displaying the Windows Admin Center interface.

1. On the `s2d-cluster.corp.contoso.com` page, in the **Storage** section, select **Volumes**, and then select **Inventory**.

1. On the **Volumes** pane, select the **WithoutTier** volume entry.

1. On the **Volumes > Volume WithoutTier** pane, note that **Fault domain awareness** is set to **Rack**.

   > **Note:** The actual **FaultDomainAwareness** property is defined on the virtual disk level.

1. Navigate back to the **Volumes** pane, and on the **Volumes** pane, select the **WithTier** volume entry.

1. On the **Volumes > Volume WithTier** pane, note that **Fault domain awareness** is also set to **Rack**.

   > **Note:** The actual **FaultDomainAwareness** property is defined on the storage tier associated with the tiered disk.

### Task 8: Test resiliency of the Storage Spaces Direct cluster

1. Switch to the lab VM.

1. In the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to simulate a failure of an entire rack containing two cluster nodes:

   ```powershell
   Get-VM -Name "WSLab-S2D1","WSLab-S2D2" | Stop-VM -TurnOff
   ```

1. Switch to the console session connected to the **WSLab-Management** VM.

1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, in the **Compute** section, select **Servers**.

1. On the **Servers** pane, select the **Inventory** tab, and verify that the **S2D1** and **S2D2** nodes in **Rack01** are down but the cluster remains online.

   > **Note:** You might need to refresh the page displaying the Windows Admin Center interface to observe the updated status of cluster nodes.

1. In the console session to the **WSLab-Management** VM, in the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, in the **Storage** section, select **Volumes**, and on the **Volumes** pane, verify that all volumes are healthy.

1. Switch to the lab VM, and in the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to simulate a storage failure caused by removing one capacity disk from both **S2D3** and **S2D4** cluster nodes:

   ```powershell
   Get-VM -Name "WSLab-S2D3","WSLab-S2D4" | Get-VMHardDiskDrive | Where-Object controllerlocation -eq 1 | Remove-VMHardDiskDrive
   ```

1. Switch to the console session connected to the **WSLab-Management** VM, and in the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, on the **Volumes** pane, verify that all volumes are still healthy.

1. In the console session to the **WSLab-Management** VM, in the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, in the **Storage** section, select **Drives**, and on the **Drives** pane, on the **Summary** tab, verify that **26** out of **72** drives are listed as **Critical**.

1. Switch to the lab VM, and on the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to simulate a failure of all disks attached to both **S2D3** and **S2D4** cluster nodes:

   ```powershell
   Get-VM -Name "WSLab-S2D3","WSLab-S2D4" | Get-VMHardDiskDrive | Where-Object controllerlocation -ne 0 | Remove-VMHardDiskDrive
   ```

1. Switch to the console session connected to the **WSLab-Management** VM.

1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, in the **Storage** section, on the **Drives** pane, on the **Summary** tab, verify that **48** out of **72** drives are listed as **Critical**.

1. In the console session to the **WSLab-Management** VM, in the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, in the **Storage** section, select **Volumes**, and then on the **Volumes** pane, verify that all volumes are healthy.

    > **Note:** The storage pool with all volumes might transition to the offline state if you trigger subsequent faults too quickly. If this happens, consider repeating this exercise and pausing between the steps of this task.

   > **Note:** The cluster, pool, and virtual disks are all capable of surviving another node failure if their respective resources reside in the surviving rack.

1. In the console session to the **WSLab-Management** VM, switch to the Server Manager window, select **Tools**, and then in the **Tools** drop-down menu, select **Failover Cluster Manager**.

1. In the **Failover Cluster Manager** window, right-click or access the context menu for the **Failover Cluster Manager** node, and then in the context menu, select **Connect to cluster**.

1. In the **Select Cluster** dialog box, in the **Cluster name** text box, enter `s2d-cluster.corp.contoso.com`, and then select **OK**.

1. In the console session to the **WSLab-Management** VM, in the **Failover Cluster Manager** window, navigate to the **Disks** subnode of the **Storage** node, and then identify the owner node of the **With Tier** and **Without Tier** virtual disks.
    
    1. If the owner node is listed as **S2D3** or **S2D4**, right-click or access the context menu for the virtual disk, and then in the context menu, select **Move**, followed by **Select Node**. In the **Move Cluster Shared Volume** dialog box, select **S2D5** or **S2D6**, and then select **OK**.
    
1. In the console session to the **WSLab-Management** VM, in the **Failover Cluster Manager** window, in the **Storage** node, select the **Pools** subnode, and identify the owner node of the **Cluster Pool 1** storage pool.
    
    1. If the owner node is listed as **S2D3** or **S2D4**, right-click or access the context menu for **Cluster Pool 1**, and then in the context menu, select **Move**, followed by **Select Node**. In the **Move Cluster Shared Volume** dialog box, select **S2D5** or **S2D6**, and then select **OK**.
    
1. Switch to the lab VM, and in the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to simulate a **S2D3** node failure:

   ```powershell
   Stop-VM -Name "WSLab-S2D3" -TurnOff
   ```

1. Switch to the console session connected to the **WSLab-Management** VM.

1. In the console session to the **WSLab-Management** VM, in the **Failover Cluster Manager** window, select **Nodes**, and then verify that **S2D1**, **S2D2**, and **S2D3** nodes are down but the cluster remains online.

1. In the console session to the **WSLab-Management** VM, in the **Failover Cluster Manager** window, in the **Storage** node, select the **Disks** subnode, and verify that all virtual disks are online.

   > **Note:** The disks might transition to the offline state if you trigger subsequent faults too quickly.

1. In the console session to the **WSLab-Management** VM, switch to the browser window displaying the Windows Admin Center interface.

1. On the `s2d-cluster.corp.contoso.com` page, in the **Storage** section, on the **Volumes** pane, verify that all volumes are healthy.

1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, in the **Storage** section, select **Drives**, and on the **Drives** pane, on the **Summary** tab, verify that **48** out of **72** drives are listed as **Critical**.

1. Review the results and verify that the cluster and its virtual disks remain online with only **24** out of **72** physical disks.

    > **Note:** The storage pool with all volumes might transition to the offline state if you trigger subsequent faults too quickly. If this happens, consider repeating this exercise and pausing between the steps of this task.

### Task 9: Restore failed disks and nodes on the Storage Spaces Direct cluster

1. Switch to the lab VM, and from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to return all disks to the **S2D3** and **S2D4** cluster nodes:

   ```powershell
   $vmNames = "WSLab-S2D3","WSLab-S2D4"
   foreach ($vmName in $vmNames){
     $vhds = (Get-ChildItem -Path "$((get-vm $vmName).ConfigurationLocation)\Virtual Hard Disks" | Where-Object name -like HDD*).FullName
     foreach ($vhd in $vhds){
        Add-VMHardDiskDrive -VMName $VMName -Path $VHD
     }
   }
   ```

1. On the lab VM, from the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to start the **S2D1**, **S2D2**, and **S2D3** cluster nodes:

   ```powershell
   Start-VM -Name "WSLab-S2D1","WSLab-S2D2","WSLab-S2D3"
   ```

1. Switch to the console session connected to the **WSLab-Management** VM, and then in the console session to the **WSLab-Management** VM, from the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to identify the repair and regeneration jobs in progress:

   ```powershell
   Get-StorageSubSystem -CimSession s2d-cluster -FriendlyName CL* | Get-StorageJob
   ```

1. In the console session to the **WSLab-Management** VM, switch to the browser window displaying the Windows Admin Center interface.

1. On the `s2d-cluster.corp.contoso.com` page, in the **Storage** section, on the **Drives** pane, select the **Inventory** tab, and then review the status of the drives.

1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, select **Dashboard** and review its contents, verifying that it reports a status of **Healthy** for all cluster components with no alerts listed.

   > **Note:** The storage subsystem should return to the healthy status in about five minutes, so you might need to wait and rerun the cmdlet if you are still observing messages indicating storage faults.

1. In the console session connected to the **WSLab-Management** VM, switch to the **Administrator: Windows PowerShell ISE** window and, from the **console** pane, run the following command to identify the health status of the Storage Spaces Direct cluster:

   ```powershell
   Get-HealthFault -CimSession s2d-cluster
   ```

   > **Note:** Ignore faults regarding memory consumption on cluster nodes; these are expected.

### Task 10: Deprovision the lab resources

1. Switch to the lab VM, and in the **Administrator: Windows PowerShell ISE** window, open and run the **F:\\WSLab-master\\Scripts\\Cleanup.ps1** script to remove all VMs provisioned in this lab. When prompted, on the **console** pane, enter **Y**, and then select **Enter**.
1. When the script completes, select any key.
1. In the **Administrator: Windows PowerShell ISE** window, close the tab displaying the **F:\\WSLab-master\\Scripts\\Cleanup.ps1** script.
