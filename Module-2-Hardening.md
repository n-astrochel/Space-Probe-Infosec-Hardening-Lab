# Module 2: SSH Network Hardening & Privilege Elevation

This module enforces access controls on administrative utilities and locks down the primary network entryway of the **Black-Meridian** probe, aligning with the **NIST Cybersecurity Framework (CSF) v2.0** under the **Protect (PR.PT - Protective Technology)** function.

---

## 1. NIST Framework Alignment

### Function: PROTECT (PR.PT - Protective Technology)
To safeguard the system kernel, we must restrict administrative superpowers (`sudo`) and eliminate weak default settings on remote network management channels.

#### Threat Modeling & Policy Constraints
1.  **Threat (Privilege Escalation):** If a regular crew account is compromised, an adversary might attempt to run arbitrary system updates or erase logs.
    *   *Mitigation:* Use `visudo` to bind users to strict, explicitly allowed commands.
2.  **Threat (Brute-Force & Credential Spraying):** Automated malicious scans constantly target default SSH Port 22 and attempt to guess passwords.
    *   *Mitigation:* Disable direct root network login entirely and migrate communication to a non-standard custom port.

---

## 2. Technical Implementation

### Step 2.1: Granular Sudo Constraints
Instead of granting full administrative rights, we restrict Engineer Olesya Ilyukhina (`o_ilyukhina`) to troubleshooting commands only (viewing logs via `journalctl` and managing the network service).

*   Command used to edit the file safely: `sudo visudo`
*   Line appended to the bottom of the configuration:
```text
o_ilyukhina ALL=(ALL) /usr/bin/systemctl restart network, /usr/bin/journalctl
```

#### Verification Test
*Authenticating as `o_ilyukhina` and attempting to read the root shadow password file:*
```bash
[o_ilyukhina@black-meridian ~]$ sudo cat /etc/shadow
```
*Verification Artifact (System Rejection):*
```text
Sorry, user o_ilyukhina is not allowed to execute '/bin/cat /etc/shadow' as root on black-meridian.

```

---

### Step 2.2: Hardening the SSH Daemon Perimeters
We secure the active network listening door by editing `/etc/ssh/sshd_config`.

1.  **Port Translation:** Shifted from default Port `22` to custom deep-space channel `2222`.
2.  **Root Exclusion:** Switched `#PermitRootLogin prohibit-password` to `PermitRootLogin no`.

*Commands used to apply changes:*
```bash
sudo systemctl restart sshd
```

#### Verification Test
*Attempting to log in as the default master `root` user directly via the host machine terminal:*
```bash
ssh root@<PROBE_IP_ADDRESS> -p 2222
```
*Verification Artifact (Network Boundary Block):*
```text
Permission denied, please try again.
```

---

### Step 2.3: Perimeter Lockdown (Closing Legacy Port 22)
To ensure the defense-in-depth model is functional, opening the new custom channel (`2222`) must be immediately followed by decommissioning and closing the default legacy entry point (`Port 22`). Leaving Port 22 open in the network filter negates the entire purpose of port migration and allows adversaries to continue brute-force scanning.

*   Commands used to replace the firewall rules and reload the perimeter tables:
```bash
# Add the new custom deep-space channel to the permanent configuration
sudo firewall-cmd --add-port=2222/tcp --permanent

# Completely remove the default legacy SSH service/port 22 from the firewall
sudo firewall-cmd --remove-service=ssh --permanent

# Reload the filtering tables to drop all active legacy configurations immediately
sudo firewall-cmd --reload
```

#### Post-Lockdown Boundary Verification Test
*Attempting to force a connection back through the original legacy Port 22 channel from Mission Control:*
```bash
ssh admin_astrochel@172.16.108.128 -p 22
```
*Verification Artifact (Network Filter Rule Deflected Successfully):*
```text
ssh: connect to host 172.16.108.128 port 22: Connection refused
```

---

## 3. Engineering Troubleshooting & Infrastructure Fixes

