---
lab:
    title: 'Lab D: Implementing SDN Software Load Balancing with private virtual IP by using PowerShell'
    module: 'Module 4: Planning for and Implementing Azure Stack HCI Networking'
---
# Lab D: Implementing SDN Software Load Balancing with private virtual IP by using PowerShell

## Scenario

You need to configure virtual machines (VMs) on virtual networks that will serve load-balanced workloads accessible from within the datacenter hosting your Software-Defined Networking (SDN) infrastructure. In addition, you need to ensure that you will be able to configure VMs on virtual networks to connect to the internet and to accept inbound connectivity from your datacenter servers. Rather than relying on third-party load balancers, you intend to use SDN software load balancer for this purpose.

## Objectives

After completing this lab, you'll be able to implement SDN Software Load Balancing by using Windows Admin Center and Windows PowerShell.

## Estimated time: 120 minutes

## Lab setup

To connect to the lab VM, follow the steps the lab hosting provider provides you.

## Exercise 1: Implementing SDN Software Load Balancing by using Windows Admin Center and Windows PowerShell

### Scenario
To meet the requirements of load-balanced workloads, you'll implement SDN Software Load Balancing by using Windows Admin Center and Windows PowerShell.

The main tasks for this exercise are as follows:

1. Review SDN virtual IP logical network configuration.
1. Install the Web Server role on VMs in a virtual network.
1. Configure an SLB private virtual IP address.
1. Verify the configuration of the SDN Software Load Balancing with private virtual IP address.
1. Configure outbound network address translation (NAT) by using SLB.
1. Verify the configuration of the SDN SLB outbound NAT.
1. Configure SLB-based traffic forwarding to a VM in a virtual network.
1. Verify connectivity to the VM in a virtual network via a public virtual IP address.

### Task 1: Review SDN virtual IP logical network configuration

1. Within the **SDNExpress2019-Management** VM, in the Windows Admin Center, from the `sddc01.corp.contoso.com` page, display the logical network inventory.
1. In the logical network inventory, display the settings for the **PrivateVIP** logical network.
1. Review the logical network settings and note that it contains a single subnet with the IP address space of **10.20.0.0/24**.
1. Review the configuration of the subnet and note that it currently has **1** allocated IP address. 

   > **Note**: You'll use an IP address from that range as a private virtual IP for connection to load balanced VMs on one of the virtual networks you created earlier in this lab.

1. In the logical network inventory, display the settings or the **PublicVIP** logical network.
1. Review the logical network settings and note that it contains a single subnet with the IP address space of **10.10.0.0/24**.
1. Review the configuration of the subnet and note that it currently has **1** allocated IP address. 

   > **Note**: You'll use an IP address from that range as a public virtual IP for outbound connectivity to the internet and for inbound connectivity from your datacenter.

### Task 2: Install the Web Server role on VMs in a virtual network

1. Switch to the **Virtual Machine Connection** to **vm-000**. From the Command Prompt, run the following command to install the Web Server role.

   ```powershell
   powershell Install-WindowsFeature -Name Web-Server
   ```

1. Use the procedure described in the previous step to install the Web server role on **vm-001**.

   > **Note**: Wait for both installations to complete.

1. Switch to the **Virtual Machine Connection** to **vm-100**. From the Command Prompt, run the following to verify that the installation was successful.

   ```powershell
   powershell Invoke-WebRequest -Uri 192.168.0.100 -UseBasicParsing
   powershell Invoke-WebRequest -Uri 192.168.1.100 -UseBasicParsing
   ```

   > **Note**: Verify that in both cases you are receiving a response including **HTTP/1.1 200 OK**.

### Task 3: Configure an SLB private virtual IP address

1. Switch to the console session to the **SDNExpress2019-Management** VM, from the Windows PowerShell Integrated Scripting Environment (ISE) window, run the following to create a load balancer object:

   ```powershell
   Import-Module NetworkController
   $uri = 'https://NCCLUSTER.corp.contoso.com'
   $LBResourceId = 'lb-000'
   $LoadBalancerProperties = New-Object Microsoft.Windows.NetworkController.LoadBalancerProperties
   ```

