<!-- pre-align:aligned sig=9be9447442b4 -->

<a id="network-load-balancer-dsr-overview"></a>
## Network > Load Balancer(DSR) > 개요 { #network-load-balancer-dsr-overview }

NHN Cloud는 DSR(direct server return) 방식의 로드 밸런서를 제공합니다. 로드 밸런서(DSR)를 이용하면,

- 인스턴스 하나로 처리하기 힘든 부하를 여러 대의 인스턴스로 분산하여 처리량을 늘릴 수 있습니다.
- 장애가 발생했거나 점검 중인 인스턴스를 자동으로 서비스에서 제외하여 가용성을 높일 수 있습니다.
- 서버 응답 트래픽이 로드 밸런서를 거치지 않고 직접 클라이언트로 전송되어 높은 성능을 제공합니다.


<a id="dsr-method"></a>
## DSR(direct server return) 방식 { #dsr-method }

로드 밸런서(DSR)는 일반 로드 밸런서와 다른 트래픽 처리 방식을 사용합니다.

<a id="differences-from-a-standard-load-balancer-proxy-mode"></a>
### 일반 로드 밸런서(프록시 모드)와 차이점 { #differences-from-a-standard-load-balancer-proxy-mode }

| 구분 | 일반 로드 밸런서(프록시 모드) | 로드 밸런서(DSR) |
|------|------|------|
| 클라이언트 → 서버 | 로드 밸런서 경유 | 로드 밸런서 경유 |
| 서버 → 클라이언트 | 로드 밸런서 경유 | 로드 밸런서를 거치지 않고 직접 전송 |
| 원본 IP 확인 | `X-Forwarded-For` 헤더 또는 Proxy Protocol 필요 | 클라이언트 IP 직접 확인 가능 |
| 로드 밸런서 부하 | 요청/응답 모두 처리 | 요청만 처리 |
| 처리 성능 | 보통 | 매우 높음 |
| 프로토콜 지원 | HTTP, HTTPS, TERMINATED_HTTPS, TCP | TCP, UDP |

<a id="how-dsr-works"></a>
### DSR 방식의 동작 원리 { #how-dsr-works }

로드 밸런서(DSR)의 VIP(virtual IP)는 로드 밸런서가 속한 서브넷에서 할당되는 **사설 IP**로, 로드 밸런서(DSR) 생성 시 **VIP(Virtual IP)** 항목에 지정하는 주소이며 목록의 **IP 주소** 열에 표시됩니다. 인터넷에서 접근할 때 사용하는 공인 IP(Floating IP, 이하 FIP)와는 별개의 주소입니다.

| 구분 | 주소 예시 | 역할 | 멤버 서버의 lo 및 추가 허용 주소 설정 |
|------|----------|------|------|
| VIP(사설 IP) | `192.168.1.100` | 로드 밸런서와 멤버 인스턴스로 전달되는 트래픽의 실제 목적지 | 필요 |
| FIP(공인 IP) | `133.186.0.31` | 인터넷에서 접근하는 진입점 | 불필요 |

트래픽은 다음 순서로 처리됩니다.

1. 클라이언트 요청: 클라이언트가 로드 밸런서의 VIP(사설 IP)로 요청을 전송합니다. 인터넷에서 접근하는 경우에는 VIP에 연결된 FIP로 요청을 보내며, 이 요청은 목적지가 VIP로 변환되어 로드 밸런서로 전달됩니다.
2. 요청 분산: 로드 밸런서가 적절한 멤버 인스턴스를 선택하여, 목적지 IP를 VIP로 유지한 채 요청을 전달합니다.
3. 응답 직접 전송: 멤버 인스턴스가 VIP를 출발지로 하여 로드 밸런서를 거치지 않고 클라이언트에게 직접 응답을 전송합니다. FIP로 유입된 요청의 응답은 출발지가 FIP로 변환되어 클라이언트에 도달합니다.

!!! danger "주의"
    멤버 서버의 lo 인터페이스와 네트워크 인터페이스의 추가 허용 주소에 등록해야 하는 주소는 VIP(사설 IP)입니다. FIP(공인 IP)를 등록하면 트래픽이 정상적으로 처리되지 않습니다.

!!! tip "알아두기"
    DSR 방식은 응답 트래픽이 로드 밸런서를 거치지 않으므로 다음과 같은 장점이 있습니다:

    - 로드 밸런서의 부하가 크게 감소하여 더 많은 동시 연결을 처리할 수 있습니다.
    - 응답 데이터가 큰 서비스(동영상 스트리밍, 대용량 파일 다운로드 등)에 특히 유리합니다.
    - 네트워크 지연 시간이 감소하여 응답 속도가 향상됩니다.
    - 클라이언트의 원본 IP를 서버에서 직접 확인할 수 있습니다.


<a id="session-affinity"></a>
## 세션 지속성 { #session-affinity }

