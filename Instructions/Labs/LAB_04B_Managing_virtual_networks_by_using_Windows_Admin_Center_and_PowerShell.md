---
lab:
    title: 'Lab B: Managing virtual networks by using Windows Admin Center and PowerShell'
    module: 'Module 4: Planning for and Implementing Azure Stack HCI Networking'
---
# Lab B: Managing virtual networks by using Windows Admin Center and PowerShell

## Scenario

Now you're ready to start testing the functionality of your Software-Defined Networking (SDN) environment. You'll start by provisioning virtual networks, deploying a few virtual machines (VMs) into them, and validating their connectivity within the same virtual network and between virtual networks.

## Objectives

After completing this lab, you'll be able to manage virtual networks by using Windows Admin Center and PowerShell.

## Estimated time: 90 minutes

## Lab setup

To connect to the lab VM, follow the steps the lab hosting provider provides you.

## Exercise 1: Managing virtual networks by using Windows Admin Center and PowerShell

### Scenario

In this exercise, to provide connectivity between virtual networks, you will implement virtual network peering and use VMs to validate the peering configuration.

The main tasks for this exercise are as follows:

1. Connect to the SDN infrastructure by using Windows Admin Center.
1. Create virtual networks by using Windows Admin Center.
1. Create a storage volume on the hyperconverged cluster by using Windows Admin Center.
1. Create VMs by using Windows Admin Center.
1. Configure VMs.
1. Test network connectivity of VMs.
1. Connect virtual networks.
1. Test connectivity between peered virtual networks.

### Task 1: Connect to the SDN infrastructure by using Windows Admin Center

1. From the lab VM, use the copy and paste functionality of the Hyper-V console session to copy F:\Source\ChromeStandaloneSetup64.exe to the C:\Library directory in the SDNExpress2019-Management VM. Within the console session to the SDNExpress2019-Management virtual machine (VM), switch to the File Explorer window displaying the content of the C:\Library folder and use the **ChromeStandaloneSetup64.exe** to install the Chrome browser. Also, install the **WindowsAdminCenter.exe** from C:\Library using all default settings except use port **9999**.

1. In the Chrome browser, navigate to the Windows Admin Center at `https://management:9999` and, if prompted to authenticate, sign in as **CORP\\LabAdmin** with **LS1setup!** as the password.1. In the Chrome browser, navigate to the Windows Admin Center at `https://management:9999` and, if prompted to authenticate, sign in as **CORP\\LabAdmin** with **LS1setup!** as the password.

   > **Note**: This URL designates the local installation of **Windows Admin Center** on the management VM. If Chrome initially refuses the connection to `https://management:9999` then try to connect to the URL with IE and then try Chrome again.

1. In the **Windows Admin Center** interface, add a connection to the `sddc01.corp.contoso.com` cluster and the Network Controller REST URI at `https://NCCLUSTER.corp.contoso.com`. If prompted, authenticate by using the **CORP\\LabAdmin** and **LS1setup!** credentials.

### Task 2: Create virtual networks by using Windows Admin Center

1. In the console session to the **SDNExpress2019-Management** VM, in the **Windows Admin Center** interface, open a connection to `sddc01.corp.contoso.com`.

1. On the `sddc01.corp.contoso.com` page, in the list of **Tools**, in the **Networking** section, select **Virtual switches**, and then review virtual switches on the members of the SDN cluster `sddc01.corp.contoso.com`.

1. Review settings of the first **sdnSwitch** on `hv1.corp.contoso.com` and note that you have the option of changing the **Load balancing algorithm** from **Hyper-V port** to **Dynamic**. Do not make any changes. 

1. From the `sddc01.corp.contoso.com` page, navigate to the inventory of virtual networks.

