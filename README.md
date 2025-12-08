# ☁️ Cloud-Native Data Center Fabric with Contatinerlab

![Platform](https://img.shields.io/badge/Platform-Nokia_SR_Linux-002d4d)
![Orchestration](https://img.shields.io/badge/Orchestration-Containerlab-success)
![Architecture](https://img.shields.io/badge/Architecture-IP_Clos_Fabric-red)
![Author](https://img.shields.io/badge/Author-Tomer_Philip-blue)

> A containerized Cloud-Native DC Fabric built with Nokia SR Linux (CNFs Routers). Implements a resilient Clos (Leaf-Spine) topology using BGP-EVPN/VXLAN. Fully automated via Containerlab to simulate hyperscale infrastructure suitable for Cloud and HPC workloads.
> 
> ### 🎯 Design Principles
* **🛡️ Resiliency:** Elimination of Single Points of Failure (SPOF) via Clos topology and  ECMP.
* **🚀 Performance:** Non-blocking East-West traffic optimization using VXLAN overlays.
* **🤸 Agility:** Seamless VM/Container *mobility* with Distributed Anycast Gateways.
* **🔐 Multi-Tenancy:** Hard isolation between tenants using VRFs and BGP-EVPN signaling.

**⚡ Simplicity: Spin up the entire Digital Twin lab with just one line of code!**

---

## 🏗️ Architecture Overview

The fabric is built on a **2-Stage IP Clos** topology, leveraging open standards (RFC-based) to ensure vendor interoperability and scalability.

| Component | Specification | Rationale |
| :--- | :--- | :--- |
| **Topology** | 2-Spine / 2-Leaf | Provides redundancy and demonstrates ECMP load balancing. Scalable horizontally. |
| **Underlay** | IS-IS Level 2 | Chosen for fast convergence, simplicity, and native Segment Routing support. |
| **Overlay** | MP-BGP EVPN | The industry standard control plane for MAC/IP learning, replacing legacy Flood-and-Learn. |
| **Data Plane** | VXLAN | Encapsulation protocol enabling Layer 2 extension over Layer 3 boundaries. |
| **NOS** | Nokia SR Linux | Open, model-driven, and Linux-native Network Operating System. |

### 🧩 Logical Topology (Clos)
*(Rendered live via Mermaid.js)*

```mermaid
graph TD
    %% --- Styles ---
    classDef spine fill:#2c3e50,stroke:#fff,stroke-width:2px,color:#fff,rx:5,ry:5,shadow:1;
    classDef leaf fill:#27ae60,stroke:#fff,stroke-width:2px,color:#fff,rx:5,ry:5,shadow:1;
    classDef server fill:#8e44ad,stroke:#fff,stroke-width:2px,color:#fff,rx:5,ry:5;
    
    %% --- Layer 1: Spines (The Core) ---
    subgraph SPINES ["🔥 Tier-1: Spines (Super Backbone)"]
        direction LR
        S1("<b>Spine-1</b><br/>RR & ECMP<br/>Loopback: 3.3.3.3"):::spine
        S2("<b>Spine-2</b><br/>RR & ECMP<br/>Loopback: 4.4.4.4"):::spine
    end

    %% --- Layer 2: Leafs (The Edge with Embedded Anycast) ---
    subgraph LEAFS ["🍃 Tier-2: Leafs (VTEP & Anycast Services)"]
        direction LR
        %% Leaf 1 Configuration
        L1("<b>Leaf-1</b><br/>Physical IP: 1.1.1.1<br/>__________________<br/><b>Anycast GW (VIP)</b><br/>IP: 192.168.10.254<br/>MAC: ...:01:01"):::leaf
        
        %% Leaf 2 Configuration (Identical GW!)
        L2("<b>Leaf-2</b><br/>Physical IP: 2.2.2.2<br/>__________________<br/><b>Anycast GW (VIP)</b><br/>IP: 192.168.10.254<br/>MAC: ...:01:01"):::leaf
    end

    %% --- Layer 3: Servers (Access) ---
    subgraph RACK1 ["Rack 1"]
        SRV1("<b>Server-1</b><br/>IP: 192.168.10.1<br/>GW: 192.168.10.254"):::server
    end

    subgraph RACK2 ["Rack 2"]
        SRV2("<b>Server-2</b><br/>IP: 192.168.10.2<br/>GW: 192.168.10.254"):::server
    end

    %% --- Connections: CLOS Fabric (Full Mesh) ---
    S1 === |"100G (IS-IS)"| L1
    S2 === |"100G (IS-IS)"| L1

    S1 === |"100G (IS-IS)"| L2
    S2 === |"100G (IS-IS)"| L2

    %% --- Connections: Access (Downlinks) ---
    L1 --- |"LACP / VLAN 10"| SRV1
    L2 --- |"LACP / VLAN 10"| SRV2

    %% --- Styling Links ---
    linkStyle 0,1,2,3 stroke:#2c3e50,stroke-width:3px;
    linkStyle 4,5 stroke:#95a5a6,stroke-width:2px;
```
### ⚡ Quick Start (Deploy in 60 Seconds)

Get the Digital Twin running on your laptop in **4 simple steps.**

> **Prerequisites:** `Linux VM /WSL2`, `Docker`, `Containerlab`.
>
> Don't have Containerlab? Install it: ```bash -c "$(curl -sL https://get.containerlab.dev)"```

### 1. Clone the Repository
Pull the infrastructure code and navigate to the project directory.

```bash
git clone https://github.com/YOUR_USERNAME/Cloud-Native-DC-Fabric-with-Contatinerlab.git
cd Cloud-Native-DC-Fabric-with-Contatinerlab
```
### 2. Deploy the Fabric
Spin up the topology. 
This command will pull the Docker images and pushes the **Zero Touch Provisioning (ZTP)** configs.

```bash
sudo containerlab deploy -t clab-topology/fabric.clab.yml --reconfigure
```
### 3. Visualize
Access the **Containerlab Graph GUI** to view the real-time network topology **in your browser.**

**Run the GUI server:** 
```bash
sudo containerlab graph -t clab-topology/fabric.clab.yml
```
**Open**: http://localhost:50080

### 4. Validation test
Verify the overlay data plane by pinging between the two isolated Servers over the VXLAN fabric.
```bash
docker exec -it clab-evpn01-srv1 ping 192.168.10.2
```
### 5. Destroy the lab! 🧹
Finished exploring? Gracefully stop and free up system.
```bash
sudo containerlab destroy -t clab-topology/fabric.clab.yml --cleanup
```

**🙌 Thank You!**

Thanks for deploying the Cloud-Native DC Fabric. I hope this provided valuable insights into modern, automated Data Center architectures.

**Feel free to ⭐ the repository if you found this engineering useful!**

### 🔮 Roadmap & Future Evolution
This project is designed to evolve from a static fabric to a fully observable, self-driving network.

**[x] Phase 1: Core Fabric Deployment**

    [x] Leaf-Spine Physical Topology

    [x] IS-IS Underlay
    
    [x] eBGP Underlay & EVPN-VXLAN Overlay
    
    [x] Anycast Gateway & Multi-Tenancy

**[ ] Phase 2: Modern Observability Stack**

   [ ] Streaming Telemetry via **gNMI** (replacing legacy SNMP)

    [ ] Data Collection with **Prometheus**

    [ ] Visualization Dashboards via **Grafana**

**[ ] Phase 3: Network Automation & NetOps**

    [ ] Day-2 Operations via Ansible Playbooks
    
    [ ] CI/CD Pipelines for Configuration Management (GitOps)
    
--- 
**Architected & Maintained by Tomer Philip.**
