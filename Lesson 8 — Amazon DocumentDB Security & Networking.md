# Lesson 8 — Amazon DocumentDB Security & Networking 🔐

Welcome to **Lesson 8**.

At this point, you know how DocumentDB works internally and how its AWS architecture is organized. Now we're going to secure it.

```text
✅ Lesson 1 — DocumentDB Fundamentals
✅ Lesson 2 — Documents, Collections & BSON
✅ Lesson 3 — CRUD Operations
✅ Lesson 4 — Queries & Operators
✅ Lesson 5 — Indexes & Query Performance
✅ Lesson 6 — Data Modeling
✅ Lesson 7 — Architecture on AWS
➡️ Lesson 8 — Security & Networking
⬜ Lesson 9 — Backup, Monitoring & Operations
⬜ Lesson 10 — Production Project
```

The most important idea in this lesson is:

> **DocumentDB security has multiple layers: network access, authentication, authorization, encryption, secrets, and monitoring.**

---

# 1. The Security Picture

Think about a DocumentDB application like this:

```text
                         AWS
                          │
                    ┌─────┴─────┐
                    │    VPC    │
                    │           │
                    │ Application
                    │     │     │
                    │     │     │
                    │     ▼     │
                    │ Security  │
                    │  Group    │
                    │     │     │
                    │     ▼     │
                    │ DocumentDB│
                    │           │
                    └───────────┘
```

There are several questions:

```text
1. Who can reach DocumentDB?
2. Who can authenticate?
3. What can they do?
4. Is the traffic encrypted?
5. Is the stored data encrypted?
6. Where are credentials stored?
7. How do we monitor access?
```

Each question corresponds to a different security layer.

---

# 2. Security Layers

A useful mental model is:

```text
┌─────────────────────────────────┐
│ Monitoring / Auditing           │
├─────────────────────────────────┤
│ Authorization                   │
├─────────────────────────────────┤
│ Authentication                  │
├─────────────────────────────────┤
│ Encryption                      │
├─────────────────────────────────┤
│ Network Security                │
├─────────────────────────────────┤
│ AWS Account / IAM               │
└─────────────────────────────────┘
```

Don't treat these as interchangeable.

For example:

> A security group controls network connectivity.

It does **not** determine which database collection a user can query.

---

# 3. DocumentDB Runs Inside a VPC

Amazon DocumentDB clusters are deployed within an AWS:

> **Virtual Private Cloud (VPC)**

Think of the VPC as your private AWS network.

```text
AWS Account
     │
     ▼
    VPC
     │
 ┌───┴─────────────┐
 │                 │
 ▼                 ▼
Application      DocumentDB
Subnet           Subnet
```

This gives you network-level control over who can reach the database.

---

# 4. Public Internet vs Private Database

A good production architecture generally looks like:

```text
Internet
   │
   ▼
Load Balancer
   │
   ▼
Application
   │
   ▼
Private Network
   │
   ▼
DocumentDB
```

Not:

```text
Internet
   │
   ▼
DocumentDB
```

You generally don't want your database directly exposed to arbitrary internet traffic.

---

# 5. Public vs Private Subnets

A simplified AWS architecture:

```text
                    VPC
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
    Public Subnet          Private Subnet
          │                     │
    Load Balancer           Application
                                │
                                ▼
                           DocumentDB
```

The database should normally be isolated in private networking.

---

# 6. What Is a Subnet?

A subnet is a network segment inside a VPC.

For example:

```text
VPC: 10.0.0.0/16

Subnet A:
10.0.1.0/24

Subnet B:
10.0.2.0/24

Subnet C:
10.0.3.0/24
```

You can use different subnets for different application components.

---

# 7. Availability Zones + Subnets

Remember Lesson 7.

You can have:

```text
                 VPC
                  │
       ┌──────────┼──────────┐
       │          │          │
      AZ-A       AZ-B       AZ-C
       │          │          │
    Subnet A   Subnet B   Subnet C
       │          │          │
       ▼          ▼          ▼
   DocumentDB instances
```

Spreading infrastructure across AZs helps improve availability.

---

# 8. DB Subnet Group

DocumentDB uses a:

> **DB subnet group**