1. From the **Inventory** tab, create the following virtual networks and subnets:

   *Table 1: vnet-000 settings*

   |Setting|Value|
   |---|---|
   |Name|vnet-000|
   |Address Prefix|192.168.0.0/20|

   *Table 2: vnet-000 subnet-0 settings*

   |Setting|Value|
   |---|---|
   |Name|subnet-0|
   |Address Prefix|192.168.0.0/24|

   *Table 3: vnet-000 subnet-1 settings*

   |Setting|Value|
   |---|---|
   |Name|subnet-1|
   |Address Prefix|192.168.1.0/24|

   *Table 4: vnet-100 settings*

   |Setting|Value|
   |---|---|
   |Name|vnet-100|
   |Address Prefix|192.168.96.0/20|

   *Table 5: vnet-100 subnet-0 settings*

   |Setting|Value|
   |---|---|
   |Name|subnet-0|
   |Address Prefix|192.168.100.0/24|

### Task 3: Create a storage volume on the hyperconverged cluster by using Windows Admin Center

1. Copy the **ISO** image from the **F:\\Source** folder on the lab VM to the **C:\\Library** folder on the **SDNExpress2019-Management** VM.

1. In the console session on the **SDNExpress2019-Management** VM, in the browser window displaying the Windows Admin Center interface, from the `sddc01.corp.contoso.com` page, navigate to the inventory of storage volumes.

1. From the inventory panel of storage volumes of the `sddc01.corp.contoso.com` cluster, create the following volume:

   *Table 6: VMStorage volume settings*

   |Setting|Value|
   |---|---|
   |Name|VMStorage|
   |Resiliency|Mirror-accelerated parity|
   |Parity percentage|90% parity, 10% mirror|
   |Size on hard disk drive (HDD)|512|
   |Size units|GB|

1. Upload the **ISO** file you copied to the **C:\\Library** folder into the **VMStorage** volume.

   > **Note**: Wait for the upload to complete. If the ISO file doesn't upload correctly, then from **SDNExpress2019-Management** virtual machine (VM), connect to **\\\\HV3\\c$** and paste the ISO file into the **\\\\HV3\\c$\\ClusterStorage\\VMStorage** folder.

### Task 4: Create VMs by using Windows Admin Center

1. In the console session on the **SDNExpress2019-Management** VM, in the Windows Admin Center interface, navigate to the inventory of VMs on the `sddc01.corp.contoso.com` cluster.
1. From the inventory panel of storage volumes of the `sddc01.corp.contoso.com` cluster, retain all other settings with their default values, and create VMs with the following settings:

   *Table 7: vm-000 settings*

   |Setting|Value|
   |---|---|
   |Name|vm-000|
   |Generation|Generation 2 (Recommended)|
   |Host|`hv3.corp.contoso.com`|
   |Path|C:\ClusterStorage\VMStorage|
   |Virtual processor count|2|
   |Enable nested virtualization|Disabled|
   |Startup memory (GB)|2|
   |Network adapter|sdnSwitch|
   |Connect to virtual network|Enabled|
   |Virtual network|vnet-000|
   |Virtual subnet|subnet-0 [192.168.0.0/24]|
   |IP Address|192.168.0.100|
   |Storage|Create an empty virtual hard disk|
   |Size (GB)|64|
   |Operating system|Install an operating system from an image file (.iso)|
   |Path|Path to the ISO file you copied to the C:\\ClusterStorage\\VMStorage volume in the previous task|

   *Table 8: vm-001 settings*

   |Setting|Value|
   |---|---|
   |Name|vm-001|
   |Generation|Generation 2 (Recommended)|
   |Host|`hv3.corp.contoso.com`|
   |Path|C:\ClusterStorage\VMStorage|
   |Virtual processor count|2|
   |Enable nested virtualization|Disabled|
   |Startup memory (GB)|2|
   |Network adapter|sdnSwitch|
   |Connect to virtual network|Enabled|
   |Virtual network|vnet-000|
   |Virtual subnet|subnet-1 [192.168.1.0/24]|
   |IP Address|192.168.1.100|
   |Storage|Create an empty virtual hard disk|
   |Size (GB)|64|
   |Operating system|Install an operating system from an image file (.iso)|
   |Path|Path to the ISO file you copied to the C:\\ClusterStorage\\VMStorage volume in the previous task|

   *Table 9: vm-100 settings*

   |Setting|Value|
   |---|---|
   |Name|vm-100|
   |Generation|Generation 2 (Recommended)|
   |Host|`hv3.corp.contoso.com`|
   |Path|C:\ClusterStorage\VMStorage|
   |Virtual processor count|2|
   |Enable nested virtualization|Disabled|
   |Startup memory (GB)|2|
   |Network adapter|sdnSwitch|
   |Connect to virtual network|Enabled|
   |Virtual network|vnet-100|
   |Virtual subnet|subnet-0 [192.168.100.0/24]|
   |IP Address|192.168.100.100|
   |Storage|Create an empty virtual hard disk|
   |Size (GB)|64|
   |Operating system|Install an operating system from an image file (.iso)|
   |Path|Path to the ISO file you copied to the C:\\ClusterStorage\\VMStorage volume in the previous task|

