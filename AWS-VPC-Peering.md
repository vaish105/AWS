{"variant":"standard","title":"Creating and Connecting Two VPCs Using VPC Peering (AWS Management Console Lab)","id":"58327"}
# Creating and Connecting Two VPCs Using VPC Peering  
*(AWS Management Console - Step-by-step Lab)*  

This detailed guide walks students through every click in the AWS Console to create two VPCs, configure public and private subnets, route tables, internet/NAT gateways, launch EC2 instances, set up VPC Peering, update routes, modify security groups, and test connectivity.

---

## ✅ Quick Checklist

| Resource | Configuration |
|-----------|----------------|
| **VPC-A CIDR** | 10.0.0.0/16 |
| **Public Subnet A** | 10.0.1.0/24 |
| **Private Subnet A** | 10.0.2.0/24 |
| **VPC-B CIDR** | 10.1.0.0/16 |
| **Public Subnet B** | 10.1.1.0/24 |
| **Private Subnet B** | 10.1.2.0/24 |
| **Internet Gateways** | IGW-A, IGW-B |
| **Route Tables** | RT-Public-A, RT-Private-A, RT-Public-B, RT-Private-B |
| **NAT Gateway** | Optional (for private subnets) |
| **VPC Peering** | Between VPC-A and VPC-B |
| **EC2 Instances** | In public subnets for testing |

---

## Step 1 — Create VPC A (10.0.0.0/16)

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com).
2. From **Services**, select **VPC** to open the VPC Dashboard.
3. In the left menu, click **Your VPCs**.
4. Click **Create VPC**.
5. Choose **VPC only**.
6. Fill in:
   - **Name tag:** `VPC-A`
   - **IPv4 CIDR block:** `10.0.0.0/16`
   - **IPv6 CIDR block:** Unchecked
   - **Tenancy:** Default
7. Click **Create VPC**.
8. Note the VPC ID (e.g., `vpc-0a1b2c3d`).

> 💡 **Tip:** Keep the VPC ID handy for subnet and route table setup.

---

## Step 2 — Create Route Tables for VPC-A

1. In the left menu, select **Route Tables** → **Create route table**.
2. Create:
   - **RT-Public-A** (VPC: `VPC-A`)
   - **RT-Private-A** (VPC: `VPC-A`)
3. RT-Public-A will later route to the Internet Gateway.
4. RT-Private-A will be for private subnets (no IGW route).

---

## Step 3 — Create Subnets for VPC-A

1. Go to **Subnets** → **Create subnet**.
2. Choose **VPC-A** and any AZ (e.g., `us-east-1a`).
3. Create:
   - **Public-Subnet-A:** `10.0.1.0/24`
   - **Private-Subnet-A:** `10.0.2.0/24`
4. Verify both subnets are associated with **VPC-A**.

> 📝 Note: Subnets do not have route table associations by default.

---

## Step 4 — Associate Route Tables with Subnets (VPC-A)

1. Go to **Route Tables** → select **RT-Public-A**.
2. **Edit subnet associations** → check **Public-Subnet-A** → Save.
3. Select **RT-Private-A** → associate **Private-Subnet-A** → Save.

---

## Step 5 — Create and Attach Internet Gateway (VPC-A)

1. Go to **Internet Gateways** → **Create internet gateway**.
2. Name: `IGW-A` → **Create internet gateway**.
3. Select **IGW-A** → **Actions → Attach to VPC → VPC-A**.
4. Edit **RT-Public-A** routes:
   - Add route `0.0.0.0/0` → **Target:** IGW-A → Save routes.

---

## Step 6 — Create VPC B (10.1.0.0/16) and Components

1. Create VPC:  
   - **Name:** `VPC-B`
   - **CIDR:** `10.1.0.0/16`
2. Create route tables: `RT-Public-B` and `RT-Private-B`.
3. Create subnets:
   - **Public-Subnet-B:** `10.1.1.0/24`
   - **Private-Subnet-B:** `10.1.2.0/24`
4. Create and attach Internet Gateway:
   - **IGW-B → Attach to VPC-B**
5. Edit **RT-Public-B** routes:
   - `0.0.0.0/0` → IGW-B
6. Associate subnets:
   - **Public-Subnet-B → RT-Public-B**
   - **Private-Subnet-B → RT-Private-B**

