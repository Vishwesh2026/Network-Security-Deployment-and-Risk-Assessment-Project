# Network Setup

## Assumptions

Before configuring the network we made a few assumptions that were not specified in the project brief.

- Business location: Melbourne, Victoria. This lets us consider realistic local threats (storms, power outages) in the risk assessment.
- Business type: a small professional services firm providing legal advisory and document management. This means the business handles sensitive client files, so confidentiality is high priority.
- Staff: eight people total — one IT support, two administrative staff, four legal consultants, and one senior partner. This size fits a single-router setup but still benefits from basic network segmentation.
- Website: a simple static HTML page served by `uhttpd` that shows the business name, a short description, and the students' names and IDs. No database or server-side scripting is required.
- Lab environment: Oracle VirtualBox on a Windows 11 host. The NAT adapter represents the WAN and the Host-Only adapter simulates the internal LAN; the Windows host acts as a staff workstation.

---

## The OpenWRT and VirtualBox Setup

### Starting the OpenWRT Virtual Machine

We downloaded the OpenWRT image, converted it to the x86-64 ext4-combined format, and created a VirtualBox VM with two network adapters: Adapter 1 (NAT) for WAN and Adapter 2 (Host-Only) for LAN. After starting the VM we pressed Enter at the console to access the root shell.

![OpenWRT first boot — console ready](/images/firewall/net-01-openwrt-boot.png)

### Setting the Root Password

OpenWRT boots with no root password by default. To secure the router we ran `passwd` and set a strong root password.

![Setting the root password using passwd](/images/firewall/net-02-set-password.png)

### Checking the Initial Interface State

We ran `ip addr` to inspect network interfaces before making changes.

![The Initial ip addr output showing default interface configurations](/images/firewall/net-03-ip-addr-initial.png)

By default `eth0` received a DHCP address from VirtualBox NAT (`10.0.2.15`) while `eth1` had no IP assigned. The default OpenWRT configuration kept everything on `eth0`, which did not match our two-adapter setup.

### Reconfiguring the Network Interfaces

We edited the network configuration with `vi`:

```
vi /etc/config/network
```

We replaced the defaults to move the LAN onto `eth1` (bridged as `br-mng`) and assigned `br-mng` the static IP `192.168.56.2`. The WAN (`eth0`) was left on DHCP so VirtualBox NAT continues to provide internet access.

![The Network configuration file after editing — new br-mng configuration](/images/firewall/net-05-network-config-edited.png)

```bash
config interface 'loopback'
	option device 'lo'
	option proto 'static'
	option ipaddr '127.0.0.1'
	option netmask '255.0.0.0'

config interface 'wan'
	option device 'eth0'
	option proto 'dhcp'

config interface 'lan'
	option device 'br-mng'
	option proto 'static'
	option ipaddr '192.168.56.2'
	option netmask '255.255.255.0'

config device
	option name 'br-mng'
	option type 'bridge'
	list ports 'eth1'
```

Save the file and restart networking:

```
service network restart
```

![To Run service network restart command to apply the new configuration](/images/firewall/net-06-network-restart.png)

### Verify the Interface Configuration

After restarting the network we ran `ip addr` to confirm the changes.

![The ip addr output showing br-mng at 192.168.56.2 and eth0 at 10.0.2.15](/images/firewall/net-07-ip-addr-final.png)

Summary:

- `eth0` — `10.0.2.15` (VirtualBox NAT, WAN)
- `eth1` — bridged into `br-mng` (no direct IP)
- `br-mng` — `192.168.56.2/24` (LAN/management)

IP allocations used in the lab:

| Interface              | Device Type            | IP Address         | Role                     |
| ---------------------- | ---------------------- | ------------------ | ------------------------ |
| `eth0`                 | VirtualBox NAT Adapter | `10.0.2.15` (DHCP) | WAN — internet access    |
| `eth1`                 | Host-Only Adapter      | (bridged)          | Physical LAN port        |
| `br-mng`               | Bridge over `eth1`     | `192.168.56.2/24`  | LAN management interface |
| Windows Host-Only      | VirtualBox Host-Only   | `192.168.56.1`     | Admin workstation        |
| VirtualBox NAT Gateway | VirtualBox internal    | `10.0.2.2`         | Default gateway for WAN  |

