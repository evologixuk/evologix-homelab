# 🧱 EvoLogix Homelab Infrastructure

> Real-world Active Directory + Infrastructure Lab built across two virtual "sites" (Guildford + Mahón) for enterprise-grade simulation, Veeam testing, firewalls, and hybrid networking scenarios.

---

## 🌍 Domain Overview

- Internal AD Domain: `corp.evologix.co.uk`
- DNS: AD-integrated across multiple DCs
- Subnets isolated via VLANs, routed by Ubiquiti
- Site separation designed for DR/migration/firewall scenarios

---

## 🌐 Network Layout

| Site     | VLAN ID | Subnet           | Description                |
|----------|---------|------------------|----------------------------|
| Default  | 1       | 192.168.1.0/24    | Home LAN                   |
| GU (A)   | 20      | 10.245.218.0/24   | Guildford - Site A         |
| MA (B)   | 30      | 10.245.219.0/24   | Mahón (Menorca) - Site B   |
| DMZ      | 50      | 10.245.50.0/24    | Public-facing services     |

---

## 🖥️ Host Inventory

### 🔹 Site A: Guildford (`GU`)

| Hostname     | IP Address      | Role                             | vCPU | RAM | Proxmox Host     |
|--------------|-----------------|----------------------------------|------|-----|------------------|
| GU-PMX-01    | 10.245.218.10   | Proxmox Hypervisor               | 8    | 64  | Physical Node    |
| GU-AD-01     | 10.245.218.20   | Primary DC, DNS, DHCP, FSMO All  | 2    | 4   | GU-PMX-01        |
| GU-AD-02     | 10.245.218.21   | Secondary DC, DNS, Failover      | 2    | 4   | GU-PMX-01        |

> ✅ Fully operational and replicating AD/DC.

---

### 🔸 Site B: Mahón (`MA`) *(Planned)*

| Hostname     | IP Address      | Role                             | vCPU | RAM | Proxmox Host     |
|--------------|-----------------|----------------------------------|------|-----|------------------|
| MA-PMX-01    | 10.245.219.10   | Proxmox Hypervisor               | 8    | 64  | Physical Node    |
| MA-AD-01     | 10.245.219.20   | Primary DC, DNS, DHCP            | 2    | 4   | MA-PMX-01        |
| MA-AD-02     | 10.245.219.21   | Secondary DC, DNS                | 2    | 4   | MA-PMX-01        |

> 🛠️ **Next:** Build domain controllers and replicate into existing `corp.evologix.co.uk` forest.

---

## ⚙️ Domain Controller Promotion Script (PowerShell)

```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

$domainCred = Get-Credential -Message "Enter Domain Admin credentials"
$dsrmPassword = Read-Host -AsSecureString "Enter DSRM password"

Install-ADDSDomainController `
    -DomainName "corp.evologix.co.uk" `
    -Credential $domainCred `
    -InstallDns `
    -SiteName "Default-First-Site-Name" `
    -SafeModeAdministratorPassword $dsrmPassword `
    -Force:$true
