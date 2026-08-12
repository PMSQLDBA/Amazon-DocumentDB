# Lesson 10 — Amazon DocumentDB Production Project 🚀

Welcome to the **final lesson**.

This lesson is different from the previous nine. Instead of learning one isolated concept, we're going to build a **production-style Amazon DocumentDB application architecture from end to end**.

The goal is that after this lesson, you can look at an Amazon DocumentDB system and explain:

> **Why it is designed this way, how the application connects to it, how the data is modeled, how queries are optimized, and how the system is secured, monitored, backed up, and recovered.**

---

# 🏗️ Our Production Project

We'll build a fictional:

> **E-Commerce Order Management System**

The application will manage:

```text
Customers
Products
Orders
```

Our architecture:

```text
                         USERS
                           │
                           ▼
                    Load Balancer
                           │
                           ▼
                   Application API
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
        Secrets Manager               IAM
              │
              │ credentials
              ▼
        ┌──────────────────────────────┐
        │             VPC              │
        │                              │
        │       Private Subnets        │
        │                              │
        │        ┌───────────┐         │
        │        │ DocumentDB│         │
        │        │  Cluster  │         │
        │        └─────┬─────┘         │
        │              │               │
        │       ┌──────┴──────┐        │
        │       ▼             ▼        │
        │    Primary        Replicas   │
        │                              │
        └──────────────────────────────┘
                │
        ┌───────┴────────┐
        ▼                ▼
   CloudWatch        CloudTrail
   Monitoring        Auditing
        │
        ▼
     Backups
```

---

# 1. Project Requirements

Imagine the business tells us:

> "We need an online ordering system that stores customers, products, and orders."

The system needs to:

### Customers

```text
Create customer
Get customer
Update customer
```

### Products

```text
Create product
Search products
Update inventory
```

### Orders

```text
Create order
Get order
Find customer's orders
Update order status
```

And production requires:

```text
✓ High availability
✓ Secure networking
✓ Encryption
✓ Backups
✓ Monitoring
✓ Good query performance
✓ Controlled database access
```

---

# 2. Step 1 — Design the Data Model

We need to decide:

```text
What documents do we store?
What collections do we need?
What relationships exist?
What queries will be common?
```

Our initial collections:

```text
ecommerce
│
├── customers
├── products
└── orders
```

---

# 3. Customers Collection

A customer document might look like:

```javascript
{
  _id: ObjectId("..."),
  customerId: "C1001",
  name: "John Smith",
  email: "john@example.com",
  address: {
    city: "New Orleans",
    state: "LA",
    country: "USA"
  },
  createdAt: ISODate("2026-08-11T10:00:00Z")
}
```

Notice that the address is embedded.

Why?

Because the application usually retrieves the customer's address together with the customer.

---

# 4. Products Collection

Example:

```javascript
{
  _id: ObjectId("..."),
  productId: "P1001",
  name: "Wireless Keyboard",
  category: "electronics",
  price: 59.99,
  inventory: 125,
  tags: [
    "keyboard",
    "wireless",
    "bluetooth"
  ],
  createdAt: ISODate("2026-08-11T10:00:00Z")
}
```

This is a natural document-oriented structure.

---

# 5. Orders Collection

Now the interesting one.

```javascript
{
  _id: ObjectId("..."),
  orderId: "O10001",
  customerId: "C1001",

  items: [
    {
      productId: "P1001",
      name: "Wireless Keyboard",
      quantity: 2,
      unitPrice: 59.99
    },
    {
      productId: "P2001",
      name: "Mouse",
      quantity: 1,
      unitPrice: 29.99
    }
  ],

  total: 149.97,

  status: "PLACED",

  shippingAddress: {
    city: "New Orleans",
    state: "LA",
    country: "USA"
  },

  createdAt: ISODate("2026-08-11T10:30:00Z")
}
```

Notice something important.

We stored:

```text
productId
name
unitPrice
```

inside the order item.