로드 밸런서(DSR)는 세션 지속성(Session persistence) 기능을 제공합니다. 이 기능을 활성화하면 동일한 클라이언트의 요청을 지속적으로 같은 멤버 인스턴스로 전달할 수 있습니다.

- 세션 지속성 비활성화: 5-tuple(소스 IP, 소스 포트, 목적지 IP, 목적지 포트, 프로토콜) 기반으로 멤버가 선택되어 트래픽이 분산됩니다. 동일한 소스 IP라도 소스 포트가 다르면 다른 멤버로 분산될 수 있습니다.
- 세션 지속성 활성화: 소스 IP 기반으로 동일한 클라이언트의 요청이 항상 같은 멤버로 전달됩니다. 소스 포트가 달라지더라도 소스 IP가 동일하면 같은 멤버가 선택됩니다.

멤버 선택은 별도의 sticky table을 두지 않고 해시 기반의 일관된 매핑(Consistent Hashing)으로 이루어집니다. 세션 지속성 비활성화 시에는 5-tuple을, 활성화 시에는 소스 IP를 해시 입력으로 사용하며, 멤버 구성이 같다면 동일한 입력 키는 항상 동일한 멤버로 매핑됩니다. 또한 한 번 수립된 세션이 유효한 동안에는 해당 세션의 모든 패킷이 동일한 멤버로 전달되어, 세션 단위의 멤버 고정이 보장됩니다.

세션 유지 방식은 프로토콜에 따라 다르게 동작합니다.

- TCP: 패킷의 TCP 플래그(FIN/RST)를 관찰하여 연결 종료를 감지하며, 종료 신호가 확인되면 세션을 짧은 만료 시간으로 빠르게 회수합니다. 그 외 트래픽이 이어지는 동안에는 세션이 유지됩니다.
- UDP: 연결 종료 신호가 없으므로 일정 시간 동안 추가 트래픽이 없으면 세션이 만료됩니다. 만료 전까지는 동일 플로우가 계속 같은 멤버로 전달됩니다.

!!! tip "알아두기"
    세션 지속성은 다음과 같은 경우에 유용합니다.

    - 사용자 로그인 세션을 각 서버에서 관리하는 경우
    - 인스턴스 간에 세션 동기화가 구현되지 않은 경우
    - 특정 사용자의 요청을 동일한 서버에서 처리해야 하는 요구사항이 있는 경우

!!! tip "알아두기"
    세션 지속성 설정은 운영 중에도 변경할 수 있습니다. 변경 시 기존에 수립된 TCP 연결 및 진행 중인 UDP 플로우에는 영향이 없으며, 새로운 연결/플로우부터 변경된 설정이 적용됩니다.


<a id="instance-health-check"></a>
## 인스턴스 상태 확인 { #instance-health-check }

로드 밸런서(DSR)는 멤버로 등록된 인스턴스가 정상적으로 동작하는지 주기적으로 상태 확인(Health check)을 수행합니다. 상태 확인은 지정된 프로토콜에 따라 정해진 응답이 오는지 확인하는 방식입니다. 지정된 횟수나 시간 내에 정상 응답이 오지 않으면 비정상 인스턴스로 간주하여 부하 분산 대상에서 제외합니다. 이 기능으로 예기치 못한 장애나 점검에도 중단 없이 서비스를 제공할 수 있습니다.

<a id="supported-protocols"></a>
### 지원 프로토콜 { #supported-protocols }

로드 밸런서(DSR)는 다음과 같은 상태 확인 프로토콜을 지원합니다.

- ICMP: ICMP Echo Request/Reply를 이용한 기본적인 연결성 확인 방식입니다. 인스턴스의 네트워크 연결 상태를 빠르게 확인할 수 있습니다. 요청은 멤버 인스턴스의 실제 IP를 목적지로 발송됩니다.

- TCP: 지정된 포트로 TCP 연결을 시도하여 연결 가능 여부를 확인합니다. 특정 서비스 포트가 정상적으로 동작하는지 확인할 수 있습니다. 요청은 로드 밸런서(DSR)의 VIP(사설 IP)를 목적지로 발송됩니다.

- HTTP: 지정된 경로로 HTTP 요청을 전송하여 응답 코드를 확인합니다. 웹 애플리케이션의 실제 서비스 상태를 보다 정확하게 확인할 수 있습니다. 요청은 로드 밸런서(DSR)의 VIP(사설 IP)를 목적지로 발송됩니다.

!!! tip "알아두기"
    TCP/HTTP 상태 확인 패킷의 목적지가 공인 IP(FIP)가 아닌 사설 VIP인 것은 정상 동작입니다. TCP/HTTP 상태 확인은 DSR VIP를 목적지로 요청하므로, 멤버 서버의 lo 인터페이스에 VIP가 설정되어 있지 않으면 해당 패킷이 수신·처리되지 못해 상태 확인이 실패하고 멤버가 `INACTIVE`로 판정됩니다. 이는 서버 측 VIP 설정 누락을 조기에 탐지하기 위한 동작입니다. ICMP 상태 확인은 멤버의 실제 IP로 요청하므로 VIP 설정과 무관하게 연결성만 확인합니다.

