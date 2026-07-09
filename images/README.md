# Images

This folder contains screenshots, diagrams, and draw.io source files used in the project. Files are organised by topic and named so they can be matched to sections in the report.

---

## Root files

Files used directly in `network.md` and `security.md`.

| File | Description |
|------|-------------|
| Lab Network Diagram.png | Lab network diagram showing OpenWRT VM interfaces (`eth0` NAT at `10.0.2.15`, `br-mng` at `192.168.56.2`) and the Windows host (`192.168.56.1`) |
| Lab Network Diagram.drawio | Editable draw.io source for the lab diagram |
| Production Network Diagram.png | Production network diagram (WAN, staff LAN `86.1.1.0/24`, DMZ `86.1.2.0/24`) |
| Production Network Diagram.drawio | Editable draw.io source for the production diagram |
| risk-tvamatrix-1.png | TVA Matrix screenshot — threats, vulnerabilities, assets (page 1) |
| risk-tvamatrix-2.png | TVA Matrix screenshot — likelihood, impact, risk rating (page 2) |

---

## firewall/

Screenshots from the network setup and firewall configuration steps. Used in `network.md`.

### Network setup (net-xx)

| File | Description |
|------|-------------|
| net-01-openwrt-boot.png | OpenWRT first boot in VirtualBox |
| net-02-set-password.png | Setting the root password with `passwd` |
| net-03-ip-addr-initial.png | `ip addr` output before interface changes |
| net-04-vi-network-file.png | `/etc/config/network` opened in `vi` |
| net-05-network-config-edited.png | Network file after adding `br-mng` and static IP |
| net-06-network-restart.png | Running `service network restart` |
| net-07-ip-addr-final.png | `ip addr` showing `br-mng` at `192.168.56.2` |
| net-08-apk-update.png | Running `apk update` before installing LuCI |
| net-09-apk-add-luci.png | Installing LuCI (`apk add luci`) |
| net-10-start-uhttpd.png | Enabling and starting `uhttpd` |
| net-11-luci-interface.png | LuCI dashboard in the browser |
| net-12-vi-index-html.png | Editing `/www/index.html` in `vi` |
| net-13-website-html-content.png | The website HTML source visible in the terminal |

### Firewall configuration (fw-xx)

| File | Description |
|------|-------------|
| fw-00-iptables-nft-install.png | Installing `iptables-nft` for `iptables` compatibility |
| fw-01-http-block-command.png | `iptables` command blocking TCP port 80 |
| fw-02-http-blocked.png | Browser error after the HTTP block rule |
| fw-03-http-unblock-command.png | `iptables` command to remove the HTTP block |
| fw-04-http-unblocked.png | Website loading again after removing the block |
| fw-05-ssh-port22-before.png | SSH to port 22 before changing the port |
| fw-06-ssh-port-change-cmd1.png | `uci set dropbear.@dropbear[0].Port=2222` |
| fw-07-ssh-port-change-cmd2.png | `uci commit dropbear` |
| fw-08-ssh-port-change-cmd3.png | `service dropbear restart` |
| fw-09-ssh-port22-fails.png | SSH to port 22 failing after the change |
| fw-10-ssh-port2222-works.png | SSH to port 2222 succeeding |
| fw-11-icmp-before.png | Ping succeeding before ICMP block |
| fw-12-icmp-block-command.png | `iptables` command to drop ICMP on INPUT |
| fw-13-icmp-blocked.png | Ping timing out after ICMP block |
| fw-14-icmp-restore-command.png | `iptables` command to restore ICMP |
| fw-15-icmp-restored.png | Ping replies resuming after removal |
| fw-16-luci-port81-cmd1.png | Change LuCI listener to port 81 (`uci` command) |
| fw-17-luci-port81-cmd2.png | `uci commit uhttpd` |
| fw-18-luci-port81-cmd3.png | `service uhttpd restart` |
| fw-19-luci-port81-verify.png | LuCI login page on port 81 |
| fw-20-luci-restrict-command.png | `iptables` rule restricting port 81 to `192.168.56.1` |
| fw-21-install-curl.png | Installing `curl` to test the restriction |
| fw-22-luci-restrict-test.png | `curl` output showing blocked access for non-allowed IPs |

---

## hardening/

Screenshots for the hardening steps (used in `harden.md`) and the SSH key files.

### Key files (non-image)

| File | Description |
|------|-------------|
| coit20246_key.ppk | PuTTY private key (`.ppk`) generated with PuTTYgen |
| coit20246_key_public | Corresponding public key added to the router |

### Step 1 — Change default credentials

| File | Description |
|------|-------------|
| harden-01-change-password.png | `passwd` confirming the root password was set |

### Step 2 — Examine password storage

| File | Description |
|------|-------------|
| harden-02-shadow-file.png | `cat /etc/shadow` showing the root password as a SHA-256 hash |

