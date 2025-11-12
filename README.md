# devops-stage4-vpc-simulator
Repository for HNG13 stages 4 devops task
# 🛰️ Stage 4: Virtual Private Cloud (VPC) Simulation on Linux

**Author:** Yinusa Kolawole (KolaDevOps)  
**Track:** DevOps Internship — Stage 4  
**Goal:** Recreate a Virtual Private Cloud (VPC) environment using native Linux networking primitives — no cloud, no Docker, no magic.

---

## 🧩 Overview

This project simulates a **cloud-style VPC** locally using:
- **Network namespaces** for subnets
- **veth pairs** for connectivity
- **Linux bridge** as a central router
- **iptables** for NAT and security rules
- **A Bash CLI tool (`vpcctl`)** to automate everything

It mirrors how AWS or GCP VPCs handle networking under the hood — supporting **subnets, routing, NAT gateways, firewall rules, and VPC peering**.

---

## 🧱 Architecture

### Core Components
| Component | Description |
|------------|-------------|
| `ip netns` | Used to isolate subnets (like EC2 instances per subnet) |
| `bridge` | Simulates the VPC router |
| `veth` | Connects subnets to the router |
| `iptables` | Handles NAT, forwarding, and firewall rules |

### Example Layout

 ┌────────────────────────────┐
 │         br-myvpc           │  ← central bridge (router)
 │   10.0.0.1/16              │
 └────────────┬───────────────┘
              │

┌──────────────┼──────────────┐
│ │
┌──────────────┐ ┌──────────────┐
│ ns-myvpc-public │ │ ns-myvpc-private │
│ 10.0.1.0/24 │ │ 10.0.2.0/24 │
│ NAT enabled │ │ Internal only │
└──────────────┘ └──────────────┘


---

## ⚙️ Features Implemented

✅ Create / delete virtual VPCs with CIDR  
✅ Add public and private subnets  
✅ Automatic IP and route configuration  
✅ NAT gateway simulation (public subnet → Internet)  
✅ Internal routing between subnets  
✅ Multiple VPCs with full isolation  
✅ VPC peering  
✅ JSON-based firewall policy support  
✅ Full cleanup lifecycle (namespaces, bridges, veths, firewall)  

---

## 🚀 CLI Usage

### Create a VPC
```bash
sudo ./vpcctl create myvpc 10.0.0.0/16
```
Add Subnets

```bash
sudo ./vpcctl add-subnet myvpc public 10.0.1.0/24 public
sudo ./vpcctl add-subnet myvpc private 10.0.2.0/24 private
```

Test Connectivity

```bash
sudo ip netns exec ns-myvpc-public ping -c 2 10.0.2.2
sudo ip netns exec ns-myvpc-public ping -c 2 8.8.8.8
```

Apply Firewall Rules

```public-policy.json```
```bash
{
  "subnet": "10.0.1.0/24",
  "ingress": [
    { "port": 80, "protocol": "tcp", "action": "allow" },
    { "port": 22, "protocol": "tcp", "action": "allow" }
  ]
}
```

Apply policy:
```bash
sudo ./vpcctl apply-firewall myvpc public public-policy.json
```

VPC Peering
```bash
sudo ./vpcctl peer vpcA vpcB
```

Delete VPC
```bash
sudo ./vpcctl delete myvpc
```

Verification Steps

| Feature           | Test                           | Result    |
| ----------------- | ------------------------------ | --------- |
| Subnet Routing    | `ping` between namespaces      | ✅ Success |
| Internet Access   | `ns-myvpc-public ping 8.8.8.8` | ✅ Success |
| Private Isolation | Blocked via iptables           | ✅ Success |
| VPC Peering       | `vpcA` ↔ `vpcB` reachable      | ✅ Success |
| Firewall Rules    | Applied with JSON policy       | ✅ Success |
| Cleanup           | No leftover bridges/namespaces | ✅ Success |


Teardown & Idempotency

```bash
sudo ./vpcctl delete myvpc
```

Then confirm:
```bash
ip netns list       # none
ip link show | grep br-
sudo iptables -t nat -S POSTROUTING
```

📄 Tech Stack

- Language: Bash

- Tools: ip, iptables, bridge-utils, jq

- Tested On: Ubuntu 22.04 LTS

© 2025 Yinusa Kolawole (KolaDevOps)