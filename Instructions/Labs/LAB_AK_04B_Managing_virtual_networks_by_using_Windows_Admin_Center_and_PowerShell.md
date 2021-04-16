---
lab:
    title: 'Lab B: Managing virtual networks by using Windows Admin Center and PowerShell'
    type: 'Answer Key'
    module: 'Module 4: Planning for and Implementing Azure Stack HCI Networking'
---
# Lab B answer key: Managing virtual networks by using Windows Admin Center and PowerShell

## Exercise 1: Managing virtual networks by using Windows Admin Center and PowerShell

### Task 1: Connect to the SDN infrastructure by using Windows Admin Center

1. Within the console session to the **SDNExpress2019-Management** virtual machine (VM), switch to the File Explorer window displaying the content of the **C:\\Library** folder and use the **chrome_installer.exe** to install the Chrome browser.

1. In the Chrome browser, navigate to `https://management:9999`. If prompted to authenticate, sign in as **CORP\\LabAdmin** with **LS1setup!** as the password.

   > **Note**: This URL designates the local installation of **Windows Admin Center** on the management VM.

1. In the **Windows Admin Center** interface, on the **All connections** page, select **+ Add**. On the **Add resources** panel, select **Add** in the **Windows Server cluster** tile, and in the **Cluster name** text box, enter `sddc01.corp.contoso.com`. If prompted, select the **Use another account for this connection**, in the **Username** text box, enter **CORP\LabAdmin**, and in the **Password** text box, enter **LS1setup!**. Select **Manage Software-Defined Networking (if set up)**. In the **Specify the Network Controller REST URI** box, enter `https://NCCLUSTER.corp.contoso.com`. If prompted, select **Connect with account**, select **Validate**, select **Install-RSAT-NetworkController**, select **Validate** again, and then select **Add**.

   > **Note**: If you are presented with the **Access was denied** message, select the option **Use another account for this connection**, and then select **OK**.

### Task 2: Create virtual networks by using Windows Admin Center

1. In the console session to the **SDNExpress2019-Management** VM, in the **Windows Admin Center** interface, on the **All connections** page, select `sddc01.corp.contoso.com`.
1. On the `sddc01.corp.contoso.com` page, in the list of **Tools**, in the **Networking** section, select **Virtual switches** and review virtual switches on the members of the Software-Defined Network (SDN) cluster `sddc01.corp.contoso.com`.
1. Select the first **sdnSwitch** on `hv1.corp.contoso.com`, select **More**, and, in the drop-down list, select **Settings**.
1. Review general settings of **sdnSwitch** and note that you have the option of changing the **Load balancing algorithm** from **Hyper-V port** to **Dynamic**.
1. Select **Close** without making any changes.
1. On the `sddc01.corp.contoso.com` page, in the list of **Tools**, in the **Networking** section, select **Virtual networks** and, on the **Virtual networks** panel, select **Inventory**.
1. On the **Inventory** tab, select **+ New** and, on the **Virtual network** panel, specify the following settings:

   *Table 1: vnet-000 settings*

   |Setting|Value|
   |---|---|
   |Name|vnet-000|
   |Address Prefix|192.168.0.0/20|

1. On the **Virtual network** panel, in the **Subnets** section, select **+ Add** and, on the **Subnets** panel, specify the following settings:

   *Table 2: vnet-000 subnet-0 settings*

   |Setting|Value|
   |---|---|
   |Name|subnet-0|
   |Address Prefix|192.168.0.0/24|

1. Select **Submit**. Switch to the **Virtual network** panel, and in the **Subnets** section, select **+ Add** again.
1. On the **Subnets** panel, specify the following settings, and then select **Submit**:

   *Table 3: vnet-000 subnet-1 settings*

   |Setting|Value|
   |---|---|
   |Name|subnet-1|
   |Address Prefix|192.168.1.0/24|

1. On the **Virtual network** panel, select **Submit**.
1. Verify that **vnet-000** with two subnets was created successfully.
1. Back on the **Virtual networks** panel, select **+ New**.
1. Create another virtual network with the following settings:

   *Table 4: vnet-100 settings*

   |Setting|Value|
   |---|---|
   |Name|vnet-100|
   |Address Prefix|192.168.96.0/20|

