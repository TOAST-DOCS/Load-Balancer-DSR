<!-- machine_translated: true -->

<!-- pre-align:aligned sig=f84534729b3d -->

<a id='network-load-balancerdsr-console-user-guide'></a>
## Network > Load Balancer(DSR) > Console User Guide { #network-load-balancerdsr-console-user-guide }

<a id='manage-dsr-loadbalancers'></a>
## Load Balancer (DSR) Management { #manage-dsr-loadbalancers }

<a id='create-dsr-loadbalancers'></a>
### Create Load Balancer (DSR) { #create-dsr-loadbalancers }
You can easily create a DSR-type load balancer by simply entering the settings in the NHN Cloud console. Load Balancer (DSR) operates in direct server return (DSR) mode, allowing server response traffic to be sent directly to the client without passing through the load balancer, providing high throughput.

The Load Balancer (DSR) creation screen consists of the following three sections:

<a id='create-dsr-loadbalancers-load-balancer-dsr-basic-information-settings'></a>
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

<a id='create-dsr-loadbalancers-health-check-settings'></a>
#### 2. Health Check Settings

<a id='manage-dsr-loadbalancers'></a>
## Load Balancer (DSR) Management { #manage-dsr-loadbalancers }

<a id='create-dsr-loadbalancers'></a>
### Create Load Balancer (DSR) { #create-dsr-loadbalancers }
You can easily create a DSR-type load balancer by simply entering the settings in the NHN Cloud console. Load Balancer (DSR) operates in direct server return (DSR) mode, allowing server response traffic to be sent directly to the client without passing through the load balancer, providing high throughput.

The Load Balancer (DSR) creation screen consists of the following three sections:

<a id='create-dsr-loadbalancers-load-balancer-dsr-basic-information-settings'></a>
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

<a id='create-dsr-loadbalancers-health-check-settings'></a>
#### 2. Health Check Settings

Configure health checks to periodically verify that member instances are operating normally.

* Health check protocol: Select the protocol to use for health checks. Select one of the following: **TCP, ICMP, or HTTP**.

<a id='create-dsr-loadbalancers-member-settings'></a>
#### 3. Member Settings

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

<a id='view-dsr-loadbalancers'></a>
### Load Balancer (DSR) Details and Modification { #view-dsr-loadbalancers }

<a id='view-dsr-loadbalancers-load-balancer-dsr-list'></a>
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

<a id='view-dsr-loadbalancers-load-balancer-dsr-details'></a>
#### Load Balancer (DSR) Details

Selecting a Load Balancer (DSR) from the list displays its details at the bottom of the screen. The details screen is divided into three tabs: **Basic information**, **Members**, and **Health Check**.

The **Basic Information** tab displays the following:

* Name, description
* Subnet, VIP address
* Floating IP connection information
* Whether to set delete protection
* Status

<a id='view-dsr-loadbalancers-rename'></a>
#### Rename
To modify the name of Load Balancer (DSR), click **Modify Name** icon in the details, enter the new name, and click **Confirm**.

<a id='view-dsr-loadbalancers-change-floating-ip'></a>
#### Change Floating IP
A Floating IP can be connected or disconnected to enable access to Load Balancer (DSR) from an external network.

!!! tip "Note"
    TCP/HTTP health checks send requests to the DSR VIP as the destination, if the VIP is not configured on the lo interface of the member server, the packets cannot be received or processed, causing the health check to fail and the member to be marked as `INACTIVE`. ICMP health checks send requests to the actual IP of the member, so they only verify connectivity regardless of the VIP configuration.

<a id='create-dsr-loadbalancers-member-settings'></a>
#### 3. Member Settings

Specify the member instances to register when creating Load Balancer (DSR). Members can also be registered after Load Balancer (DSR) is created.

<a id='view-dsr-loadbalancers-delete-protection'></a>
#### Deletion Protection
If you activate deletion protection, you can protect load balancers (DSR) from accidental deletion. The load balancer (DSR) cannot be deleted until deletion protection is deactivated. The deletion protection setting can be changed at any time, even after the load balancer (DSR) is created.

<a id='delete-dsr-loadbalancers'></a>
### Delete Load Balancer (DSR) { #delete-dsr-loadbalancers }
On the Load Balancer (DSR) list screen, select the Load Balancer (DSR) to delete, click **Delete** button, and then click **Confirm** button in the confirmation window to delete the selected Load Balancer (DSR).

!!! tip "Note"
    Load Balancer (DSR) forwards client requests to member instances while preserving the destination port of the client request (VIP port). Therefore, unlike a standard load balancer, the service port is not specified per member when registering members; only the network interface of the member instance is selected. The application on the member server must be bound to `0.0.0.0` or the VIP and listen on the same port that the client sends requests to.

