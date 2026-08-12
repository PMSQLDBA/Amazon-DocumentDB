# Lesson 5 — Indexes & Query Performance ⚡

Welcome to **Lesson 5 of the Amazon DocumentDB course**.

So far:

1. DocumentDB Fundamentals
2. Documents, Collections & BSON
3. CRUD Operations
4. Queries & Operators
5. **Indexes & Query Performance ← Today**

This is one of the most important lessons for **real-world DocumentDB development and AWS interviews**.

The central question is:

> **How does DocumentDB find the documents you ask for efficiently?**

---

# 1. Why Do We Need Indexes?

Imagine a collection containing:

```text
100 documents
```

You run:

```javascript
db.customers.find({
    "email": "john@example.com"
})
```

The database needs to find John's document.

With a small collection, searching documents may not seem like a big deal.

Now imagine:

```text
100 million documents
```

Searching every document one by one would be expensive.

An **index** gives the database an additional data structure that can help it locate matching documents more efficiently.

Think of a book.

Without an index:

```text
Search for "DocumentDB"
        ↓
Read page 1
Read page 2
Read page 3
Read page 4
...
```

With an index:

```text
Index
  ↓
DocumentDB → Page 417
  ↓
Go directly toward the relevant location
```

The database index serves a similar purpose.

---

# 2. Query vs Index

This distinction is extremely important.

A **query** asks:

> What data do I want?

An **index** helps the database determine:

> How can I find that data efficiently?

For example:

```javascript
db.customers.find({
    "email": "john@example.com"
})
```

The query says:

```text
Find email = john@example.com
```

An index on:

```text
email
```

can help DocumentDB locate matching documents efficiently.

---

# 3. Without an Index

Suppose we have:

```text
customers
│
├── Customer 1
├── Customer 2
├── Customer 3
├── ...
└── Customer 1,000,000
```

And query:

```javascript
db.customers.find({
    "email": "john@example.com"
})
```

Conceptually, without a useful index, the database may need to inspect many documents.

This is commonly referred to as a:

> **Collection scan**

or:

> **COLLSCAN**

The exact execution behavior depends on the query and engine, but the important idea is:

```text
No useful index
      ↓
Potentially examine many documents
      ↓
More work
      ↓
Slower query as data grows
```

---

# 4. With an Index

Now suppose we create an index on:

```text
email
```

Conceptually:

```text
Index on email
       │
       ├── alice@example.com
       ├── bob@example.com
       ├── john@example.com
       └── ...
```

Then:

```javascript
db.customers.find({
    "email": "john@example.com"
})
```

can use that index to locate the relevant document(s).

Conceptually:

```text
Query
  │
  ▼
Email Index
  │
  ▼
Matching document(s)
```

This can dramatically reduce the amount of data the engine needs to inspect.

---

# 5. Creating an Index

A common MongoDB-compatible syntax is:

```javascript
db.customers.createIndex({
    "email": 1
})
```

Here:

```text
1
```

represents ascending index order.

So:

```text
email: 1
```

means:

> Create an ascending index on `email`.

---

# 6. Descending Index

You can also create:

```javascript
db.customers.createIndex({
    "email": -1
})
```

where:

```text
1  → ascending
-1 → descending
```

For many equality lookups, the direction isn't the main concern. Index ordering becomes especially relevant for sorting and compound indexes.

---

# 7. Single-Field Index

An index on one field is a:

> **Single-field index**

Example:

```javascript
db.customers.createIndex({
    "email": 1
})
```

The indexed field is:

```text
email
```

This can be useful for queries such as:

```javascript
db.customers.find({
    "email": "john@example.com"
})
```

---

# 8. `_id` Index

Remember `_id` from Lesson 2?

```json
{
  "_id": "C1001",
  "name": "John"
}
```

The `_id` field is special.

DocumentDB maintains an index for `_id` so that document identity can be efficiently managed.

This is one reason queries such as:

```javascript
db.customers.findOne({
    "_id": "C1001"
})
```

are natural and efficient.

---

# 9. Compound Indexes

Now suppose your application frequently runs:

```javascript
db.employees.find({
    "department": "Engineering",
    "age": {
        "$gt": 30
    }
})
```

You could consider a **compound index**:

```javascript
db.employees.createIndex({
    "department": 1,
    "age": 1
})
```

This index contains multiple fields.

Conceptually:

```text
Compound Index
│
├── department
└── age
```

---

# 10. Why Compound Indexes?

Suppose your application repeatedly asks:

```text
department = Engineering
AND
age > 30
```

A compound index can be designed around this access pattern.

Instead of thinking:

```text
Index 1 → department
Index 2 → age
```

you can potentially create:

```text
Compound index
→ department + age
```

The exact effectiveness depends on the query shape and index ordering.

---

# 11. Index Field Order Matters

This is one of the most important concepts in compound indexes.

Consider:

```javascript
db.employees.createIndex({
    "department": 1,
    "age": 1
})
```

The order is:

```text
1. department
2. age
```

This is **not necessarily equivalent** to:

```javascript
db.employees.createIndex({
    "age": 1,
    "department": 1
})
```

Index design must take query patterns into account.

---

# 12. Equality Before Range — Important Pattern

Suppose your common query is:

```javascript
db.employees.find({
    "department": "Engineering",
    "age": {
        "$gt": 30
    }
})
```

A common index design principle is:

```text
Equality fields
       ↓
Range fields
```

So an index like:

```javascript
{
    "department": 1,
    "age": 1
}
```

can be a natural candidate for this query pattern.

Think:

```text
department = Engineering
       ↓
then evaluate age > 30
```

This isn't a universal rule for every workload, but it's an important starting principle for compound-index design.

---

# 13. Indexes and Sorting

Indexes aren't only about filtering.

Suppose you frequently run:

```javascript
db.employees.find({
    "department": "Engineering"
}).sort({
    "salary": -1
})
```

You might consider an index such as:

```javascript
db.employees.createIndex({
    "department": 1,
    "salary": -1
})
```

Conceptually:

```text
Filter:
department = Engineering

        +

Sort:
salary DESC
```

A suitable compound index can potentially help with both operations.

---

# 14. Query Performance Isn't Just About Indexes

This is important.

Don't think:

> "Add an index and everything becomes fast."

Query performance depends on several things:

```text
Query performance
│
├── Query shape
├── Index design
├── Data distribution
├── Number of documents
├── Number of returned documents
├── Projection
├── Sort
├── Network latency
├── Instance resources
└── Application behavior
```

Indexes are a major part of the story, but not the entire story.

---

# 15. `explain()`

One of the most important tools for query-performance analysis is:

```javascript
explain()
```

Instead of simply asking:

```javascript
db.customers.find({
    "email": "john@example.com"
})
```

you can inspect how the query is being executed.

Conceptually:

```javascript
db.customers.find({
    "email": "john@example.com"
}).explain()
```

This helps you investigate:

> Is my query using an index?

> How much work is the database doing?

---

# 16. Why `explain()` Matters

Suppose you created:

```javascript
db.customers.createIndex({
    "email": 1
})
```

Then run:

```javascript
db.customers.find({
    "email": "john@example.com"
}).explain()
```

You want to understand whether the execution plan uses the index.

Conceptually:

```text
Query
 │
 ▼
Query Planner
 │
 ├── Possible plans
 │
 └── Selected plan
        │
        ▼
     Execution
```

This is much better than simply guessing whether an index is useful.

---

# 17. COLLSCAN vs IXSCAN

Two terms you'll often encounter when studying MongoDB-compatible query execution are:

```text
COLLSCAN
IXSCAN
```

### COLLSCAN

Collection scan.

Conceptually:

```text
Collection
│
├── Check document
├── Check document
├── Check document
├── ...
└── Check document
```

### IXSCAN

Index scan.

Conceptually:

```text
Index
 │
 ▼
Find relevant index entries
 │
 ▼
Fetch matching documents
```

For selective queries, an appropriate index can greatly reduce work.

---

# 18. Selectivity

A very important performance concept is:

> **Selectivity**

Suppose you have 10 million customers.

Query A:

```javascript
{
    "country": "USA"
}
```

Maybe 7 million documents match.

Query B:

```javascript
{
    "email": "john@example.com"
}
```

Maybe only one document matches.

The second query is much more selective.

Conceptually:

```text
High selectivity
      ↓
Few matching documents
      ↓
Index can be very useful
```

Whereas:

```text
Low selectivity
      ↓
Huge number of matches
```

may reduce the benefit of an index for certain query shapes.

---

# 19. Don't Index Everything

A beginner might think:

