# Rocky Linux 9 vs Rocky Linux 10 비교

> DHCP(Kea), dnf5, RDP(xrdp), Python 3.12, nftables 강화 등 주요 변경 사항

## 버전 개요

| 항목 | Rocky Linux 9 | Rocky Linux 10 |
|------|-------------|--------------|
| 코드명 | Blue Onyx | Red Quartz |
| 출시일 | 2022년 7월 14일 | 2025년 6월 11일 |
| 커널 버전 | 5.14.0 계열 | 6.12.0 계열 |
| 최신 버전 | 9.7 | 10.1 |
| 활성 지원 종료 | 2027년 5월 31일 | 2030년 5월 31일 |
| EOL(수명 종료) | 2032년 5월 31일 | 2035년 5월 31일 |

**정리**: **Rocky Linux 10**은 2025년 6월 출시된 최신 버전으로 커널 6.12 계열을 사용하며, 9보다 지원 기간이 길다.

---

## 주요 변경 사항

### 아키텍처 변경

| 항목 | Rocky Linux 9 | Rocky Linux 10 |
|------|-------------|--------------|
| x86 아키텍처 | x86-64-v2 이상 지원 | **x86-64-v3 이상 필요** (v2 지원 중단) |
| 32비트 패키지 | 일부 지원 | **완전 제거** |
| RISC-V | 미지원 | **신규 지원** |

> **주의**: Rocky Linux 10은 구형 CPU(x86-64-v2 전용)에서 동작하지 않는다.

### 소프트웨어 버전 업데이트

| 소프트웨어 | Rocky Linux 9 | Rocky Linux 10 |
|---------|-------------|--------------|
| Python | 3.9 | **3.12** |
| PHP | 8.1 | **8.3** |
| Rust | - | **1.84.1** |
| Go | - | **1.23** |
| PostgreSQL | 13 | **16.8** |
| MySQL | 8.0 | **8.4** |
| MariaDB | 10.5 | **10.11** |
| nginx | 1.20 | **1.26** |
| GDB | - | **14.2** |
| Grafana | - | **10.2.6** |

### 데스크톱/디스플레이

| 항목 | Rocky Linux 9 | Rocky Linux 10 |
|------|-------------|--------------|
| 디스플레이 서버 | X.Org / Wayland | **Wayland 기본** (X.Org 제거) |
| 원격 데스크톱 기본 | VNC | **RDP** (Remote Desktop Protocol) |

### 네트워크

| 항목 | Rocky Linux 9 | Rocky Linux 10 |
|------|-------------|--------------|
| DHCP 클라이언트 | dhclient | **NetworkManager에 통합** |
| DHCP 서버 | ISC DHCP Server | **Kea DHCP Server**로 대체 |

### 보안

| 항목 | Rocky Linux 9 | Rocky Linux 10 |
|------|-------------|--------------|
| 빌드 시스템 | Peridot | **Koji** |
| IMA 서명 | - | **지원** (커널 수준 파일 무결성 검증) |