### Step 3 — SSH key-based authentication

| File | Description |
|------|-------------|
| harden-03-puttygen-open.png | PuTTYgen ready to generate an RSA key |
| harden-04-puttygen-generate.png | PuTTYgen during key generation |
| harden-05-puttygen-key-done.png | Completed RSA key pair with comment `rsa-key-20260518` |
| harden-06-luci-open-browser.png | Opening LuCI at `http://192.168.56.2:81` |
| harden-07-luci-system-admin.png | Navigating to System → Administration in LuCI |
| harden-08-luci-administration.png | Administration page with SSH-Keys tab |
| harden-09-luci-ssh-keys-empty.png | SSH-Keys tab before adding a key |
| harden-10-luci-ssh-key-paste.png | Public key pasted into the SSH-Keys input |
| harden-11-luci-ssh-key-saved.png | RSA-2048 key saved with comment `rsa-key-20260518` |
| harden-12-putty-open.png | PuTTY ready to configure key-based login |
| harden-13-putty-host-port.png | PuTTY configured for host `192.168.56.2` and port `2222` |
| harden-14-putty-ssh-auth.png | PuTTY Auth settings for selecting the private key |
| harden-14b-putty-select-key.png | Selecting the private key file in PuTTY |
| harden-15-putty-accept-key.png | PuTTY host key prompt — accept to continue |
| harden-16-putty-login-prompt.png | Login prompt shown — no password required |
| harden-17-putty-key-login-success.png | Successful key-based login to the OpenWRT shell |

### Step 4 — Disable unnecessary services

| File | Description |
|------|-------------|
| harden-18-list-services-command.png | `ls -l /etc/rc.d/S*` listing startup services |
| harden-19-services-list.png | Output showing enabled services including `sysfixtime` |
| harden-20-sysfixtime-enabled.png | Status check showing `sysfixtime` enabled |
| harden-21-sysfixtime-disable.png | Command used to disable `sysfixtime` |
| harden-22-sysfixtime-stop.png | Command used to stop `sysfixtime` immediately |
| harden-23-sysfixtime-disabled.png | Status check confirming `sysfixtime` is disabled |

---

## traffic/

Screenshots and packet captures used in the traffic analysis section.

### Packet capture files

| File | Description |
|------|-------------|
| http_capture.pcap | HTTP capture between Windows host and OpenWRT |
| ssh_capture.pcap | SSH capture between Windows host and OpenWRT |

### HTTP traffic capture (traffic-01 to traffic-18)

| File | Description |
|------|-------------|
| traffic-01-install-tcpdump.png | Installing `tcpdump` on OpenWRT |
| traffic-02-tcpdump-installed.png | Confirmation `tcpdump` installed |
| traffic-03-http-capture-start.png | Starting HTTP capture on `br-mng` |
| traffic-04-http-capture-running.png | `tcpdump` capturing packets |
| traffic-05-website-during-capture.png | Website loaded while capture ran |
| traffic-06-http-capture-stop.png | `tcpdump` output after stopping capture |
| traffic-07-apk-update.png | `apk update` before installing SFTP server |
| traffic-08-sftp-server-install.png | Installing `openssh-sftp-server` |
| traffic-09-powershell-open.png | Windows PowerShell ready for SCP transfer |
| traffic-10-scp-transfer-http.png | SCP transferring `http_capture.pcap` to Windows |
| traffic-11-wireshark-open.png | Wireshark opened on Windows host |
| traffic-12-open-http-pcap.png | Opening `http_capture.pcap` in Wireshark |
| traffic-13-wireshark-http-loaded.png | HTTP capture loaded in Wireshark |
| traffic-14-http-filter-applied.png | Applying `http` display filter |
| traffic-15-http-packets-list.png | Filtered HTTP packet list |
| traffic-16-http-get-selected.png | HTTP GET packet selected |
| traffic-17-follow-http-stream.png | Follow → HTTP Stream showing plaintext HTML |
| traffic-18-http-stream-html.png | HTTP stream window with full HTML |

### SSH traffic capture (traffic-19 to traffic-26)

| File | Description |
|------|-------------|
| traffic-19-ssh-capture-start.png | Starting SSH capture on `br-mng` |
| traffic-20-putty-connect-ssh.png | PuTTY configured for SSH capture test |
| traffic-21-ssh-login.png | PuTTY session after login |
| traffic-22-ssh-commands.png | Commands typed to generate SSH traffic |
| traffic-23-ssh-capture-stop.png | `tcpdump` output after stopping SSH capture |
| traffic-24-scp-transfer-ssh.png | SCP transferring `ssh_capture.pcap` to Windows |
| traffic-25-open-ssh-pcap.png | Opening `ssh_capture.pcap` in Wireshark |
| traffic-26-wireshark-ssh-encrypted.png | SSH capture showing encrypted packets |