1. Add to the virtual network **vnet-100** a single subnet with the following settings:

   *Table 5: vnet-100 subnet-0 settings*

   |Setting|Value|
   |---|---|
   |Name|subnet-0|
   |Address Prefix|192.168.100.0/24|

1. On the **Virtual network** panel, select **Submit**.
1. Verify that **vnet-100** with one subnet was created successfully.

### Task 3: Create a storage volume on the hyperconverged cluster by using Windows Admin Center

1. Switch back to the lab VM and use the copy and paste functionality of the **Hyper-V** console session to copy the **ISO** image from the **F:\\Source** folder to the **C:\\Library** folder on the **SDNExpress2019-Management** VM.
1. In the browser window displaying the Windows Admin Center interface, on the `sddc01.corp.contoso.com` page, in the list of **Tools**, in the **Storage** section, select **Volumes**, and then select the **Inventory** tab.
1. On the **Inventory** tab of the **Volumes** panel, select **+ Create**.
1. On the **Create volume** panel, specify the following settings, and then select **Create**:

   *Table 6: VMStorage volume settings*

   |Setting|Value|
   |---|---|
   |Name|VMStorage|
   |Resiliency|Mirror-accelerated parity|
   |Parity percentage|90% parity, 10% mirror|
   |Size on hard disk drive (HDD)|512|
   |Size units|GB|

1. On the **Create volume** panel, select the **Refresh** icon and ensure that the new volume appears in the **Inventory** listing.
1. Select the **VMStorage** entry, and on the **Volumes > Volume VMStorage** panel, select **Open**.

   > **Note**: This will automatically redirect you to the **Files** panel, which displays the content of the **VMStorage** volume.

1. On the **Files** panel, ensure that the **VMStorage** folder is selected, and in the toolbar, select **Upload**.
1. On the **Upload** panel, select **Select files**, in the **Open** dialog box, navigate to the **C:\\Library** folder, select the ISO file, and select **Open**.
1. Back on the **Upload** panel, select **Submit**.

   > **Note**: Wait for the upload to complete. If the ISO file doesn't upload correctly, then from **SDNExpress2019-Management** virtual machine (VM), connect to **\\\\HV3\\c$** and paste the ISO file into the **\\\\HV3\\c$\\ClusterStorage\\VMStorage** folder.

### Task 4: Create virtual machines by using Windows Admin Center

1. In the browser window displaying the Windows Admin Center interface, navigate back to the `sddc01.corp.contoso.com` page. In the **Tools** list, in the **Compute** section, select **Virtual machines**, and then select the **Inventory** tab.
1. On the **Inventory** tab, select **+ New**.
1. On the **New virtual machine** panel, retain the default values for all other settings, specify the following settings, and then select **Create**:

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

1. Repeat the previous step to create an additional virtual machine with the following settings:

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

1. Repeat the previous step to create an additional virtual machine with the following settings:

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

### Task 5: Configure virtual machines

1. In the browser window displaying the Windows Admin Center interface, on the **New virtual machine** panel, on the **Inventory** tab, select **vm-000**.

1. On the **Virtual machines > vm-000** panel, identify the **Host** where the VM is located.

1. Within the console session to the **SDNExpress2019-Management** VM, switch to the Server Manager window, select the **Tools** menu, and start **Hyper-V Manager**.

1. In the **Hyper-V Manager** window, right-click or access the context menu for the **Hyper-V Manager** node, and then select **Connect to Server**.

1. In the **Select Computer** dialog box, in the **Another computer** text box, enter the name of the Hyper-V host you identified earlier in this task, and select **OK**.

1. On the **tree** pane of the **Hyper-V Manager** console, select the newly added host. On the **details** pane, right-click or access the context menu for the entry representing the **vm-000** virtual machine, and then select **Connect**.

1. Within the **Virtual Machine Connection** window, select the **Start** button, and when prompted, select any key to initiate the boot process.

1. On the **Windows Server 2019** page, in the **Windows Setup** window, accept the default settings, select **Next**, and then select **Install now**.

1. On the **Select the operating system you want to install** page, select **Windows Server 2019 Datacenter Evaluation**, and then select **Next**.

1. On the **Applicable notices and license terms** page, select the **I accept the license terms** check box, and then select **Next**.

1. On the **Which type of installation do you want?** page, select the **Custom: Install Windows only (advanced)** option.

1. On the **where do you want to install Windows?** page, accept the default settings, and then select **Next**.