Why store the product name and price again?

Because an order represents a **historical transaction**.

If the product price later changes from:

```text
$59.99
```

to:

```text
$69.99
```

we don't want an old order suddenly displaying the new price.

That's an important data-modeling decision.

---

# 6. Embedding vs Referencing

Our design uses both.

### Embedded

```text
Order
 └── items[]
```

### Referenced

```text
Order
 └── customerId
```

Why?

Because the relationship and access patterns are different.

This is exactly what you learned in **Lesson 6 — Data Modeling**.

---

# 7. Step 2 — Create the Database

We need an Amazon DocumentDB cluster.

Conceptually:

```text
AWS Account
     │
     ▼
    VPC
     │
     ▼
DB Subnet Group
     │
     ▼
DocumentDB Cluster
```

We should plan:

```text
Region
VPC
Subnets
Availability Zones
Instance class
Cluster size
Security groups
Encryption
Backup retention
```

---

# 8. Step 3 — Network Design

Our application and database should not simply sit on the public internet.

Instead:

```text
                    VPC
                     │
        ┌────────────┴────────────┐
        │                         │
        ▼                         ▼
 Application Subnets         DB Subnets
        │                         │
        ▼                         ▼
   Application                DocumentDB
```

The database is placed in private networking.

---

# 9. Step 4 — Security Groups

Create:

```text
SG-App
SG-DocumentDB
```

Then configure:

```text
SG-DocumentDB
       │
       │ TCP 27017
       │
       ▼
    SG-App
```

Meaning:

> Only the application tier is allowed to connect to DocumentDB on the database port.

Not:

```text
Internet → TCP 27017 → DocumentDB
```

---

# 10. Step 5 — Authentication

We create a database administrator/master account when provisioning the cluster.

But the application shouldn't casually use that administrative account.

Instead:

```text
Admin
  │
  ▼
Create application database user
  │
  ▼
Application
```

The application gets only the permissions it needs.

This is:

> **Least privilege.**

---

# 11. Step 6 — Secrets Manager

Don't do this:

```javascript
const username = "admin";
const password = "SuperSecretPassword";
```

Instead:

```text
Application
    │
    ▼
Secrets Manager
    │
    ▼
Database credentials
    │
    ▼
DocumentDB
```

The application retrieves the credentials securely.

---

# 12. Step 7 — Encryption

We need two types of encryption.

### In transit

```text
Application
     ║
     ║ TLS
     ║
DocumentDB
```

### At rest

```text
DocumentDB
     │
     ▼
Encrypted storage
     │
     ▼
KMS
```

Remember:

```text
TLS → moving data
KMS/encryption at rest → stored data
```

---

# 13. Step 8 — Connect the Application

The application needs the DocumentDB cluster endpoint.

Conceptually:

```text
Application
     │
     ▼
DocumentDB Cluster Endpoint
     │
     ▼
Cluster
```

Use the appropriate cluster endpoint rather than hard-coding a particular database instance.

This is important for failover.

---

# 14. Connection Pooling

Our application shouldn't create a new database connection for every HTTP request.

Bad:

```text
HTTP Request
    ↓
Create DB connection
    ↓
Query
    ↓
Close connection
```

Repeated thousands of times.

Better:

```text
Application
     │
     ▼
Connection Pool
 ┌───┼───┬───┐
 ▼   ▼   ▼   ▼
 DB  DB  DB  DB
```

The driver manages reusable connections.

---

# 15. Step 9 — Create Collections

We can create:

```javascript
db.createCollection("customers")
db.createCollection("products")
db.createCollection("orders")
```

Depending on the application, collections may also be created as documents are first inserted.

---

# 16. Step 10 — Insert a Customer

Example:

```javascript
db.customers.insertOne({
  customerId: "C1001",
  name: "John Smith",
  email: "john@example.com",
  address: {
    city: "New Orleans",
    state: "LA",
    country: "USA"
  },
  createdAt: new Date()
})
```