!!! tip "Note"
    You cannot delete a Load Balancer (DSR) with deletion protection enabled. To delete it, you must first disable [deletion protection](#view-dsr-loadbalancers-delete-protection) for the Load Balancer (DSR).

<a id='manage-dsr-members'></a>
## Member management { #manage-dsr-members }

    - Add the VIP as an additional allowed address on the network interface (console Network Interface menu)
    - Configure kernel parameters (`arp_ignore=1`, `arp_announce=2`)
    - Add the VIP to the `lo` interface with a `/32` subnet
    - Allow service port and health check traffic in Security Groups

<a id='member-list'></a>
### Member List { #member-list }

<a id='view-dsr-loadbalancers-load-balancer-dsr-list'></a>
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

<a id='add-dsr-members'></a>
### Add Member { #add-dsr-members }
Click **+ Add Member** button on the **Member** tab to display the add member modal.

<a id='view-dsr-loadbalancers-load-balancer-dsr-details'></a>
#### Load Balancer (DSR) Details

Selecting a Load Balancer (DSR) from the list displays its details at the bottom of the screen. The details screen is divided into three tabs: **Basic information**, **Members**, and **Health Check**.

The **Basic Information** tab displays the following:

* Name, Description
* Subnet, VIP address
* Floating IP connection information
* Whether to set delete protection
* Status

<a id='view-dsr-loadbalancers-rename'></a>
#### Rename
To modify the name of Load Balancer (DSR), click **Modify Name** icon in the details, enter the new name, and click **Confirm**.

<a id='deactivate-dsr-members'></a>
### Deactivate Members { #deactivate-dsr-members }
You can temporarily exclude a member from the service without deleting it. Select the member to exclude from the list on the Members tab, click the **Deactivate Members** button, and then click **Confirm**. The status of the deactivated member changes to `ONLINE`, and the member is excluded from traffic distribution.

!!! tip "Note"
    Disabled members are not removed from Load Balancer (DSR) and remain registered, so they continue to count toward the number of members per Load Balancer (DSR) quota. For more information about member status values, see [Member List](#member-list).

<a id='delete-dsr-members'></a>
### Delete Members { #delete-dsr-members }
Select the member to delete from the list on the Members tab and click **Delete** button. When the confirmation window appears, click **Confirm** to remove the member from Load Balancer (DSR).

1. Click **Change Floating IP** button in the Load Balancer (DSR) details.
2. Select the Floating IP to associate. To disassociate a Floating IP, select **Disabled**.
3. Click **Confirm** to apply the settings.

<a id='manage-dsr-health-monitor'></a>
## Health Check Management { #manage-dsr-health-monitor }

!!! tip "Note"
    The VPC, subnet, and VIP address connected to Load Balancer (DSR) cannot be changed after creation. If a change is needed, delete Load Balancer (DSR) and recreate it.

<a id='view-dsr-health-monitor'></a>
### View health check { #view-dsr-health-monitor }
The **Health Check** tab displays the following information about the currently configured health check:

!!! danger "Warning"
    Deleting Load Balancer (DSR) will also delete all members registered in the DSR. If a Floating IP is associated, it will be automatically released.

<a id='change-dsr-health-monitor'></a>
### Change Health Check Settings { #change-dsr-health-monitor }
Click **Change Setting** button on the **Health Check** tab to modify the health check settings.

<a id='manage-dsr-members'></a>
## Member Management { #manage-dsr-members }

Select the desired load balancer (DSR) from the load balancer (DSR) list, then click **Members** tab to display the member instance management screen.

<a id='member-list'></a>
### Member List { #member-list }

The **Members** tab displays the list and status of member instances registered in Load Balancer (DSR). The items displayed in the list are as follows:

<a id='dsr-quota'></a>
## Quota and Limitations { #dsr-quota }

!!! tip "Note"
    Since Load Balancer (DSR) forwards client requests to members while preserving the destination port, the L4 service port is not displayed separately in the member list. The actual service port is the port that the client uses to send requests to the VIP, and the application on the member server must listen on the port.

    | `ACTIVE` | Health check passed, target for traffic distribution |
    | `INACTIVE` | Health check failed or immediately after being newly registered, excluded from traffic distribution |
    | `ONLINE` | The member is manually disabled |

<a id='add-dsr-members'></a>
### Deactivate Members { #add-dsr-members }
You can temporarily exclude a member from the service without deleting it. Select the member to exclude from the list on the Members tab, click the **Deactivate Member** button, and then click **Confirm**. The status of a deactivated member changes to `ONLINE` and the member is excluded from traffic distribution.
