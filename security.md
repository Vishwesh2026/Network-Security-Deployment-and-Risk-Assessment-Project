# Risk Assessment and Security Controls

## Risk Assessment

[risk assessment spreadsheet](./risk-tvamatrix.xlsx)

We conducted a compact cyber security risk assessment for ProfServ Consulting using the TVAMatrix method. The assessment covers the network we built and hardened and is based on vulnerabilities we observed and demonstrated in the lab.

TVAMatrix identifies valuable assets, links threats to each asset via specific vulnerabilities, and rates the likelihood and impact for each combination. The resulting scores are shown in the spreadsheet and make it easy to compare relative risks.

---

### Assets identified

We identified twelve assets covering Data, Hardware, Software, Network, People, and Processes.

| Asset No. | Asset Name | Asset Type |
|-----------|------------|------------|
| 1 | Client Financial Records | Data |
| 2 | Business Website Content | Data |
| 3 | Staff Login Credentials | Data |
| 4 | Network Configuration Files | Data |
| 5 | OpenWRT Router / Firewall VM | Hardware |
| 6 | Staff Windows Workstations | Hardware |
| 7 | OpenWRT Firmware and OS | Software |
| 8 | uhttpd Web Server | Software |
| 9 | Office LAN Network (br-mng) | Network |
| 10 | IT Support Staff | People |
| 11 | Administrative Staff | People |
| 12 | Router Maintenance Procedures | Processes |

---

### Vulnerabilities assessed

We identified ten vulnerabilities based on our lab work. Each entry lists the threat, affected asset, and the assessed likelihood and impact.

**Vulnerability 1 — Software attacks (T2) → Client Financial Records (Asset 1)**
Likelihood: **High** | Impact: **Very High**

During the HTTP traffic capture we could read the website content (names and IDs) directly in Wireshark. Any attacker on the same network could do the same and access sensitive client data.

---

**Vulnerability 2 — Espionage and trespass (T4) → Staff login credentials (Asset 3)**
Likelihood: **High** | Impact: **High**

Before hardening, SSH used password authentication on port 22. Automated scanners and brute-force tools target that port; we demonstrated the risk and mitigated it by moving SSH to port 2222 and switching to key-based authentication.

---

**Vulnerability 3 — People errors (T1) → Administrative staff (Asset 11)**
Likelihood: **Moderate** | Impact: **High**

Administrative staff regularly handle client emails. Without security training, a phishing link or malicious attachment could expose credentials or allow attackers into internal systems.

**Vulnerability 4 — Information extortion (T3) → Client Financial Records (Asset 1)**
Likelihood: **Moderate** | Impact: **Very High**

If staff store client data on their workstations, ransomware could encrypt those files after a malicious email attachment is opened. The business could be forced to pay or lose the data.

---

**Vulnerability 5 — Theft (T5) → Staff Windows workstations (Asset 6)**
Likelihood: **Low** | Impact: **High**

If laptops are stolen from an office with poor physical security (no cable locks or CCTV), an attacker could access locally stored client files.

---

**Vulnerability 6 — Technological obsolescence (T6) → OpenWRT firmware and OS (Asset 7)**
Likelihood: **Moderate** | Impact: **Moderate**

If the router firmware is not updated, known vulnerabilities in older releases could be exploited by attackers targeting unpatched systems.

---

**Vulnerability 7 — Forces of nature (T7) → OpenWRT router / firewall VM (Asset 5)**
Likelihood: **Low** | Impact: **Very High**

Severe storms, flooding or power surges could damage the physical host running the VirtualBox VM, causing loss of the network and configuration.

---

**Vulnerability 8 — Technical hardware failures (T8) → OpenWRT router / firewall VM (Asset 5)**
Likelihood: **Moderate** | Impact: **High**

The router runs as a VM on a single host. If the host's disk fails, the VM and its configuration could be lost because there is no redundancy in this setup.

---

**Vulnerability 9 — Changing quality of service (T10) → Office LAN network (Asset 9)**
Likelihood: **Moderate** | Impact: **Moderate**

An ISP outage or NBN disruption would cut internet access, making the website and cloud services unavailable.

---

**Vulnerability 10 — Sabotage and vandalism (T11) → Network configuration files (Asset 4)**
Likelihood: **Low** | Impact: **Very High**

