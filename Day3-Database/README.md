# Day 3 — AWS Databases (RDS & DynamoDB) - SAA Week 1 Day 3

Today I learned **AWS Database Services** — structured data storage for production applications.

Two different databases for two different workload types.

---

## Two Database Types

### RDS (Relational Database Service)
```
SQL database (structured)

Use for:
- Customer data (name, email, address)
- Orders (order_id, customer_id, amount)
- Products (product_id, name, price)
- Complex queries and transactions

Think of it as:
Traditional database (like MySQL/PostgreSQL)
But AWS manages the infrastructure
```

### DynamoDB (NoSQL)
```
NoSQL database (unstructured)

Use for:
- Session data (user logins)
- Real-time metrics (page views)
- Massive scale (billions of requests)
- Simple key-value lookups

Think of it as:
Document storage (like MongoDB)
But completely serverless
```

---

## RDS Deep Dive

### Supported Databases

```
PostgreSQL
├── Open source, powerful
├── Full text search, JSON support
└── Great for: complex applications

MySQL / MariaDB
├── Open source, lightweight
├── Good performance
└── Great for: web applications

Oracle Database
├── Enterprise database
├── Powerful, expensive
└── Great for: large enterprises

SQL Server
├── Microsoft database
├── Windows integration
└── Great for: enterprise apps
```

### How RDS Works

```
1. Create RDS Instance
   ├── Choose engine (PostgreSQL, MySQL)
   ├── Choose size (db.t2.micro → db.r5.24xlarge)
   └── Configure storage (20 GB → 65 TB)

2. AWS Manages
   ├── Automated backups (daily)
   ├── Security patches (automatic)
   ├── High availability options
   └── You focus on application, not infrastructure

3. Connect from Application
   └── Use standard database drivers

4. Scale or Backup
   ├── Modify instance class for more power
   ├── Increase storage (up to 65 TB)
   ├── Create manual snapshots
   └── Create read replicas
```

### Multi-AZ (High Availability) - SAA Critical

```
Problem:
Single database in one Availability Zone
If AZ fails → entire database unavailable

Solution: Multi-AZ Deployment
├── Primary database (us-east-1a)
├── Standby replica (us-east-1b)
├── Synchronous replication (zero data loss)
├── Automatic failover (< 2 minutes)
└── If primary fails → standby promoted

Cost: 2x the database cost
Benefit: 99.95% availability SLA

When to use:
✅ Production applications
✅ Mission-critical data
✅ E-commerce, banking, healthcare
```

### Read Replicas (Scaling Reads) - SAA Important

```
Problem:
One database can handle ~1000 reads/second
Website has 10,000 reads/second
Database gets overwhelmed (SLOW!)

Solution: Read Replicas
├── Primary (reads + writes)
├── Replica 1 (reads only)
├── Replica 2 (reads only)
├── Replica 3 (reads only)

Distribution:
- Writes always go to primary
- Reads distributed across replicas
- Replication lag: ~100 milliseconds
- Up to 15 read replicas

Cost: Each replica = 1 database

When to use:
✅ High read traffic (analytics, reporting)
✅ Reduce load on primary
✅ Geographic distribution (low latency)
```

### Backup & Recovery

```
Automatic Backups
├── Daily automated snapshots
├── Transaction logs captured
├── Retention: 1-35 days (default 7)
├── Can restore to any point in time
└── No action needed

Manual Snapshots
├── Create on-demand
├── Unlimited retention
├── Copy to another region
└── For compliance, point-in-time recovery

Restore Process
├── Creates NEW database from snapshot
├── Same data, isolated from original
├── Different endpoint
├── Takes 5-10 minutes
```

### Instance Types (SAA Important)

| Class | Use | Cost |
|-------|-----|------|
| db.t2.micro | Dev/test | Low |
| db.t2.small | Small production | Low |
| db.m5.large | Medium production | Medium |
| db.r5.large | High memory | High |
| db.x1.32xlarge | Enterprise | Very High |

---

## DynamoDB Deep Dive

### NoSQL vs SQL

```
SQL (RDS):
├── Tables with defined schema
├── ACID transactions
├── Complex queries (JOINs)
├── Structured data
└── Limited scaling

NoSQL (DynamoDB):
├── Flexible schema
├── Eventually consistent
├── Simple key lookups
├── Unstructured data
└── Unlimited scaling
```

### DynamoDB Data Model

```
Table = collection of items
Item = single record (like a row)
Attribute = field in item (like a column, but flexible)

Example:
Table: Orders
└── Items:
    ├── Item 1:
    │   ├── order_id: "order-001"
    │   ├── customer_id: "customer-123"
    │   ├── amount: 99.99
    │   └── status: "delivered"
    │
    ├── Item 2:
    │   ├── order_id: "order-002"
    │   ├── customer_id: "customer-456"
    │   └── amount: 149.99

Key Concepts:
- Partition Key: How data is distributed (like customer_id)
- Sort Key: Optional sorting within partition (like timestamp)
```

### Capacity Modes (Billing) - SAA Important