1. From the same Windows PowerShell ISE window, run the following script to configure the private virtual IP:

   ```powershell
   $vipIP = '10.20.0.100'
   $VIPLogicalNetwork = Get-NetworkControllerLogicalNetwork -ConnectionUri $uri -ResourceId 'PrivateVIP' -PassInnerException
   $FrontEndIPConfig = New-Object Microsoft.Windows.NetworkController.LoadBalancerFrontendIpConfiguration
   $FrontEndIPConfig.ResourceId = 'lb-000-fe-1'
   $FrontEndIPConfig.ResourceRef = "/loadBalancers/$LBResourceId/frontendIPConfigurations/$($FrontEndIPConfig.ResourceId)"

   $FrontEndIPConfig.Properties = New-Object Microsoft.Windows.NetworkController.LoadBalancerFrontendIpConfigurationProperties
   $FrontEndIPConfig.Properties.Subnet = New-Object Microsoft.Windows.NetworkController.Subnet
   $FrontEndIPConfig.Properties.Subnet.ResourceRef = $VIPLogicalNetwork.Properties.Subnets[0].ResourceRef
   $FrontEndIPConfig.Properties.PrivateIPAddress = $vipIP
   $FrontEndIPConfig.Properties.PrivateIPAllocationMethod = 'Static'
   $LoadBalancerProperties.FrontEndIPConfigurations += $FrontEndIPConfig
   ```

   > **Note**: The virtual IP belongs to the IP address range of the subnet of the logical network you identified in the first task of this exercise.

1. From the same Windows PowerShell ISE window, run the following to configure the back-end address pool, which contains the Dynamic IPs assigned to the load-balanced set of VMs:

   ```powershell
   $BackEndAddressPool = New-Object Microsoft.Windows.NetworkController.LoadBalancerBackendAddressPool
   $BackEndAddressPool.ResourceId = 'lb-000-be-1'
   $BackEndAddressPool.ResourceRef = "/loadBalancers/$LBResourceId/backendAddressPools/$($BackEndAddressPool.ResourceId)"
   $BackEndAddressPool.Properties = New-Object Microsoft.Windows.NetworkController.LoadBalancerBackendAddressPoolProperties
   $LoadBalancerProperties.backendAddressPools += $BackEndAddressPool
   ```

1. From the same Windows PowerShell ISE window, run the following script to define a health probe that the load balancer will use to determine the health state of the back-end pool members:

   ```powershell
   $Probe = New-Object Microsoft.Windows.NetworkController.LoadBalancerProbe
   $Probe.ResourceId = 'lb-000-hp-1'
   $Probe.ResourceRef = "/loadBalancers/$LBResourceId/Probes/$($Probe.ResourceId)"
   $Probe.properties = New-Object Microsoft.Windows.NetworkController.LoadBalancerProbeProperties
   $Probe.properties.Protocol = 'HTTP'
   $Probe.properties.Port = '80'
   $Probe.properties.RequestPath = '/'
   $Probe.properties.IntervalInSeconds = 5
   $Probe.properties.NumberOfProbes = 5
   $LoadBalancerProperties.Probes += $Probe
   ```

1. From the same Windows PowerShell ISE window, run the following script to define a load balancing rule to distribute traffic that arrives at the front-end IP to back-end IPs. In this case, the back-end pool receives Transmission Control Protocol (TCP) traffic to port **80**.

   ```powershell
   $Rule = New-Object Microsoft.Windows.NetworkController.LoadBalancingRule
   $Rule.ResourceId = 'web-000'
   $Rule.Properties = New-Object Microsoft.Windows.NetworkController.LoadBalancingRuleProperties
   $Rule.Properties.FrontEndIPConfigurations += $FrontEndIPConfig
   $Rule.Properties.backendaddresspool = $BackEndAddressPool
   $Rule.Properties.protocol = 'TCP'
   $Rule.Properties.FrontEndPort = 80
   $Rule.Properties.BackEndPort = 80
   $Rule.Properties.IdleTimeoutInMinutes = 4
   $Rule.Properties.Probe = $Probe
   $LoadBalancerProperties.loadbalancingRules += $Rule
   ```

1. From the same Windows PowerShell ISE window, run the following command to apply the change by adding the load balancer configuration to Network Controller:

   ```powershell
   $LoadBalancerResource = New-NetworkControllerLoadBalancer -ConnectionUri $URI -ResourceId $LBResourceId -Properties $LoadBalancerProperties -Force -PassInnerException
   ```

1. From the same Windows PowerShell ISE window, run the following commands to retrieve the reference to the load balancer object:

   ```powershell
   $lbresourceid = 'lb-000'
   $lb = Get-NetworkControllerLoadBalancer -ConnectionUri $uri -ResourceID $LBResourceId -PassInnerException
   ```

1. From the same Windows PowerShell ISE window, run the following command to identify values of the **ResourceId** property of network interfaces assigned to the VMs you created earlier in the lab:

   ```powershell
   Get-NetworkControllerNetworkInterface -ConnectionUri $uri | Select-Object ResourceId
   ```

