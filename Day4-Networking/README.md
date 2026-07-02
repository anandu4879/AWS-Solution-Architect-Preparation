# Day 4  — AWS Networking (VPC & Subnets) - SAA Week 1 Day 4

Today I learned **AWS Networking** — designing network architecture, isolating resources, and controlling traffic flow.

Networking is critical for AWS security and architecture (20%+ of SAA exam).

---

## VPC (Virtual Private Cloud)

```
VPC = Your private network in AWS

Think of it as:
Your home network (WiFi router creates network)
AWS creates network for you
You control everything inside

Default VPC:
- Automatically created per region
- 172.31.0.0/16 CIDR block

Custom VPC:
- You create it
- You choose CIDR block (usually 10.0.0.0/16)
- More control and security
```

### Why VPC Matters

```
Without VPC (resources exposed):
┌─────────────────┐
│ EC2 instance    │ ← Direct internet access (EXPOSED!)
│ RDS database    │ ← Anyone can try to connect
│ S3 bucket       │ ← Public by default
└─────────────────┘

With VPC (resources isolated):
┌──────────────────────────┐
│ VPC (Private Network)     │
│ ┌────────────────────┐   │
│ │ EC2 (private)      │   │
│ │ RDS (private)      │   │
│ │ S3 (within VPC)    │   │
│ └────────────────────┘   │
│ Internet Gateway          │
└──────────────────────────┘
     ↓ Controlled
 Internet Access
```

---

## Subnets

```
Subnet = subdivision of VPC

VPC: 10.0.0.0/16 (65,536 IPs)
├── Subnet 1: 10.0.1.0/24 (256 IPs)
├── Subnet 2: 10.0.2.0/24 (256 IPs)
├── Subnet 3: 10.0.10.0/24 (256 IPs)
└── Subnet 4: 10.0.11.0/24 (256 IPs)

Each subnet in ONE Availability Zone
```

### Public vs Private Subnets

```
Public Subnet
- Has route to Internet Gateway (0.0.0.0/0 → IGW)
- Instances get public IP
- Can reach internet
- Internet can reach them (if SG allows)
- Use for: ALB, bastion hosts

Private Subnet
- No route to Internet Gateway
- Instances don't get public IP
- Can't reach internet directly
- Internet can't reach them
- Use for: App servers, databases
```

### CIDR Notation (SAA Critical)

```
CIDR = Classless Inter-Domain Routing

10.0.0.0/16
├── 10.0.0.0 = network address
└── /16 = number of fixed bits

/8 = 16 million IPs (huge)
/16 = 65,536 IPs (large, for VPC)
/24 = 256 IPs (small, for subnet)
/32 = 1 IP (single host)

How it works:
10.0.1.0/24 means:
├── First 24 bits are fixed: 10.0.1
└── Last 8 bits are variable: .0 to .255

Usable IPs: 10.0.1.1 to 10.0.1.254
(AWS reserves .0 and .255)
```

---

## Internet Gateway & NAT Gateway

### Internet Gateway (IGW)

```
Internet Gateway = Connection to internet

Enables:
- Instances in public subnet reach internet
- Internet reaches public instances

For Public Subnet:
1. Attach IGW to VPC
2. Route table: 0.0.0.0/0 → IGW
3. Instance has public IP
4. Instance can reach internet

Flow:
EC2 (10.0.1.5) → IGW → Internet → Response → IGW → EC2
```

### NAT Gateway

```
NAT = Network Address Translation

Problem:
Private instances need to download packages, call APIs
But they shouldn't be accessible from internet

Solution: NAT Gateway
├── Placed in PUBLIC subnet
├── Has public (elastic) IP
├── Private instances route to NAT
├── NAT translates: 10.0.10.5 → EIP
├── Internet thinks traffic from EIP
└── Response comes back to NAT → forwarded to instance

Flow:
EC2 (10.0.10.5) → NAT → Internet → Response → NAT → EC2

Benefits:
✅ Private instances can reach internet
✅ Internet can't initiate connection to them
✅ More secure than public subnet
```

---

## Route Tables

