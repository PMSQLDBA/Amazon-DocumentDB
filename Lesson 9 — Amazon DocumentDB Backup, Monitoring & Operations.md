# Lesson 9 — Amazon DocumentDB Backup, Monitoring & Operations 📊

Welcome to **Lesson 9** of your Amazon DocumentDB course.

You now know how to work with documents, query them, model data, understand the AWS architecture, and secure the cluster.

Today we're moving into **production operations**:

> **How do I protect my data, monitor the cluster, detect problems, recover from failures, and operate DocumentDB reliably?**

```text
✅ Lesson 1 — DocumentDB Fundamentals
✅ Lesson 2 — Documents, Collections & BSON
✅ Lesson 3 — CRUD Operations
✅ Lesson 4 — Queries & Operators
✅ Lesson 5 — Indexes & Query Performance
✅ Lesson 6 — Data Modeling
✅ Lesson 7 — Architecture on AWS
✅ Lesson 8 — Security & Networking
➡️ Lesson 9 — Backup, Monitoring & Operations
⬜ Lesson 10 — Production Project
```

---

# 1. The Production Operations Picture

A production database needs more than:

```text
Database
  ↓
Queries
```

You need:

```text
                 DOCUMENTDB
                     │
       ┌─────────────┼─────────────┐
       │             │             │
       ▼             ▼             ▼
    Backup       Monitoring     Operations
       │             │             │
       ▼             ▼             ▼
   Recovery       Metrics       Scaling
   Snapshots      Alarms        Failover
   Restore        Logs          Maintenance
```

The three major areas are:

### Backup

> Can I recover my data?

### Monitoring

> Is my database healthy?

### Operations

> Can I keep the database running efficiently?

---

# 2. Backup vs Snapshot vs Recovery

These terms are related but different.

```text
Backup
  ↓
Protect data

Snapshot
  ↓
Point-in-time copy

Recovery
  ↓
Restore data/system
```

A production engineer should understand all three.

---

# 3. Automated Backups

Amazon DocumentDB supports automated backups.

Conceptually:

```text
DocumentDB
     │
     ▼
Automated Backup
     │
     ├── Historical recovery
     └── Point-in-time recovery
```

Automated backups are intended to help you recover from situations such as:

```text
Accidental deletion
Bad application deployment
Data corruption
Operational mistakes
```

---

# 4. Point-in-Time Recovery

One of the most important concepts.

Suppose:

```text
10:00 → Good data
11:00 → Good data
12:00 → Good data
13:00 → Application bug
14:00 → Bad data
```

You discover the problem at 14:30.

With point-in-time recovery, you can restore the database to an appropriate earlier point within the available backup window.

Conceptually:

```text
Timeline

10:00 ─── 11:00 ─── 12:00 ─── 13:00 ─── 14:00
                         ▲
                         │
                  Desired recovery
```

The exact retention period and supported recovery behavior depend on the cluster configuration and current DocumentDB service capabilities.

---

# 5. Why Point-in-Time Recovery Matters

Imagine someone accidentally runs:

```javascript
db.customers.deleteMany({})
```

😱

You don't necessarily want:

> "Restore the last backup from yesterday."

You may want:

> "Recover the database to just before the accidental deletion."

That's the value of point-in-time recovery.

---

# 6. Manual Snapshots

You can also create manual cluster snapshots.

Conceptually:

```text
DocumentDB Cluster
       │
       ▼
Manual Snapshot
       │
       ▼
Stored recovery point
```

Manual snapshots are useful for things such as:

* Major application changes
* Before migrations
* Before risky maintenance
* Creating a known recovery point
* Long-term retention strategies

---

# 7. Automated Backup vs Manual Snapshot

Think of them this way:

| Feature           | Automated Backup            | Manual Snapshot         |
| ----------------- | --------------------------- | ----------------------- |
| Purpose           | Ongoing recovery protection | Explicit recovery point |
| Created           | Automatically               | You initiate it         |
| Useful for        | Point-in-time recovery      | Planned milestones      |
| Operational model | Continuous protection       | Deliberate checkpoint   |

