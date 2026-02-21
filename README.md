# Azure-Virtual-Network-VM-Communication-
This project demonstrates core Azure IaaS and networking concepts by deploying two Linux virtual machines inside a single Azure Virtual Network and validating secure private communication between them.
The focus is on **shared responsibility**, **basic cloud security**, and **private networking** concepts

## Objective

- Understand how Azure Virtual Networks work
- Deploy multiple VMs in isolated subnets
- Secure VM access using SSH keys
- Fix insecure SSH key permissions
- Validate private VM-to-VM communication using internal IPs
- Understand customer responsibility in IaaS

---

## Architecture Overview

- Azure Virtual Network: `Ping-Net`
- Address Space: `10.0.0.0/16`
- Subnet 1: `VM1 (10.0.0.0/24)`
- Subnet 2: `VM2 (10.0.1.0/24)`
- Two Linux Virtual Machines (Ubuntu)
- Network Security Groups (NSGs)
- SSH key-based authentication

All communication between VMs occurs **inside Azure**, without using the public internet.

---

## Step 1: Create a Virtual Network

- Resource Group: `Projects`
- Region: `UAE North`
- Virtual Network Name: `Ping-Net`
- Address Space: `10.0.0.0/16`

### Subnets Created
- `VM1` → `10.0.0.0/24`
- `VM2` → `10.0.1.0/24`

<img width="1912" height="723" alt="Screenshot 2026-02-21 065135" src="https://github.com/user-attachments/assets/d1b6c10d-05d0-4a76-ad69-4080c05a515a" />

Security services such as Azure Firewall, Bastion, and DDoS Protection were intentionally disabled
to clearly understand customer responsibility.

---

## Step 2: Create Virtual Machines

Two Linux VMs were created with the following configuration:

- Image: Ubuntu Server
- Authentication: SSH public key
- Public IP: Enabled (for SSH access)
- Network Security Group:
  - Allow SSH (port 22)
  - Source restricted to client IP

### VM Placement
<img width="1911" height="916" alt="Screenshot 2026-02-21 065944" src="https://github.com/user-attachments/assets/6ef89b35-6203-421e-ac6a-891d6ca2e938" />
 VM-1 deployed in subnet `VM1`
<img width="1919" height="952" alt="Screenshot 2026-02-21 070434" src="https://github.com/user-attachments/assets/a763005a-bd35-4f10-b7e7-9d7b41aa7fc6" />
 VM-2 deployed in subnet `VM2`

### VM Private Key
Download Both Private keys when the VM is created
<img width="513" height="340" alt="Screenshot 2026-02-21 065640" src="https://github.com/user-attachments/assets/964f4fea-bb4e-4843-af7a-551e72249d75" />

The `.pem` file is your SSH _private key_.

---

## Step 3: Secure SSH Key Permissions (Windows)

Initial SSH connection failed due to insecure permissions on the private key.
<img width="1391" height="356" alt="Screenshot 2026-02-21 071553" src="https://github.com/user-attachments/assets/c6648abf-1c77-4382-badb-8ec3e8d2e95f" />
This was fixed by locking down the SSH private key file.

### Fix SSH Key Permissions
```bat
icacls VM-1_key.pem /inheritance:r
icacls VM-1_key.pem /grant:r "%USERNAME%:R"
````
<img width="724" height="363" alt="Screenshot 2026-02-21 071602" src="https://github.com/user-attachments/assets/dd36f924-fb47-4eb7-85b4-d265a589f178" />

This ensures:
- Only the current user can read the private key
- SSH refuses insecure keys by design

---

## Step 4: Connect to Virtual Machines

SSH access was established using key-based authentication:
<img width="1820" height="752" alt="Screenshot 2026-02-21 071236" src="https://github.com/user-attachments/assets/4dcc7ba4-d460-4970-9b46-a9d973875e6f" />

```bash
ssh -i VM-1_key.pem azureuser@<VM_PUBLIC_IP>
```
<img width="993" height="870" alt="Screenshot 2026-02-21 072148" src="https://github.com/user-attachments/assets/9756adda-f2f9-4fb7-9275-841952039352" />
VM-1

<img width="886" height="873" alt="Screenshot 2026-02-21 071840" src="https://github.com/user-attachments/assets/14f2538e-5d02-406f-a642-c964c9d4ba6a" />
VM-2

This confirms:

- OS-level access is **customer-managed**
- Azure does not manage SSH access for IaaS

---

## Step 5: Identify Private IP Addresses

Inside each VM:

```bash
ip a
```
- VM-1: `10.0.0.4`
- VM-2: `10.0.1.4`

These private IPs are used for internal communication.

---
## Step 6: Validate VM-to-VM Communication (Ping Test)

From **VM-1**:

```bash
ping 10.0.1.4 -c 4
```

From **VM-2**:

```bash
ping 10.0.0.4 -c 4
```
<img width="1919" height="470" alt="Screenshot 2026-02-21 072431" src="https://github.com/user-attachments/assets/4170905d-8d7b-462f-94e0-0baf4e498921" />

### Result

- 0% packet loss
- Low latency
- Communication succeeded over private IPs
<img width="1919" height="1000" alt="Screenshot 2026-02-21 072652" src="https://github.com/user-attachments/assets/ff27a6d8-a434-4fc1-98b1-24b76d6130aa" />

This confirms:
- Azure automatically routes traffic within a VNet
- Public IPs are not required for internal communication
- East-west traffic is allowed by default and controlled via NSGs

---

## Key Learnings

- Virtual Machines must be deployed in the same region as their Virtual Network
- Azure secures the physical infrastructure, but customers secure:
	- OS access
	- SSH keys
    - Network exposure
- SSH private keys must have strict permissions to be considered secure
- VMs in different subnets can communicate privately within a VNet
- Network Security Groups control east-west traffic in IaaS
---

## Concepts Covered

- IaaS vs PaaS responsibility
- Virtual Networks and Subnets
- Network Security Groups (NSGs)
- Private vs Public IP communication
- Shared Responsibility Model
- Basic cloud security practices
