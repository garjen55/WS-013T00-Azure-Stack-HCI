---
lab:
    title: 'Lab C: Implementing SDN Access Control List by using Windows Admin Center'
    type: 'Answer Key'
    module: 'Module 4: Planning for and Implementing Azure Stack HCI Networking'
---
# Lab C answer key: Implementing SDN Access Control List by using Windows Admin Center

## Exercise 1: Implementing SDN Access Control List by using Windows Admin Center

### Task 1: Create an ACL

1. To create an access control list (ACL), Within the **SDNExpress2019-Management** VM, in the Windows Admin Center, on the ```sddc01.corp.contoso.com``` page, in the list of **Tools**, in the **Networking** section, select **Access control lists**.

1. On the **Access control lists** panel, on the **Inventory** tab, select **+ New**. In the **Access Control List** panel, in the **Name** text box, enter **acl-100**, select the **acl-100** link, and then select **Submit**.

1. On the **Access Control List > acl-100** panel, in the **Access Control Rule** section, select **+ New**.

1. In the **Access Control Rule** section, specify the following settings and then select **Submit**:

   The allow-all access rule settings are:

   - Name: **allow-all**
   - Priority: **1000**
   - Types: **Inbound**
   - Protocol: **All**
   - Source Address Prefix: **\***
   - Source Port Range: **\***
   - Destination Address Prefix: **\***
   - Destination Port Range: **\***
   - Action: **Allow**
   - Logging: **Enabled**

1. On the **Access Control List > acl-100** panel, in the **Access Control Rule** section, select **+ New**.

1. In the **Access Control Rule** section, specify the following deny-winrm-from-vnet-000-subnet-0 access rule settings:

   - Name: **deny-winrm-from-vnet-000-subnet-0**
   - Priority: **500**
   - Types: **Inbound**
   - Protocol: **TCP**
   - Source Address Prefix: **192.168.0.0/24**
   - Source Port Range: **\***
   - Destination Address Prefix: **\***
   - Destination Port Range: **5985,5986**
   - Action: **Deny**
   - Logging: **Enabled**

1. Select **Submit** and verify that the rule was created successfully.

### Task 2: Assign the ACL to a subnet

1. Within the console session to the **SDNExpress2019-Management** VM, start Windows PowerShell ISE as Administrator. In the Windows PowerShell Integrated Scripting Environment (ISE) window, in the **script** pane, open a new tab, paste the following script, and run it to list the properties of the virtual networks you created earlier in this exercise:

   ```powershell
   Import-Module NetworkController
   $uri = 'https://NCCLUSTER.corp.contoso.com'
   Get-NetworkControllerVirtualNetwork -ConnectionUri $uri
   ```

1. In the Windows PowerShell ISE window, in the **script** pane, open a new tab, paste the following script, and run it to assign the access control list you created in the previous task to the first subnet (**subnet-0**) of the virtual network **vnet-100**:

   ```powershell
   $vnet2 = Get-NetworkControllerVirtualNetwork -ConnectionUri $uri -ResourceId 'vnet-100'
   $acl = Get-NetworkControllerAccessControlList -ConnectionUri $uri -resourceid 'acl-100'
   $vnet2.properties.subnets[0].Properties.AccessControlList = $acl
   $subnet = Get-NetworkControllerVirtualSubnet -VirtualNetworkId $vnet2.ResourceId -ConnectionUri $uri
   New-NetworkControllerVirtualSubnet -ConnectionUri $uri -Properties $vnet2.Properties.Subnets[0].Properties -ResourceId $subnet.ResourceId -VirtualNetworkId $vnet2.ResourceId -Force
   ```

   > **Note**: Verify that the access control list assignment was created successfully.

1. Switch to the browser window displaying the Windows Admin Center and refresh the browser page displaying the **Access control lists > acl-100** panel.
1. On the **Access Control List > acl-000** panel, in the **Related Tab** section, on the **Applied Virtual Subnets** tab, note the **subnet-0** entry of the **vnet-100** virtual network.

### Task 3: Verify functionality of the access control list