A simple mental model:

```text
Automated backup
=
"Keep protecting me."

Manual snapshot
=
"Save this state now."
```

---

# 8. Snapshot Before Risky Operations

Suppose you're about to perform a major migration:

```text
Production
   │
   ▼
Data migration
```

A good operational approach is:

```text
Production
   │
   ▼
Create recovery point
   │
   ▼
Perform migration
```

If something goes badly:

```text
Migration failure
      │
      ▼
Recovery plan
```

A backup isn't a substitute for testing, but it is an important safety net.

---

# 9. Restore

Restoring a database generally means creating a restored environment from a backup or snapshot.

Conceptually:

```text
Snapshot
    │
    ▼
Restore
    │
    ▼
New DocumentDB Cluster
```

Don't think of restoration as:

```text
Snapshot
   ↓
Magic rewind of the existing production cluster
```

Operationally, restoring often results in a separate cluster that you validate before directing application traffic to it.

---

# 10. Disaster Recovery Flow

A simplified recovery process:

```text
        FAILURE
           │
           ▼
     Identify problem
           │
           ▼
    Choose recovery point
           │
           ▼
      Restore cluster
           │
           ▼
      Validate data
           │
           ▼
      Validate application
           │
           ▼
       Redirect traffic
```

That last part is extremely important.

> **A backup is only useful if you know how to restore it.**

---

# 11. Backup Testing

Imagine your organization says:

> "We have backups."

Ask:

> **Have you successfully restored one?**

If nobody has tested recovery, your disaster-recovery plan is theoretical.

A production team should periodically test:

```text
Backup
  ↓
Restore
  ↓
Validate
  ↓
Application test
```

---

# 12. RPO and RTO

Two very important operations concepts.

## RPO

> **Recovery Point Objective**

How much data loss can you tolerate?

Example:

```text
RPO = 5 minutes
```

means the business might tolerate losing up to approximately five minutes of data, depending on the recovery architecture.

---

## RTO

> **Recovery Time Objective**

How long can recovery take?

Example:

```text
RTO = 30 minutes
```

means the target is to restore service within roughly 30 minutes.

---

# 13. RPO vs RTO

Memorize:

```text
RPO
=
How much data can I lose?

RTO
=
How long can I be down?
```

Example:

```text
Database disaster

RPO → 5 minutes
RTO → 30 minutes
```

These requirements influence your backup and disaster-recovery design.

---

# 14. High Availability vs Backup

Don't confuse them.

### High availability

Protects primarily against:

```text
Instance failure
Availability Zone issues
```

through cluster topology and failover.

### Backup

Protects against:

```text
Accidental deletion
Bad data changes
Corruption
Operational mistakes
```

Think:

```text
HA
↓
"Keep the service running."

Backup
↓
"Help me recover my data."
```

You need both.

---

# 15. Monitoring

Now let's move to monitoring.

A database can be:

```text
UP
```

and still be unhealthy.

For example:

```text
Database reachable
    ✓

CPU overloaded
    ✗

Connections exhausted
    ✗

Queries slow
    ✗
```

Therefore:

> **Availability is not the same as health.**

---

# 16. Amazon CloudWatch

AWS uses:

> **Amazon CloudWatch**

for monitoring AWS resources and workloads.

Conceptually:

```text
DocumentDB
    │
    ▼
CloudWatch
    │
 ┌──┼─────────────┐
 ▼  ▼             ▼
Metrics        Alarms       Logs
```

CloudWatch helps you understand how the cluster is behaving.

---

# 17. What Should You Monitor?

At a high level:

```text
CPU
Memory
Connections
Database load
Latency
Read/write activity
Storage
Replication
Network
Errors
```

The exact available metrics depend on the DocumentDB configuration and AWS service capabilities.

---

