# Module 1: Identity & Access Management (IAM)

This module implements role-based access control (RBAC) and data isolation for the **Black-Meridian** deep-space probe onboard computer, aligned with the **NIST Cybersecurity Framework (CSF) v2.0** (specifically the **Identify (ID.AM)** and **Protect (PR.AC)** functions).

---

## 1. NIST Framework Alignment

### Function: IDENTIFY (ID.AM - Asset Management)
We map the mission-critical human resources (crew members) and data assets orbiting the black hole to enforce granular security isolation boundaries.

#### Crew Role Inventory (User Accounts)

| Crew Member | Username | Group | Role / Mission Responsibility | Privilege Level |
| :--- | :--- | :--- | :--- | :--- |
| Eva Stratt | `e_stratt` | `engineers` | Mission Commander and Vessel Pilot | Non-privileged |
| Olesya Ilyukhina | `o_ilyukhina` | `engineers` | Chief Flight Systems Engineer | Non-privileged |
| Ryland Grace | `r_grace` | `scientists` | Chief Mission Scientist | Non-privileged |
| Lana Veyr | `l_veyr` | `scientists` | Lead Astrophysicist | Non-privileged |

#### Critical Data Assets (Logical Directories)
*   `/space/propulsion/` — Engine configurations and thruster navigation variables (Critical Asset).
*   `/space/astrophysics/` — Raw sensor telemetry from black hole observation (Scientific Asset).

---

### Function: PROTECT (PR.AC - Access Control)
To mitigate the risk of internal accidental data tampering or credential hijacking, the following access control policies are defined:
*   **Policy 1 (Least Privilege):** Scientific crew must have zero write or read access to critical flight propulsion systems. Engineering crew must be isolated from scientific research data.
*   **Policy 2 (Role Isolation):** Administrative tasks are executed exclusively via authorized accounts using `sudo`. No crew member has direct access to the `root` environment.

---

## 2. Technical Implementation & Hardening

### Step 2.1: Group and User Provisioning
Execution of account creation commands to map physical crew roles to isolated system objects:

```bash
sudo groupadd scientists
sudo groupadd engineers

sudo useradd -m -g engineers -c "Engineer Olesya Ilyukhina" o_ilyukhina
sudo useradd -m -g engineers -c "Mission Commander and Pilot Eva Stratt" e_stratt
sudo useradd -m -g scientists -c "Scientist Ryland Grace" r_grace
sudo useradd -m -g scientists -c "Astrophysicist Lana Veyr" l_veyr
```

### Step 2.2: Directory Creation and Access Control List (ACL) Application
We enforce a strict `2770` permissions profile using native Linux `chown` and `chmod` utilities. 

```bash
sudo mkdir -p /space/astrophysics
sudo mkdir -p /space/propulsion

sudo chown root:scientists /space/astrophysics
sudo chown root:engineers /space/propulsion

# Applying 2770 (Enforcing SGID for collaborative space environments)
sudo chmod 2770 /space/astrophysics
sudo chmod 2770 /space/propulsion
```

> **Design Justification (The "2" Bit / SGID):** The leading `2` activates the **Set Group ID (SGID)** bit. In deep-space operations where scientists must collaborate seamlessly, SGID ensures that any new file created inside `/space/astrophysics/` automatically inherits the `scientists` group ownership (instead of the personal group of the user who created it). This prevents files created by `r_grace` from being locked as read-only for `l_veyr`, allowing secure, shared scientific analysis.

### Step 2.3: System Verification (Verification Artifacts)
*   Command used: `ls -ld /space/astrophysics /space/propulsion`

```text
drwxrws---. 2 root scientists 6 Aug 11 23:45 /space/astrophysics
drwxrws---. 2 root engineers  6 Aug 11 23:45 /space/propulsion
```
*(Notice how the traditional `x` execution flag for the group is replaced with an `s`—this verifies that the SGID bit is active).*


---

## 3. Threat Modeling Scenario & Verification
*   **Threat Scenario:** Insider Access / Cross-Role Tampering.
*   **Test Condition:** Attempting to browse the propulsion directory while authenticated as scientist `r_grace`.

```text
[root@black-meridian ~]# su - r_grace
[r_grace@black-meridian ~]\$ ls -la /space/propulsion
ls: cannot open directory '/space/propulsion': Permission denied
```








