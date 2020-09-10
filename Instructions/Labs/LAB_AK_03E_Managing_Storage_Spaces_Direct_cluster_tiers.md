---
lab:
    title: 'Lab E: Managing Storage Spaces Direct cluster tiers'
    type: 'Answer Key'
    module: 'Module 3: Planning for and implementing Azure Stack HCI Storage'
---
# Lab E answer key: Managing Storage Spaces Direct cluster tiers

## Exercise 1: Managing Storage Spaces Direct cluster tiers

### Task 1: Provision the lab environment VMs

1. On the lab VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to rename **LabConfig.ps1** and **Scenario.ps1**:

   ```powershell
   Set-Location -Path 'F:\WSLab-master\Scripts'
   Move-Item -Path '.\LabConfig.ps1' -Destination '.\LabConfig.m3l4.ps1' -Force -ErrorAction SilentlyContinue
   Move-Item -Path '.\Scenario.ps1' -Destination '.\Scenario.m3l4.ps1' -Force -ErrorAction SilentlyContinue
   ```

1. In the **Administrator: Windows PowerShell ISE** window, open a new tab in the **script** pane, paste the following command, and then save it as **F:\\WSLab-master\\Scripts\\LabConfig.ps1**:

   ```powershell
   $LabConfig=@{ DomainAdminName = 'LabAdmin'; AdminPassword = 'LS1setup!'; Prefix = 'WSLab-'; SwitchName = 'LabSwitch'; DCEdition = '4' ; Internet = $true ;AdditionalNetworksConfig = @(); VMs = @()}
   1..2 | % {$VMNames = "2T2node"; $LABConfig.VMs += @{ VMName = "$VMNames$_" ; Configuration = 'S2D' ; ParentVHD = 'Win2019Core_G2.vhdx'; SSDNumber = 0; SSDSize = 800GB ; HDDNumber = 4; HDDSize = 4TB ; MemoryStartupBytes = 1GB ; MemoryMinimumBytes = 512MB }}
   1..3 | % {$VMNames = "2T3node"; $LABConfig.VMs += @{ VMName = "$VMNames$_" ; Configuration = 'S2D' ; ParentVHD = 'Win2019Core_G2.vhdx'; SSDNumber = 0; SSDSize = 800GB ; HDDNumber = 4; HDDSize = 4TB ; MemoryStartupBytes = 1GB ; MemoryMinimumBytes = 512MB }}
   1..2 | % {$VMNames = "3T2node"; $LABConfig.VMs += @{ VMName = "$VMNames$_" ; Configuration = 'S2D' ; ParentVHD = 'Win2019Core_G2.vhdx'; SSDNumber = 4; SSDSize = 800GB ; HDDNumber = 4; HDDSize = 4TB ; MemoryStartupBytes = 1GB ; MemoryMinimumBytes = 512MB }}
   1..3 | % {$VMNames = "3T3node"; $LABConfig.VMs += @{ VMName = "$VMNames$_" ; Configuration = 'S2D' ; ParentVHD = 'Win2019Core_G2.vhdx'; SSDNumber = 4; SSDSize = 800GB ; HDDNumber = 4; HDDSize = 4TB ; MemoryStartupBytes = 1GB ; MemoryMinimumBytes = 512MB }}

   $LABConfig.VMs += @{
        VMName = "Management" ;
        Configuration = 'S2D' ;
        ParentVHD = 'Win2019_G2.vhdx';
        SSDNumber = 1;
        SSDSize = 50GB ;
        MemoryStartupBytes= 8GB;
        NestedVirt = $false;
        StaticMemory = $true;
        VMProcessorCount = 4
    }
   ```

1. On the lab VM, in the **Administrator: Windows PowerShell ISE** window, open and run the **F:\\WSLab-master\\Scripts\\3_Deploy.ps1** script to provision VMs for the Storage Spaces Direct environment.

   > **Note:** The script should complete in about 15 minutes. When prompted to **Press enter to continue**, select the **Enter** key.

1. After the script completes, in the **Administrator: Windows PowerShell ISE** window, from the **console** pane, run the following command to start the newly provisioned VMs that will host the Storage Spaces Direct environment:

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

### Task 2: Configure the management server