# 18. CPU Utilization

Suppose your instance is running:

```text
CPU = 95%
```

for an extended period.

That may indicate:

```text
Heavy queries
Too many operations
Insufficient instance capacity
Poor indexing
Application workload spike
```

But don't immediately conclude:

> "CPU is high, therefore the database needs a larger instance."

First investigate **why** CPU is high.

---

# 19. Memory Pressure

Memory is extremely important for databases.

Why?

Because databases benefit from keeping frequently accessed data and indexes in memory.

If memory pressure becomes significant, you can see:

```text
Slower operations
More I/O
Higher latency
```

So don't monitor only CPU.

---

# 20. Database Connections

Applications create connections to DocumentDB.

For example:

```text
Application Server 1 → connections
Application Server 2 → connections
Application Server 3 → connections
```

If your application creates too many connections:

```text
Connections
     ↑
     ↑
     ↑
Resource pressure
```

This can become a production problem.

---

# 21. Connection Pooling

A common application-side technique is:

> **Connection pooling**

Instead of opening a new database connection for every request:

```text
Request 1 → new connection
Request 2 → new connection
Request 3 → new connection
...
```

use a reusable pool:

```text
Application
    │
    ▼
Connection Pool
 ┌──┬──┬──┬──┐
 ▼  ▼  ▼  ▼  ▼
 DB connections
```

This reduces unnecessary connection creation.

---

# 22. Why Connection Storms Are Dangerous

Imagine an application suddenly starts 5,000 instances.

Each instance creates:

```text
100 DB connections
```

You could quickly create:

```text
500,000 connections
```

That can overwhelm the database.

Therefore:

> **Application scaling and database connection management must be designed together.**

---

# 23. Database Load

You also want to understand:

> **How much work is the database actually doing?**

A workload might look like:

```text
Reads: 90%
Writes: 10%
```

or:

```text
Reads: 20%
Writes: 80%
```

These workloads require different optimization strategies.

---

# 24. Read-Heavy Workload

Example:

```text
90,000 reads
10,000 writes
```

You may consider:

```text
Indexes
Query optimization
Read replicas
Reader endpoint
Caching
```

Conceptually:

```text
                Reads
                  │
                  ▼
             Reader Endpoint
             ┌────┼────┐
             ▼    ▼    ▼
            R1   R2    R3
```

---

# 25. Write-Heavy Workload

Example:

```text
10,000 reads
90,000 writes
```

You need to pay close attention to:

```text
Data model
Indexes
Document size
Write patterns
Primary instance capacity
Application batching
```

Adding read replicas doesn't automatically solve a write bottleneck.

Remember Lesson 7:

> **Replicas primarily provide read scaling.**

---

# 26. Latency

Latency tells you:

> **How long an operation takes.**

For example:

```text
Query A → 5 ms
Query B → 20 ms
Query C → 900 ms
```

That 900 ms query deserves investigation.

Potential causes:

```text
Missing index
Poor query pattern
Large documents
High database load
Resource contention
Network issues
```

---

# 27. Slow Queries

Suppose your application performs:

```javascript
db.orders.find({
    "customerId": "C1001"
})
```

and it becomes slow.

Possible investigation:

```text
1. Does an index exist?
2. Is the query using the index?
3. How many documents are examined?
4. How much data is returned?
5. Is the database under load?
```

This connects directly to **Lesson 5 — Indexes & Query Performance**.

---

# 28. Query Profiling

When investigating query performance, query profiling can provide information about operations and their execution behavior.

The exact profiling features and configuration available in Amazon DocumentDB can differ from those in MongoDB.

Important lesson:

> **Don't assume every MongoDB monitoring/profiling feature exists identically in DocumentDB.**

DocumentDB is MongoDB-compatible, but it is **not identical to MongoDB**.

---

# 29. DocumentDB Compatibility

This is an important theme throughout your course.

You may know MongoDB commands such as:

```javascript
db.collection.find(...)
```

