# Lesson 7 — Amazon DocumentDB Architecture on AWS ☁️

Welcome to **Lesson 7 of your Amazon DocumentDB course**.

So far:

```text
✅ Lesson 1 — DocumentDB Fundamentals
✅ Lesson 2 — Documents, Collections & BSON
✅ Lesson 3 — CRUD Operations
✅ Lesson 4 — Queries & Operators
✅ Lesson 5 — Indexes & Query Performance
✅ Lesson 6 — Data Modeling
➡️ Lesson 7 — DocumentDB Architecture on AWS
⬜ Lesson 8 — Security & Networking
⬜ Lesson 9 — Backup, Monitoring & Operations
⬜ Lesson 10 — Production Project
```

Today we move from:

> **"How does DocumentDB store and query documents?"**

to:

> **"How is Amazon DocumentDB actually built and deployed inside AWS?"**

---

# 1. What Is Amazon DocumentDB Architecture?

At a high level, an Amazon DocumentDB cluster consists of:

```text
                    AWS
                     │
                     ▼
             DocumentDB Cluster
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
      Compute                 Storage
      Instances              Cluster Volume
          │                     │
          ├── Primary           │
          ├── Replicas          │
          └── Replicas          │
                                │
                         Shared storage
```

The most important architectural idea is:

> **Compute and storage are separated.**

This is one of the key things you should understand.

---

# 2. DocumentDB Cluster

The fundamental unit is a:

> **DocumentDB cluster**

A cluster contains:

* A cluster storage volume
* One or more database instances
* A primary instance
* Optional replica instances
* Cluster endpoints

Conceptually:

```text
                 DocumentDB Cluster
                        │
          ┌─────────────┴─────────────┐
          │                           │
          ▼                           ▼
       Compute                    Shared Storage
      Instances                  Cluster Volume
          │
     ┌────┼────┐
     ▼    ▼    ▼
 Primary Replica Replica
```

---

# 3. Cluster vs Instance

This distinction is extremely important.

### Cluster

Represents the overall database system.

### Instance

Provides compute capacity to run the database engine.

Think:

```text
Cluster
   │
   ├── Primary instance
   ├── Replica instance
   └── Replica instance
```

The **cluster** owns the storage and database topology.

The **instances** provide compute.

---

# 4. Primary Instance

The primary instance is responsible for:

> **Processing write operations.**

For example:

```javascript
db.customers.insertOne({
    "_id": "C1001",
    "name": "John"
})
```

The application sends the write to the appropriate cluster endpoint.

Conceptually:

```text
Application
     │
     │ WRITE
     ▼
 Primary Instance
     │
     ▼
Cluster Storage
```

---

# 5. Replica Instances

A cluster can have replica instances.

Their main purpose is to provide:

* Read scaling
* High availability
* Failover capability

Conceptually:

```text
                Primary
                   │
          ┌────────┴────────┐
          ▼                 ▼
       Replica 1         Replica 2
```

Applications can direct read traffic to appropriate endpoints.

---

# 6. Read Scaling

Imagine your application receives:

```text
100 writes/sec
10,000 reads/sec
```

You don't necessarily want all read traffic going to one instance.

You can add replicas.

Conceptually:

```text
                Primary
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
    Replica 1  Replica 2  Replica 3
        ▲          ▲          ▲
        │          │          │
        └──────── Read traffic
```

This gives you additional compute capacity for read workloads.

---

# 7. Important: Replicas Are Not Independent Databases

A common beginner misconception is:

```text
Primary
   +
Replica 1
   +
Replica 2
```

means three independent databases.

That's not the right mental model.

They are instances within the same DocumentDB cluster and use the cluster's shared storage architecture.

Think:

```text
        Compute Instances
       ┌──────┬──────┬──────┐
       │      │      │      │
       ▼      ▼      ▼      ▼
    Primary Replica Replica
       │      │      │
       └──────┴──────┴──────┐
                            ▼
                     Shared Storage
```

---

# 8. Separation of Compute and Storage

This is one of the most important concepts in the architecture.

Traditional architecture might look like:

```text
Database Server
 ├── CPU
 ├── RAM
 └── Local Storage
```

DocumentDB uses a different architectural approach.

Conceptually:

