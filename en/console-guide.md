## Network > Load Balancer (DSR) > Console User Guide

<a id='manage-dsr-loadbalancers'></a>
## Manage Load Balancer (DSR)

<a id='create-dsr-loadbalancers'></a>
### Create Load Balancer (DSR)
You can easily create a DSR load balancer by simply entering the configuration settings in the NHN Cloud console. The load balancer (DSR) operates using the direct server return (DSR) method, where server response traffic is sent directly to the client without passing through the load balancer, providing high performance.

The Load Balancer (DSR) creation screen consists of three sections:

#### 1. Configure Load Balancer (DSR) Basic Information

Set the basic information for the load balancer (DSR). The required items are as follows:

* Name: Enter the name of the load balancer (DSR).
* Description: Provide a description of the load balancer (DSR).
* VPC: Select the VPC to which the load balancer (DSR) will be connected.
* Subnet: Select the subnet to which the load balancer (DSR) will belong. The load balancer (DSR) and member instances must be located in the same subnet.
* VIP (Virtual IP): The VIP address to be assigned to the load balancer (DSR). You can choose the assignment method between **Auto assign** or **Manual assign**.
  * Auto assign: Automatically assign one of the available IPs in the subnet to use as the VIP.
  * Manual assign: Directly enter an IP within the CIDR range of the subnet to use as the VIP.

!!! danger "Caution"
    Creation will fail if the manually specified VIP address is not within the subnet's CIDR range. Be sure to specify an address within the IP range of the subnet.

!!! tip "Note"
    The load balancer (DSR) operates at the TCP/UDP L4 level, and server response traffic does not pass through the load balancer. Therefore, unlike standard load balancers, L7 features (HTTP header-based routing, SSL offloading, listener/member group concepts, etc.) are not provided.

#### 2. Configure Health Check

Set up health checks to periodically verify that member instances are functioning properly.

* Health check protocol: Select the protocol to be used for health checks. Choose one from **TCP / ICMP / HTTP**.
* Interval: The period (in seconds) at which health check requests are sent.
* Timeout: The timeout duration (in seconds) for each health check request. If no response is received within this time, it is considered a failure.
* Maximum retries: The maximum number of retries before an instance is marked as unhealthy. (1~10)

Configure additional items depending on the protocol as follows:

**TCP**

* Health check port: Specify the port number to attempt TCP connection.

**ICMP**

* No additional port configuration is required. Connectivity is verified using ICMP Echo Request/Reply.

**HTTP**

* Health check port: Specify the port number to send HTTP requests.
* HTTP path (URL): Enter the URL path for health checks. The default value is `/`.
* Expected HTTP response code: Enter the HTTP response code to be considered a healthy response. The default value is `200`.

!!! danger "Caution"
    The interval must be greater than or equal to the timeout. If the timeout is greater than the interval, health checks may not work properly.

!!! tip "Note"
    TCP/HTTP health checks send requests to the DSR VIP as the destination, so if the VIP is not configured on the lo interface of the member server, the packet cannot be received or processed, causing the health check to fail and the member to be marked as `INACTIVE`. ICMP health checks send requests to the member's actual IP, so connectivity is verified regardless of VIP configuration.

#### 3. Configure Members

Specify the member instances to register when creating the load balancer (DSR). Member registration can also be done after creating the load balancer (DSR).

* Instance selection: Select instances (network interfaces) that belong to the same subnet as the load balancer (DSR). You can select one or more instances simultaneously to register as members.

!!! tip "Note"
    The load balancer (DSR) forwards the destination port (VIP port) of client requests to member instances without modification. Therefore, unlike standard load balancers, you do not specify service ports for each member during member registration, and only select the network interface of the member instance. The application on the member server must bind to `0.0.0.0` or the VIP and listen on the same port sent by the client.

