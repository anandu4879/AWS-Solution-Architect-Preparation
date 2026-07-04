# Day 5 — Load Balancing & Auto Scaling

Today I learned **Load Balancing and Auto Scaling** — distributing traffic, ensuring high availability, and automatically scaling applications.

These are critical for building resilient, scalable applications (10%+ of SAA exam).

---

## The Problem

```
Application is popular
100 users per second
Single instance can handle ~10 req/s

Result:
- Users wait 10+ seconds
- Some get timeout
- Users leave angry

Solution:
- Multiple instances
- Load Balancer distributes traffic
- Auto Scaling adds/removes instances
```

---

## Load Balancers

### Three Types

```
ALB (Application Load Balancer)
├─ Layer 7 (Application)
├─ Smart routing (hostname, path, headers)
├─ Best for: Web apps, microservices
└─ Most common choice

NLB (Network Load Balancer)
├─ Layer 4 (Transport)
├─ Ultra-high performance
├─ Millions of req/second
└─ Best for: Gaming, IoT, extreme performance

CLB (Classic Load Balancer)
├─ Older, less common
├─ Layer 4/7 hybrid
└─ Avoid for new applications
```

### How ALB Works

```
User request to example.com

        ↓
    
┌─────────────────────┐
│ ALB (port 80)       │
├─────────────────────┤
│ Rule 1:             │
│ Path = /api?        │
│ → API servers       │
│                     │
│ Rule 2:             │
│ Hostname = api.?    │
│ → API servers       │
│                     │
│ Rule 3 (Default):   │
│ → Web servers       │
└─────────────────────┘

        ↓

Pick server & send traffic
```

### Listener Rules

```
Rules determine where traffic goes

Example Rules:

Rule 1: If path = /api/*
  → Forward to: API target group

Rule 2: If hostname = api.example.com
  → Forward to: API target group

Rule 3: If path = /admin/*
  → Forward to: Admin target group

Rule 4: (Default)
  → Forward to: Web target group

Rules are evaluated in priority order
First matching rule wins
```

---

## Health Checks

```
Health Check = Verify instance works

Process:
1. ALB sends request
   GET /health HTTP/1.1

2. Instance responds
   HTTP 200 OK

3. ALB marks: Healthy ✅

4. If error:
   HTTP 500
   ALB marks: Unhealthy ❌

5. Stop sending traffic to unhealthy instance

6. Auto Scaling removes it
```

### Configuration

```
Settings:
├─ Protocol: HTTP or HTTPS
├─ Path: /health (endpoint to check)
├─ Port: 80 (usually same as app)
├─ Interval: 30 seconds
├─ Timeout: 5 seconds (wait for response)
├─ Healthy threshold: 2
│   └─ Pass 2 checks to be "healthy"
└─ Unhealthy threshold: 2
    └─ Fail 2 checks to be "unhealthy"

Timeline:
Health check 1: FAIL
Health check 2: FAIL
→ Marked UNHEALTHY
→ Traffic stops
→ Auto Scaling removes
→ New instance launched
→ Health checks pass
→ Marked HEALTHY
→ Traffic starts
```

---

## Target Groups

```
Target Group = Group of instances

Each has:
├─ Name
├─ Protocol (HTTP, TCP)
├─ Port (80, 443, custom)
├─ Health check settings
├─ Instances (EC2 instances)
└─ Status of each instance

Example:
Target Group: api-servers
├─ 3 instances (all healthy)
├─ Health checks: Every 30 seconds
├─ Instance 1: Healthy ✅
├─ Instance 2: Healthy ✅
└─ Instance 3: Unhealthy ❌ (not receiving traffic)

How ALB uses:
Request → Evaluate rules → Select target group
                        → Pick healthy instance
                        → Send request
```

---

## Auto Scaling

### Capacity

```
Min size: 2
├─ Never fewer than 2 instances
└─ Even if traffic is zero

Max size: 4
├─ Never more than 4 instances
└─ Even if traffic is very high

Desired: 2
├─ What we want right now
└─ Auto Scaling adjusts to maintain
```