A DB subnet group identifies the subnets where DocumentDB instances can be deployed.

Conceptually:

```text
DB Subnet Group
      │
      ├── Subnet A
      ├── Subnet B
      └── Subnet C
```

For production, you generally want subnets spanning multiple Availability Zones.

---

# 9. Security Groups 🔥

One of the most important concepts.

A:

> **Security Group**

acts as a virtual firewall for AWS resources.

Conceptually:

```text
Application
     │
     │ TCP 27017
     ▼
┌──────────────────┐
│ DocumentDB SG    │
│                  │
│ Allow App SG     │
└──────────────────┘
```

The security group determines whether network traffic is allowed to reach the database.

---

# 10. Security Group Example

Suppose:

```text
Application SG:
sg-app

DocumentDB SG:
sg-docdb
```

You can configure the DocumentDB security group to allow:

```text
Source:
sg-app

Port:
27017

Protocol:
TCP
```

Conceptually:

```text
sg-app
  │
  │ TCP 27017
  ▼
sg-docdb
```

Now application instances associated with `sg-app` can connect to DocumentDB on the database port.

---

# 11. Don't Allow 0.0.0.0/0 🚨

You may see beginners configure:

```text
Source:
0.0.0.0/0

Port:
27017
```

This means:

> Allow traffic from essentially anywhere over IPv4.

That's a dangerous production configuration for a database.

Prefer:

```text
Application Security Group
          ↓
DocumentDB Security Group
```

rather than:

```text
Entire Internet
       ↓
DocumentDB
```

---

# 12. Security Groups Are Stateful

Security groups are stateful.

Conceptually:

```text
Application
    │
    │ Request
    ▼
DocumentDB
    │
    │ Response
    ▼
Application
```

Once the allowed connection is established, the return traffic is handled as part of that stateful connection.

You don't need to create an independent inbound rule simply for the response traffic.

---

# 13. Security Group vs Network ACL

AWS has multiple networking controls.

Two important ones are:

```text
Security Group
Network ACL
```

### Security Group

Associated with resources/interfaces.

```text
Resource-level firewall
```

### Network ACL

Associated with subnets.

```text
Subnet-level traffic control
```

For most DocumentDB connectivity troubleshooting, security groups are one of the first things you inspect.

---

# 14. Security Group Mental Model

Think:

```text
VPC
 │
 ├── Application
 │      │
 │      └── SG-App
 │
 └── DocumentDB
        │
        └── SG-DocDB
```

Rule:

```text
SG-DocDB:
Allow TCP 27017
from SG-App
```

This is much cleaner than allowing an IP range representing an entire network when you can use security-group-based access.

---

# 15. Authentication vs Authorization

This distinction is critical.

### Authentication

> **Who are you?**

Example:

```text
username = appuser
password = ********
```

### Authorization

> **What are you allowed to do?**

Example:

```text
appuser
   │
   ├── Read orders
   ├── Write orders
   └── Cannot drop certain collections
```

Think:

```text
Authentication
      ↓
WHO?

Authorization
      ↓
WHAT CAN THEY DO?
```

---

# 16. DocumentDB Authentication

DocumentDB supports MongoDB-compatible authentication mechanisms, including database users and password-based authentication.

At a high level:

```text
Application
    │
    │ username + password
    ▼
DocumentDB
    │
    ▼
Authenticate user
```

After authentication, database permissions determine what that user can do.

---

# 17. Master User

When you create a DocumentDB cluster, you configure a master username and password.

Conceptually:

```text
DocumentDB Cluster
      │
      └── Master user
```

This is an administrative database account.

Don't use the master account as your application's everyday account unless there's a specific reason.

---

# 18. Application Database User

A better production pattern is:

```text
Master/Admin
     │
     └── Creates application user
                   │
                   ▼
              Application
```

For example:

```text
app_orders
```

should ideally have only the permissions necessary for the application.

This is:

> **Least privilege**

---

# 19. Principle of Least Privilege

This is one of the most important security principles.

> **Give an identity only the permissions it actually needs.**

Bad:

```text
Application
    ↓
Administrator privileges
```

Better:

```text
Application
    ↓
Limited database user
    ↓
Only required operations
```

