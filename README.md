# Azure-Secure-Firewall
 Designed and implemented a secure cloud network architecture on Microsoft Azure, deploying a private Nginx web application fully protected behind an enterprise-grade firewall.

This repository documents a hands-on Azure project where I designed and deployed a **private Nginx web application** hosted on a virtual machine, fully protected behind **Azure Firewall**, **Azure Bastion**, and **Network Security Groups (NSGs)**.

The goal was to build a **production-style secure network architecture** where:

- The VM has **no public IP**.
- All HTTP access is **proxied through Azure Firewall** via a DNAT rule.
- Administrative access is provided **only through Azure Bastion**.
- **NSGs enforce least privilege** at the subnet/NIC level.

---

## 🔐 High-Level Architecture

Key components:

- **Virtual Network (VNet)**
  - Contains subnets for **Firewall**, **Bastion**, and **Application (Subnet-1)**.
- **Azure Firewall**
  - Exposed to the internet with a **public IP**.
  - Implements a **DNAT rule** to forward traffic from `FirewallPublicIP:4000` → `NginxVM:80`.
  - DNAT is locked down to **only my laptop's public IP** as the source.
- **Azure Bastion**
  - Deployed in its own subnet.
  - Provides secure **SSH access** to the VM from the portal.
- **Nginx Web Server VM**
  - Ubuntu image.
  - Deployed into **Subnet-1** with **no public IP**.
  - Hosts a simple static HTML page (`app/index.html`).
- **Network Security Groups (NSGs)**
  - Applied to the VM subnet.
  - Only allow:
    - HTTP (80) from the **Firewall's private IP/subnet**.
    - SSH (22) from the **AzureBastionSubnet**.
  - Everything else inbound is denied.

### Diagram

> 📌 `docs/architecture-diagram.png`

```text
User → Azure Firewall (DNAT, IP restricted) → Nginx VM (Subnet-1)
          ↑                                      ↑
          └──────────── Azure Bastion (SSH) ─────┘
All of this lives inside a single Azure Virtual Network.
