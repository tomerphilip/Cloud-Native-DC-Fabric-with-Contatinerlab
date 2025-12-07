# 📑 LLD-02: Underlay Routing Strategy (IS-IS)

| Document Meta | Details |
| :--- | :--- |
| **Project** | Cloud-Native DC Fabric |
| **Scope** | Layer 3 Routing (Underlay) |
| **Protocol** | IS-IS Level 2 |
| **Status** | vesion 1 |

---

## 1. Design Rationale
We utilize **IS-IS (Intermediate System to Intermediate System)** as the IGP for the fabric underlay.
* **Why IS-IS?** It offers neutral encapsulation, Segment Routing ready, and rapid convergence suitable for AI/HPC fabrics.

## 2. Protocol Configuration

### 2.1 Global Settings
* **Level Capability:** `L2` (Flat backbone topology).
* **Area ID:** `49.0001` (Private Area).
* **Metric Style:** Wide (Default in SR Linux).

### 2.2 NET ID Assignment
The Network Entity Title (NET) is constructed as: `AreaID + SystemID + Selector`.

| Node | System ID | Full NET ID |
| :--- | :--- | :--- |
| **Leaf1** | `...0001` | `49.0001.0000.0000.0001.00` |
| **Leaf2** | `...0002` | `49.0001.0000.0000.0002.00` |
| **Spine1** | `...0003` | `49.0001.0000.0000.0003.00` |
| **Spine2** | `...0004` | `49.0001.0000.0000.0004.00` |

## 3. Adjacency Optimization
* **Point-to-Point (P2P):** All fabric links are configured as `circuit-type: point-to-point`. This disables the DIS (Designated IS) election process, speeding up convergence.
* **Passive Interfaces:** The System Loopback (`system0.0`) is advertised but configured as `passive` to prevent unnecessary adjacency formation attempts.

---
**End of Document**