For example, if an application only needs to read product data, it shouldn't need administrative database privileges.

---

# 20. Authentication Does Not Equal Authorization

Imagine:

```text
User:
appuser
```

Authentication says:

```text
✓ Password is correct
```

That doesn't automatically mean:

```text
✓ Can perform every database operation
```

Authorization determines the allowed actions.

---

# 21. IAM Is Not the Same as Database Authentication

This is a common AWS interview topic.

AWS IAM controls access to:

```text
AWS resources
```

such as:

```text
DocumentDB cluster management
CloudWatch
Secrets Manager
KMS
EC2
VPC
```

Database authentication controls:

```text
Database connections
Database operations
```

Don't confuse the two.

---

# 22. Example: IAM Permission

An AWS IAM role might allow an application to:

```text
Describe DocumentDB clusters
Read a secret from Secrets Manager
Use a KMS key
```

That doesn't automatically mean the role can:

```text
db.orders.find(...)
```

inside DocumentDB.

Those are different authorization layers.

---

# 23. Secrets Management 🔑

Never hard-code database passwords in application source code.

Bad:

```javascript id="3r5xhi"
const password = "MySuperSecretPassword123";
```

Also bad:

```text id="9g0j5x"
Git repository
   │
   └── database password
```

This can lead to serious security problems.

---

# 24. AWS Secrets Manager

A common AWS pattern is:

```text id="wy1n6c"
                Secrets Manager
                     │
                     │ stores
                     ▼
             DB credentials
                     │
                     ▼
                Application
                     │
                     ▼
                DocumentDB
```

The application retrieves credentials securely rather than storing them in source code.

---

# 25. Application + Secrets Manager

Conceptually:

```text id="tqv8cj"
Application
     │
     │ IAM permission
     ▼
Secrets Manager
     │
     │ username/password
     ▼
Application
     │
     │ secure connection
     ▼
DocumentDB
```

IAM controls whether the application can retrieve the secret.

The database credentials authenticate the database connection.

Again:

```text id="l7q2u9"
IAM
  ≠
Database authentication
```

---

# 26. Encryption in Transit 🔒

When an application communicates with DocumentDB, the network traffic can be encrypted using:

> **TLS**

Conceptually:

```text
Application
     │
     │ Encrypted TLS connection
     ▼
DocumentDB
```

Without encryption:

```text
Application
     │
     │ readable traffic
     ▼
Database
```

That's undesirable for sensitive production workloads.

---

# 27. TLS

TLS protects data while it is traveling over the network.

For example:

```text
username
password
queries
documents
responses
```

can travel through an encrypted connection.

Mental model:

```text
Data at rest
      ↓
Storage encryption

Data in transit
      ↓
TLS
```

These solve different problems.

---

# 28. TLS Certificate

DocumentDB uses TLS certificates to establish secure connections.

Applications/drivers need to trust the appropriate AWS certificate authority chain.

Depending on the driver and environment, you may need to configure the CA certificate.

Conceptually:

```text
Application
    │
    │ Trust CA
    ▼
DocumentDB TLS
```

This helps prevent connecting securely to an untrusted endpoint.

---

# 29. Encryption at Rest

TLS protects:

```text
Data moving
```

Encryption at rest protects:

```text
Data stored
```

Conceptually:

```text
Application
    │
    │ TLS
    ▼
DocumentDB
    │
    │ encrypted storage
    ▼
Cluster Storage
```

So you want to understand both:

```text
In Transit → TLS

At Rest → Storage Encryption
```

---

# 30. AWS KMS

DocumentDB encryption at rest integrates with:

> **AWS Key Management Service (KMS)**

Conceptually:

```text
DocumentDB
     │
     ▼
KMS key
     │
     ▼
Encrypted data
```

You can use AWS-managed key options or customer-managed keys depending on your security requirements and configuration.

---

# 31. Customer-Managed KMS Keys

Organizations with stricter security requirements may use customer-managed KMS keys.

This gives you more control over:

* Key policies
* Key permissions
* Key lifecycle
* Auditing
* Key rotation configuration

Conceptually:

```text
Your Organization
       │
       ▼
Customer KMS Key
       │
       ▼
DocumentDB Encryption
```