**정리**: 아키텍처 요건 상승(x86-64-v3), 최신 개발 스택, **Wayland**/**RDP** 전환, **Kea DHCP** 도입 등 Rocky 10은 전방위적으로 스택이 현대화됐다.

---

## 주요 차이점 요약

### Rocky Linux 10의 신규 기능
- **커널 6.12**: 최신 하드웨어 지원, 성능 향상
- **RISC-V 아키텍처 지원**: 임베디드/서버 시장 확장
- **Wayland 전환 완료**: X.Org 완전 제거
- **RDP 기본 원격 데스크톱**: VNC 대체
- **Kea DHCP**: ISC DHCP 서버 대체, 현대적 API 지원
- **IMA (Integrity Measurement Architecture)**: 커널 레벨 파일 서명 검증
- **최신 개발 스택**: Python 3.12, PHP 8.3, Rust 1.84.1 등

### Rocky Linux 10에서 제거된 기능
- x86-64-v2 CPU 지원 (v3 이상만 지원)
- 모든 32비트(i686) 패키지
- X.Org 디스플레이 서버
- ISC DHCP Server

**정리**: 신규 기능은 성능·보안·최신 스택 강화에 집중되어 있고, 제거된 기능은 구형 CPU와 레거시 컴포넌트를 정리하는 방향이다.

---

## 지원 수명 주기

```
Rocky Linux 8   (2021 ~ 2029)  ══════════════════════════╗
Rocky Linux 9   (2022 ~ 2032)    ══════════════════════════════════════╗
Rocky Linux 10  (2025 ~ 2035)          ════════════════════════════════════════════
```

| 기간 | 지원 내용 |
|------|---------|
| 출시 ~ 활성 지원 종료 | 새 기능 업데이트 + 보안 패치 |
| 활성 지원 종료 ~ EOL | 보안 패치만 |
| EOL 이후 | 지원 없음 |

**정리**: 세 버전이 겹치는 지원 기간을 유지하며 순차적으로 EOL을 맞이하므로, 업그레이드 시점은 활성 지원 종료일을 기준으로 계획해야 한다.

---

## 마이그레이션 고려사항

Rocky Linux 9 → Rocky Linux 10 업그레이드 시 확인 사항:

1. **CPU 확인**: `grep -m1 flags /proc/cpuinfo | grep -o 'avx2'` 결과 있어야 함 (x86-64-v3 요건)
2. **32비트 패키지**: 의존성 제거 필요
3. **DHCP 서버**: ISC dhcpd 설정을 Kea로 마이그레이션
4. **X.Org 앱**: Wayland 호환 버전으로 교체
5. **원격 데스크톱**: VNC 설정을 RDP(xrdp)로 전환

**정리**: CPU 요건, 32비트 의존성, DHCP·디스플레이·원격 접속 설정을 사전에 점검해야 업그레이드 후 서비스 중단을 피할 수 있다.

---

## Rocky Linux 10 명령어 변경 사항

### DHCP 서버 — ISC dhcpd → Kea

Rocky 9까지는 `dhcp-server` 패키지(ISC dhcpd)를 사용했지만, Rocky 10부터는 **Kea DHCP**로 완전 대체됐다.

| 항목 | Rocky Linux 9 (ISC dhcpd) | Rocky Linux 10 (Kea) |
|------|--------------------------|----------------------|
| 패키지 | `dhcp-server` | `kea` |
| 설정 파일 | `/etc/dhcp/dhcpd.conf` | `/etc/kea/kea-dhcp4.conf` (JSON) |
| 서비스명 | `dhcpd` | `kea-dhcp4` |
| 서비스 시작 | `systemctl start dhcpd` | `systemctl start kea-dhcp4` |

```bash
# Rocky 9 — ISC dhcpd
dnf install -y dhcp-server
systemctl start dhcpd
systemctl enable dhcpd

# Rocky 10 — Kea DHCP
dnf install -y kea
systemctl start kea-dhcp4
systemctl enable kea-dhcp4
```

#### Kea 설정 파일 형식 (/etc/kea/kea-dhcp4.conf)

```json
{
    "Dhcp4": {
        "interfaces-config": {
            "interfaces": ["ens160"]
        },
        "subnet4": [{
            "subnet": "192.168.10.0/24",
            "pools": [{"pool": "192.168.10.150 - 192.168.10.200"}],
            "option-data": [
                {"name": "routers", "data": "192.168.10.2"},
                {"name": "domain-name-servers", "data": "192.168.10.2"}
            ]
        }]
    }
}
```

**정리**: **Kea DHCP**는 JSON 기반 설정 파일과 별도 서비스명(`kea-dhcp4`)을 사용하므로 기존 `dhcpd.conf`를 그대로 옮길 수 없고 새 형식으로 재작성해야 한다.

---

### 네트워크 설정 — nmcli / nmtui 강화

Rocky 10에서는 `ifconfig`, `route`, `netstat` 등 net-tools 계열 명령어가 기본 설치에서 완전히 제외됐다. `ip`, `ss` 계열 명령어를 사용해야 한다.

| 기존 명령어 (Rocky 9 이하) | 대체 명령어 (Rocky 10) |
|--------------------------|----------------------|
| `ifconfig` | `ip addr` / `ip a` |
| `ifconfig eth0 up/down` | `ip link set ens160 up/down` |
| `route -n` | `ip route` / `ip r` |
| `netstat -tuln` | `ss -tuln` |
| `netstat -an` | `ss -an` |
| `arp -n` | `ip neigh` |

```bash
# Rocky 10 기준 네트워크 명령어
ip addr                          # IP 주소 확인
ip link show                     # 인터페이스 상태
ip route show                    # 라우팅 테이블
ss -tuln                         # 리스닝 포트 확인
ss -s                            # 소켓 요약 통계

# net-tools가 필요하면 별도 설치
dnf install -y net-tools         # ifconfig, netstat 사용 가능
```

**정리**: net-tools 계열 명령어(`ifconfig`, `netstat` 등)는 기본 제거되었으므로 **ip**, **ss** 명령어 사용에 익숙해져야 하며, 필요 시 net-tools를 별도 설치할 수 있다.

---

### 원격 데스크톱 — VNC → RDP (xrdp)

Rocky 10에서는 RDP가 기본 원격 데스크톱 프로토콜이다.

| 항목 | Rocky Linux 9 | Rocky Linux 10 |
|------|-------------|--------------|
| 기본 프로토콜 | VNC | **RDP** |
| 패키지 | `tigervnc-server` | `xrdp` |
| 포트 | 5901 (VNC) | 3389 (RDP) |
| 서비스명 | `vncserver@:1` | `xrdp` |

```bash
# Rocky 9 — VNC 서버
dnf install -y tigervnc-server
vncpasswd
systemctl start vncserver@:1
firewall-cmd --permanent --add-port=5901/tcp

# Rocky 10 — xrdp (RDP)
dnf install -y xrdp
systemctl start xrdp
systemctl enable xrdp
firewall-cmd --permanent --add-port=3389/tcp
firewall-cmd --reload
```

**정리**: Rocky 10은 **xrdp** 패키지와 포트 3389를 기본으로 사용하므로, 기존 VNC(5901) 기반 원격 접속 스크립트/방화벽 규칙을 RDP 기준으로 전환해야 한다.

---

### Python 버전 기본값 변경

| 항목 | Rocky Linux 9 | Rocky Linux 10 |
|------|-------------|--------------|
| 기본 Python | `python3` → 3.9 | `python3` → **3.12** |
| pip | `pip3` | `pip3` (동일, 버전 업) |

```bash
# Rocky 9
python3 --version    # Python 3.9.x
pip3 --version

# Rocky 10
python3 --version    # Python 3.12.x

# 특정 버전 설치 (필요 시)
dnf install -y python3.11
python3.11 --version
```

**정리**: 기본 `python3`가 **3.12**로 상향되므로, 이전 버전에 의존하는 스크립트는 특정 버전 패키지를 병행 설치해 호환성을 확보해야 한다.

---

### 패키지 관리 — dnf 변경 사항

Rocky 10에서는 `dnf5`가 기본 패키지 관리자로 전환됐다. 기존 `dnf` 명령어는 대부분 호환되지만 일부 플러그인·출력 형식이 다르다.

| 항목 | Rocky Linux 9 | Rocky Linux 10 |
|------|-------------|--------------|
| 패키지 관리자 | dnf (libdnf) | **dnf5** (libdnf5) |
| 명령어 호환성 | — | 대부분 동일 |
| 모듈스트림 | `dnf module` | `dnf5 module` (일부 변경) |

```bash
# 대부분 동일하게 사용 가능
dnf install -y httpd
dnf remove -y httpd
dnf update -y
dnf search nginx

# dnf5 전용 명령 예시
dnf5 install httpd
dnf5 history                 # 설치 이력 확인
dnf5 repo list               # 저장소 목록 (출력 형식 변경)
```

**정리**: **dnf5**는 대부분 기존 `dnf` 명령어와 호환되지만 일부 플러그인과 출력 형식이 다르므로 자동화 스크립트의 출력 파싱 로직은 재검증이 필요하다.

---

### 방화벽 — firewalld 동일, nftables 백엔드 강화

Rocky 10에서도 `firewalld` 명령어는 동일하지만 내부 백엔드가 완전히 nftables로 전환됐다 (Rocky 9도 nftables 사용이지만 iptables 호환 레이어 제공). iptables 직접 명령어는 더 이상 권장하지 않는다.

```bash
# Rocky 9/10 공통 — firewalld 명령어 동일
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload
firewall-cmd --list-all

# iptables는 Rocky 10에서 사용 비권장 (nftables 직접 사용 가능)
nft list ruleset             # 현재 nftables 규칙 확인
```

**정리**: `firewall-cmd` 명령어 자체는 동일하게 유지되지만 내부적으로 **nftables** 백엔드가 강화되었으므로 iptables 직접 조작은 지양하고 `nft` 명령어를 병행 활용하는 것이 좋다.

---

## 참고 링크

- Rocky Linux 공식 사이트: https://rockylinux.org
- Rocky Linux 10 출시 발표: https://rockylinux.org/news/rocky-linux-10-0-ga-release/
- 버전 정보: https://wiki.rockylinux.org/rocky/version/
- 문서: https://docs.rockylinux.org