1. Switch back to the **Hyper-V Manager** console. In the **details** pane, right-click or access the context menu for the entry representing the **vm-001** virtual machine, and then select **Connect**.

1. Repeat the same sequence of steps you applied to **vm-000** to start installation of **Windows Server 2019 Datacenter Evaluation** on **vm-001** and **vm-100**.

   > **Note**: Wait for the operating installation to complete on all three VMs.

1. Switch back to the **Virtual Machine Connection** window to **vm-000**. When prompted to change the password of the built-in Administrator account, select the **Enter** key, at the **New password** prompt, enter **Pa55w.rd**, select the **Tab** key, enter **Password** again, and then select the **Enter** key.

1. Repeat the previous step to set the password of the built-in Administrator account to **Pa55w.rd** on **vm-001** and **vm-100**.

1. In the browser window, navigate back to the `sddc01.corp.contoso.com` blade, in the list of **Tools**, select **Virtual machines**, and select the **Inventory** tab.

1. On the **Inventory** tab, select the check box next to each virtual machine entry, select **More**, and in the drop-down menu, select **Shut down**, and then select **Yes**.

1. On the **Inventory** tab, select the check next to **vm-000**, select **Settings**, and on the **Settings for vm000** panel, select **Networks**.

1. On the **Networks** panel, specify the following settings:

   *Table 10: vm-000 network adapter settings*

   |Setting|Value|
   |---|---|
   |Connect to|Virtual network|
   |Virtual network|vnet-000|
   |Virtual subnet|subnet-0 [192.168.0.0/24]|
   |IP Address|192.168.0.100|

1. On the **Settings for vm-000** panel, select **Advanced**, on the **Advanced network adapter settings for vm-000**, in the **MAC address type** section, select the **Static** option, and then select **Save**.

   > **Note**: Network Controller automatically assigns the next available MAC address from its pool.

1. Switch to the **Settings for vm-000** panel, and then select **Save network settings**.

1. Repeat the same sequence of steps to assign the following IP configuration to the **vm-001** virtual machine:

   *Table 11: vm-001 network adapter settings*

   |Setting|Value|
   |---|---|
   |Connect to|Virtual network|
   |Virtual network|vnet-000|
   |Virtual subnet|subnet-1 [192.168.1.0/24]|
   |IP Address|192.168.1.100|
   |MAC address type|Static|

1. Repeat the same sequence of steps to assign the following IP configuration to the **vm-100** virtual machine:

   *Table 12: vm-100 network adapter settings*

   |Setting|Value|
   |---|---|
   |Connect to|Virtual network|
   |Virtual network|vnet-100|
   |Virtual subnet|subnet-0 [192.168.100.0/24]|
   |IP Address|192.168.100.100|
   |MAC address type|Static|

### Task 6: Test network connectivity of virtual machines

1. On the **SDNExpress2019-Management** VM, in the Windows Admin Center, navigate to the **Inventory** tab of the **Virtual machines** panel of `sddc01.corp.contoso.com` page, select the check boxes next to all three VMs you configured in this task, select **More**, and then in the drop-down menu, select **Start**.

1. Switch to the **Virtual Machine Connection** to **vm-000**, in the **Virtual Machine Connection** window to **vm-000**, in the **Action** menu, select the **Ctrl+Alt+Delete** entry. When prompted, enter **Pa55w.rd**, and then select **Enter**.

1. In the **Virtual Machine Connection** window to **vm-000**, at the Command Prompt, run the following command to disable Windows Defender Firewall:

   ```powershell
   powershell Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
   ```

1. Repeat the previous two steps to disable Windows Defender Firewall on **vm-001** and **vm-100**.

1. Switch back to the **Virtual Machine Connection** window to **vm-000**. From the Command Prompt, run the following command to test connectivity over WinRM to **vm-001**:

   ```powershell
   powershell Test-NetConnection -ComputerName 192.168.1.100 -Port 5985 -InformationLevel Detailed
   ```

1. Review the output and verify that the connection was successful.

   > **Note**: This output is expected because Remote Management is by default enabled and **vm-000** and **vm-001** are on the same virtual network. The fact that they are not two different subnets does not have any significance in this case.

1. From the Command Prompt, run the following command to test connectivity over WinRM to **vm-100**:

   ```powershell
   powershell Test-NetConnection -ComputerName 192.168.100.100 -Port 5985 -InformationLevel Detailed
   ```