> ⚠️ Ensure CIDR blocks do **not overlap**.

---

## Step 7 — (Optional) Create NAT Gateway for Private Subnet Internet Access

1. Go to **NAT Gateways** → **Create NAT gateway**.
2. Subnet: `Public-Subnet-A`
3. Allocate a new **Elastic IP**.
4. Create and wait until status = *Available*.
5. Edit **RT-Private-A**:
   - Add route `0.0.0.0/0` → **Target:** NAT Gateway (nat-xxxx)
6. Repeat for VPC-B if needed.

---

## Step 8 — Launch EC2 Instances for Testing

1. Go to **EC2** → **Launch instances**.
2. **AMI:** Amazon Linux 2 AMI (free tier)
3. **Instance Type:** t2.micro or t3.micro
4. **Network:** `VPC-A`, **Subnet:** `Public-Subnet-A`
5. **Auto-assign Public IP:** Enable
6. **Tag:** Name = `EC2-VPC-A`
7. **Security Group:** Create `SG-EC2-Public-A` with:
   - SSH (TCP 22): YOUR_IP/32
   - HTTP (TCP 80): 0.0.0.0/0 (optional)
   - ICMP (All): 10.1.0.0/16 (for VPC-B ping)
8. **Launch instance** (use or create Key Pair).
9. Repeat for **VPC-B**:
   - Name: `EC2-VPC-B`
   - ICMP inbound from `10.0.0.0/16`

> 🔒 Restrict SSH access to your IP only.

---

## Step 9 — Create VPC Peering Connection (VPC-A ↔ VPC-B)

1. In **VPC** console → **Peering Connections** → **Create Peering Connection**.
2. **Name:** `Peering-A-B`
3. **Requester VPC:** VPC-A  
   **Accepter VPC:** VPC-B
4. Click **Create Peering Connection**.
5. If status = *Pending Acceptance*, select and **Accept Request**.
6. Once accepted, status = *Active*.

---

## Step 10 — Update Route Tables for Peering

1. Select **RT-Public-A** → **Edit routes**.
   - Add: `Destination = 10.1.0.0/16` → **Target:** Peering-A-B
2. Select **RT-Public-B** → **Edit routes**.
   - Add: `Destination = 10.0.0.0/16` → **Target:** Peering-A-B
3. For private-only communication, add to **RT-Private-A** and **RT-Private-B** instead.

---

## Step 11 — Security Group Rules for Cross-VPC Communication

1. Go to **EC2 → Security Groups**.
2. Edit **SG-EC2-Public-A** inbound rules:
   - ICMP (All) from `10.1.0.0/16`
3. Edit **SG-EC2-Public-B** inbound rules:
   - ICMP (All) from `10.0.0.0/16`
4. Optionally use **security group references** (use sg-ids) for tighter control.

---

## Step 12 — Test Connectivity Between EC2 Instances

1. SSH into **EC2-VPC-A**:
   ```bash
   ssh -i path/to/key.pem ec2-user@<Public-IP-of-EC2-VPC-A>
   ```
2. Ping **EC2-VPC-B** private IP:
   ```bash
   ping <Private-IP-of-EC2-VPC-B>
   ```
3. If ping fails, check:
   - Peering status = Active  
   - Route tables have correct routes  
   - SG rules allow ICMP  
   - NACLs permit traffic
4. Optional test:
   ```bash
   curl http://<Private-IP-of-EC2-VPC-B>
   ```

---

## 🧭 Troubleshooting Checklist

- ❌ **Peering not Active** → Accept the request.  
- 🚫 **Missing Routes** → Add to route tables.  
- 🔒 **SG rules** → Allow ICMP or required ports.  
- 🧱 **NACLs** → Ensure inbound/outbound allow traffic.  
- ⚠️ **Overlapping CIDRs** → Peering won’t work.

---

## 💡 Final Tips

- **Peering is non-transitive** — A↔B and B↔C ≠ A↔C. Use **Transit Gateway** for multi-VPC setups.  
- **Enable DNS resolution** across peers:
  - Select Peering → Actions → Edit DNS settings → Allow remote DNS resolution (both sides).
- Use **Security Group references** instead of CIDRs when possible.  
- **Clean up** resources (NAT, EC2, IGW, Peering) after lab to avoid charges.

---

**Good luck with your lab! 🎓**