```
Provisioned Capacity
├── You specify RCU (read) and WCU (write)
├── Example: 100 RCU, 100 WCU per second
├── Pay even if you don't use it
├── Good for: Predictable workloads
└── Cost: Cheaper if predictable

On-Demand Capacity
├── Pay per request
├── No pre-provisioning
├── Unlimited scalability
├── Good for: Variable/unpredictable workloads
└── Cost: More expensive, but simpler

Decision:
- Startup with variable traffic → On-Demand
- Established with known traffic → Provisioned
```

### Throughput Concepts

```
Read Capacity Unit (RCU)
├── 1 RCU = 1 strongly consistent read/second
├── Or 2 eventually consistent reads/second
├── For 4 KB item

Write Capacity Unit (WCU)
├── 1 WCU = 1 write/second
├── For 1 KB item

Example Calculation:
- 1000 reads/second of 4 KB items
  └── Need: 1000 RCU

- 500 reads/second of 8 KB items
  └── Need: 1000 RCU (8 KB = 2 units)

- 100 writes/second of 2 KB items
  └── Need: 200 WCU (2 KB = 2 units)
```

### Global Secondary Index (GSI)

```
Problem:
Table indexed by customer_id
Query by order_status? FULL SCAN (SLOW!)

Solution: Global Secondary Index
├── Create index on order_status
├── Query by status (FAST!)
├── Separate throughput
└── Can query any attribute

Performance:
- Without GSI: Scan all items (slow)
- With GSI: Query index (fast)
```

### Backup & Recovery

```
On-Demand Backup
├── Create anytime
├── Full copy of table
├── Unlimited retention
└── Can restore to new table

Point-in-Time Recovery (PITR)
├── Automatic backups last 35 days
├── Restore to any point in time
└── Recover from accidental deletes

Restore Process
├── Creates NEW table (old one untouched)
├── Same structure and data
├── Different table name
```

---

## Real-World Scenarios

### E-commerce Platform

```
Database Layer:
├── RDS (PostgreSQL)
│   ├── Customers table
│   ├── Products table
│   ├── Orders table
│   ├── Multi-AZ (high availability)
│   └── Read replicas (reporting)
│
├── DynamoDB (Sessions)
│   ├── User sessions
│   ├── Shopping carts
│   ├── On-demand (variable traffic)
│   └── TTL (expire after 1 day)
│
└── DynamoDB (Analytics)
    ├── Page views
    ├── Click tracking
    ├── Real-time metrics
    └── Provisioned (predictable)
```

### Mobile App

```
Backend Database:
├── RDS (User profiles, data)
│   └── Read replica in different region (low latency)
│
└── DynamoDB (Real-time)
    ├── Push notifications queue
    ├── User activity
    ├── On-demand (spiky traffic)
    └── Auto scales with traffic
```

---

## SAA Exam Topics Covered

✅ **RDS**
- Database engines (PostgreSQL, MySQL, Oracle, SQL Server)
- Instance classes and sizing
- Multi-AZ (high availability)
- Read replicas (scaling reads)
- Backup and point-in-time recovery
- Storage types (gp2, io1, magnetic)
- Performance and monitoring

✅ **DynamoDB**
- NoSQL vs SQL
- Tables, items, attributes
- Partition key and sort key
- Capacity modes (provisioned vs on-demand)
- Throughput units (RCU, WCU)
- Global Secondary Indexes
- Backup and recovery
- TTL (time-to-live)
- Encryption

✅ **Database Selection**
- When to use RDS vs DynamoDB
- Scaling strategies
- Cost optimization
- Disaster recovery

---


## Terraform Features Used

### RDS
```hcl
- Multi-AZ deployment
- Read replicas
- Automatic backups (retention)
- Parameter groups
- CloudWatch logs
- Encryption
- Snapshots
```

### DynamoDB
```hcl
- Flexible schema
- Global Secondary Index (GSI)
- Billing modes (on-demand & provisioned)
- Point-in-time recovery
- Encryption
- TTL
```

---

## Key Takeaways

1. **RDS**: Managed SQL database, Multi-AZ for HA, read replicas for scale
2. **DynamoDB**: Serverless NoSQL, on-demand for variable workload
3. **Multi-AZ**: Primary + standby, automatic failover, 99.95% availability
4. **Read Replicas**: Scales reads, reduces load on primary
5. **Backup**: Automatic daily snapshots, point-in-time recovery
6. **PITR**: Restore to any second in retention window
7. **GSI**: Query by different attribute in DynamoDB
8. **Capacity**: Provision for known workloads, on-demand for variable

---

## Statistics

```
Day 3:
- Database services: 2 (RDS, DynamoDB)
- SAA topics: 12+ (Multi-AZ, read replicas, backup, indexes, billing modes)
- Terraform resources: 10+
- Production setup: Complete
- Backup strategies: Implemented
```

---

## Interview Talking Points

"I can architect database solutions using both RDS for structured data with Multi-AZ high availability and read replicas for scaling, and DynamoDB for serverless NoSQL at massive scale. I understand backup and disaster recovery strategies, can optimize costs with capacity modes, and can implement point-in-time recovery for compliance."

---


---

## Cost Comparison

| Service | Dev Cost/Month | Prod Cost/Month |
|---------|---|---|
| RDS db.t2.micro | $15 | $30+ (Multi-AZ) |
| RDS read replica | - | $15 |
| DynamoDB on-demand | $5-50 | $50-500 |
| DynamoDB provisioned | $25 | $100+ |

Estimate actual costs in AWS calculator before deploying.