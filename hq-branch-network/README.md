<div align="center">

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&weight=700&size=34&duration=3000&pause=100000&color=00D4FF&center=true&vCenter=true&repeat=false&width=750&height=60&lines=HQ+%26+Branch+Enterprise+Network" alt="HQ & Branch Enterprise Network" />

### 🏢 A secure two-site company network built with Cisco devices

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=17&pause=1000&color=8AB4F8&center=true&vCenter=true&width=700&lines=Cisco+ASA+firewall+with+inside+%2F+DMZ+%2F+outside+zones;VLANs%2C+router-on-a-stick%2C+DHCP+and+OSPF;NAT%2C+a+public+website+in+the+DMZ+and+guest+isolation;SSH-only+management+from+one+admin+VLAN" alt="Typing animation" />

![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer%209.0-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Firewall](https://img.shields.io/badge/Firewall-ASA%205506--X-C0392B?style=for-the-badge)
![Routing](https://img.shields.io/badge/Routing-OSPF-2E8B57?style=for-the-badge)
![Security](https://img.shields.io/badge/Security-NAT%20%7C%20ACLs%20%7C%20SSH-8E44AD?style=for-the-badge)

</div>

---

## 📖 The scenario

> A company has a **head office (HQ)** and a **branch office**. I designed and built their network from scratch:
> a **Cisco ASA firewall** protecting HQ, a **DMZ** for the public website, **VLANs** separating staff, guests
> and management, and a **private WAN link with OSPF** connecting the branch.
> Branch users reach the internet **through HQ's firewall**, so security is managed in one place.

Built step by step in **8 phases**, configured entirely through the **Cisco CLI**, and verified with **20+ tests**
backed by **30 screenshots**.

---

## 🗺️ Topology

> 📄 The full Packet Tracer topology is the first page of **[screenshots.pdf](screenshots.pdf)** (`01-topology`).

```text
                 Internet-Server (198.51.100.10)
                          │
                    [ ISP router ]
                          │ 203.0.113.0/30
                 ┌──[ HQ-FW  ASA ]──── DMZ-Web (192.168.50.10)
                 │   inside│
                 │   [ HQ-R1 ]────── WAN 172.16.0.0/30 ──────[ BR-R1 ]
                 │      │ trunk                                  │
                 │   [ HQ-SW1 ]                              [ BR-SW1 ]
                 │   VLAN 10 / 20 / 99                       VLAN 30
                 │   Staff · Guest · Admin                   Branch PCs
```

---

## 🧰 Devices

| Device | Model | Role |
|---|---|---|
| **HQ-FW** | Cisco ASA 5506-X | Firewall: inside / DMZ / outside zones, NAT, access rules |
| **HQ-R1** | Cisco ISR 2911 | HQ router: inter-VLAN routing, DHCP, OSPF, guest ACL |
| **BR-R1** | Cisco ISR 4331 | Branch router: DHCP, OSPF |
| **HQ-SW1** | Cisco Catalyst 2960 | HQ switch: VLANs 10 / 20 / 99, 802.1Q trunk |
| **BR-SW1** | Cisco Catalyst 2960 | Branch switch: VLAN 30 |
| **ISP** | Cisco ISR 4331 | Simulated internet provider |
| **DMZ-Web** | Server | Company's public website |
| **Internet-Server** | Server | Simulated internet (website + outside tester) |
| 6 end devices | PCs / laptop | Staff, guest, admin and branch users |

---

## 🌐 IP addressing plan

Addresses follow a pattern: **10.`site`.`VLAN`.`host`** (site 10 = HQ, site 20 = branch).

| Network | Subnet | Gateway | Purpose |
|---|---|---|---|
| VLAN 10: Staff | 10.10.10.0/24 | 10.10.10.1 | HQ staff PCs (DHCP) |
| VLAN 20: Guest | 10.10.20.0/24 | 10.10.20.1 | Guests: internet only (DHCP) |
| VLAN 99: Management | 10.10.99.0/24 | 10.10.99.1 | Admin PC + switch management (10.10.99.2) |
| VLAN 30: Branch | 10.20.30.0/24 | 10.20.30.1 | Branch PCs (DHCP) |
| WAN link | 172.16.0.0/30 | – | HQ-R1 (.1) ↔ BR-R1 (.2) |
| Firewall inside link | 10.0.0.0/30 | – | HQ-FW (.1) ↔ HQ-R1 (.2) |
| DMZ | 192.168.50.0/24 | 192.168.50.1 | Public web server (.10) |
| Firewall outside link | 203.0.113.0/30 | – | ISP (.1) ↔ HQ-FW (.2, company public IP) |
| Public server block | 203.0.113.8/29 | – | DMZ website published as **203.0.113.10** |
| "Internet" | 198.51.100.0/24 | 198.51.100.1 | Simulated internet (server .10) |

> `.1–.10` are excluded from DHCP in every user VLAN and reserved for gateways and network devices.

---

## 🛡️ Security design

### Firewall zones (Cisco ASA)
| Zone | Interface | Security level | Connected to |
|---|---|:---:|---|
| **outside** | Gi1/1 | 0 | ISP / internet |
| **dmz** | Gi1/3 | 50 | Public web server |
| **inside** | Gi1/2 | 100 | HQ-R1 → all office networks |

Traffic flows from higher to lower security levels by default. Anything else needs an explicit rule.

### Who can reach what
| From → To | Allowed? | How |
|---|:---:|---|
| Staff & branch → internet | ✅ | PAT to the firewall's public IP |
| Internet → company website | ✅ | Static NAT 203.0.113.10 → 192.168.50.10 + ACL (TCP 80/443 only) |
| Internet → office PCs | ❌ | Blocked by the firewall |
| DMZ → office PCs | ❌ | DMZ (50) can't start connections to inside (100) |
| Guest → internet | ✅ | Allowed by the guest ACL |
| Guest → staff / admin / branch | ❌ | Guest ACL on HQ-R1 |
| Admin VLAN → device management (SSH) | ✅ | VTY access restricted to 10.10.99.0/24 |
| Anyone else → device management | ❌ | SSH refused, Telnet disabled |

### Hardening checklist
- ✅ `enable secret` and local user accounts (hashed passwords) on every router and switch
- ✅ Console login required on every router and switch
- ✅ **SSH version 2 only**: Telnet disabled on all management lines
- ✅ Management restricted to the **admin VLAN** with an access-class ACL
- ✅ Guest VLAN isolated from every internal network
- ✅ **Unused switch ports shut down**
- ✅ Public server isolated in a **DMZ**, exposed only on web ports (least privilege)

---

## ⚙️ How it was built: 8 phases

| Phase | What I built | Key technologies |
|:---:|---|---|
| 1 | Placed, named and cabled 14 devices to a port plan | Topology design, cabling |
| 2 | Created VLANs and an 802.1Q trunk | VLANs, access/trunk ports |
| 3 | Inter-VLAN routing + DHCP | Router-on-a-stick, DHCP pools |
| 4 | Branch site + WAN link + dynamic routing | OSPF (area 0), /30 links |
| 5 | Firewall zones and routing | ASA nameif, security levels, static/default routes |
| 6 | Internet access + public website | PAT, static NAT, ACLs, stateful inspection |
| 7 | Security hardening | Guest ACL, SSH, passwords, port security |
| 8 | Testing + documentation | Test plan, configs, this README |

---

## ✅ Test results

> 📄 Every screenshot is in **[screenshots.pdf](screenshots.pdf)**, each one titled with the name shown in the last column.

| # | Test | Expected | Result | Screenshot in the PDF |
|:---:|---|:---:|:---:|---|
| 1 | VLANs 10 / 20 / 99 on the HQ switch | ✅ | ✅ | `02-vlans` |
| 2 | VLAN 30 on the branch switch | ✅ | ✅ | `02b-branch-vlans` |
| 3 | PCs receive addresses from DHCP | ✅ | ✅ | `03-dhcp` · `04-dhcp-bindings` |
| 4 | OSPF neighbors reach FULL state (both routers) | ✅ | ✅ | `05-ospf-neighbor` · `05b-ospf-neighbor-branch` |
| 5 | HQ learns the branch network via OSPF | ✅ | ✅ | `06-ospf-routes` |
| 6 | Branch PC → HQ staff PC | ✅ | ✅ | `07-branch-to-hq-ping` |
| 7 | Firewall zones up (inside / DMZ / outside) | ✅ | ✅ | `08-firewall-interfaces` |
| 8 | Firewall routing table complete | ✅ | ✅ | `08b-firewall-routes-before-inside-up` · `09-firewall-routes` |
| 9 | Branch learns the default route via OSPF | ✅ | ✅ | `10-branch-default-route` |
| 10 | Office browses the internet (PAT) | ✅ | ✅ | `11-office-browsing-internet` |
| 11 | NAT translating in both directions | ✅ | ✅ | `12-nat-translations` |
| 12 | Internet opens the company website | ✅ | ✅ | `13-dmz-website-from-internet` |
| 13 | Internet → office PC | ❌ blocked | ✅ blocked | `14-internet-to-office-blocked` |
| 14 | DMZ server → office PC | ❌ blocked | ✅ blocked | `15-dmz-to-office-blocked` |
| 15 | Staff opens the company website | ✅ | ✅ | `16-staff-to-dmz-website` |
| 16 | Guest → staff PC | ❌ blocked | ✅ blocked | `17-guest-to-staff-blocked` |
| 17 | Guest → internet | ✅ | ✅ | `18-guest-internet-ok` |
| 18 | Guest ACL counters prove the block | ✅ | ✅ | `19-guest-acl-matches` |
| 19 | Admin PC → SSH to HQ router | ✅ | ✅ | `20-admin-ssh-success` |
| 20 | Telnet to the router | ❌ refused | ✅ refused | `20b-telnet-refused` |
| 21 | Staff PC → SSH to the router | ❌ refused | ✅ refused | `21-staff-ssh-denied` |
| 22 | Console requires username + password | ✅ | ✅ | `21b-console-login-required` |
| 23 | Admin PC → SSH to branch router (over the WAN) | ✅ | ✅ | `21c-admin-ssh-branch-router` |
| 24 | Admin PC → SSH to the switch | ✅ | ✅ | `22-admin-ssh-switch` |
| 25 | Unused switch ports disabled | ✅ | ✅ | `23-unused-ports-disabled` |
| 26 | Staff PC → SSH to the switch | ❌ refused | ✅ refused | `23b-staff-ssh-switch-denied` |

> The "blocked" and "refused" results are just as important as the successes: they prove the security rules work.

---

## 🔧 Troubleshooting log: real problems I solved

| Problem | How I found it | Fix |
|---|---|---|
| Router model had only 2 network ports, but HQ needed 3 | Checked the hardware before cabling | Swapped HQ-R1 to a Cisco 2911 (3 Gigabit ports) |
| Branch PCs got no IP address (DHCP failed) | Pinged outward hop by hop, then read the config | A typo in `ip dhcp excluded-address` excluded the whole pool. Removed it. |
| `no` command didn't remove the bad line | Compared it word by word with `show running-config` | The command had an extra word. Retyped the exact line. |
| Branch router lost its configuration | Prompt showed `Router>` and an empty routing table | Restored from the saved startup-config |
| Office pings to the internet were translated but no reply came back | `show nat` / `show xlate` showed outbound hits only | Enabled stateful inspection (`inspect icmp`, `inspect http`) |
| A guest "block" looked like it worked, but the laptop had no IP | Verified with `ipconfig` and ACL match counters | Renewed DHCP, applied the ACL to the right sub-interface, re-tested |

---

## 📁 Repository contents

```text
hq-branch-network/
├── README.md                 ← this page
├── hq-branch-network.pkt     ← Cisco Packet Tracer file (open with Packet Tracer 9.0+)
├── configs/                  ← running configuration of every device
│   ├── ISP.txt
│   ├── HQ-FW.txt
│   ├── HQ-R1.txt
│   ├── BR-R1.txt
│   ├── HQ-SW1.txt
│   └── BR-SW1.txt
└── screenshots.pdf           ← 30 titled screenshots: evidence for every test
```

### Open it yourself
1. Install **Cisco Packet Tracer 9.0+** (free from [netacad.com](https://www.netacad.com))
2. Download `hq-branch-network.pkt` and open it
3. Routers and switches require a login (lab-only credentials, not published here)

---

## 🎓 Skills demonstrated

`VLANs` · `802.1Q trunking` · `Router-on-a-stick` · `DHCP` · `OSPF` · `Static & default routing` ·
`Cisco ASA zones & security levels` · `PAT / static NAT` · `ACLs` · `DMZ design` · `Stateful inspection` ·
`SSH hardening` · `Least privilege` · `Structured troubleshooting` · `Network documentation`

---

<div align="center">

**Built by Urvisha Bhoi** · part of **[Route & Rule](https://github.com/route-and-rule)** 🌐

*"Every network is only as strong as its weakest rule."*

</div>
