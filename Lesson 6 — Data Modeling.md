# Lesson 6 — Amazon DocumentDB Data Modeling 🏗️

Welcome to **Lesson 6**.

So far:

1. ✅ DocumentDB Fundamentals
2. ✅ Documents, Collections & BSON
3. ✅ CRUD Operations
4. ✅ Queries & Operators
5. ✅ Indexes & Query Performance
6. **➡️ Data Modeling**

This lesson is where DocumentDB starts to feel very different from a traditional relational database.

The most important idea is:

> **In DocumentDB, you design your data model around how your application reads and writes data.**

---

# 1. What Is Data Modeling?

Data modeling means deciding:

* What documents should look like
* What fields they contain
* Which data belongs together
* What should be embedded
* What should be stored separately
* How collections should be organized
* How your application will access the data

For example, a customer might look like:

```json
{
  "_id": "C1001",
  "name": "John",
  "email": "john@example.com",
  "address": {
    "city": "New Orleans",
    "state": "LA"
  }
}
```

The question is:

> **What else should be inside this document?**

Should orders be embedded?

Should addresses be embedded?

Should payments be separate?

That's data modeling.

---

# 2. SQL vs DocumentDB Thinking

With a relational database, you often start by thinking:

```text
Tables
 ↓
Rows
 ↓
Relationships
 ↓
JOINs
```

For example:

```text
customers
orders
order_items
products
payments
```

You might join them together when you need information.

With DocumentDB, you should often start with:

```text
Application
     ↓
Access pattern
     ↓
Document structure
```

Instead of asking:

> "How do I normalize this data?"

ask:

> **"What data does my application usually need together?"**

That's a major mindset shift.

---

# 3. The Golden Rule

Remember this:

> **Model for your application's access patterns.**

Suppose an application constantly asks:

> "Give me a customer and their recent orders."

You might consider putting some order information directly inside the customer document.

Instead of:

```text
customers
orders
```

and requiring multiple operations, you might have:

```json
{
  "_id": "C1001",
  "name": "John",
  "email": "john@example.com",
  "orders": [
    {
      "orderId": "O1001",
      "total": 150
    },
    {
      "orderId": "O1002",
      "total": 75
    }
  ]
}
```

Now one document contains data frequently accessed together.

---

# 4. Embedding vs Referencing

There are two fundamental modeling approaches.

## Embedding

Put related data inside the same document.

```json
{
  "_id": "C1001",
  "name": "John",
  "address": {
    "city": "New Orleans",
    "state": "LA"
  }
}
```

## Referencing

Keep related data in another document and store an identifier.

```json
{
  "_id": "C1001",
  "name": "John",
  "addressId": "A5001"
}
```

And separately:

```json
{
  "_id": "A5001",
  "city": "New Orleans",
  "state": "LA"
}
```

The decision between these approaches is one of the most important DocumentDB modeling decisions.

---

# 5. Embedding

Embedding means:

> **Store related information inside the parent document.**

Example:

```json
{
  "_id": "P1001",
  "name": "Laptop",
  "manufacturer": {
    "name": "ExampleTech",
    "country": "USA"
  }
}
```

Instead of creating a separate manufacturer document.

---

# 6. Why Embed?

Embedding is attractive when:

* Data is frequently accessed together
* The child data belongs strongly to the parent
* The child data is relatively small
* The child data doesn't need independent access
* The child data doesn't grow without bound

For example:

```json
{
  "_id": "U1001",
  "name": "John",
  "preferences": {
    "language": "en",
    "notifications": true,
    "theme": "dark"
  }
}
```

Preferences naturally belong to the user.

Embedding makes sense.

---

# 7. Example: Address

Suppose an application always displays:

```text
Customer
+
Address
```

You could embed:

```json
{
  "_id": "C1001",
  "name": "John",
  "address": {
    "street": "123 Main St",
    "city": "New Orleans",
    "state": "LA",
    "zip": "70112"
  }
}
```

One document gives you everything.

---

# 8. Embedding Reduces Application Complexity

With embedded data:

```text
Application
     │
     ▼
Read customer
     │
     ▼
Customer + address
```

With separate documents:

```text
Application
     │
     ├── Read customer
     │
     └── Read address
```

