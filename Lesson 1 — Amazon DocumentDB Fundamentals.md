# Lesson 1 — Amazon DocumentDB Fundamentals

We’ll keep this lesson **strictly focused on Amazon DocumentDB**.

By the end, you should be able to explain:

* What Amazon DocumentDB is
* Why it exists
* How a document database works
* DocumentDB’s MongoDB compatibility
* The basic DocumentDB architecture
* Cluster vs instance
* Primary vs replica
* Collections and documents
* Endpoints
* Where DocumentDB fits in AWS

---

## 1. What is Amazon DocumentDB?

**Amazon DocumentDB** is a **fully managed document database service from AWS**.

It is designed for applications that store data as **JSON-like documents** rather than rows and columns.

A simple document might look like:

```json
{
  "_id": 101,
  "name": "John",
  "email": "john@example.com",
  "age": 30,
  "address": {
    "city": "New Orleans",
    "state": "Louisiana"
  },
  "skills": ["AWS", "Python", "MongoDB"]
}
```

Instead of thinking:

```text
Database
   ↓
Table
   ↓
Row
   ↓
Column
```

you generally think:

```text
DocumentDB
   ↓
Database
   ↓
Collection
   ↓
Document
   ↓
Field
```

That's the first major concept to remember.

---

# 2. Why use a document database?

Traditional relational databases are excellent when your data naturally fits tables.

For example:

### Relational model

**CUSTOMER**

| customer_id | name | email                                       | age |
| ----------- | ---- | ------------------------------------------- | --: |
| 101         | John | [john@example.com](mailto:john@example.com) |  30 |

But modern applications often have data that is more naturally represented as nested objects.

For example:

```json
{
  "customerId": 101,
  "name": "John",
  "contact": {
    "email": "john@example.com",
    "phone": "504-555-1234"
  },
  "orders": [
    {
      "orderId": 5001,
      "amount": 150
    },
    {
      "orderId": 5002,
      "amount": 75
    }
  ]
}
```

The document model lets related information exist naturally inside the document.

This can be particularly useful for applications such as:

* Product catalogs
* User profiles
* Content management
* Mobile/web applications
* Gaming applications
* Real-time application data
* Applications with rapidly changing document structures

---

# 3. DocumentDB and MongoDB

This is **VERY important**.

Amazon DocumentDB is designed to provide **MongoDB compatibility through MongoDB APIs and drivers**.

That means developers familiar with MongoDB can use many familiar concepts and tools when working with DocumentDB.

For example, you'll encounter concepts such as:

```text
MongoDB API
   ↓
Database
   ↓
Collection
   ↓
Document
```

and commands such as:

```javascript
db.users.find()
```

However:

> **Amazon DocumentDB is NOT MongoDB.**

This distinction is important.

DocumentDB is an AWS-managed database service with its **own implementation and capabilities**, even though it supports a MongoDB-compatible API.

Therefore, you should never assume:

> "If MongoDB supports X, DocumentDB must support X."

When working professionally with DocumentDB, always check the current AWS compatibility documentation for the specific MongoDB feature you need.

---

# 4. Document database vs relational database

Let's compare the mental models.

### Relational database

```text
Database
│
├── Customers
│    ├── Row
│    ├── Row
│    └── Row
│
├── Orders
│    ├── Row
│    ├── Row
│    └── Row
│
└── Products
     ├── Row
     ├── Row
     └── Row
```

Data is organized into tables.

### Document database

```text
Database
│
├── customers
│    ├── Document
│    ├── Document
│    └── Document
│
├── orders
│    ├── Document
│    ├── Document
│    └── Document
│
└── products
     ├── Document
     ├── Document
     └── Document
```

The individual records are documents.

---

# 5. What is a document?

A **document** is the fundamental unit of data.

Example:

```json
{
  "_id": "C1001",
  "name": "Alice",
  "email": "alice@example.com",
  "age": 28
}
```

Think of this as one customer record.

Another document could be:

```json
{
  "_id": "C1002",
  "name": "Bob",
  "email": "bob@example.com",
  "age": 35
}
```

Notice something important:

The documents don't necessarily need to have exactly the same structure.

For example:

```json
{
  "_id": "C1003",
  "name": "Charlie",
  "email": "charlie@example.com",
  "premiumCustomer": true
}
```

This flexibility is one of the major characteristics of document databases.

---

# 6. What is a collection?

