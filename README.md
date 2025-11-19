# 🔐 Azure Firewall Secured Nginx Application

This project demonstrates how to deploy a **private Nginx web application** securely on Microsoft Azure using:

- **Azure Firewall** (DNAT for controlled ingress)
- **Azure Bastion** (SSH access without public IPs)
- **Network Security Groups (NSGs)** (subnet-level isolation)
- **Subnet segmentation** inside an Azure VNet

The virtual machine hosting the Nginx site has **no public IP**.  
All application traffic flows strictly through Azure Firewall, and SSH administration goes via Bastion.

This project follows **Zero Trust** and **Least Privilege** design principles.

---

## 📌 Architecture Diagram

<img width="1536" height="1024" alt="ChatGPT Image Nov 18, 2025, 10_42_47 PM" src="https://github.com/user-attachments/assets/30eed410-d345-4ec2-b65f-f0de18b8d786" />


---

## 🎯 Project Goals

This project demonstrates real-world cloud security concepts:

### ✅ Network Segmentation  
- Three subnets: *Firewall, Bastion, Application (Subnet-1)*

### ✅ Perimeter Security  
- Azure Firewall as the *only* public entry point  
- VM has **no public IP**

### ✅ Secure Remote Access  
- SSH access only through Azure Bastion  
- No public SSH exposure

### ✅ Application Behind Firewall  
- Web traffic flows only through Firewall DNAT

### ✅ Least Privilege  
- DNAT rule allows only **my laptop's public IP**  
- NSG rules allow only:
  - Port **80** from Firewall subnet  
  - Port **22** from Bastion subnet  

---

## 🛠 Step-by-Step Deployment Process

### **1️⃣ Create Resource Group**
Created a dedicated RG for networking and compute.

---

### **2️⃣ Create Virtual Network + Subnets**

VNet: `vnet-hub`

Subnets:

| Subnet              | Purpose                    |
|---------------------|----------------------------|
| `AzureFirewallSubnet` | Hosts Azure Firewall       |
| `AzureBastionSubnet`  | Hosts Bastion Service      |
| `subnet-1`            | Nginx Application VM       |

---

### **3️⃣ Deploy Azure Firewall + Firewall Policy**

Firewall deployed in `AzureFirewallSubnet` with a public IP.

**DNAT Rule Configured:**

- **Source:** My laptop public IP  
- **Destination IP:** Firewall Public IP  
- **Destination Port:** `4000`  
- **Translate To:** VM private IP on port `80`  
- **Protocol:** TCP  


#### 🔥 DNAT Rule Screenshot
<img width="1915" height="992" alt="Screenshot 2025-11-19 002138" src="https://github.com/user-attachments/assets/f0eb7e1a-ce4f-41f6-ba3a-f1007b388e3c" />


---

### **4️⃣ Deploy Azure Bastion**

- Bastion deployed in its own subnet  
- SSH access to the VM through Azure Portal  
- No SSH exposed to the internet  

---

### **5️⃣ Create Ubuntu VM + Install Nginx**

The VM was deployed:

- In `subnet-1`
- Without a public IP
- SSH access via Bastion

<img width="2553" height="1270" alt="Screenshot 2025-11-19 105933" src="https://github.com/user-attachments/assets/eddecdc3-2249-40f9-9c02-286374e5b460" />

### Connect via Bastion → SSH → Install Nginx


<img width="1913" height="982" alt="Screenshot 2025-11-19 001924" src="https://github.com/user-attachments/assets/feaba85a-0982-4a87-a8db-ca94354c81bf" />

Nginx installation:

```bash
sudo su -
sudo apt update
sudo apt install -y nginx
sudo vim /var/www/html/index.html
```
Inside Vim:
- Press i for insert mode**
- Paste your HTML (you can use the same one I used in the index.html)**
- Press Esc**
- Type :wq → press Enter to save**

This updates the Nginx landing page.

Test Nginx Locally From the VM

```bash
curl http://localhost:80
```

If you see your HTML, Nginx is working correctly.

### **6️⃣ Final Test — Web App Served Through Azure Firewall**

```bash
http://<FirewallPublicIP>:4000
```

This goes through DNAT → VM private IP → port 80.

<img width="1917" height="903" alt="Screenshot 2025-11-19 002832" src="https://github.com/user-attachments/assets/b1482a9c-e56b-47e0-b1fa-f7c8aa0d62bd" />

This confirms:

- Firewall is working
- DNAT is configured correctly
- NSGs are allowing only valid traffic
- VM remains private
- No public IP exposure