A disgruntled employee with router access could delete or corrupt configuration files (for example, `/etc/config/network`) and bring down the network.

---

### TVAMatrix screenshot

The completed TVAMatrix spreadsheet is linked above and screenshots are included below.

![TVAMatrix Risk Assessment](/images/risk-tvamatrix-1.png)
![TVAMatrix Risk Assessment](/images/risk-tvamatrix-2.png)

---

### Key findings — highest-risk data asset

Client Financial Records (Asset 1) is the highest-risk data asset. It appears in two key vulnerabilities:

- Vulnerability 1 — Eavesdropping via unencrypted HTTP (Likelihood: High, Impact: Very High)
- Vulnerability 4 — Ransomware encrypting local copies (Likelihood: Moderate, Impact: Very High)

The most critical issue is the eavesdropping risk (Vulnerability 1). Our HTTP capture proved that the site's content is readable in plain text. If the site carried client financial information, an attacker on the same network could read that data without any decryption.

---

## Security controls

We chose three controls from NIST SP 800-53 Rev. 5 to address the highest-risk asset (Client Financial Records).

---

### Control 1 — SC-8: Transmission confidentiality and integrity

**Definition:** Protect the confidentiality and integrity of transmitted information.

1) How it reduces the risk:

The HTTP capture showed the site content in plaintext. SC-8 requires a cryptographic mechanism (TLS) so captured traffic is ciphertext and cannot be read without the server's private key. Enforcing HTTPS removes the eavesdropping risk for client records.

2) Implementation for ProfServ Consulting:

- Obtain a TLS certificate (Let's Encrypt is suitable for a small business).
- Configure `uhttpd` to listen on port 443 and reference the certificate and private key.
- Add an `iptables` REDIRECT from port 80 to 443 so users are automatically redirected to HTTPS.
- Verify the HTTPS connection from a browser and confirm the padlock icon.

3) Reference to our setup:

We already use `iptables` rules (for example, blocking port 80) and the router supports TLS libraries. No extra packages are required beyond certificate files.

4) Disadvantages:

Certificates expire (Let's Encrypt: 90 days) and must be renewed. Missed renewals cause browser warnings and service disruption. Automation is possible but requires setup and maintenance. TLS also adds minor CPU overhead during handshakes.

---

### Control 2 — SI-2: Flaw remediation

**Definition:** Identify, report, test and correct system flaws; install security-relevant updates in a timely, controlled manner.

1) How it reduces the risk:

Keeping firmware and packages up to date closes known vulnerabilities that attackers might exploit to take control of the router and access client data.

2) Implementation for ProfServ Consulting:

- Subscribe to OpenWRT security advisories.
- Schedule a monthly maintenance window and check for updates with `apk update` and `apk list --upgradable`.
- Test updates on a separate VirtualBox test instance before applying to production.
- Apply tested updates during maintenance and log each change (version, date, operator) in the Router Maintenance Procedures.

3) Reference to our setup:

We used `apk` during the lab to install `tcpdump`, `luci`, and `iptables-nft`. The same package manager supports update checks, and our documented lab environment can act as the test instance.

4) Disadvantages:

Updates can change defaults or break configurations. Testing adds time and requires a maintained test environment and staff knowledge. If the staff member who manages updates leaves, institutional knowledge may be lost.

---

### Control 3 — IA-2: Identification and authentication (organizational users)

**Definition:** Uniquely identify and authenticate users and link actions to those identities.

1) How it reduces the risk:

Disabling password SSH and requiring unique key-based authentication prevents brute-force attacks from gaining router access. Only holders of registered private keys can authenticate.

2) Implementation for ProfServ Consulting:

- Each IT staff member generates a 4096-bit RSA key pair (PuTTYgen).
- Register each public key individually in LuCI → System → Administration → SSH-Keys.
- Disable password authentication by setting `option PasswordAuth 0` in `/etc/config/dropbear`.
- Keep SSH on port 2222 to reduce automated scanning noise.

3) Reference to our setup:

We demonstrated key generation, upload, and successful key-based login to port 2222 in the Hardening section. The `uci` commands to change the port are already documented and tested.

4) Disadvantages:

If a private key is lost or inaccessible, the user is locked out until another admin removes the old key and registers a replacement. Private keys require secure storage practices to avoid introducing new risks.