### Install the LuCI Web Interface

With networking configured we updated package lists and installed the LuCI web interface.

```bash
apk update
apk add luci
```

Enable and start the web server:

```bash
/etc/init.d/uhttpd enable
/etc/init.d/uhttpd start
```

![Starting with the uhttpd web server after installing the LuCI](/images/firewall/net-10-start-uhttpd.png)

We then opened `http://192.168.56.2` in the Windows host browser and logged into LuCI with the root credentials.

![The LuCI web interface loaded successfully in the Windows host browser](/images/firewall/net-11-luci-interface.png)

### Setting Up the Business Website

The project requires a simple website hosted on the router. We edited the default HTML file served by `uhttpd`:

```bash
vi /www/index.html
```

We replaced the content with a small static page containing the business name, a short description, and the student details.

![HTML content of the business website showing business name and student details](/images/firewall/net-13-website-html-content.png)

index.html file content:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>ProfServ Consulting-Brisbane</title>
    <style>
      body {
        font-family: sans-serif;
        background-color: #f4f4f4;
        text-align: center;
        padding: 50px;
      }
      .box {
        background: white;
        padding: 20px;
        border-radius: 10px;
        display: inline-block;
        box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
      }
      h1 {
        color: #2c3e50;
      }
      .luci-button {
        display: inline-block;
        padding: 12px 24px;
        background-color: #2c3e50;
        color: white;
        text-decoration: none;
        border-radius: 8px;
        font-weight: bold;
        box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
        transition: background-color 0.3s;
      }
      .luci-button:hover {
        background-color: #1a252f;
      }
    </style>
  </head>
  <body>
    <div class="box">
      <h1>ProfServ Consulting Pty Ltd</h1>
      <p>Providing professional services to local Brisbane clients.</p>
      <hr />
      <h3>Project Team</h3>
      <p><strong>Student 1:</strong> 12325286</p>
      <p><strong>Student 2:</strong> 12287775</p>
    </div>
    <br />
    <a href="cgi-bin/luci/" class="luci-button">Go to LuCI Admin Interface</a>
  </body>
</html>
```

### Lab Network Diagram

The diagram below shows the lab environment: the OpenWRT VM, network adapters, the bridge, and the Windows host.

![The Lab network diagram showing OpenWRT VM, network interfaces, and Windows host connection.](/images/Lab%20Network%20Diagram.png)

The draw.io source file is available as [`images/Lab Network Diagram.drawio`](/images/Lab%20Network%20Diagram.drawio).

---

## Firewall Configuration

Before adding firewall rules we installed the `iptables-nft` compatibility package so we could use familiar `iptables` commands on our OpenWRT build.

```bash
apk add iptables-nft
```

![Install the iptables-nft for the firewall rule management](/images/firewall/fw-00-iptables-nft-install.png)

We configured and tested four firewall rules required by the specification. For each rule we captured the state before and after applying the change and recorded the results.

---

### Rule 1: Block and Allow HTTP Traffic (Port 80)

**Purpose:** Demonstrate controlling access to the web server. Blocking port 80 simulates maintenance or an emergency shutdown.

**Before — website accessible:** The site at `http://192.168.56.2` loaded normally from the Windows host.

**Applying the block:**

```bash
iptables -I INPUT -p tcp --dport 80 -j DROP
```

![Enter the Command to block the HTTP traffic on port 80](/images/firewall/fw-01-http-block-command.png)

After applying the rule the browser timed out and the site was unreachable. To restore access remove the rule:

```bash
iptables -D INPUT -p tcp --dport 80 -j DROP
```

![Enter the Command to remove the HTTP block and then restore port 80 access](/images/firewall/fw-03-http-unblock-command.png)

**Applying the access restriction:**

```bash
iptables -I INPUT -p tcp --dport 81 ! -s 192.168.56.1 -j DROP
```

![The iptables rule to block all connections to port 81 except from 192.168.56.1](/images/firewall/fw-20-luci-restrict-command.png)

This rule drops all the TCP connections to port `81` unless the source IP address is `192.168.56.1` which is the Windows host. Any other machine on the network attempting to reach the management interface would be blocked.