### Scaling Policies (SAA Critical)

```
Target Tracking Scaling (RECOMMENDED)
├─ "Keep CPU at 70%"
├─ AWS automatically adjusts
├─ Simplest to use
└─ Recommended for most applications

Example:
Target CPU = 70%
Current CPU = 45%
→ All good, do nothing

Current CPU = 85%
→ Add instances
→ Distribute load
→ CPU → 70%

Current CPU = 25%
→ Remove instances
→ Lower bound = min_size
→ Don't go below minimum
```

### Scaling Timeline

```
Time 0: Normal traffic
├─ 2 instances
├─ CPU 50%
└─ All good

Time 1: Traffic spike!
├─ CPU jumps to 85%
├─ Exceeds target 70%
└─ Scaling policy triggered

Time 2: New instance launching
├─ Still 2 instances running
├─ 1 new instance starting
├─ Auto Scaling adds to ASG

Time 3: New instance ready
├─ 3 instances total
├─ Load distributed
├─ CPU per instance 60%
├─ ALB sends traffic to new instance

Time 4: Traffic subsides
├─ CPU drops to 25%
├─ Below target 70%
├─ Scaling policy triggered

Time 5: Instance terminating
├─ Existing connections finish
├─ Remove instance from ALB
├─ Terminate instance
├─ Back to 2 instances
```

---

## CloudWatch Integration

### Metrics Tracked

```
Automatically measured:
├─ CPU Utilization (%)
├─ Network In/Out (bytes)
├─ Request Count
├─ Target Response Time
└─ Health Check Status

Used for:
- Scaling decisions
- Alarms/notifications
- Performance monitoring
```

### Alarms

```
CloudWatch Alarm = Decision trigger

Example:
Alarm: CPU > 70% for 2 minutes
→ Trigger scaling policy
→ Add instance

Alarm: CPU < 30% for 2 minutes
→ Trigger scaling policy
→ Remove instance

Alarms can:
✅ Trigger scaling
✅ Send notifications (SNS)
✅ Execute Lambda functions
```

---

## Real-World Architecture

### E-commerce Website

```
ALB (port 80/443)
├─ Rule 1: /api/* → API ASG
├─ Rule 2: /admin/* → Admin ASG
└─ Rule 3: Default → Web ASG

ASG 1: Web Servers
├─ 2-5 instances
├─ Instance type: t2.medium
├─ Target CPU: 70%
├─ Serves HTML, CSS, JS

ASG 2: API Servers
├─ 2-10 instances
├─ Instance type: t2.large
├─ Target CPU: 75%
├─ Serves /api endpoints

ASG 3: Admin Servers
├─ 1-3 instances
├─ Instance type: t2.small
├─ Manual scaling (less traffic)
├─ Admin panel

Each:
- Health checks every 30s
- Unhealthy instances removed
- Automatic scaling based on CPU
- CloudWatch monitoring
- Access logs
```

### SaaS Application

```
ALB (port 80/443)
└─ Single target group → App ASG

ASG: App Servers
├─ 3-20 instances
├─ Instance type: t2.xlarge
├─ Target CPU: 65%
├─ All instances identical

How traffic scales:
Low traffic: 3 instances
Normal: 5 instances
High: 10 instances
Spike: 15-20 instances (max)

Auto-adjusts:
✅ User experience unchanged
✅ Response time constant
✅ Cost optimized
```

---

## ALB Features

### Sticky Sessions

```
Without sticky:
- Request 1 → Instance A
- Request 2 → Instance B (may lose session)
- Problem: User session not in memory

With sticky (enabled):
- Request 1 → Instance A
- Request 2 → Instance A (same instance)
- Request 3 → Instance A (same instance)
- Benefit: Session data persists

Duration: 1 second - 7 days
Trade-off: Less load distribution, but sessions work
```

### Access Logs

