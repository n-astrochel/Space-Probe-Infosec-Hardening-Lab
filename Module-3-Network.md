# Module 3: Network Filtration, Perimeter Defense & Stealth Mode

This module implements global network packet filtering, perimeter lockdown, and host-reconnaissance mitigation for the **Black-Meridian** probe onboard computer. All controls are strictly aligned with the **NIST Cybersecurity Framework (CSF) v2.0** under the **Protect (PR.PT - Protective Technology)** function.

---

## 1. NIST Framework Alignment

### Function: PROTECT (PR.PT - Protective Technology & PR.AC - Access Control)
To minimize the communication attack surface of the interstellar probe, the primary network interface must reject unauthorized inbound discovery requests and operate under a strict "Default Deny" network architecture [NIST].

#### Threat Modeling & Policy Constraints
1.  **Threat (Network Reconnaissance / Port Scanning):** Adversaries or automated malicious scanning bots use specialized tools (such as `nmap`) to discover active host nodes, map topologies, and identify open entry ports.
    *   *Mitigation:* Migrate the main network interface to the strictest native Linux filtering profile (`drop` zone) to silently discard unauthorized packets without returning metadata [NIST].
2.  **Threat (Host Tracking via ICMP):** Threat actors transmit automated network pings (ICMP Echo Requests) to verify if the machine is powered on, online, and responsive.
    *   *Mitigation:* Enforce complete silence against ICMP packets, forcing the host to mimic a completely offline or non-existent device [NIST].

---

## 2. Technical Implementation & Micro-Hardening

### Step 3.1: Enforcing the "Default Deny" Perimeter (Drop Zone Migration)
To establish a hardened perimeter, we configure the core network architecture to drop all inbound traffic by default, maintaining an explicit exception map only for our custom communication channel (`Port 2222/tcp`) [NIST].

*   Commands executed to bind the administration port and shift the default system zone:
```bash
# Append Port 2222/tcp to the drop zone permanent configuration to prevent session lockout
sudo firewall-cmd --zone=drop --add-port=2222/tcp --permanent

# Shift the system-wide active network perimeter profile to drop mode
sudo firewall-cmd --set-default-zone=drop

# Reload the filtering tables to enforce the modifications safely
sudo firewall-cmd --reload
```

#### Verification Artifact (Active Filters Audit)
*Auditing the active network filtering zone structure to verify the interface migration:*
*   Command used: `sudo firewall-cmd --get-active-zones`

```text
drop
  interfaces: ens160
```
*(The verification log confirms that the main ethernet interface `ens160` is bound to the drop zone, ensuring all non-authorized traffic is discarded by default [NIST]).*

---

## 3. Advanced Troubleshooting: The ICMP Information Disclosure Fix

During the final perimeter hardening phase, a critical logical flaw was discovered and remediated regarding the configuration of ICMP (ping) traffic [NIST].

### Issue: Passive Information Disclosure via "Friendly" Filters
*   **Symptom:** An initial policy was applied using the command `sudo firewall-cmd --zone=drop --add-icmp-block=echo-request --permanent` to restrict pings. However, testing the connection from Mission Control (macOS Host Terminal) resulted in the following live output:
```text
[admin_astrochel@mission-control ~]$ ping 172.16.108.128
92 bytes from 172.16.108.128: Communication prohibited by filter
```
*   **Root Cause Analysis:** The `--add-icmp-block` flag forces the Linux kernel to send an active administrative rejection packet (`ICMP Type 3 Code 13 - Communication Administratively Prohibited`) back to the sender [NIST]. In a threat modeling scenario, **this response actively leaks information**. It proves to an attacker that a live, responsive server exists at that IP address, despite the active `drop` zone policy.
*   **Resolution (True Stealth Mode Architecture):** To transition the probe into absolute silence, the verbose ICMP block rule was purged. By removing the explicit block rule, the system defaults back to the raw, unadulterated nature of the `drop` zone—silently swallowing packets into a void without issuing any return metadata [NIST].

*   Command executed to remediate the leak:
```bash
sudo firewall-cmd --zone=drop --remove-icmp-block=echo-request --permanent
sudo firewall-cmd --reload
```

#### Final Stealth Verification Log
*Re-testing the network perimeter ping boundary from Mission Control:*
*   Command executed: `ping 172.16.108.128`

```text
[admin_astrochel@mission-control ~]\$ ping 172.16.108.128
PING 172.16.108.128 (172.16.108.128): 56 data bytes
Request timeout for icmp_seq 0
Request timeout for icmp_seq 1
Request timeout for icmp_seq 2
Request timeout for icmp_seq 3

--- 172.16.108.128 ping statistics ---
4 packets transmitted, 0 packets received, 100.0% packet loss
```
*(Success Profile: The `Communication prohibited by filter` leakage is eradicated. The host terminal experiences 100% silent packet drops and infinite request timeouts, proving the Black-Meridian probe is completely masked from hostile network discovery scans [NIST]).*

---

## Final Project Conclusion
The core **Linux Security Foundation (LSF)** framework for the **Black-Meridian** deep-space exploration probe is successfully deployed:
*   **Module 0:** Base environmental sanitization and remote access channel optimization (SSH terminal deployment).
*   **Module 1:** Strict separation of duties and data isolation via role-based access control (RBAC) using a `2770` SGID collaborative matrix.
*   **Module 2:** System service hardening via root network account exclusion, port transposition (`22` to `2222`), and granular limited `sudo` policies.
*   **Module 3:** Total network cloaking through automated packet deletion (Drop zone perimeter) and stealth configuration auditing [NIST].
