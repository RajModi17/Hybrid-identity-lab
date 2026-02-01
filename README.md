# 🛡️ Avengers Enterprise Hybrid Identity Lab (Enterprise‑grade, Step‑by‑Step)

Author: Raj Modi

---

## 📌 Project Overview

This lab implements a complete enterprise hybrid identity environment using on‑premises Active Directory and Microsoft Entra ID. It demonstrates how real organizations integrate devices and users into a hybrid identity platform and how engineers troubleshoot common failures in production.

---

## 🎯 Objectives

- Build an on‑premises Active Directory forest
- Configure enterprise DNS and DHCP
- Synchronize identities to Microsoft Entra ID
- Enable Hybrid Azure AD Join
- Hybrid‑join a domain device
- Issue a Primary Refresh Token (PRT)
- Validate and troubleshoot the full hybrid sign‑in flow

---

## 🏗️ Environment Summary

| Component | Value |
|--------|-----|
| Domain Controller | STARK-DC01 |
| Client | CAP-WKS01 |
| AD Forest | AVENGERS.local |
| Entra tenant | 8tp8fd.onmicrosoft.com |
| Hypervisor | VMware Workstation |
| Network | VMnet8 (NAT) |
| Subnet | 192.168.40.0/24 |
| Gateway | 192.168.40.2 |

---
## Workflow Chart

<img width="1049" height="4096" alt="Workflow chart" src="https://github.com/user-attachments/assets/cda431de-6fc7-49b2-82b8-982b4e9eee6a" />





## 🏛️ Enterprise Design Decisions

- Servers use static IP addresses.
- Clients use DHCP.
- Only the domain controller is configured with external DNS forwarders.
- All clients point only to internal DNS.
- Password Hash Synchronization is used for cloud authentication.
- Hybrid Azure AD Join is used instead of pure Entra join.

---

# 🧩 PHASE 1 – VMware Network Preparation

1. Open VMware Workstation.
2. Select Edit → Virtual Network Editor.
3. Select VMnet8.
4. Verify the following:
   - NAT is enabled.
   - Subnet IP: 192.168.40.0
   - Subnet mask: 255.255.255.0
   - Gateway: 192.168.40.2
5. Apply and close.

Purpose: Provides outbound internet for all virtual machines while keeping them isolated.

---

# 🧩 PHASE 2 – Domain Controller Deployment (STARK‑DC01)

### Step 2.1 – Install Windows Server

Install Windows Server 2019 (Desktop Experience).

### Step 2.2 – Configure static networking

1. Control Panel → Network and Sharing Center → Change adapter settings.
2. Right‑click Ethernet → Properties.
3. Select Internet Protocol Version 4.
4. Configure:
   - IP: 192.168.40.10
   - Mask: 255.255.255.0
   - Gateway: 192.168.40.2
   - DNS: 192.168.40.10

### Step 2.3 – Install AD DS role

1. Server Manager → Add Roles and Features.
2. Role‑based installation.
3. Select Active Directory Domain Services.
4. Install.

### Step 2.4 – Promote to domain controller

1. Click Promote this server to a domain controller.
2. Select Add a new forest.
3. Root domain: AVENGERS.local.
4. Accept defaults.
5. Set DSRM password.
6. Complete wizard.
7. Reboot.

Purpose: Creates the on‑prem identity authority.

---

# 🧩 PHASE 3 – DNS Configuration

### Step 3.1 – Verify AD‑integrated zone

1. Run dnsmgmt.msc.
2. Expand Forward Lookup Zones.
3. Confirm AVENGERS.local exists.

### Step 3.2 – Configure DNS forwarder

1. Right‑click STARK‑DC01 → Properties.
2. Forwarders tab.
3. Add 192.168.40.2.
4. Apply.

### Step 3.3 – Restart DNS service

Purpose: Enables internet name resolution for all domain members.

---

# 🧩 PHASE 4 – DHCP Server Installation

### Step 4.1 – Install role

Server Manager → Add Roles and Features → DHCP Server.

### Step 4.2 – Authorize DHCP

Complete post‑deployment configuration and authorize in AD.

### Step 4.3 – Create scope

1. DHCP console → IPv4 → New Scope.
2. Range: 192.168.40.50 – 192.168.40.200.
3. Exclude server IPs.

