---
lab:
    title: 'Lab A: Implementing a Storage Spaces Direct cluster by using Windows Admin Center'
    module: 'Module 3: Planning for and implementing Azure Stack HCI Storage'
---

# Lab A: Implementing a Storage Spaces Direct cluster by using Windows Admin Center

## Scenario

You will provision the first proof of concept environment that consists of a single Storage Spaces Direct cluster by using Windows Admin Center, and then review the resulting configuration.

## Objectives

After completing this lab, you'll be able to implement a Storage Spaces Direct cluster by using Windows Admin Center.

## Estimated time: 80 minutes

## Lab setup

To connect to the virtual machine (VM) for the lab, follow the steps provided to you by the lab hosting provider.

## Exercise 1: Implementing a Storage Spaces Direct cluster by using Windows Admin Center

### Scenario

Your proof of concept environment relies on nested virtualization and PowerShell scripts that leverage the [WSLab GitHub project](https://github.com/microsoft/WSLab). The scripts provision a number of Hyper-V VMs that run Windows Server 2019. All of the Hyper-V VMs are part of the same Active Directory Domain Services (AD DS) domain, including a single domain controller, one or more Storage Spaces Direct clusters, and a management server. You will be provisioning multiple proof of concept environments throughout this lab in the same manner. In each case, you start by running the same set of scripts with a different set of configuration parameters and configure the management server.

You will find the scripts and artifacts used to provision Hyper-V VMs in the **F:\\WSLab-master** directory on the lab VM. The **F:\\Source** directory contains binaries and a single Windows Server 2019 Datacenter Evaluation Edition ISO file that serve as the basis for the initial setup of the lab environment. That initial setup has already been completed for you to minimize the time it takes to provision each environment that you will test.

In this exercise, first you will prepare for provisioning of the first environment that  consists of a single Storage Spaces Direct cluster. After that, you will deploy it by using Windows Admin Center, and then review the resulting configuration.

The main tasks for this exercise are as follows:

1. Deploy VM servers that will host Storage Spaces Direct infrastructure.
1. Configure the management server.
1. Prepare Windows Admin Center for deployment of a Storage Spaces Direct cluster.
1. Deploy a Storage Spaces Direct cluster by using Windows Admin Center.
1. Connect to a Storage Spaces Direct cluster by using Windows Admin Center.
1. Review the installation of the Storage Spaces Direct cluster on the lab VMs.
1. Deprovision the lab resources.

### Task 1: Deploy VM servers that will host Storage Spaces Direct infrastructure

1. On the lab VM, start Windows PowerShell ISE as an administrator and run the following command to remove the **Zone.Identifier** alternate data stream, which has a value of **3**, indicating that it was downloaded from the internet:

   ```powershell
   Get-ChildItem -Path F:\WSLab-master\ -File -Recurse | Unblock-File
   ```

1. On the lab VM, in the **Administrator: Windows PowerShell ISE** window, run the following command to set the current directory:

   ```powershell
   Set-Location -Path F:\WSLab-master\Scripts
   ```

1. On the lab VM, in the **Administrator: Windows PowerShell ISE** window, run the following command to rename **LabConfig.ps1**:

   ```powershell
   Move-Item -Path '.\LabConfig.ps1' -Destination '.\LabConfig.m3l1.ps1' -Force -ErrorAction SilentlyContinue
   ```

1. On the lab VM, in the **Administrator: Windows PowerShell ISE** window, create a new file **F:\\WSLab-master\\Scripts\\LabConfig.ps1** with the following command:

   ```powershell
   $LabConfig=@{ DomainAdminName = 'LabAdmin'; AdminPassword = 'LS1setup!'; Prefix = 'WSLab-'; SwitchName = 'LabSwitch'; DCEdition = '4'; Internet = $true; AdditionalNetworksConfig = @(); VMs = @()}
   1..4 | ForEach-Object {
        $VMNames = "S2D";
        $LABConfig.VMs += @{
           VMName = "$VMNames$_";
           Configuration = 'S2D';
           ParentVHD = 'Win2019Core_G2.vhdx';
           HDDNumber = 8;
           HDDSize = 4TB;
           MemoryStartupBytes = 4GB;
           StaticMemory = $True;
           NestedVirt = $True;
           VMProcessorCount = 2
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

1. On the lab VM, in the **Administrator: Windows PowerShell ISE** window, run the **F:\\WSLab-master\\Scripts\\3_Deploy.ps1** script that provisions VMs for the Storage Spaces Direct environment.

   > **Note:** The script should complete in less than 10 minutes. When prompted with **Press enter to continue**, select Enter.

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

1. On the lab VM, from **Hyper-V Manager**, connect via a console session to **WSLab-Management**. When prompted to sign in, provide the username **CORP\\LabAdmin** and the password **LS1setup!**.
1. In the **WSLab-Management** VM console session, start **Windows PowerShell ISE** as an administrator.
1. In the **WSLab-Management** VM console session, in the **Administrator: Windows PowerShell ISE** window, run the following command to install Remote Server Administration Tools:

   ```powershell
   Install-WindowsFeature -Name RSAT-Clustering,RSAT-Clustering-Mgmt,RSAT-Clustering-PowerShell,RSAT-Hyper-V-Tools,RSAT-AD-PowerShell,RSAT-ADDS
   ```

   > **Note:** Proceed to the next step without waiting for the installation to complete.

1. In the **WSLab-Management** VM console session, start another instance of Windows PowerShell ISE as an Administrator.
1. In the **WSLab-Management** VM console session, from the newly started **Administrator: Windows PowerShell ISE** window, run the following command to download and install the Microsoft Edge (Chromium) browser:

   ```powershell
   $progressPreference='SilentlyContinue'
   Invoke-WebRequest -Uri "https://go.microsoft.com/fwlink/?linkid=2069324&language=en-us&Consent=1" -UseBasicParsing -OutFile "$env:USERPROFILE\Downloads\MicrosoftEdgeSetup.exe"
   Start-Process -FilePath "$env:USERPROFILE\Downloads\MicrosoftEdgeSetup.exe" -Wait
   ```

   > **Note:** Proceed to the next step without waiting for the installation to complete.

1. In the **WSLab-Management** VM console session, start another instance of Windows PowerShell ISE as Administrator.
1. In the **WSLab-Management** VM console session, from the newly started **Administrator: Windows PowerShell ISE** window, run the following command to download and install Windows Admin Center:

   ```powershell
   Invoke-WebRequest -UseBasicParsing -Uri https://aka.ms/WACDownload -OutFile "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v waclog.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > **Note:** Proceed to the next step without waiting for the installation to complete.

1. Switch to the first **Administrator: Windows PowerShell ISE** window where you initiated installation of the Remote Server Administration Tools, wait for the installation to complete, and then from the **script** pane, run the following command to configure Kerberos constrained delegation to minimize prompts for credentials when using Windows Admin Center:

   ```powershell
   $gateway = "Management"
   $nodes = Get-ADComputer -Filter * -SearchBase "ou=workshop,DC=corp,dc=contoso,DC=com"
   $gatewayObject = Get-ADComputer -Identity $gateway
   foreach ($node in $nodes){
    Set-ADComputer -Identity $node -PrincipalsAllowedToDelegateToAccount $gatewayObject
   }
   ```

1. In the **WSLab-Management** VM console session, from the same the **Administrator: Windows PowerShell ISE** window, run the following command to enable Credential Security Support Provider (CredSSP) authentication from the management server to the four Hyper-V servers that will host the Storage Spaces Direct cluster:

   ```powershell
   $servers = @('S2D1','S2D2','S2D3','S2D4')
   Enable-WSMANCredSSP -Role client -DelegateComputer $servers -Force
   ```

1. In the **WSLab-Management** VM console session, from the same **Administrator: Windows PowerShell ISE** window, run the following command to enable Credential Security Support Provider (CredSSP) authentication on the four Hyper-V servers that will host the Storage Spaces Direct cluster:

   ```powershell
   $servers = @('S2D1','S2D2','S2D3','S2D4')
   Invoke-Command -ComputerName $servers -ScriptBlock {Enable-WSMANCredSSP -Role server -Force}
   ```

   > **Note:** This step is necessary to be able to use Windows Admin Center for deployment of a Storage Spaces Direct cluster.

   > **Note:** CredSSP authentication delegates the user credentials from the local computer to a remote computer. This configuration increases the security risk of the remote operation. If the remote computer is compromised, when credentials are passed to it, the credentials can be used to control the network session.

   > **Note:** Before you proceed to the next step, verify that the installation of Microsoft Edge and Windows Admin Center completed.

1. Close the other two instances of the **Administrator: Windows PowerShell ISE** window you opened earlier in this task.
1. Switch to the **Microsoft Edge** browser window, navigate to `https://management.corp.contoso.com` and, when prompted to authenticate, sign in as **CORP\\LabAdmin** with the password **LS1setup!**.

### Task 3: Prepare Windows Admin Center for deployment of a Storage Spaces Direct cluster

1. In the **WSLab-Management** VM console session, in the browser window displaying Windows Admin Center, uninstall the **Cluster Creation (Preview)** extension version 1.294.0.
1. In Windows Admin Center, install the **Cluster Creation (Preview)** extension version 1.2.0.

   > **Note:** This step is necessary to install the current version of a Storage Spaces Direct hyperconverged cluster (rather than the Azure Stack HCI version 20H2, which is in preview as of the time of writing).

1. In Windows Admin Center, connect to `management.corp.contoso.com` as **CORP\\LabAdmin** with the password **LS1setup!**.

### Task 4: Deploy a Storage Spaces Direct cluster by using Windows Admin Center

1. In the **WSLab-Management** VM console session, from the browser window displaying Windows Admin Center, launch the **Cluster Creation** wizard and select the **Hyperconverged** cluster type.
1. On the **Check the prerequisites** pane of the **Deploy hyperconverged infrastructure** wizard, review the list of prerequisites.
1. On the **Enter an account** pane of the **Deploy hyperconverged infrastructure** wizard, specify the following credentials:

   *Table 1: Deploy hyperconverged infrastructure user credentials*

   |Setting|Value|
   |---|---|
   |Username|CORP\\LabAdmin|
   |Password|LS1setup!|

1. On the **Add servers** pane of the **Deploy hyperconverged infrastructure** wizard, add `S2D1.corp.contoso.com`, `S2D2.corp.contoso.com`, `S2D3.corp.contoso.com`, and `S2D4.corp.contoso.com`, and verify that all of them appear with a status of **Online**.
1. On the **Install features** pane of the **Deploy hyperconverged infrastructure** wizard, select **Install features** and verify that the installation of all features completed successfully.

   > **Note:** Wait for the installation to complete. This should take about three minutes.

1. On the **Restart servers** pane of the **Deploy hyperconverged infrastructure** wizard, select **Restart servers**.

   > **Note:** Disregard a warning about issues with restarting the servers. Instead, switch to the lab VM. In the **Hyper-V Manager** console, connect to each of the four VMs and verify that all of them successfully restarted, and then switch the console session to the **WSLab-Management** VM.

1. On the **Verify network adapters** pane of the **Deploy hyperconverged infrastructure** wizard, ensure that both network adapters of each Hyper-V host displays with the **Up** status.
1. On the **Select management adapters** pane of the **Deploy hyperconverged infrastructure** wizard, for each pair of network adapters on each of the four Hyper-V hosts, select the check box next to the network adapter labeled **Microsoft Hyper-V Network Adapter #2**.
1. On the **Edit adapters properties** pane of the **Deploy hyperconverged infrastructure** wizard, review the current settings, including the option to set the packet size, without making any changes.
1. On the **Virtual switch** pane of the **Deploy hyperconverged infrastructure** wizard, expand the **Advanced** section, review the autogenerated switch settings, including the switch name (**ConvergedSwitch**), the load balancing algorithm set to **Hyper-V port (recommended)**, **Use VMMQ (recommended)** enabled, and the number of queue pairs set to **8**, and then initiate creation of a virtual switch.

   > **Note:** Wait until the virtual switch is created. This should take about two minutes.

1. On the **Validate the cluster** pane of the **Deploy hyperconverged infrastructure** wizard, start cluster validation. If prompted to enable Credential Security Service Provider (CredSSP), select **Yes**.

   > **Note:** Wait until the cluster validation is completed. This should take about five minutes.

   > **Note:** Disregard the warning regarding the validation of network configuration. It's expected.

1. On the **Create the cluster** pane of the **Deploy hyperconverged infrastructure** wizard, set the **Cluster name** to **Storage Spaces Direct-cluster**, assign a static IP address of **10.0.0.100** to the cluster, and then initiate cluster creation.

   > **Note:** Wait until the cluster is created. This should take about two minutes.

1. On the **Verify drives** pane of the **Deploy hyperconverged infrastructure** wizard, verify that there are 32 drives in total and that all report a status of **OK**.
1. From the **Clean drives** pane of the **Deploy hyperconverged infrastructure** wizard, proceed to the next step without cleaning the drives.
1. On the **Validate storage** pane of the **Deploy hyperconverged infrastructure** wizard, start storage validation. If prompted to enable Credential Security Service Provider (CredSSP), select **Yes**.

1. On the **Validate** pane of the **Deploy hyperconverged infrastructure** wizard, ensure that all of the results were successful and then proceed to the next step.
1. On the **Enable Storage Spaces Direct** pane of the **Deploy hyperconverged infrastructure** wizard, enable **Storage Spaces Direct**.
1. When the process of enabling **Storage Spaces Direct** completes, proceed to the connections list.

### Task 5: Connect to a Storage Spaces Direct cluster by using Windows Admin Center

1. In the **WSLab-Management** VM console session, in the browser window displaying Windows Admin Center, connect to the **s2d-cluster** as **CORP\\LabAdmin** with the password **LS1setup!**.

### Task 6: Review the installation of the Storage Spaces Direct cluster on the lab VMs

1. In the **WSLab-Management** VM console session, in the browser window displaying Windows Admin Center, connect to the **s2d-cluster** cluster and authenticate as **CORP\LabAdmin** with the password **LS1setup!**.
1. In the browser window displaying Windows Admin Center, on the `s2d-cluster.corp.contoso.com` page, navigate to the inventory of the Storage Spaces Direct cluster volumes.
1. Note that at this point, the cluster contains only the **ClusterPerformanceHistory** volume. Verify that the volume is listed with the status **OK**.
1. In the browser window displaying Windows Admin Center, on the `s2d-cluster.corp.contoso.com` page, navigate to the inventory of the Storage Spaces Direct cluster drives.
1. Review the list of drives and verify that each of them is listed with the status **OK**. Note that all drives are HDD capacity drives and are part of a single storage pool named **Storage Spaces Direct on s2d-cluster**.
1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, review the virtual switch configuration. Note that each node is connected to an external switch named **ConvergedSwitch**.
1. Review the list of network adapters and verify that the load balancing algorithm is set to **Hyper-V port**.
1. On the **Settings** panel, select **Storage Spaces Direct** and review the cache settings.

   > **Note:** Verify that the **Cache mode for HDD** is set by default to **Read/Write** and **Cache mode for SSD** is set to **Write only**, but you have the option of modifying these settings.

1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, on the **Settings** panel, in the **Cluster** section, review the following entries:

      - Access point. Verify that **Cluster name** is set to **S2D-Cluster**.
      - Node shutdown behavior. Verify that the setting **Move virtual machines on node shutdown** is enabled.
      - Cluster traffic encryption. Verify that **Core traffic** is set to **Sign** and **Storage traffic** to **Clear text**.
      - Virtual machine load balancing. Verify that **Balance virtual machines** is set to **Always** with **Aggressiveness** set to **Low**.
      - Witness. Verify that **Witness type** is set to **File share witness** with **File share path** set to **\\\\DC\\S2D-ClusterWitness**.

1. In the **WSLab-Management** VM console session, switch to the **Administrator: Windows PowerShell ISE** window and run the following command to set up a file share-based witness for the **S2D-Cluster** failover cluster:

   ```powershell
   $clusterName = 'S2D-Cluster'
   $witnessName = $clusterName + "Witness"
   Invoke-Command -ComputerName DC -ScriptBlock {New-Item -Path c:\Shares -Name $using:witnessName -ItemType Directory}
   $accounts = @()
   $accounts += "CORP\$($clusterName)$"
   $accounts += 'CORP\Domain Admins'
   New-SmbShare -Name $WitnessName -Path "c:\Shares\$witnessName" -FullAccess $accounts -CimSession DC -ErrorAction SilentlyContinue
   Invoke-Command -ComputerName DC -ScriptBlock {(Get-SmbShare $using:witnessName).PresetPathAcl | Set-Acl}
   # Set Quorum
   Set-ClusterQuorum -Cluster $clusterName -FileShareWitness "\\DC\$witnessName"
   ```

1. In the **WSLab-Management** VM console session, switch to Windows Admin Center, on the `s2d-cluster.corp.contoso.com` page, on the **Settings** panel, in the **Cluster** section, select the **Witness** entry and verify that the **Witness type** is set to **File share witness** with the **File share path** set to **\\\\DC\\S2D-ClusterWitness**.
1. In the **WSLab-Management** VM console session, start **Failover Cluster Manager**.
1. In **Failover Cluster Manager**, connect to the `s2d-cluster.corp.contoso.com` cluster, review the list of virtual disks (volumes), verify that the cluster contains a single pool labeled **Cluster Pool 1**, and examine the storage pool properties, including virtual and physical disks. Note that the pool name is set to **S2D on s2d-cluster**.

### Task 7: Deprovision the lab resources

1. Switch to the lab VM.
1. On the lab VM, in the **Administrator: Windows PowerShell ISE** window, run the **F:\\WSLab-master\\Scripts\\Cleanup.ps1** script to remove all VMs provisioned in this lab.
1. In the **Administrator: Windows PowerShell ISE** window, close the tab displaying the **F:\\WSLab-master\\Scripts\\Cleanup.ps1** script.

## Results

After completing this lab, you will have prepared for deployments of a Storage Spaces Direct cluster, implemented a Storage Spaces Direct cluster by using Windows Admin Center, reviewed the resulting configuration, and deprovisioned the lab environment.