Result:

```text
Customer created
      ↓
C1001
```

---

# 17. Insert a Product

```javascript
db.products.insertOne({
  productId: "P1001",
  name: "Wireless Keyboard",
  category: "electronics",
  price: 59.99,
  inventory: 125,
  tags: [
    "keyboard",
    "wireless",
    "bluetooth"
  ],
  createdAt: new Date()
})
```

---

# 18. Insert an Order

```javascript
db.orders.insertOne({
  orderId: "O10001",
  customerId: "C1001",

  items: [
    {
      productId: "P1001",
      name: "Wireless Keyboard",
      quantity: 2,
      unitPrice: 59.99
    }
  ],

  total: 119.98,

  status: "PLACED",

  createdAt: new Date()
})
```

---

# 19. Step 11 — CRUD

Our application now needs CRUD.

### Create

```javascript
db.products.insertOne(...)
```

### Read

```javascript
db.products.find({
  category: "electronics"
})
```

### Update

```javascript
db.products.updateOne(
  { productId: "P1001" },
  { $set: { price: 64.99 } }
)
```

### Delete

```javascript
db.products.deleteOne({
  productId: "P1001"
})
```

You've already learned all of this in Lesson 3.

---

# 20. Step 12 — Design Queries First

This is where production data modeling becomes important.

Suppose the application frequently asks:

```text
Find customer by email
```

Then:

```javascript
db.customers.findOne({
  email: "john@example.com"
})
```

Should be fast.

That means:

> We should consider an index on `email`.

---

# 21. Customer Index

Conceptually:

```javascript
db.customers.createIndex({
  email: 1
})
```

Now:

```text
Query
  ↓
email index
  ↓
matching customer
```

instead of potentially scanning every document.

---

# 22. Product Indexes

Suppose we frequently query:

```javascript
db.products.find({
  category: "electronics"
})
```

An index could be:

```javascript
db.products.createIndex({
  category: 1
})
```

If we search products by tags, we may consider an appropriate multikey index.

Again:

> Index based on actual query patterns.

---

# 23. Order Index

One of our most important queries:

```text
Find all orders for a customer.
```

Query:

```javascript
db.orders.find({
  customerId: "C1001"
})
```

So we create:

```javascript
db.orders.createIndex({
  customerId: 1
})
```

Now:

```text
Customer
   │
   ▼
customerId index
   │
   ▼
Orders
```

---

# 24. Compound Index

Suppose the application frequently asks:

```text
Find a customer's recent orders.
```

Query:

```javascript
db.orders.find({
  customerId: "C1001",
  status: "PLACED"
})
```

and sorts by:

```javascript
{ createdAt: -1 }
```

We might consider a compound index aligned with this access pattern, such as:

```javascript
{
  customerId: 1,
  status: 1,
  createdAt: -1
}
```

The exact index should be validated against actual workload and query plans.

---

# 25. Don't Index Everything

A common beginner mistake:

```text
Every field
   ↓
Index it!
```

That's not a good strategy.

Indexes have costs:

```text
More storage
More write overhead
More maintenance
More memory pressure
```

Instead:

> **Create indexes for important query patterns.**

---

# 26. Step 13 — Query Performance

Suppose our application becomes slow.

First question:

```text
What query is slow?
```

Then:

```text
Does it have an appropriate index?
```

Then:

```text
How much data is being examined?
```

Then:

```text
Is the database under resource pressure?
```

Then:

```text
Can the query or data model be improved?
```

Only after that should we blindly increase infrastructure capacity.

---

# 27. Step 14 — Order Lifecycle

Our order starts:

```text
PLACED
```

Then:

```text
PLACED
   ↓
PROCESSING
   ↓
SHIPPED
   ↓
DELIVERED
```

If cancelled:

```text
PLACED
   ↓
CANCELLED
```

We can update:

```javascript
db.orders.updateOne(
  { orderId: "O10001" },
  {
    $set: {
      status: "SHIPPED"
    }
  }
)
```

---

# 28. Step 15 — Atomic Updates

Suppose inventory is:

```text
inventory = 125
```

A customer purchases:

```text
quantity = 2
```

We need to avoid:

```text
Read inventory
   ↓
Calculate 123
   ↓
Write 123
```

being unsafe under concurrent requests.

Instead, use an atomic update pattern appropriate to the application's requirements.

For example:

```javascript
db.products.updateOne(
  {
    productId: "P1001",
    inventory: { $gte: 2 }
  },
  {
    $inc: {
      inventory: -2
    }
  }
)
```

This expresses:

> Decrease inventory by 2 only if at least 2 units are available.

The application then checks whether the update matched a document.

---

# 29. Why This Matters

Imagine two customers simultaneously buy the last item.

Without proper concurrency handling:

```text
Customer A → sees inventory = 1
Customer B → sees inventory = 1

A → buys
B → buys
```

Now you've oversold.

Atomic update conditions help protect this operation.

---

# 30. Step 16 — Transactions

Sometimes an operation spans multiple documents and needs transactional consistency.

For example:

```text
Create order
+
Update inventory
```

Conceptually:

```text
BEGIN
   │
   ├── Create order
   │
   ├── Decrease inventory
   │
   ▼
COMMIT
```

If the transaction fails:

```text
ROLLBACK
```

However, don't automatically put every operation into a transaction.

Transactions have costs and should be used where the business operation genuinely requires multi-document atomicity.

---

# 31. Step 17 — Read Scaling

Suppose our application has:

```text
10,000 writes/day
1,000,000 reads/day
```

That's read-heavy.

Our architecture can use replicas:

```text
                DocumentDB
                    │
             ┌──────┴──────┐
             │             │
          Primary       Replicas
             │          ┌──┼──┐
             │          ▼  ▼  ▼
          Writes       R1 R2 R3
                         │
                         ▼
                    Read Traffic
```

The application can use the appropriate reader endpoint for read workloads.

---

# 32. Separate Read and Write Paths

Conceptually:

```text
                    Application
                   /           \
                  /             \
             Writes             Reads
                │                 │
                ▼                 ▼
             Primary        Reader Endpoint
                                  │
                              Replicas
```

This can help scale read-heavy workloads.

---

# 33. Step 18 — High Availability

Suppose:

```text
Primary
   │
   X
```

The cluster can fail over to an eligible replica.

Conceptually:

```text
Before:

Primary
Replica
Replica


Failure:

Primary ✗

Replica
   ↓
New Primary
```

The application should be designed to tolerate this event.

---

# 34. Step 19 — Backup Strategy

Our production system needs automated backups.

Conceptually:

```text
DocumentDB
     │
     ▼
Automated Backups
     │
     ├── Point-in-time recovery
     │
     └── Recovery protection
```

Before a major migration:

```text
DocumentDB
     │
     ▼
Manual Snapshot
     │
     ▼
Migration
```

---

# 35. Step 20 — Recovery Plan

Imagine the production database has a serious problem.

Our plan:

```text
Incident
   │
   ▼
Identify recovery point
   │
   ▼
Restore
   │
   ▼
Validate
   │
   ▼
Application testing
   │
   ▼
Redirect traffic
```

We should document this before the disaster happens.

---

# 36. Step 21 — Monitoring

Our DocumentDB cluster should be monitored through CloudWatch.

Monitor areas such as:

```text
CPU
Memory
Connections
Database load
Latency
Read/write activity
Storage
Replication health
Errors
```

Then establish useful alarms.

---

# 37. Example Monitoring Dashboard

Imagine:

```text
┌────────────────────────────────────────┐
│       DocumentDB Production             │
├────────────────────────────────────────┤
│ CPU              ███████░░░   70%       │
│ Connections      ████░░░░░░   Normal    │
│ Latency          ███░░░░░░░   Normal    │
│ DB Load          █████░░░░░   Moderate  │
│ Storage          ██████░░░░   60%       │
│ Replication      ✓ Healthy              │
│ Errors           ✓ Normal               │
└────────────────────────────────────────┘
```

The exact dashboard metrics should be chosen based on your workload.

---

# 38. Step 22 — Alerting

Suppose:

```text
CPU
   ↑
   │
   │          🚨
   │         /
   │________/
```

CloudWatch alarm:

```text
Threshold exceeded
      ↓
Alarm
      ↓
Notification
      ↓
On-call engineer
```

The goal is proactive detection.

---

# 39. Step 23 — Auditing

Someone changes the security group.

You need to know:

```text
Who?
When?
What changed?
```

AWS CloudTrail can help answer questions about AWS API activity.

Conceptually:

```text
AWS API Action
      │
      ▼
CloudTrail
      │
      ▼
Audit Record
```

---

# 40. Step 24 — Application Error Handling

Suppose DocumentDB temporarily becomes unavailable during failover.

Your application shouldn't respond:

```text
HTTP 500
Database exploded!!!
```

Instead, handle transient failures gracefully.

Conceptually:

```text
Database error
      │
      ▼
Is transient?
   /       \
 Yes       No
  │         │
Retry      Return
carefully  appropriate error
```

---

# 41. Step 25 — Retry Strategy

Use controlled retries.

```text
Attempt 1
    ↓
Failure
    ↓
Backoff
    ↓
Attempt 2
    ↓
Failure
    ↓
Backoff
    ↓
Attempt 3
```

Avoid infinite retry loops.

Otherwise:

```text
Database struggling
       ↑
       │
Application retries
       │
       ↑
More load
```

This is a:

> **Retry storm**

---

# 42. Step 26 — Security Review

Before production, verify:

```text
☑ Database is privately accessible
☑ Security groups are restricted
☑ No broad internet access
☑ TLS configured
☑ Encryption at rest enabled
☑ KMS strategy defined
☑ Credentials stored securely
☑ Least privilege applied
☑ IAM roles are restricted
☑ Secrets aren't in source control
```

---

# 43. Step 27 — Backup Review

Verify:

```text
☑ Automated backups enabled
☑ Retention meets requirements
☑ Manual snapshot strategy defined
☑ Restore procedure documented
☑ Restore tested
☑ RPO defined
☑ RTO defined
```

---

# 44. Step 28 — Performance Review

Verify:

```text
☑ Important queries identified
☑ Appropriate indexes created
☑ Query performance tested
☑ Connection pooling configured
☑ Read scaling considered
☑ Instance capacity appropriate
☑ Large documents reviewed
```

---

# 45. Step 29 — Failure Testing

Don't wait for production to fail.

Test:

```text
Application loses connection
        ↓
Can it reconnect?
```

Test:

```text
Primary failure
        ↓
Does application recover?
```

Test:

```text
Bad deployment
        ↓
Can we restore data?
```

Test:

```text
Security group mistake
        ↓
Can we identify it quickly?
```

This is how you turn an architecture diagram into a real production system.

---

# 46. Complete Production Architecture

Now let's put **all 10 lessons together**.

```text
                              USERS
                                │
                                ▼
                         ┌────────────┐
                         │   ALB/API  │
                         └─────┬──────┘
                               │
                               ▼
                    ┌────────────────────┐
                    │  Application Tier  │
                    │                    │
                    │ Connection Pool    │
                    │ Retry Logic        │
                    │ Business Logic     │
                    └─────────┬──────────┘
                              │
                ┌─────────────┴─────────────┐
                │                           │
                ▼                           ▼
           IAM Role                   Secrets Manager
                │                           │
                └─────────────┬─────────────┘
                              │
                              ▼
                    ┌─────────────────────┐
                    │         VPC         │
                    │                     │
                    │   Private Subnets   │
                    │                     │
                    │   ┌─────────────┐   │
                    │   │ DocumentDB  │   │
                    │   │   Cluster   │   │
                    │   └──────┬──────┘   │
                    │          │          │
                    │     ┌────┴────┐     │
                    │     ▼         ▼     │
                    │  Primary   Replicas │
                    │                     │
                    └─────────┬───────────┘
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
               CloudWatch           CloudTrail
               Monitoring             Audit
                    │
                    ▼
                 Backups
                    │
                    ▼
              Point-in-Time
                 Recovery
```