### Step 4.4 – Scope options

- 003 Router: 192.168.40.2
- 006 DNS: 192.168.40.10
- 015 DNS domain: avengers.local

Purpose: Provides centralized enterprise IP addressing.

---

# 🧩 PHASE 5 – Client Deployment (CAP‑WKS01)

### Step 5.1 – Install Windows 10

### Step 5.2 – Attach VM to VMnet8

### Step 5.3 – Verify network

ipconfig /all

### Step 5.4 – Join domain

1. System → About → Advanced system settings.
2. Computer Name → Change.
3. Domain: AVENGERS.local.
4. Reboot.

---

# 🧩 PHASE 6 – Microsoft Entra Connect Installation

### Step 6.1 – Obtain installer from Entra portal

### Step 6.2 – Run installer on STARK‑DC01

### Step 6.3 – Select Custom installation

### Step 6.4 – Choose sign‑in method

Password Hash Synchronization.

### Step 6.5 – Provide credentials

- Global Admin (Entra)
- Enterprise Admin (on‑prem)

### Step 6.6 – Complete installation

Purpose: Synchronizes identities to the cloud tenant.

---

# 🧩 PHASE 7 – Enable Hybrid Azure AD Join

1. Launch Microsoft Entra Connect.
2. Click Configure.
3. Select Configure device options.
4. Select Configure Hybrid Azure AD Join.
5. Select Windows 10 or later.
6. Select AVENGERS.local forest.
7. Complete wizard.

Purpose: Enables hybrid registration workflow.

---

# 🧩 PHASE 8 – Verify Service Connection Point (SCP)

1. Run adsiedit.msc.
2. Connect to Configuration naming context.
3. Navigate:
   CN=Configuration → CN=Services → CN=Device Registration Configuration.

Purpose: Confirms Entra Connect stamped AD with registration endpoints.

---

# 🧩 PHASE 9 – UPN Alignment

### Step 9.1 – Add tenant UPN suffix

1. Run domain.msc.
2. Right‑click Active Directory Domains and Trusts → Properties.
3. Add 8tp8fd.onmicrosoft.com.

### Step 9.2 – Modify user UPN

1. Run dsa.msc.
2. Open steve.rogers.
3. Change UPN to steve.rogers@8tp8fd.onmicrosoft.com.

Purpose: Enables identity matching for PRT issuance.

---

# 🧩 PHASE 10 – Synchronization

On STARK‑DC01:

Start-ADSyncSyncCycle -PolicyType Delta

Confirm successful sync.

---

# 🧩 PHASE 11 – Hybrid Join on Client

On CAP‑WKS01 (admin):

1. dsregcmd /leave
2. Reboot.
3. Sign in as AVENGERS\\steve.rogers.
4. Wait two minutes.
5. dsregcmd /join.

---

# 🧩 PHASE 12 – DNS & Internet Failure Troubleshooting

### Symptom

Client connected but no internet.

### Verification

nslookup google.com timed out.

### Root cause

DNS forwarder missing on STARK‑DC01.

### Fix

Configure forwarder and restart DNS.

---

# 🧩 PHASE 13 – PRT Troubleshooting

### Symptom

AzureAdPrt : NO

### Root cause

UPN mismatch and user signed in before alignment.

### Fix

1. Confirm UPN.
2. Force sync.
3. Sign out.
4. Sign in.
5. dsregcmd /refreshprt.

---

# 🧩 PHASE 14 – Final Validation

On CAP‑WKS01:

dsregcmd /status

Expected:

DomainJoined : YES
AzureAdJoined : YES
AzureAdPrt : YES
EnterpriseJoined : NO

---

# 🚧 Real Challenges Encountered

- SCP not created
- Hybrid join not enabled in Entra Connect
- DNS forwarder missing
- No internet access
- UPN mismatch
- PRT not issued
- Developer tenant domain limitation

---

# 🔁 Logical Flow

Build DC → DNS → DHCP → Client → Join domain → Entra Connect → Hybrid Join → SCP → UPN alignment → Sync → Join device → Issue PRT → Validate

---

# 🧠 Skills Demonstrated

Hybrid identity architecture, DNS troubleshooting, Entra Connect operations, authentication flow analysis, enterprise device registration and security identity foundations.