1. On the lab VM, in the **Server Manager** window, select **Tools**, and then from the drop-down list, select **Hyper-V Manager**.

1. On the lab VM, in the **Hyper-V Manager** console, in the list of virtual machines, right-click or access the context menu for the **WSLab-Management** entry, and select **Connect** to establish a console session to the **WSLab-Management** VM. When prompted to sign in, provide the **CORP\\LabAdmin** username and **LS1setup!** password.

1. In the console session to the **WSLab-Management** VM, start Windows PowerShell ISE as Administrator.

1. In the console session to the **WSLab-Management** VM, from the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to install Remote Server Administration Tools:

   ```powershell
   Install-WindowsFeature -Name RSAT-Clustering,RSAT-Clustering-Mgmt,RSAT-Clustering-PowerShell,RSAT-Hyper-V-Tools,RSAT-AD-PowerShell,RSAT-ADDS
   ```

   > **Note:** Proceed to the next step without waiting for the installation to complete.

1. In the console session to the **WSLab-Management** VM, start another instance of Windows PowerShell ISE as Administrator.

1. In the console session to the **WSLab-Management** VM, from the **script** pane of the newly started **Administrator: Windows PowerShell ISE** window, run the following command to download and install Windows Admin Center:

   ```powershell
   Invoke-WebRequest -UseBasicParsing -Uri https://aka.ms/WACDownload -OutFile "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v waclog.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > **Note:** Proceed to the next step without waiting for the installation to complete.

1. In the console session to the **WSLab-Management** VM, from the **script** pane of the newly started **Administrator: Windows PowerShell ISE** window, run the following command to download and install the Microsoft Edge (Chromium) browser:

   ```powershell
   $progressPreference='SilentlyContinue'
   Invoke-WebRequest -Uri "https://go.microsoft.com/fwlink/?linkid=2069324&language=en-us&Consent=1" -UseBasicParsing -OutFile "$env:USERPROFILE\Downloads\MicrosoftEdgeSetup.exe"
   Start-Process -FilePath "$env:USERPROFILE\Downloads\MicrosoftEdgeSetup.exe" -Wait
   ```

   > **Note:** Proceed to the next step without waiting for the installation to complete.

1. To configure Kerberos constrained delegation to minimize prompts for credentials when using Windows Admin Center, switch to the first **Administrator: Windows PowerShell ISE** window where you initiated installation of the Remote Server Administration Tools, wait for the installation to complete, and then from the **script** pane, run the following command:

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

### Task 3: Deploy Storage Spaces Direct clusters

1. In the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to provision Storage Spaces Direct clusters:

   ```powershell
   $clusters=@()
   $clusters+=@{Nodes=1..2 | % {"2T2node$_"} ; Name="2T2nodeClus" ; IP="10.0.0.112" }
   $clusters+=@{Nodes=1..3 | % {"2T3node$_"} ; Name="2T3nodeClus" ; IP="10.0.0.113" }
   $clusters+=@{Nodes=1..2 | % {"3T2node$_"} ; Name="3T2nodeClus" ; IP="10.0.0.115" }
   $clusters+=@{Nodes=1..3 | % {"3T3node$_"} ; Name="3T3nodeClus" ; IP="10.0.0.116" }

   # Install features on servers
   Invoke-Command -computername $clusters.nodes -ScriptBlock {
     Install-WindowsFeature -Name "Failover-Clustering","Hyper-V-PowerShell","RSAT-Clustering-PowerShell" #RSAT is needed for Windows Admin Center
   }

   # Restart servers since failover clustering in Windows Server 2019 requires reboot
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
     $WitnessName = $clusterName+"Witness"
     Invoke-Command -ComputerName DC -ScriptBlock {New-Item -Path c:\Shares -Name $using:WitnessName -ItemType Directory}
     $accounts = @()
     $accounts += "CORP\$($clusterName)$"
     $accounts += "CORP\Domain Admins"
     New-SmbShare -Name $WitnessName -Path "c:\Shares\$WitnessName" -FullAccess $accounts -CimSession DC
     # Set NTFS permissions
     Invoke-Command -ComputerName DC -ScriptBlock {(Get-SmbShare $using:WitnessName).PresetPathAcl | Set-Acl}
     # Set Quorum
     Set-ClusterQuorum -Cluster $clusterName -FileShareWitness "\\DC\$WitnessName"
   }

   # Enable Storage Spaces Direct and configure mediatype to simulate 3 tier system with SCM (all 800GB disks are SCM, all 4T are SSDs)
   foreach ($cluster in $clusters.Name){
     Enable-ClusterS2D -CimSession $cluster -Verbose -Confirm:0
     if ($cluster -like "3T*"){
       invoke-command -computername $cluster -scriptblock {
         Get-PhysicalDisk | Where-Object size -eq 800GB | Set-PhysicalDisk -MediaType SCM
         Get-PhysicalDisk | Where-Object size -eq 4TB | Set-PhysicalDisk -MediaType SSD
       }
     }
   }
   ```

   > **Note:** Wait for the script to complete before you proceed to the next task. The script should take about 10 minutes to complete.

1. To generate a text file, you will use to import the list of Storage Spaces Direct clusters into Windows Admin Center, in the console session to the **WSLab-Management** VM, from the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command:

   ```powershell
   (Get-Cluster -Domain $env:userdomain | Where-Object S2DEnabled -eq 1).Name | Out-File c:\s2dclusters.txt
   ```

1. Switch to the Microsoft Edge browser window, select **Get started**, accept the default tab page settings, select the **Continue without Signing-in** link, use the Microsoft Edge browser to navigate to `https://management.corp.contoso.com`, and then when prompted to authenticate, sign in as **CORP\\LabAdmin** with the password **LS1setup!**. 
1. In the console session to the **WSLab-Management** VM, in the browser window displaying the Windows Admin Center interface, on the **All connections** page, select **+ Add**.
1. On the **Add or create resources** panel, on the **Server clusters** tile, select **Add**, select the **Import clusters** tab, select **Select a file**, in the **Open** dialog box, locate the **c:\s2dclusters.txt** file, select **Open**, and then select **Add**.