```
Route Table = How traffic flows

Route: 0.0.0.0/0 (all traffic) → destination

Public Route Table:
├── 10.0.0.0/16 → local (stay in VPC)
└── 0.0.0.0/0 → igw-123 (go to internet)

Private Route Table:
├── 10.0.0.0/16 → local (stay in VPC)
└── 0.0.0.0/0 → nat-456 (go via NAT)

Database Route Table:
└── 10.0.0.0/16 → local (stay in VPC)

Each subnet must be associated with route table
```

---

## Security Groups (Instance-Level Firewall)

```
Security Group = Stateful firewall at instance level

Inbound Rules:
├── Protocol: TCP, UDP, ICMP
├── Port: 22 (SSH), 80 (HTTP), 443 (HTTPS), 5432 (PostgreSQL)
├── Source: IP, CIDR, or security group
└── Action: Allow or Deny

Outbound Rules:
├── Default: Allow all
└── Can restrict if needed

Stateful:
- If inbound allowed, response automatically allowed
- Don't need to add reverse rule

Example Rules:
✅ Allow TCP 80 from 0.0.0.0/0 (HTTP)
✅ Allow TCP 443 from 0.0.0.0/0 (HTTPS)
✅ Allow TCP 22 from my-office-ip (SSH)
✅ Allow TCP 5000 from app-sg (from app servers)
❌ Deny all other inbound (implicit)
✅ Allow all outbound (default)
```

---

## Architecture Layers

### Production VPC Design

```
VPC (10.0.0.0/16)

┌─ Public Layer ─────────────────┐
│ us-east-1a:      us-east-1b:   │
│ 10.0.1.0/24      10.0.2.0/24   │
│ ┌──────────┐     ┌──────────┐  │
│ │ ALB      │     │ NAT GW   │  │
│ │(IGW)     │     │          │  │
│ └──────────┘     └──────────┘  │
└────────────────────────────────┘
          ↓ SG: Allow 80, 443

┌─ Application Layer ────────────┐
│ us-east-1a:      us-east-1b:   │
│ 10.0.10.0/24     10.0.11.0/24  │
│ ┌──────────┐     ┌──────────┐  │
│ │ App 1    │     │ App 2    │  │
│ │          │     │          │  │
│ └──────────┘     └──────────┘  │
└────────────────────────────────┘
          ↓ SG: Allow 5000 from ALB

┌─ Database Layer ───────────────┐
│ us-east-1a:      us-east-1b:   │
│ 10.0.20.0/24     10.0.21.0/24  │
│ ┌──────────┐     ┌──────────┐  │
│ │ Primary  │     │ Standby  │  │
│ │ DB       │     │ DB       │  │
│ └──────────┘     └──────────┘  │
└────────────────────────────────┘
   SG: Allow 5432 from App SG
```

---

## Availability Zones (High Availability)

```
Availability Zone = Separate data center

Region: us-east-1
├── Availability Zone us-east-1a
├── Availability Zone us-east-1b
└── Availability Zone us-east-1c

Each AZ is:
- Geographically separate (miles apart)
- Connected by low-latency network
- Independent power, cooling, networking
- If one AZ fails, others still work

Best Practice:
- Spread subnets across AZs
- If one AZ goes down, other AZ handles traffic
- Zero downtime (high availability)
```

---

## Network Flow Example

### Request to Web Application

```
1. User sends request
   User → ALB public IP

2. Request hits ALB (public subnet, us-east-1a)
   ├─ Enters via Internet Gateway
   ├─ ALB SG checks: Allow TCP 80 from 0.0.0.0/0 ✅
   └─ Request matches

3. ALB forwards to app server (private subnet)
   ├─ Route table: 10.0.10.0/24 → local ✅
   ├─ App SG checks: Allow TCP 5000 from ALB SG ✅
   └─ Request matched

4. App server connects to RDS (database subnet)
   ├─ Route table: 10.0.20.0/24 → local ✅
   ├─ DB SG checks: Allow TCP 5432 from App SG ✅
   └─ Connection established

5. Response flows back
   RDS → App → ALB → IGW → Internet → User
```

---

## SAA Exam Topics Covered

✅ **VPC Fundamentals**
- VPC and CIDR blocks
- Subnets and Availability Zones
- Public vs private subnets
- CIDR calculation and planning