!!! danger "Caution"
    For member instances to properly receive and respond to traffic destined for the VIP, the following configuration is required within the server:

    - Add the VIP as an additional allowed address to the network interface (console Network Interface menu)
    - Configure kernel parameters (`arp_ignore=1`, `arp_announce=2`)
    - Add the VIP to the lo interface as a `/32` subnet
    - Allow service ports and health check traffic in Security Groups

    For detailed procedures, refer to the member server configuration guide in [Load Balancer (DSR) Overview](/Network/Load%20Balancer(DSR)/en/overview/).

!!! tip "Note"
    The initial state of a newly registered member is `INACTIVE`. It automatically transitions to `ACTIVE` status after passing health checks and begins receiving traffic.

After entering all items, click the **Create Load Balancer** button to create the load balancer (DSR).

<a id='view-dsr-loadbalancers'></a>
### View and Modify Load Balancer (DSR)

#### Load Balancer (DSR) List

After creating a load balancer (DSR), you can view the basic information of the created load balancers (DSR) on the list screen. The items displayed on the list screen are as follows:

* Name: The name specified when creating the load balancer (DSR).
* VIP address: The private IP assigned to the load balancer (DSR). This IP can be accessed within the VPC.
* Floating IP: The floating IP connected for external access.
* Network: The name of the VPC to which the load balancer (DSR) belongs and the subnet CIDR.
* Number of members: The number of member instances registered with the load balancer (DSR).
* Status: The creation/operational status of the load balancer (DSR).

!!! tip "Note"
    The status of the load balancer (DSR) is determined by one of the following:

    | Status | Meaning |
    |--|--|
    | `ACTIVE` | Operating normally |
    | `BUILD` | Load balancer (DSR) is being created |
    | `ERROR` | An error occurred. Please contact the administrator. |

You can create additional load balancers (DSR) using the **+ Create DSR** button at the top, and delete load balancers (DSR) by selecting them with checkboxes from the list and clicking the **Delete** button.

#### Load Balancer (DSR) Details

When you select a load balancer (DSR) from the list, detailed information is displayed at the bottom of the screen. The details screen is divided into three tabs: **Basic Information**, **Members**, and **Health Check**.

The **Basic Information** tab displays the following information:

* Name, description
* Subnet and VIP address
* Floating IP connection information
* Status

#### Change Name
To change the name of the load balancer (DSR), click the **Edit Name** icon in the details section, enter the new name, and click the **OK** button.

#### Change Floating IP
You can connect or disconnect a floating IP to allow external network access to the load balancer (DSR).

1. Click the **Change Floating IP** button in the load balancer (DSR) details section.
2. Select the floating IP to connect. To disconnect the floating IP, select **Disabled**.
3. Click the **OK** button to apply the settings.

!!! tip "Note"
    Disconnecting a floating IP does not affect access through the VIP within the VPC.

!!! tip "Note"
    The VPC, subnet, and VIP address to which the load balancer (DSR) is connected cannot be changed after creation. If changes are needed, you must delete and recreate the load balancer (DSR).

<a id='delete-dsr-loadbalancers'></a>
### Delete Load Balancer (DSR)
From the load balancer (DSR) list screen, select the load balancer (DSR) you want to delete and click the **Delete** button. When a confirmation dialog appears, click the **OK** button to delete the load balancer (DSR).

!!! danger "Caution"
    When you delete a load balancer (DSR), all members registered with that DSR are also deleted. If a floating IP is connected, it is automatically disconnected.

<a id='manage-dsr-members'></a>
## Manage Members

Select the desired load balancer (DSR) from the load balancer (DSR) list and click the **Members** tab to display the member instance management screen.

### Member List

In the **Members** tab, you can view the list and status of member instances registered with the load balancer (DSR). The items displayed on the list are as follows:

* IP address: The IP address of the member instance.
* Device model: The resource type that owns the network port registered as a member.
* Device information: Integrated display of identification information (instance name, port ID, etc.) of the network port registered as a member.
* Status: The current status of the member.

!!! tip "Note"
    Since the load balancer (DSR) forwards the destination port of client requests to members without modification, the L4 service port is not separately displayed in the member items. The actual service port is the port on which the client made the request to the VIP, and the application on the member server must listen on that port.