> "If indexes make queries faster, I'll index every field."

That's a mistake.

Indexes have costs.

Think:

```text
Indexes
   │
   ├── Faster reads
   │
   ├── Additional storage
   │
   └── Additional work for writes/updates
```

When you insert or modify data, relevant indexes may also need to be maintained.

So:

> **Indexes are a trade-off.**

---

# 20. Too Many Indexes

Suppose you create indexes on:

```text
name
email
age
salary
department
city
state
zip
phone
status
createdAt
...
```

You may end up with a large collection of indexes that aren't actually useful.

This can cause:

* More storage usage
* More maintenance overhead
* More complicated index management
* Potentially slower writes

The goal isn't:

> Maximum number of indexes.

The goal is:

> **The right indexes for the application's important query patterns.**

---

# 21. Query-Driven Index Design

This is a professional way to think about indexing.

Don't start with:

> "What fields should I index?"

Start with:

> **"What queries does my application execute frequently?"**

Suppose your application frequently does:

### Query 1

```javascript
{
    "email": "john@example.com"
}
```

### Query 2

```javascript
{
    "department": "Engineering",
    "age": {
        "$gt": 30
    }
}
```

### Query 3

```javascript
{
    "status": "ACTIVE"
}
```

Then you can evaluate indexes based on those access patterns.

---

# 22. Example: Customer Application

Suppose our application has:

```text
customers
```

and common queries are:

```text
1. Find customer by email
2. Find customer by customerId
3. Find customers by status
4. Find customers by city
```

Potential indexes might include:

```text
email
customerId
status
city
```

But we don't blindly create them.

We evaluate:

```text
How often?
How selective?
How expensive?
How much data?
What are the write patterns?
```

---

# 23. Example: E-Commerce Query

Suppose:

```text
products
```

contains:

```json
{
  "_id": "P1001",
  "category": "Laptop",
  "brand": "ExampleTech",
  "price": 1299,
  "inStock": true
}
```

A common application query might be:

> Find laptops from ExampleTech under $1500.

Query:

```javascript
db.products.find({
    "category": "Laptop",
    "brand": "ExampleTech",
    "price": {
        "$lt": 1500
    }
})
```

Potential compound-index design could reflect:

```text
category
brand
price
```

But the exact index should be validated against real query patterns and execution plans.

---

# 24. Prefix Principle for Compound Indexes

Suppose you have:

```javascript
db.products.createIndex({
    "category": 1,
    "brand": 1,
    "price": 1
})
```

Think of the index as having a logical prefix:

```text
category
   ↓
brand
   ↓
price
```

Queries that use the leading fields generally have more opportunity to benefit from the index.

For example:

```text
category
```

or:

```text
category + brand
```

align naturally with the beginning of the index.

A query only on:

```text
price
```

doesn't align with the leading prefix in the same way.

This is why **field ordering matters**.

---

# 25. A Simple Analogy

Imagine a phone directory sorted by:

```text
Last Name → First Name
```

You can quickly find:

```text
Smith
```

and:

```text
Smith, John
```

But searching the entire directory for:

```text
John
```

isn't equally efficient because John isn't the leading sort key.

Compound indexes behave similarly.

---

# 26. Covered Queries — Concept

Another advanced concept is a **covered query**.

Imagine an index contains:

```text
email
name
```

and your query needs:

```text
email
name
```

If the index contains everything required to answer the query, the database may be able to avoid fetching the full documents.

Conceptually:

```text
Query
  ↓
Index
  ↓
All required information
  ↓
No full document fetch required
```

This can be very efficient.

However, whether a specific query is covered depends on the exact query, projection, `_id` behavior, and engine capabilities. Always validate with `explain()`.

---

# 27. Projection Can Help

Suppose your document is:

```json
{
  "_id": "C1001",
  "name": "John",
  "email": "john@example.com",
  "address": {...},
  "orders": [...],
  "preferences": {...},
  "history": [...]
}
```

But your API only needs:

```text
name
email
```

Instead of returning the entire document, use projection:

```javascript
db.customers.find(
    {
        "email": "john@example.com"
    },
    {
        "name": 1,
        "email": 1
    }
)
```

This can reduce the amount of data returned to the application.

---

# 28. Query Performance and Network Cost

Performance isn't just database CPU.

Imagine your document is:

```text
500 KB
```

but your API only needs:

```text
name
email
```