This is the architecture you should now be able to explain.

---

# 47. End-to-End Request

Let's follow a real request.

User clicks:

> **"View my orders."**

### Step 1

Request reaches the application.

```text
User
 ↓
Load Balancer
 ↓
Application
```

### Step 2

Application gets a database connection from its pool.

```text
Application
 ↓
Connection Pool
```

### Step 3

Connection reaches DocumentDB through private networking.

```text
Application
 ↓
VPC
 ↓
Security Group
 ↓
DocumentDB
```

### Step 4

TLS protects the connection.

```text
Application
 ║
 ║ TLS
 ║
DocumentDB
```

### Step 5

Database authenticates the application user.

```text
Credentials
 ↓
Authentication
```

### Step 6

Application executes:

```javascript
db.orders.find({
  customerId: "C1001"
})
```

### Step 7

DocumentDB uses:

```text
customerId index
```

to efficiently locate orders.

### Step 8

Results return to the application.

```text
DocumentDB
 ↓
Application
 ↓
Load Balancer
 ↓
User
```

That's the complete journey.

---

# 48. What Happens During Failure?

Now imagine the primary instance fails.

```text
Application
     │
     ▼
DocumentDB
     │
     X
Primary failure
```

DocumentDB performs the appropriate failover process.

Then:

```text
Replica
   │
   ▼
New Primary
```

Application:

```text
Connection fails
      │
      ▼
Driver/application reconnects
      │
      ▼
Cluster endpoint
      │
      ▼
New primary
```

This is why endpoint usage, connection management, and retry handling matter.

---

# 49. What Happens During an Accidental Delete?

Suppose:

```javascript
db.orders.deleteMany({})
```

😨

Our recovery strategy:

```text
Accidental deletion
       │
       ▼
Identify time
       │
       ▼
Choose recovery point
       │
       ▼
Restore
       │
       ▼
Validate
       │
       ▼
Recover application
```

The lesson:

> **Operations planning is part of database engineering.**

---

# 50. What Happens During a Performance Incident?

Suppose:

```text
API latency ↑
```

We investigate:

```text
CloudWatch
    │
    ├── CPU
    ├── Memory
    ├── Connections
    ├── Load
    └── Latency
```

Then:

```text
Slow query
    ↓
Check index
    ↓
Check query pattern
    ↓
Optimize
```

If resources remain insufficient:

```text
Evaluate scaling
```

This is a much better approach than:

```text
CPU high
 ↓
Immediately double instance size
```

---

# 51. Production Readiness Checklist

Before calling this system production-ready:

## Architecture

```text
☐ VPC designed
☐ Private subnets
☐ Multiple AZs
☐ Appropriate DocumentDB instances
☐ Reader capacity considered
```

## Security

```text
☐ Security groups restricted
☐ TLS configured
☐ Encryption at rest enabled
☐ KMS strategy
☐ Secrets Manager
☐ Least privilege
☐ IAM roles restricted
```

## Data

```text
☐ Data model designed around access patterns
☐ Document sizes reviewed
☐ Embedding/reference decisions documented
☐ Indexes created intentionally
```

## Application

```text
☐ Connection pooling
☐ Timeout configuration
☐ Retry strategy
☐ Failover handling
☐ Error handling
```

## Operations

```text
☐ Automated backups
☐ Snapshot strategy
☐ Restore tested
☐ RPO defined
☐ RTO defined
☐ CloudWatch monitoring
☐ CloudWatch alarms
☐ CloudTrail auditing
☐ Runbooks
```