1. From the same Windows PowerShell ISE window, run the following script to add the network interface of **vm-000** to the back-end pool of the load balancer **lb-000**:

   ```powershell
   $nic1 = Get-NetworkControllerNetworkInterface -ConnectionUri $uri -ResourceId 'vm-000_Net_Adapter_0' -PassInnerException
   $nic1.properties.IpConfigurations[0].properties.LoadBalancerBackendAddressPools += $lb.properties.backendaddresspools[0]
   New-NetworkControllerNetworkInterface -ConnectionUri $uri -ResourceId 'vm-000_Net_Adapter_0' -properties $nic1.properties -Force -PassInnerException
   ```

1. From the same Windows PowerShell ISE window, run the following script to add the network interface of **vm-001** to the back-end pool of the load balancer **lb-000**:

   ```powershell
   $nic2 = Get-NetworkControllerNetworkInterface -ConnectionUri $uri -ResourceId vm-001_Net_Adapter_0 -PassInnerException
   $nic2.properties.IpConfigurations[0].properties.LoadBalancerBackendAddressPools += $lb.properties.backendaddresspools[0]
   New-NetworkControllerNetworkInterface -ConnectionUri $uri -ResourceId 'vm-001_Net_Adapter_0' -properties $nic2.properties -Force -PassInnerException
   ```

### Task 4: Verify the configuration of the SDN Software Load Balancing with private virtual IP

1. Within the **SDNExpress2019-Management** VM, switch to the browser window displaying the **Logical subnet > 10.20.0.0_24** panel within the Windows Admin Center interface.
1. Refresh the browser page, review the updated configuration, and note that it currently has **2** allocated IP addresses.
1. In the console session to the **SDNExpress2019-Management** VM, open a new browser window.
1. In the browser window, navigate to `http://10.20.0.100` and verify that you can access the default Microsoft Internet Information Services (IIS) home page.
1. Within the console session to the **SDNExpress2019-Management** VM, switch to the Windows PowerShell ISE window, and run the following command from the **console** pane to identify the Border Gateway Patrol (BGP) router information, which is hosted on the **SDNExpress2019-DC** VM:

   ```powershell
   Invoke-Command -ComputerName DC -ScriptBlock {Get-BgpRouter}
   ```

   > **Note**: Note that the BGP peers include two Gateway VMs and the two multiplexer (MUX) VMs.

1. In the Windows PowerShell ISE window, and run the following from the **console** pane to identify the BGP route information, (with the router hosted on the **SDNExpress2019-DC** VM):

   ```powershell
   Invoke-Command -ComputerName DC -ScriptBlock {Get-BgpRouteInformation}
   ```

   > **Note**: Note that the output includes two routes to the private virtual IP you configured in this exercise (one per MUX) and that each route was learned from the corresponding MUX VM.

### Task 5: Configure outbound NAT by using SLB

1. In the console session to the **SDNExpress2019-Management** VM, from the Windows PowerShell ISE window, run the following to create a load balancer object:

   ```powershell
   Import-Module NetworkController
   $uri = 'https://NCCLUSTER.corp.contoso.com'
   $LBResourceId = 'lb-nat-outbound-100'
   $LoadBalancerProperties = New-Object Microsoft.Windows.NetworkController.LoadBalancerProperties
   ```

1. From the same Windows PowerShell ISE window, run the following to create the load balancer, its front-end IP, and the back-end pool:

   ```powershell
   $vipIP = '10.10.0.100'
   $vipLogicalNetwork = Get-NetworkControllerLogicalNetwork -ConnectionUri $uri -resourceid 'PublicVIP' -PassInnerException

   $FrontEndIPConfig = new-object Microsoft.Windows.NetworkController.LoadBalancerFrontendIpConfiguration
   $FrontEndIPConfig.ResourceId = 'fe-100'
   $FrontEndIPConfig.ResourceRef = "/loadBalancers/$LBResourceId/frontendIPConfigurations/$($FrontEndIPConfig.ResourceId)"
   $FrontEndIPConfig.Properties = new-object Microsoft.Windows.NetworkController.LoadBalancerFrontendIpConfigurationProperties
   $FrontEndIPConfig.Properties.Subnet = new-object Microsoft.Windows.NetworkController.Subnet
   $FrontEndIPConfig.Properties.Subnet.ResourceRef = $vipLogicalNetwork.Properties.Subnets[0].ResourceRef
   $FrontEndIPConfig.Properties.PrivateIPAddress = $vipIP
   $FrontEndIPConfig.Properties.PrivateIPAllocationMethod = 'Static'
   $LoadBalancerProperties.FrontEndIPConfigurations += $FrontEndIPConfig

   $BackEndAddressPool = new-object Microsoft.Windows.NetworkController.LoadBalancerBackendAddressPool
   $BackEndAddressPool.ResourceId = 'bepool-100'
   $BackEndAddressPool.ResourceRef = "/loadBalancers/$LBResourceId/backendAddressPools/$($BackEndAddressPool.ResourceId)"
   $BackEndAddressPool.Properties = new-object Microsoft.Windows.NetworkController.LoadBalancerBackendAddressPoolProperties

   $LoadBalancerProperties.backendAddressPools += $BackEndAddressPool
   ```

   > **Note**: The virtual IP belongs to the IP address range of the subnet of the logical network you identified in the first task of this exercise.