---

# 32. Encryption at Rest vs In Transit

Memorize this:

| Security concern         | Technology                   |
| ------------------------ | ---------------------------- |
| Data moving over network | **TLS**                      |
| Data stored in cluster   | **Encryption at rest / KMS** |
| AWS resource permissions | **IAM**                      |
| Database login           | **Database authentication**  |
| Network connectivity     | **Security Groups / VPC**    |

This table is extremely useful for interviews.

---

# 33. Defense in Depth

A secure DocumentDB architecture shouldn't rely on one security mechanism.

Instead:

```text id="czx6u7"
Internet
   │
   X
   │
VPC
   │
   ▼
Private Subnet
   │
   ▼
Security Group
   │
   ▼
TLS
   │
   ▼
Authentication
   │
   ▼
Authorization
   │
   ▼
Encrypted Storage
```

If one layer is misconfigured, other layers can still provide protection.

This is:

> **Defense in depth**

---

# 34. Example Secure Architecture

Here's a good production-style conceptual architecture:

```text id="8bq8g6"
                         INTERNET
                            │
                            ▼
                     Load Balancer
                            │
                            ▼
                  Application Servers
                            │
                       SG-App
                            │
                     TCP 27017
                            │
                            ▼
                  ┌───────────────────┐
                  │      VPC          │
                  │                   │
                  │  Private Subnets  │
                  │       │           │
                  │       ▼           │
                  │   DocumentDB      │
                  │   SG-DocDB        │
                  │       │           │
                  │       ▼           │
                  │   Encrypted       │
                  │   Cluster Storage │
                  └───────────────────┘

             Secrets Manager
                    │
                    ▼
             DB Credentials
```

---

# 35. Network Flow

Let's follow one request.

### Step 1

User sends request:

```text id="phl6hx"
User
 ↓
Application
```

### Step 2

Application needs data:

```text id="r25m4u"
Application
 ↓
DocumentDB
```

### Step 3

Security group checks network connectivity.

```text id="k9u6cx"
SG-App
    ↓
SG-DocDB
    ↓
TCP 27017 allowed?
```

### Step 4

Application establishes TLS.

```text id="8jlykr"
Application
    ║
    ║ TLS
    ║
DocumentDB
```

### Step 5

Database authenticates the user.

```text id="d5q6ij"
Credentials
    ↓
Authentication
```

### Step 6

Database checks permissions.

```text id="x2p2ob"
Authorization
    ↓
Allowed?
```

### Step 7

Query executes.

```text id="00f7my"
db.orders.find(...)
```

That's the security journey.

---

# 36. Network Troubleshooting

Suppose your application says:

```text id="vcyt6m"
Connection timed out
```

Don't immediately assume the username/password is wrong.

A timeout often points toward:

```text id="jv5x8x"
VPC
Subnet
Route
Security Group
Network connectivity
Port
```

Whereas an authentication error is more likely:

```text id="h5u5b0"
Username
Password
Database authentication
TLS configuration
```

Understanding the layers makes troubleshooting much easier.

---

# 37. Timeout vs Authentication Error

### Timeout

```text
Application
    │
    X
    │
DocumentDB
```

Potential causes:

```text
Security group
Routing
Subnet
DNS
Network path
Wrong endpoint/port
```

### Authentication failure

```text
Application
    │
    ▼
DocumentDB
    │
    X
    │
Invalid credentials
```

Potential causes:

```text
Wrong username
Wrong password
Wrong authentication configuration
```

---

# 38. DNS

DocumentDB endpoints use DNS names.

Conceptually:

```text
docdb-cluster.example
       │
       ▼
      DNS
       │
       ▼
IP address
       │
       ▼
DocumentDB
```

Your application generally uses the DocumentDB endpoint rather than manually managing database IP addresses.

This is especially important because cluster topology can change during failover.

---

# 39. Why DNS + Endpoints Matter

Suppose:

```text
Primary A
```

fails.

Then:

```text
Replica B
   ↓
New Primary
```

The cluster endpoint can continue representing the cluster's current primary.

That's another reason not to hard-code infrastructure details into application code.

---

# 40. Secrets Rotation