---

# 52. The 10-Lesson Knowledge Map 🧠

You have now covered:

```text
LESSON 1
Fundamentals
    ↓
What is DocumentDB?


LESSON 2
Documents / Collections / BSON
    ↓
How is data represented?


LESSON 3
CRUD
    ↓
How do I manipulate documents?


LESSON 4
Queries / Operators
    ↓
How do I find the data?


LESSON 5
Indexes / Performance
    ↓
How do I make queries fast?


LESSON 6
Data Modeling
    ↓
How should I structure documents?


LESSON 7
AWS Architecture
    ↓
How does DocumentDB run on AWS?


LESSON 8
Security / Networking
    ↓
How do I protect it?


LESSON 9
Backup / Monitoring / Operations
    ↓
How do I operate it reliably?


LESSON 10
Production Project
    ↓
How do I put everything together?
```

---

# 53. The Ultimate DocumentDB Mental Model

If you remember only one diagram from this entire course, remember this:

```text
                       AMAZON DOCUMENTDB
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
      DATA                 QUERY                 AWS
        │                     │                     │
        ▼                     ▼                     ▼
 Documents               Operators              VPC
 Collections             Indexes                Subnets
 BSON                    Performance             Security Groups
        │                     │                  Instances
        ▼                     ▼                     │
    Modeling              Optimization              ▼
                                                Architecture
        │
        └─────────────────────┬─────────────────────┘
                              │
                              ▼
                          SECURITY
                              │
                     ┌────────┼────────┐
                     ▼        ▼        ▼
                    TLS      IAM    Secrets
                     │
                     ▼
                   KMS
                              │
                              ▼
                         OPERATIONS
                              │
                 ┌────────────┼────────────┐
                 ▼            ▼            ▼
              Backups      Monitoring    Failover
                 │            │            │
                 ▼            ▼            ▼
             Recovery      CloudWatch   Availability
```

---

# 🎯 Final Lesson — 10 Things You Must Be Able to Explain

### 1. Data Modeling

Why did you embed some data and reference other data?

### 2. Indexing

Why does a particular query need a particular index?

### 3. Networking

Why is DocumentDB inside private subnets?

### 4. Security Groups

Who is allowed to connect to port 27017?

### 5. Authentication

How does the application authenticate to the database?

### 6. Encryption

How are data in transit and data at rest protected?

### 7. High Availability

What happens when the primary fails?

### 8. Backups

How would you recover accidentally deleted data?

### 9. Monitoring

How would you know the database is becoming unhealthy?

### 10. Production Troubleshooting

If the application suddenly becomes slow, how would you systematically find the cause?

If you can answer all ten confidently, you have moved beyond simply knowing DocumentDB commands—you understand **how to design and operate Amazon DocumentDB in an AWS production environment**.

---

# 🏆 Amazon DocumentDB — Course Complete

```text
╔══════════════════════════════════════════════╗
║        AMAZON DOCUMENTDB — 10 LESSONS        ║
╠══════════════════════════════════════════════╣
║                                              ║
║  ✅ 2. Documents, Collections & BSON         ║
║  ✅ 3. CRUD Operations                       ║
║  ✅ 4. Queries & Operators                   ║
║  ✅ 5. Indexes & Query Performance           ║
║  ✅ 6. Data Modeling                         ║
║  ✅ 7. AWS Architecture                      ║
║  ✅ 8. Security & Networking                 ║
║  ✅ 9. Backup, Monitoring & Operations       ║
║  ✅ 10. Production Project                   ║
║                                              ║
╚══════════════════════════════════════════════╝
```

## 🧠 The one-line summary

> **Amazon DocumentDB is a managed, MongoDB-compatible document database on AWS; production success comes from combining good document modeling and indexing with private networking, strong security, high availability, backups, monitoring, and disciplined operations.**