### Task 5: Configure VMs

1. In the console session on the **SDNExpress2019-Management** VM, in the Windows Admin Center interface, on the inventory panel of VMs on the `sddc01.corp.contoso.com` cluster, identify the Microsoft Hyper-V host to which you deployed the VMs in the previous task (**HV3**).
1. In the console session on the **SDNExpress2019-Management** VM, start **Hyper-V Manager**, and use it to add the Hyper-V host you identified in the previous step to the console.
1. From the **Hyper-V Manager** console, establish console connections to the three VMs you deployed in the previous task.
1. Use the console connection to start the installation of **Windows Server 2019 Datacenter Evaluation** on each VM.

   > **Note**: Wait for the operating system installation to complete on all three VMs.

1. Following the operating system installation, set the password of the built-in Administrator account to **Pa55w.rd** on each VM.
1. In the Windows Admin Center interface, on the inventory panel of VMs on the `sddc01.corp.contoso.com` cluster, shut down all three VMs.
1. In the Windows Admin Center interface, from the inventory panel of VMs on the `sddc01.corp.contoso.com` cluster, navigate to the network settings of each VM and configure their network adapters with the following settings:

   *Table 10: vm-000 network adapter settings*

   |Setting|Value|
   |---|---|
   |Connect to|Virtual Network|
   |Virtual network|vnet-000|
   |Virtual subnet|subnet-0 [192.168.0.0/24]|
   |IP Address|192.168.0.100|
   |MAC address type (Advanced)|Static|

   *Table 11: vm-001 network adapter settings*

   |Setting|Value|
   |---|---|
   |Connect to|Virtual Network|
   |Virtual network|vnet-000|
   |Virtual subnet|subnet-1 [192.168.1.0/24]|
   |IP Address|192.168.1.100|
   |MAC address type|Static|

   *Table 12: vm-100 network adapter settings*

   |Setting|Value|
   |---|---|
   |Connect to|Virtual Network|
   |Virtual network|vnet-100|
   |Virtual subnet|subnet-0 [192.168.100.0/24]|
   |IP Address|192.168.100.100|
   |MAC address type|Static|

   > **Note**: Network Controller automatically assigns the next available MAC address from its pool.

### Task 6: Test network connectivity of VMs

1. On the **SDNExpress2019-Management** VM, in the Windows Admin Center, from the  inventory panel of the `sddc01.corp.contoso.com` cluster, start the three VMs.

1. Use the Hyper-V Virtual Machine Connections to sign in to the three VMs you previously deployed in this exercise, and from the Command Prompt, disable Windows Defender Firewall by running the following command:

   ```powershell
   powershell Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
   ```

1. Switch back to Virtual Machine Connection to **vm-000**, and from the Command Prompt, run the following to test connectivity over WinRM to **vm-001**:

   ```powershell
   powershell Test-NetConnection -ComputerName 192.168.1.100 -Port 5985 -InformationLevel Detailed
   ```