and they may work.

But:

```text
MongoDB
   ≠
Amazon DocumentDB
```

They share compatibility, but there are feature and behavior differences.

When designing production systems, always verify DocumentDB-specific support.

---

# 30. CloudWatch Alarms 🚨

Monitoring is useful.

But automated alerts are even better.

For example:

```text
CPU > threshold
       │
       ▼
CloudWatch Alarm
       │
       ▼
Notification
       │
       ▼
Operations Team
```

You don't want someone manually staring at a dashboard 24/7.

---

# 31. Example Alarm Strategy

Imagine:

```text
CPU > 80%
for 10 minutes
```

could trigger:

```text
CloudWatch Alarm
      ↓
SNS notification
      ↓
On-call engineer
```

The exact thresholds should be based on your workload rather than arbitrary numbers.

---

# 32. Don't Alert on Everything

If you create:

```text
100 alarms
```

and all of them constantly trigger, your team starts ignoring alerts.

This creates:

> **Alert fatigue**

Good monitoring focuses on actionable signals.

A useful alert should answer:

```text
Is something wrong?
Does someone need to act?
How urgent is it?
```

---

# 33. CloudTrail

Lesson 8 introduced CloudTrail.

Remember:

> **CloudTrail is primarily for AWS API activity and auditing.**

For example:

```text
Who modified the DocumentDB cluster?

Who changed the security group?

Who changed an AWS configuration?
```

This is different from performance monitoring.

---

# 34. CloudWatch vs CloudTrail

Memorize:

```text
CloudWatch
   ↓
How is my system behaving?

CloudTrail
   ↓
Who did what through AWS APIs?
```

Example:

```text
CPU = 90%
      ↓
CloudWatch


Security group changed by IAM user
      ↓
CloudTrail
```

---

# 35. Operational Dashboards

A production team can build a dashboard showing:

```text
DocumentDB Dashboard

CPU
Memory
Connections
Latency
Database load
Read operations
Write operations
Replication
Storage
Errors
```

The goal is:

> **See the health of the database quickly.**

---

# 36. Scaling Operations

Eventually your workload grows.

You might have:

```text
Day 1
10,000 requests/day

Day 100
10,000,000 requests/day
```

Your database architecture may need to change.

Possible actions:

```text
Increase instance size
Add replicas
Optimize indexes
Optimize queries
Change data model
Improve connection pooling
Introduce caching
```

Scaling isn't always:

> "Buy a bigger server."

---

# 37. Vertical Scaling

You learned this in Lesson 7.

```text
Current
   │
   ▼
Larger instance
```

Useful when:

```text
CPU constrained
Memory constrained
Workload requires more compute
```

But it may involve an operational event such as a maintenance/restart depending on the change.

Plan changes carefully in production.

---

# 38. Horizontal Scaling

For read-heavy workloads:

```text
Primary
   │
   ├── Replica 1
   ├── Replica 2
   └── Replica 3
```

Then:

```text
Reader Endpoint
       │
   ┌───┼───┐
   ▼   ▼   ▼
  R1  R2  R3
```

This gives additional read capacity and improves availability.

---

# 39. Scaling Isn't the First Optimization

Suppose a query takes:

```text
3 seconds
```

You discover it scans a huge number of documents because there's no appropriate index.

Adding more hardware might help somewhat, but the better first step could be:

```text
Fix query
   ↓
Add appropriate index
   ↓
Reduce work
```

Therefore:

> **Optimize before blindly scaling.**

---

# 40. Maintenance

Managed databases still require operational maintenance.

You need to think about:

```text
Engine updates
Instance changes
Parameter/configuration changes
Security updates
Application compatibility
Maintenance windows
```

AWS manages much of the infrastructure, but you still own operational decisions.

---

# 41. Maintenance Window

A maintenance window is a period during which planned maintenance can occur.

Conceptually:

```text
Production traffic
       │
       ▼
Maintenance Window
       │
       ▼
Planned change
```

