<!-- machine_translated: true -->

<!-- pre-align:aligned sig=9be9447442b4 -->
{% set ni_suffix = "-gov" if "gov" in build_flags
    else "-ncgn" if "ncgn" in build_flags
    else "-ngoic" if "ngoic" in build_flags
    else "-ngovc" if "ngovc" in build_flags
    else "-ngsc" if "ngsc" in build_flags
    else "-ninc" if "ninc" in build_flags
    else "" -%}

<a id="network-load-balancer-dsr-overview"></a>
## Network > Load Balancer (DSR) > Overview { #network-load-balancer-dsr-overview }

NHN Cloud provides a load balancer that supports direct server return (DSR). With the DSR load balancer:

- Distribute workloads that a single instance cannot handle across multiple instances to increase throughput.
- Improve availability by automatically removing instances that have failed or are under maintenance from the service.
- Achieve high performance, as server response traffic is sent directly to the client without passing through the load balancer.


<a id="dsr-method"></a>
## DSR Method { #dsr-method }

The DSR load balancer uses a different traffic handling method than a standard load balancer.

<a id="differences-from-a-standard-load-balancer-proxy-mode"></a>
### Differences from a standard load balancer (proxy mode) { #differences-from-a-standard-load-balancer-proxy-mode }

| Category | Standard load balancer (proxy mode) | Load balancer (DSR) |
|------|------|------|
| Client → Server | Via load balancer | Via load balancer |
| Server → Client | Via load balancer | Sent directly, bypassing the load balancer |
| Original IP verification | `X-Forwarded-For` header or Proxy Protocol required | Client IP directly verifiable |
| Load balancer burden | Processes both requests and responses | Processes requests only |
| Throughput | Neutral | Very high |
| Protocol support | HTTP, HTTPS, TERMINATED_HTTPS, TCP | TCP, UDP |

<a id="how-dsr-works"></a>
### How DSR works { #how-dsr-works }

The VIP (Virtual IP) of Load Balancer (DSR) is a **private IP** assigned from the subnet to which the load balancer belongs. It is the address specified in the **VIP (Virtual IP)** field when creating Load Balancer (DSR), and it is displayed in the **IP Address** column of the list. This is a separate address from the Floating IP used for access from the internet.

| Type | Example Address | Role | lo Interface and Additional Allowed Address Settings for Member Servers |
|------|----------|------|------|
| VIP (Private IP) | `192.168.1.100` | The actual destination of traffic forwarded to the load balancer and member instances | Required |
| Floating IP | `133.186.0.31` | Entry point for internet access | Not required |
The traffic processing order is as follows.

1. Client request: The client sends a request to the VIP (private IP) of the load balancer. When accessing from the internet, the request is sent to the Floating IP associated with the VIP, and the destination of the request is translated to the VIP and forwarded to the load balancer.
2. Request distribution: The load balancer selects an appropriate member instance and forwards the request while keeping the destination IP as the VIP.
3. Direct response delivery: The member instance sends the response directly to the client without passing through the load balancer, using the VIP as the source. For responses to requests that arrived via a Floating IP, the source is translated back to the Floating IP before reaching the client.

!!! danger "Caution"
    The address that must be registered in the lo interface and the additional allowed addresses of the network interface of the member server is the VIP (private IP). If you register a Floating IP, traffic will not be processed correctly.

!!! tip "Note"
    The DSR method has the following advantages since the response traffic does not pass through the load balancer:

    - The load on the load balancer is significantly reduced, allowing more concurrent connections to be handled.
    - It is particularly beneficial for services with large response data, such as video streaming and large file downloads.
    - Network latency is reduced, improving response speed.
    - The client's source IP can be directly identified on the server.


<a id="session-affinity"></a>
## Session Affinity { #session-affinity }

Load balancer (DSR) provides a session affinity feature. When enabled, requests from the same client are consistently forwarded to the same member instance.

- Session affinity disabled: Members are selected based on the 5-tuple (source IP, source port, destination IP, destination port, protocol), and traffic is distributed accordingly. Even if the source IP is the same, requests may be routed to different members if the source port differs.
- Session affinity enabled: Requests from the same client are always forwarded to the same member based on the source IP. Even if the source port changes, the same member is selected as long as the source IP remains the same.

