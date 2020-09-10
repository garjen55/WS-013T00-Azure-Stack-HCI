---
lab:
    title: 'Lab B: Implementing a Storage Spaces Direct cluster by using Windows PowerShell'
    type: 'Answer Key'
    module: 'Module 3: Planning for and implementing Azure Stack HCI Storage'
---
# Lab B answer key: Implementing a Storage Spaces Direct cluster by using Windows PowerShell

## Exercise 1: Implementing a Storage Spaces Direct cluster by using Windows PowerShell

### Task 1: Provision the lab environment VMs

1. On the lab VM, in the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to rename **LabConfig.ps1** and **Scenario.ps1**:

   ```powershell
   Set-Location -Path 'F:\WSLab-master\Scripts'
   Move-Item -Path '.\LabConfig.ps1' -Destination '.\LabConfig.m3l2.ps1' -Force -ErrorAction SilentlyContinue
   Move-Item -Path '.\Scenario.ps1' -Destination '.\Scenario.m3l2.ps1' -Force -ErrorAction SilentlyContinue
   ```

1. On the lab VM, in the **Administrator: Windows PowerShell ISE** window, open a new tab on the **script** pane, paste the following command, and then save it as **F:\\WSLab-master\\Scripts\\LabConfig.ps1**:

   ```powershell
   $LabConfig=@{ DomainAdminName = 'LabAdmin'; AdminPassword = 'LS1setup!'; Prefix = 'WSLab-'; SecureBoot = $false; SwitchName = 'LabSwitch'; DCEdition = '4'; VMs = @(); InstallSCVMM = 'No'; PullServerDC = $false; Internet = $true ; AdditionalNetworksConfig = @(); EnableGuestServiceInterface = $true; AddToolsVHD = $True ; DisableWCF = $True }
   1..4 | % {
        $VMNames = "S2D";
        $LABConfig.VMs += @{
        VMName = "$VMNames$_" ;
        Configuration = 'S2D' ;
        ParentVHD = 'Win2019Core_G2.vhdx';
        HDDNumber = 12;
        HDDSize = 4TB ;
        MemoryStartupBytes = 4GB;
        NestedVirt = $True
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

   > **Note:** The script should complete in about 10 minutes. When prompted **Press enter to continue**, select the **Enter** key.

1. After the script completes, in the **Administrator: Windows PowerShell ISE** window, from the **console** pane, run the following command to start the newly provisioned VMs that will host the Storage Spaces Direct environment:

   ```powershell
   Get-VM -Name 'WSLab-Management' | Start-VM
   Start-Sleep 150
   Get-VM | Where-Object Name -like 'WSLab-S2D*' | Start-VM -AsJob
   ```

1. On the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to copy the **Scenario.ps1** file from **F:\\WSLab-master\\Scenarios\\S2D Hyperconverged** to the current directory:

   ```powershell
   Copy-Item -Path 'F:\WSLab-master\Scenarios\S2D Hyperconverged\Scenario.ps1' -Destination '.\'
   ```

1. On the lab VM, start **Hyper-V Manager** and connect via a console session to **WSLab-DC**. When prompted to sign in, provide the username **CORP\\LabAdmin** and the password **LS1setup!**.
1. In the **WSLab-DC** VM console session, start **Windows PowerShell ISE** as an administrator.
1. From the **Administrator: Windows PowerShell ISE** window, run `slmgr -rearm` and then select **OK**.
1. From the **Administrator: Windows PowerShell ISE** window, run `Restart-Computer -Force`.

 > **Note**: Make sure that the **WSLab-DC** VM is running before you proceed to the next task.

### Task 2: Configure the management server

1. On the lab VM, in the **Server Manager** window, select **Tools**, and then from the drop-down list, select **Hyper-V Manager**.
1. On the lab VM, in the **Hyper-V Manager** console, in the list of virtual machines, right-click or access the context menu for the **WSLab-Management** entry, and then select **Connect** to establish a console session to the **WSLab-Management** VM. When prompted to sign in, provide the username **CORP\\LabAdmin** and password **LS1setup!**.
1. In the console session to the **WSLab-Management** VM, start **Windows PowerShell ISE** as Administrator.
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

1. In the console session to the **WSLab-Management** VM, start another instance of **Windows PowerShell ISE** as Administrator.
1. In the console session to the **WSLab-Management** VM, from the **script** pane of the newly started **Administrator: Windows PowerShell ISE** window, run the following command to download and install Windows Admin Center:

   ```powershell
   Invoke-WebRequest -UseBasicParsing -Uri https://aka.ms/WACDownload -OutFile "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v waclog.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > **Note:** Proceed to the next step without waiting for the installation to complete.

1. Return to the first **Administrator: Windows PowerShell ISE** window where you initiated installation of the Remote Server Administration Tools, wait for the installation to complete, and then from the **script** pane, run the following command to configure Kerberos constrained delegation to minimize prompts for credentials when using Windows Admin Center:

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
1. Switch to the Microsoft Edge browser window, select **Get started**, accept the default tab page settings, select the **Continue without Signing-in** link, use the Microsoft Edge browser to navigate to `https://management.corp.contoso.com`, and when prompted to authenticate, sign in as **CORP\\LabAdmin** with the password **LS1setup!**.

### Task 3: Deploy a Storage Spaces Direct cluster on the lab VMs by using PowerShell

1. In the console session to the **WSLab-Management** VM, start File Explorer and create a **C:\\Library** folder.
1. Switch to the lab VM, start File Explorer, navigate to the **F:\\WSLab-master\\Scripts\\** folder, right-click or access the context menu for **Scenario.ps1**, and then in the context menu, select **Copy**.
1. Switch to the **WSLab-Management** VM, in the File Explorer window, right-click or access the context menu for the **C:\\Library** folder, and then select **Paste**.
1. In the console session to the **WSLab-Management** VM, switch to the **Administrator: Windows PowerShell ISE** window, and in the **Administrator: Windows PowerShell ISE** window, open the **C:\\Library\\Scenario.ps1** script.
1. In the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the **Scenario.ps1** script to deploy and configure a Storage Spaces Direct cluster on the lab VMs.

   > **Note:** Wait for the script to complete before you proceed to the next lab. The script should complete in about 35 minutes.