```
ALB logs all requests:

Example log entry:
http 2024-01-15T10:30:45.123456Z day34-alb 10.0.1.5:52214
10.0.2.3:80 0.001 0.002 0.000 200 200 0 12345
"GET /api/products HTTP/1.1" "Mozilla/5.0..." ECDHE-RSA-AES128-GCM-SHA256 TLSv1.2
arn:aws:elasticloadbalancing:...

Contains:
- Timestamp
- Source IP
- Target IP
- Response times
- HTTP status
- Request method
- Browser user agent

Use for:
✅ Debugging
✅ Monitoring
✅ Security analysis
✅ Compliance
```

---

## SAA Exam Topics Covered

✅ **Load Balancer Types**
- ALB (Application), NLB (Network), CLB (Classic)
- When to use each
- Differences and similarities

✅ **ALB Features**
- Listener and listener rules
- Target groups
- Hostname/path-based routing
- Health checks

✅ **Health Checks**
- Configuration
- Automatic removal of unhealthy instances
- Integration with Auto Scaling

✅ **Auto Scaling**
- Capacity (min, max, desired)
- Scaling policies (target tracking, step, simple)
- CloudWatch metrics
- Integration with load balancer

✅ **CloudWatch**
- Metrics (CPU, network, requests)
- Alarms
- Scaling decisions

---


### Key Resources

```hcl
- aws_lb: Create load balancer
- aws_lb_target_group: Create target group
- aws_lb_listener: Create listener
- aws_lb_listener_rule: Create routing rules
- aws_launch_template: Define instance template
- aws_autoscaling_group: Create ASG
- aws_autoscaling_policy: Define scaling policy
- aws_cloudwatch_metric_alarm: Create alarms
```

---

## Best Practices

```bash
☐ Use ALB (unless NLB needed)
☐ Health checks on all target groups
☐ Multiple target groups for microservices
☐ Target tracking scaling (simpler)
☐ CloudWatch monitoring enabled
☐ Access logs enabled (for compliance)
☐ Spread across multiple AZs
☐ Min size >= 2 (for HA)
☐ Health check grace period (300s)
☐ Proper timeout configuration
```

---

## Performance Comparison

```
ALB:
- ~400 new connections/second
- Layer 7 (smarter routing)
- Good for most web apps

NLB:
- 6 million new connections/second
- Layer 4 (ultra-fast)
- For extreme performance

Cost: Similar (price per LB + traffic)
Choose based on: Throughput + routing needs
```

---

## Interview Talking Points

"I can design highly available, scalable applications using Application Load Balancers with listener rules for intelligent routing, health checks for automatic failover, and Auto Scaling Groups with target tracking policies for automatic scaling based on demand. I understand the differences between ALB and NLB, and how to integrate them with monitoring for optimal performance."

---

## Common Mistakes

❌ Single instance (no HA)
- Always use min 2 instances across AZs

❌ Not setting up health checks
- Health checks are essential for failover

❌ Using simple scaling
- Target tracking is simpler and better

❌ No monitoring
- CloudWatch gives visibility

❌ Not spreading across AZs
- One AZ failure → outage

❌ Max size = desired size
- No room for scaling up

```

---

## Conclusion

Load Balancing and Auto Scaling are **critical for reliability**:
- **Load Balancer**: Distributes traffic, intelligent routing
- **Health Checks**: Automatic failover
- **Target Groups**: Organize instances
- **Auto Scaling**: Automatic capacity management
- **CloudWatch**: Visibility and decisions

This day covers ~10% of SAA exam.

Together with everything so far, you now understand the **complete application architecture** from compute to networking to scaling.

Next: Monitoring and Observability (Day 35). 🚀

---

## Cost Impact

Load Balancer: $20-30/month per load balancer
Auto Scaling: Free (charge based on instances)

Example monthly cost:
- ALB: $20
- 3 t2.micro instances: $20
- **Total: ~$40/month**

---

## Resources Used

- AWS ALB documentation
- Auto Scaling documentation
- CloudWatch documentation
- Terraform AWS provider
- SAA study materials