Members are selected using consistent hash-based mapping (Consistent Hashing) without a separate sticky table. When session affinity is disabled, the 5-tuple is used as the hash input; when enabled, the source IP is used. If the member configuration remains the same, the same input key always maps to the same member. Additionally, while an established session remains valid, all packets in that session are forwarded to the same member, ensuring per-session member affinity.

The session persistence method varies by protocol:

- TCP: Connection termination is detected by observing TCP flags (FIN/RST). Once a termination signal is confirmed, the session is quickly reclaimed with a short expiration time. The session is maintained as long as other traffic continues.
- UDP: Since there is no connection termination signal, the session expires if no additional traffic is received for a certain period. Until expiration, the same flow continues to be forwarded to the same member.

!!! tip "Note"
    Session affinity is useful in the following cases:

    - When user login sessions are managed on individual servers
    - When session synchronization between instances is not implemented
    - When there is a requirement to process a specific user's requests on the same server

!!! tip "Note"
    Session affinity settings can be changed during operation. Changes do not affect existing TCP connections or ongoing UDP flows, and the updated settings are applied to new connections and flows.


<a id="instance-health-check"></a>
## Instance health check { #instance-health-check }

Load Balancer (DSR) periodically performs health checks to verify that member instances are operating normally. A health check confirms whether the expected response is received according to the specified protocol. If a normal response is not received within the specified number of attempts or time limit, the instance is considered unhealthy and excluded from load balancing. This feature ensures uninterrupted service even in the event of unexpected failures or maintenance.

<a id="supported-protocols"></a>
### Supported Protocols { #supported-protocols }

Load Balancer (DSR) supports the following health check protocols:

- ICMP: A basic connectivity check method using ICMP Echo Request/Reply. Quickly verifies the network connectivity of an instance. Requests are sent to the actual IP of the member instance as the destination.

- TCP: Checks connectivity by attempting a TCP connection on the specified port. Verifies whether a specific service port is operating normally. Requests are sent to the VIP (private IP) of Load Balancer (DSR) as the destination.

- HTTP: Sends an HTTP request to the specified path and checks the response code. Provides a more accurate check of the actual service status of a web application. Requests are sent to the VIP (private IP) of Load Balancer (DSR) as the destination.

!!! tip "Note"
    The destination of TCP/HTTP health check packets is the VIP, not the Floating IP, and this is expected behavior. Since TCP/HTTP health checks send requests to the DSR VIP as the destination, if the VIP is not configured on the lo interface of the member server, the packets cannot be received or processed, causing the health check to fail and the member to be marked as `INACTIVE`. This behavior is intended to detect missing VIP configuration on the server side at an early stage. ICMP health checks send requests to the actual IP of the member, so they only verify connectivity regardless of the VIP configuration.

<a id="health-check-settings"></a>
### Health Check Settings { #health-check-settings }

The following items must be configured for health checks:

| Item | Description | Note |
|------|------|------|
| Delay | The interval (in seconds) at which health check requests are sent. | - |
| Maximum retries (max_retries) | The maximum number of retries before an instance is considered unhealthy. | 1-10 |
| Timeout (timeout) | The timeout period (in seconds) for each health check request. If no response is received within this time, the request is considered failed. | - |
| Protocol (type) | The protocol to use for health checks. | ICMP, TCP, HTTP |
| Port (health_check_port) | The port number on which health checks are performed. | Required when using TCP or HTTP |
| HTTP Path (http_path) | The URL path to which requests are sent during HTTP health checks. | Configurable when using HTTP (default: `/`) |
| Expected HTTP response code (expected_http_code) | The response code considered normal during HTTP health checks. | Configurable when using HTTP (default: `200`) |

!!! danger "Caution"
    The delay must be greater than or equal to the timeout. If the timeout is greater than the delay, health checks may not function correctly.


<a id="create-load-balancer-dsr"></a>
## Create Load Balancer (DSR) { #create-load-balancer-dsr }