> **Note**: For the change to take effect, the Network Controller Host agent on the Hyper-V host where the virtual machines reside must process the corresponding policy. To expedite the change, you will restart the agent and the third virtual machine **vm-100**.

1. Within the console session to the **SDNExpress2019-Management** VM, switch to the browser window displaying the Windows Admin Center. On the upper left hand side of the page, select **Windows Admin Center**.

1. Select the the Hyper-V host **HV3** to which you deployed all three virtual machines.

1. On the page displaying the properties of the Hyper-V host, in the **Tools** list, select **Services**.

1. On the **Services** panel, locate and select the **NcHostAgent** entry, and then select **Restart** in the toolbar.

1. Within the console session to the **SDNExpress2019-Management** VM, switch to the **Hyper-V Manager** console displaying the Hyper-V host (**HV3**) to which you deployed all three virtual machines.

1. In the **Hyper-V Manager** window, select **vm-100** and, in the **Actions** pane, in the **vm-100** section, select **Shut down**.

1. Wait until **vm-100** is listed in the **Off** state and, in the **Actions** pane, in the **vm-100** section, select **Start**.

1. Switch to the **Virtual Machine Connection** to **vm-000**. In the **Virtual Machine Connection** window to **vm-000**, from the Command Prompt, run the following command to test connectivity over WinRM to **vm-100**:

   ```powershell
   powershell Test-NetConnection -ComputerName 192.168.100.100 -Port 5985 -InformationLevel Detailed
   ```

1. Review the output and verify that the connection failed.

   > **Note**: This output is expected because Windows Remote Management traffic from **subnet-0** of **vnet-000** to which **vm-000** is attached is blocked by the newly created access control list assigned to **subnet-0** of **vnet-100**, to which the **vm-100** is attached.

1. In the Virtual Machine Connection window to **vm-000**, from the Command Prompt, run the following command to test connectivity over Internet Control Message Protocol (ICMP) to **vm-100**:

   ```cmd
   ping 192.168.100.100
   ```

1. Review the output and verify that the connection was successful.

   > **Note**: This output is expected because all other types of traffic, except for Remote Management, from **vnet-000** (including **subnet-0** to which **vm-000** is attached) is allowed by the newly created access control list (ACL) assigned to **subnet-0** of **vnet-100**, to which the **vm-100** is attached.

1. Switch to the **Virtual Machine Connection** to **vm-001**. In the **Virtual Machine Connection** window to **vm-001**, in the **Action** menu, select the **Ctrl+Alt+Delete** entry. When prompted, enter **Pa55w.rd**, and select **Enter**.

1. In the **Virtual Machine Connection** window to **vm-001**, from the Command Prompt, run the following command to test connectivity over WinRM to **vm-100**:

   ```powershell
   powershell Test-NetConnection -ComputerName 192.168.100.100 -Port 5985 -InformationLevel Detailed
   ```

1. Review the output and verify that the connection was successful.

    > **Note**: This output is expected because Windows Remote Management traffic to **subnet-0** of **vnet-100**, to which the **vm-100** is attached is blocked only from **subnet-0** of **vnet-000**, not from **subnet-1** to which **vm-001** is attached.

1. Switch to the **Virtual Machine Connection** window to **vm-100**. In the **Virtual Machine Connection** window to **vm-100**, in the **Action** menu, select the **Ctrl+Alt+Delete** entry. When prompted, enter **Pa55w.rd**, and then select **Enter**.

1. In the **Virtual Machine Connection** window to **vm-100**, from the Command Prompt, run the following command to test connectivity over WinRM to **vm-000**:

   ```powershell
   powershell Test-NetConnection -ComputerName 192.168.0.100 -Port 5985 -InformationLevel Detailed
   ```

1. Review the output and verify that the connection was successful.

   > **Note**: This output is expected because the access control rule blocking Windows Remote Management traffic applies only to inbound traffic from **subnet-0** of **vnet-000**, and does not apply to outbound traffic from **subnet-0** of **vnet-100**, to which the **vm-100** is attached.