Choose maintenance periods that minimize business impact.

---

# 42. Failover Testing

High availability shouldn't merely exist on paper.

You should understand:

```text
What happens if primary fails?
```

A mature operations team tests failure scenarios.

Conceptually:

```text
Primary
   │
   X
   │
   ▼
Replica
   │
   ▼
New Primary
```

Then verify:

```text
Application reconnects
Queries resume
No unexpected data loss
Alerts work
```

---

# 43. Application Retry Logic

During failover, the application may temporarily receive errors.

A resilient application can use controlled retries.

Conceptually:

```text
Query
  │
  ▼
Failure?
  │
 Yes
  │
  ▼
Wait briefly
  │
  ▼
Retry
```

But don't do:

```text
while(true) retry immediately
```

That can create a retry storm.

---

# 44. Exponential Backoff

A common retry strategy is:

```text
Attempt 1
   ↓
wait 100 ms

Attempt 2
   ↓
wait 200 ms

Attempt 3
   ↓
wait 400 ms

Attempt 4
   ↓
wait 800 ms
```

This is called:

> **Exponential backoff**

The exact values depend on your application.

---

# 45. Retry Only Appropriate Errors

Not every error should be retried.

For example:

```text
Temporary network error
        ↓
Potentially retry
```

But:

```text
Invalid credentials
        ↓
Retrying won't fix it
```

Similarly:

```text
Invalid query
        ↓
Retrying the same query
```

usually won't solve the problem.

Your application should distinguish transient failures from permanent failures.

---

# 46. Operational Runbook

A production team should have a runbook.

For example:

```text
PROBLEM:
DocumentDB latency increased.

STEP 1:
Check CloudWatch metrics.

STEP 2:
Check CPU/memory/load.

STEP 3:
Check connections.

STEP 4:
Check slow queries.

STEP 5:
Check recent deployments.

STEP 6:
Check replication/cluster health.

STEP 7:
Determine mitigation.

STEP 8:
Document incident.
```

This makes incident response much more systematic.

---

# 47. Incident Example

Imagine your API suddenly becomes slow.

```text
Users
  ↓
API slow
```

You check:

```text
DocumentDB CPU = 95%
```

Then:

```text
Connections = normal
```

You inspect queries and discover:

```text
New query
   ↓
Missing index
   ↓
Huge collection scan
```

The problem wasn't:

> "AWS is slow."

It was:

> **An inefficient database query.**

This is why monitoring + query knowledge must work together.

---

# 48. Another Incident Example

Suppose:

```text
Application
   ↓
DocumentDB
   X
Connection timeout
```

You check:

```text
CPU = normal
Memory = normal
```

Then inspect networking:

```text
Security group changed
   ↓
Port 27017 no longer allowed
```

CloudTrail can help identify who changed the AWS configuration.

This is a perfect example of:

```text
CloudWatch + CloudTrail + networking knowledge
```

working together.

---

# 49. Backup Incident Example

Suppose a developer accidentally deletes data:

```javascript
db.orders.deleteMany({
    "status": "test"
})
```

Oops.

Your response:

```text
1. Stop further destructive activity
2. Determine exact incident time
3. Identify recovery point
4. Restore appropriate backup
5. Validate restored data
6. Determine how to reconcile changes
7. Resume application operations
```

This is why backup testing matters.

---

# 50. Backup Doesn't Replace Data Protection

Backups are only one layer.

A production system should also consider:

```text
Authorization
Validation
Application safeguards
Change controls
Testing
Monitoring
Backups
```

For example, preventing an accidental:

```javascript
deleteMany({})
```

may be better than relying entirely on recovery afterward.

---

# 51. Operational Best Practices

A strong DocumentDB production setup generally includes:

```text
✓ Automated backups
✓ Tested restore procedures
✓ Appropriate snapshot strategy
✓ CloudWatch monitoring
✓ Useful alarms
✓ CloudTrail auditing
✓ Connection pooling
✓ Query/index monitoring
✓ Capacity planning
✓ Failover planning
✓ Maintenance planning
✓ Incident runbooks
```

