## Network > Load Balancer (DSR) > Console User Guide

<a id='manage-dsr-loadbalancers'></a>
## Load Balancer (DSR) Management

<a id='create-dsr-loadbalancers'></a>
### Creating a Load Balancer (DSR)
You can easily create a DSR load balancer in the NHN Cloud console by simply entering load balancer (DSR) configuration values. The load balancer (DSR) operates using the direct server return (DSR) method, providing high performance by sending server response traffic directly to the client without passing through the load balancer.

The load balancer (DSR) creation screen consists of three sections:

#### 1. Load Balancer (DSR) Basic Information Settings

Configure basic information for the load balancer (DSR). Required items are as follows:

* Name: Enter the name of the load balancer (DSR).
* Description: Describe the load balancer (DSR).
* VPC: Select the VPC to which the load balancer (DSR) will be connected.
* Subnet: Select the subnet where the load balancer (DSR) will belong. The load balancer (DSR) and member instances must be located in the same subnet.
* VIP (Virtual IP): VIP address to be assigned to the load balancer (DSR). You can choose between **Auto assign** or **Manual assign**.
  * Auto assign: Automatically assign one of the available IPs from the subnet to use as VIP.
  * Manual assign: Directly input the desired IP within the subnet's CIDR range to use as VIP.

!!! danger "Caution"
    Creation will fail if the manually specified VIP address is not within the subnet's CIDR range. Make sure to specify within the IP range of the corresponding subnet.

!!! tip "Note"
    Load balancer (DSR) operates at the TCP/UDP L4 level, and server response traffic does not pass through the load balancer. Therefore, unlike standard load balancers, L7 features (HTTP header-based routing, SSL Offloading, listener/member group concepts, etc.) are not provided.

#### 2. Health Check Settings

Configure health checks to periodically verify that member instances are operating normally.

* Health check protocol: Select the protocol to use for health checks. Choose one of **TCP / ICMP / HTTP**.
* Delay: Interval (in seconds) for sending health check requests.
* Timeout: Timeout duration (in seconds) for each health check request. If there is no response within this time, it is considered a failure.
* Max retries: Maximum number of retries before determining an instance as unhealthy. (1-10 times)

Configure additional items by protocol as follows:

**TCP**

* Health check port: Specify the port number for TCP connection attempts.

**ICMP**

* No separate port configuration is required. Connectivity is verified using ICMP Echo Request/Reply.

**HTTP**

* Health check port: Specify the port number to send HTTP requests to.
* HTTP path (URL): Enter the URL path for health checks. The default value is `/`.
* Expected HTTP response code: Enter the HTTP response code to be considered a normal response. The default value is `200`.

!!! danger "Caution"
    The delay time must be greater than or equal to the timeout. If the timeout is greater than the delay time, health checks may not function properly.

!!! tip "Note"
    TCP/HTTP health checks request to the DSR VIP as destination, so if the VIP is not configured on the member server's lo interface, those packets cannot be received and processed, causing health check failures and the member to be determined as `INACTIVE`. ICMP health checks request to the member's actual IP, so they only verify connectivity regardless of VIP configuration.

#### 3. Member Settings

Specify member instances to register when creating the load balancer (DSR). Member registration can also be done after creating the load balancer (DSR).

* Instance selection: Select instances (network interfaces) that belong to the same subnet as the load balancer (DSR). You can select one or more instances simultaneously to register as members.

!!! tip "Note"
    Load balancer (DSR) forwards to member instances while maintaining the destination port (VIP port) of the client request as-is. Therefore, unlike standard load balancers, when registering members, you do not specify service ports per member, and only select the network interface of the member instance. The application on the member server must bind to `0.0.0.0` or VIP to listen for the same port sent by the client.

!!! danger "Caution"
    For member instances to properly receive and respond to traffic coming to the VIP, the following configuration is required inside the server:

    - Add VIP as additional allowed addresses to the network interface (Console Network Interface menu)
    - Kernel parameter configuration (`arp_ignore=1`, `arp_announce=2`)
    - Add VIP to lo interface with `/32` subnet
    - Allow service port and health check traffic in Security Groups

    For detailed procedures, refer to the member server configuration guide in [Load Balancer (DSR) Overview](/Network/Load%20Balancer(DSR)/en/overview/).