1. Review the output and verify that the connection was successful.

   > **Note**: This output is expected because Remote Management is enabled by default and **vm-000** and **vm-001** are on the same virtual network. The fact that they are not two different subnets does not have any significance in this case.

1. From the Command Prompt, run the following command to test connectivity over WinRM to **vm-100**:

   ```powershell
   powershell Test-NetConnection -ComputerName 192.168.100.100 -Port 5985 -InformationLevel Detailed
   ```

1. Review the output and verify that the connection failed.

   > **Note**: This output is expected because although Remote Management is enabled by default, **vm-000** and **vm-100** are on different virtual networks. These virtual networks are not connected to each other at this point.

### Task 7: Connect virtual networks

1. On the **SDNExpress2019-Management** VM, in the Windows Admin Center, navigate to the virtual network inventory panel of the `sddc01.corp.contoso.com` cluster. From that panel, display the settings of the **vnet-000** virtual network.

1. From the **vnet-000 Settings** panel, create a new peering with the following settings:

   *Table 13: vnet-000-to-vnet-100 peering settings*

   |Setting|Value|
   |---|---|
   |Name|vnet-000-to-vnet-100|
   |Virtual networks|vnet-100|
   |Allow Virtual network access from 'vnet-000' to remote virtual network|Enabled|
   |Allow forwarded traffic from 'vnet-000' to remote virtual network|Enabled|
   |Allow Gateway Transit|Disabled|

1. On the **vnet-000 Settings Peerings** panel, verify that the new peering is listed as **Connected** or **Initiated** in the **Peering Status** column.

1. In the Windows Admin Center, navigate to the virtual network inventory panel of the `sddc01.corp.contoso.com` cluster. From that panel, display the settings of the **vnet-100** virtual network.

1. From the **vnet-100 Settings** panel, create a new peering with the following settings:

   *Table 14: vnet-100-to-vnet-000 peering settings*

   |Setting|Value|
   |---|---|
   |Name|vnet-100-to-vnet-000|
   |Virtual networks|vnet-000|
   |Allow Virtual network access from 'vnet-100' to remote virtual network|Enabled|
   |Allow forwarded traffic from 'vnet-100' to remote virtual network|Enabled|
   |Allow Gateway Transit|Disabled|

1. Switch back to the **vnet-100 Settings** **Peerings** panel, and then verify that the new peering is listed as **Connected** in the **Peering Status** column.

### Task 8: Test connectivity between peered virtual networks

> **Note**: For the change to take effect, the Network Controller Host agent on the Hyper-V host where the VMs reside must process the corresponding policy. To expedite the change, you will restart the agent and each of the VMs.

1. Within the Remote Desktop session to the **SDNExpress2019-Management** VM, switch to the browser window displaying the Windows Admin Center and on the upper left hand side of the page select **Windows Admin Center**.

1. Select the Hyper-V host (**HV3**) to which you deployed all three VMs.

1. On the page displaying the properties of the Hyper-V host, in the **Tools** list, select **Services**.

1. From the **Services** panel, restart the **NcHostAgent** service.

1. Within the console session to the **SDNExpress2019-Management** VM, switch to the **Hyper-V Manager** console displaying the Hyper-V host (**HV3**) to which you deployed all three VMs.

1. From the Hyper-V Manager console, restart the **vm-000**, **vm-001**, and **vm-100** VMs.

1. Switch to the **Virtual Machine Connection** to **vm-000** and sign in. From the Command Prompt, run the following to test connectivity over WinRM to **vm-100**:

   ```powershell
   powershell Test-NetConnection -ComputerName 192.168.100.100 -Port 5985 -InformationLevel Detailed
   ```
   
1. Review the output and verify that the connection was successful.

   > **Note**: This output is expected because Remote Management is by default enabled and, at this point, while **vm-000** and **vm-100** are on different virtual networks, there are peering connections between them.

   > **Note**: If the connection fails, wait a few minutes, and try again.

### Results

After completing this lab, you will have successfully managed virtual networks by using Windows Admin Center and PowerShell.