---

# 52. Capacity Planning

Don't wait until:

```text
CPU = 99%
```

before thinking about scaling.

Watch trends:

```text
Month 1 → 30%
Month 2 → 40%
Month 3 → 50%
Month 4 → 65%
Month 5 → 75%
```

You can then plan capacity before the system reaches a crisis.

---

# 53. Trend vs Snapshot

One metric at one moment:

```text
CPU = 70%
```

doesn't tell you much.

But:

```text
CPU
100% ┤
 80% ┤                  ╭─
 60% ┤             ╭────╯
 40% ┤       ╭─────╯
 20% ┤───────╯
     └────────────────────
           Time →
```

shows a trend.

Operations is about understanding:

> **How the system is changing over time.**

---

# 54. Production Health Model

I want you to think about DocumentDB health in five categories:

```text
                DATABASE HEALTH
                      │
       ┌──────────────┼──────────────┐
       │              │              │
       ▼              ▼              ▼
 Performance      Availability      Data
       │              │              │
       ▼              ▼              ▼
 CPU/Memory       Failover        Backup
 Latency          Replicas        Recovery
 Queries          AZs             Snapshots

       ┌──────────────┴──────────────┐
       ▼                             ▼
    Security                      Capacity
       │                             │
       ▼                             ▼
 TLS/IAM/SG                    Scaling/Planning
```

---

# 55. The Production Lifecycle

A database isn't:

```text
Create → Done
```

It's:

```text
Create
  ↓
Secure
  ↓
Monitor
  ↓
Optimize
  ↓
Scale
  ↓
Backup
  ↓
Test recovery
  ↓
Maintain
  ↓
Monitor
  ↓
Repeat
```

That's what database operations really means.

---

# 56. Interview Question — Backups

### Q:

> What is the purpose of automated backups?

### Answer:

> **Automated backups provide ongoing protection of the database and support recovery, including point-in-time recovery within the configured backup retention period.**

---

# 57. Interview Question — Snapshot

### Q:

> Why would you create a manual snapshot?

### Answer:

> **A manual snapshot provides an explicit recovery point that can be useful before major changes, migrations, or other risky operations.**

---

# 58. Interview Question — RPO

### Q:

> What is RPO?

### Answer:

> **Recovery Point Objective defines how much data loss an organization can tolerate following a failure.**

---

# 59. Interview Question — RTO

### Q:

> What is RTO?

### Answer:

> **Recovery Time Objective defines how quickly the service should be restored after a failure.**

---

# 60. Interview Question — CloudWatch

### Q:

> What would you monitor for DocumentDB?

### Answer:

> **I would monitor resource utilization, connections, database workload, latency, read/write activity, replication-related health, storage-related metrics, and errors, and create actionable CloudWatch alarms around important thresholds and trends.**

---

# 61. Interview Question — CloudTrail

### Q:

> Why would you use CloudTrail with DocumentDB?

### Answer:

> **CloudTrail helps audit AWS API activity, such as who changed cluster or networking configuration and when those changes occurred.**

---

# 62. Interview Question — Failover

### Q:

> How would you prepare an application for DocumentDB failover?

### Answer:

> **I would use the appropriate cluster endpoints rather than hard-coded instance endpoints, configure the driver correctly, use connection pooling, and implement controlled retry behavior for transient connection failures.**

---

# 63. Interview Question — Scaling

### Q:

> Your DocumentDB CPU is constantly high. What do you do?

Don't immediately say:

> "Increase the instance."

A stronger answer is:

```text
1. Confirm the workload.
2. Examine CPU and other resource metrics.
3. Identify expensive queries.
4. Check indexes.
5. Check connection behavior.
6. Optimize queries/data model if appropriate.
7. Then evaluate vertical or horizontal scaling.
```