A **collection** is a group of documents.

Think of it roughly like a table in a relational database, although the behavior and data-modeling approach are different.

For example:

```text
Database: ecommerce

Collection: customers
│
├── Document 1
├── Document 2
├── Document 3
└── Document 4
```

Another collection:

```text
Database: ecommerce

Collection: products
│
├── Document 1
├── Document 2
└── Document 3
```

So:

```text
Database
   │
   ├── Collection
   │      ├── Document
   │      ├── Document
   │      └── Document
   │
   └── Collection
          ├── Document
          └── Document
```

---

# 7. BSON — what you need to know

DocumentDB works with **BSON-style document data**.

BSON means:

> **Binary JSON**

It is designed to represent JSON-like documents while supporting additional data types.

For example:

```json
{
  "name": "Alice",
  "age": 28,
  "active": true
}
```

A document can also contain nested structures:

```json
{
  "name": "Alice",
  "address": {
    "city": "New Orleans",
    "state": "LA"
  }
}
```

And arrays:

```json
{
  "name": "Alice",
  "skills": [
    "AWS",
    "Python",
    "DocumentDB"
  ]
}
```

You'll work extensively with:

* Strings
* Numbers
* Boolean values
* Arrays
* Embedded/nested documents
* Dates
* Object IDs
* Null values
* Other BSON-supported types

We'll go much deeper into this in **Lesson 2**.

---

# 8. The `_id` field

You'll frequently see:

```json
{
  "_id": 101,
  "name": "John"
}
```

`_id` identifies a document within a collection.

For example:

```text
customers

_id       name
----------------
101       John
102       Alice
103       Bob
```

A document normally has an `_id`.

When you insert documents without explicitly specifying one, the client/database ecosystem can generate an identifier such as an ObjectId.

For example:

```json
{
  "_id": ObjectId("..."),
  "name": "John"
}
```

Don't worry about ObjectId details yet. We'll cover those in Lesson 2.

---

# 9. The BIG AWS concept: DocumentDB Cluster

Now let's move from the **data model** to the **AWS architecture**.

This is one of the most important concepts for AWS interviews.

Amazon DocumentDB uses a **cluster architecture**.

Conceptually:

```text
                DocumentDB Cluster
                       │
          ┌────────────┴────────────┐
          │                         │
      Primary                    Replicas
       Instance                 Instance
          │                         │
          └────────────┬────────────┘
                       │
                 Cluster Storage
```

The cluster contains database instances and a distributed storage layer.

---

# 10. Primary instance

The **primary instance** is responsible for handling writes.

Conceptually:

```text
Application
     │
     │ WRITE
     ▼
 Primary Instance
     │
     ▼
 DocumentDB Storage
```

For example:

```text
Application
     │
     │ insert customer
     ▼
Primary
     │
     ▼
Cluster storage
```

The primary is the main instance responsible for database operations that modify data.

---

# 11. Replica instances

A DocumentDB cluster can also have **replica instances**.

Conceptually:

```text
                 Cluster
                    │
             ┌──────┴──────┐
             │             │
          Primary       Replica
             │             │
             └──────┬──────┘
                    │
              Shared storage
```

Replicas are particularly useful for:

* Read scaling
* High availability
* Failover

For example:

```text
                    Application
                   /           \
              WRITE             READ
                │                 │
                ▼                 ▼
             Primary          Replica
```

This gives you an important architectural pattern:

> **Primary → writes**
> **Replicas → reads**

There are nuances around endpoints and read routing that we'll cover later.

---

# 12. Storage architecture

One important characteristic of DocumentDB is that the database compute instances and underlying storage are architecturally separated.

Conceptually:

```text
             DocumentDB Cluster
                    │
        ┌───────────┼───────────┐
        │           │           │
     Primary     Replica     Replica
        │           │           │
        └───────────┼───────────┘
                    │
             Distributed
              cluster storage
```

This architecture helps AWS provide capabilities such as:

* Replication
* High availability
* Failover
* Storage scalability
* Multiple compute instances using the cluster storage

This is a major architecture concept you should understand for interviews.

---

# 13. Cluster vs Instance

This is a common interview question.

### Instance

An **instance** provides the compute resources.

Think:

```text
Instance
├── CPU
├── Memory
└── Database engine process
```

### Cluster

The **cluster** is the larger database environment containing:

```text
Cluster
├── Primary instance
├── Replica instance
├── Replica instance
└── Cluster storage
```

