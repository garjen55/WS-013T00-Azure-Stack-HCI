---
lab:
    title: 'Lab A: Implementing a Storage Spaces Direct cluster by using Windows Admin Center'
    type: 'Answer Key'
    module: 'Module 3: Planning for and implementing Azure Stack HCI Storage'
---
# Lab A answer key: Implementing a Storage Spaces Direct cluster by using Windows Admin Center

## Exercise 1: Implementing a Storage Spaces Direct cluster by using Windows Admin Center

### Task 1: Deploy VMs that will host Storage Spaces Direct infrastructure

1. On the lab VM, start Windows PowerShell ISE as Administrator, and from the **console** pane, run the following command to remove the **Zone.Identifier** alternate data stream, which has a value of **3** indicating that it was downloaded from the internet:

   ```powershell
   Get-ChildItem -Path F:\WSLab-master\ -File -Recurse | Unblock-File
   ```

1. On the lab VM, on the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to set the current directory:

   ```powershell
   Set-Location -Path F:\WSLab-master\Scripts
   ```

1. On the lab VM, on the **console** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to rename **LabConfig.ps1**:

   ```powershell
   Move-Item -Path '.\LabConfig.ps1' -Destination '.\LabConfig.m3l1.ps1' -Force -ErrorAction SilentlyContinue
   ```

1. On the lab VM, in the **Administrator: Windows PowerShell ISE** window, open a new tab in the **script** pane, paste the following command, and then save it as **F:\\WSLab-master\\Scripts\\LabConfig.ps1**:

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

1. On the lab VM, in the **Administrator: Windows PowerShell ISE** window, open and run the **F:\\WSLab-master\\Scripts\\3_Deploy.ps1** script to provision VMs for the Storage Spaces Direct environment.

   > **Note:** The script should complete in less than 10 minutes. When prompted to **Press enter to continue**, select Enter.

1. Once the script completes, in the **Administrator: Windows PowerShell ISE** window, from the **script** pane, run the following command to start the newly provisioned VMs that will host the Storage Spaces Direct environment:

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
1. On the lab VM, in the **Hyper-V Manager** console, in the list of virtual machines, right-click or access the context menu for the **WSLab-Management** entry, and then select **Connect** to establish a console session to the **WSLab-Management** VM. When prompted to sign in, provide the username **CORP\\LabAdmin** and the password **LS1setup!**.
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

1. Switch back to the first **Administrator: Windows PowerShell ISE** window where you initiated installation of the Remote Server Administration Tools, wait for the installation to complete, and then from the **script** pane, run the following command to configure Kerberos constrained delegation to minimize prompts for credentials when using Windows Admin Center:

   ```powershell
   $gateway = "Management"
   $nodes = Get-ADComputer -Filter * -SearchBase "ou=workshop,DC=corp,dc=contoso,DC=com"
   $gatewayObject = Get-ADComputer -Identity $gateway
   foreach ($node in $nodes){
    Set-ADComputer -Identity $node -PrincipalsAllowedToDelegateToAccount $gatewayObject
   }
   ```

1. In the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to enable Credential Security Support Provider (CredSSP) authentication from the management server to the four Hyper-V servers that will host the Storage Spaces Direct cluster:

   ```powershell
   $servers = @('S2D1','S2D2','S2D3','S2D4')
   Enable-WSMANCredSSP -Role client -DelegateComputer $servers -Force
   ```

1. In the console session to the **WSLab-Management** VM, from the **script** pane of the **Administrator: Windows PowerShell ISE** window, run the following command to enable CredSSP authentication on the four Hyper-V servers that will host the Storage Spaces Direct cluster:

   ```powershell
   $servers = @('S2D1','S2D2','S2D3','S2D4')
   Invoke-Command -ComputerName $servers -ScriptBlock {Enable-WSMANCredSSP -Role server -Force}
   ```

   > **Note:** This step is necessary to be able to use Windows Admin Center for deployment of a Storage Spaces Direct cluster.

   > **Note:** CredSSP authentication delegates the user credentials from the local computer to a remote computer. This configuration increases the security risk of the remote operation. If the remote computer is compromised when credentials are passed to it, the credentials can be used to control the network session.

   > **Note:** Before you proceed to the next step, verify that the installation of Microsoft Edge and Windows Admin Center is complete.

1. Close the other two instances of the **Administrator: Windows PowerShell ISE** window you opened earlier in this task without saving the scripts you ran from each.
1. Switch to the Microsoft Edge browser window, select **Get started**, accept the default tab page settings, select the **Continue without Signing-in** link, use the Microsoft Edge browser to navigate to `https://management.corp.contoso.com`, and then when prompted to authenticate, sign in as **CORP\\LabAdmin** with the password **LS1setup!**.

### Task 3: Prepare Windows Admin Center for deployment of a Storage Spaces Direct cluster