✅ **Connectivity**
- Internet Gateway (IGW)
- NAT Gateway
- Route tables
- Routing decisions

✅ **Security**
- Security groups (stateful firewall)
- Network ACLs (subnet firewall)
- Principle of least privilege
- Security group chaining

✅ **High Availability**
- Multiple AZs
- Multi-tier architecture
- Failover and redundancy

✅ **Advanced Topics** (covered later)
- VPC peering
- VPN
- AWS PrivateLink
- Transit Gateway

---

## Challenges Completed

### Challenge 1 — Design VPC Architecture
Designed complete architecture on paper with all layers.

### Challenge 2 — Create VPC with Terraform
Built VPC, subnets, IGW, NAT, route tables using Terraform.

### Challenge 3 — Security Groups with Terraform
Created security groups with proper rules and chaining.

### Challenge 4 (Implied) — Complete VPC
Built production-ready architecture with all components.

---

## Terraform Implementation

### Key Resources Used

```hcl
- aws_vpc: VPC creation
- aws_subnet: Subnets (across AZs)
- aws_internet_gateway: IGW
- aws_nat_gateway: NAT
- aws_route_table: Route tables
- aws_route_table_association: Connect subnets
- aws_security_group: Firewalls
- aws_security_group_rule: Flexible rules
```

### Architecture as Code

VPC architecture is fully defined in Terraform:
- Reproducible (same result every time)
- Versioned (track changes in Git)
- Automated (no manual clicking)
- Scalable (easy to expand)

---

## Key Takeaways

1. **VPC**: Isolated network, you control everything
2. **Subnets**: Divide VPC, each in one AZ
3. **Public**: Route to IGW (internet access)
4. **Private**: Route to NAT (no inbound from internet)
5. **IGW**: Direct internet connection
6. **NAT**: Secure outbound-only internet
7. **Route Tables**: Control traffic flow
8. **SG**: Instance-level firewall (stateful)
9. **CIDR**: IP address allocation (/16 VPC, /24 subnet)
10. **AZs**: Spread across for high availability

---

## Production Checklist

```bash
☐ VPC with appropriate CIDR (/16)
☐ Subnets across multiple AZs (/24)
☐ Public subnets with IGW route
☐ Private subnets with NAT route
☐ Database subnets (isolated)
☐ Security groups with least privilege
☐ Group rules reference other SGs
☐ Route tables for each subnet type
☐ Flow logs for monitoring (optional)
☐ VPC Flow Logs enabled (optional)
☐ Documented and version controlled
```

---

## Statistics

```
Day 32:
- VPC components: 8+ (VPC, subnets, IGW, NAT, route tables, SGs, NACLs, AZs)
- SAA topics: 10+ (CIDR, subnet design, routing, security, HA)
- Terraform resources: 15+
- Production setup: Complete
```

---

## Interview Talking Points

"I can design and implement production VPC architecture with public and private subnets spread across multiple Availability Zones for high availability. I understand CIDR notation, routing, security groups, and the difference between Internet Gateway and NAT Gateway. I can implement security best practices with layered security groups and proper isolation."

---

## Next Topics (Day 4)

```
Day 33: Identity & Access Management (IAM)
├── Users and roles
├── Policies and permissions
├── Assume roles
└── Resource-based policies
```

---

## Conclusion

VPC is the **foundation of AWS security and architecture**:
- **Isolation**: Private network separate from internet
- **Control**: You define every connection
- **Security**: Layered firewalls (SG, NACL)
- **HA**: Spread across Availability Zones

This day covers ~20% of SAA exam.

Together with compute (EC2), storage (S3, EBS, EFS), databases (RDS, DynamoDB), and networking (VPC), you've now covered the **core AWS services**.

Next: Identity & Access (IAM) — who can do what. 🚀

---

## Resource Limits

Single VPC:
- 5 VPCs per region (can increase)
- 200 subnets per VPC
- 200 route tables per VPC
- 5 Internet Gateways per region (one per VPC)
- 5 Elastic IPs for NAT (can increase)

Security Groups:
- 500 per VPC (can increase)
- 60 rules per security group

These are rarely hit in practice.