So:

> **Instance = compute**

> **Cluster = database environment containing instances + storage architecture**

---

# 14. What is an endpoint?

Your application needs a way to connect to DocumentDB.

It doesn't normally connect to:

```text
Primary instance IP address
```

Instead, you use a **DocumentDB endpoint**.

Conceptually:

```text
Application
      │
      │ connection
      ▼
 DocumentDB Endpoint
      │
      ▼
 DocumentDB Cluster
```

There are different endpoint types designed for different purposes.

The major concepts you'll encounter are:

### Cluster endpoint

Used for connecting to the cluster and generally routes connections to the primary instance.

### Reader endpoint

Used for read traffic and can distribute connections among available replicas.

### Instance endpoint

Points to a specific instance.

We'll cover endpoints in much greater detail when we study DocumentDB architecture.

---

# 15. Availability Zones

DocumentDB is designed to operate within an AWS Region and can use multiple **Availability Zones (AZs)**.

Conceptually:

```text
AWS Region
│
├── AZ-1
│    └── DocumentDB instance
│
├── AZ-2
│    └── DocumentDB instance
│
└── AZ-3
     └── DocumentDB instance
```

Why?

Because if one Availability Zone experiences a problem, the architecture can support continued availability through other infrastructure.

For production workloads, distributing instances across AZs is an important high-availability consideration.

---

# 16. Failover

Imagine:

```text
Primary
   │
   X   ← failure
```

A replica can potentially become the new primary.

Conceptually:

```text
Before:

Primary ──────── Replica
   │
   └── Writes


After failure:

Failed Primary     Replica
                      │
                      ▼
                  New Primary
```

This is called **failover**.

The application should be designed to connect using appropriate cluster endpoints rather than hard-coding an individual instance endpoint.

That allows the infrastructure to handle topology changes more gracefully.

---

# 17. Where does DocumentDB live?

DocumentDB is an **AWS managed service**.

You don't normally install and maintain the database engine yourself on EC2.

Instead:

```text
You
 │
 ├── Create cluster
 ├── Configure networking
 ├── Configure security
 ├── Choose instance types
 └── Manage database
        │
        ▼
       AWS
        │
        ├── Infrastructure
        ├── Storage
        ├── Maintenance
        └── Managed database operations
```

AWS manages much of the underlying infrastructure.

But **you are still responsible for things such as**:

* Data modeling
* Indexes
* Query design
* Security configuration
* Network configuration
* Instance sizing
* Monitoring
* Application configuration

Managed doesn't mean "AWS manages everything."

---

# 18. DocumentDB networking

This is another critical AWS concept.

DocumentDB clusters are deployed within an **Amazon VPC**.

Conceptually:

```text
                    AWS Region
                       │
                      VPC
                       │
             ┌─────────┴─────────┐
             │                   │
          Subnet              Subnet
             │                   │
        DocumentDB           DocumentDB
        Instance             Instance
```

Your application might be running on:

* EC2
* ECS
* EKS
* Lambda in a VPC
* Another AWS service
* An application running elsewhere with appropriate network connectivity

The application needs network connectivity to the DocumentDB cluster.

We'll study VPCs, subnets, security groups, TLS, and authentication in **Lesson 8**.

---

# 19. A simple real-world example

Imagine we're building an online store.

We could have:

```text
DocumentDB
│
└── ecommerce database
       │
       ├── customers
       │
       ├── products
       │
       ├── orders
       │
       └── reviews
```

A product document might look like:

```json
{
  "_id": "P1001",
  "name": "Laptop",
  "brand": "ExampleTech",
  "price": 999,
  "categories": [
    "electronics",
    "computers"
  ],
  "specifications": {
    "memory": "16GB",
    "storage": "1TB"
  }
}
```

Notice how naturally the product's nested information fits into a document.

---

# 20. DocumentDB mental model

You should now have this picture in your head:

```text
                    AWS
                     │
                    VPC
                     │
             DocumentDB Cluster
                     │
          ┌──────────┴──────────┐
          │                     │
       Primary               Replicas
          │                     │
          └──────────┬──────────┘
                     │
              Cluster Storage
                     │
              ┌──────┴──────┐
              │             │
          Database       Database
              │
        ┌─────┼─────┐
        │     │     │
     users orders products
        │     │     │
    Documents Documents Documents
```

And at the application level:

```text
Application
     │
     ▼
DocumentDB Endpoint
     │
     ▼
DocumentDB Cluster
     │
     ├── Write → Primary
     │
     └── Read  → Appropriate read path
```

---

# 21. Key terminology to memorize

| Term                | Meaning                                                                  |
| ------------------- | ------------------------------------------------------------------------ |
| **DocumentDB**      | AWS managed document database service                                    |
| **Document**        | Individual JSON/BSON-style data record                                   |
| **Collection**      | Group of documents                                                       |
| **Database**        | Logical container for collections                                        |
| **Cluster**         | DocumentDB database environment containing instances and cluster storage |
| **Instance**        | Compute resource running the database engine                             |
| **Primary**         | Instance responsible for primary/write operations                        |
| **Replica**         | Additional instance used primarily for reads and availability            |
| **Endpoint**        | DNS connection point used by applications                                |
| **Reader endpoint** | Endpoint intended for read connections                                   |
| **BSON**            | Binary representation of JSON-like documents                             |
| **`_id`**           | Identifier for a document                                                |
| **VPC**             | AWS network in which DocumentDB is deployed                              |
| **AZ**              | Availability Zone                                                        |

---

# 22. What DocumentDB is NOT

These distinctions are important.

### ❌ DocumentDB isn't Aurora

Aurora is a relational database service.

DocumentDB is a document database service.

### ❌ DocumentDB isn't DynamoDB

Both can be used for NoSQL workloads, but their data models, APIs, architectures, and use cases are different.

### ❌ DocumentDB isn't MongoDB

DocumentDB provides MongoDB compatibility, but it is an AWS-managed service with its own implementation and feature set.

### ❌ DocumentDB isn't an EC2 database you install yourself

AWS manages the underlying database infrastructure.

---

# 23. Interview Questions

### Q1. What is Amazon DocumentDB?

**Answer:**

Amazon DocumentDB is a fully managed AWS document database service designed to provide MongoDB-compatible APIs and drivers for applications that store JSON-like document data.

---

### Q2. What is the basic data hierarchy?

Think:

```text
Database
   ↓
Collection
   ↓
Document
   ↓
Fields
```

---

### Q3. What is a DocumentDB cluster?

A cluster is the overall DocumentDB database environment consisting of database instances and the underlying cluster storage architecture.

---

### Q4. What is the difference between a primary and a replica?

The **primary** handles primary/write operations, while **replicas** can serve read traffic and provide additional availability/failover capacity.

---

### Q5. Is Amazon DocumentDB MongoDB?

**No.**

DocumentDB supports a MongoDB-compatible API, but DocumentDB and MongoDB are separate database products with different implementations and supported features.

---

### Q6. Where is DocumentDB deployed?

Inside an **AWS VPC**, across the configured subnets and Availability Zones.

---

### Q7. Why use a reader endpoint?

To provide an endpoint intended for read connections and allow the application to use available replicas for read workloads.

---

# 🧠 Lesson 1 Quiz

Try answering these **without looking above**.

### 1.

What is Amazon DocumentDB?

### 2.

What is the difference between a **collection** and a **document**?

### 3.

What is the difference between an **instance** and a **cluster**?

### 4.

What is the role of the **primary instance**?

### 5.

Why would you use **replica instances**?

### 6.

Is DocumentDB the same database engine as MongoDB?

### 7.

What does `_id` represent?

### 8.

Where is a DocumentDB cluster deployed from a networking perspective?

### 9.

What is the purpose of a DocumentDB endpoint?

### 10.

Complete this:

```text
Database
    ↓
    ?
    ↓
    ?
```

---

## 🎯 Lesson 1 Takeaway

If you remember only **one diagram** from today's lesson, remember this:

```text
             Amazon DocumentDB
                     │
                  Cluster
                     │
          ┌──────────┴──────────┐
          │                     │
       Primary               Replicas
          │                     │
          └──────────┬──────────┘
                     │
              Cluster Storage
                     │
                  Database
                     │
                Collections
                     │
                 Documents
                     │
                   Fields
```

And the most important distinction:

> **DocumentDB is an AWS-managed, MongoDB-compatible document database service—not MongoDB itself.**

### Next lesson

**Lesson 2 — Documents, Collections & BSON**

We'll get hands-on with the actual data model: **JSON/BSON, `_id`, ObjectId, data types, nested documents, arrays, collections, and inserting your first documents.**