!!! tip "Note"
    The initial status of newly registered members is `INACTIVE`. They automatically transition to `ACTIVE` status and receive traffic after passing health checks.

After entering all items, click the **Create Load Balancer** button to create the load balancer (DSR).

<a id='view-dsr-loadbalancers'></a>
### Viewing and Modifying Load Balancer (DSR)

#### Load Balancer (DSR) List

After completing load balancer (DSR) creation, you can check basic information of created load balancers (DSR) in the list screen. Items displayed in the list screen are as follows:

* Name: The name specified when creating the load balancer (DSR).
* VIP address: Private IP assigned to the load balancer (DSR). This IP can be accessed from within the VPC.
* Floating IP: Floating IP connected for external access.
* Network: VPC name and subnet CIDR where the load balancer (DSR) belongs.
* Member count: Number of member instances registered to the load balancer (DSR).
* Status: Creation/operation status of the load balancer (DSR).

!!! tip "Note"
    The status of load balancer (DSR) is determined as one of the following:

    | Status | Meaning |
    |--|--|
    | `ACTIVE` | Operating normally |
    | `BUILD` | Load balancer (DSR) being created |
    | `ERROR` | Error occurred. Please contact administrator. |

You can create additional load balancers (DSR) with the **+ Create DSR** button at the top, and delete them by selecting load balancers (DSR) with checkboxes from the list and clicking the **Delete** button.

#### Load Balancer (DSR) Detailed Information

When you select a load balancer (DSR) from the list, detailed information is displayed at the bottom of the screen. The detail screen is divided into three tabs: **Basic Information**, **Members**, and **Health Check**.

In the **Basic Information** tab, you can check the following information:

* Name, description
* Subnet and VIP address
* Floating IP connection information
* Status

#### Changing Name
To change the name of a load balancer (DSR), click the **Edit Name** icon in the detailed information, enter the new name, and click the **Confirm** button.

#### Changing Floating IP
You can connect or disconnect a floating IP to allow access to the load balancer (DSR) from external networks.

1. Click the **Change Floating IP** button in the load balancer (DSR) detailed information.
2. Select the floating IP to connect. To disconnect the floating IP, select **Disable**.
3. Click the **Confirm** button to apply the settings.

!!! tip "Note"
    Even if you disconnect the floating IP, access through VIP from within the VPC is not affected.

!!! tip "Note"
    The VPC, subnet, and VIP address connected to the load balancer (DSR) cannot be changed after creation. If changes are needed, you must delete the load balancer (DSR) and recreate it.

<a id='delete-dsr-loadbalancers'></a>
### Deleting Load Balancer (DSR)
In the load balancer (DSR) list screen, select the load balancer (DSR) you want to delete, click the **Delete** button, and click the **Confirm** button in the confirmation window to delete the corresponding load balancer (DSR).

!!! danger "Caution"
    When you delete a load balancer (DSR), all members registered to that DSR are also deleted. If a floating IP is connected, it is automatically released.

<a id='manage-dsr-members'></a>
## Member Management

Select the desired load balancer (DSR) from the load balancer (DSR) list and click the **Members** tab to display the member instance management screen.

### Member List

In the **Members** tab, you can check the list and status of member instances registered to the load balancer (DSR). Items displayed in the list are as follows:

* IP address: IP address of the member instance.
* Device model: Resource type that owns the network port registered as a member.
* Device information: Integrated display of identification information (instance name, port ID, etc.) of the network port registered as a member.
* Status: Current status of the member.

!!! tip "Note"
    Since load balancer (DSR) forwards to members while maintaining the destination port of client requests as-is, L4 service ports are not separately displayed in member items. The actual service port is the port requested by the client to the VIP, and the application on the member server must listen for that port.