<a id="health-check-settings"></a>
### 상태 확인 설정 { #health-check-settings }

상태 확인을 위해서는 다음 항목을 설정해야 합니다.

| 항목 | 설명 | 비고 |
|------|------|------|
| 지연 시간(delay) | 상태 확인 요청을 보내는 주기(초)입니다. | - |
| 최대 재시도 횟수(max_retries) | 인스턴스를 비정상으로 판단하기까지 재시도할 최대 횟수입니다. | 1~10회 |
| 타임아웃(timeout) | 각 상태 확인 요청의 타임아웃 시간(초)입니다. 이 시간 내에 응답이 없으면 실패로 간주합니다. | - |
| 프로토콜(type) | 상태 확인에 사용할 프로토콜입니다. | ICMP, TCP, HTTP |
| 포트(health_check_port) | 상태 확인을 수행할 포트 번호입니다. | TCP, HTTP 사용 시 필수 |
| HTTP 경로(http_path) | HTTP 상태 확인 시 요청을 보낼 URL 경로입니다. | HTTP 사용 시 설정(기본값: `/`) |
| 기대 HTTP 응답 코드(expected_http_code) | HTTP 상태 확인 시 정상으로 판단할 응답 코드입니다. | HTTP 사용 시 설정(기본값: `200`) |

!!! danger "주의"
    지연 시간(delay)은 타임아웃(timeout)보다 크거나 같아야 합니다. 타임아웃이 지연 시간보다 크면 상태 확인이 정상적으로 동작하지 않을 수 있습니다.


<a id="create-load-balancer-dsr"></a>
## 로드 밸런서(DSR) 생성 { #create-load-balancer-dsr }