Embedding can therefore reduce the number of database operations needed for common access patterns.

---

# 9. Referencing

Referencing means:

> Store related data separately and connect it using an identifier.

Example:

### Customer

```json
{
  "_id": "C1001",
  "name": "John",
  "addressId": "A5001"
}
```

### Address

```json
{
  "_id": "A5001",
  "street": "123 Main St",
  "city": "New Orleans",
  "state": "LA"
}
```

The relationship is:

```text
Customer
   │
   └── addressId
          │
          ▼
       Address
```

---

# 10. When Should You Reference?

Referencing can be appropriate when:

* Related data is large
* Related data grows continuously
* The child data is accessed independently
* The relationship is many-to-many
* The child data is shared by many parents
* You don't want repeated copies of the same data
* The data has an independent lifecycle

---

# 11. The Most Important Question

When deciding between embedding and referencing, ask:

> **"Will I usually need this data together?"**

If yes:

```text
Consider embedding.
```

If no:

```text
Consider referencing.
```

But that's only the beginning.

Also consider:

```text
Size
Growth
Update frequency
Ownership
Sharing
Access patterns
Consistency requirements
```

---

# 12. One-to-One Relationship

Example:

```text
User
  │
  └── Profile
```

A user has one profile.

You could embed:

```json
{
  "_id": "U1001",
  "name": "John",
  "profile": {
    "age": 30,
    "occupation": "Engineer"
  }
}
```

This is often a natural embedded model when the profile is small and accessed with the user.

---

# 13. One-to-Many Relationship

Suppose:

```text
Customer
   │
   ├── Order 1
   ├── Order 2
   ├── Order 3
   └── ...
```

This is one-to-many.

Now we have an important modeling question:

> Should orders be embedded inside the customer?

Sometimes yes.

Sometimes absolutely not.

It depends on how many orders a customer can have and how those orders are accessed.

---

# 14. Small One-to-Many → Embedding

Imagine a restaurant customer has a few saved addresses:

```json
{
  "_id": "C1001",
  "name": "John",
  "addresses": [
    {
      "type": "HOME",
      "city": "New Orleans"
    },
    {
      "type": "WORK",
      "city": "Metairie"
    }
  ]
}
```

This can be a good embedded model.

Why?

```text
Small
+
Bounded
+
Usually accessed with customer
```

---

# 15. Large One-to-Many → Referencing

Now consider:

```text
Customer
   │
   ├── Order 1
   ├── Order 2
   ├── ...
   └── Order 100,000
```

Embedding every order into one customer document would be a poor design.

Why?

Because the array grows continuously.

Instead:

```text
customers
    │
    └── customerId

orders
    │
    └── customerId
```

For example:

```json
{
  "_id": "O1001",
  "customerId": "C1001",
  "total": 150,
  "status": "SHIPPED"
}
```

---

# 16. Unbounded Arrays 🚨

This is one of the biggest DocumentDB modeling warnings.

Imagine:

```json
{
  "_id": "C1001",
  "orders": [
    ...
  ]
}
```

and every year:

```text
100 orders
→
1,000 orders
→
10,000 orders
→
100,000 orders
```

You now have a document that continuously grows.

That's an **unbounded array**.

Avoid modeling unlimited growth as one embedded array.

---

# 17. Bounded vs Unbounded

This distinction is extremely useful.

### Bounded

```text
Customer
 ├── Home address
 ├── Work address
 └── Billing address
```

Maximum maybe:

```text
3–5
```

Good candidate for embedding.

### Unbounded

```text
Customer
 ├── Order 1
 ├── Order 2
 ├── ...
 └── Order N
```

where N can keep growing.

Usually better to model orders separately.

---

# 18. One-to-Many Rule of Thumb

Remember:

```text
Small + bounded + accessed together
                ↓
            EMBED

Large + growing + independently accessed
                ↓
           REFERENCE
```

This isn't an absolute law, but it's a very useful starting point.

---

# 19. Many-to-Many Relationships

Now consider:

```text
Students
   ↕
Courses
```

One student can take many courses.

One course can have many students.

That's:

> **Many-to-many**

Example:

```text
Student A → Course 1
Student A → Course 2

Student B → Course 1
Student B → Course 3
```