!!! tip "Note"
    The status of a member is determined by one of the following:

    | Status | Meaning |
    |--|--|
    | `ACTIVE` | Health check passed, target for traffic distribution |
    | `INACTIVE` | Health check failed or immediately after new registration, excluded from traffic distribution targets |
    | `ONLINE` | Member is manually disabled |

<a id='add-dsr-members'></a>
### Add Members
Click the **+ Add Member** button in the **Members** tab to open the member addition dialog.

1. Select the **Instance** to register as a member from the list. You can select one or more instances simultaneously.
2. Click the **OK** button to register the selected instances as members.

!!! tip "Note"
    Unlike standard load balancers, the load balancer (DSR) does not require you to enter the L4 service port when adding members. Since the load balancer (DSR) forwards the destination port of client requests to members without modification, port mapping per member is unnecessary.

!!! danger "Caution"
    Note the following constraints when registering members:

    * Member instances must belong to the same subnet as the load balancer (DSR).
    * Only compute instances can be registered as members.
    * The same instance port cannot be registered multiple times with the same load balancer (DSR).
    * A single load balancer (DSR) can register up to 30 members by default.

!!! tip "Note"
    To properly receive traffic after registering members, you must add the VIP as an additional allowed address to the network interface, and configure ARP kernel parameters, add the VIP to the lo interface, and set Security Groups rules within the member server. For detailed procedures, refer to the member server configuration guide in [Load Balancer (DSR) Overview](/Network/Load%20Balancer(DSR)/en/overview/).

<a id='delete-dsr-members'></a>
### Delete Members
From the member list on the Members tab, select the member to delete and click the **Delete** button. When a confirmation dialog appears, click the **OK** button to remove the member from the load balancer (DSR).

!!! tip "Note"
    Deleting a member from the load balancer (DSR) does not delete the instance itself. Conversely, if you delete an instance that is registered as a member, that member is automatically removed from the load balancer (DSR).

<a id='manage-dsr-health-monitor'></a>
## Manage Health Check

In the **Health Check** tab of the load balancer (DSR) details screen, you can view and change the currently configured health check information.

<a id='view-dsr-health-monitor'></a>
### View Health Check
In the **Health Check** tab, you can view the following information about the currently configured health check:

* Health check protocol: TCP / ICMP / HTTP
* Health check port: The target port for health checks when using TCP or HTTP protocol
* Interval: Health check request period (in seconds)
* Timeout: Health check timeout (in seconds)
* Maximum retries: Number of retries before marking as unhealthy
* HTTP path (URL) / Expected HTTP response code: Only displayed when using HTTP protocol

<a id='change-dsr-health-monitor'></a>
### Change Health Check Settings
Click the **Change Settings** button in the **Health Check** tab to change the health check settings.

* Health check protocol: Select from TCP / ICMP / HTTP
* Enter required items for each protocol:
  * TCP: Health check port
  * ICMP: No additional items
  * HTTP: Health check port, HTTP path, expected HTTP response code
* Configure the interval, timeout, and maximum retries.

After completing the settings, click the **OK** button to apply the changes.

!!! danger "Caution"
    The interval must be greater than or equal to the timeout. If the timeout is greater than the interval, health checks may not work properly.

!!! tip "Note"
    Health check requests are sent from a dedicated health check IP automatically assigned in the same subnet as the load balancer (DSR). The member instance's Security Group must allow this traffic for health checks to work properly. For more details, refer to the Security Groups configuration section in [Load Balancer (DSR) Overview](/Network/Load%20Balancer(DSR)/en/overview/).

<a id='dsr-quota'></a>
## Quotas and Limitations

The following quotas and limitations apply when using the load balancer (DSR):

| Item | Default Limit | Description |
|--|--|--|
| Number of load balancers (DSR) per project | 10 | Maximum number of load balancers (DSR) that can be created per project |
| Number of members per load balancer (DSR) | 30 | Maximum number of members that can be registered with a single load balancer (DSR) |

!!! tip "Note"
    If you need to use quotas exceeding the default limits, please contact customer support.