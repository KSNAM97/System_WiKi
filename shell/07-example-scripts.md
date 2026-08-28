# Shell-07 실습 문제

## 목차

1. [개요](#개요)
2. [문제 1) Rocky Linux 9 필수 패키지 & 셸 환경 일괄 설정 스크립트](#문제-1-rocky-linux-9-필수-패키지--셸-환경-일괄-설정-스크립트)
3. [문제 2) .vimrc + 안전 alias 배포 스크립트](#문제-2-vimrc--안전-alias-배포-스크립트)
4. [문제 3) guest 계정 dotfile 배포 재실행 스크립트](#문제-3-guest-계정-dotfile-배포-재실행-스크립트)

## 개요

앞선 SH-01 ~ SH-06 문서에서 다룬 변수, 조건문, 반복문, 함수, 배열 등의 문법을 실무 스크립트 작성에 어떻게 적용하는지 확인하는 실습 문제다. 각 문제는 요구사항을 먼저 제시하고, 실제 운영 환경(Rocky Linux 9)에서 사용 중인 정답 스크립트를 이어서 제공한다. `set -euo pipefail`, 함수 분리, 배열, `case`, 반복문, heredoc, 로그 함수, 색상 코드 등 이 챕터에서 다룬 개념이 두루 사용된다.

## 문제 1) Rocky Linux 9 필수 패키지 & 셸 환경 일괄 설정 스크립트

**요구사항**

SRE/시스템 엔지니어에게 필요한 CLI 도구를 카테고리별로 일괄 설치하고, root와 모든 일반 계정(UID 1000 이상) 및 `/etc/skel`(신규 계정 템플릿)에 동일한 셸 환경(`.bashrc`, `.vimrc`, starship 프롬프트, Oh My Bash)을 배포하는 스크립트(`rl9_setup.sh`)를 작성하라. 다음 조건을 만족해야 한다.

- `set -euo pipefail`로 에러 발생 시 즉시 종료, 미정의 변수 참조 시 종료, 파이프라인 중간 실패도 감지할 것
- 색상 코드(`\033[...]`)를 적용한 `log_info/log_ok/log_warn/log_err/log_step` 로그 함수를 둘 것
- 한글(CJK)이 섞인 문자열의 터미널 표시 폭을 계산하는 `str_width()` 함수를 만들 것 (UTF-8 바이트 수와 문자 수의 차이 이용)
- `top`, `free`, `df`를 파싱해 CPU/RAM/SWAP/DISK 사용률을 프로그레스 바로 보여주는 `show_resources()`를 만들 것
- `dnf`, `dnf_multi`, `dnf_group`, `dnf_module`, `pip`, `manual` 6가지 설치 방식을 인자로 받아 분기 처리하는 공통 설치 함수 `install_pkg()`를 만들고, 배열 `INSTALLED`/`FAILED`/`RETRY_LIST`에 결과를 누적할 것
- 빌드 도구, 모니터링, 네트워크, 스토리지, 언어 런타임, 컨테이너, IaC, 로그 처리, 보안, CLI 생산성, Python 생태계 11개 카테고리의 패키지를 `install_pkg` 호출로 설치하는 `install_all_packages()`를 작성할 것
- heredoc으로 starship 프롬프트 설정과 `.vimrc`를 생성하는 `write_starship_toml()`/`write_vimrc()`, 그리고 이를 대상 계정에 배포하는 `configure_starship()`/`configure_vimrc()`를 작성할 것 (기존 파일은 날짜 백업 후 배포)
- Oh My Bash를 비대화형으로 설치하고 `.bashrc`의 `OSH_THEME`/`plugins` 설정을 `sed`로 조정하는 `install_ohmybash()`/`configure_ohmybash()`를 작성할 것
- 안전 alias(`rm -i` 등), lsd alias, fzf 키바인딩, zoxide, starship init을 `.bashrc`에 순서대로 추가하는 `configure_bashrc()`를 작성하되, 각 블록은 마커 주석으로 중복 실행을 방지할 것
- `awk`로 `/etc/passwd`를 파싱해 root + UID 1000 이상 로그인 가능 계정만 추출하는 `list_target_users()`를 작성할 것
- `/etc/starship.toml`, `/etc/skel/.vimrc`, `/etc/skel/.bashrc`에도 동일 설정을 배포해 신규 계정에 자동 적용되도록 `configure_system_wide()`를 작성할 것
- 1차 설치 실패 패키지를 재시도하는 `retry_failed()`와, 설치 성공/실패 목록·설치율을 계산해 보여주는 `print_final_report()`를 작성할 것
- 위 함수들을 순서대로 호출하는 진입점 `main()`을 작성하고 스크립트 마지막에서 호출할 것

### 정답

```bash
#!/usr/bin/env bash
# =============================================================================
# Rocky Linux 9 - Essential Plugins Installer
# 목적    : SRE/시스템 엔지니어 필수 도구 일괄 설치 + 셸 환경 통일
# 권한    : sudo 필요
# 위험도  : 낮음 (패키지 설치 및 셸 설정 파일 배포)
# 적용범위: root + 모든 일반 계정(UID>=1000) + /etc/skel(신규 계정)
# =============================================================================

set -euo pipefail   # -e: 명령 실패 시 즉시 종료 / -u: 미정의 변수 참조 시 종료 / pipefail: 파이프라인 중 하나라도 실패하면 전체를 실패로 처리

# ─── 색상 정의 ────────────────────────────────────────────────────────────────
RED='\033[0;31m'; GREEN='\033[0;32m'; YELLOW='\033[1;33m'
CYAN='\033[0;36m'; BLUE='\033[0;34m'; MAGENTA='\033[0;35m'
BOLD='\033[1m'; DIM='\033[2m'; RESET='\033[0m'
BG_DARK='\033[48;5;235m'; WHITE='\033[1;37m'

# ─── 설치 결과 추적 ───────────────────────────────────────────────────────────
declare -a INSTALLED=()
declare -a FAILED=()
declare -a RETRY_LIST=()
PKG_COUNT=0

# ─── 유틸리티 ─────────────────────────────────────────────────────────────────
log_info()    { echo -e "  ${CYAN}[INFO]${RESET}  $*"; }
log_ok()      { echo -e "  ${GREEN}[  OK ]${RESET}  $*"; }
log_warn()    { echo -e "  ${YELLOW}[ WARN]${RESET}  $*"; }
log_err()     { echo -e "  ${RED}[FAIL ]${RESET}  $*"; }
log_step()    { echo -e "  ${MAGENTA}[STEP ]${RESET}  $*"; }

# 문자열의 터미널 표시 폭 계산 (한글/CJK는 2칸)
# UTF-8에서 ASCII=1바이트, 한글/CJK=3바이트인 성질을 이용
#   문자수 = ascii + cjk,  바이트수 = ascii + 3*cjk
#   → cjk = (바이트수 - 문자수) / 2,  표시폭 = 문자수 + cjk
str_width() {
    local s=$1 chars bytes
    chars=${#s}
    bytes=$(printf '%s' "$s" | LC_ALL=C wc -c)
    echo $(( chars + (bytes - chars) / 2 ))
}

print_banner() {
    clear
    echo -e "${BG_DARK}${WHITE}"
    echo "  ╔══════════════════════════════════════════════════════════════════╗"
    echo "  ║        Rocky Linux 9 — Essential Plugins Installer               ║"
    echo "  ║        SRE / System Engineer Setup Script  v2.0                  ║"
    echo "  ╚══════════════════════════════════════════════════════════════════╝"
    echo -e "${RESET}"
    echo -e "  ${DIM}시작 시각: $(date '+%Y-%m-%d %H:%M:%S')${RESET}"
    echo
}

section_header() {
    local title=$1 emoji=$2
    local tw pad
    tw=$(str_width "$title")
    pad=$(( 56 - tw ))
    (( pad < 0 )) && pad=0
    echo
    echo -e "${BOLD}${BLUE}  ╔══════════════════════════════════════════════════════════════╗${RESET}"
    echo -e "${BOLD}${BLUE}  ║  ${emoji}  ${title}$(printf '%*s' "$pad" '')║${RESET}"
    echo -e "${BOLD}${BLUE}  ╚══════════════════════════════════════════════════════════════╝${RESET}"
    echo
}

printf_bar() {
    local val=$1 max=$2 width=$3
    local filled=$(( val * width / max ))
    local empty=$(( width - filled ))
    printf '%0.s█' $(seq 1 "$filled" 2>/dev/null) || true
    printf '%0.s░' $(seq 1 "$empty"  2>/dev/null) || true
}

get_color() {
    local pct=$1
    if   [[ $pct -ge 90 ]]; then echo -n "${RED}"
    elif [[ $pct -ge 75 ]]; then echo -n "${YELLOW}"
    else                          echo -n "${GREEN}"
    fi
}

# ─── 실시간 시스템 리소스 모니터 ─────────────────────────────────────────────
show_resources() {
    local cpu_usage cpu_bar
    cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1 | tr -d ' ')
    cpu_usage=${cpu_usage%%.*}
    cpu_bar=$(printf_bar "${cpu_usage}" 100 20)

    local mem_info mem_total mem_used mem_pct mem_bar
    mem_info=$(free -m)
    mem_total=$(echo "$mem_info" | awk '/^Mem:/{print $2}')
    mem_used=$(echo  "$mem_info" | awk '/^Mem:/{print $3}')
    mem_pct=$(( mem_used * 100 / mem_total ))
    mem_bar=$(printf_bar "${mem_pct}" 100 20)

    local swap_total swap_used swap_pct swap_bar
    swap_total=$(echo "$mem_info" | awk '/^Swap:/{print $2}')
    swap_used=$(echo  "$mem_info" | awk '/^Swap:/{print $3}')
    if [[ "$swap_total" -gt 0 ]]; then
        swap_pct=$(( swap_used * 100 / swap_total ))
    else
        swap_pct=0
    fi
    swap_bar=$(printf_bar "${swap_pct}" 100 20)

    local disk_info disk_used disk_total disk_pct disk_bar
    disk_info=$(df -h / | awk 'NR==2{print $3, $2, $5}')
    disk_used=$(echo "$disk_info" | awk '{print $1}')
    disk_total=$(echo "$disk_info" | awk '{print $2}')
    disk_pct=$(echo "$disk_info" | awk '{print $3}' | tr -d '%')
    disk_bar=$(printf_bar "${disk_pct}" 100 20)

    local cpu_color mem_color swap_color disk_color
    cpu_color=$(get_color  "${cpu_usage}")
    mem_color=$(get_color  "$mem_pct")
    swap_color=$(get_color "$swap_pct")
    disk_color=$(get_color "$disk_pct")

    echo -e "${BOLD}${CYAN}  ┌─ 시스템 리소스 모니터 ─────────────────────────────────────────┐${RESET}"
    printf "  ${CYAN}│${RESET}  %-6s ${cpu_color}[%-20s]${RESET} %3s%%  %-16s ${CYAN}│${RESET}\n" \
        "CPU" "$cpu_bar" "$cpu_usage" ""
    printf "  ${CYAN}│${RESET}  %-6s ${mem_color}[%-20s]${RESET} %3s%%  %s / %s MiB  ${CYAN}│${RESET}\n" \
        "RAM" "$mem_bar" "$mem_pct" "$mem_used" "$mem_total"
    printf "  ${CYAN}│${RESET}  %-6s ${swap_color}[%-20s]${RESET} %3s%%  %s / %s MiB  ${CYAN}│${RESET}\n" \
        "SWAP" "$swap_bar" "$swap_pct" "$swap_used" "$swap_total"
    printf "  ${CYAN}│${RESET}  %-6s ${disk_color}[%-20s]${RESET} %3s%%  %s / %s      ${CYAN}│${RESET}\n" \
        "DISK" "$disk_bar" "$disk_pct" "$disk_used" "$disk_total"
    echo -e "${CYAN}  └────────────────────────────────────────────────────────────────┘${RESET}"
    echo
}

# =============================================================================
# 패키지 설치
# =============================================================================
# 인자: "표시명" "패키지명" "설치방법(dnf|dnf_multi|dnf_group|dnf_module|pip|manual)" "설명"
install_pkg() {
    local display_name=$1
    local pkg_name=$2
    local method=$3
    local description=$4

    # ── 이미 설치 여부 사전 확인 ───────────────────────────────────────────
    local already=false
    case $method in
        dnf|dnf_multi)
            local all_installed=true
            for p in $pkg_name; do
                rpm -q "$p" &>/dev/null || { all_installed=false; break; }
            done
            $all_installed && already=true ;;
        dnf_group)
            rpm -q gcc &>/dev/null && already=true ;;
        dnf_module)
            local pkg_rest all_installed=true
            pkg_rest=$(echo "$pkg_name" | cut -d' ' -f2-)
            for p in $pkg_rest; do
                rpm -q "$p" &>/dev/null || { all_installed=false; break; }
            done
            $all_installed && already=true ;;
        pip)
            pip3 show "$pkg_name" &>/dev/null && already=true ;;
        manual)
            local bin_name="${pkg_name#install_}"
            command -v "$bin_name" &>/dev/null && already=true ;;
    esac

    if $already; then
        echo -e "  ${BOLD}▶ ${display_name}${RESET}  ${DIM}— ${description}${RESET}"
        echo -e "    ${CYAN}✔ 이미 설치됨 — 건너뜁니다${RESET}"
        INSTALLED+=("${display_name} (기설치)")
        (( PKG_COUNT++ )) || true
        return 0
    fi

    echo -e "  ${BOLD}▶ ${display_name}${RESET}  ${DIM}— ${description}${RESET}"

    local ok=false
    case $method in
        dnf)
            if sudo dnf install -y "$pkg_name" &>/dev/null; then ok=true; fi ;;
        dnf_multi)
            # shellcheck disable=SC2086
            if sudo dnf install -y $pkg_name &>/dev/null; then ok=true; fi ;;
        dnf_group)
            if sudo dnf groupinstall -y "$pkg_name" &>/dev/null; then ok=true; fi ;;
        dnf_module)
            local mod_stream pkg_rest
            mod_stream=$(echo "$pkg_name" | awk '{print $1}')
            pkg_rest=$(echo   "$pkg_name" | cut -d' ' -f2-)
            # shellcheck disable=SC2086
            if sudo dnf module enable -y "$mod_stream" &>/dev/null && \
               sudo dnf install -y $pkg_rest &>/dev/null; then ok=true; fi ;;
        pip)
            # 전역 설치 — 모든 계정이 공유
            if sudo pip3 install "$pkg_name" &>/dev/null; then ok=true; fi ;;
        manual)
            if "$pkg_name"; then ok=true; fi ;;
    esac

    if $ok; then
        echo -e "    ${GREEN}✔ 설치 완료${RESET}"
        INSTALLED+=("$display_name")
    else
        echo -e "    ${YELLOW}⚠ 설치 실패 — 건너뜁니다 (재시도 목록에 추가)${RESET}"
        FAILED+=("$display_name")
        RETRY_LIST+=("$display_name|$pkg_name|$method|$description")
    fi

    (( PKG_COUNT++ )) || true
    if (( PKG_COUNT % 5 == 0 )); then
        show_resources
    fi
}

# ─── EPEL / 저장소 활성화 ─────────────────────────────────────────────────────
enable_repos() {
    section_header "저장소 활성화" "📦"
    show_resources

    log_step "EPEL 9 저장소 설치 중..."
    if sudo dnf install -y epel-release &>/dev/null; then
        log_ok "EPEL 저장소 활성화 완료"
    else
        log_warn "EPEL 설치 실패 (계속 진행)"
    fi

    log_step "PowerTools (CRB) 활성화 중..."
    if sudo dnf config-manager --set-enabled crb &>/dev/null; then
        log_ok "CRB 활성화 완료"
    else
        log_warn "CRB 활성화 실패 (계속 진행)"
    fi

    log_step "패키지 목록 최신화 중..."
    sudo dnf makecache -q &>/dev/null && log_ok "캐시 갱신 완료"
}

install_all_packages() {

    # ── 1. 빌드 / 개발 기본 도구 ──────────────────────────────────────────────
    section_header "빌드 & 개발 기본 도구" "🔧"

    install_pkg "Development Tools (gcc, make 등)" \
        "Development Tools" dnf_group "C/C++ 컴파일러, make, binutils 등 핵심 빌드 도구"
    install_pkg "git"             "git"             dnf "버전 관리 시스템"
    install_pkg "git-lfs"         "git-lfs"         dnf "대용량 파일 Git 확장"
    install_pkg "curl / wget"     "curl wget"       dnf_multi "HTTP 클라이언트 도구"
    install_pkg "jq"              "jq"              dnf "JSON 파싱 CLI 도구"
    install_pkg "yq"              "yq"              dnf "YAML/JSON/XML 처리 CLI"
    install_pkg "tree"            "tree"            dnf "디렉터리 구조 시각화"
    install_pkg "unzip / zip"     "unzip zip"       dnf_multi "압축 파일 처리"
    install_pkg "rsync"           "rsync"           dnf "원격·로컬 파일 동기화"
    install_pkg "tmux"            "tmux"            dnf "터미널 멀티플렉서 (세션 유지)"
    install_pkg "screen"          "screen"          dnf "레거시 터미널 세션 관리"
    install_pkg "vim-enhanced"    "vim-enhanced"    dnf "향상된 Vim 에디터"
    install_pkg "nano"            "nano"            dnf "초보자 친화적 텍스트 에디터"
    install_pkg "bash-completion" "bash-completion" dnf "Bash 자동완성 확장"

    # ── 2. 시스템 모니터링 & 성능 분석 ───────────────────────────────────────
    section_header "시스템 모니터링 & 성능 분석" "📊"

    install_pkg "htop"      "htop"      dnf "대화형 프로세스 모니터 (top 대체)"
    install_pkg "btop"      "btop"      dnf "풍부한 UI의 리소스 모니터"
    install_pkg "glances"   "glances"   dnf "시스템 전반 실시간 모니터링"
    install_pkg "iotop"     "iotop"     dnf "디스크 I/O 프로세스별 모니터링"
    install_pkg "iftop"     "iftop"     dnf "네트워크 인터페이스 대역폭 모니터"
    install_pkg "nload"     "nload"     dnf "네트워크 트래픽 실시간 그래프"
    install_pkg "dstat"     "dstat"     dnf "시스템 리소스 통계 (vmstat/iostat 통합)"
    install_pkg "sysstat (iostat, sar, mpstat)" \
                            "sysstat"   dnf "I/O 통계 및 시스템 성능 이력 수집"
    install_pkg "perf"      "perf"      dnf "Linux 커널 성능 분석 도구"
    install_pkg "atop"      "atop"      dnf "프로세스·자원 이력 기반 모니터"
    install_pkg "lsof"      "lsof"      dnf "열린 파일·소켓 목록 확인"
    install_pkg "strace"    "strace"    dnf "시스템 콜 추적"
    install_pkg "ltrace"    "ltrace"    dnf "라이브러리 콜 추적"

    # ── 3. 네트워크 진단 & 보안 ──────────────────────────────────────────────
    section_header "네트워크 진단 & 보안 도구" "🌐"

    install_pkg "net-tools (ifconfig, netstat)" "net-tools" dnf "레거시 네트워크 도구 모음"
    install_pkg "iproute2 (ss, ip)"    "iproute"        dnf "현대적 네트워크 관리 도구"
    install_pkg "nmap"                 "nmap"           dnf "포트 스캐닝 & 네트워크 탐색"
    install_pkg "tcpdump"              "tcpdump"        dnf "패킷 캡처 & 분석"
    install_pkg "traceroute"           "traceroute"     dnf "경로 추적"
    install_pkg "mtr"                  "mtr"            dnf "traceroute + ping 통합 진단"
    install_pkg "bind-utils (dig, nslookup)" "bind-utils" dnf "DNS 조회 도구"
    install_pkg "whois"                "whois"          dnf "도메인 등록 정보 조회"
    install_pkg "telnet"               "telnet"         dnf "포트 연결 테스트 (레거시)"
    install_pkg "nc (netcat)"          "nmap-ncat"      dnf "범용 네트워크 통신 도구"
    install_pkg "socat"                "socat"          dnf "양방향 데이터 스트림 릴레이"
    install_pkg "openssh-server"       "openssh-server" dnf "SSH 서버"
    install_pkg "fail2ban"             "fail2ban"       dnf "무차별 대입 공격 방어"

    # ── 4. 파일시스템 & 스토리지 ─────────────────────────────────────────────
    section_header "파일시스템 & 스토리지 관리" "💾"

    install_pkg "lvm2"          "lvm2"          dnf "논리 볼륨 관리자"
    install_pkg "parted"        "parted"        dnf "파티션 관리 도구"
    install_pkg "smartmontools" "smartmontools" dnf "디스크 S.M.A.R.T 상태 확인"
    install_pkg "ncdu"          "ncdu"          dnf "디스크 사용량 대화형 탐색기"
    install_pkg "fio"           "fio"           dnf "I/O 벤치마크 도구"
    install_pkg "hdparm"        "hdparm"        dnf "디스크 파라미터 조회/설정"
    install_pkg "xfsprogs"      "xfsprogs"      dnf "XFS 파일시스템 유틸리티"
    install_pkg "e2fsprogs"     "e2fsprogs"     dnf "ext2/3/4 파일시스템 유틸리티"
    install_pkg "nfs-utils"     "nfs-utils"     dnf "NFS 클라이언트/서버 유틸"

    # ── 5. 프로그래밍 언어 런타임 ────────────────────────────────────────────
    section_header "프로그래밍 언어 런타임" "🐍"

    install_pkg "Python 3.11" "python3.11 python3.11-devel" dnf_multi "Python 최신 안정 버전"
    install_pkg "pip3"        "python3-pip"        dnf "Python 패키지 관리자"
    install_pkg "Python 가상환경 (venv)" "python3-virtualenv" dnf "Python 가상 환경 도구"
    install_pkg "Node.js 20 LTS" "nodejs:20 nodejs npm" dnf_module "JavaScript 런타임 & npm"
    install_pkg "Go (golang)"    "golang"                dnf "Go 언어 컴파일러"
    install_pkg "Java 17 OpenJDK" "java-17-openjdk-devel" dnf "Java 개발 키트 (LTS)"
    install_pkg "Rust (rustc + cargo)" "rust cargo" dnf_multi "시스템 프로그래밍 언어 (전역 설치)"

    # ── 6. 컨테이너 & 오케스트레이션 ─────────────────────────────────────────
    section_header "컨테이너 & 오케스트레이션" "🐳"

    install_pkg "podman"          "podman"          dnf "데몬리스 컨테이너 런타임"
    install_pkg "podman-compose"  "podman-compose"  dnf "docker-compose 호환 도구"
    install_pkg "buildah"         "buildah"         dnf "OCI 컨테이너 이미지 빌드 도구"
    install_pkg "skopeo"          "skopeo"          dnf "컨테이너 이미지 검사·복사 도구"
    install_pkg "cri-tools (crictl)" "install_critools" manual "CRI 런타임 디버깅 도구"

    # ── 7. 자동화 & IaC ──────────────────────────────────────────────────────
    section_header "자동화 & IaC (Infrastructure as Code)" "⚙️"

    install_pkg "ansible"      "ansible"      dnf "에이전트리스 IT 자동화 도구"
    install_pkg "ansible-lint" "ansible-lint" pip "Ansible 플레이북 린터"
    install_pkg "make"         "make"         dnf "빌드 자동화 도구"
    install_pkg "parallel"     "parallel"     dnf "병렬 작업 실행 도구"

    # ── 8. 로그 & 텍스트 처리 ────────────────────────────────────────────────
    section_header "로그 처리 & 텍스트 분석" "📝"

    install_pkg "ripgrep (rg)" "ripgrep"   dnf "초고속 grep 대체 도구"
    install_pkg "bat"          "bat"       dnf "syntax highlighting 지원 cat 대체"
    install_pkg "fd-find"      "fd-find"   dnf "직관적 find 대체 도구"
    install_pkg "fzf (퍼지 검색)" "fzf"     dnf "대화형 퍼지 파인더"
    install_pkg "awk (gawk)"   "gawk"      dnf "텍스트 처리 스크립팅 언어"
    install_pkg "sed"          "sed"       dnf "스트림 텍스트 편집기"
    install_pkg "multitail"    "multitail" dnf "다중 로그 파일 동시 tailing"
    install_pkg "logrotate"    "logrotate" dnf "로그 자동 순환 관리"
    install_pkg "lnav (로그 내비게이터)" "lnav" dnf "로그 파일 분석·내비게이션 도구"

    # ── 9. 보안 & 감사 ───────────────────────────────────────────────────────
    section_header "보안 & 감사 도구" "🔐"

    install_pkg "audispd-plugins" "audispd-plugins" dnf "auditd 플러그인"
    install_pkg "audit"           "audit"           dnf "Linux 감사 프레임워크"
    install_pkg "aide"            "aide"            dnf "파일 무결성 검사 도구"
    install_pkg "lynis"           "lynis"           dnf "시스템 보안 감사 스캐너"
    install_pkg "openssl"         "openssl"         dnf "TLS/SSL 암호화 도구"
    install_pkg "gnupg2"          "gnupg2"          dnf "GPG 암호화 & 서명 도구"
    install_pkg "rkhunter"        "rkhunter"        dnf "루트킷 탐지 도구"

    # ── 10. 편의성 & CLI 생산성 ──────────────────────────────────────────────
    section_header "편의성 & CLI 생산성 향상" "✨"

    install_pkg "zsh"                     "zsh"                     dnf "고급 셸"
    install_pkg "zsh-autosuggestions"     "zsh-autosuggestions"     dnf "Zsh 명령어 자동 제안"
    install_pkg "zsh-syntax-highlighting" "zsh-syntax-highlighting" dnf "Zsh 실시간 구문 강조"
    install_pkg "starship (크로스셸 프롬프트)" "install_starship" manual "스마트 프롬프트"
    install_pkg "delta (git diff 시각화)" "git-delta" dnf "구문강조 기반 git diff 뷰어"
    install_pkg "glow (Markdown 렌더러)"  "install_glow" manual "터미널 Markdown 뷰어"
    install_pkg "tldr (man 요약)"         "tldr"     pip "명령어 사용법 간결 요약"
    install_pkg "thefuck (명령어 오류 수정)" "thefuck" pip "잘못 입력한 명령어 자동 교정"
    install_pkg "direnv (디렉터리 환경변수)" "install_direnv" manual "디렉터리별 환경변수 로드"
    install_pkg "zoxide (스마트 cd)"      "zoxide"   dnf "빈도 기반 디렉터리 이동"
    install_pkg "pv (파이프 진행률)"      "pv"       dnf "파이프라인 데이터 진행률"
    install_pkg "progress (cp/mv 진행률)" "install_progress" manual "Coreutils 진행률 모니터"
    install_pkg "neofetch (시스템 정보)"  "neofetch" dnf "시스템 정보 시각화"
    install_pkg "lsd (ls 대체)"           "lsd"      dnf "아이콘·색상 지원 ls 대체"

    # ── 11. Python 개발 생태계 ───────────────────────────────────────────────
    section_header "Python 개발 생태계" "🐍"

    install_pkg "black (Python 포매터)"   "black"    pip "PEP 8 기반 코드 자동 포맷"
    install_pkg "flake8 (Python 린터)"    "flake8"   pip "코드 스타일·오류 검사"
    install_pkg "mypy (타입 검사기)"      "mypy"     pip "Python 정적 타입 분석"
    install_pkg "pytest"                  "pytest"   pip "단위 테스트 프레임워크"
    install_pkg "httpie (HTTPie CLI)"     "httpie"   pip "사람 친화적 HTTP 클라이언트"
    install_pkg "rich (터미널 시각화)"    "rich"     pip "Python rich 출력 라이브러리"
    install_pkg "ptpython (고급 REPL)"    "ptpython" pip "구문강조·자동완성 Python 셸"
}

# =============================================================================
# Manual 설치 함수 (install_pkg의 manual 메서드에서 호출)
# =============================================================================
install_starship() {
    curl -sS https://starship.rs/install.sh | sudo sh -s -- --yes &>/dev/null
}

install_glow() {
    if command -v go &>/dev/null; then
        go install github.com/charmbracelet/glow@latest &>/dev/null
    else
        return 1
    fi
}

install_critools() {
    local repo_file="/etc/yum.repos.d/kubernetes.repo"
    if [[ ! -f "$repo_file" ]]; then
        sudo tee "$repo_file" > /dev/null << 'REPO'
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/repodata/repomd.xml.key
REPO
    fi
    sudo dnf install -y cri-tools 2>/dev/null
}

install_direnv() {
    local version url
    version=$(curl -sS "https://api.github.com/repos/direnv/direnv/releases/latest" \
        | grep '"tag_name"' | cut -d'"' -f4)
    url="https://github.com/direnv/direnv/releases/download/${version}/direnv.linux-amd64"
    sudo curl -sSL "$url" -o /usr/local/bin/direnv 2>/dev/null \
        && sudo chmod +x /usr/local/bin/direnv
}

install_progress() {
    local tmp
    tmp=$(mktemp -d)
    curl -sSL "https://github.com/Xfennec/progress/archive/refs/heads/master.tar.gz" \
        | tar -xz -C "$tmp" 2>/dev/null \
        && make -C "${tmp}/progress-master" -s 2>/dev/null \
        && sudo install -m755 "${tmp}/progress-master/progress" /usr/local/bin/progress
    rm -rf "$tmp"
}


# =============================================================================
# starship 설정
# =============================================================================
# 단일 원본(Single Source of Truth) — 모든 배포 경로가 이 함수를 사용
# 인자: 출력 경로
write_starship_toml() {
    local out=$1
    sudo mkdir -p "$(dirname "$out")"
    sudo tee "$out" > /dev/null << 'STARSHIP_TOML'
# rl9_setup: managed
add_newline = false
format = """
$username$hostname$directory$git_branch$git_status$cmd_duration
$character"""

[username]
show_always = true
style_user  = "bold red"
style_root  = "bold red"
format      = "[$user]($style)"

[hostname]
ssh_only = false
disabled = false
style    = "bold green"
format   = "[@$hostname]($style)"

[directory]
style             = "white"
truncation_length = 4
truncate_to_repo  = false
format            = ":[$path]($style)[$read_only]($read_only_style)"
read_only         = " 🔒"

[git_branch]
symbol = " "
style  = "bold purple"
format = " [$symbol$branch]($style)"

[git_status]
style     = "yellow"
format    = "[ $all_status$ahead_behind]($style)"

[cmd_duration]
min_time = 2000
format   = " [⏱$duration]($style)"
style    = "dimmed white"

[character]
success_symbol = "[>](bold cyan)"
error_symbol   = "[>](bold red)"
vicmd_symbol   = "[<](bold yellow)"

[docker_context]
disabled = true

[kubernetes]
disabled = true

[aws]
disabled = true

[memory_usage]
disabled = true
STARSHIP_TOML
}

# 인자: target_user  target_home
configure_starship() {
    local target_user=$1 target_home=$2

    if ! command -v starship &>/dev/null; then
        log_warn "[${target_user}] starship 미설치 — toml 설정 건너뜁니다"
        return 0
    fi

    local toml_file="${target_home}/.config/starship.toml"

    # 기존 파일이 있으면 백업 (스크립트가 만든 것은 제외)
    if [[ -f "$toml_file" ]] && ! grep -qF "# rl9_setup: managed" "$toml_file" 2>/dev/null; then
        local bak="${toml_file}.bak.$(date +%F)"
        cp -a "$toml_file" "$bak"
        log_info "[${target_user}] 기존 starship.toml 백업: $bak"
    fi

    log_step "[${target_user}] starship.toml 배포 중..."
    write_starship_toml "$toml_file"
    chown -R "${target_user}:${target_user}" "${target_home}/.config" 2>/dev/null || true
    log_ok "[${target_user}] starship.toml 배포 완료 → $toml_file"
}

# =============================================================================
# .vimrc
# =============================================================================
# 단일 원본 — 계정별 홈과 /etc/skel 양쪽이 이 함수를 사용
# 인자: 출력 경로
write_vimrc() {
    local out=$1
    sudo tee "$out" > /dev/null << 'VIMRC'
set nu
set autoindent
set paste
set nohlsearch
set background=dark

" 구문 강조 — termguicolors로 24비트 RGB 사용 (터미널 팔레트 영향 없음)
" darkblue는 vim 기본 포함. silent!로 부재 시에도 에러 없음
syntax on
set termguicolors
silent! colorscheme darkblue
VIMRC
}

# 인자: target_user  target_home
configure_vimrc() {
    local target_user=$1 target_home=$2
    local vimrc="${target_home}/.vimrc"

    if grep -qF "set nohlsearch" "$vimrc" 2>/dev/null; then
        log_info "[${target_user}] .vimrc 이미 설정됨 — 건너뜁니다"
        return 0
    fi

    if [[ -f "$vimrc" ]]; then
        cp -a "$vimrc" "${vimrc}.bak.$(date +%F)"
        log_info "[${target_user}] 기존 .vimrc 백업: ${vimrc}.bak.$(date +%F)"
    fi

    log_step "[${target_user}] .vimrc 배포 중..."
    write_vimrc "$vimrc"
    chown "${target_user}:${target_user}" "$vimrc" 2>/dev/null || true
    log_ok "[${target_user}] .vimrc 배포 완료 → $vimrc"
}

# =============================================================================
# Oh My Bash
# =============================================================================
# 인자: target_user  target_home
install_ohmybash() {
    local target_user=$1 target_home=$2

    if [[ -d "${target_home}/.oh-my-bash" ]]; then
        log_info "[${target_user}] Oh My Bash 이미 설치됨 — 건너뜁니다"
        return 0
    fi
    log_step "[${target_user}] Oh My Bash 설치 중 (비대화형)..."

    # 설치 스크립트를 파일로 받아서 실행한다.
    # `bash -c "$(curl ...)" -- --unattended` 형태는 OMB가 `--`를 인자로 인식해 실패한다.
    local installer="/tmp/omb-install.sh"
    if [[ ! -f "$installer" ]]; then
        if ! curl -fsSL \
            https://raw.githubusercontent.com/ohmybash/oh-my-bash/master/tools/install.sh \
            -o "$installer"; then
            log_warn "[${target_user}] OMB 설치 스크립트 다운로드 실패"
            return 1
        fi
        chmod 644 "$installer"
    fi

    # -H : HOME을 대상 계정 홈으로 설정 (없으면 호출자 HOME이 남아 설치 위치가 꼬임)
    local errlog="/tmp/omb-install-${target_user}.log"
    if sudo -u "$target_user" -H bash "$installer" --unattended >"$errlog" 2>&1; then
        log_ok "[${target_user}] Oh My Bash 설치 완료 → ${target_home}/.oh-my-bash"
        rm -f "$errlog"
    else
        log_warn "[${target_user}] Oh My Bash 설치 실패 — 로그: ${errlog}"
        tail -3 "$errlog" | sed 's/^/           /'
        return 1
    fi
}

# OMB가 생성한 .bashrc 조정
#   - OSH_THEME="" : starship이 프롬프트를 담당하므로 OMB 테마 비활성화
#                    (OMB에는 "none" 테마가 없어 지정 시 module not found 발생)
#   - plugins      : OMB에 실제 존재하는 플러그인만 사용
#                    (history, alias-completion은 존재하지 않음)
# 인자: target_user  target_home
configure_ohmybash() {
    local target_user=$1 target_home=$2
    local bashrc="${target_home}/.bashrc"
    local omb_dir="${target_home}/.oh-my-bash"

    if [[ ! -d "$omb_dir" ]]; then
        log_warn "[${target_user}] Oh My Bash 미설치 — OMB 설정 건너뜁니다"
        return 0
    fi

    if grep -qF "# rl9_setup: omb configured" "$bashrc" 2>/dev/null; then
        log_info "[${target_user}] OMB .bashrc 이미 설정됨 — 건너뜁니다"
        return 0
    fi

    log_step "[${target_user}] Oh My Bash .bashrc 설정 조정 중..."

    cp -a "$bashrc" "${bashrc}.bak.$(date +%F)"
    log_info "[${target_user}] 수정 전 .bashrc 백업: ${bashrc}.bak.$(date +%F)"

    sed -i 's/^OSH_THEME=.*/OSH_THEME=""/' "$bashrc"
    sed -i '/^plugins=(/,/^)/d' "$bashrc"
    sed -i '/^OSH_THEME=/a \
\
# Oh My Bash 플러그인 — SRE 권장 세트\
plugins=(\
  git\
  sudo\
  bashmarks\
  bash-preexec\
  progress\
)' "$bashrc"

    echo "# rl9_setup: omb configured" >> "$bashrc"
    chown "${target_user}:${target_user}" "$bashrc" 2>/dev/null || true

    log_ok "[${target_user}] OSH_THEME 비활성화, SRE 플러그인 활성화 완료"
    log_info "적용된 플러그인: git · sudo · bashmarks · bash-preexec · progress"
}

# =============================================================================
# .bashrc 환경 설정
# =============================================================================
# 인자: target_user  target_home
configure_bashrc() {
    local target_user=$1 target_home=$2
    local bashrc="${target_home}/.bashrc"

    # ── 안전 alias + vi→vim + 디렉터리 색상 ───────────────────────────────────
    if grep -qF "# >>> rl9_setup: safe aliases <<<" "$bashrc" 2>/dev/null; then
        log_info "[${target_user}] 안전 alias 이미 존재 — 건너뜁니다"
    else
        log_step "[${target_user}] .bashrc에 안전 alias 추가 중..."
        cat >> "$bashrc" << 'BASHRC_ALIAS'

# >>> rl9_setup: safe aliases <<<
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
alias vi='vim'

# 에디터 — alias는 셸에서만 동작하므로 외부 도구용으로 환경변수 지정
# (kubectl edit, crontab -e, git commit 등)
export EDITOR='vim'
export KUBE_EDITOR='vim'

export LS_COLORS='di=36'
# <<< rl9_setup: safe aliases >>>
BASHRC_ALIAS
        log_ok "[${target_user}] 안전 alias 추가 완료 (rm/cp/mv -i, vi→vim, EDITOR)"
    fi

    # ── lsd alias ─────────────────────────────────────────────────────────────
    if grep -qF "# >>> rl9_setup: lsd aliases <<<" "$bashrc" 2>/dev/null; then
        log_info "[${target_user}] lsd alias 이미 존재 — 건너뜁니다"
    elif command -v lsd &>/dev/null; then
        log_step "[${target_user}] .bashrc에 lsd alias 추가 중..."
        cat >> "$bashrc" << 'BASHRC_LSD'

# >>> rl9_setup: lsd aliases <<<
if command -v lsd &>/dev/null; then
    alias ls='lsd'
    alias ll='lsd -alF'
    alias la='lsd -A'
    alias lt='lsd --tree'
    alias l='lsd -l'
fi
# <<< rl9_setup: lsd aliases >>>
BASHRC_LSD
        log_ok "[${target_user}] lsd alias 추가 완료"
    else
        log_warn "[${target_user}] lsd 미설치 — alias 생략"
    fi

    # ── fzf key-bindings ──────────────────────────────────────────────────────
    if grep -qF "key-bindings.bash" "$bashrc" 2>/dev/null; then
        log_info "[${target_user}] fzf key-bindings 이미 존재 — 건너뜁니다"
    elif [[ -f /usr/share/fzf/shell/key-bindings.bash ]]; then
        log_step "[${target_user}] .bashrc에 fzf key-bindings 추가 중..."
        cat >> "$bashrc" << 'BASHRC_FZF'

# >>> rl9_setup: fzf <<<
[ -f /usr/share/fzf/shell/key-bindings.bash ] && \
    source /usr/share/fzf/shell/key-bindings.bash
# <<< rl9_setup: fzf >>>
BASHRC_FZF
        log_ok "[${target_user}] fzf key-bindings 추가 완료"
    else
        log_warn "[${target_user}] fzf key-bindings 파일 없음 — 생략"
    fi

    # ── zoxide ────────────────────────────────────────────────────────────────
    if grep -qF "zoxide init bash" "$bashrc" 2>/dev/null; then
        log_info "[${target_user}] zoxide init 이미 존재 — 건너뜁니다"
    elif command -v zoxide &>/dev/null; then
        log_step "[${target_user}] .bashrc에 zoxide init 추가 중..."
        cat >> "$bashrc" << 'BASHRC_ZOXIDE'

# >>> rl9_setup: zoxide <<<
eval "$(zoxide init bash)"
# <<< rl9_setup: zoxide >>>
BASHRC_ZOXIDE
        log_ok "[${target_user}] zoxide init 추가 완료"
    else
        log_warn "[${target_user}] zoxide 미설치 — 생략"
    fi

    # ── starship (반드시 마지막) ──────────────────────────────────────────────
    if grep -qF "starship init bash" "$bashrc" 2>/dev/null; then
        log_info "[${target_user}] starship init 이미 존재 — 건너뜁니다"
    elif command -v starship &>/dev/null; then
        log_step "[${target_user}] .bashrc에 starship init 추가 중 (마지막 줄)..."
        cat >> "$bashrc" << 'BASHRC_STARSHIP'

# >>> rl9_setup: starship <<<
# starship 프롬프트 — 반드시 .bashrc 마지막에 위치
eval "$(starship init bash)"
# <<< rl9_setup: starship >>>
BASHRC_STARSHIP
        log_ok "[${target_user}] starship init 추가 완료"
    else
        log_warn "[${target_user}] starship 미설치 — 생략"
    fi

    chown "${target_user}:${target_user}" "$bashrc" 2>/dev/null || true
    echo
}

# =============================================================================
# 계정별 환경 설정
# =============================================================================
# 인자: target_user  target_home
apply_user_config() {
    local target_user=$1 target_home=$2

    section_header "사용자 환경 설정: ${target_user}" "👤"

    if [[ ! -d "$target_home" ]]; then
        log_warn "[${target_user}] 홈 디렉터리 없음 (${target_home}) — 건너뜁니다"
        return 0
    fi

    # .bashrc가 없으면 skel에서 생성 (신규 계정 대응)
    if [[ ! -f "${target_home}/.bashrc" ]]; then
        cp /etc/skel/.bashrc "${target_home}/.bashrc" 2>/dev/null || touch "${target_home}/.bashrc"
        chown "${target_user}:${target_user}" "${target_home}/.bashrc" 2>/dev/null || true
        log_info "[${target_user}] .bashrc 신규 생성"
    fi

    install_ohmybash      "$target_user" "$target_home"
    configure_ohmybash    "$target_user" "$target_home"
    configure_starship    "$target_user" "$target_home"
    configure_vimrc       "$target_user" "$target_home"
    configure_bashrc      "$target_user" "$target_home"
}

# ─── 설정 대상 계정 목록 산출 ────────────────────────────────────────────────
# root(UID 0) + 일반 사용자(UID >= 1000) 중 로그인 가능한 계정
# 출력: "사용자명 홈디렉터리" (줄 단위)
list_target_users() {
    awk -F: '
        $7 ~ /(nologin|false|sync|shutdown|halt)$/ { next }
        ($3 == 0 || $3 >= 1000) && $6 != "" { print $1, $6 }
    ' /etc/passwd | sort -u
}

# ─── 시스템 전역 설정 ────────────────────────────────────────────────────────
# /etc/starship.toml : STARSHIP_CONFIG 환경변수가 이 경로를 가리키는 환경 대비.
#                      구버전 파일이 남아 사용자 설정을 덮어쓰는 사고를 방지한다.
# /etc/skel          : 이후 useradd로 생성되는 신규 계정에 자동 적용
configure_system_wide() {
    section_header "시스템 전역 설정 (/etc)" "🧬"

    if command -v starship &>/dev/null; then
        write_starship_toml "/etc/starship.toml"
        log_ok "/etc/starship.toml 배포 완료 (STARSHIP_CONFIG 대비)"

        write_starship_toml "/etc/skel/.config/starship.toml"
        log_ok "/etc/skel/.config/starship.toml 배포 완료"
    else
        log_warn "starship 미설치 — 전역 toml 배포 생략"
    fi

    write_vimrc "/etc/skel/.vimrc"
    log_ok "/etc/skel/.vimrc 배포 완료"

    local skel_bashrc="/etc/skel/.bashrc"
    if sudo grep -qF "# >>> rl9_setup: skel <<<" "$skel_bashrc" 2>/dev/null; then
        log_info "/etc/skel/.bashrc 이미 설정됨 — 건너뜁니다"
        return 0
    fi

    sudo tee -a "$skel_bashrc" > /dev/null << 'SKEL_BLOCK'

# >>> rl9_setup: skel <<<
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
alias vi='vim'

export EDITOR='vim'
export KUBE_EDITOR='vim'

export LS_COLORS='di=36'

if command -v lsd &>/dev/null; then
    alias ls='lsd'
    alias ll='lsd -alF'
    alias la='lsd -A'
    alias lt='lsd --tree'
    alias l='lsd -l'
fi

[ -f /usr/share/fzf/shell/key-bindings.bash ] && \
    source /usr/share/fzf/shell/key-bindings.bash

command -v zoxide   &>/dev/null && eval "$(zoxide init bash)"
command -v starship &>/dev/null && eval "$(starship init bash)"
# <<< rl9_setup: skel >>>
SKEL_BLOCK

    log_ok "/etc/skel/.bashrc 배포 완료 — 신규 계정 자동 적용"
}

# =============================================================================
# 재시도 / 보고서
# =============================================================================
retry_failed() {
    if [[ ${#RETRY_LIST[@]} -eq 0 ]]; then return; fi

    section_header "실패 패키지 재시도" "🔄"
    log_info "총 ${#RETRY_LIST[@]}개 패키지 재시도 중..."
    echo

    local still_failed=()
    for entry in "${RETRY_LIST[@]}"; do
        IFS='|' read -r dname pname method _desc <<< "$entry"
        echo -e "  ${BOLD}▶ [재시도] ${dname}${RESET}"

        local ok=false
        case $method in
            dnf)    sudo dnf install -y --enablerepo=epel "$pname" &>/dev/null && ok=true ;;
            pip)    sudo pip3 install "$pname" &>/dev/null && ok=true ;;
            manual) "$pname" &>/dev/null && ok=true ;;
        esac

        if $ok; then
            echo -e "    ${GREEN}✔ 재시도 성공${RESET}"
            INSTALLED+=("${dname} (재시도 성공)")
        else
            echo -e "    ${RED}✘ 재시도 실패${RESET}"
            still_failed+=("$dname")
        fi
    done

    FAILED=("${still_failed[@]}")
}

print_final_report() {
    local target_list=$1

    clear
    print_banner
    show_resources

    echo -e "${BOLD}${GREEN}"
    echo "  ╔══════════════════════════════════════════════════════════════════╗"
    echo "  ║                     📋  설치 완료 보고서                         ║"
    echo "  ╚══════════════════════════════════════════════════════════════════╝"
    echo -e "${RESET}"

    local success_count=${#INSTALLED[@]}
    echo -e "  ${GREEN}${BOLD}✔ 설치 성공  (${success_count}개)${RESET}"
    echo -e "  ${GREEN}──────────────────────────────────────────${RESET}"
    local i=1
    for pkg in "${INSTALLED[@]}"; do
        [[ -z "$pkg" ]] && continue
        printf "  ${GREEN}%3d.${RESET} %s\n" "$i" "$pkg"
        (( i++ ))
    done
    echo

    local fail_count=${#FAILED[@]}
    if [[ $fail_count -gt 0 ]]; then
        echo -e "  ${RED}${BOLD}✘ 설치 실패  (${fail_count}개)${RESET}"
        echo -e "  ${RED}──────────────────────────────────────────${RESET}"
        local j=1
        for pkg in "${FAILED[@]}"; do
            [[ -z "$pkg" ]] && continue
            printf "  ${RED}%3d.${RESET} %-40s ${DIM}← 수동 설치 필요${RESET}\n" "$j" "$pkg"
            (( j++ ))
        done
        echo
        echo -e "  ${YELLOW}${BOLD}[수동 설치 가이드]${RESET}"
        echo -e "  ${YELLOW}• dnf 패키지: sudo dnf install -y <패키지명>${RESET}"
        echo -e "  ${YELLOW}• pip 패키지: sudo pip3 install <패키지명>${RESET}"
        echo -e "  ${YELLOW}• starship  : curl -sS https://starship.rs/install.sh | sudo sh${RESET}"
        echo -e "  ${YELLOW}• glow      : go install github.com/charmbracelet/glow@latest${RESET}"
        echo
    fi

    local total=$(( success_count + fail_count ))
    local rate=$(( success_count * 100 / (total == 0 ? 1 : total) ))
    echo -e "  ${BOLD}${CYAN}──────────────────────────────────────────────────────────────────${RESET}"
    printf "  ${BOLD}설치율: %d%% (%d / %d)${RESET}\n" "$rate" "$success_count" "$total"
    echo -e "  ${DIM}완료 시각: $(date '+%Y-%m-%d %H:%M:%S')${RESET}"
    echo -e "  ${BOLD}${CYAN}──────────────────────────────────────────────────────────────────${RESET}"
    echo

    echo -e "  ${BOLD}${MAGENTA}📌 환경 설정 적용 결과${RESET}"
    echo -e "  ${GREEN}  ✔ 적용 완료 계정${RESET}"
    while read -r u h; do
        [[ -z "$u" ]] && continue
        echo -e "  ${DIM}      • ${u}  (${h})${RESET}"
    done <<< "$target_list"
    echo -e "  ${GREEN}  ✔ 배포 항목${RESET}              .bashrc · starship.toml · .vimrc · Oh My Bash"
    echo -e "  ${GREEN}  ✔ /etc/starship.toml${RESET}       STARSHIP_CONFIG 환경 대비"
    echo -e "  ${GREEN}  ✔ /etc/skel${RESET}                신규 계정 자동 적용"
    echo
    echo -e "  ${BOLD}${YELLOW}  ⚠ 이미 열려있는 셸에는 자동 반영되지 않습니다${RESET}"
    echo -e "  ${YELLOW}      적용하려면:  exec bash${RESET}"
    echo
    echo -e "  ${DIM}── 선택 작업 (수동) ──────────────────────────────────────────────${RESET}"
    echo -e "  ${DIM}1. zsh 기본 셸 변경    :  chsh -s \$(which zsh)${RESET}"
    echo -e "  ${DIM}2. 감사 데몬 시작      :  sudo systemctl enable --now auditd${RESET}"
    echo -e "  ${DIM}3. fail2ban 시작       :  sudo systemctl enable --now fail2ban${RESET}"
    echo
}

# =============================================================================
# 사전 확인
# =============================================================================
check_sudo() {
    if ! sudo -n true 2>/dev/null; then
        echo -e "${RED}[오류] 이 스크립트는 sudo 권한이 필요합니다.${RESET}"
        echo -e "${YELLOW}  sudo 비밀번호를 입력한 후 재실행 하세요.${RESET}"
        sudo true || exit 1
    fi
}

check_os() {
    if ! grep -qi "rocky linux 9" /etc/os-release 2>/dev/null; then
        echo -e "${YELLOW}[경고] Rocky Linux 9 환경이 아닙니다. 계속 진행하시겠습니까? (y/N)${RESET}"
        read -r answer
        [[ "${answer,,}" == "y" ]] || exit 0
    fi
}

# =============================================================================
# 메인
# =============================================================================
main() {
    print_banner
    check_sudo
    check_os

    local target_list
    target_list=$(list_target_users)

    echo -e "  ${BOLD}설치를 시작합니다. 환경에 따라 10~30분 소요될 수 있습니다.${RESET}"
    echo -e "  ${DIM}설치 중 실패한 패키지는 건너뛰고 마지막에 재시도합니다.${RESET}"
    echo
    echo -e "  ${BOLD}환경 설정 적용 대상 계정:${RESET}"
    while read -r u h; do
        [[ -z "$u" ]] && continue
        echo -e "  ${DIM}    • ${u}  (${h})${RESET}"
    done <<< "$target_list"
    echo -e "  ${DIM}    • /etc/skel  (이후 생성되는 신규 계정)${RESET}"
    echo
    sleep 2

    enable_repos
    install_all_packages
    retry_failed

    # 모든 계정에 환경 설정 적용
    # fd 3으로 읽는다 — 루프 안의 sudo/curl이 stdin을 삼켜 목록이 잘리는 것을 방지
    while read -r u h <&3; do
        [[ -z "$u" ]] && continue
        apply_user_config "$u" "$h"
    done 3<<< "$target_list"

    # 시스템 전역 설정 (/etc/starship.toml, /etc/skel)
    configure_system_wide

    print_final_report "$target_list"
}

# --source-only 로 source 하면 main()을 실행하지 않고 함수 정의만 로드한다.
# (guest_setup.sh 등 다른 스크립트에서 install_ohmybash/configure_starship 등
#  개별 함수만 재사용할 때 사용)
if [[ "${1:-}" != "--source-only" ]]; then
    main "$@"
fi
```

**해설**: `set -euo pipefail`, 함수 분리, 배열(`declare -a`), `case`, `for`/`while` 반복문, heredoc(`<< 'TAG'`), 파라미터 확장(`${var#prefix}`, `${var%%.*}`), 산술 연산(`$(( ))`), 서브셸 명령치환(`$(...)`) 등 SH-01 ~ SH-06에서 다룬 문법이 실제 운영 스크립트에서 어떻게 조합되는지 보여주는 대표 예시다. 맨 끝의 `--source-only` 가드 덕분에 아래 `guest_setup.sh`처럼 다른 스크립트가 이 파일을 `source`로 불러와 개별 함수만 재사용할 수 있다.

## 문제 2) .vimrc + 안전 alias 배포 스크립트

**요구사항**

root + 모든 일반 계정(UID 1000 이상) + `/etc/skel`(신규 계정)에 `.vimrc`와 안전 alias를 배포하는 스크립트(`vimrc_setup.sh`)를 작성하라. 다음 조건을 만족해야 한다.

- `set -euo pipefail`을 선언하고, `EUID` 체크 후 root가 아니면 `exec sudo bash "$0" "$@"`로 자기 자신을 root 권한으로 재실행할 것
- `/usr/bin/vi`가 vim-minimal(Small version)인 경우 vim-enhanced(Huge)로 교체할 것 — `alias vi='vim'`은 대화형 셸에서만 동작하므로 `sudo vi`로 직접 열 때도 문법강조가 적용되도록 `alternatives`로 바이너리 자체를 교체해야 한다. 기존 vim-minimal 바이너리는 백업해 둘 것
- `.vimrc` 내용을 담은 `VIMRC_CONTENT`와 안전 alias(`rm -i` 등) + `EDITOR`/`KUBE_EDITOR`/`LS_COLORS` 환경변수를 담은 `ALIAS_BLOCK`을 변수로 정의할 것
- 대상 경로에 `.vimrc`를 배포하는 `deploy_vimrc()`와 `.bashrc`에 alias 블록을 추가하는 `deploy_alias()`를 작성하되, 각각 기존 파일은 날짜 백업 후 덮어쓰고, alias는 마커 주석으로 중복 추가를 방지할 것
- `awk`로 `/etc/passwd`를 파싱해 root + UID 1000 이상 로그인 가능 계정을 추출하고, `while read` 반복문으로 각 계정의 홈 디렉터리에 `.vimrc`/alias를 배포할 것
- 마지막으로 `/etc/skel`에도 동일하게 배포해 신규 계정에 자동 적용되도록 할 것

### 정답

```bash
#!/usr/bin/env bash
# =============================================================================
# .vimrc + 안전 alias 배포 스크립트
# 대상: root + 모든 일반 계정(UID>=1000) + /etc/skel(신규 계정)
# 추가: /usr/bin/vi가 vim-minimal(Small version)인 경우 vim-enhanced(Huge)로 교체
#       -> sudo vi 로 직접 열 때도(alias를 거치지 않는 경우) 문법강조 적용됨
# 권한: sudo 필요
# =============================================================================
set -euo pipefail   # -e: 명령 실패 시 즉시 종료 / -u: 미정의 변수 참조 시 종료 / pipefail: 파이프라인 중 하나라도 실패하면 전체를 실패로 처리
GREEN='\033[0;32m'; YELLOW='\033[1;33m'; CYAN='\033[0;36m'; RESET='\033[0m'

if [[ $EUID -ne 0 ]]; then          # root가 아니면
    if ! sudo -n true 2>/dev/null; then   # sudo 캐시가 없으면 여기서 인증
        sudo true || exit 1
    fi
    exec sudo bash "$0" "$@"        # 스크립트 자신을 root 권한으로 재실행하고 현재 프로세스는 대체됨
fi

echo
echo "  vi/vim 전역 설정 배포 시작"
echo "  ────────────────────────────────────"

# -----------------------------------------------------------------------------
# 0. vim-enhanced 설치 확인
# -----------------------------------------------------------------------------
if ! rpm -q vim-enhanced &>/dev/null; then
    echo -e "  ${CYAN}설치${RESET}  vim-enhanced"
    dnf install -y vim-enhanced
else
    echo -e "  ${CYAN}확인${RESET}  vim-enhanced 이미 설치됨"
fi

# -----------------------------------------------------------------------------
# 1. /usr/bin/vi 를 vim-enhanced(Huge)로 교체
#    (RHEL 계열은 /usr/bin/vi가 vim-minimal(Small, -syntax -eval)인 경우가 많음.
#     alias vi='vim'은 대화형 셸에서만 동작하므로 sudo vi 직접 실행 시엔 안 먹힘.
#     alternatives로 바이너리 자체를 교체해야 근본 해결됨)
# -----------------------------------------------------------------------------
if [[ -L /usr/bin/vi ]] && [[ "$(readlink -f /usr/bin/vi)" == "$(readlink -f /usr/bin/vim)" ]]; then
    echo -e "  ${CYAN}확인${RESET}  /usr/bin/vi 이미 vim-enhanced 연결됨"    # 이미 심볼릭 링크가 vim을 가리키면 건너뜀
else
    if [[ -f /usr/bin/vi && ! -L /usr/bin/vi ]]; then
        [[ -f /usr/bin/vi.small.bak ]] || mv /usr/bin/vi /usr/bin/vi.small.bak   # 기존 vim-minimal 바이너리 백업
        echo -e "  ${CYAN}백업${RESET}  /usr/bin/vi.small.bak (기존 vim-minimal)"
    fi
    alternatives --install /usr/bin/vi vi /usr/bin/vim 100   # /usr/bin/vi를 alternatives 그룹에 등록
    alternatives --set vi /usr/bin/vim                       # /usr/bin/vi가 vim-enhanced를 가리키도록 지정
    echo -e "  ${GREEN}교체${RESET}  /usr/bin/vi -> vim-enhanced"
fi

# -----------------------------------------------------------------------------
# 2. .vimrc 내용
# -----------------------------------------------------------------------------
VIMRC_CONTENT='set nu
set autoindent
set nohlsearch
set background=dark
" 구문 강조 — 256색 팔레트 사용 (SSH 클라이언트 truecolor 미지원 대응)
" darkblue는 vim 기본 포함. silent!로 부재 시에도 에러 없음
syntax on
set t_Co=256
silent! colorscheme darkblue'
# set paste는 넣지 않음: 상시 켜두면 autoindent/자동완성/매핑이 전부 무력화되어
# 오히려 일반 편집이 불편해짐. 붙여넣기 시엔 :set paste 를 그때그때 사용.

ALIAS_BLOCK="
# >>> vimrc_setup: safe aliases <<<
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
alias vi='vim'
# 에디터 — alias는 셸에서만 동작하므로 외부 도구용으로 환경변수 지정
# (kubectl edit, crontab -e, git commit 등)
export EDITOR='vim'
export KUBE_EDITOR='vim'
if command -v dircolors >/dev/null 2>&1; then
    eval \"\$(dircolors)\"
fi
export LS_COLORS=\"\${LS_COLORS:-}:di=36\"
# <<< vimrc_setup: safe aliases >>>"

# -----------------------------------------------------------------------------
# 3. 배포 함수
# -----------------------------------------------------------------------------
deploy_vimrc() {
    local path=$1 owner=${2:-}          # owner 생략 시 빈 문자열 → 아래 chown 건너뜀
    if [[ -f "$path" ]]; then
        cp -a "$path" "${path}.bak.$(date +%F)"   # 기존 .vimrc를 날짜 붙여 백업 (덮어쓰기 전 안전장치)
        echo -e "  ${CYAN}백업${RESET}  ${path}.bak.$(date +%F)"
    fi
    printf '%s\n' "$VIMRC_CONTENT" > "$path"        # 위에서 정의한 VIMRC_CONTENT로 파일을 새로 씀 (덮어쓰기)
    [[ -n "$owner" ]] && chown "${owner}:${owner}" "$path"   # owner가 지정된 경우에만 소유자 변경
    echo -e "  ${GREEN}배포${RESET}  $path"
}

deploy_alias() {
    local path=$1 owner=${2:-}
    [[ -f "$path" ]] || touch "$path"   # .bashrc가 아예 없으면 빈 파일로 생성
    if grep -qF "# >>> vimrc_setup: safe aliases <<<" "$path" 2>/dev/null; then
        echo -e "  ${CYAN}건너뜀${RESET}  ${path} — alias 이미 존재"
        return 0    # 마커 주석이 이미 있으면 중복 추가하지 않고 종료
    fi
    printf '%s\n' "$ALIAS_BLOCK" >> "$path"   # 파일 끝에 alias 블록 append (덮어쓰기 아님)
    [[ -n "$owner" ]] && chown "${owner}:${owner}" "$path"
    echo -e "  ${GREEN}배포${RESET}  $path (alias)"
}

# -----------------------------------------------------------------------------
# 4. root + 일반 계정(UID>=1000) + /etc/skel 에 배포
# -----------------------------------------------------------------------------
awk -F: '
    $7 ~ /(nologin|false|sync|shutdown|halt)$/ { next }
    ($3 == 0 || $3 >= 1000) && $6 != "" { print $1, $6 }
' /etc/passwd | sort -u | while read -r user home; do   # $1=계정명, $6=홈 디렉터리를 한 줄씩 읽음
    if [[ -d "$home" ]]; then
        deploy_vimrc "${home}/.vimrc"  "$user"    # 계정 홈에 .vimrc 배포
        deploy_alias "${home}/.bashrc" "$user"    # 같은 계정의 .bashrc에 alias 추가
    else
        echo -e "  ${YELLOW}건너뜀${RESET}  ${user} — 홈 디렉터리 없음 (${home})"
    fi
done

# 신규 계정용
deploy_vimrc /etc/skel/.vimrc
deploy_alias /etc/skel/.bashrc

echo "  ────────────────────────────────────"
echo -e "  ${GREEN}완료${RESET}"
echo -e "  vi/vim : 새로 실행하면 바로 적용 (sudo vi 포함)"
echo -e "  alias  : ${CYAN}source ~/.bashrc${RESET} 또는 ${CYAN}exec bash${RESET}"
echo
```

**해설**: `rl9_setup.sh`의 `write_vimrc`/`configure_vimrc` 로직을 단독 스크립트로 분리한 형태로, `EUID` 체크와 `exec sudo bash "$0" "$@"` 자기 재실행 패턴, `alternatives`를 이용한 시스템 바이너리 교체, 함수 인자 기본값 처리(`${2:-}`), `awk` 필드 파싱, 파이프(`|`)로 연결된 `while read` 반복문 등 Shell Script 챕터의 핵심 문법이 짧은 스크립트 안에 응축되어 있다.

## 문제 3) guest 계정 dotfile 배포 재실행 스크립트

**요구사항**

`rl9_setup.sh`가 이미 시스템 전체에 배포된 뒤, guest 계정 하나에만 Oh My Bash/starship/`.bashrc` 설정을 재적용하는 짧은 래퍼 스크립트(`guest_setup.sh`)를 작성하라. 다음 조건을 만족해야 한다.

- `set -euo pipefail`을 선언할 것
- `sudo -u guest bash -c '...'`로 guest 계정 컨텍스트에서 명령을 실행할 것
- 먼저 guest 계정의 `~/.bashrc`를 날짜 백업할 것
- `rl9_setup.sh`를 `--source-only` 옵션으로 `source`하여 `main()`은 실행하지 않고 함수 정의만 로드할 것
- 로드한 `install_ohmybash`, `configure_ohmybash`, `configure_starship`, `configure_bashrc` 함수를 `"guest" "$HOME"` 인자로 순서대로 호출해 guest 계정에 Oh My Bash 설치, OMB `.bashrc` 조정, starship.toml 배포, 안전 alias/lsd/fzf/zoxide/starship init 추가를 재적용할 것

### 정답

```bash
#!/usr/bin/env bash
# =============================================================================
# 목적: guest 계정 컨텍스트에서 dotfile 배포 재실행
# sudo: 필요(su 전환용) | 위험도: 중 | 영향: guest 계정 .bashrc/.config
# =============================================================================
set -euo pipefail   # -e: 명령 실패 시 즉시 종료 / -u: 미정의 변수 참조 시 종료 / pipefail: 파이프라인 중 하나라도 실패하면 전체를 실패로 처리

# 사전 백업
sudo -u guest bash -c 'cp -a ~/.bashrc ~/.bashrc.bak.$(date +%F) 2>/dev/null'   # guest 홈의 ~는 guest 계정 기준으로 해석됨

# 스크립트에서 함수만 분리 실행 (source 후 함수 호출)
sudo -u guest bash -c '
source /path/to/rl9_setup.sh --source-only  # main() 실행 없이 함수 정의만 로드
install_ohmybash   "guest" "$HOME"          # guest 계정에 Oh My Bash 설치
configure_ohmybash "guest" "$HOME"          # OSH_THEME 비활성화, SRE 플러그인 적용
configure_starship "guest" "$HOME"          # ~/.config/starship.toml 배포
configure_bashrc   "guest" "$HOME"          # 안전 alias, lsd, fzf, zoxide, starship init 추가
'
```

**해설**: 별도 셸에서 실행되는 `sudo -u guest bash -c '...'` 안의 `$HOME`/`$(date +%F)`는 작은따옴표로 감싸 바깥 스크립트가 아닌 guest 계정의 서브셸에서 평가되도록 한 점이 핵심이다. `rl9_setup.sh`에 추가된 `--source-only` 가드(문제 1의 정답 마지막 부분 참고)가 있어야만 이 스크립트가 정상 동작한다.