Load Balancer (DSR) is created within the [VPC](/Network/VPC/en/overview{% if "gov" in build_flags %}-gov{% endif %}/#glossary) in the [subnet](/Network/VPC/en/overview{% if "gov" in build_flags %}-gov{% endif %}/#glossary).

<a id="assign-vip-address"></a>
### Assign VIP Address { #assign-vip-address }

Virtual IP (VIP) is a private IP assigned from the subnet to which Load Balancer (DSR) belongs. When creating Load Balancer (DSR), the VIP can be assigned in one of the following two ways:

- Auto assign: An available IP from the subnet is automatically assigned and used as the VIP.
- Manual assign: A desired IP within the CIDR range of the subnet is specified and used as the VIP.

If you need to access it from the internet, associate a Floating IP with this VIP. For more information, see [Floating IP Association](#floating-ip-association).

!!! danger "Caution"
    If the manually specified VIP address is not within the CIDR range of the subnet, creation will fail. Make sure to specify an IP within the IP range of the subnet.

<a id="register-member"></a>
### Register Member { #register-member }

Load Balancer (DSR) distributes incoming traffic by registering instances as members. The following requirements must be met when registering members:

- Subnet match: The port of the member instance must belong to the same subnet as Load Balancer (DSR).
- Compute instance: Members must be compute instances. (`device_owner` starts with the `compute:` prefix)
- SDN support: The member port must operate in an SDN (software defined network) environment.

!!! danger "Caution"
    By default, up to 30 members can be registered per Load Balancer (DSR). If more members are needed, contact us separately.

!!! tip "Note"
    * The initial status of a newly registered member is `INACTIVE`. Once the health check passes, the status automatically transitions to `ACTIVE` and the member begins receiving traffic.
    * The same instance port cannot be registered more than once in the same Load Balancer (DSR).
    * For a member instance to properly receive and respond to traffic, ARP and VIP settings must be configured within the server. For more information, see the Member server configuration guide section below.

<a id="member-server-configuration-guide"></a>
## Member Server Configuration Guide { #member-server-configuration-guide }

Load Balancer (DSR) forwards client requests to member servers with the VIP (Virtual IP) as the destination. For the member server to properly receive and respond to these packets, the following settings are required on the server side. All `<VIP>` values used in the settings below refer to this private IP.

!!! danger "Caution"
    Configurations must be applied in the following order: Step 1 (kernel parameters) → Step 2 (VIP configuration). If the VIP is assigned before configuring the kernel parameters, an ARP conflict with the load balancer's VIP may occur, resulting in a network failure.

<a id="kernel-parameter-configuration-arp-ignoreannounce"></a>
### 1. Kernel Parameter Configuration (ARP Ignore/Announce) { #kernel-parameter-configuration-arp-ignoreannounce }

Before configuring the VIP on a network interface, the kernel must first be configured to prevent the server from responding to ARP requests for the VIP. If the VIP is assigned without this configuration, an ARP conflict with the load balancer's VIP may occur, resulting in a network failure.

<a id="kernel-parameter-configuration-arp-ignoreannounce-parameter-value-definitions"></a>
#### Parameter value definitions

| Parameter | Value | Description |
|---------|---|------|
| `arp_ignore` | `1` | Responds to ARP requests only when the requested IP is configured on the interface from which the request was received. (Prevents responses to ARP requests for the VIP configured on lo) |
| `arp_announce` | `2` | When sending ARP packets externally, fixes the source IP to the address of the outgoing interface to prevent the VIP address from being exposed. |

<a id="kernel-parameter-configuration-arp-ignoreannounce-real-time-application"></a>
#### Real-time application

```bash
sudo sysctl -w net.ipv4.conf.all.arp_ignore=1
sudo sysctl -w net.ipv4.conf.all.arp_announce=2
sudo sysctl -w net.ipv4.conf.lo.arp_ignore=1
sudo sysctl -w net.ipv4.conf.lo.arp_announce=2
```

<a id="kernel-parameter-configuration-arp-ignoreannounce-permanent-application-etcsysctlconf"></a>
#### Permanent application (/etc/sysctl.conf)

Add the following content to the end of the file:

```bash
sudo tee -a /etc/sysctl.conf <<EOF
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2
net.ipv4.conf.lo.arp_ignore = 1
net.ipv4.conf.lo.arp_announce = 2
EOF

# Apply Settings
sudo sysctl -p
```

!!! tip "Note"
    You can check if the value has been configured to `1` by using the command `sysctl net.ipv4.conf.all.arp_ignore` after applying it.

<a id="vip-configuration-on-loopback-interface"></a>
### 2. VIP Configuration on Loopback Interface { #vip-configuration-on-loopback-interface }

The VIP is assigned to the lo interface so that the server can recognize packets forwarded from the load balancer (packets with the VIP as the destination) as its own.

<a id="vip-configuration-on-loopback-interface-temporary-configuration-deleted-on-reboot"></a>
#### Temporary configuration (deleted on reboot)

```bash
# Replace <VIP> with the actual load balancer VIP address.
sudo ip addr add <VIP>/32 dev lo
```

<a id="vip-configuration-on-loopback-interface-permanent-configuration"></a>
#### Permanent configuration

##### Ubuntu 18.04 and later (Netplan)

Modify the configuration file in the `/etc/netplan/` directory (e.g., `01-netcfg.yaml`).

!!! danger "Caution"
    The existing interface configuration must be preserved. Add or merge only the lo section.

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    lo:
      addresses:
        - 127.0.0.1/8
        - <VIP>/32  # Add load balancer VIP
```

Apply the configuration:

```bash
sudo netplan apply
```

##### CentOS / RHEL 7 and later

Create the file `/etc/sysconfig/network-scripts/ifcfg-lo:0`.

```bash
sudo tee /etc/sysconfig/network-scripts/ifcfg-lo:0 <<EOF
DEVICE=lo:0
IPADDR=<VIP>
NETMASK=255.255.255.255
ONBOOT=yes
EOF
```

Apply the configuration:

```bash
sudo ifup lo:0
```

<a id="vip-configuration-on-loopback-interface-when-a-member-of-multiple-dsr-instances"></a>
#### When a member of multiple DSR instances

If a single instance is registered as a member of multiple Load Balancer (DSR) instances, all VIPs must be added to the lo interface, and each VIP must also be registered in the additional allowed addresses of the network interface.

```bash
sudo ip addr add <VIP_1>/32 dev lo
sudo ip addr add <VIP_2>/32 dev lo
```

Example of permanent Netplan configuration:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    lo:
      addresses:
        - 127.0.0.1/8
        - <VIP_1>/32
        - <VIP_2>/32
```

<a id="configuration-verification-and-testing"></a>
### 3. Configuration Verification and Testing { #configuration-verification-and-testing }

If a member is in the `INACTIVE` status or is not receiving traffic, check the following items in order for each member instance. The missing configuration items may vary for each member. The order below is a diagnostic sequence, not a configuration sequence. When configuring for the first time, you must apply the kernel parameters first.

| Order | Inspection Item | How to View | Normal Status |
|------|----------|----------|----------|
| 1 | VIP on the lo interface | `ip addr show lo` | VIP (Private IP) registered as `/32` |
| 2 | Kernel parameters | `sysctl net.ipv4.conf.{all,lo}.arp_ignore`, `sysctl net.ipv4.conf.{all,lo}.arp_announce` | `arp_ignore=1`, `arp_announce=2` |
| 3 | Additional allowed addresses | **Additional Allowed Addresses** in the console **Network > Network Interface** | `<VIP>/32` or a range that includes the VIP is registered |
| 4 | Security Groups | Security Groups rules of member instances | Both the service port and the health check port are allowed (see [Security Groups Configuration](#security-groups-configuration) below) |
| 5 | Application Binding | `ss -ltnp` | Listening on `0.0.0.0` or VIP |
<a id="configuration-verification-and-testing-verify-ip-configuration"></a>
#### Verify IP configuration

Verifies that the VIP has been correctly registered on the `/32`.

```bash
ip addr show lo
```

Example output:

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536
    inet 127.0.0.1/8 scope host lo
    inet 192.168.1.100/32 scope host lo
```

<a id="configuration-verification-and-testing-verify-arp-response"></a>
#### Verify ARP response

When an ARP request is sent to the VIP from an external server (another server on the same subnet), the MAC address of the member server should not respond. (Only the MAC of the load balancer should respond for it to be normal)

```bash
# Run from another instance on the same subnet
arping -c 3 <VIP>
```

Verifies that the MAC address returned is the load balancer's MAC. If the member server's MAC is returned, the ARP configuration is incorrect.

<a id="configuration-verification-and-testing-verify-kernel-parameter"></a>
#### Verify kernel parameter

```bash
sysctl net.ipv4.conf.all.arp_ignore
sysctl net.ipv4.conf.all.arp_announce
sysctl net.ipv4.conf.lo.arp_ignore
sysctl net.ipv4.conf.lo.arp_announce
```

Each should output `1`, `2`, `1`, and `2`.

<a id="service-configuration"></a>
### 4. Service Configuration { #service-configuration }

<a id="service-configuration-application-binding"></a>
#### Application Binding

The application (Nginx, Apache, Tomcat, etc.) must be configured so that the socket is listening on `0.0.0.0` (Any) or the VIP to receive packets.

| Binding | Description | Example |
|------------|------|------|
| `0.0.0.0:port` | Listen on all IPs (recommended) | `listen 80;` (Nginx default) |
| `<VIP>:port` | Listen only on VIP | `listen 192.168.1.100:80;` |
| `<Server IP>:Port` | Only receive from the server's own IP — **VIP traffic cannot be received** | `listen 10.0.0.5:80;` |

!!! danger "Caution"
If the application is bound only to the server's actual interface IP (e.g., the IP of `eth0`), it cannot receive packets arriving at the VIP. The application must be bound to `0.0.0.0`, or the VIP address must be explicitly added as an additional binding.

<a id="service-configuration-response-to-health-check"></a>
#### Response to health check

Load Balancer (DSR) periodically sends health check requests to the member servers. The server must respond normally to the status check to maintain `ACTIVE` status.

| Health check type | Server requirements |
|---------------|-------------|
| ICMP | Must respond to ICMP Echo requests. If ICMP is blocked by the internal firewall of the server, it needs to be allowed. |
| TCP | Must accept TCP connections on the specified port. The service on the port must be running. |
| HTTP | Must return the expected HTTP response code (default 200) on the specified port/path. |

!!! tip "Note"
    * Health check requests are sent from a dedicated health check IP, not from the load balancer's VIP. Traffic from this IP must be allowed in the Security Group and the server's internal firewall. The dedicated health check IP is automatically assigned to the same subnet as the DSR.
    * If an internal firewall is configured on the server, ensure that the service port and health check port (including ICMP) are not blocked.

<a id="security-groups-configuration"></a>
### 5. Security Groups Configuration { #security-groups-configuration }

The Security Groups of member instances must allow service traffic and health check traffic from the DSR.

!!! tip "Note"
    The default Security Group is associated with the ports corresponding to the VIP and dedicated health check IP of Load Balancer (DSR). However, Security Groups filtering (flow) is not applied to the DSR port itself.

    Security Groups filtering is applied normally to the ports of member instances. Since DSR does not perform source IP translation (No SNAT), the source IP of service traffic is the client's original IP. Therefore, the client IP range or ANY (`0.0.0.0/0`) must be specified as the remote for service traffic.

    Since health check traffic is sent from a dedicated health check IP assigned to the same subnet as the DSR, specifying the CIDR of that subnet as the remote allows the traffic. The port of the dedicated health check IP also belongs to the default Security Group, so specifying the `default` SG as the remote is another way to allow it.

<a id="security-groups-configuration-method-1-easy-configuration"></a>
#### Method 1: Easy configuration

Since the DSR retains the client's source IP, both service traffic and status check traffic must be allowed separately.

| Direction | IP protocol | Port Range | Remote | Description |
|------|-----------|----------|------|------|
| Receive | TCP or UDP | Service Port (e.g., 80) | 0.0.0.0/0 | Allow service traffic from clients (specified according to the service protocol) |
| Inbound | Any | - | DSR subnet CIDR (e.g., `192.168.1.0/24`) | Allow health check traffic (the dedicated health check IP is assigned to the same subnet as the DSR) |

!!! tip "Note"
    Since DSR does not perform source IP translation, the source IP of service traffic arriving at the member server is the client's original IP. If the client IP range is not specified, allow `0.0.0.0/0`. If the client range is confirmed, it can be restricted to the corresponding CIDR.

!!! danger "Warning"
    You can also specify the `default` Security Group as the remote for health check traffic; however, in this case, all resources belonging to the default SG can access that port. If Security Groups are not specified when creating an instance, the default SG is applied, which loosens the network boundary segmented by subnet or purpose (web, app, db, etc.) at the Security Group level. We recommend using the `default` SG only for initial testing and validation, and specifying the DSR subnet CIDR in production environments.

<a id="security-groups-configuration-method-2-individual-rules-fine-grained-control"></a>
#### Method 2: Individual rules (fine-grained control)

Add individual rules when applying the principle of least privilege for security policy or when only specific ports need to be allowed.

| Usage | Protocol | Port | Remote (Recommended) | Note |
|------|---------|------|------|------|
| Service traffic (TCP) | TCP | Service port (e.g., 80, 443) | Client IP range or 0.0.0.0/0 | DSR does not perform SNAT, so the source IP is the client's original IP |
| Service traffic (UDP) | UDP | Service port (e.g., 53, 514) | Client IP range or 0.0.0.0/0 | When using UDP protocol services |
| TCP health check | TCP | `health_check_port` | DSR subnet CIDR | The dedicated health check IP is assigned to the same subnet as the DSR |
| ICMP health check | ICMP | - | DSR subnet CIDR | When using ICMP type |
| HTTP health check | TCP | `health_check_port` | DSR subnet CIDR | When using HTTP type |

##### Example of adding rules in the console

If the service port and TCP health check port are both 80, and the DSR subnet is `192.168.1.0/24`:

| Direction | IP protocol | Port Range | Remote | Description |
|------|-----------|----------|------|------|
| Receive | TCP | 80 | 0.0.0.0/0 | Service traffic from clients |
| Inbound | TCP | 80 | 192.168.1.0/24 | Health check traffic |

When using ICMP health checks, add the following:

| Direction | IP protocol | Port Range | Remote | Description |
|------|-----------|----------|------|------|
| Inbound | ICMP | - | 192.168.1.0/24 | ICMP health check |

!!! tip "Note"
    * If the health check port (`health_check_port`) is set differently from the service port, both ports must be allowed in the Security Groups.
    * If the client IP range is limited to a specific CIDR (e.g., `10.0.0.0/8`), the principle of least privilege can be applied by specifying that CIDR instead of `0.0.0.0/0`.
    * In a production environment, we recommend that you create dedicated Security Groups for each purpose (such as web, app, and db) and apply only the necessary rules, rather than using the default Security Group as-is. If you specify the DSR subnet CIDR instead of the `default` SG as the remote for health check rules, you can narrow the allowed range to that subnet.

<a id="network-interface-security-settings-update"></a>
### 6. Network Interface Security Settings Update { #network-interface-security-settings-update }

In the DSR method, the load balancer forwards packets to the member server while keeping the destination IP as the VIP. In the NHN Cloud network environment, packets whose source or destination is an IP other than the IP assigned to the instance are blocked by default for security purposes.

Therefore, the VIP must be added as an additional allowed address on the network interface so that the member instance can receive packets destined for the VIP and respond with the VIP as the source.

<a id="network-interface-security-settings-update-reason-for-the-settings"></a>
#### Reason for the Settings

```
[Client] → dst: VIP → [Load Balancer (DSR)] → dst: VIP → [Member Server]
                                                         ↑
                                         The assigned port IP differs from the destination (VIP) → Packet dropped if VIP is not registered as an additional allowed address
```

<a id="network-interface-security-settings-update-main-configuration-method"></a>
#### Main configuration method

Add the VIP of Load Balancer (DSR) to the additional allowed address section of the network interface (port) of the member server.

* Registering the VIP as an additional allowed address allows packets with that IP as the source or destination to pass through the port. This adheres to the principle of least privilege by selectively allowing only the necessary VIPs without disabling port security entirely.
* Configuration location: In the console, select the corresponding interface from the **Network > Network Interface** menu, then add the VIP address (`<VIP>/32`) to the **additional allowed addresses** section.
* If a single instance is a member of multiple Load Balancer (DSR) instances, all VIPs must be added to the additional allowed addresses.

!!! tip "Note"
    For the procedure to configure additional allowed addresses, see the [Console User Guide](/Network/Network%20Interface/en/console-guide$[ ni_suffix ]$/).

<a id="floating-ip-association"></a>
## Floating IP Association { #floating-ip-association }

A Floating IP can be associated with the VIP of Load Balancer (DSR) to enable access from external networks.

- Associating a Floating IP allows traffic to be forwarded from the internet to Load Balancer (DSR).
- Dissociating a Floating IP blocks external access, making the load balancer accessible only from the internal network.
- Associating or dissociating a Floating IP is automatically reflected in the load balancer.

Requests that arrive via a Floating IP are forwarded to the load balancer after the destination is converted to the VIP. The response has its source converted back to the Floating IP. Therefore, you must register the VIP — not the Floating IP — in the lo interface of the member server and in the additional allowed addresses of the network interface.

!!! tip "Note"
    Dissociating a Floating IP does not affect access to the VIP from the internal network.


<a id="quota-and-limitations"></a>
## Quota and Limitations { #quota-and-limitations }

The following quotas and limitations apply when using Load Balancer (DSR):

| Item | Default Limit | Description |
|------|----------|------|
| Number of Load Balancers (DSR) per project | 10 | Number of Load Balancers (DSR) that can be created per project |
| Number of members per Load Balancer (DSR) | 30 | Number of members that can be registered to a single Load Balancer (DSR) |
| Number of members per project | No limit | |

!!! tip "Note"
    If you need to exceed the default quota, contact customer support.


<a id="load-balancer-dsr-monitoring"></a>
## Load Balancer (DSR) Monitoring { #load-balancer-dsr-monitoring }

The status of Load Balancer (DSR) and the health check results of member instances can be monitored in real time.

<a id="status-information"></a>
### Status Information { #status-information }

Load Balancer (DSR) status

| Status | Description |
|------|------|
| `ACTIVE` | Operating normally |
| `BUILD` | Being created |
| `ERROR` | Error occurred |

Member Status

| Status | Description |
|------|------|
| `ACTIVE` | Health check passed; included in traffic distribution |
| `INACTIVE` | Health check failed or immediately after registration; excluded from traffic distribution |
| `ONLINE` | Member is manually disabled (`admin_state_up=false`) |

!!! tip "Note"
    The member status is automatically changed to `ACTIVE` or `INACTIVE` based on the health check result. A member that becomes `INACTIVE` due to a health check failure is automatically excluded from traffic distribution, and if the health check subsequently passes, the member transitions back to `ACTIVE` and resumes receiving traffic. A manually deactivated member is displayed as `ONLINE` and excluded from traffic distribution.

!!! tip "Note"
    Newly registered members start in the `INACTIVE` state. They automatically transition to `ACTIVE` after passing the health check.


<a id="caution"></a>
## Caution { #caution }

Note the following when using Load Balancer (DSR).

- Same subnet requirement: Load Balancer (DSR) and all member instances must be located in the same subnet.
- Protocol limitations: Load Balancer (DSR) operates at the L4 level and does not provide L7 features (such as HTTP header-based routing and SSL offloading), unlike a standard load balancer.
- Fragmented packet handling: Fragmented IP packets are dropped because the L4 header (port/flags) cannot be inspected, making consistent member mapping impossible. Configure the MTU of the client and member instances appropriately, or ensure Path MTU Discovery is functioning correctly to prevent fragmentation from occurring.
- Instance deletion: If an instance registered as a load balancer member is deleted, the member is automatically removed from the load balancer.
- VM live migration: When VM live migration is performed on a member instance, the network information is automatically updated internally. A temporary traffic interruption may occur during migration, but it is automatically restored upon completion.
- Routing method change: Changing the routing method (DVR ↔ CVR) of the router in use may cause a temporary communication interruption (within 1 minute).
- Resource cleanup on load balancer deletion: Deleting Load Balancer (DSR) releases all member registration information registered in that DSR (the instances themselves are not deleted). If a Floating IP is associated, it is automatically released.