로드 밸런서(DSR)는 [VPC](/Network/VPC/ko/overview/#_2)의 [서브넷](/Network/VPC/ko/overview/#_2) 내에서 생성됩니다.

<a id="assign-vip-address"></a>
### VIP 주소 할당 { #assign-vip-address }

VIP(virtual IP)는 로드 밸런서(DSR)가 속한 서브넷에서 할당되는 사설 IP로, 로드 밸런서(DSR) 생성 시 다음 두 가지 방식으로 할당할 수 있습니다.

- 자동 할당: 서브넷의 가용 IP 중 하나를 자동으로 할당받아 VIP로 사용합니다.
- 직접 지정: 서브넷의 CIDR 범위 내에서 원하는 IP를 지정하여 VIP로 사용합니다.

인터넷에서 접근해야 하는 경우에는 이 VIP에 Floating IP(공인 IP)를 연결합니다. 자세한 내용은 [Floating IP 연결](#floating-ip-association)을 참고하세요.

!!! danger "주의"
    직접 지정한 VIP 주소가 서브넷의 CIDR 범위에 속하지 않으면 생성이 실패합니다. 반드시 해당 서브넷의 IP 범위 내에서 지정하세요.

<a id="register-member"></a>
### 멤버 등록 { #register-member }

로드 밸런서(DSR)는 인스턴스를 멤버로 등록하여 유입된 트래픽을 분배합니다. 멤버 등록 시 다음 사항을 준수해야 합니다.

- 서브넷 일치: 멤버 인스턴스의 포트는 로드 밸런서(DSR)와 동일한 서브넷에 속해야 합니다.
- 컴퓨트 인스턴스: 멤버는 반드시 컴퓨트 인스턴스여야 합니다. (`device_owner`가 `compute:` 접두사로 시작)
- SDN 지원: 멤버 포트는 SDN(software defined network) 환경에서 동작해야 합니다.

!!! danger "주의"
    하나의 로드 밸런서(DSR)에는 기본적으로 최대 30개의 멤버를 등록할 수 있습니다. 더 많은 멤버가 필요한 경우 별도 문의가 필요합니다.

!!! tip "알아두기"
    * 새로 등록된 멤버의 초기 상태는 `INACTIVE`입니다. 상태 확인을 통과하면 자동으로 `ACTIVE` 상태로 전환되어 트래픽을 수신하기 시작합니다.
    * 동일한 인스턴스 포트를 같은 로드 밸런서(DSR)에 중복으로 등록할 수 없습니다.
    * 멤버로 등록된 인스턴스가 트래픽을 정상적으로 수신·응답하려면 서버 내부에서 ARP 및 VIP 설정이 필요합니다. 자세한 내용은 아래 멤버 서버 설정 가이드 섹션을 참고하세요.

<a id="member-server-configuration-guide"></a>
## 멤버 서버 설정 가이드 { #member-server-configuration-guide }

로드 밸런서(DSR)는 클라이언트의 요청을 멤버 서버에 VIP(virtual IP, 사설 IP)를 목적지로 하여 전달합니다. 멤버 서버가 이 패킷을 정상적으로 수신하고 응답하려면, 서버 측에서 아래의 설정이 필요합니다. 아래 설정에 사용하는 `<VIP>`는 모두 이 사설 IP입니다.

!!! danger "주의"
    반드시 1단계(커널 파라미터) → 2단계(VIP 설정) 순서로 설정해야 합니다. 커널 파라미터를 설정하지 않고 VIP를 먼저 할당하면 로드 밸런서의 VIP와 ARP 충돌이 발생하여 네트워크 장애가 일어날 수 있습니다.

<a id="kernel-parameter-configuration-arp-ignoreannounce"></a>
### 1. 커널 파라미터 설정(ARP Ignore/Announce) { #kernel-parameter-configuration-arp-ignoreannounce }

네트워크 인터페이스에 VIP를 설정하기 전에 서버가 VIP에 대한 ARP 요청에 응답하지 않도록 커널 설정을 먼저 수행해야 합니다. 이 설정을 하지 않고 VIP를 할당하면 로드 밸런서의 VIP와 충돌이 발생하여 네트워크 장애가 일어날 수 있습니다.

<a id="kernel-parameter-configuration-arp-ignoreannounce-parameter-value-definitions"></a>
#### 설정값의 의미

| 파라미터 | 값 | 설명 |
|---------|---|------|
| `arp_ignore` | `1` | ARP 요청을 받았을 때 해당 IP가 요청이 들어온 인터페이스에 설정되어 있는 경우에만 응답합니다. (lo에 설정된 VIP에 대해서는 응답하지 않게 됨) |
| `arp_announce` | `2` | 외부로 ARP 패킷을 보낼 때 소스 IP를 해당 인터페이스의 주소로 고정하여 VIP 주소를 유출하지 않습니다. |

<a id="kernel-parameter-configuration-arp-ignoreannounce-real-time-application"></a>
#### 실시간 적용

```bash
sudo sysctl -w net.ipv4.conf.all.arp_ignore=1
sudo sysctl -w net.ipv4.conf.all.arp_announce=2
sudo sysctl -w net.ipv4.conf.lo.arp_ignore=1
sudo sysctl -w net.ipv4.conf.lo.arp_announce=2
```

<a id="kernel-parameter-configuration-arp-ignoreannounce-permanent-application-etcsysctlconf"></a>
#### 영구 적용(/etc/sysctl.conf)

파일 끝에 아래 내용을 추가합니다.

```bash
sudo tee -a /etc/sysctl.conf <<EOF
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2
net.ipv4.conf.lo.arp_ignore = 1
net.ipv4.conf.lo.arp_announce = 2
EOF

# 설정 반영
sudo sysctl -p
```

!!! tip "알아두기"
    적용 후 `sysctl net.ipv4.conf.all.arp_ignore` 명령으로 값이 `1`로 설정되었는지 확인할 수 있습니다.

<a id="vip-configuration-on-loopback-interface"></a>
### 2. Loopback 인터페이스에 VIP 설정 { #vip-configuration-on-loopback-interface }

서버가 로드 밸런서로부터 전달된 패킷(목적지가 VIP인 패킷)을 자신의 패킷으로 인식할 수 있도록 lo 인터페이스에 VIP를 부여합니다.

<a id="vip-configuration-on-loopback-interface-temporary-configuration-deleted-on-reboot"></a>
#### 임시 설정(재부팅 시 삭제)

```bash
# <VIP> 부분을 실제 로드 밸런서 VIP 주소로 변경하세요.
sudo ip addr add <VIP>/32 dev lo
```

<a id="vip-configuration-on-loopback-interface-permanent-configuration"></a>
#### 영구 설정

##### Ubuntu 18.04 이상(Netplan)

`/etc/netplan/` 디렉터리 내의 설정 파일(예: `01-netcfg.yaml`)을 수정합니다.

!!! danger "주의"
    기존 인터페이스 설정을 유지하면서 lo 부분만 추가하거나 병합해야 합니다.

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    lo:
      addresses:
        - 127.0.0.1/8
        - <VIP>/32  # 로드 밸런서 VIP 추가
```

설정 적용:

```bash
sudo netplan apply
```

##### CentOS / RHEL 7 이상

`/etc/sysconfig/network-scripts/ifcfg-lo:0` 파일을 생성합니다.

```bash
sudo tee /etc/sysconfig/network-scripts/ifcfg-lo:0 <<EOF
DEVICE=lo:0
IPADDR=<VIP>
NETMASK=255.255.255.255
ONBOOT=yes
EOF
```

설정 적용:

```bash
sudo ifup lo:0
```

<a id="vip-configuration-on-loopback-interface-when-a-member-of-multiple-dsr-instances"></a>
#### 여러 DSR의 멤버인 경우

하나의 인스턴스가 여러 로드 밸런서(DSR)의 멤버로 등록된 경우, 각 VIP를 모두 lo 인터페이스에 추가하고 네트워크 인터페이스의 추가 허용 주소에도 각 VIP를 모두 등록해야 합니다.

```bash
sudo ip addr add <VIP_1>/32 dev lo
sudo ip addr add <VIP_2>/32 dev lo
```

Netplan 영구 설정 예시:

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
### 3. 설정 확인 및 테스트 { #configuration-verification-and-testing }

멤버가 `INACTIVE` 상태이거나 트래픽을 수신하지 못하는 경우, 멤버 인스턴스마다 아래 항목을 순서대로 점검합니다. 설정 누락 항목은 멤버마다 다를 수 있으므로 모든 멤버를 각각 확인해야 합니다.

| 순서 | 점검 항목 | 확인 방법 | 정상 상태 |
|------|----------|----------|----------|
| 1 | lo 인터페이스의 VIP | `ip addr show lo` | VIP(사설 IP)가 `/32`로 등록됨 |
| 2 | 커널 파라미터 | `sysctl net.ipv4.conf.{all,lo}.arp_ignore`, `sysctl net.ipv4.conf.{all,lo}.arp_announce` | `arp_ignore=1`, `arp_announce=2` |
| 3 | 추가 허용 주소 | 콘솔 **Network > Network Interface**의 **추가 허용 주소** | VIP(`<VIP>/32`) 또는 VIP를 포함하는 대역이 등록됨 |
| 4 | Security Groups | 멤버 인스턴스의 Security Groups 규칙 | 서비스 포트와 상태 확인 포트가 모두 허용됨(아래 [Security Groups 설정](#security-groups-configuration) 참고) |
| 5 | 애플리케이션 바인딩 | `ss -ltnp` 등 | `0.0.0.0` 또는 VIP를 수신 대기 |

<a id="configuration-verification-and-testing-verify-ip-configuration"></a>
#### IP 설정 확인

lo 인터페이스에 VIP가 `/32` 서브넷으로 정상 등록되었는지 확인합니다.

```bash
ip addr show lo
```

출력 예시:

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536
    inet 127.0.0.1/8 scope host lo
    inet 192.168.1.100/32 scope host lo
```

<a id="configuration-verification-and-testing-verify-arp-response"></a>
#### ARP 응답 확인

외부(동일 서브넷의 다른 서버)에서 VIP로 ARP 요청을 보냈을 때 멤버 서버의 MAC 주소가 응답되지 않아야 합니다. (로드 밸런서의 MAC만 응답해야 정상)

```bash
# 동일 서브넷의 다른 인스턴스에서 실행
arping -c 3 <VIP>
```

응답되는 MAC 주소가 로드 밸런서의 MAC인지 확인합니다. 멤버 서버의 MAC이 응답되면 ARP 설정이 잘못된 것입니다.

<a id="configuration-verification-and-testing-verify-kernel-parameter"></a>
#### 커널 파라미터 확인

```bash
sysctl net.ipv4.conf.all.arp_ignore
sysctl net.ipv4.conf.all.arp_announce
sysctl net.ipv4.conf.lo.arp_ignore
sysctl net.ipv4.conf.lo.arp_announce
```

각각 `1`, `2`, `1`, `2`가 출력되어야 합니다.

<a id="service-configuration"></a>
### 4. 서비스 설정 { #service-configuration }

<a id="service-configuration-application-binding"></a>
#### 애플리케이션 바인딩

애플리케이션(Nginx, Apache, Tomcat 등) 설정 시 소켓이 `0.0.0.0`(Any) 또는 VIP를 수신 대기(Listen)하고 있어야 패킷을 수신할 수 있습니다.

| 바인딩 방식 | 설명 | 예시 |
|------------|------|------|
| `0.0.0.0:포트` | 모든 IP에서 수신(권장) | `listen 80;`(Nginx 기본값) |
| `<VIP>:포트` | VIP에서만 수신 | `listen 192.168.1.100:80;` |
| `<서버 IP>:포트` | 서버 자체 IP에서만 수신 — **VIP 트래픽 수신 불가** | `listen 10.0.0.5:80;` |

!!! danger "주의"
    애플리케이션이 서버의 실제 인터페이스 IP(예: `eth0`의 IP)에만 바인딩되어 있으면 VIP로 도착한 패킷을 수신할 수 없습니다. `0.0.0.0`으로 바인딩하거나, VIP 주소를 명시적으로 추가 바인딩해야 합니다.

<a id="service-configuration-response-to-health-check"></a>
#### 상태 확인 대응

로드 밸런서(DSR)는 멤버 서버에 주기적으로 상태 확인 요청을 보냅니다. 서버가 상태 확인에 정상 응답해야 `ACTIVE` 상태를 유지할 수 있습니다.

| 상태 확인 타입 | 서버 요구사항 |
|---------------|-------------|
| ICMP | ICMP Echo Request에 응답해야 합니다. 서버 내부 방화벽에서 ICMP를 차단하고 있다면 허용이 필요합니다. |
| TCP | 지정된 포트에서 TCP 연결을 수락해야 합니다. 해당 포트의 서비스가 기동 중이어야 합니다. |
| HTTP | 지정된 포트/경로에서 기대하는 HTTP 응답 코드(기본 200)를 반환해야 합니다. |

!!! tip "알아두기"
    * 상태 확인 요청은 로드 밸런서의 VIP가 아닌 상태 확인 전용 IP에서 발송됩니다. Security Group 및 서버 내부 방화벽에서 해당 IP의 트래픽을 허용해야 합니다. 상태 확인 전용 IP는 DSR과 동일한 서브넷에 자동으로 할당됩니다.
    * 서버에 내부 방화벽이 설정되어 있는 경우, 서비스 포트와 상태 확인 포트(ICMP 포함)가 차단되지 않는지 확인하세요.

<a id="security-groups-configuration"></a>
### 5. Security Groups 설정 { #security-groups-configuration }

멤버 인스턴스의 Security Groups에서 DSR로부터의 서비스 트래픽과 상태 확인 트래픽을 허용해야 합니다.

!!! tip "알아두기"
    로드 밸런서(DSR)의 VIP와 상태 확인 전용 IP에 해당하는 포트에는 default Security Group이 연결되어 있습니다. 다만, DSR 포트 자체에는 Security Groups의 필터링(flow)이 적용되지 않습니다.

    멤버 인스턴스의 포트에는 Security Groups 필터링이 정상 적용됩니다. 이때 DSR은 패킷의 소스 IP를 변환하지 않으므로(No SNAT), 서비스 트래픽의 소스 IP는 클라이언트의 원본 IP입니다. 따라서 서비스 트래픽에는 클라이언트 IP 대역 또는 ANY(`0.0.0.0/0`)를 원격으로 지정해야 합니다.

    상태 확인 트래픽은 DSR과 동일한 서브넷에 할당된 상태 확인 전용 IP에서 발송되므로, 해당 서브넷의 CIDR을 원격으로 지정하면 허용됩니다. 상태 확인 전용 IP의 포트는 default Security Group에도 속해 있어 `default` SG를 원격으로 지정하는 방법으로도 허용할 수 있습니다.

<a id="security-groups-configuration-method-1-easy-configuration"></a>
#### 방법 1: 간편 설정

DSR은 클라이언트의 소스 IP를 그대로 유지하므로, 서비스 트래픽과 상태 확인 트래픽을 각각 허용해야 합니다.

| 방향 | IP 프로토콜 | 포트 범위 | 원격 | 설명 |
|------|-----------|----------|------|------|
| 수신 | TCP 또는 UDP | 서비스 포트(예: 80) | 0.0.0.0/0 | 클라이언트로부터의 서비스 트래픽 허용(서비스 프로토콜에 맞게 지정) |
| 수신 | 임의 | - | DSR 서브넷 CIDR(예: `192.168.1.0/24`) | 상태 확인 트래픽 허용(상태 확인 전용 IP가 DSR과 동일한 서브넷에 할당됨) |

!!! tip "알아두기"
    DSR은 패킷의 소스 IP를 변환하지 않으므로, 멤버 서버에 도착하는 서비스 트래픽의 소스 IP는 클라이언트의 원본 IP입니다. 클라이언트 IP 대역이 특정되지 않는 경우 `0.0.0.0/0`으로 허용해야 합니다. 클라이언트 대역이 확정된 경우 해당 CIDR로 제한할 수 있습니다.

!!! danger "주의"
    상태 확인 트래픽의 원격으로 `default` Security Group을 지정할 수도 있으나, 이 경우 default SG에 속한 모든 리소스가 해당 포트로 접근할 수 있게 됩니다. 인스턴스는 별도로 지정하지 않으면 default SG에 포함되는 경우가 많아, 서브넷이나 용도(web/app/db 등)로 구분한 네트워크 경계가 Security Group 수준에서 느슨해질 수 있습니다. `default` SG 지정은 초기 테스트·검증 용도로만 사용하고, 운영 환경에서는 DSR 서브넷 CIDR을 지정하는 방식을 권장합니다.

<a id="security-groups-configuration-method-2-individual-rules-fine-grained-control"></a>
#### 방법 2: 개별 규칙으로 허용(세밀한 제어)

보안 정책상 최소 권한 원칙을 적용하거나, 특정 포트만 허용해야 하는 경우 개별 규칙을 추가합니다.

| 용도 | 프로토콜 | 포트 | 원격(권장) | 비고 |
|------|---------|------|------|------|
| 서비스 트래픽(TCP) | TCP | 서비스 포트(예: 80, 443) | 클라이언트 IP 대역 또는 0.0.0.0/0 | DSR은 SNAT하지 않으므로 소스 IP가 클라이언트 원본 IP |
| 서비스 트래픽(UDP) | UDP | 서비스 포트(예: 53, 514) | 클라이언트 IP 대역 또는 0.0.0.0/0 | UDP 프로토콜 서비스 사용 시 |
| TCP 상태 확인 | TCP | `health_check_port` | DSR 서브넷 CIDR | 상태 확인 전용 IP가 DSR과 동일한 서브넷에 할당됨 |
| ICMP 상태 확인 | ICMP | - | DSR 서브넷 CIDR | ICMP 타입 사용 시 |
| HTTP 상태 확인 | TCP | `health_check_port` | DSR 서브넷 CIDR | HTTP 타입 사용 시 |

##### 콘솔에서 규칙 추가 예시

서비스 포트 80, TCP 상태 확인 포트 80이고 DSR 서브넷이 `192.168.1.0/24`인 경우:

| 방향 | IP 프로토콜 | 포트 범위 | 원격 | 설명 |
|------|-----------|----------|------|------|
| 수신 | TCP | 80 | 0.0.0.0/0 | 클라이언트로부터의 서비스 트래픽 |
| 수신 | TCP | 80 | 192.168.1.0/24 | 상태 확인 트래픽 |

ICMP 상태 확인을 사용하는 경우 추가:

| 방향 | IP 프로토콜 | 포트 범위 | 원격 | 설명 |
|------|-----------|----------|------|------|
| 수신 | ICMP | - | 192.168.1.0/24 | ICMP 상태 확인 |

!!! tip "알아두기"
    * 상태 확인 포트(`health_check_port`)를 서비스 포트와 다르게 설정한 경우, Security Groups에서 두 포트를 모두 허용해야 합니다.
    * 클라이언트 IP 대역이 특정 CIDR(예: `10.0.0.0/8`)로 한정되어 있다면, `0.0.0.0/0` 대신 해당 CIDR을 지정하여 최소 권한 원칙을 적용할 수 있습니다.
    * 운영 환경에서는 default Security Group을 그대로 사용하기보다, 용도(web/app/db 등)별 전용 Security Group을 생성하여 필요한 규칙만 적용하는 방식을 권장합니다. 상태 확인 규칙의 원격도 `default` SG 대신 DSR 서브넷 CIDR로 지정하면, 서브넷·용도 단위의 네트워크 경계를 유지할 수 있습니다.

<a id="network-interface-security-settings-update"></a>
### 6. 네트워크 인터페이스 보안 설정 변경 { #network-interface-security-settings-update }

DSR 방식에서는 로드 밸런서가 패킷의 목적지 IP를 VIP로 유지한 채 멤버 서버에 전달합니다. NHN Cloud의 네트워크 환경에서는 보안을 위해 인스턴스에 할당된 IP가 아닌 다른 IP를 소스나 목적으로 하는 패킷을 기본적으로 차단합니다.

따라서 멤버 인스턴스가 VIP를 목적지로 하는 패킷을 수신하고, 다시 VIP를 소스로 하여 응답할 수 있도록 네트워크 인터페이스에 VIP를 추가 허용 주소로 추가해야 합니다.

<a id="network-interface-security-settings-update-reason-for-the-settings"></a>
#### 설정이 필요한 이유

```
[클라이언트] → dst: VIP → [로드 밸런서(DSR)] → dst: VIP → [멤버 서버]
                                                         ↑
                                         포트에 할당된 IP와 목적지(VIP)가 다름 → 추가 허용 주소 미등록 시 패킷 드롭
```

<a id="network-interface-security-settings-update-main-configuration-method"></a>
#### 주요 설정 방법

멤버 서버의 네트워크 인터페이스(Port)의 추가 허용 주소 항목에 로드 밸런서(DSR)의 VIP를 추가합니다.

* VIP를 추가 허용 주소로 등록하면 해당 IP를 소스 또는 대상으로 하는 패킷이 포트에서 허용됩니다. 포트 보안 전체를 비활성화하지 않고 필요한 VIP만 선택적으로 허용하므로 최소 권한 원칙에 부합합니다.
* 설정 위치: 콘솔의 **Network > Network Interface** 메뉴에서 해당 인터페이스 선택 후 **추가 허용 주소** 항목에 VIP 주소(`<VIP>/32`)를 추가합니다.
* 하나의 인스턴스가 여러 로드 밸런서(DSR)의 멤버인 경우, 각 VIP를 모두 추가 허용 주소에 추가해야 합니다.

!!! tip "알아두기"
    추가 허용 주소 설정 절차는 [콘솔 사용 가이드](/Network/Network%20Interface/ko/console-guide/)를 참고하세요.


<a id="floating-ip-association"></a>
## Floating IP 연결 { #floating-ip-association }

로드 밸런서(DSR)의 VIP에 Floating IP를 연결하여 외부 네트워크에서 접근할 수 있도록 설정할 수 있습니다.

- Floating IP를 연결하면 인터넷에서 로드 밸런서(DSR)로 트래픽을 전달할 수 있습니다.
- Floating IP를 분리하면 외부 접근이 차단되고 내부 네트워크에서만 접근 가능합니다.
- Floating IP 연결/분리 시 자동으로 로드 밸런서에 반영됩니다.

Floating IP로 유입된 요청은 목적지가 VIP(사설 IP)로 변환되어 로드 밸런서로 전달되고, 응답은 출발지가 다시 Floating IP로 변환됩니다. 따라서 멤버 서버의 lo 인터페이스와 네트워크 인터페이스의 추가 허용 주소에는 Floating IP가 아닌 VIP를 등록해야 합니다.

!!! tip "알아두기"
    Floating IP를 분리해도 내부 네트워크에서 VIP를 통한 접근은 영향을 받지 않습니다.


<a id="quota-and-limitations"></a>
## 쿼터 및 제한사항 { #quota-and-limitations }

로드 밸런서(DSR) 사용 시 다음과 같은 쿼터 및 제한사항이 적용됩니다:

| 항목 | 기본 제한 | 설명 |
|------|----------|------|
| 프로젝트당 로드 밸런서(DSR) 수 | 10개 | 프로젝트당 생성 가능한 로드 밸런서(DSR) 개수 |
| 로드 밸런서(DSR)당 멤버 수 | 30개 | 하나의 로드 밸런서(DSR)에 등록 가능한 멤버 개수 |
| 프로젝트당 멤버 수 | 제한 없음 | |

!!! tip "알아두기"
    기본 쿼터를 초과하여 사용해야 하는 경우 고객지원으로 문의하세요.


<a id="load-balancer-dsr-monitoring"></a>
## 로드 밸런서(DSR) 모니터링 { #load-balancer-dsr-monitoring }

로드 밸런서(DSR)의 상태와 멤버 인스턴스의 상태 확인 결과를 실시간으로 모니터링할 수 있습니다.

<a id="status-information"></a>
### 상태 정보 { #status-information }

로드 밸런서(DSR) 상태

| 상태 | 설명 |
|------|------|
| `ACTIVE` | 정상 동작 중 |
| `BUILD` | 생성 중 |
| `ERROR` | 오류 발생 |

멤버 상태

| 상태 | 설명 |
|------|------|
| `ACTIVE` | 상태 확인 성공, 트래픽 분산 대상 |
| `INACTIVE` | 상태 확인 실패 또는 신규 등록 직후, 트래픽 분산 대상에서 제외 |
| `ONLINE` | 멤버가 수동으로 비활성화된 상태(`admin_state_up=false`) |

!!! tip "알아두기"
    멤버의 상태는 상태 확인 결과에 따라 자동으로 `ACTIVE` 또는 `INACTIVE`로 변경됩니다. 상태 확인 실패로 `INACTIVE` 상태가 된 멤버는 자동으로 트래픽 분산 대상에서 제외되며, 이후 상태 확인에 성공하면 다시 `ACTIVE` 상태로 전환되어 트래픽을 수신합니다. 멤버를 수동으로 비활성화하면 `ONLINE` 상태로 표시되며 트래픽 분산 대상에서 제외됩니다.

!!! tip "알아두기"
    새로 등록된 멤버는 `INACTIVE` 상태로 시작합니다. 상태 확인 통과 후 자동으로 `ACTIVE`로 전환됩니다.


<a id="caution"></a>
## 주의 사항 { #caution }

로드 밸런서(DSR) 사용 시 다음 사항에 주의하세요.

- 동일 서브넷 요구사항: 로드 밸런서(DSR)와 모든 멤버 인스턴스는 동일한 서브넷에 위치해야 합니다.
- 프로토콜 제한: 로드 밸런서(DSR)는 L4 레벨에서 동작하며, 일반 로드 밸런서와 달리 L7 기능(HTTP 헤더 기반 라우팅, SSL Offloading 등)을 제공하지 않습니다.
- IP 단편화 패킷 처리: IP 단편화(fragmentation)된 패킷은 L4 헤더(포트/플래그)를 확인할 수 없어 일관된 멤버 매핑이 불가능하므로 드롭됩니다. 단편화가 발생하지 않도록 클라이언트와 멤버 인스턴스의 MTU를 적절히 설정하거나, Path MTU Discovery가 정상 동작하도록 구성하세요.
- 인스턴스 삭제: 로드 밸런서 멤버로 등록된 인스턴스를 삭제하면, 해당 멤버는 자동으로 로드 밸런서에서 제거됩니다.
- VM 라이브 마이그레이션: 멤버 인스턴스에 대해 VM 라이브 마이그레이션을 수행하면, 내부적으로 네트워크 정보가 자동 갱신됩니다. 마이그레이션 중 일시적인 트래픽 단절이 발생할 수 있으나, 완료 후 자동으로 복구됩니다.
- 라우팅 방식 변경: 사용 중인 라우터의 라우팅 방식(DVR ↔ CVR)을 변경하면 일시적인 통신 단절(1분 이내)이 발생할 수 있습니다.
- 로드 밸런서 삭제 시 리소스 정리: 로드 밸런서(DSR)를 삭제하면 해당 DSR에 등록된 멤버 등록 정보가 모두 해제됩니다(인스턴스 자체는 삭제되지 않습니다). Floating IP가 연결된 경우 자동으로 해제됩니다.