```text
Compute
  │
  ├── Primary
  ├── Replica
  └── Replica
        │
        ▼
  Shared Cluster Storage
```

This separation provides architectural flexibility.

---

# 9. Cluster Storage

The cluster has a distributed storage layer.

Conceptually:

```text
                Cluster Storage
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
   Storage copy   Storage copy   Storage copy
```

The storage layer is designed for durability and availability.

The important thing for you to remember is:

> **Instances are compute; the cluster volume is persistent database storage.**

---

# 10. Why Shared Storage Matters

Suppose the primary instance fails.

If the database depended entirely on that instance's local disk, recovery would be much more complicated.

With shared cluster storage:

```text
Primary fails
     │
     ▼
Replica promoted
     │
     ▼
Uses cluster storage
```

This contributes to faster failover.

---

# 11. Availability Zones

AWS infrastructure is organized into:

> **Availability Zones (AZs)**

An AWS Region contains multiple AZs.

Conceptually:

```text
                 AWS Region
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
      AZ-A          AZ-B          AZ-C
       │             │             │
       ▼             ▼             ▼
    Primary       Replica       Replica
```

For production systems, distributing instances across AZs helps protect against an individual AZ failure.

---

# 12. High Availability

High availability means:

> **The system is designed to remain available despite certain infrastructure failures.**

A typical architecture could be:

```text
AZ-A                 AZ-B                 AZ-C
 │                     │                     │
 ▼                     ▼                     ▼
Primary             Replica 1             Replica 2
 │                     │                     │
 └─────────────────────┴─────────────────────┘
                       │
                       ▼
                Cluster Storage
```

If the primary instance becomes unavailable, a replica can be promoted.

---

# 13. Failover

Suppose:

```text
Application
     │
     ▼
 Primary
```

Then the primary experiences a failure:

```text
Primary ❌
```

DocumentDB can perform a failover process.

Conceptually:

```text
Primary ❌
   │
   ▼
Select replica
   │
   ▼
Promote replica
   │
   ▼
New primary
```

The application should connect through the appropriate cluster endpoint rather than hard-coding an individual instance address.

---

# 14. Why Endpoints Matter

Suppose your application connects directly to:

```text
primary-instance.example
```

If that instance fails, your application may continue trying to reach the failed instance.

Instead, you normally use a **cluster endpoint**.

Conceptually:

```text
Application
     │
     ▼
Cluster Endpoint
     │
     ▼
Current Primary
```

If failover occurs:

```text
Application
     │
     ▼
Same Cluster Endpoint
     │
     ▼
New Primary
```

This abstracts the changing topology from the application.

---

# 15. Cluster Endpoint

The cluster endpoint is commonly used for:

> **Write traffic**

Think:

```text
Application
     │
     ▼
Cluster Endpoint
     │
     ▼
Primary
```

The endpoint continues to represent the current primary instance as the cluster topology changes.

---

# 16. Reader Endpoint

DocumentDB also provides a:

> **Reader endpoint**

This is designed for read workloads.

Conceptually:

```text
Application
     │
     ▼
Reader Endpoint
     │
     ├── Replica 1
     ├── Replica 2
     └── Replica 3
```

The reader endpoint can distribute connections/read traffic among replicas.

---

# 17. Cluster Endpoint vs Reader Endpoint

Remember this table:

| Endpoint             | Typical purpose              |
| -------------------- | ---------------------------- |
| **Cluster endpoint** | Writes / primary operations  |
| **Reader endpoint**  | Read traffic across replicas |

Mental model:

```text
WRITE
  │
  ▼
Cluster Endpoint
  │
  ▼
Primary


READ
  │
  ▼
Reader Endpoint
  │
  ├── Replica
  ├── Replica
  └── Replica
```

---

# 18. Instance Endpoint

There can also be an endpoint associated with an individual instance.

Conceptually:

```text
Primary instance endpoint
Replica instance endpoint
```

Applications generally shouldn't use an individual instance endpoint when they need automatic failover behavior.

For normal application connectivity, cluster/reader endpoints are usually more appropriate depending on workload.

---

# 19. Application Architecture

A common AWS architecture could look like:

```text
                    Internet
                       │
                       ▼
                Application
                   Server
                       │
                 ┌─────┴─────┐
                 │           │
              Writes       Reads
                 │           │
                 ▼           ▼
          Cluster Endpoint  Reader Endpoint
                 │           │
                 ▼       ┌───┴───┐
              Primary    ▼       ▼
                       Replica Replica
                 │       │       │
                 └───────┴───────┘
                         │
                         ▼
                  Cluster Storage
```

This is the architecture pattern you should understand.

---

# 20. DocumentDB Inside a VPC

Amazon DocumentDB is designed to run within an AWS:

> **Virtual Private Cloud (VPC)**

This is extremely important.

Your database isn't normally exposed directly to the public internet like a public website.

Conceptually:

```text
                  AWS VPC
┌───────────────────────────────────────────┐
│                                           │
│     Application                           │
│        │                                  │
│        ▼                                  │
│   Private Subnet                          │
│        │                                  │
│        ▼                                  │
│   DocumentDB                              │
│                                           │
└───────────────────────────────────────────┘
```

Networking and security are covered in depth in Lesson 8.

---

# 21. Subnets

A subnet is a subdivision of a VPC IP address range.

For a highly available DocumentDB deployment, instances can be placed in subnets associated with different Availability Zones.

Conceptually:

```text
VPC
│
├── Subnet A → AZ-A
│      └── DocumentDB instance
│
├── Subnet B → AZ-B
│      └── DocumentDB instance
│
└── Subnet C → AZ-C
       └── DocumentDB instance
```

---

# 22. DB Subnet Group

DocumentDB uses a:

> **DB subnet group**

to identify the subnets where database instances can be placed.

Conceptually:

```text
DB Subnet Group
      │
      ├── Subnet A
      ├── Subnet B
      └── Subnet C
```

These subnets should span multiple Availability Zones for resilient deployments.

---

# 23. Security Groups

A DocumentDB cluster can be associated with:

> **Security groups**

Security groups act as virtual firewalls controlling network traffic.

For example:

```text
Application Security Group
          │
          │ TCP database port
          ▼
DocumentDB Security Group
```

Only permitted traffic should be allowed.

You'll learn this in detail in Lesson 8.

---

# 24. Ports

DocumentDB uses the MongoDB-compatible wire protocol and commonly uses:

```text
TCP 27017
```

for client connections.

Conceptually:

```text
Application
    │
    │ TCP 27017
    ▼
DocumentDB
```

Security groups determine which sources are allowed to connect.

---

# 25. Compute Scaling

Remember:

```text
Cluster
   │
   ├── Primary instance
   ├── Replica
   └── Replica
```

The instances provide compute resources such as:

```text
CPU
Memory
Network
```

If your workload requires more compute capacity, you can change instance sizing and/or add replicas depending on the workload.

---

# 26. Vertical vs Horizontal Scaling

Two important concepts:

## Vertical scaling

Increase the size of an instance.

```text
Small instance
     ↓
Larger instance
```

More:

```text
CPU
RAM
```

---

## Horizontal scaling

Add more instances.

```text
Primary
   │
   ├── Replica 1
   ├── Replica 2
   └── Replica 3
```

This is particularly useful for read scaling and availability.

---

# 27. Read Scaling

Suppose:

```text
Application
   │
   └── 50,000 reads/sec
```

You can potentially add replicas:

```text
               Reader Endpoint
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
    Replica 1     Replica 2     Replica 3
```

This distributes read workloads across multiple compute instances.

---

# 28. Write Scaling

This is an important distinction.

Adding replicas primarily helps:

> **Read scalability**

It does not mean:

```text
3 replicas
=
3 independent write engines
```

The cluster has a primary responsible for writes.

Therefore:

```text
More replicas
       ↓
More read capacity
       ≠
Automatically more write capacity
```

For write-heavy workloads, you need to think about data modeling, indexing, instance sizing, workload design, and other architectural approaches rather than simply adding read replicas.

---

# 29. Storage Scaling

The storage layer is separate from the compute instances.

You should understand this conceptually as:

```text
Compute capacity
        ≠
Storage capacity
```

You can scale compute independently from the underlying cluster storage architecture.

This is one of the major benefits of the architecture.

---

# 30. High-Level Storage Architecture

Conceptually:

```text
                 DocumentDB
                     │
        ┌────────────┴────────────┐
        │                         │
     Compute                   Storage
        │                         │
 ┌──────┼──────┐                  │
 ▼      ▼      ▼                  ▼
P      R1     R2            Distributed
                              storage
```

The exact internal implementation is managed by AWS.

As an application developer, you generally don't manage individual storage disks.

---

# 31. What You Manage vs AWS Manages

This is an important cloud concept.

### You manage:

```text
Database configuration
Instance sizing
Cluster topology
Indexes
Queries
Data model
Security configuration
Network configuration
Monitoring
```

### AWS manages much of the underlying infrastructure:

```text
Physical servers
Underlying storage infrastructure
Hardware maintenance
Infrastructure replacement
Availability-zone infrastructure
```

The exact division depends on the feature/configuration, but the important principle is:

> **DocumentDB is a managed database service.**

---

# 32. Managed Service

With a traditional database server, you might have to manage:

```text
OS
Database installation
Storage disks
Hardware
Patching
Backups
Replication setup
Failover infrastructure
```

With Amazon DocumentDB, AWS manages much of the underlying infrastructure.

You still configure and operate the database, but you don't manually build the entire infrastructure stack.

---

# 33. Backup Architecture

DocumentDB supports automated backup capabilities.

Conceptually:

```text
DocumentDB
     │
     ▼
Automated backups
     │
     ▼
Point-in-time recovery capability
```

The exact retention/configuration depends on your cluster settings and AWS service behavior.

We'll cover backups and recovery in much more detail in **Lesson 9**.

---

# 34. Snapshots

You can also create database snapshots.

Think:

```text
Cluster
   │
   ▼
Snapshot
```

A snapshot gives you a point-in-time copy that can be used for recovery or creating another cluster.

Don't confuse:

```text
Automated backups
```

with:

```text
Manual snapshots
```

They serve related but different operational purposes.

---

# 35. Failure Scenario

Let's walk through a realistic scenario.

Architecture:

```text
AZ-A                 AZ-B
 │                     │
 ▼                     ▼
Primary              Replica
 │                     │
 └──────────┬──────────┘
            ▼
       Cluster Storage
```

Now:

```text
AZ-A / Primary
      ❌
```

DocumentDB can detect the failure and promote a suitable replica.

Conceptually:

```text
Primary ❌
   │
   ▼
Replica
   │
   ▼
Promoted to Primary
```

The cluster endpoint then directs clients toward the new primary.

---

# 36. Why Applications Should Handle Failover

Even with managed failover, applications should be designed properly.

Your application should:

* Use appropriate cluster endpoints
* Have sensible connection handling
* Retry transient failures where appropriate
* Avoid assuming a particular instance is permanently primary

Conceptually:

```text
Application
    │
    ▼
Cluster Endpoint
    │
    ▼
Current Primary
```

rather than:

```text
Application
    │
    ▼
Hard-coded instance
```

---

# 37. Read After Write

Here's an important distributed-systems concept.

Suppose:

```text
Application
    │
    ▼
Primary
    │
    └── WRITE
```

Immediately afterward, your application reads through a replica:

```text
Application
    │
    ▼
Reader Endpoint
    │
    ▼
Replica
```

Depending on replication timing and configuration, the replica may not yet reflect the most recent write.

This is a form of:

> **Replication lag**

Therefore, applications must understand the consistency characteristics of reading from replicas.

---

# 38. Replication Lag

Conceptually:

```text
Time →
────────────────────────────────>

Primary:
WRITE X
  │
  ├───────────────►
  │
Replica:
      receives X
```

Between those moments:

```text
Primary knows X
Replica may not yet know X
```

This matters for workloads that require immediately consistent reads after writes.

---

# 39. Choosing Read Destination

Think about the workload.

### Writes

```text
Application
   ↓
Cluster endpoint
   ↓
Primary
```

### Normal scalable reads

```text
Application
   ↓
Reader endpoint
   ↓
Replica
```

### Reads requiring the latest write

Your application may need to carefully choose where it reads from and how it handles consistency/replication behavior.

---

# 40. Cluster Topology

A typical production topology could look like:

```text
                         VPC
                          │
             ┌────────────┴────────────┐
             │                         │
            AZ-A                      AZ-B
             │                         │
             ▼                         ▼
         Primary                   Replica
             │                         │
             └────────────┬────────────┘
                          │
                          ▼
                  Cluster Storage
```

And perhaps:

```text
                         AZ-C
                          │
                          ▼
                       Replica
```

The exact number and configuration depend on your workload and availability requirements.

---

# 41. What Happens During Failover?

Let's simplify it into five steps:

```text
1. Primary fails
        ↓
2. Cluster detects failure
        ↓
3. Suitable replica is selected
        ↓
4. Replica becomes primary
        ↓
5. Cluster endpoint resolves to new primary
```

Your application may experience a temporary interruption while this happens.

Therefore:

> **High availability does not mean zero interruption.**

It means the architecture is designed to recover from certain failures.

---

# 42. Endpoint Mental Model

This is worth memorizing.

```text
                 DOCUMENTDB CLUSTER
                         │
              ┌──────────┴──────────┐
              │                     │
              ▼                     ▼
       Cluster Endpoint       Reader Endpoint
              │                     │
              ▼                     ▼
           Primary              Replicas
```

Then:

```text
Failover
   ↓
New Primary
   ↓
Cluster Endpoint
   ↓
New Primary
```

The application doesn't need to discover the new primary manually when using the appropriate endpoint.

---

# 43. AWS Region vs Availability Zone

Don't confuse these.

### Region

A geographic AWS location.

Example conceptually:

```text
US Region
```

### Availability Zone

An isolated infrastructure location within a Region.

```text
Region
 ├── AZ-A
 ├── AZ-B
 └── AZ-C
```

A DocumentDB cluster's high-availability design uses multiple AZs within a Region.

---

# 44. What About Multi-Region?

A multi-region architecture is different from simply using multiple AZs.

```text
Region A
   │
DocumentDB

Region B
   │
DocumentDB
```

Cross-region architectures involve additional replication/disaster-recovery considerations.

For this course, remember:

```text
Multi-AZ
=
High availability within a Region

Multi-Region
=
Geographic resilience / disaster recovery / global architecture
```

---

# 45. Disaster Recovery

Think in layers:

```text
Instance failure
      ↓
Failover

AZ-level problem
      ↓
Multi-AZ architecture

Regional disaster
      ↓
Potential multi-region DR strategy
```

These are different failure scenarios.

Don't say:

> "Multi-AZ protects me from everything."

It doesn't.

---

# 46. Production Architecture Example

Imagine an application running on AWS:

```text
                         USERS
                           │
                           ▼
                    Load Balancer
                           │
                           ▼
                  Application Servers
                           │
                ┌──────────┴──────────┐
                │                     │
             Writes                 Reads
                │                     │
                ▼                     ▼
        Cluster Endpoint        Reader Endpoint
                │                     │
                ▼              ┌──────┴──────┐
             Primary           ▼             ▼
                              Replica       Replica
                │                │             │
                └────────────────┴─────────────┘
                                 │
                                 ▼
                         Cluster Storage
```

This is the architecture you should be able to draw in an interview.

---

# 47. Architecture Interview Question

### Question:

> What is the difference between a DocumentDB cluster and a DocumentDB instance?

### Strong answer:

> **A cluster represents the overall DocumentDB database deployment, including its shared storage and database topology. Instances provide the compute capacity within the cluster, such as the primary and replica instances.**

---

# 48. Interview Question

### Question:

> What is the purpose of DocumentDB replicas?

Answer:

> **Replicas provide additional read capacity and support high availability and failover. They are not independent write databases; the primary handles the cluster's write workload.**

---

# 49. Interview Question

### Question:

> What is the difference between the cluster endpoint and reader endpoint?

Answer:

> **The cluster endpoint is generally used for connections to the current primary, especially writes, while the reader endpoint is designed to distribute read connections across available replicas.**

---

# 50. Interview Question

### Question:

> Why should an application avoid connecting directly to a specific DocumentDB instance?

Answer:

> **Because the instance's role can change during failover. Using the appropriate cluster endpoint allows the application to continue connecting to the cluster's current primary rather than hard-coding a specific instance.**

---

# 51. Interview Question

### Question:

> How does DocumentDB provide high availability?