**Testing the restriction:**

To test it without using the Windows host so we have installed curl on the OpenWRT router itself and tried to access port 81 from the router's own terminal (which has a source address of `127.0.0.1`, not the allowed `192.168.56.1`):

```bash
apk add curl
```

![Install the curl on OpenWRT to test the restriction](/images/firewall/fw-21-install-curl.png)

```bash
curl -I http://192.168.56.2:81
```

![The curl test from the OpenWRT showing the connection to port 81 is blocked for non-allowed sources](/images/firewall/fw-22-luci-restrict-test.png)

The curl command shows no response and confirming the rule was working only connections from `192.168.56.1` are permitted to reach the LuCI interface.

> **Note:** We were careful throughout this step to keep the Windows host (192.168.56.1) excluded from the DROP rule at all times, so we did not lock ourselves out of the management interface.

---

## Production Network Design

The lab setup has been described in the sections above is a simulation using VirtualBox's internal networking where it uses the `192.168.56.x` address range that is specific to the VirtualBox Host-Only adapter and does not reflect how a real office network would be structured.

For the production network design then we have drew a full network diagram showing how the actual business premises would be connected if the router were deployed in a real office environment. The production design includes:

- An OpenWRT router or firewall sitting at the boundary between the internet and the internal office
- A **LAN zone** (`86.1.1.0/24`) for all staff workstations, connected through a managed switch
- A **DMZ zone** (`86.1.2.0/24`) for the public-facing web server, isolated from the internal LAN
- Firewall rules are enforcing strict traffic separation between zones

**The IP Addressing for the Production Network**

As required by the project specification and all IP addresses in the production network must begin with the last two digits of one of the group members student IDs.

- Student 1's student ID is **12325286** → last two digits: **86**
- Student 2's student ID is **12287775** → last two digits: **75**

We have used **86** as the first octet for all production IP addresses and since `86` is already a documented first octet and makes addressing consistent across all zones.

| Zone        | Network      | Subnet Mask | Gateway    | Purpose            |
| ----------- | ------------ | ----------- | ---------- | ------------------ |
| WAN         | ISP-assigned | —           | ISP router | Internet uplink    |
| LAN (Staff) | `86.1.1.0`   | `/24`       | `86.1.1.1` | Staff workstations |
| DMZ (Web)   | `86.1.2.0`   | `/24`       | `86.1.2.1` | Public web server  |

**Device IP Allocation Table:**

| Device                   | Interface | IP Address   |
| ------------------------ | --------- | ------------ |
| OpenWRT Firewall — WAN   | `eth0`    | ISP-assigned |
| OpenWRT Firewall — LAN   | `eth1`    | `86.1.1.1`   |
| OpenWRT Firewall — DMZ   | `eth2`    | `86.1.2.1`   |
| Web Server (uhttpd)      | `eth0`    | `86.1.2.10`  |
| IT Support Workstation   | `eth0`    | `86.1.1.5`   |
| Admin Workstation 1      | `eth0`    | `86.1.1.10`  |
| Admin Workstation 2      | `eth0`    | `86.1.1.11`  |
| Consultant Workstation 1 | `eth0`    | `86.1.1.20`  |
| Consultant Workstation 2 | `eth0`    | `86.1.1.21`  |
| Consultant Workstation 3 | `eth0`    | `86.1.1.22`  |
| Consultant Workstation 4 | `eth0`    | `86.1.1.23`  |

The firewall obeyes the following traffic policy between zones:

- The Staff workstations on the LAN cannot initiate connections to the DMZ web server directly and they can access the website through the internet path.
- The web server in the DMZ cannot initiate any connections into the LAN.
- Only the IT Support Workstation (`86.1.1.5`) can reach both zones for maintenance and administration purposes.
- Inbound traffic from the internet is only permitted to port `80` and port `443` on the DMZ web server.

![The Production network diagram showing the full office network with LAN, DMZ, and internet zones](/images/Production%20Network%20Diagram.png)

> The draw.io source file for this diagram is available in the repository as [`images/Production Network Diagram.drawio`](/images/Production%20Network%20Diagram.drawio).