Returning the entire document repeatedly can increase:

```text
Database → Network → Application
```

traffic.

So good query design considers:

```text
Filter
+
Projection
+
Index
```

---

# 29. Bad Query Pattern

Imagine:

```javascript
db.customers.find({})
```

This asks for:

> Every document.

If the collection contains millions of documents, that's potentially expensive.

Then imagine:

```javascript
db.customers.find({}).sort({
    "name": 1
})
```

Now you're asking for everything and sorting it.

That's usually very different from:

```javascript
db.customers.find({
    "status": "ACTIVE"
})
```

with an appropriate index.

---

# 30. Avoid Returning More Data Than Necessary

A common API mistake is:

```text
GET /customers
```

and returning enormous documents for every customer.

Instead, ask:

> What does the API actually need?

For example:

```text
Customer list
→ id
→ name
→ email
```

rather than:

```text
Customer list
→ entire customer profile
→ orders
→ history
→ preferences
→ metadata
→ ...
```

Query design and application API design should work together.

---

# 31. Indexing Nested Fields

Suppose your document contains:

```json
{
  "name": "John",
  "address": {
    "city": "New Orleans"
  }
}
```

You can create an index on the nested field:

```javascript
db.customers.createIndex({
    "address.city": 1
})
```

Then a query such as:

```javascript
db.customers.find({
    "address.city": "New Orleans"
})
```

can potentially use that index.

The important concept:

> **Indexes aren't limited to top-level fields.**

---

# 32. Indexing Arrays

Arrays introduce additional considerations.

Suppose:

```json
{
  "name": "John",
  "skills": [
    "AWS",
    "Python",
    "DocumentDB"
  ]
}
```

You may encounter indexes on array fields.

These are commonly called:

> **Multikey indexes**

because one document can contribute multiple index entries for array elements.

Conceptually:

```text
Document
  │
  └── skills
       ├── AWS
       ├── Python
       └── DocumentDB
             │
             ▼
       Multiple index entries
```

Array indexing has important rules and limitations, so don't assume every compound-array index design is valid.

---

# 33. Indexing and Writes

Imagine:

```text
Collection
   │
   ├── Documents
   │
   ├── Index A
   ├── Index B
   └── Index C
```

When a document changes, the relevant indexes may need maintenance.

Therefore:

```text
More indexes
    ↓
More index maintenance
    ↓
Potential write overhead
```

This is why you shouldn't create indexes without a reason.

---

# 34. Performance Investigation Process

When a query is slow, don't immediately create an index.

Use a process:

```text
1. Identify slow query
        ↓
2. Understand query pattern
        ↓
3. Run explain()
        ↓
4. Inspect execution plan
        ↓
5. Determine whether an index helps
        ↓
6. Create/adjust index
        ↓
7. Test again
        ↓
8. Monitor in production
```

This is a much better engineering approach.

---

# 35. Example Troubleshooting

Suppose this query is slow:

```javascript
db.orders.find({
    "customerId": "C1001",
    "status": "SHIPPED"
})
```

First ask:

> Is this query frequent?

If yes, investigate an index such as:

```javascript
db.orders.createIndex({
    "customerId": 1,
    "status": 1
})
```

Then test the query with:

```javascript
db.orders.find({
    "customerId": "C1001",
    "status": "SHIPPED"
}).explain()
```

The important point is:

> **Don't blindly index; validate.**

---

# 36. Query Optimization Checklist

When you encounter a slow query, ask:

### 1. What is the query?

```text
What exactly are we searching for?
```

### 2. How many documents match?

```text
Few?
Many?
Almost all?
```

### 3. Is there an appropriate index?

```text
Yes / No
```

### 4. Is the index aligned with the query?

Especially:

```text
Compound field order
```

### 5. Are we returning too much data?

Consider projection.

### 6. Are we sorting?

Does the index support the filtering/sorting pattern?

### 7. What does `explain()` show?

Don't guess.

---

# 37. Indexing Strategy

A strong production strategy looks like:

```text
Application queries
       ↓
Identify important query patterns
       ↓
Design indexes
       ↓
Test with explain()
       ↓
Measure performance
       ↓
Monitor
       ↓
Adjust when workload changes
```

Indexes should evolve with the application.

---

# 38. Common Beginner Mistakes

## Mistake 1: Index everything

❌ Bad:

```text
Every field gets an index
```

