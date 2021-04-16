---
lab:
    title: 'Lab C: Implementing SDN Access Control List by using Windows Admin Center'
    module: 'Module 4: Planning for and Implementing Azure Stack HCI Networking'
---
# Lab C: Implementing SDN Access Control List by using Windows Admin Center

## Scenario

As part of the security requirements within the Software-Defined Networking (SDN) environment, you need to be able to filter specific types of traffic between virtual network subnets. You intend to use the SDN functionality for this purpose, rather than relying exclusively on the operating system to perform this task.

## Objectives

After completing this lab, you'll be able to implement SDN Access Control List by using Windows Admin Center.

## Estimated time: 30 minutes

## Lab setup

To connect to the lab VM, follow the steps the lab hosting provider provides you.

## Exercise 1: Implementing SDN Access Control List by using Windows Admin Center

### Scenario

In this exercise, you will create access control lists (ACLs) to filter specific types of traffic between virtual network subnets. You will use Windows Admin Center to create ACLs and to verify their functionality.

The main tasks for this exercise are as follows:

1. Create an ACL.
1. Assign the ACL to a subnet.
1. Verify functionality of the ACL.

### Task 1: Create an ACL

1. Within the **SDNExpress2019-Management** VM, in the Windows Admin Center, on the `sddc01.corp.contoso.com` page, in the list of **Tools**, in the **Networking** section, select **Access control lists**.
1. On the **Access control lists** panel, from the **Inventory** tab, create a new ACL named **acl-100** with the following rules:

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

   The deny-winrm-from-vnet-000-subnet-0 access rule settings are:

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

### Task 2: Assign the ACL to a subnet

1. Within the console session to the **SDNExpress2019-Management** VM, start Windows PowerShell Integrated Scripting Environment (ISE) as Administrator and run the following script to list the properties of the virtual networks you created earlier in this exercise:

   ```powershell
   Import-Module NetworkController
   $uri = 'https://NCCLUSTER.corp.contoso.com'
   Get-NetworkControllerVirtualNetwork -ConnectionUri $uri
   ```

1. From the Windows PowerShell ISE window, run the following script to assign the ACL you created in the previous task to the first subnet (**subnet-0**) of the virtual network **vnet-100**:

   ```powershell
   $vnet2 = Get-NetworkControllerVirtualNetwork -ConnectionUri $uri -ResourceId 'vnet-100'
   $acl = Get-NetworkControllerAccessControlList -ConnectionUri $uri -resourceid 'acl-100'
   $vnet2.properties.subnets[0].Properties.AccessControlList = $acl
   $subnet = Get-NetworkControllerVirtualSubnet -VirtualNetworkId $vnet2.ResourceId -ConnectionUri $uri
   New-NetworkControllerVirtualSubnet -ConnectionUri $uri -Properties $vnet2.Properties.Subnets[0].Properties -ResourceId $subnet.ResourceId -VirtualNetworkId $vnet2.ResourceId -Force
   ```

   > **Note**: Verify that the ACL assignment was created successfully.
   

1. Switch to the Windows Admin Center interface and refresh the browser page displaying the **Access control lists > acl-100** panel.
1. On the **Access Control List > acl-000** panel, in the **Related Tab** section, on the **Applied Virtual Subnets** tab, note the **subnet-0** entry of the **vnet-100** virtual network.

### Task 3: Verify functionality of the ACL

> **Note**: For the change to take effect, the Network Controller Host agent on the Microsoft Hyper-V host where the VMs reside must process the corresponding policy. To expedite the change, you will restart the agent and the third VM **vm-100**.

1. Within the console session to the **SDNExpress2019-Management** VM, switch to the browser window displaying the Windows Admin Center, and from the upper left hand side of the page, select **Windows Admin Center**.

1. Select the Hyper-V host (**HV3**) to which you deployed all three virtual machines.

1. On the page displaying the properties of the Hyper-V host, in the **Tools** list, select **Services**.

1. From the **Services** panel, restart the **NcHostAgent** service.

1. Within the console session to the **SDNExpress2019-Management** VM, switch to the **Hyper-V Manager** console displaying the Hyper-V host (**HV3**) to which you deployed all three VMs.

1. From the Hyper-V Manager console, restart the **vm-100** VM.

1. Switch to the **Virtual Machine Connection** to **vm-000**, from the Command Prompt, run the following command to test connectivity over WinRM to **vm-100**.

   ```powershell
   powershell Test-NetConnection -ComputerName 192.168.100.100 -Port 5985 -InformationLevel Detailed
   ```

1. Review the output and verify that the connection failed.

   > **Note**: This output is expected because the Windows Remote Management traffic from **subnet-0** of **vnet-000** to which **vm-000** is attached is blocked by the newly created ACL assigned to **subnet-0** of **vnet-100**, to which **vm-100** is attached.

1. In the **Virtual Machine Connection** window to **vm-000**, from the Command Prompt, run the following command to test connectivity over Internet Control Message Protocol (ICMP) to **vm-100**:

   ```cmd
   ping 192.168.100.100
   ```

1. Review the output and verify that the connection was successful.

   > **Note**: This output is expected because all other types of traffic (except for Remote Management) from **vnet-000** (including **subnet-0** to which **vm-000** is attached) are allowed by the newly created ACL assigned to **subnet-0** of **vnet-100**, to which the **vm-100** is attached.

1. Switch to the **Virtual Machine Connection** to **vm-001** and sign in. From the Command Prompt, run the following to test connectivity over WinRM to **vm-100**:

   ```powershell
   powershell Test-NetConnection -ComputerName 192.168.100.100 -Port 5985 -InformationLevel Detailed
   ```

1. Review the output and verify that the connection was successful.

    > **Note**: This output is expected because Windows Remote Management traffic to **subnet-0** of **vnet-100**, to which the **vm-100** is attached is blocked only from **subnet-0** of **vnet-000**, and not from **subnet-1** to which **vm-001** is attached.

1. Switch to the **Virtual Machine Connection** to **vm-001** and sign in. From the Command Prompt, run the following command to test connectivity over WinRM to **vm-000**:

   ```powershell
   powershell Test-NetConnection -ComputerName 192.168.0.100 -Port 5985 -InformationLevel Detailed
   ```

1. Review the output and verify that the connection was successful.

   > **Note**: This output is expected because the access control rule blocking Windows Remote Management traffic applies only to inbound traffic from **subnet-0** of **vnet-000**, not to outbound traffic from **subnet-0** of **vnet-100**, to which the **vm-100** is attached.

### Results

After completing this lab, you will have successfully implemented SDN Access Control List by using Windows Admin Center.