Embedding becomes much more complicated.

---

# 20. Referencing Many-to-Many

You might model:

### Student

```json
{
  "_id": "S1001",
  "name": "John"
}
```

### Course

```json
{
  "_id": "C1001",
  "name": "AWS"
}
```

### Enrollment

```json
{
  "_id": "E1001",
  "studentId": "S1001",
  "courseId": "C1001"
}
```

Conceptually:

```text
Student
   │
   ▼
Enrollment
   ▲
   │
Course
```

This is a reference-oriented model.

---

# 21. But DocumentDB Is Not a Relational Database

Don't automatically recreate:

```text
3NF
+
JOIN tables
+
normalization
```

just because that's what you're used to.

DocumentDB gives you flexibility to embed information where it makes sense.

The goal is not:

> "Make DocumentDB look like SQL."

The goal is:

> **"Design a document model that works well for the application's access patterns."**

---

# 22. Denormalization

In relational databases, you often normalize data to reduce duplication.

Document databases frequently use **denormalization**.

Denormalization means:

> **Intentionally storing some duplicated information to make reads simpler or faster.**

Example:

```json
{
  "_id": "O1001",
  "customerId": "C1001",
  "customerName": "John",
  "total": 150
}
```

You already have:

```text
customerId → C1001
```

Why duplicate:

```text
customerName
```

?

Because the order screen may need the customer name every time.

---

# 23. Why Denormalize?

Suppose your application frequently needs:

```text
Order
+
Customer Name
+
Order Total
```

If customer name is stored in the order:

```text
One read
   ↓
Order
   ↓
Customer name already there
```

If it isn't:

```text
Read Order
   ↓
Get customerId
   ↓
Read Customer
   ↓
Get customer name
```

Denormalization can make common reads simpler.

---

# 24. The Cost of Denormalization

There's a catch.

Suppose John changes his name.

You have:

```text
Customer:
John Smith

Order 1:
John Smith

Order 2:
John Smith

Order 3:
John Smith
```

Now you potentially have multiple copies to update.

So:

```text
Denormalization
     │
     ├── Faster/simpler reads
     │
     └── More complicated updates
```

This is a classic trade-off.

---

# 25. Read-Heavy Applications

Denormalization can be especially useful when:

```text
Reads >> Writes
```

For example:

```text
1 write
10,000 reads
```

You might accept some duplicated data to make reads efficient.

But always consider consistency requirements.

---

# 26. Example: Product Catalog

Imagine an e-commerce product:

```json
{
  "_id": "P1001",
  "name": "Laptop",
  "brand": "ExampleTech",
  "price": 1299,
  "specifications": {
    "cpu": "Example CPU",
    "ram": "16GB",
    "storage": "1TB"
  }
}
```

The specifications naturally belong to the product.

Embedding is intuitive.

---

# 27. Example: Order

An order might look like:

```json
{
  "_id": "O1001",
  "customerId": "C1001",
  "status": "SHIPPED",
  "items": [
    {
      "productId": "P1001",
      "name": "Laptop",
      "quantity": 1,
      "price": 1299
    },
    {
      "productId": "P2001",
      "name": "Mouse",
      "quantity": 2,
      "price": 25
    }
  ],
  "total": 1349
}
```

Notice something interesting.

We have:

```text
productId
+
name
+
price
```

inside the order item.

Why?

Because the order needs a historical snapshot of what was purchased.

---

# 28. Historical Data Is a Special Case

Suppose a product currently costs:

```text
$1,500
```

But the customer purchased it when it cost:

```text
$1,299
```

If the order only stored:

```text
productId
```

and looked up the current product price, you might incorrectly display:

```text
$1,500
```

instead of:

```text
$1,299
```

So the order can store:

```json
{
  "productId": "P1001",
  "name": "Laptop",
  "price": 1299,
  "quantity": 1
}
```

This is intentional denormalization.

---

# 29. Snapshot Pattern

This is commonly called a:

> **Snapshot**

The order stores the information relevant at the time of purchase.

Conceptually:

```text
Product today
    ≠
Product at purchase time
```

Therefore:

```text
Order
  ↓
Historical snapshot
```

This is a very important real-world modeling pattern.

---

# 30. Document Boundaries

