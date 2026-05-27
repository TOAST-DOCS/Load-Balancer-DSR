## Network > Load Balancer(DSR) > Console User Guide

<a id='manage-dsr-loadbalancers'></a>
## Load Balancer (DSR) Management

<a id='create-dsr-loadbalancers'></a>
### Create Load Balancer (DSR)
You can easily create a DSR-type load balancer by simply entering the settings in the NHN Cloud console. Load Balancer (DSR) operates in direct server return (DSR) mode, allowing server response traffic to be sent directly to the client without passing through the load balancer, providing high throughput.

The Load Balancer (DSR) creation screen consists of the following three sections:

#### 1. Load Balancer (DSR) Basic Information Settings

Configure the basic information for Load Balancer (DSR). The required items are as follows:

* Name: Enter the name of Load Balancer (DSR).
* Description: Enter the description of Load Balancer (DSR).
* VPC: Select the VPC to which Load Balancer (DSR) will be connected.
* Subnet: Select the subnet to which Load Balancer (DSR) will belong. Load Balancer (DSR) and member instances must be located in the same subnet.
* Virtual IP (VIP): The VIP address to be assigned to Load Balancer (DSR). The assignment method can be selected from **Auto Assign** or **Manual Assign**.
  * Auto assign: An available IP from the subnet is automatically assigned and used as the VIP.
  * Manual assign: A desired IP within the CIDR range of the subnet is entered directly and used as the VIP.

!!! danger "Caution"
    If the manually specified VIP address is not within the CIDR range of the subnet, creation will fail. Make sure to specify an IP within the IP range of the subnet.

!!! tip "Note"
    Load Balancer (DSR) operates at the TCP/UDP L4 level, and server response traffic does not pass through the load balancer. Therefore, unlike a standard load balancer, L7 features such as HTTP header-based routing, SSL offloading, and the listener/member group concept are not provided.

#### 2. Health Check Settings

Configure health checks to periodically verify that member instances are operating normally.

* Health check protocol: Select the protocol to use for health checks. Select one of the following: **TCP, ICMP, or HTTP**.

* Delay: The interval (in seconds) at which health check requests are sent.
* Maximum response wait time (timeout): The timeout period (in seconds) for each health check request. If no response is received within this time, the request is considered failed.
* Max retries: The maximum number of retries before an instance is considered unhealthy. (1–10)

Configure the following additional items for each protocol:

**TCP**

* Health check port: Specify the port number on which TCP connections are attempted.

**ICMP**

* No separate port configuration is required. Connectivity is verified using ICMP Echo Request/Reply.

**HTTP**

* Health check port: Specify the port number to which HTTP requests are sent.
* HTTP path (URL): Enter the URL path on which health checks are performed. The default value is `/`.
* Expected HTTP response code: Enter the HTTP response code to be considered a normal response. The default value is `200`.

!!! danger "Caution"
    The delay must be greater than or equal to the timeout. If the timeout is greater than the delay, health checks may not function correctly.

!!! tip "Note"
    TCP/HTTP health checks send requests to the DSR VIP as the destination, if the VIP is not configured on the lo interface of the member server, the packets cannot be received or processed, causing the health check to fail and the member to be marked as `INACTIVE`. ICMP health checks send requests to the actual IP of the member, so they only verify connectivity regardless of the VIP configuration.

#### 3. Member Settings

Specify the member instances to register when creating Load Balancer (DSR). Members can also be registered after Load Balancer (DSR) is created.

* Select instance: Select the instance (network interface) that belongs to the same subnet as Load Balancer (DSR). One or more instances can be selected simultaneously and registered as members.

!!! tip "Note"
    Load Balancer (DSR) forwards client requests to member instances while preserving the destination port of the client request (VIP port). Therefore, unlike a standard load balancer, the service port is not specified per member when registering members; only the network interface of the member instance is selected. The application on the member server must be bound to `0.0.0.0` or the VIP and listen on the same port that the client sends requests to.