### Task 4: Configure Storage Spaces Direct cluster tiers

1. In the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to identify the tiers on a two-node cluster with HDDs only:

   ```powershell
   Get-StorageTier -CimSession 2T2NodeClus |
     ft FriendlyName,MediaType,ResiliencySettingName,NumberOfDataCopies,PhysicalDiskRedundancy,FaultDomainAwareness,ColumnIsolation,NumberOfGroups,NumberOfColumns
   ```

   > **Note:** On a two-node Storage Spaces Direct cluster with HDDs only, there are two tiers. One is **Capacity**, created to provide compatibility with the naming convention in Windows Server 2016, and the other is **MirrorOnHDD**, which follows the naming convention in Windows Server 2019. The value of **NumberOfDataCopies** represents the **2way mirror** configuration and **PhysicalDiskRedundancy** reflects the ability to tolerate a single fault. The value of **FaultDomainAwareness** indicates that the two copies are distributed across instances of **StorageScaleUnit**. The number of columns is automatically calculated, depending on the number of nodes and disks in each node, and is assigned during the creation of the virtual disks. The value of **NumberOfGroups** indicates the parity setting, which in this case is set to **1**.

1. In the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to identify all of the tiers on all of the Storage Spaces Direct clusters in the lab environment:

   ```powershell
   $clusters=(Get-Cluster -Domain $env:userdomain | Where-Object S2DEnabled -eq 1).Name
   Get-StorageTier -CimSession $clusters |
     Sort-Object PSComputerName |
     ft PSComputerName,FriendlyName,MediaType,ResiliencySettingName,NumberOfDataCopies,PhysicalDiskRedundancy,FaultDomainAwareness,ColumnIsolation,NumberOfGroups,NumberOfColumns
   ```

   > **Note:** The tiers are generated automatically when you invoke the **Enable-ClusterS2D** PowerShell cmdlet. Tiers reflect the media type present in cluster nodes.