1. From the same Windows PowerShell ISE window, run the following to define the outbound NAT rule:

   ```powershell
   $OutboundNAT = new-object Microsoft.Windows.NetworkController.LoadBalancerOutboundNatRule
   $OutboundNAT.ResourceId = 'outbound-nat-100'

   $OutboundNAT.properties = new-object Microsoft.Windows.NetworkController.LoadBalancerOutboundNatRuleProperties
   $OutboundNAT.properties.frontendipconfigurations += $FrontEndIPConfig
   $OutboundNAT.properties.backendaddresspool = $BackEndAddressPool
   $OutboundNAT.properties.protocol = 'ALL'
   ```

1. From the same Windows PowerShell ISE window, run the following command to apply the change by adding the load balancer configuration to Network Controller:

   ```powershell
   $LoadBalancerResource = New-NetworkControllerLoadBalancer -ConnectionUri $uri -ResourceId $LBResourceId -Properties $LoadBalancerProperties -Force -PassInnerException
   ```

1. From the same Windows PowerShell ISE window, run the following command to retrieve the reference to the load balancer object:

   ```powershell
   $lbresourceid = 'lb-nat-outbound-100'
   $lb = Get-NetworkControllerLoadBalancer -ConnectionUri $uri -ResourceID $LBResourceId -PassInnerException
   ```

1. From the same Windows PowerShell ISE window, run the following command to identify values of the **ResourceId** properties of network interfaces assigned to the VMs you previously created in the lab:

   ```powershell
   Get-NetworkControllerNetworkInterface -ConnectionUri $uri | Select-Object ResourceId
   ```

1. From the same Windows PowerShell ISE window, run the following script to add the network interface of **vm-100** to the back-end pool of the load balancer **lb-nat-outbound-100**:

   ```powershell
   $nic1 = Get-NetworkControllerNetworkInterface -ConnectionUri $uri -ResourceId 'vm-100_Net_Adapter_0' -PassInnerException
   $nic1.properties.IpConfigurations[0].properties.LoadBalancerBackendAddressPools += $lb.properties.backendaddresspools[0]
   New-NetworkControllerNetworkInterface -ConnectionUri $uri -ResourceId 'vm-100_Net_Adapter_0' -properties $nic1.properties -Force -PassInnerException
   ```

### Task 6: Verify the configuration of the SDN Software Load Balancing outbound NAT

1. Within the console session to the **SDNExpress2019-Management** VM, switch to the browser window displaying within Windows Admin Center, and navigate to the **Logical subnet > 10.10.0.0_24** panel in the **PublicVIP** section.

1. Refresh the browser page, review the updated configuration, and note that it currently has **2** allocated IP addresses.

1. Switch to the lab VM and, from the console pane of the Administrator: Windows PowerShell ISE window, run the following to install the Web Server role:

   ```powershell 
   Install-WindowsFeature -Name Web-Server
   ```

   > **Note**: Wait for the installation to complete.

1. On the lab VM, from the console pane of the Administrator: Windows PowerShell ISE window, run the following to identify the local IP configuration:

   ```powershell 
   Get-NetIPConfiguration
   ```

   > **Note**: Review the output of the cmdlet you ran in the previous step and identify the IP address of its network interace which is **NOT** used for internal NAT. You will need it in this task.

1. On the lab VM, open a new browser window and navigate to the IP address you identified in the previous step and verify that you can access the default IIS home page.

   > **Note**: This is the default web site installed on the lab VM, accessible via its private IP address. Verify that the Windows Firewall on the lab VM is allowing inbound traffic on port 80 for all network profiles.

1. Record the IP address and switch back to the console session to the **SDNExpress2019-Management** VM.

