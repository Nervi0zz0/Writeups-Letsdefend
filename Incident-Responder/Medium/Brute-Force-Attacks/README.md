# Write-up: Brute Force Attacks Analysis

**Challenge:** [Brute Force Attacks](https://app.letsdefend.io/challenge/brute-force-attacks)
**Platform:** LetsDefend.io
**Role:** Incident Responder
**Difficulty:** Medium
**Date Completed:** 2025-04-06 *(Actualizado con fecha actual)*

---

## 🎯 1. Objective

Analyze provided network traffic (`.pcap`) and authentication logs (`auth.log`) from a compromised web server to determine the scope and success of brute-force attacks targeting web login, RDP, and SSH services. Identify attacker details, compromised credentials, and map the activity to the MITRE ATT&CK framework.

---

## 📁 2. Evidence Provided

* `BruteForce.pcap`: Network packet capture file.
* `auth.log`: Linux server authentication log file.

---

## 🛠️ 3. Tools Used

* Wireshark
* Linux Command Line (`grep`, `wc`, `uniq`, `cat`, `tshark`)
* Text Editor

---

## 📊 4. Analysis & Findings

The investigation involved correlating network traffic analysis with server log examination.

### 4.1. Target Server IP Address

> **Methodology:** Network traffic in `BruteForce.pcap` was analyzed. Filtering by protocols (HTTP, RDP) and examining endpoint statistics (`Statistics > Endpoints > IPv4` in Wireshark) helped identify the public IP address receiving the malicious traffic.

<details>
<summary>💡 Finding: Target IP Address</summary>
The targeted web server IP address is **`51.116.96.181`**.
</details>

* _Supporting Image Placeholder: `![Wireshark Endpoints View](./images/wireshark_endpoints.png)`_

### 4.2. Targeted Web Directory

> **Methodology:** Wireshark filter `http.request.method == POST` was applied to `BruteForce.pcap`. The URI (Uniform Resource Identifier) targeted by these frequent POST requests was identified.

<details>
<summary>💡 Finding: Targeted Web Resource</summary>
The brute-force attack primarily targeted the **`/index.php`** page/script.
</details>

* _Supporting Image Placeholder: `![HTTP POST Requests](./images/http_post_requests.png)`_

### 4.3. Compromised Web Credentials

> **Methodology:** HTTP traffic within `BruteForce.pcap` was inspected. Successful login was identified by comparing HTTP response sizes or searching for specific success indicators (e.g., `>correct`, 'Welcome', redirects) in response bodies. The corresponding request packet revealed the credentials used.

<details>
<summary>🔑 Finding: Compromised Web Credentials</summary>
* Username: **`web-hacker`**
* Password: **`admin12345`**
</details>

* _Supporting Image Placeholder: `![HTTP Stream Credentials](./images/http_stream_creds.png)`_

### 4.4. Number of Users Targeted (HTTP Brute-Force)

> **Methodology:** Used `tshark` or Wireshark export combined with command-line tools (`grep`, `sort`, `uniq`, `wc`) to extract and count unique usernames from the data portion of HTTP POST requests targeting the login page.
> ```bash
> # Example command (adjust field names/patterns as needed):
> # tshark -r BruteForce.pcap -Y "http.request.method == POST && ip.dst==51.116.96.181" -T fields -e http.file_data | grep 'username=' | sort | uniq | wc -l
> ```

*(Nota: Debes ejecutar un comando similar al de arriba para obtener este recuento específico de usuarios HTTP y rellenar el spoiler)*
<details>
<summary>🔢 Finding: Unique HTTP Users Targeted Count</summary>
**`[COUNT_HERE]`** unique user accounts were targeted via the web interface.
</details>

### 4.5. Attacker Machine Client Name (RDP)

> **Methodology:** Filtered `BruteForce.pcap` in Wireshark using `rdp.client.name` or searched RDP negotiation packets (specifically, Client X.224 Connection Request) for the `clientName` field/string.

<details>
<summary>💻 Finding: Attacker RDP Client Name</summary>
The attacker's machine name observed in RDP traffic is **`t3m0-virtual-ma`**.
</details>

* _Supporting Image Placeholder: `![RDP Client Name](./images/rdp_clientname.png)`_

### 4.6. Last Successful SSH Login

> **Methodology:** Analyzed `auth.log` using `grep` to filter for lines indicating successful SSH authentication (`Accepted password` or `Accepted publickey`). The last chronological entry was examined.
> ```bash
> grep -i "accepted" auth.log | tail -n 1
> ```

<details>
<summary>⏰ Finding: Last Successful SSH Login</summary>
User **`mmox`** at **`11:43:54`**. *(Nota: Añade la fecha completa si está disponible en tu archivo auth.log)*.
</details>

* _Supporting Image Placeholder: `![Auth Log Accepted SSH](./images/auth_log_accepted.png)`_

### 4.7. Number of Unsuccessful SSH Attempts

> **Methodology:** Used `grep` and `wc` on `auth.log` to count the total number of lines explicitly indicating a failed password attempt for the SSH service (`sshd`).
> ```bash
> grep -i "failed password" auth.log | wc -l
> ```

<details>
<summary>❌ Finding: Unsuccessful SSH Attempts Count</summary>
**`7480`** unsuccessful SSH login attempts were recorded.
</details>

### 4.8. Attack Technique Identification (MITRE ATT&CK®)

> **Methodology:** Correlated the observation of numerous failed login attempts across multiple protocols (HTTP, SSH, RDP indicators) with the MITRE ATT&CK® framework tactics and techniques.

<details>
<summary>⚔️ Finding: MITRE ATT&CK® Technique</summary>
The primary technique identified is **Brute Force (T1110)**.
</details>

---

## 📝 5. Summary of Key Findings / Indicators

| Indicator Type                | Finding                                                                 |
| :---------------------------- | :---------------------------------------------------------------------- |
| **Target IP Address** | <details><summary>View</summary>`51.116.96.181`</details>             |
| **Target Web Resource** | <details><summary>View</summary>`/index.php`</details>                 |
| **Compromised Web Creds** | <details><summary>View</summary>`web-hacker` / `admin12345`</details> |
| **Last Successful SSH User** | <details><summary>View</summary>`mmox` (at `11:43:54`)</details>         |
| **Attacker RDP ClientName** | <details><summary>View</summary>`t3m0-virtual-ma`</details>            |
| **MITRE ATT&CK® Technique** | <details><summary>View</summary>T1110 (Brute Force)</details>          |
| **Failed SSH Attempts** | <details><summary>View</summary>`7480`</details>                       |
| **HTTP Users Targeted** | <details><summary>View</summary>`[COUNT_HERE]` *(Requires calculation)*</details> |

---

## 🎓 6. Conclusion & Lessons Learned

The investigation successfully identified multi-protocol brute-force attempts against the web server. Analysis of network traffic using Wireshark revealed web and RDP indicators, while server logs confirmed SSH brute-forcing activity and the last successful login. This exercise highlighted the importance of correlating network and host-based evidence and reinforced skills in log analysis (`auth.log`) and packet inspection (`Wireshark`, filtering, stream following, string searching). Protecting against brute-force requires strong password policies, account lockouts, monitoring failed login attempts, and potentially IP-based blocking (like fail2ban) or CAPTCHAs.