1. In the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to identify the Windows Server 2019-specific mirror tiers on all of the Storage Spaces Direct clusters in the lab environment (tiers that reference **MirrorOnHDD**, **MirrorOnSSD**, and **MirrorOnSCM**, where *SCM* designates Storage Class Memory):

   ```powershell
   $clusters=(Get-Cluster -Domain $env:userdomain | Where-Object S2DEnabled -eq 1).Name
   Get-StorageTier -CimSession $clusters |
     Where-Object friendlyname -like mirror* |
     Sort-Object PSComputerName |
     ft PSComputerName,FriendlyName,MediaType,ResiliencySettingName,NumberOfDataCopies,PhysicalDiskRedundancy,FaultDomainAwareness,ColumnIsolation,NumberOfGroups,NumberOfColumns
   ```

   > **Note:** The values of **NumberOfCopies** and **PhysicalDiskRedundancy** is **2** on two-node clusters and **3** for clusters with three or more nodes.

1. In the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to identify the Windows Server 2019-specific parity tiers on all of the Storage Spaces Direct clusters in the lab environment (tiers that reference **MirrorOnHDD**, **MirrorOnSSD**, and **MirrorOnSCM**, where *SCM* designates Storage Class Memory):

   ```powershell
   $clusters=(Get-Cluster -Domain $env:userdomain | Where-Object S2DEnabled -eq 1).Name
   Get-StorageTier -CimSession $clusters |
     Where-Object friendlyname -like parity* |
     Sort-Object PSComputerName |
     ft PSComputerName,FriendlyName,MediaType,ResiliencySettingName,NumberOfDataCopies,PhysicalDiskRedundancy,FaultDomainAwareness,ColumnIsolation,NumberOfGroups,NumberOfColumns
   ```

   > **Note:** The script does not return any results, because by default, parity tiers are created only on Storage Spaces Direct clusters with four or more nodes. With Windows Server 2019-based Storage Spaces Direct clusters, you have the option to create parity tiers by implementing nested resiliency on two-node clusters.

1. In the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to create nested resiliency tiers:

   ```powershell
   #Select clusters to fix tiers
   $clusterNames=(Get-Cluster -Domain $env:userdomain | Where-Object S2DEnabled -eq 1 | Out-GridView -OutputMode Multiple -Title "Select Clusters to Check on tiers").Name

   foreach ($clusterName in $clusterNames){
      $storageTiers=Get-StorageTier -CimSession $clusterName
      $numberOfNodes=(Get-ClusterNode -Cluster $clusterName).Count
      $MediaTypes=(Get-PhysicalDisk -CimSession $clusterName |where mediatype -ne Unspecified | Where-Object usage -ne Journal).MediaType | Select-Object -Unique
      $clusterFunctionalLevel=(Get-Cluster -Name $clusterName).ClusterFunctionalLevel

      foreach ($mediaType in $mediaTypes){
         if ($numberOfNodes -eq 2) {
            # Create Mirror Tiers
            if (-not ($storageTiers | Where-Object FriendlyName -eq "MirrorOn$mediaType")){
               New-StorageTier -CimSession $clusterName -StoragePoolFriendlyName "S2D on $clusterName" -FriendlyName "MirrorOn$mediaType" -MediaType $mediaType -ResiliencySettingName Mirror -NumberOfDataCopies 2
            }
            if ($clusterFunctionalLevel -ge 10){
               # Create NestedMirror Tiers
               if (-not ($storageTiers | Where-Object FriendlyName -eq "NestedMirrorOn$mediaType")){
                  New-StorageTier -CimSession $clusterName -StoragePoolFriendlyName "S2D on $clusterName" -FriendlyName "NestedMirrorOn$mediaType" -MediaType $mediaType -ResiliencySettingName Mirror -NumberOfDataCopies 4
               }
               #Create NestedParity Tiers
               if (-not ($storageTiers | Where-Object FriendlyName -eq "NestedParityOn$mediaType")){
                  New-StorageTier -CimSession $clusterName -StoragePoolFriendlyName "S2D on $clusterName" -FriendlyName "NestedParityOn$mediaType" -MediaType $mediaType -ResiliencySettingName Parity -NumberOfDataCopies 2 -PhysicalDiskRedundancy 1 -NumberOfGroups 1 -ColumnIsolation PhysicalDisk
               }
            }
         }
      }
   }
   ```

   > **Note:** When prompted, in the **Select Clusters to Check on tiers** window, select the **Ctrl** key, select both the **2T2nodeClus** and **3T2nodeClus** entry, and then select **OK**.

