# Day 2 — AWS Storage Services (S3, EBS, EFS) - SAA Week 1 Day 2

Today I learned **AWS Storage Services** — how to store files, databases, and data in the cloud.

Three different services for three different use cases.

---

## Three Storage Services

### S3 (Simple Storage Service)
```
Object storage in the cloud

Think of it as:
A giant file storage locker

Characteristics:
- Store any file (images, videos, backups)
- Access from anywhere
- Versioning and lifecycle
- Cheap (and cheaper with archival)

Use for:
✅ Website static content
✅ Backups and archives
✅ Data lake
✅ Log files
```

### EBS (Elastic Block Store)
```
Hard drive for EC2

Think of it as:
A hard drive attached to your computer

Characteristics:
- Fast storage
- Only one instance can use it
- Must attach to EC2
- Survives stop/start

Use for:
✅ EC2 root volume (OS)
✅ Databases (need speed)
✅ Application data
```

### EFS (Elastic File System)
```
Network file system

Think of it as:
A shared Google Drive for EC2

Characteristics:
- Multiple instances access simultaneously
- Automatically scales
- Network-based (slower than EBS)
- Shares data across instances

Use for:
✅ Shared data
✅ Big data processing
✅ Machine learning datasets
```

---

## S3 Deep Dive

### How S3 Works

```
1. Create bucket (container)
   └── my-photos

2. Upload objects (files)
   ├── photo1.jpg
   ├── photo2.jpg
   └── video.mp4

3. Access via URL
   └── https://my-photos.s3.amazonaws.com/photo1.jpg
```

### Storage Classes (Cost Optimization)

```
S3 Standard
├── Default
├── Fast access
└── Most expensive

S3 Intelligent-Tiering
├── Auto moves based on access patterns
└── Balances cost and performance

S3 Glacier
├── Archive storage
├── Slow to retrieve (hours)
└── Very cheap

S3 Glacier Deep Archive
├── Long-term archival
├── Slowest retrieval (12+ hours)
└── Cheapest
```

### Lifecycle Policies (Save Money)

```
Day 0: Upload to S3 Standard (fast, expensive)
  ↓
Day 30: Auto move to S3 Glacier (slow, cheap)
  ↓
Day 90: Auto delete (save space)

Example:
- Website static files: Keep in Standard
- Monthly backups: Transition to Glacier after 30 days
- Compliance archives: Keep forever in Deep Archive
```

### Versioning (Disaster Recovery)

```
Without versioning:
Upload file.txt (version 1)
Upload file.txt (version 2) → version 1 is lost!

With versioning:
Upload file.txt (version 1) → stored
Upload file.txt (version 2) → stored
Upload file.txt (version 3) → stored

Can restore any version!
```

### Encryption

```
SSE-S3: AWS manages encryption keys
SSE-KMS: You manage keys (more control)
SSE-C: You provide keys (highest control)

All encrypt data at rest and in transit
```

---

## EBS Deep Dive

### How EBS Works

```
1. Create EBS volume (1-16 TB)
   ↓
2. Attach to EC2 instance
   ↓
3. Mount in instance
   └── ec2-user: mount /dev/sdf /data

4. Use like normal hard drive
   ├── Read/write files
   ├── Install databases
   └── Run applications

5. Create snapshot (backup)
   └── Can restore to new instance
```

### Volume Types (Performance vs Cost)

```
gp2/gp3 (General Purpose)
├── Default choice
├── SSD (fast)
├── Medium cost
└── 99.9% uptime

io1/io2 (High Performance)
├── Database performance
├── SSD (very fast)
├── High cost
└── 99.95% uptime

st1/sc1 (Throughput)
├── Big data, data warehouse
├── HDD (slower)
├── Cheap
└── 99.9% uptime
```

### Snapshots (Backups)

```
Process:
1. Create EBS volume with important data
2. Create snapshot (copies data to S3)
3. Can delete original volume (save cost)
4. Later: Create volume from snapshot
5. Restore to new instance

Use cases:
✅ Backups (point-in-time copies)
✅ Disaster recovery
✅ Moving volumes between regions
✅ Creating volume copies
```

### Encryption

```
Without encryption:
Data is readable if someone gets disk

With encryption:
Data encrypted at rest
Encrypted in transit
Automatically decrypted when accessed

You can encrypt existing volume:
1. Create snapshot
2. Create new volume from snapshot with encryption enabled
3. Attach to instance
4. Old volume can be deleted
```

---

## EFS Deep Dive

### How EFS Works

```
Create EFS file system
        ↓
Create mount targets in subnets
        ↓
EC2 instances mount EFS
        ├── Instance 1: mount /efs
        ├── Instance 2: mount /efs
        └── Instance 3: mount /efs
        ↓
All instances read/write same data!
        ↓
Automatically grows as data added
```

### When to Use EFS

