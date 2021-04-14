---
lab:
    title: 'Lab A: Deploying Software-Defined Networking'
    type: 'Answer Key'
    module: 'Module 4: Planning for and Implementing Azure Stack HCI Networking'
---
# Lab A answer key: Deploying Software-Defined Networking

## Exercise 1: Deploying Software-Defined Networking by using PowerShell

### Task 1: Deploy the VMs that will serve as the SDN infrastructure Hyper-V hosts

1. On the lab virtual machine (VM), start Windows PowerShell Integrated Scripting Environment (ISE) as Administrator, and from the **console** pane, run the following to remove the **Zone.Identifier** alternate data stream, which has a value of **3** indicating that it was downloaded from the internet:

   ```powershell
   Get-ChildItem -Path F:\WSLab-master\ -File -Recurse | Unblock-File
   ```

1. On the lab VM, from the **console** pane of the Administrator: Windows PowerShell ISE window, run the following to set the current directory:

   ```powershell
   Set-Location -Path F:\WSLab-master\Scripts
   ```

1. On the lab VM, from the **script** pane of the Administrator: Windows PowerShell ISE window, run the following commands to rename **LabConfig.ps1** and **Scenario.ps1**.

   ```powershell
   Move-Item -Path '.\LabConfig.ps1' -Destination '.\LabConfig.m4l0.ps1' -Force -ErrorAction SilentlyContinue
   Move-Item -Path '.\Scenario.ps1' -Destination '.\Scenario.m4l0.ps1' -Force -ErrorAction SilentlyContinue
   ```

1. On the lab VM, in the Administrator: Windows PowerShell ISE window, open a new tab in the **script** pane, paste the following content, and then save it as **F:\\WSLab-master\\Scripts\\LabConfig.ps1**:

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

1. On the lab VM, in the Administrator: Windows PowerShell ISE window, from the **script** pane, run the following commands to copy the **Scenario.ps1** and **MultiNodeConfig.psd1** files from **F:\WSLab-master\Scenarios\SDNExpress with Windows Admin Center** to the current directory:

   ```powershell
   Copy-Item -Path 'F:\WSLab-master\Scenarios\SDNExpress with Windows Admin Center\Scenario.ps1' -Destination '.\'
   Copy-Item -Path 'F:\WSLab-master\Scenarios\SDNExpress with Windows Admin Center\MultiNodeConfig.psd1' -Destination '.\'
   ```

1. In the Administrator: Windows PowerShell ISE window open and run the **F:\\WSLab-master\\Scripts\\3_Deploy.ps1** script to provision VMs for the SDN environment.

   > **Note**: The script should complete in about 7 minutes. When prompted **Press enter to continue**, select the **Enter** key.

1. In the Administrator: Windows PowerShell ISE window, open the **F:\\WSLab-master\\Scripts\\Scenario.ps1** script, remove all content following the line **128**, starting from `# ENDING Run from Hyper-V Host ENDING #`, and then save the modified file as **Scenario_Part1.ps1**.

   > **Note**: This part of the scenario script needs to be run from the Hyper-V host.

1. From the Administrator: Windows PowerShell ISE window, run the newly saved **F:\\WSLab-master\\Scripts\\Scenario_Part1.ps1** script to configure the VMs that will host the lab environment. When prompted for the location of the parent virtual hard disk (VHDX) for SDN VMs, point to **F:\\WSLab-master\\Scripts\\ParentDisks\\Win2019Core_G2.vhdx**. When prompted for the **MultiNodeConfig.psd1** file, point to the file you copied to **F:\\WSLab-master\\Scripts**. When prompted for Windows Admin Center MSI, point to the downloaded Windows Installer file in the **F:\\Source** folder.

   > **Note**: If the script fails with the message **Copy-VMFile : Failed to initiate copying files to the guest**, rerun the script.

   > **Note**: The script should complete in about 15 minutes.

   > **Note**: Ignore the error following the line **ScriptHalted** and message prompting to restart **SDNExpress2019-Management**. That's expected.

1. After the script completes, in the Administrator: Windows PowerShell ISE window, open a new tab, and run the following script to expand the size of the disks hosting drive **C** of the newly provisioned VMs that will host the SDN environment:

   ```powershell
   $servers = @('SDNExpress2019-HV1','SDNExpress2019-HV2','SDNExpress2019-HV3','SDNExpress2019-HV4')
   $paths = (Get-VM -Name $servers | Get-VMHardDiskDrive | Where-Object {$_.ControllerLocation -eq 0} | Select-Object Path).Path
   foreach ($path in $paths) { Resize-VHD -Path $path -SizeBytes 100GB }
   ```

### Task 2: Deploy the SDN infrastructure VMs

   > **Note**: Make sure all of the VMs you provisioned in the previous task are running and that their operating system has been activated before you proceed to the next task. If that is not the case, start all of the VMs, sign in to each of them  using the **CORP\\LabAdmin** username and **LS1setup!** password and, from the elevated Command Prompt, run `slmgr -rearm`. The **DC** VM will require you to run `slmgr -rearm` and be restarted. Be sure to logoff from all VMs when complete.

1. On the lab VM, start the **Hyper-V Manager** console and establish a console session to the **SDNExpress2019-Management** VM. When prompted to sign in, provide the **CORP\\LabAdmin** username and **LS1setup!** password.

1. Within the console session to the **SDNExpress2019-Management** VM, start Windows PowerShell ISE as Administrator.

1. Within the console session to the **SDNExpress2019-Management** VM, in the Administrator: Windows PowerShell ISE window, in the **script** pane, open a new tab, paste the following script, and then run it to expand the size of drive **C** of the VMs that will host the SDN environment:

   ```powershell
   $servers = @('HV1','HV2','HV3','HV4')
   Invoke-Command -ComputerName $servers -ScriptBlock {
     $size = Get-PartitionSupportedSize -DriveLetter C
     Resize-Partition -DriveLetter C -Size $size.SizeMax
   }
   ```

1. Within the console session to the **SDNExpress2019-Management** VM, start File Explorer and navigate to the **C:\\Library** folder.

1. Switch back to the lab VM and use the copy and paste functionality of the **Hyper-V** console session to copy **F:\\WSLab-master\\Scripts\\Scenario.ps1** on the lab VM to **C:\\Library** on the **SDNExpress2019-Management** VM.

1. Within the console session to the **SDNExpress2019-Management** VM, in the Administrator: Windows PowerShell ISE window, open the **C:\\Library\\Scenario.ps1** script, remove all content before the line 136, up to the line prior to `# Run from DC / VMM #`, and then save the modified file as **Scenario_Part2.ps1**.

   > **Note**: This part of the scenario script needs to be run from the management VM.

1. Within the console session to the **SDNExpress2019-Management** VM, in the Administrator: Windows PowerShell ISE window, run the newly saved **C:\\Library\\Scenario_Part2.ps1** script to configure the SDN VMs.

   > **Note**: Wait until the script completes before you proceed. The script should complete in about 90 minutes. Disregard any cluster validation errors.