A powerful question is:

> **Where should one document end and another begin?**

For example:

```text
Customer
```

might contain:

```text
identity
contact information
preferences
addresses
```

while:

```text
Order
```

contains:

```text
order information
items
shipping information
payment status
```

Think of each document as a meaningful unit of application data.

---

# 31. Ownership

Ask:

> **Who owns this data?**

For example:

```text
Customer
  └── Preferences
```

Preferences belong to the customer.

Embedding makes sense.

But:

```text
Product
  ↕
Supplier
```

may be a more independent relationship.

Referencing might make more sense depending on access patterns and lifecycle.

---

# 32. Independent Lifecycle

Another useful question:

> **Does this data have its own lifecycle?**

For example:

```text
Customer
```

might exist for years.

An:

```text
Audit Event
```

might have:

```text
createdAt
retention policy
expiration
```

and be processed independently.

That can favor separate documents/collections.

---

# 33. Access Pattern First

Let's imagine our application has these operations:

```text
1. Get customer profile
2. Get customer's recent orders
3. Get a specific order
4. Search products
5. Get product details
```

Now model around them.

### Customer profile

```text
Customer document
```

### Recent orders

Could be:

```text
Orders collection
```

with:

```text
customerId
createdAt
```

and appropriate indexes.

### Specific order

```text
Order document
```

### Product search

```text
Products collection
```

This is access-pattern-driven design.

---

# 34. Don't Embed Just Because You Can

Document databases make embedding easy.

That doesn't mean everything should be embedded.

Bad model:

```json
{
  "_id": "C1001",
  "orders": [
    ... 100,000 orders ...
  ]
}
```

Better:

```text
customers
    │
    └── C1001

orders
    │
    ├── customerId = C1001
    ├── customerId = C1001
    └── ...
```

---

# 35. Don't Reference Just Because You're Used to SQL

The opposite mistake is also common.

Bad over-normalization:

```text
Customer
   ↓
Address
   ↓
City
   ↓
State
   ↓
Country
```

If your application always needs:

```text
Customer + Address + City + State
```

this can create unnecessary complexity.

A document model might simply embed:

```json
{
  "name": "John",
  "address": {
    "city": "New Orleans",
    "state": "LA",
    "country": "USA"
  }
}
```

---

# 36. The Modeling Decision Tree

Use this mental model:

```text
Related data?
      │
      ▼
Is it usually accessed together?
      │
   ┌──┴──┐
  YES    NO
   │      │
   ▼      ▼
Embed   Reference
```

But then ask:

```text
Is it bounded?
   │
   ├── YES → Embedding becomes more attractive
   │
   └── NO  → Be careful with embedding
```

Then:

```text
Does it have an independent lifecycle?
   │
   ├── YES → Referencing becomes more attractive
   │
   └── NO  → Embedding may be appropriate
```

---

# 37. Modeling Questions to Ask

For every relationship, ask:

### Question 1

> How is the data queried?

### Question 2

> How frequently is it queried?

### Question 3

> Is it always retrieved together?

### Question 4

> How large can it become?

### Question 5

> Does it grow indefinitely?

### Question 6

> Does it change independently?

### Question 7

> Is it shared by many documents?

### Question 8

> What consistency behavior do we need?

These questions are more important than memorizing:

```text
"Embed"
or
"Reference"
```

---

# 38. Example: Social Media Post

Suppose:

```text
Post
 ├── author
 ├── text
 ├── createdAt
 └── comments
```

Should comments be embedded?

Maybe.

If comments are:

```text
small
bounded
rare
```

embedding could work.

But if a post can receive:

```text
1 million comments
```

then:

```text
comments[]
```

becomes an unbounded array.

A separate comments collection may be more appropriate.

---

# 39. Example: User Preferences

```json
{
  "_id": "U1001",
  "name": "John",
  "preferences": {
    "theme": "dark",
    "language": "en",
    "notifications": true
  }
}
```

Excellent embedding candidate.

Why?

```text
Small
+
Bounded
+
Owned by user
+
Usually accessed with user
```

---

# 40. Example: Shopping Cart

A shopping cart might be modeled as:

```json
{
  "_id": "CART1001",
  "userId": "U1001",
  "items": [
    {
      "productId": "P1001",
      "quantity": 2,
      "price": 1299
    },
    {
      "productId": "P2001",
      "quantity": 1,
      "price": 25
    }
  ],
  "updatedAt": "2026-08-11T10:00:00Z"
}
```

Why embed items?

Because the application commonly asks:

> "Give me this user's current cart."

The cart and its items are naturally accessed together.

---

# 41. Example: Audit Logs

Audit events might look like:

```json
{
  "_id": "A1001",
  "userId": "U1001",
  "action": "LOGIN",
  "timestamp": "2026-08-11T10:00:00Z"
}
```

You probably don't want:

```json
{
  "_id": "U1001",
  "auditLogs": [
      ...
      millions of events
  ]
}
```

Instead:

```text
users
   │
   └── U1001

auditEvents
   │
   ├── userId = U1001
   ├── userId = U1001
   └── ...
```

This is a classic case where continuously growing event data belongs separately.

---

# 42. Time-Series-Like Data

Consider:

```text
Sensor
   │
   ├── Reading 1
   ├── Reading 2
   ├── Reading 3
   └── ...
```

If readings arrive every second:

```text
60/minute
3,600/hour
86,400/day
```

Embedding all readings into the sensor document is generally a poor design.

The data is:

```text
high volume
+
continuously growing
```

So model it separately.

---

# 43. Duplication Isn't Automatically Bad

In a document database, seeing the same value in multiple documents doesn't automatically mean your model is wrong.

For example:

```text
Order 1 → customerName = John
Order 2 → customerName = John
Order 3 → customerName = John
```

This may be intentional.

The real question is:

> **What consistency requirements does that duplicated value have?**

If it represents historical information, duplication may be exactly what you want.

---

# 44. Data Consistency Trade-Off

Consider:

```text
Embedding
```

Advantages:

```text
Simple reads
Fewer round trips
Data naturally grouped
```

Potential disadvantages:

```text
Larger documents
Possible duplication
Updates may touch larger documents
```

Referencing:

```text
Advantages:
Independent data
Avoid duplication
Better for large/unbounded relationships
```

Potential disadvantages:

```text
Multiple reads
More application logic
Potentially more complicated access patterns
```

---

# 45. Modeling Is About Trade-Offs

There is rarely one universally correct schema.

For example:

```text
Option A
Embed

Option B
Reference

Option C
Hybrid
```

All three might be valid depending on:

```text
Read patterns
Write patterns
Data size
Growth
Consistency
Latency
```

A good DocumentDB engineer explains **why** they chose the model.

---

# 46. Hybrid Modeling

You don't have to choose:

```text
100% embedded
```

or:

```text
100% referenced
```

You can combine both.

Example:

```json
{
  "_id": "O1001",
  "customerId": "C1001",
  "customerName": "John",
  "items": [
    {
      "productId": "P1001",
      "name": "Laptop",
      "price": 1299,
      "quantity": 1
    }
  ]
}
```

Here:

```text
customerId
```

provides a reference.

But:

```text
customerName
```

is duplicated.

And:

```text
items[]
```

are embedded.

This is a **hybrid model**.

---

# 47. Schema Flexibility

DocumentDB documents don't have to all contain exactly the same fields.

For example:

### Customer A

```json
{
  "_id": "C1",
  "name": "John",
  "email": "john@example.com"
}
```

### Customer B

```json
{
  "_id": "C2",
  "name": "Alice",
  "email": "alice@example.com",
  "loyalty": {
    "tier": "GOLD"
  }
}
```

This flexibility can be useful.

But:

> **Schema flexibility doesn't mean schema chaos.**

Your application should still establish sensible conventions.

---

# 48. Schema-on-Read vs Schema Discipline

Document databases are often described as:

> Schema flexible.

That means the database doesn't necessarily force every document to have an identical structure.

But production applications should still maintain:

```text
Consistent field names
Consistent data types
Validation rules
Versioning strategy
Clear document contracts
```

Otherwise your application becomes difficult to maintain.

---

# 49. Example of Bad Inconsistency

Bad:

```json
{
  "age": 30
}
```

Another document:

```json
{
  "age": "30"
}
```

Another:

```json
{
  "age": "thirty"
}
```

Technically flexible doesn't mean:

> "Anything goes."

Consistent data types make querying, indexing, and application logic much easier.

---

# 50. Schema Versioning

Applications evolve.

Suppose version 1 contains:

```json
{
  "name": "John",
  "phone": "123"
}
```

Later you introduce:

```json
{
  "name": "John",
  "contact": {
    "phone": "123"
  }
}
```

You may need a version field:

```json
{
  "_id": "C1001",
  "schemaVersion": 2,
  "name": "John",
  "contact": {
    "phone": "123"
  }
}
```

This can help applications understand different document versions during migrations.

---

# 51. Data Modeling + Indexes

Lessons 5 and 6 connect directly.

Suppose your application frequently runs:

```javascript
db.orders.find({
    "customerId": "C1001",
    "status": "SHIPPED"
})
```

Your model should expose those fields clearly:

```json
{
  "_id": "O1001",
  "customerId": "C1001",
  "status": "SHIPPED"
}
```

Then Lesson 5 tells us to consider an appropriate index:

```javascript
{
  "customerId": 1,
  "status": 1
}
```

So:

```text
Data Model
    ↓
Access Pattern
    ↓
Query
    ↓
Index
    ↓
Performance
```

These concepts are tightly connected.

---

# 52. Data Modeling + Query Design

Suppose your application constantly asks:

> "Show a user's five most recent orders."

A model like:

```text
orders
 ├── customerId
 ├── createdAt
 └── ...
```

supports a query such as:

```javascript
db.orders.find({
    "customerId": "C1001"
})
.sort({
    "createdAt": -1
})
.limit(5)
```

Now you can design the index around that access pattern.

This demonstrates why:

> **You should design the model and indexes together.**

---

# 53. Data Modeling + Application Workflow

Think of the complete chain:

```text
Business requirement
        ↓
Application access pattern
        ↓
Document model
        ↓
Query design
        ↓
Index design
        ↓
Performance
```

If you skip the first steps and simply copy a relational schema into DocumentDB, you may miss many of the benefits of the document model.

---

# 54. Common Data Modeling Mistakes

## Mistake 1 — Huge documents

Avoid documents that continuously grow without bounds.

---

## Mistake 2 — Unbounded arrays

Be very careful with:

```text
comments[]
orders[]
events[]
transactions[]
```

when they can grow indefinitely.

---

## Mistake 3 — Over-normalization

Don't create many tiny documents simply because that's how relational modeling works.

---

## Mistake 4 — Over-embedding

Don't put huge independent datasets into one document.

---

## Mistake 5 — Ignoring access patterns

Don't design the schema first and ask how to query it later.

---

## Mistake 6 — Ignoring consistency

If you duplicate data, understand how and when those copies are updated.

---

## Mistake 7 — Inconsistent field types

Avoid:

```text
age: 30
```

in one document and:

```text
age: "30"
```

in another unless there is a deliberate reason.

---

# 55. Interview Question: Embedding vs Referencing

A strong answer:

> **Embedding stores related data inside the same document and is useful when the data is usually accessed together, is relatively small, and has bounded growth. Referencing stores related data separately and is often better when the related data is large, independently accessed, shared, or unbounded.**

---

# 56. Interview Question: Why is an unbounded array dangerous?

Strong answer:

> **An unbounded array can cause a document to grow continuously, resulting in large documents, inefficient updates and reads, and poor scalability. Frequently growing child data is often better stored as separate documents.**

---

# 57. Interview Question: What is denormalization?

> **Denormalization is intentionally duplicating related data to optimize common reads or simplify access, accepting additional storage and update/consistency considerations.**

---

# 58. Interview Question: How do you design a DocumentDB schema?

A strong answer:

> **I start with the application's access patterns rather than simply converting relational tables. I identify the most common reads and writes, determine which data is naturally accessed together, decide between embedding and referencing based on size, growth, ownership, and access patterns, and then design indexes around the resulting queries.**

That's a strong production-level answer.

---

# 59. Real-World Design Exercise

Let's design an e-commerce system.

We have:

```text
Users
Products
Orders
Reviews
```

Ask:

### User

What belongs inside?

Potentially:

```text
name
email
preferences
addresses
```

### Product

Potentially:

```text
name
price
brand
specifications
```

### Order

Potentially:

```text
customerId
items[]
status
shipping snapshot
payment status
timestamps
```

### Reviews

Potentially:

```text
productId
userId
rating
comment
createdAt
```

Notice that reviews can grow indefinitely for a product.

Therefore:

```text
Product
   │
   └── Don't necessarily embed every review
```

Instead:

```text
reviews
   │
   └── productId
```

---

# 60. A Possible High-Level Model

```text
                    ┌──────────────┐
                    │    USER      │
                    │              │
                    │ name         │
                    │ email        │
                    │ preferences  │
                    │ addresses[]  │
                    └──────┬───────┘
                           │
                           │ customerId
                           ▼
                    ┌──────────────┐
                    │    ORDER     │
                    │              │
                    │ customerId   │
                    │ items[]      │
                    │ status       │
                    │ total        │
                    └──────┬───────┘
                           │
                           │ productId
                           ▼
                    ┌──────────────┐
                    │   PRODUCT    │
                    │              │
                    │ name         │
                    │ price        │
                    │ specs        │
                    └──────┬───────┘
                           │
                           │ productId
                           ▼
                    ┌──────────────┐
                    │   REVIEW     │
                    │              │
                    │ productId    │
                    │ userId       │
                    │ rating       │
                    └──────────────┘
```

This is not the only possible design.

The correct model depends on actual application access patterns.

---

# 61. Lesson 6 Mental Model 🧠

When you model a DocumentDB application, think:

```text
                    BUSINESS REQUIREMENT
                            │
                            ▼
                     ACCESS PATTERNS
                            │
                            ▼
                    DOCUMENT BOUNDARIES
                            │
                 ┌──────────┴──────────┐
                 │                     │
             EMBEDDING             REFERENCE
                 │                     │
        Small / bounded        Large / independent
        Accessed together      Growing / shared
                 │                     │
                 └──────────┬──────────┘
                            ▼
                      QUERY DESIGN
                            │
                            ▼
                       INDEX DESIGN
                            │
                            ▼
                        PERFORMANCE
```

---

# 🎯 The 7 Rules to Remember

If you remember nothing else from this lesson, remember these:

### Rule 1

> **Model for access patterns.**

### Rule 2

> **Embed data that is naturally accessed together.**

### Rule 3

> **Reference data that is large, independent, shared, or unbounded.**

### Rule 4

> **Avoid unbounded arrays.**

### Rule 5

> **Denormalization is allowed—and often useful—but understand the consistency cost.**

### Rule 6

> **Schema flexibility does not mean inconsistent data.**

### Rule 7

> **Design the data model, queries, and indexes together.**

---

# 📝 Lesson 6 Mini Quiz

Try these yourself:

### 1.

What is the main difference between embedding and referencing?

### 2.

When is embedding usually a good choice?

### 3.

Why are unbounded arrays dangerous?

### 4.

A customer has 3 saved addresses. Would you consider embedding them?

Why?

### 5.

A customer has 500,000 orders. Would you embed the orders?

Why or why not?

### 6.

What is denormalization?

### 7.

Why might an order store the product's historical price instead of reading the current product price?

### 8.

What is an access pattern?

### 9.

Why shouldn't you simply convert every SQL table into a DocumentDB collection?

### 10.

What three things should be designed together?

---

# 📚 Course Progress

```text
✅ Lesson 1 — DocumentDB Fundamentals
✅ Lesson 2 — Documents, Collections & BSON
✅ Lesson 3 — CRUD Operations
✅ Lesson 4 — Queries & Operators
✅ Lesson 5 — Indexes & Query Performance
✅ Lesson 6 — Data Modeling
⬜ Lesson 7 — DocumentDB Architecture on AWS
⬜ Lesson 8 — Security & Networking
⬜ Lesson 9 — Backup, Monitoring & Operations
⬜ Lesson 10 — Production Project
```

## Next: **Lesson 7 — Amazon DocumentDB Architecture on AWS ☁️**

We'll move from the **database itself** to how DocumentDB actually runs inside AWS: **clusters, instances, primary/replica architecture, storage, compute, endpoints, Multi-AZ, failover, scaling, high availability, and how an application connects to a DocumentDB cluster.**