1. In the console session to the **WSLab-Management** VM, in the browser window displaying the Windows Admin Center interface, on the **All connections** page, select the gear icon to display the **Settings** page.
1. On the **Settings** page, select **Extensions**, on the **Extensions** panel, select the **Installed extensions** tab, select the **Cluster Creation (Preview)** extension entry (version 1.294.0), select **Uninstall**, and then when prompted to confirm, select **OK**. 
1. Return to the **Extensions** panel, select the **Available extensions** tab, select the **Cluster Creation (Preview)** entry (version 1.2.0), and then select **Install**.

   > **Note:** This step is necessary to install the current version of a Storage Spaces Direct hyperconverged cluster (rather than the Azure Stack HCI version 20H2, which is in preview as of 08/2020).

1. Wait until the installation completes and select the **Windows Admin Center** header.
1. Return to the **All connections** page of the **Windows Admin Center** page, and then select the `management.corp.contoso.com` link.
1. On the **Specify your credentials** panel, select the **Use another account for this connection** option, and then in the **Username** text box, enter **CORP\\LabAdmin**, and in the **Password** text box, enter **LS1setup!**, select the **Use these credentials for all connections**, and then select **Continue**.

### Task 4: Deploy a Storage Spaces Direct cluster by using Windows Admin Center

1. On the `management.corp.contoso.com` page, in the menu, select **Server Manager**, and in the drop-down list, select **Cluster Creation**.
1. On the **Choose the type of cluster to create** page, ensure that the **Hyperconverged** tile is selected, and then select **Create**.
1. On the **Check the prerequisites** pane of the **Deploy hyperconverged infrastructure** wizard, review the list of prerequisites, and then select **Next**.
1. On the **Enter an account** pane of the **Deploy hyperconverged infrastructure** wizard, specify the settings listed in the following table, and then select **Next**:

   *Table 1: Deploy hyperconverged infrastructure user credentials*

   |Setting|Value|
   |---|---|
   |Username|CORP\\LabAdmin|
   |Password|LS1setup!|

1. On the **Add servers** pane of the **Deploy hyperconverged infrastructure** wizard, in the `server.example.domain.com` text box, enter `S2D1.corp.contoso.com`, and then select **Add**.
1. Repeat the previous step to add the list of servers `S2D2.corp.contoso.com`, `S2D3.corp.contoso.com`, and `S2D4.corp.contoso.com`, ensure that all servers display with the **Online** status on the **Add servers** panel, and then select **Next**.
1. On the **Install features** pane of the **Deploy hyperconverged infrastructure** wizard, select **Install features**.
1. Verify that the installation of all features completed successfully and select **Next**.

   > **Note:** Wait for the installation to complete. This will take about three minutes.

1. On the **Restart servers** pane of the **Deploy hyperconverged infrastructure** wizard, select **Restart servers**.

   > **Note:** Disregard any warnings about issues with restarting the servers. Instead, switch to the lab VM and in the **Hyper-V Manager** console, connect to each of the four VMs to verify that all of them successfully restarted. Then switch back to the console session to the **WSLab-Management** VM.

1. On the **Restart Servers** pane of the **Deploy hyperconverged infrastructure** wizard, select **Next: Networking**.
1. On the **Verify network adapters** pane of the **Deploy hyperconverged infrastructure** wizard, ensure that both network adapters of each Hyper-V host display with the **Up** status, and then select **Next**.
1. On the **Select management adapters** pane of the **Deploy hyperconverged infrastructure** wizard, for each pair of network adapters on each of the four Hyper-V hosts, select the check box next to the network adapter labeled **Microsoft Hyper-V Network Adapter #2**, and then select **Next**.
1. On the **Edit adapters properties** pane of the **Deploy hyperconverged infrastructure** wizard, review the current settings, expand the **Advanced** section noting that you have the option of setting the packet size, and then select **Next**.
1. On the **Virtual switch** pane of the **Deploy hyperconverged infrastructure** wizard, expand the **Advanced** section, review the autogenerated switch settings, including the switch name (**ConvergedSwitch**), the load balancing algorithm set to **Hyper-V port (recommended)**, **Use VMMQ (recommended)** enabled, and the number of queue pairs set to **8**, and then select **Next (Clustering)**.

   > **Note:** Wait until the virtual switch is created. This will take about two minutes.

1. On the **Validate the cluster** pane of the **Deploy hyperconverged infrastructure** wizard, select **Validate**. If prompted to enable Credential Security Service Provider (CredSSP), select **Yes**.

   > **Note:** Wait until the cluster validation is completed. This will take about five minutes.

   > **Note:** Disregard a warning about the validation of the network configuration. It's expected.

1. On the **Validate the cluster** pane of the **Deploy hyperconverged infrastructure** wizard, select **Next**.
1. On the **Create the cluster** pane of the **Deploy hyperconverged infrastructure** wizard, in the **Cluster name** text box, enter **s2d-cluster**, expand the **Advanced** section, in the **IP addresses** section, select **Specify one or more static addresses**, enter **10.0.0.100** in the text box, select **Add**, and then select **Create cluster**.

   > **Note:** Wait until the cluster is created. This will take about two minutes.

1. On the **Create the cluster** pane, select **Next: Storage**.
1. On the **Verify drives** pane of the **Deploy hyperconverged infrastructure** wizard, verify that there are 32 drives in total with all reporting an **OK** status, and then select **Next**.
1. On the **Clean drives** pane of the **Deploy hyperconverged infrastructure** wizard, select **Next**.
1. On the **Validate storage** pane of the **Deploy hyperconverged infrastructure** wizard, select **Validate**.

   > **Note:** If prompted to enable **Credential Security Service Provider (CredSSP)**, select **Yes**.

