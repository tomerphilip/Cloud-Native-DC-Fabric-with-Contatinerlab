# 📑 LLD-03: Overlay & Services (EVPN-VXLAN)

| Document Meta | Details |
| :--- | :--- |
| **Project** | Cloud-Native DC Fabric |
| **Scope** | Overlay Control Plane & Tenant Services |
| **Protocols** | MP-BGP EVPN, VXLAN |
| **Status** | vesion 1.0 |

---

## 1. Overlay Control Plane (MP-BGP)
We implement **iBGP** with Route Reflectors to distribute EVPN signaling (MAC/IP reachability) without requiring a full mesh.

### 1.1 BGP Architecture
* **ASN:** `65000` (Internal BGP).
* **Address Families:** `evpn`, `ipv4-unicast`.
* **Route Reflectors (RR):** Spine1 (`3.3.3.3`) and Spine2 (`4.4.4.4`).
* **RR Clients:** Leaf1 and Leaf2.

---

## 2. Data Plane & Encapsulation
* **Protocol:** VXLAN (RFC 7348).
* **Tunnel Interface:** `vxlan0`.
* **VTEP Source:** System Loopback (`1.1.1.1`, `2.2.2.2`).

---

## 3. Multi-Tenancy Design (The "Service")

The fabric implements a **Symmetric IRB** (Integrated Routing and Bridging) model using two VNI types for strict tenant isolation.

### 3.1 Tenant Profile: `tenant-1`

| Component | Identifier | Details |
| :--- | :--- | :--- |
| **Layer 2 Domain** | **MAC-VRF** | `network-instance: tenant-1` |
| **L2 VNI** | **`10`** | Carries L2 traffic (Bridged). Mapped to VLAN 10. |
| **Layer 3 Domain** | **IP-VRF** | `network-instance: ip-vrf-1` |
| **L3 VNI** | **`20`** | Carries L3 traffic (Routed) between subnets. |
| **Route Target** | `65000:10` | Used for Import/Export of EVPN routes. |

### 3.2 Access Layer (Server Connectivity)
* **Port:** `ethernet-1/1`
* **VLAN Encapsulation:** **Untagged** (Traffic enters the fabric as standard Ethernet frames and is mapped internally to VNI 10).

---

## 4. Distributed Anycast Gateway (DAG)

To enable seamless workload mobility, we implement a distributed gateway. The **same IP and MAC** exist simultaneously on all Leaf switches.

### 4.1 Gateway Parameters
* **Interface:** `irb0` (Integrated Routing and Bridging).
* **Gateway IP:** `192.168.10.254/24`
* **Virtual MAC:** `00:00:5E:00:01:01` (VRRP standard MAC).
* **Feature:** `arp learn-unsolicited true` (Optimizes mobility detection).

### 4.2 Traffic Flow

1. **VM A (Leaf1)** sends traffic to GW `192.168.10.254`.
2. **Leaf1** answers locally (Layer 3 lookup).
3. If dest is remote, Leaf1 encapsulates in **VXLAN (L3 VNI 20)**.
4. Packet is routed over IS-IS to Leaf2.
5. Leaf2 decapsulates and delivers to VM B.

---
**End of Document**