Better:

```text
Index important query patterns
```

---

## Mistake 2: Ignore compound index order

❌

```text
department + age
```

and:

```text
age + department
```

are not interchangeable designs.

---

## Mistake 3: Never use `explain()`

Don't assume your index is being used.

Verify.

---

## Mistake 4: Return giant documents unnecessarily

Use projection when appropriate.

---

## Mistake 5: Create indexes without considering writes

Indexes help reads but have maintenance/storage costs.

---

# 39. Interview Question: What is an index?

A strong answer:

> **An index is a data structure maintained to help DocumentDB locate matching documents more efficiently for supported query patterns, reducing the amount of data that may need to be scanned.**

---

# 40. Interview Question: Why not index every field?

Strong answer:

> **Indexes consume storage and require maintenance when data changes. Too many indexes can increase write overhead and operational complexity, so indexes should be designed around important application query patterns.**

---

# 41. Interview Question: What is a compound index?

> **A compound index contains multiple fields and is designed to support query and/or sort patterns involving those fields. Field order is important because it affects which query patterns can efficiently use the index.**

---

# 42. Interview Question: What is `explain()`?

> **`explain()` provides information about how a query is executed, allowing you to inspect the query plan and determine whether indexes and other execution strategies are being used.**

---

# 43. Interview Question: What is COLLSCAN?

> **COLLSCAN refers to a collection scan, where the engine examines documents in the collection rather than efficiently navigating an applicable index for the query.**

---

# 44. Interview Question: What is IXSCAN?

> **IXSCAN refers to scanning an index to identify relevant index entries as part of query execution.**

---

# 45. Interview Question: What is selectivity?

> **Selectivity describes how narrowly a query condition identifies documents. A highly selective query matches relatively few documents, while a low-selectivity query matches many documents.**

---

# 🧠 Lesson 5 Mini Quiz

Try these without looking back.

### 1.

Why do we use indexes?

### 2.

What is the difference between a query and an index?

### 3.

Create an ascending index on:

```text
email
```

### 4.

What is a compound index?

### 5.

Why does field order matter in a compound index?

### 6.

What is the difference between:

```text
COLLSCAN
```

and:

```text
IXSCAN
```

### 7.

What does `explain()` help you determine?

### 8.

Why shouldn't you create indexes on every field?

### 9.

What is selectivity?

### 10.

Suppose your application frequently executes:

```javascript
db.orders.find({
    "customerId": "C1001",
    "status": "SHIPPED"
})
```

What kind of index might you investigate?

---

# 🎯 Lesson 5 — Final Mental Model

Think about performance this way:

```text
                    APPLICATION
                         │
                         ▼
                       QUERY
                         │
             ┌───────────┴───────────┐
             │                       │
          FILTER                 PROJECTION
             │
             ▼
        QUERY PLANNER
             │
             ▼
        ┌────┴────┐
        │         │
     INDEX      SCAN
        │         │
        ▼         ▼
   Matching     Documents
    entries     examined
        │
        └────┬────┘
             ▼
          RESULTS
```

And the professional workflow is:

```text
Slow Query
    ↓
Understand Query Pattern
    ↓
explain()
    ↓
Inspect Plan
    ↓
Design/Adjust Index
    ↓
Test
    ↓
Monitor
```

## 🔥 The one thing I want you to remember

> **Don't think "indexes make databases fast." Think "the right index makes the right query efficient."**

That's the difference between simply knowing what an index is and understanding **database performance engineering**.

---

## 📚 Course Progress

```text
✅ Lesson 1 — DocumentDB Fundamentals
✅ Lesson 2 — Documents, Collections & BSON
✅ Lesson 3 — CRUD Operations
✅ Lesson 4 — Queries & Operators
✅ Lesson 5 — Indexes & Query Performance
⬜ Lesson 6 — Data Modeling
⬜ Lesson 7 — DocumentDB Architecture on AWS
⬜ Lesson 8 — Security & Networking
⬜ Lesson 9 — Backup, Monitoring & Operations
⬜ Lesson 10 — Production Project
```

### Next: **Lesson 6 — Data Modeling 🏗️**

This is a **very important DocumentDB lesson**. We'll learn **embedding vs referencing, one-to-one, one-to-many, many-to-many, access-pattern-driven design, denormalization, document boundaries, and how to decide what should actually live inside a DocumentDB document.**