A strong answer:

> **DocumentDB can use multiple database instances distributed across Availability Zones. If the primary becomes unavailable, a replica can be promoted, allowing the cluster to recover while using the shared cluster storage.**

---

# 52. Interview Question

### Question:

> Does adding replicas increase write capacity?

Answer:

> **Not directly. Replicas primarily provide additional read capacity and high availability. Writes are handled by the primary.**

---

# 53. Interview Question

### Question:

> What is the difference between vertical and horizontal scaling?

Answer:

> **Vertical scaling increases the capacity of an individual database instance, while horizontal scaling adds additional instances, such as replicas, to increase capacity for workloads such as reads and to improve availability.**

---

# 54. Interview Question

### Question:

> Why does DocumentDB use separate compute and storage architecture?

Answer:

> **Separating compute from persistent cluster storage allows database instances to use shared durable storage, simplifies failover and instance replacement, and provides flexibility in scaling compute independently from the storage layer.**

---

# 55. Architecture Cheat Sheet 🧠

Memorize this:

```text
                    DOCUMENTDB CLUSTER
                           │
             ┌─────────────┴─────────────┐
             │                           │
             ▼                           ▼
         COMPUTE                      STORAGE
             │                           │
      ┌──────┼──────┐                    │
      ▼      ▼      ▼                    │
   Primary Replica Replica               │
      │      │      │                    │
      └──────┴──────┴────────────────────┘
                           │
                           ▼
                    Cluster Storage
```

Endpoints:

```text
                    APPLICATION
                         │
             ┌───────────┴───────────┐
             │                       │
           WRITE                    READ
             │                       │
             ▼                       ▼
      Cluster Endpoint        Reader Endpoint
             │                       │
             ▼                 ┌─────┴─────┐
          Primary              ▼           ▼
                            Replica     Replica
```

Failure:

```text
Primary
   ❌
   │
   ▼
Replica selected
   │
   ▼
New Primary
   │
   ▼
Cluster Endpoint
```

---

# 🎯 Lesson 7 — The 10 Things You Must Remember

### 1.

**Cluster ≠ instance.**

### 2.

The **cluster** represents the overall DocumentDB deployment.

### 3.

**Instances provide compute.**

### 4.

The cluster uses **shared persistent storage**.

### 5.

The **primary handles writes**.

### 6.

**Replicas primarily support reads and high availability.**

### 7.

Use the **cluster endpoint** for primary/write-oriented connections.

### 8.

Use the **reader endpoint** for scalable read workloads.

### 9.

Deploying instances across **multiple Availability Zones** improves resilience.

### 10.

During failure, a suitable replica can be **promoted to primary**, allowing the cluster to recover.

---

# 📝 Lesson 7 Mini Quiz

Try answering these before looking anything up:

### 1.

What is the difference between a DocumentDB cluster and a DocumentDB instance?

### 2.

What is the role of the primary instance?

### 3.

What is the role of replica instances?

### 4.

Does adding three replicas give you four independent write databases?

### 5.

What is the purpose of the cluster endpoint?

### 6.

What is the purpose of the reader endpoint?

### 7.

Why are multiple Availability Zones useful?

### 8.

What happens conceptually when the primary fails?

### 9.

Why shouldn't an application hard-code a specific primary instance?

### 10.

What is the difference between vertical and horizontal scaling?

---

# 📚 Course Progress

```text
✅ Lesson 1 — DocumentDB Fundamentals
✅ Lesson 2 — Documents, Collections & BSON
✅ Lesson 3 — CRUD Operations
✅ Lesson 4 — Queries & Operators
✅ Lesson 5 — Indexes & Query Performance
✅ Lesson 6 — Data Modeling
✅ Lesson 7 — DocumentDB Architecture on AWS
⬜ Lesson 8 — Security & Networking
⬜ Lesson 9 — Backup, Monitoring & Operations
⬜ Lesson 10 — Production Project
```

## Next: **Lesson 8 — Amazon DocumentDB Security & Networking 🔐**

We'll cover **VPCs, subnets, security groups, network paths, TLS/SSL, encryption at rest, encryption in transit, IAM integration, Secrets Manager, authentication, least privilege, and how to securely connect an application to DocumentDB**.