Passwords shouldn't necessarily live forever.

A mature architecture considers:

```text
Credential
    │
    ▼
Rotation
    │
    ▼
New credential
```

AWS Secrets Manager can be part of a broader credential-management strategy.

The important principle is:

> **Don't make long-lived credentials impossible to change.**

---

# 41. Least Privilege Example

Suppose you have three applications:

```text
Orders API
Payments API
Reporting API
```

Don't necessarily give all three the same database account.

Conceptually:

```text
Orders API
   ↓
orders_user

Payments API
   ↓
payments_user

Reporting API
   ↓
reporting_user
```

Each should have only the permissions it requires.

This limits the blast radius if one application is compromised.

---

# 42. Separate Environments

Another good practice:

```text
Development
    ↓
Development DocumentDB

Testing
    ↓
Testing DocumentDB

Production
    ↓
Production DocumentDB
```

Don't have:

```text
Developer laptop
      ↓
Production DocumentDB
```

unless there's a very deliberate, tightly controlled reason.

Environment isolation reduces accidental production changes.

---

# 43. Developer Access

A common production architecture is:

```text
Developer
    │
    ▼
VPN / Bastion / Approved access path
    │
    ▼
Private VPC
    │
    ▼
DocumentDB
```

rather than:

```text
Developer
    │
    ▼
Internet
    │
    ▼
Public DocumentDB
```

Again:

> **Private by default.**

---

# 44. Don't Put Credentials in Git

This deserves repeating.

Never commit:

```text id="xg4t4v"
username
password
connection string
private key
secret token
```

to source control.

A connection string such as:

```text
mongodb://admin:SuperSecretPassword@...
```

inside Git is a security incident waiting to happen.

Use a secrets-management solution instead.

---

# 45. Connection String Security

Applications often use a connection string conceptually like:

```text
mongodb://username:password@documentdb-endpoint:27017/database
```

Treat the entire connection string as sensitive if it contains credentials.

Don't put it into:

```text
Git
Logs
Error messages
Screenshots
Tickets
```

---

# 46. Logging Security

Be careful with application logging.

Bad:

```text
Connecting to:
mongodb://admin:Password123@docdb...
```

That exposes credentials in logs.

Better:

```text
Connecting to DocumentDB cluster
```

without logging secrets.

---

# 47. Security Monitoring

Security doesn't stop after configuration.

You should monitor:

```text
Connection failures
Authentication failures
Network changes
Configuration changes
Unexpected activity
Database performance
AWS API activity
```

AWS services such as CloudWatch and CloudTrail are important parts of the broader AWS monitoring/auditing ecosystem.

We'll go deeper into monitoring in Lesson 9.

---

# 48. CloudTrail vs CloudWatch

Another important AWS distinction.

### CloudTrail

Primarily concerned with:

> **AWS API activity and account-level auditing**

For example:

```text
Who changed the DocumentDB cluster?
Who modified a security group?
Who changed an AWS resource?
```

### CloudWatch

Primarily concerned with:

> **Metrics, logs, alarms, and operational monitoring**

For example:

```text
CPU
Connections
Database performance
Alarms
Logs
```

Think:

```text
CloudTrail → "Who did what in AWS?"

CloudWatch → "How is the system behaving?"
```

---

# 49. Security Architecture Checklist

For a production DocumentDB deployment, think through:

```text
☐ Private VPC placement
☐ Private subnets
☐ Multiple Availability Zones
☐ Restricted security groups
☐ No unnecessary public access
☐ TLS enabled
☐ Encryption at rest
☐ KMS strategy
☐ Strong database credentials
☐ Secrets Manager
☐ Least-privilege database users
☐ Least-privilege IAM roles
☐ Monitoring
☐ Auditing
☐ Credential rotation strategy
```

---

# 50. A Complete Secure Request

Let's put everything together.