!!! danger "Caution"
    For a member instance to properly receive and respond to traffic arriving at the VIP, the following configurations are required within the server.

    - Add the VIP as an additional allowed address on the network interface (console Network Interface menu)
    - Configure kernel parameters (`arp_ignore=1`, `arp_announce=2`)
    - Add the VIP to the `lo` interface with a `/32` subnet
    - Allow service port and health check traffic in Security Groups

    For detailed instructions, see the Member server configuration guide in [Load Balancer (DSR) Overview](/Network/Load%20Balancer(DSR)/en/overview/).

!!! tip "Note"
    The initial status of a newly registered member is `INACTIVE`. Once the health check passes, the status automatically transitions to `ACTIVE` and the member begins receiving traffic.

After entering all items, click **Create Load Balancer** to create Load Balancer (DSR).

<a id='view-dsr-loadbalancers'></a>
### Load Balancer (DSR) Details and Modification

#### Load Balancer (DSR) List

Once Load Balancer (DSR) creation is complete, the basic information of the created Load Balancer (DSR) instances can be viewed on the list screen. The items displayed on the list screen are as follows:

* Name: The name specified when creating Load Balancer (DSR).
* VIP address: The private IP assigned to Load Balancer (DSR). This IP can be used for access within the VPC.
* Floating IP: The Floating IP connected for external access.
* Network: The name of the VPC and subnet CIDR to which Load Balancer (DSR) belongs.
* Number of members: The number of member instances registered in Load Balancer (DSR).
* Status: The creation/operation status of Load Balancer (DSR).

!!! tip "Note"
    The status of Load Balancer (DSR) is determined by one of the following:

    | Status | Description |
    |--|--|
    | `ACTIVE` | Operating normally |
    | `BUILD` | Load Balancer (DSR) being created |
    | `ERROR` | Error occurred. Contact the administrator. |

Additional Load Balancer (DSR) instances can be created using **+ Create DSR** button at the top. To delete, select Load Balancer (DSR) instances using the checkboxes in the list, then click **Delete** button.

#### Load Balancer (DSR) Details

Selecting a Load Balancer (DSR) from the list displays its details at the bottom of the screen. The details screen is divided into three tabs: **Basic information**, **Members**, and **Health Check**.

The **Basic Information** tab displays the following:

* Name, description
* Subnet, VIP address
* Floating IP connection information
* Status

#### Rename
To modify the name of Load Balancer (DSR), click **Modify Name** icon in the details, enter the new name, and click **Confirm**.

#### Change Floating IP
A Floating IP can be connected or disconnected to enable access to Load Balancer (DSR) from an external network.

1. Click **Change Floating IP** button in the Load Balancer (DSR) details.
2. Select the Floating IP to associate. To disassociate a Floating IP, select **Disabled**.
3. Click **Confirm** to apply the settings.

!!! tip "Note"
    Disassociating a Floating IP does not affect access to the VIP from within the VPC.

!!! tip "Note"
    The VPC, subnet, and VIP address connected to Load Balancer (DSR) cannot be changed after creation. If a change is needed, delete Load Balancer (DSR) and recreate it.

<a id='delete-dsr-loadbalancers'></a>
### Delete Load Balancer (DSR)
On the Load Balancer (DSR) list screen, select the Load Balancer (DSR) to delete, click **Delete** button, and then click **Confirm** button in the confirmation window to delete the selected Load Balancer (DSR).

!!! danger "Warning"
    Deleting Load Balancer (DSR) will also delete all members registered in the DSR. If a Floating IP is associated, it will be automatically released.

<a id='manage-dsr-members'></a>
## Member Management

Select the desired load balancer (DSR) from the load balancer (DSR) list, then click **Members** tab to display the member instance management screen.

### Member List

The **Members** tab displays the list and status of member instances registered in Load Balancer (DSR). The items displayed in the list are as follows:

* IP address: The IP address of the member instance.
* Device model: The type of resource owned by the network port registered as a member.
* Device information: The identification information (instance name, port ID, etc.) of the network port registered as a member is displayed in a consolidated format.
* Status: The current status of the member.

!!! tip "Note"
    Since Load Balancer (DSR) forwards client requests to members while preserving the destination port, the L4 service port is not displayed separately in the member list. The actual service port is the port that the client uses to send requests to the VIP, and the application on the member server must listen on the port.