1. In the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to identify the Windows Server 2019-specific nested tiers on two-node Storage Spaces Direct clusters in the lab environment:

   ```powershell
   $clusters=(Get-Cluster -Domain $env:userdomain | Where-Object S2DEnabled -eq 1).Name
   Get-StorageTier -CimSession $clusters |
     Where-Object friendlyname -like nested* |
     Sort-Object PSComputerName |
     ft PSComputerName,FriendlyName,MediaType,ResiliencySettingName,NumberOfDataCopies,PhysicalDiskRedundancy,FaultDomainAwareness,ColumnIsolation,NumberOfGroups,NumberOfColumns
   ```

   > **Note:** By default, parity tiers are created only on Storage Spaces Direct clusters with four or more nodes. With Windows Server 2019-based Storage Spaces Direct clusters, you have the option to create parity tiers by implementing nested resiliency on two-node clusters.

### Task 5: Provision nested tier volumes

1. In the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to provision a volume on the Storage Spaces Direct cluster **2T2nodeClus** based on the **NestedMirrorOnHDD** nested resiliency tier:

   ```powershell
   $clusterName = '2T2nodeClus'
   New-Volume -StoragePoolFriendlyName s2d* -FriendlyName NestedMirroronHDDVolume -FileSystem CSVFS_ReFS -StorageTierFriendlyNames NestedMirrorOnHDD -StorageTierSizes 128GB -CimSession $clusterName
   ```

   > **Note:** Wait until the volume is provisioned. This should take less than one minute.

1. In the console session to the **WSLab-Management** VM, in the browser window displaying the Windows Admin Center interface, on the **All connections** page, select **2T2nodeClus**.
1. On the **Specify your credentials** panel, select the **Use another account for this connection** option, and then in the **Username** text box, enter **CORP\\LabAdmin**, and in the **Password** text box, enter **LS1setup!**, select the **Use these credentials for all connections**, and then select **Continue**.
1. On the **2T2nodeClus** page, in the **Storage** section, select **Volumes**, and then on the **Volumes** pane, select the **Inventory** tab.
1. In the list of volumes, note that the **NestedMirroronHDDVolume** has resiliency set to **Nested two-way mirror**.
1. In the browser window displaying the Windows Admin Center interface, navigate back to the **2T2nodeClus** page.
1. In the **Storage** section, select **Drives**, and then on the **Drives** pane, select the **Inventory** tab.
1. In the list of drives, review the list of drive types.
1. In the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to provision a volume on the Storage Spaces Direct cluster **3T2nodeClus** based on the **NestedMirrorOnSSD** nested resiliency tier:

   ```powershell
   $clusterName = '3T2nodeClus'
   New-Volume -StoragePoolFriendlyName s2d* -FriendlyName NestedMirroronSSDVolume -FileSystem CSVFS_ReFS -StorageTierFriendlyNames NestedMirrorOnSSD -StorageTierSizes 128GB -CimSession $clusterName
   ```

1. In the console session to the **WSLab-Management** VM, in the browser window displaying the Windows Admin Center interface, navigate back to the **All connections** page, and then select **3T2nodeClus**.
1. On the **3T2nodeClus** page, in the **Storage** section, select **Volumes**, and then on the **Volumes** pane, select the **Inventory** tab.
1. In the list of volumes, note that the **NestedMirroronSSDVolume** has the resiliency set to **Nested two-way mirror**.
1. In the browser window displaying the Windows Admin Center interface, navigate back to the **3T2nodeClus** page, in the **Storage** section, select **Drives**, and then on the **Drives** pane, select the **Inventory** tab.
1. In the list of drives, review the list of drive types.

### Task 6: Deprovision the lab resources

1. Switch to the lab VM, and in the **Administrator: Windows PowerShell ISE** window, open and run the **F:\\WSLab-master\\Scripts\\Cleanup.ps1** script to remove all VMs provisioned in this lab. When prompted, on the **console** pane, enter **Y**, and then select **Enter**.
1. When the script completes, select any key.
1. In the **Administrator: Windows PowerShell ISE** window, close the tab displaying the **F:\\WSLab-master\\Scripts\\Cleanup.ps1** script.