1. Review the output and verify that the connection failed.

   > **Note**: This output is expected because while Remote Management is enabled by default, **vm-000** and **vm-100** are on different virtual networks. At this point, these virtual networks are not connected to each other.

### Task 7: Connect virtual networks

1. Switch back to the **SDNExpress2019-Management** VM. In Windows Admin Center, navigate to the **Inventory** tab of the **Virtual networks** panel of the `sddc01.corp.contoso.com` page.

1. On the **Inventory** tab of the **Virtual networks** panel, select the **vnet-000** entry, and then select **Settings**.

1. On the **vnet-000 Settings** panel, select **Peerings**, and on the **Peerings** panel, select **+ New**

1. On the **New Peering** panel, specify the following settings, and then select **Submit**:

   *Table 13: vnet-000-to-vnet-100 peering settings*

   |Setting|Value|
   |---|---|
   |Name|vnet-000-to-vnet-100|
   |Virtual networks|vnet-100|
   |Allow Virtual network access from "vnet-000" to remote virtual network|Enabled|
   |Allow forwarded traffic from "vnet-000" to remote virtual network|Enabled|
   |Allow Gateway Transit|Disabled|

1. Switch to the **vnet-000 Settings** **Peerings** panel, verify that the new peering is listed as **Initiated** or **Connected** in the **Peering Status** column.

1. Navigate back to the **Inventory** tab of the **Virtual networks** panel, select the **vnet-100** entry, and then select **Settings**.

1. On the **vnet-100 Settings** panel, select **Peerings**, and on the **Peerings** panel, select **+ New**.

1. On the **New Peering** panel, specify the following settings and select **Submit**:

   *Table 14: vnet-100-to-vnet-000 peering settings*

   |Setting|Value|
   |---|---|
   |Name|vnet-100-to-vnet-000|
   |Virtual networks|vnet-000|
   |Allow Virtual network access from 'vnet-100' to remote virtual network|Enabled|
   |Allow forwarded traffic from 'vnet-100' to remote virtual network|Enabled|
   |Allow Gateway Transit|Disabled|

1. Switch to the **vnet-100 Settings** **Peerings** panel, and then verify that the new peering is listed as **Connected** in the **Peering Status** column.

### Task 8: Test connectivity between peered virtual networks

> **Note**: For the change to take effect, the Network Controller Host agent on the Hyper-V host where the virtual machines reside must process the corresponding policy. To expedite the change, you will restart the agent and each of the virtual machines.

1. Within the console session to the **SDNExpress2019-Management** VM, switch to the browser window displaying the Windows Admin Center, and on the upper left side of the page select **Windows Admin Center**.

1. Select the the Hyper-V host **HV3** to which you deployed all three VMs.

1. On the page displaying the properties of the Hyper-V host, in the **Tools** list, select **Services**.

1. On the **Services** panel, locate and select the **NcHostAgent** entry, and then select **Restart** in the toolbar.

1. Within the console session to the **SDNExpress2019-Management** VM, switch to the **Hyper-V Manager** console displaying the Hyper-V host (**HV3**) to which you deployed all three VMs.

1. In the **Hyper-V Manager** window, select **vm-000**, **vm-001**, and **vm-100**. In the **Actions** pane, in the **Selected Virtual Machines** section, select **Shut down** twice.

1. Wait until all VMs are listed in the **Off** state. In the **Actions** pane, in the **Selected Virtual Machines** section, select **Start**.

1. Switch to the **Virtual Machine Connection** to **vm-000**.

1. In the **Virtual Machine Connection** to **vm-000** window, in the **Action** menu, select the **Ctrl+Alt+Delete** entry. When prompted, enter **Pa55w.rd**, and then select **Enter**.

1. In the **Virtual Machine Connection** to **vm-000** window, from the Command Prompt, run the following to test connectivity over WinRM to **vm-100**:

   ```powershell
   powershell Test-NetConnection -ComputerName 192.168.100.100 -Port 5985 -InformationLevel Detailed
   ```

1. Review the output and verify that the connection was successful.

   > **Note**: This output is expected because Remote Management is enabled by default. At this point, although **vm-000** and **vm-100** are on different virtual networks, there are peering connections between them.

   > **Note**: If the connection fails, wait a few minutes and try again.