```
✅ Multiple instances need same data
✅ Big data processing (Hadoop, Spark)
✅ Machine learning (training data)
✅ Media processing
✅ Database backups shared

❌ Single instance (use EBS instead)
❌ Very high performance (use EBS instead)
❌ Different regions (use S3 instead)
```

### Performance

```
EBS:
- Microseconds latency
- Single instance
- Limited to instance bandwidth

EFS:
- Milliseconds latency
- Multiple instances
- NFS protocol overhead

S3:
- Milliseconds to seconds latency
- Any service
- Via HTTP/HTTPS
```

---

## S3 vs EBS vs EFS Decision Tree

```
Need file storage for archive/backup?
└─ S3 ✓

Need automatic archival after 30 days?
└─ S3 + Lifecycle ✓

Need to recover old versions?
└─ S3 + Versioning ✓

Need fast storage for single instance?
└─ EBS ✓

Need database storage?
└─ EBS ✓

Need disk snapshots for backup?
└─ EBS Snapshots ✓

Need shared storage for multiple instances?
└─ EFS ✓

Need high availability database?
└─ EBS + RDS (managed) ✓

Need cheapest backup storage?
└─ S3 + Glacier lifecycle ✓

Need all instances to access same data?
└─ EFS ✓
```

---

## Real-World Scenario

### E-commerce Company

```
Website static content (images, CSS, JS):
└─ Store in S3
└─ Serve globally via CloudFront
└─ Keep in S3 Standard (fast)

Product database:
└─ EBS volume on RDS instance
└─ Fast, reliable storage

Customer logs:
└─ Log to S3
└─ Use S3 Lifecycle: 30 days → Glacier, 90 days → delete
└─ Saves money automatically

Machine learning model training:
└─ Training data on EFS
└─ Multiple EC2 instances access
└─ Parallel training on shared data

Daily backups:
└─ Snapshot EBS volumes
└─ Upload to S3
└─ Retain for 30 days then delete
```

---

## SAA Exam Topics Covered

✅ **S3**
- Storage classes and lifecycle
- Versioning and durability
- Encryption (SSE-S3, SSE-KMS, SSE-C)
- Access control (public/private)
- Performance optimization

✅ **EBS**
- Volume types (gp2, io1, st1, sc1)
- Snapshots and recovery
- Encryption and replication
- Performance and throughput
- Instance store vs EBS

✅ **EFS**
- When to use EFS vs EBS vs S3
- Throughput modes
- Performance modes
- Multiple instance access
- Regional availability

✅ **Storage Selection**
- Choosing right service
- Cost optimization
- Performance requirements
- Durability and availability

---

## Challenges Completed

### Challenge 1 — S3 Bucket Creation
Created bucket, uploaded file, made public, accessed via URL.
Learned: S3 basics, URL structure, public/private.

### Challenge 2 — S3 with Terraform
Automated S3 with versioning, encryption, lifecycle.
Learned: Infrastructure as code for S3.

### Challenge 3 — EBS Volumes
Created instance, added data volume, created snapshot.
Learned: EBS attachment, snapshots, backup.

### Challenge 4 — EFS Sharing
Created EFS, mounted from multiple subnets.
Learned: Shared file system, multi-instance access.

---

## Terraform Features Used

### S3
```hcl
- Versioning
- Encryption (SSE-S3)
- Lifecycle policies (transition to Glacier)
- Public access block
```

### EBS
```hcl
- Volume creation and attachment
- Encryption
- Snapshots for backups
- Multiple volumes per instance
```

### EFS
```hcl
- File system creation
- Mount targets (multi-subnet)
- Security groups (NFS port 2049)
- Encryption
```

---

## Key Takeaways

1. **S3**: Object storage, versioning, lifecycle, cheap archival
2. **EBS**: Block storage for EC2, fast, single instance, snapshots
3. **EFS**: Shared file system, multiple instances, network-based
4. **Lifecycle**: Auto move old data to cheaper storage
5. **Versioning**: Recover deleted/overwritten files
6. **Snapshots**: Point-in-time backups
7. **Encryption**: Data at rest and in transit
8. **Cost Optimization**: Use Glacier for archives

---

## Statistics

```
Day 30:
- Storage services: 3 (S3, EBS, EFS)
- SAA topics: 8 (storage classes, versioning, lifecycle, encryption, snapshots, types, availability, durability)
- Terraform resources: 15+
- Production setup: Complete
- Cost optimization: Implemented (Glacier lifecycle)
```

---

## Interview Talking Points

"I understand the three main AWS storage services:
- S3 for object storage with versioning and lifecycle policies
- EBS for fast storage attached to EC2 instances
- EFS for shared file systems across multiple instances

I can design cost-optimized storage architectures using lifecycle policies to automatically archive old data to Glacier, implement backup strategies with snapshots, and select the right storage service based on performance and availability requirements."

---
