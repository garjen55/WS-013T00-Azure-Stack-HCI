---
lab:
    title: 'Lab A: Deploying Software-Defined Networking'
    module: 'Module 4: Planning for and Implementing Azure Stack HCI Networking'
---

# Lab A: Deploying Software-Defined Networking

## Scenario

To address the requirements for deploying an isolated VDI farm for users in the Contoso Securities Research department, which is supposed to replace an aging Windows Server 2012 R2–based RDS deployment, you'll implement Software-Defined Networking (SDN) on hyperconverged infrastructure. As the first step in this process, you need to provision the SDN infrastructure by using the scripts available online.

## Objectives

After completing this lab, you'll be able to deploy SDN by using PowerShell.

## Estimated time: 120 minutes

## Lab setup

To connect to the lab VM, follow the steps the lab hosting provider provides you.

## Exercise 1: Deploying Software-Defined Networking by using PowerShell

### Scenario

To prepare for the rest of this lab, you need to provision the Software-Defined Networking (SDN) infrastructure by leveraging the scripts available at [microsoft/WSLab](https://aka.ms/sdnexpress-with-windows-admin-center).

The main tasks for this exercise are as follows:

1. Deploy the VMs that will serve as the SDN infrastructure Hyper-V hosts.
1. Deploy the SDN infrastructure VMs.

### Task 1: Deploy the VMs that will serve as the SDN infrastructure Hyper-V hosts

1. On the lab VM, start Windows PowerShell ISE as Administrator and run the following command to remove the **Zone.Identifier** alternate data stream, which has a value of **3** indicating that it was downloaded from the internet:

   ```powershell
   Get-ChildItem -Path F:\WSLab-master\ -File -Recurse | Unblock-File
   ```

1. On the lab VM, from the Administrator: Windows PowerShell ISE window, run the following command to set the current directory:

   ```powershell
   Set-Location -Path F:\WSLab-master\Scripts
   ```
1. On the lab VM, from the Administrator: Windows PowerShell ISE window, run the following command to rename **LabConfig.ps1** and **Scenario.ps1**:

   ```powershell
   Move-Item -Path '.\LabConfig.ps1' -Destination '.\LabConfig.m4l0.ps1' -Force -ErrorAction SilentlyContinue
   Move-Item -Path '.\Scenario.ps1' -Destination '.\Scenario.m4l0.ps1' -Force -ErrorAction SilentlyContinue
   ```

1. On the lab VM, in the Administrator: Windows PowerShell ISE window, open a new tab in the **script** pane, paste the following content and save it as **F:\\WSLab-master\\Scripts\\LabConfig.ps1**:

   ```powershell
   $LabConfig=@{ DomainAdminName = 'LabAdmin'; AdminPassword = 'LS1setup!'; Prefix = 'SDNExpress2019-'; SecureBoot = $false; SwitchName = 'LabSwitch'; DCEdition = '4'; VMs = @(); InstallSCVMM = 'No'; PullServerDC = $false; Internet = $true; AllowedVLANs = "1-400"; AdditionalNetworksInDC = $true; AdditionalNetworksConfig = @(); EnableGuestServiceInterface = $true}
   $LABConfig.AdditionalNetworksConfig += @{
        NetName = 'HNV';
        NetAddress = '10.103.33.';
        NetVLAN = '201';
        Subnet = '255.255.255.0'
    }

   1..4 | % {
   $VMNames = "HV";
   $LABConfig.VMs += @{
        VMName = "$VMNames$_";
        Configuration = 'S2D';
        ParentVHD = 'Win2019Core_G2.vhdx';
        SSDNumber = 2;
        SSDSize = 800GB;
        HDDNumber = 4;
        HDDSize = 4TB;
        MemoryStartupBytes = 20GB;
        NestedVirt = $True;
        StaticMemory = $True;
        VMProcessorCount = 6
       }
   }

   $LABConfig.VMs += @{
        VMName = "Management";
        Configuration = 'S2D';
        ParentVHD = 'Win2019_G2.vhdx';
        SSDNumber = 1;
        SSDSize = 50GB;
        MemoryStartupBytes = 4GB;
        NestedVirt = $false;
        StaticMemory = $false;
        VMProcessorCount = 4
   }
   ```

1. Copy the **Scenario.ps1** and **MultiNodeConfig.psd1** files from **F:\\WSLab-master\\Scenarios\\SDNExpress with Windows Admin Center** to **F:\\WSLab-master\\Scripts**.
1. From the Windows PowerShell ISE window, run the **F:\\WSLab-master\\Scripts\\3_Deploy.ps1** script to provision **SDNExpress2019-DC** based on the **DC** VM and the remaining VMs for the SDN environment.

   > **Note**: Select **None** at the Telemetry prompt. The script should complete in about 7 minutes.
 
1. In the Windows PowerShell ISE window, open the **F:\\WSLab-master\\Scripts\\Scenario.ps1** script, remove all content following the line **128**, starting from `# ENDING Run from Hyper-V Host ENDING #`, and then save the modified file as **Scenario_Part1.ps1**.

   > **Note**: You must run this part of the scenario script from the Hyper-V host.

1. In the Windows PowerShell ISE window, run the **F:\\WSLab-master\\Scripts\\Scenario_Part1.ps1** script to configure the VMs that will host the lab environment. When prompted for the location of the parent VHDX for the SDN VMs, point to **F:\\WSLab-master\\Scripts\\ParentDisks\\Win2019Core_G2.vhdx**. When prompted for the **MultiNodeConfig.psd1** file, point to the file you copied to **F:\\WSLab-master\\Scripts**. When prompted for Windows Admin Center MSI, point to the downloaded Windows Installer file in the **F:\\Source** folder.

   > **Note**: If the script fails with the message **Copy-VMFile : Failed to initiate copying files to the guest**, rerun the script.

   > **Note**: The script should complete in about 15 minutes.

   > **Note**: Ignore the error following the line **ScriptHalted** and message prompting to restart **SDNExpress2019-Management**. That's expected.

1. After the script completes, in the Windows PowerShell ISE window, run the following script to expand the size of the disks hosting drive **C** of the newly provisioned VMs that will host the SDN environment:

   ```powershell
   $servers = @('SDNExpress2019-HV1','SDNExpress2019-HV2','SDNExpress2019-HV3','SDNExpress2019-HV4')
   $paths = (Get-VM -Name $servers | Get-VMHardDiskDrive | Where-Object {$_.ControllerLocation -eq 0} | Select-Object Path).Path
   foreach ($path in $paths) { Resize-VHD -Path $path -SizeBytes 100GB }
   ```

### Task 2: Deploy the SDN infrastructure VMs

   > **Note**: Sign in to the **DC** VM using the **CORP\\LabAdmin** username and **LS1setup!** password, run `slmgr -rearm` and restart it.

1. On the lab VM, use the Hyper-V Manager console to connect to the **SDNExpress2019-Management** VM. When prompted to sign in, provide the **CORP\\LabAdmin** username and **LS1setup!** password.
1. Within the console session to the **SDNExpress2019-Management** VM, start Windows PowerShell ISE as Administrator and run the following script to expand the size of drive **C** of the VMs that will host the SDN environment:

   ```powershell
   $servers = @('HV1','HV2','HV3','HV4')
   Invoke-Command -ComputerName $servers -ScriptBlock {
     $size = Get-PartitionSupportedSize -DriveLetter C
     Resize-Partition -DriveLetter C -Size $size.SizeMax
   }
   ```
1. On the lab VM, from the **script** pane of the Administrator: Windows PowerShell ISE window, run the following commands to download the following file:

    ```powershell
   New-Item F:\Allfiles -itemtype directory -Force
   Invoke-Webrequest -Uri "https://raw.githubusercontent.com/MicrosoftLearning/WS-013T00-Azure-Stack-HCI/master/Allfiles/SDNExpressModule.psm1" -Outfile "F:\Allfiles\SDNExpressModule.psm1"
    ```

1. Within the console session to the **SDNExpress2019-Management** VM, start File Explorer and navigate to the **C:\\Library** folder.

1. Switch back to the lab VM and use the copy and paste functionality of the **Hyper-V** console session to copy **F:\\WSLab-master\\Scripts\\Scenario.ps1** and **F:\\Allfiles\\SDNExpressModule.psm1** on the lab VM to **C:\\Library** on the **SDNExpress2019-Management** VM.

1. Within the console session to the **SDNExpress2019-Management** VM, in the Administrator: Windows PowerShell ISE window, open the **C:\\Library\\Scenario.ps1** script, and comment out line 375 so it looks like so: `# Expand-Archive -Path C:\SDN-Master.zip -DestinationPath C:\Library` 

1. Within the console session to the **SDNExpress2019-Management** VM, in the Administrator: Windows PowerShell ISE window, remove all content before the line 136, up to the line prior to `# Run from DC / VMM #`, and then save the modified file as **Scenario_Part2.ps1**.

1. Within the console session to the **SDNExpress2019-Management** VM, in the Administrator: Windows PowerShell ISE window, run the following command:

   ```powershell
   Expand-Archive -Path C:\SDN-Master.zip -DestinationPath C:\Library
   Copy-Item -Path C:\Library\SDNExpressModule.psm1 -Destination C:\Library\SDN-master\SDNExpress\scripts -Force
   ```
   > **Note**: This part of the scenario script needs to be run from the management VM.

1. Within the console session to the **SDNExpress2019-Management** VM, in the Administrator: Windows PowerShell ISE window, run the newly saved **C:\\Library\\Scenario_Part2.ps1** script to configure the SDN VMs.

   > **Note**: Wait until the script completes before you proceed. The script should complete in about 90 minutes. Disregard any cluster validation errors.

### Results

After completing this lab, you will have successfully provisioned the SDN infrastructure.