```text
                     USER
                       │
                       ▼
                Application
                       │
                       │
                  IAM Role
                       │
                       ▼
              Secrets Manager
                       │
                  Credentials
                       │
                       ▼
             ┌─────────────────┐
             │      VPC        │
             │                 │
             │  Private Subnet │
             │        │        │
             │        ▼        │
             │   Security      │
             │    Group        │
             │        │        │
             │        ▼        │
             │   DocumentDB    │
             │        │        │
             │       TLS       │
             │        │        │
             │        ▼        │
             │ Authentication  │
             │        │        │
             │ Authorization   │
             │        │        │
             │        ▼        │
             │ Encrypted Data  │
             └─────────────────┘
```

This is the mental architecture I want you to remember.

---

# 51. Interview Question: How would you secure DocumentDB?

A strong answer:

> **I would deploy DocumentDB inside private subnets in a VPC, restrict inbound access with security groups so only the application tier can reach the database port, use TLS for encryption in transit, enable encryption at rest with KMS, store credentials in Secrets Manager rather than source code, use least-privilege database users and IAM roles, and monitor and audit the environment using appropriate AWS services.**

That's a strong interview answer.

---

# 52. Interview Question: What does a security group do?

> **A security group controls network traffic to the DocumentDB instances. For example, I can allow TCP port 27017 from the application's security group while denying other sources.**

---

# 53. Interview Question: Security Group vs IAM?

> **Security groups control network connectivity, while IAM controls permissions to AWS resources and APIs. They operate at different security layers.**

---

# 54. Interview Question: TLS vs Encryption at Rest?

> **TLS protects data while it is transmitted between the application and DocumentDB. Encryption at rest protects data stored by DocumentDB.**

---

# 55. Interview Question: Why use Secrets Manager?

> **Secrets Manager allows applications to retrieve database credentials securely without hard-coding passwords into source code, configuration files, or deployment artifacts. It also supports centralized secret management and rotation strategies.**

---

# 56. Interview Question: What is least privilege?

> **Least privilege means giving each user, application, or AWS role only the permissions required to perform its intended tasks and nothing more.**

---

# 57. The 8-Layer Security Mental Model 🧠

Memorize this:

```text
                 DOCUMENTDB SECURITY
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
      NETWORK       AUTHENTICATION   AUTHORIZATION
        │                │                │
        ▼                ▼                ▼
     VPC/SG          DB Credentials    DB Roles
        │
        └────────────────┐
                         ▼
                    ENCRYPTION
                   ┌──────┴──────┐
                   ▼             ▼
                 TLS            KMS
              In Transit      At Rest
                   │             │
                   └──────┬──────┘
                          ▼
                     SECRETS
                          │
                          ▼
                    MONITORING
                          │
                          ▼
                      AUDITING
```

---

# 🎯 Lesson 8 — The 10 Things You Must Remember

### 1.

**DocumentDB should generally be deployed inside a VPC.**

### 2.

Use **private networking** for the database.

### 3.

Use **security groups** to restrict who can connect.

### 4.

Avoid exposing database port **27017** broadly to the internet.

### 5.

**Authentication = Who are you?**

### 6.

**Authorization = What are you allowed to do?**

### 7.

Use **TLS** to protect data in transit.

### 8.

Use **encryption at rest/KMS** to protect stored data.

### 9.

Use **Secrets Manager** instead of hard-coding database credentials.

### 10.

Apply **least privilege** at both the AWS and database levels.

---

# 📝 Lesson 8 Mini Quiz

Try these without looking back:

### 1.

Where does Amazon DocumentDB run from a networking perspective?

### 2.

What is the purpose of a DB subnet group?

### 3.

What does a security group control?

### 4.

Why is allowing `0.0.0.0/0` to port 27017 dangerous?

### 5.

What is the difference between authentication and authorization?

### 6.

What does TLS protect?

### 7.

What does encryption at rest protect?

### 8.

What AWS service can you use to securely store database credentials?

### 9.

What is the difference between IAM and a database user?

### 10.

What does the principle of least privilege mean?

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
⬜ Lesson 9 — Backup, Monitoring & Operations
⬜ Lesson 10 — Production Project
```

## Next: **Lesson 9 — Amazon DocumentDB Backup, Monitoring & Operations 📊**

We'll cover **automated backups, snapshots, point-in-time recovery, restore operations, CloudWatch metrics, alarms, CloudTrail, performance monitoring, logs, maintenance, failover testing, scaling, and day-to-day DocumentDB administration**.