!!! tip "Note"
    The member status is determined by one of the following:

    | Status | Description |
    |--|--|
    | `ACTIVE` | Health check passed, target for traffic distribution |
    | `INACTIVE` | Health check failed or immediately after being newly registered, excluded from traffic distribution |
    | `ONLINE` | The member is manually disabled |

<a id='add-dsr-members'></a>
### Add Member
Click **+ Add Member** button on the **Member** tab to display the add member modal.

1. Select the **instance** to register as a member from the list. One or more instances can be selected simultaneously.
2. Click **Confirm** button to register the selected instances as members.

!!! tip "Note"
    Unlike a standard load balancer, Load Balancer (DSR) does not require entering an L4 service port when adding members. Since Load Balancer (DSR) forwards the destination port of client requests to members without modification, per-member port mapping is not required.

!!! danger "Caution"
    Note the following restrictions when registering members:

    * Member instances must belong to the same subnet as Load Balancer (DSR).
    * Only compute instances can be registered as members.
    * The same instance port cannot be registered more than once in the same Load Balancer (DSR).
    * By default, up to 30 members can be registered per Load Balancer (DSR).

!!! tip "Note"
    To properly receive traffic after registering a member, add the VIP as an additional allowed address on the network interface, and configure the ARP kernel parameters, add the VIP to the lo interface, and set up Security Groups rules within the member server. For detailed instructions, see the Member server configuration guide in the [Load Balancer (DSR) Overview](/Network/Load%20Balancer(DSR)/ko/overview/).

<a id='delete-dsr-members'></a>
### Delete Member
Select the member to delete from the list on the Members tab and click **Delete** button. When the confirmation window appears, click **Confirm** to remove the member from Load Balancer (DSR).

!!! tip "Note"
    Deleting a member from Load Balancer (DSR) does not delete the instance itself. Conversely, if an instance registered as a member is deleted, the member is automatically removed from Load Balancer (DSR).

<a id='manage-dsr-health-monitor'></a>
## Health Check Management

The current health check settings can be viewed and modified on the **Health Check** tab of the Load Balancer (DSR) details screen.

<a id='view-dsr-health-monitor'></a>
### View Health Check
The **Health Check** tab displays the following information about the currently configured health check:

* Health check protocol: TCP / ICMP / HTTP
* Health check port: The target port for health checks when using TCP or HTTP protocol
* Delay: The health check request interval (in seconds)
* Maximum response wait time: The health check timeout (in seconds)
* Max retries: The number of retries before an instance is considered unhealthy
* HTTP path (URL) / Expected HTTP response code: Displayed only when using HTTP protocol

<a id='change-dsr-health-monitor'></a>
### Change Health Check Settings
Click **Change Setting** button on the **Health Check** tab to modify the health check settings.

* Health check protocol: Select one of TCP, ICMP, or HTTP.
* Enter the required items for each protocol:
  * TCP: Health check port
  * ICMP: No additional items
  * HTTP: Health check port, HTTP path, and expected HTTP response code
* Configure the delay, maximum response wait time, and max retries.

Click **Confirm** after completing the settings to apply the changes

!!! danger "Caution"
    The delay must be greater than or equal to the timeout. If the timeout is greater than the delay, health checks may not function correctly.

!!! tip "Note"
    Health check requests are sent from a dedicated health check IP automatically assigned to the same subnet as Load Balancer (DSR). The Security Group of member instances must allow this traffic for health checks to function correctly. For more information, see the Security Groups configuration section in the [Load Balancer (DSR) Overview](/Network/Load%20Balancer(DSR)/ko/overview/).

<a id='dsr-quota'></a>
## Quota and Limitations

The following quotas and limitations apply when using Load Balancer (DSR):

| Item | Default Limit | Description |
|--|--|--|
| Number of Load Balancers (DSR) per project | 10 | Maximum number of Load Balancer (DSR) instances that can be created per project |
| Number of members per Load Balancer (DSR) | 30 | Maximum number of members that can be registered in a single Load Balancer (DSR) |

!!! tip "Note"
    If you need to exceed the default quota, contact customer support.