!!! tip "Note"
    Member status is determined as one of the following:

    | Status | Meaning |
    |--|--|
    | `ACTIVE` | Health check successful, traffic distribution target |
    | `INACTIVE` | Health check failed or immediately after new registration, excluded from traffic distribution |
    | `ONLINE` | Member manually disabled |

<a id='add-dsr-members'></a>
### Adding Members
Click the **+ Add Member** button in the **Members** tab to display the add member modal.

1. Select **instances** to register as members from the list. You can select one or more instances simultaneously.
2. Click the **Confirm** button to register the selected instances as members.

!!! tip "Note"
    Unlike standard load balancers, load balancer (DSR) does not input L4 service ports during the member addition step. Since load balancer (DSR) forwards the destination port of client requests to members without conversion, per-member port mapping is unnecessary.

!!! danger "Caution"
    Note the following constraints when registering members:

    * Member instances must belong to the same subnet as the load balancer (DSR).
    * Only compute instances can be registered as members.
    * The same instance port cannot be registered multiple times to the same load balancer (DSR).
    * A single load balancer (DSR) can register up to 30 members by default.

!!! tip "Note"
    To properly receive traffic after member registration, you need to add VIP as additional allowed addresses to the network interface, configure ARP kernel parameters inside the member server, add VIP to lo interface, and set Security Groups rules. For detailed procedures, refer to the member server configuration guide in [Load Balancer (DSR) Overview](/Network/Load%20Balancer(DSR)/en/overview/).

<a id='delete-dsr-members'></a>
### Deleting Members
Select the member to delete from the list in the member tab, then click the **Delete** button. When the confirmation window appears, click the **Confirm** button to remove the corresponding member from the load balancer (DSR).

!!! tip "Note"
    Deleting a member from the load balancer (DSR) does not delete the instance itself. Conversely, if you delete an instance that is registered as a member, that member is automatically removed from the load balancer (DSR).

<a id='manage-dsr-health-monitor'></a>
## Health Check Management

In the **Health Check** tab of the load balancer (DSR) detail screen, you can check currently configured health check information and make changes.

<a id='view-dsr-health-monitor'></a>
### Viewing Health Check
In the **Health Check** tab, you can check the following information about the currently configured health check:

* Health check protocol: TCP / ICMP / HTTP
* Health check port: Target port for health checks when using TCP or HTTP protocol
* Delay: Health check request interval (seconds)
* Timeout: Health check timeout (seconds)
* Max retries: Number of retries until unhealthy determination
* HTTP path (URL) / Expected HTTP response code: Displayed only when using HTTP protocol

<a id='change-dsr-health-monitor'></a>
### Changing Health Check Settings
Click the **Change Settings** button in the **Health Check** tab to modify health check settings.

* Health check protocol: Choose from TCP / ICMP / HTTP
* Enter required items by protocol:
  * TCP: Health check port
  * ICMP: No additional items
  * HTTP: Health check port, HTTP path, expected HTTP response code
* Set delay time, timeout, and max retries.

After completing the configuration, click the **Confirm** button to apply the changes.

!!! danger "Caution"
    The delay time must be greater than or equal to the timeout. If the timeout is greater than the delay time, health checks may not function properly.

!!! tip "Note"
    Health check requests are sent from a dedicated health check IP automatically assigned to the same subnet as the load balancer (DSR). The Security Group of member instances must allow this traffic for health checks to function properly. For detailed information, refer to the Security Groups configuration section in [Load Balancer (DSR) Overview](/Network/Load%20Balancer(DSR)/en/overview/).

<a id='dsr-quota'></a>
## Quotas and Limitations

The following quotas and limitations apply when using load balancer (DSR):

| Item | Default Limit | Description |
|--|--|--|
| Load balancer (DSR) count per project | 10 | Number of load balancers (DSR) that can be created per project |
| Member count per load balancer (DSR) | 30 | Number of members that can be registered to one load balancer (DSR) |

!!! tip "Note"
    If you need to use more than the default quota, please contact customer support.