1. Within the console session to the **SDNExpress2019-Management** VM, switch to the **Virtual Machine Connection** window to **vm-100**, and from the Command Prompt, run the following command to test connectivity to the public IP address you identified in the previous step. Replace the `[ip_address]` placeholder with the IP address you recorded in the previous step:

   ```powershell
   powershell Invoke-WebRequest -Uri [ip_address] -UseBasicParsing
   ```

   > **Note**: Verify that you receive a response with the **Status Code** of **200**.

1. In the Windows PowerShell ISE window, run the following from the **console** pane to identify the BGP route information (with the router hosted on the **SDNExpress2019-DC** VM):

   ```powershell
   Invoke-Command -ComputerName DC -ScriptBlock {Get-BgpRouteInformation}
   ```

   > **Note**: Note that the output includes two routes to the public virtual IP you configured in this exercise (one per MUX) and that each route was learned from the corresponding MUX VM.
   

### Task 7: Configure SLB-based traffic forwarding to a VM in a virtual network

1. Within the console session to the **SDNExpress2019-Management** VM, switch to the Windows PowerShell ISE window, and run the following commands to create a public IP address object referencing a public virtual IP:

   ```powershell
   $publicIPProperties = new-object Microsoft.Windows.NetworkController.PublicIpAddressProperties
   $publicIPProperties.ipaddress = '10.10.0.200'
   $publicIPProperties.PublicIPAllocationMethod = 'static'
   $publicIPProperties.IdleTimeoutInMinutes = 4
   $publicIP = New-NetworkControllerPublicIpAddress -ResourceId 'vm-100-pip' -Properties $publicIPProperties -ConnectionUri $uri -Force -PassInnerException
   ```

1. From the same Windows PowerShell ISE window, run the following commands to assign the newly created public IP address object to the network interface of the **vm-100** VM:

   ```powershell
   $nic = Get-NetworkControllerNetworkInterface  -connectionuri $uri -resourceid 'vm-100_Net_Adapter_0'
   $nic.properties.IpConfigurations[0].Properties.PublicIPAddress = $publicIP
   New-NetworkControllerNetworkInterface -ConnectionUri $uri -ResourceId $nic.ResourceId -Properties $nic.properties -PassInnerException -Force
   ```

### Task 8: Verify connectivity to the VM in a virtual network via a public virtual IP

1. In the Windows PowerShell ISE window, and run the following command from the **console** pane to identify the BGP route information, with the router hosted on the **SDNExpress2019-DC** VM:

   ```powershell
   Invoke-Command -ComputerName DC -ScriptBlock {Get-BgpRouteInformation}
   ```

   > **Note**: Note that the output includes two routes to the public virtual IP you configured in this exercise (one per MUX) and that each route was learned from the corresponding MUX VM.

   > **Note**: If the routes are not displayed yet, you might need to wait a few minutes.

1. Within the console session to the **SDNExpress2019-Management** VM, the Windows PowerShell ISE window, from the **console** pane, run the following to determine whether you have connectivity to the **vm-100** via the IP address allocated from the **PublicVIP** subnet.

   ```powershell
   Test-NetConnection -ComputerName 10.10.0.200 -Port 5985 -InformationLevel Detailed
   ```

1. Verify that the connection attempt was successful.
1. Within the console session to the **SDNExpress2019-Management** VM, in the Windows PowerShell ISE window, from the **console** pane, run the following to establish a PowerShell Remoting session to the **vm-100** VM via the IP address allocated from the **PublicVIP** subnet.

   ```powershell
   $username = 'Administrator'
   $password = ConvertTo-SecureString -String 'Pa55w.rd' -AsPlainText -Force
   $creds = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $username,$password
   Enter-PSSession -ComputerName 10.10.0.200 -Credential $creds
   ```

1. After the session is established, from the **console** pane in the Windows PowerShell ISE window, from the `[10.10.0.200]: PS C:\Users\Administrator\Documents>` prompt, run `ipconfig` and verify that the output displays the IP configuration of the **vm-100** VM, with the IP address of **192.168.100.100**.

### Task 9: Deprovision the lab resources

1. Within the Remote Desktop session to lab VM, start Windows PowerShell ISE as Administrator.

1. From the Windows PowerShell ISE window, run the **F:\\WSLab-master\\Scripts\\Cleanup.ps1** script to remove all VMs provisioned in this lab.

### Results

After completing this lab, you would have deployed SDN by using PowerShell, configured routing and management for the SDN lab environment, managed virtual networks by using Windows Admin Center and PowerShell, implemented SDN Access Control List by using Windows Admin Center, implemented SDN Software Load Balancing with private virtual IP by using PowerShell, and deprovisioned the lab environment.
