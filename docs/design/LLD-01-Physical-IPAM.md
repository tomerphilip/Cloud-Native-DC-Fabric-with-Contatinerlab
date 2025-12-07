# 📑 LLD-01: Physical Topology & IPAM

| Document Meta | Details |
| :--- | :--- |
| **Project** | Cloud-Native DC Fabric |
| **Scope** | Layer 1 (Physical) & Layer 3 (Addressing) |
| **Author** | Tomer Philip |
| **Status** | vesion 1.0 |

---

## 1. Physical Architecture
The fabric follows a **2-Stage Folded Clos (Leaf-Spine)** topology to ensure non-blocking performance and high availability.

### 1.1 Node Roles & Hardware
| Role | Hostname | Hardware Type | Function |
| :--- | :--- | :--- | :--- |
| **Spine** | `spine1`, `spine2` | Nokia IXR-D3 | **Backbone:** High-speed IP transport. **Control:** BGP Route Reflector. |
| **Leaf** | `leaf1`, `leaf2` | Nokia IXR-D3 | **Edge:** VTEP (VXLAN Tunnel Endpoint), Services, Anycast Gateway. |
| **Server** | `srv1`, `srv2` | Linux Host | Workload simulation. |

### 1.2 Cabling Plan (Interconnects)
All inter-switch links are point-to-point Layer 3 links.

| Source | Port | Destination | Port | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| **Leaf1** | `e1-2` | **Spine1** | `e1-1` | Fabric Uplink 1 |
| **Leaf1** | `e1-3` | **Spine2** | `e1-1` | Fabric Uplink 2 (ECMP) |
| **Leaf2** | `e1-2` | **Spine1** | `e1-2` | Fabric Uplink 1 |
| **Leaf2** | `e1-3` | **Spine2** | `e1-2` | Fabric Uplink 2 (ECMP) |
| **Leaf1** | `e1-1` | **Srv1** | `eth1` | Access Port (Untagged) |
| **Leaf2** | `e1-1` | **Srv2** | `eth1` | Access Port (Untagged) |

---

## 2. IP Address Management (IPAM)

### 2.1 System Loopbacks (Router-ID & VTEP Source)
/32 addresses used for stable identification and VXLAN tunneling source.

| Node | Loopback IP (system0.0) |
| :--- | :--- |
| **Leaf1** | `1.1.1.1/32` |
| **Leaf2** | `2.2.2.2/32` |
| **Spine1** | `3.3.3.3/32` |
| **Spine2** | `4.4.4.4/32` |

### 2.2 Point-to-Point Links (Underlay)
/30 subnets for minimizing broadcast domains on fabric links.

| Link | Subnet | IP (Side A) | IP (Side B) |
| :--- | :--- | :--- | :--- |
| Leaf1 <-> Spine1 | `10.1.1.0/30` | `.2` (Leaf1) | `.1` (Spine1) |
| Leaf1 <-> Spine2 | `10.1.1.4/30` | `.6` (Leaf1) | `.5` (Spine2) |
| Leaf2 <-> Spine1 | `10.1.2.0/30` | `.2` (Leaf2) | `.1` (Spine1) |
| Leaf2 <-> Spine2 | `10.1.2.4/30` | `.6` (Leaf2) | `.5` (Spine2) |

---
**End of Document**
