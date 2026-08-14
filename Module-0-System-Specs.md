### Infrastructure Baseline Specifications
*   **Hypervisor:** VMware Fusion (Apple Silicon Architecture)
*   **Guest OS:** CentOS Stream 9 (64-bit ARM)
*   **Installation Profile:** Minimal Install (CLI Only, No GUI for maximum security baseline)
*   **Virtual Hardware Resources:** 2 vCPUs, 2 GB RAM, 20 GB HDD
*   **Network Configuration:** NAT (Network Address Translation)

---

### System Architecture Check
Before running any security modifications, we verify the underlying kernel and architecture.

*   Command used: `uname -a`

```text
Linux black-meridian 5.14.0-654.el9.aarch64 #1 SMP PREEMPT_DYNAMIC Fri Dec 19 09:02:06 UTC 2025 aarch64 aarch64 aarch64 GNU/Linux
```

*   Command used: `cat /etc/os-release`

``` text
NAME="CentOS Stream"
VERSION="9"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="9"
PLATFORM_ID="platform:el9"
PRETTY_NAME="CentOS Stream 9"
ANSI_COLOR="0;31"
LOGO="fedora-logo-icon"
CPE_NAME="cpe:/o:centos:centos:9"
HOME_URL="https://centos.org/"
BUG_REPORT_URL="https://issues.redhat.com/"
REDHAT_SUPPORT_PRODUCT="Red Hat Enterprise Linux 9"
REDHAT_SUPPORT_PRODUCT_VERSION="CentOS Stream"

```