That's a much more senior answer.

---

# 64. Interview Question — Backup Testing

### Q:

> How do you know your backup strategy works?

Answer:

> **By periodically performing restore tests and validating that the restored cluster contains the expected data and that the application can operate against it.**

---

# 65. Your Production Architecture

Combine Lessons 7, 8, and 9:

```text
                         USERS
                           │
                           ▼
                    Load Balancer
                           │
                           ▼
                   Application Tier
                           │
                  ┌────────┴────────┐
                  │                 │
                IAM              Secrets
                  │              Manager
                  │                 │
                  └────────┬────────┘
                           │
                           ▼
                     Private VPC
                           │
                     Security Group
                           │
                 ┌─────────┴─────────┐
                 │                   │
                 ▼                   ▼
              Primary             Replicas
                 │                   │
                 └─────────┬─────────┘
                           │
                           ▼
                    Cluster Storage
                           │
                     Encryption/KMS

          ┌────────────────┴────────────────┐
          │                                 │
      CloudWatch                         CloudTrail
          │                                 │
      Metrics/Alarms                    AWS Audit

                           │
                           ▼
                      Backups
                           │
                           ▼
                     Recovery
```

That is the overall production mindset.

---

# 66. 🧠 Lesson 9 Cheat Sheet

```text
BACKUP
  ↓
Automated backups
Manual snapshots
Point-in-time recovery

RECOVERY
  ↓
Choose recovery point
Restore
Validate
Redirect traffic

MONITORING
  ↓
CloudWatch
Metrics
Logs
Alarms

AUDITING
  ↓
CloudTrail

OPERATIONS
  ↓
Scaling
Failover
Maintenance
Connection pooling
Query optimization

DR
  ↓
RPO = How much data can I lose?
RTO = How quickly must I recover?
```

---

# 🎯 Lesson 9 — The 10 Things You Must Remember

### 1.

**Automated backups protect your data continuously.**

### 2.

**Manual snapshots create deliberate recovery points.**

### 3.

**Point-in-time recovery helps recover to an earlier point within the available backup window.**

### 4.

**RPO = acceptable data loss.**

### 5.

**RTO = acceptable recovery time.**

### 6.

**CloudWatch = operational monitoring.**

### 7.

**CloudTrail = AWS API auditing.**

### 8.

**High availability and backups solve different problems.**

### 9.

**Connection pooling and retry logic are important application-side operational practices.**

### 10.

**Always test your restore and failover procedures—not just your backup creation.**

---

# 📝 Lesson 9 Mini Quiz

Try these without looking back:

### 1.

What is the difference between an automated backup and a manual snapshot?

### 2.

What is point-in-time recovery?

### 3.

What does RPO mean?

### 4.

What does RTO mean?

### 5.

What is the difference between CloudWatch and CloudTrail?

### 6.

Why should an application use connection pooling?

### 7.

Why shouldn't you immediately scale the DocumentDB instance when CPU is high?

### 8.

What should you do to verify that backups actually work?

### 9.

How should an application behave during a temporary failover-related connection failure?

### 10.

Why are backups and high availability both necessary?

---

# 📚 Course Progress

```text
✅ Lesson 1 — DocumentDB Fundamentals
✅ Lesson 2 — Documents, Collections & BSON
✅ Lesson 3 — CRUD Operations
✅ Lesson 4 — Queries & Operators
✅ Lesson 5 — Indexes & Query Performance
✅ Lesson 6 — Data Modeling
✅ Lesson 7 — Architecture on AWS
✅ Lesson 8 — Security & Networking
✅ Lesson 9 — Backup, Monitoring & Operations
⬜ Lesson 10 — Production Project
```

## Next: **Lesson 10 — Amazon DocumentDB Production Project 🚀**

This will bring everything together: **VPC → security groups → DocumentDB cluster → collections → data modeling → CRUD → indexes → queries → application connectivity → monitoring → backups → failover → production troubleshooting**.
