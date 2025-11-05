# Network-Simulation
Cisco Packet Tracer build for a campus network: VLSM IPv4 design, VLANs &amp; inter-VLAN routing, OSPFv2, dual-ISP with floating static failover, and NAT (PAT + static).

# Network Design & Implementation

---

## 1) Project Overview

This project designs and implements a complete IPv4 campus network. It covers hierarchical switching (VLANs, 802.1Q trunking, inter‑VLAN routing), interior routing with **OSPFv2**, **static & floating static** routing to dual ISPs, and **NAT** (PAT + 1:1 static) at the borders. The prototype is built and verified in **Cisco Packet Tracer**.

---

## 2) Objectives Achieved

- Design a VLSM‑optimised IPv4 addressing plan for all LANs, SVIs and point‑to‑point links.
- Implement OSPFv2 for internal routing, including redistribution of connected interfaces where required.
- Configure static, default static, and floating static routes for dual‑ISP failover.
- Build switching with VLANs (Students, Staff, Spare, Management), 802.1Q trunks, and inter‑VLAN routing.
- Configure NAT: PAT (overload) for user egress + 1:1 static NAT for the Server host.
- Verify end‑to‑end connectivity, redundancy behaviour, and NAT operation.

---

## 3) Assigned Address Blocks (Group E, normalised to individual work)

- **Internal Private Supernet:** `172.17.112.0/21`
- **Public /29 for ISP Point‑to‑Point links:** `209.165.199.112/29`
- **Public /28 for NAT pool:** `209.165.199.224/28`

---

## 4) Topology Summary (High Level)

- **City Campus:** Main Building (Server Farm + border to ISP), West Tower, East Tower (redundant L3 switching pathing).
- **Branch Campus:** Linked to City via WAN (Serial).
- **Two ISP links:** Primary via Main; Secondary (backup) via Branch.
- **Redundancy goals:**
    - Switching users have two exit points (West default, East backup).
    - Internet egress prefers Main‑ISP; falls back to Branch‑ISP if Main fails.

> In Packet Tracer, the “Internet” is represented by an ISP router with loopback 1.1.1.1/32 for testing reachability.
> 

---

## 5) VLSM Addressing Plan (Individual Build)

### 5.1 LAN subnets (from `172.17.112.0/21`)

| Purpose | Hosts Req. | Chosen Prefix | Subnet | Host Range (usable) | Notes |
| --- | --- | --- | --- | --- | --- |
| West Students (VLAN 10) | 500 | /23 | **172.17.112.0/23** | 172.17.112.1 – 172.17.113.254 | GW = .1 |
| Branch Spare (VLAN 30) | 300 | /23 | **172.17.114.0/23** | 172.17.114.1 – 172.17.115.254 | GW = .1 |
| Branch Students (VLAN 10) | 200 | /24 | **172.17.116.0/24** | 172.17.116.1 – 172.17.116.254 | GW = .1 |
| West Staff (VLAN 20) | 120 | /25 | **172.17.117.0/25** | 172.17.117.1 – 172.17.117.126 | GW = .1 |
| Server Farm (Main) | 10 | /28 | **172.17.117.128/28** | 172.17.117.129 – 172.17.117.142 | GW = .129 |
| Mgmt VLAN (City, VLAN 99) | ~5+ | /27 | **172.17.117.160/27** | 172.17.117.161 – 172.17.117.190 | GW = .161 |
| Mgmt VLAN (Branch, VLAN 99) | ~5+ | /27 | **172.17.117.192/27** | 172.17.117.193 – 172.17.117.222 | GW = .193 |

### 5.2 Transit /30 pool for internal point‑to‑point links

Reserve **`172.17.118.0/24`** as a /30 pool for serial links and routed links between distribution/core.

(Each /30 provides 2 usable IPs; allocate per link as required in the Packet Tracer file.)

### 5.3 ISP point‑to‑point addressing (/29 → two /30s)

From **`209.165.199.112/29`**:

- **Main–ISP link:** `209.165.199.112/30` → usable: **.113** (Main border), **.114** (ISP)
- **Branch–ISP link:** `209.165.199.116/30` → usable: **.117** (Branch border), **.118** (ISP)

### 5.4 NAT Public Pool (/28)

- **NAT pool supernet:** `209.165.199.224/28` → usable **.225 – .238**
- **Static NAT for Server (PC3):** **`209.165.199.225` ↔ `172.17.117.129`**
- **PAT (overload):** users egress via outside interface; fallback mirrors on Branch.

---

## 6) VLANs, Trunks, and Inter‑VLAN Routing

- VLANs: **10 Students**, **20 Staff**, **30 Spare**, **99 Management** (native VLAN = **555** on trunks).
- Allocate at least 5 access ports per user VLAN on each switch; 1 port for VLAN 99 per switch.
- Create SVIs on the L3 switch/router subinterfaces for each VLAN; use **.1** (or the listed GW) as the SVI IP.
- Disable VLAN 1 and unknown VLANs on trunks; set **allowed VLANs** explicitly.

---

## 7) OSPFv2 + Static / Floating Static

- Single OSPF process for internal IPv4; advertise all LANs/transits; passive on user‑facing SVIs.
- **Primary default** towards Main‑ISP; **floating static** towards Branch‑ISP with a higher AD.

---

## 8) NAT (PAT and Static) at Borders

**Main border (outside = Main–ISP P2P):**

**Branch border mirrors** the above with `ip nat outside` on the Branch–ISP interface to preserve egress during failover (optionally using the same pool if shared via provider, or PAT on interface IP for simplicity in PT labs).

---

## 9) ISP Router (Internet)

- Loopback to emulate Internet: `interface lo0` → `ip address 1.1.1.1 255.255.255.255`
- Standard static routes **only when needed** to the AIT internal supernet(s). For Packet Tracer, return routes can be summarised or pointed to the Main/Branch P2P next‑hops.

---

## 10) Device Hardening & Base Settings

- Hostnames per diagram; console/vty passwords; encrypted enable secret; no DNS lookup; logging synchronous; SSH enabled on mgmt SVIs.

---

## 11) Verification & Test Plan

- **Interface status:** `show ip interface brief`
- **VLANs/SVIs:** `show vlan brief`, `show interfaces trunk`, `show ip interface vlan <id>`
- **Routing:** `show ip route`, `show ip ospf neighbor`, `show ip protocols`
- **NAT:** `show ip nat translations`, `show ip nat statistics`
- **Connectivity:** `ping`/`traceroute` among hosts, to **1.1.1.1**, and to the server’s public IP.
- **Failover:** shut the Main–ISP interface; confirm default switches to Branch, users maintain Internet via NAT on Branch border.

---

## 12) Repository Structure (suggested)

```
Network-Simulation/
├── README.md  ← this file
├── PacketTracer/
│   └── Network.pkt
├── addressing/
│   ├── addressing.pdf
├── configs/
│   ├── main_router.txt
│   ├── branch_router.txt
│   ├── west_router.txt
│   └── east_router.txt
│   └── isp_router.txt
│   └── thomas_switch.txt
│   └── jones_switch.txt

```

---

## 13) How to Run

1. Open the `.pkt` file in Cisco Packet Tracer.
2. Verify base configs (hostnames, passwords, mgmt VLAN IP).
3. Confirm OSPF adjacencies and route population.
4. Test user VLAN connectivity and inter‑VLAN routing.
5. Test NAT: from a student PC, `ping 1.1.1.1` and observe PAT.
6. Induce failover by shutting the Main–ISP interface; confirm egress via Branch.

---

## 14) Notes

- Addressing is VLSM‑optimised and stays fully within `172.17.112.0/21`.
- Public addressing follows the assigned Group E ranges while presented as an individual build.
- In Packet Tracer, ACL simplifications or single‑pool NAT are acceptable for demonstration.

---