During the hardening process, several enterprise-level system blockages were encountered. This section documents the operational issues and the technical modifications applied to resolve them.

 ### 📌 Issue 3.1: Text Editor Navigation Malfunction & Terminal Environment Instability (CentOS aarch64)

*   **Symptom:** As soon as the text editor was opened to modify system configuration files, the arrow keys were completely unresponsive, and the standard status indicator (e.g., `-- INSERT --`) failed to appear at the bottom of the screen. The cursor remained entirely frozen.
*   **Root Cause Analysis & Investigation Steps:**
    1.  **Package Verification:** The system was verified to have the full advanced editor package installed (`vim-enhanced-8.2.2637-40.el9.aarch64`), ruling out the missing binary issue common in minimal distributions.
    2.  **Configuration Scope and Path Failures:** Standard configuration interventions—such as generating local and user-level `.vimrc` files containing `set nocompatible`—failed to resolve the frozen state. 
    3.  **Environment Instability under Root:** A highly inconsistent behavior was observed while operating under the `root` profile: running `sudo vim` temporarily restored functionality and displayed the Insert prompt for a single session, while dropping `sudo` (running straight `vim` as root) immediately broke navigation again. Subsequent retries with `sudo` also failed. Targeted debugging overrides (forcing non-compatible flags `vim -N`, altering terminal types `TERM=linux`, and stripping mouse bindings `set mouse=`) proved entirely ineffective.
*   **Final Resolution (Adopted Alternative Navigation):** Due to the unpredictable nature of the terminal's input buffer and environment states, further environment debugging was abandoned. To complete the critical deployment phase without delays, native, environment-independent `vi` terminal command navigation was adopted:
    *   **String Search:** Executed `/Port` and `/PermitRootLogin` in Normal mode to force the cursor to immediately jump directly to the target configuration variables.
    *   **Manual Traversal:** Utilized standard micro-navigation keys (`l` for rightward traversal, `w` to jump forward by word) to position the prompt over the exact parameter values.
    *   **In-Place Editing:** Leveraged the `cw` (Change Word) command to cleanly overwrite values (e.g., changing `yes` to `no`), avoiding manual arrow-key usage, followed by a safe save sequence using `Esc` and `:wq`.



### 📌 Issue 3.2: SSH Port Migration Blocked by Operating System Kernel (SELinux Policy)
*   **Symptom:** After modifying `/etc/ssh/sshd_config` to use custom Port `2222`, execution of `systemctl restart sshd` failed completely.
*   **Root Cause:** **SELinux (Security-Enhanced Linux)** was operating in strict `Enforcing` mode. The active kernel policy restricted the SSH daemon from binding to any network port outside the standard default Port 22.
*   **Resolution:** Used the `semanage` utility to dynamically append the custom deep-space channel to the allowed SSH kernel policy map:
```bash
sudo semanage port -a -t ssh_port_t -p tcp 2222
```

### 📌 Issue 3.3: Network Perimeter Exclusion (Firewalld Blockage)
*   **Symptom:** The SSH daemon successfully bound to Port `2222` locally, but remote connection attempts from the host machine terminal (Mission Control) resulted in a network timeout.
*   **Root Cause:** The native Linux network filter (**Firewalld**) was executing its default drop/deny policy on all non-standard incoming TCP packets.
*   **Resolution:** Opened the perimeter gateway by explicitly adding the new TCP port to the active zone configuration and reloading the packet filtering tables:
```bash
sudo firewall-cmd --add-port=2222/tcp --permanent
sudo firewall-cmd --reload
```

---

## 🔬 Verification Log (The Result)
The hardening modifications are fully synchronized. The probe has migrated its communication link to Port 2222, verified by establishing an active remote SSH session from Mission Control.


## Conclusion
Module 2 is fully operational. The master backdoor is locked, root network access is severed, and engineering access is strictly partitioned. The probe is ready for **Module 3: Network Filtration & Firewall**.