1. On the **Validate** pane of the **Deploy hyperconverged infrastructure** wizard, ensure that all of the results were successful, and then select **Next**.
1. On the **Enable Storage Spaces Direct** pane of the **Deploy hyperconverged infrastructure** wizard, select **Enable**.
1. Once the process of enabling Storage Spaces Direct completes, select **Finish**, and then select **Go to connections list**.

### Task 5: Connect to a Storage Spaces Direct cluster by using Windows Admin Center

1. In the console session to the **WSLab-Management** VM, in the browser window displaying the Windows Admin Center interface, on the **All connections** page, select the **s2d-cluster** entry.
1. If prompted, on the **Specify your credentials** panel, select the **Use another account for this connection** option, then in the **Username** text box, enter **CORP\\LabAdmin** and in the **Password** text box, enter **LS1setup!**, select the **Use these credentials for all connections**, and then select **Continue**.

### Task 6: Review the installation of the Storage Spaces Direct cluster on the lab VMs

1. In the console session to the **WSLab-Management** VM, in the browser window displaying the Windows Admin Center interface, on the **s2d-cluster** page, in the **Storage** section, select **Volumes**, and then on the **Volumes** pane, select the **Inventory** tab.

1. Note that the cluster contains only the **ClusterPerformanceHistory** volume. Verify that the volume is listed with a status of **OK**.

1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, in the **Storage** section, select **Drives**, and on the **Drives** panel, select the **Inventory** tab.

1. Review the list of drives and verify that each of them is listed with a status of **OK**. Note that all drives are HDD capacity drives and are part of a single storage pool named **S2D on s2d-cluster**.

1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, in the **Networking** section, select **Virtual switches**, on the **Virtual switches** panel, note that each node is connected to an external switch named **ConvergedSwitch**, select one of the **ConvergedSwitch** entries, and then select **Settings**.

1. On the **Settings for ConvergedSwitch** panel, review the list of network adapters and the load balancing algorithm (set to **Hyper-V port**) and select **Close**.

1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, in the lower-left corner, select **Settings**.

1. On the **Settings** panel, select **Storage Spaces Direct**, and then review the cache settings.

1. In the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, on the **Settings** panel, in the **Cluster** section, select the following entries:

      - Access point. Verify that **Cluster name** is set to **S2D-Cluster**.
      - Node shutdown behavior. Verify that the setting **Move virtual machines on node shutdown** is enabled.
      - Cluster traffic encryption. Verify that **Core traffic** is set to **Sign**, and that **Storage traffic** is set to **Clear text**.
      - Virtual machine load balancing. Verify that **Balance virtual machines** is set to **Always** with **Aggressiveness** set to **Low**.
      - Witness. Note that **Witness type** is set to **None**.

1. In the console session to the **WSLab-Management** VM, switch to the **Administrator: Windows PowerShell ISE** window, and from the **script** pane, run the following command to set up a file share-based witness for the **S2D-Cluster** failover cluster:

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

1. In the console session to the **WSLab-Management** VM, switch to the browser window displaying the Windows Admin Center interface, on the `s2d-cluster.corp.contoso.com` page, on the **Settings** panel, in the **Cluster** section, select **Witness** entry, and then verify that the **Witness type** is set to **File share witness** with the **File share path** set to **\\\\DC\\S2D-ClusterWitness**.

1. In the console session to the **WSLab-Management** VM, switch to the **Server Manager** window, select **Tools**, and then in the **Tools** drop-down menu, select **Failover Cluster Manager**.

1. In the **Failover Cluster Manager** window, right-click or access the context menu for the **Failover Cluster Manager** node, in the context menu, select **Connect to cluster**, in the **Select Cluster** dialog box, in the **Cluster name** text box, enter `s2d-cluster.corp.contoso.com`, and then select **OK**.

1. In the **Failover Cluster Manager** window, in the **Storage** node tree, select **Disks**, and then review the list of virtual disks (volumes).

1. In the **Failover Cluster Manager** window, in the **Storage** node tree, select **Pools**, and then verify that it contains a single pool labeled **Cluster Pool 1**.

1. Select the **Cluster Pool 1** entry, and on the **Cluster Pool 1** pane, examine its properties by selecting the **Summary** tab, followed by the **Virtual Disks** and **Physical Disks** tabs. Note that the pool name is set to **S2D on s2d-cluster**.

### Task 7: Deprovision the lab resources

1. Switch to the lab VM, and in the **Administrator: Windows PowerShell ISE** window, open and run the **F:\\WSLab-master\\Scripts\\Cleanup.ps1** script to remove all VMs provisioned in this lab. When prompted, on the **console** pane, enter **Y**, and then select Enter.
1. Once the script completes, select any key.
1. In the **Administrator: Windows PowerShell ISE** window, close the tab displaying the **F:\\WSLab-master\\Scripts\\Cleanup.ps1** script.
