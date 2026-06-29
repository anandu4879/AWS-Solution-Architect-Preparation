# Day 29 — AWS EC2 & Compute Services (SAA Certification Week 1)

Today I started AWS SAA certification prep focusing on **EC2 (Elastic Compute Cloud)**.

This is Week 1 of a 9-week intensive SAA preparation journey.

---

## What is EC2?

```
EC2 = Elastic Compute Cloud

Virtual computer in the cloud:
├── vCPU (virtual CPU)
├── Memory (RAM)
├── Storage (EBS volume)
├── Network interface
└── Operating system (AMI)

Like renting a computer from AWS:
- Pay hourly
- Scale up/down instantly
- No physical maintenance
```

---

## Why EC2 Matters (SAA Exam)

```
EC2 in SAA:
- 20% of exam questions
- Most important AWS service
- All others build on EC2
- Need deep understanding

Topics covered:
✅ Instance types and sizing
✅ Launching and terminating
✅ Security groups (firewall)
✅ Elastic IPs (fixed IP)
✅ Auto Scaling (horizontal scaling)
✅ Load Balancing (distribute traffic)
✅ CloudWatch monitoring
✅ Instance profiles and IAM
```

---

## EC2 Instance Types

### Naming Convention

```
t2.micro
├── t2 = family (burstable)
└── micro = size (very small)

Instance Families:
- t2/t3: Burstable (testing, small workloads)
- m5/m6: General purpose (balanced)
- c5/c6: Compute optimized (processing)
- r5/r6: Memory optimized (databases)
- i3/i4: Storage optimized (data warehouse)
- p3/p4: GPU (machine learning)
```

### Common Instance Types

| Type | vCPU | Memory | Use Case | Cost |
|------|------|--------|----------|------|
| t2.micro | 1 | 1 GB | Testing, free tier | Low |
| t2.small | 1 | 2 GB | Small apps | Low |
| t2.medium | 2 | 4 GB | Medium apps | Low |
| m5.large | 2 | 8 GB | General purpose | Medium |
| c5.large | 2 | 4 GB | Compute heavy | High |
| r5.large | 2 | 16 GB | Memory heavy | High |

---

## EC2 Instance States

```
Running
├── Instance is active
├── You're being charged
└── Can SSH and use it

Stopped
├── Instance is off
├── You're NOT being charged (compute)
├── Storage still charged
└── Can restart later

Terminated
├── Instance is deleted
├── Can't restart
└── Final cleanup

Restarted ≠ Rebooted
├── Restart = Stop + Start
├── Public IP may change
└── Reboot = keep running
```

---

## Security Groups (SAA Critical)

Security groups are stateful firewalls:

```
Inbound Rules: What traffic can COME IN
├── SSH (port 22)
├── HTTP (port 80)
└── HTTPS (port 443)

Outbound Rules: What traffic can GO OUT
└── All (default)

Key Point:
- Deny all by default
- Allow specific (principle of least privilege)
```

### Example

```
Allow SSH from anywhere:
- Protocol: TCP
- Port: 22
- Source: 0.0.0.0/0 (anywhere)

Allow HTTP from ALB:
- Protocol: TCP
- Port: 80
- Source: sg-12345 (security group)
```

---

## Auto Scaling (Horizontal Scaling)

Problem: Fixed number of instances can't handle traffic spikes.

Solution: Auto Scaling

```
Launch Template (blueprint)
    ↓
Auto Scaling Group (manage fleet)
    ├── Min size: 2 (minimum)
    ├── Max size: 5 (maximum)
    └── Desired: 3 (target)

Scaling Policy (when to scale):
├── Scale Up: If CPU > 70% for 2 periods
└── Scale Down: If CPU < 30% for 2 periods

Result:
- Automatically add instances when busy
- Automatically remove when quiet
- Saves money and improves availability
```

### How It Works

```
Traffic increases
    ↓
CPU goes above 70%
    ↓
CloudWatch alarm triggers
    ↓
Auto Scaling adds instance
    ↓
Load Balancer routes to it
    ↓
Traffic distributed
```

---

## Load Balancing

Application Load Balancer (ALB):

```
Internet traffic
    ↓
Load Balancer (single entry point)
    ↓
Distributes to multiple instances
    ├── Instance 1
    ├── Instance 2
    └── Instance 3

Benefits:
✅ Distributes load
✅ High availability (if one fails, others handle traffic)
✅ Auto scaling friendly
✅ Automatic failover
```

### Health Checks

```
ALB regularly checks instance health:

GET / HTTP/1.1

If response 200: Healthy ✅
  - Send traffic to it

If no response or error: Unhealthy ❌
  - Stop sending traffic
  - Auto Scaling removes it
```

---

## Elastic IP (SAA Important)

Fixed public IP address:

```
Without Elastic IP:
- Stop instance → public IP released
- Start instance → new IP assigned
- Problem: Applications using old IP break

With Elastic IP:
- IP stays assigned even if stopped
- Can reassign to different instance
- Good for DNS that points to IP
```

---

## CloudWatch Monitoring (SAA Important)

AWS monitoring service:

```
Automatically tracks:
├── CPU Utilization
├── Network In/Out
├── Disk Operations
└── Instance Status Checks

Can create alarms:
- Alert if metric exceeds threshold
- Trigger Auto Scaling
- Send notifications
```

---

## Terraform Implementation

### Launch Template

```hcl
resource "aws_launch_template" "app" {
  name_prefix   = "app-"
  image_id      = "ami-123456"
  instance_type = "t2.micro"
  
  user_data = base64encode("#!/bin/bash...")
  
  monitoring {
    enabled = true
  }
}
```

### Auto Scaling Group

```hcl
resource "aws_autoscaling_group" "app" {
  launch_template {
    id      = aws_launch_template.app.id
    version = "$Latest"
  }
  
  min_size         = 2
  max_size         = 5
  desired_capacity = 3
  
  target_group_arns = [aws_lb_target_group.app.arn]
}
```

### Scaling Policies

```hcl
resource "aws_autoscaling_policy" "scale_up" {
  adjustment_type        = "ChangeInCapacity"
  scaling_adjustment     = 1
  autoscaling_group_name = aws_autoscaling_group.app.name
}

resource "aws_cloudwatch_metric_alarm" "cpu_high" {
  comparison_operator = "GreaterThanThreshold"
  threshold          = 70
  alarm_actions      = [aws_autoscaling_policy.scale_up.arn]
}
```

---

## Real-World Scenario

### E-commerce on Black Friday

```
Normal day:
- 2 instances running
- Low CPU usage
- Users happy

Black Friday:
- Traffic spikes
- CPU hits 80%
- Auto Scaling kicks in
- Adds 2 more instances
- CPU drops to 50%
- Users happy
- All automatic!

After Black Friday:
- Traffic drops
- CPU drops to 20%
- Auto Scaling removes instances
- Back to 2
- Costs saved!
```

---

## For SAA Certification

This project demonstrates:

✅ **EC2 Concepts**
- Instance types and sizing
- Instance lifecycle (states)
- Security groups

✅ **Networking**
- Load Balancing (ALB)
- Health checks
- Traffic distribution

✅ **Scaling**
- Auto Scaling Groups
- Scaling policies
- CloudWatch alarms

✅ **Monitoring**
- Metrics collection
- Alarms and thresholds
- Alerting

---


## Key Takeaways

- **EC2**: Most important AWS service
- **Auto Scaling**: Horizontal scaling (add/remove instances)
- **Load Balancing**: Distribute traffic
- **Security Groups**: Firewall for EC2
- **Monitoring**: CloudWatch tracks everything
- **High Availability**: Multiple instances + LB + Auto Scaling

---

## Statistics

```
Day 1:
- EC2 concepts: 8 (types, states, security, scaling, LB, monitoring, Elastic IP, AMI)
- Terraform resources: 10+ (launch template, ASG, ALB, target group, security groups, alarms)
- Production setup: Complete (2-4 instances, auto-scaling, load balancing)
- SAA exam coverage: ~20% (EC2 is major topic)
```

---

## SAA Exam Topics Covered

```
✅ EC2 instance types and families
✅ Instance lifecycle and states
✅ Security groups (stateful firewall)
✅ Auto Scaling Groups (horizontal scaling)
✅ Application Load Balancer (ALB)
✅ Target groups and health checks
✅ Elastic IP (static public IP)
✅ CloudWatch monitoring and alarms
✅ Launch templates and configurations
✅ IAM roles for EC2 (basics)
```

---
## Interview Talking Points

"I can design and implement production-grade EC2 infrastructure with auto-scaling and load balancing. I understand instance types, security groups, and scaling policies. I can deploy using both AWS console and Terraform."

---

## Portfolio Value

This project shows:
- ✅ Understanding of EC2 (core AWS service)
- ✅ Auto Scaling knowledge (production skill)
- ✅ Load Balancing (high availability)
- ✅ Terraform expertise (IaC)
- ✅ AWS best practices
- ✅ SAA exam preparation

---

## Resources Used

- AWS EC2 documentation
- Terraform AWS provider
- CloudWatch monitoring
- Auto Scaling documentation
- SAA study materials

---

## Conclusion

EC2 is the foundation of AWS. Understanding it deeply is essential for:
- AWS certifications (SAA, Solutions Architect Professional)
- DevOps roles
- Cloud architecture
- Production systems

This is Week 1 of SAA prep.
8 weeks remaining.
Building solid foundation.
