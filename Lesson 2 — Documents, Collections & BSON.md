# Lesson 2 — Documents, Collections & BSON

Welcome to **Lesson 2 of the Amazon DocumentDB course**.

In Lesson 1, we built the mental model:

```text
DocumentDB
   ↓
Cluster
   ↓
Database
   ↓
Collection
   ↓
Document
   ↓
Fields
```

Today we're going one level deeper into the **actual data stored inside DocumentDB**.

By the end of this lesson, you should understand:

* Documents
* Collections
* JSON vs BSON
* BSON data types
* `_id`
* ObjectId
* Nested documents
* Arrays
* Document structure
* Flexible schemas
* Creating and inserting documents
* How DocumentDB represents real application data

---

# 1. What is a Document?

A **document** is an individual record stored in a DocumentDB collection.

For example:

```json
{
  "_id": 101,
  "name": "John",
  "email": "john@example.com",
  "age": 30
}
```

You can think of this as:

```text
One customer
```

In a relational database, you might represent the same information as:

```text
customer_id | name | email              | age
------------------------------------------------
101         | John | john@example.com   | 30
```

In DocumentDB:

```text
One document
        ↓
{
   customer information
}
```

---

# 2. What is a Collection?

Documents are stored inside **collections**.

For example:

```text
Database: ecommerce

customers
│
├── Document 1
├── Document 2
├── Document 3
└── Document 4
```

The collection is:

```text
customers
```

The individual documents are:

```text
Document 1
Document 2
Document 3
Document 4
```

A simple analogy:

```text
Database
   ↓
Collection = folder
   ↓
Document = file
```

This isn't a perfect technical analogy, but it's useful when learning.

---

# 3. Collection vs Table

If you're coming from SQL, this is an important comparison.

| Relational  | DocumentDB |
| ----------- | ---------- |
| Database    | Database   |
| Table       | Collection |
| Row         | Document   |
| Column      | Field      |
| Primary key | `_id`      |

So you might have:

### SQL

```text
customers table

customer_id | name | age
-------------------------
101         | John | 30
102         | Alice| 25
```

### DocumentDB

```json
{
  "_id": 101,
  "name": "John",
  "age": 30
}
```

```json
{
  "_id": 102,
  "name": "Alice",
  "age": 25
}
```

stored in:

```text
customers
```

---

# 4. JSON vs BSON

Now we reach one of the most important concepts of this lesson.

You've probably seen JSON:

```json
{
  "name": "John",
  "age": 30
}
```

Document databases use a JSON-like document model.

But internally, MongoDB-compatible systems use **BSON**.

BSON stands for:

> **Binary JSON**

BSON is a binary representation designed for storing and transmitting document data.

It also supports data types that standard JSON doesn't represent as cleanly.

---

# 5. JSON

JSON stands for:

> **JavaScript Object Notation**

Example:

```json
{
  "name": "John",
  "age": 30,
  "active": true
}
```

JSON has common types such as:

```text
String
Number
Boolean
Array
Object
Null
```

It's human-readable.

---

# 6. BSON

BSON extends the JSON-like model with additional data types.

Conceptually:

```text
JSON
 │
 ├── String
 ├── Number
 ├── Boolean
 ├── Array
 ├── Object
 └── Null
```

BSON adds types such as:

```text
ObjectId
Date
Binary data
Decimal
Timestamp-related types
Regular expressions
```

This becomes particularly useful for databases.

---

# 7. Why does BSON matter?

Imagine you want to store a date.

JSON might represent it as a string:

```json
{
  "createdAt": "2026-08-11"
}
```

But a database benefits from understanding:

> "This value is actually a date."

BSON provides a native date representation.

Conceptually:

```text
createdAt
   ↓
Date type
```

rather than merely:

```text
createdAt
   ↓
String
```

This becomes important when querying, sorting, indexing, and manipulating data.

---

# 8. Common BSON Data Types

You don't need to memorize every BSON type today.

Focus on these:

| BSON type         | Example                     |
| ----------------- | --------------------------- |
| String            | `"John"`                    |
| Int32             | `30`                        |
| Int64             | Large integer               |
| Double            | `99.99`                     |
| Boolean           | `true`                      |
| Null              | `null`                      |
| Array             | `["AWS", "Python"]`         |
| Embedded document | `{ "city": "New Orleans" }` |
| ObjectId          | `ObjectId("...")`           |
| Date              | Date value                  |
| Decimal128        | Precise decimal value       |
| Binary            | Binary data                 |

We'll use several of these throughout the course.

---

# 9. The `_id` Field

Every document needs an identifier.

DocumentDB uses:

```text
_id
```

For example:

```json
{
  "_id": 1001,
  "name": "John"
}
```

Here:

```text
_id = 1001
```

identifies that document.

Another document:

```json
{
  "_id": 1002,
  "name": "Alice"
}
```

So:

```text
customers

_id     name
-------------
1001    John
1002    Alice
```

---

# 10. `_id` must be unique within the collection

Consider:

```json
{
  "_id": 1001,
  "name": "John"
}
```

and:

```json
{
  "_id": 1001,
  "name": "Alice"
}
```

You cannot have two documents with the same `_id` in the same collection.

Think:

```text
customers

_id 1001 → John
_id 1002 → Alice
_id 1003 → Bob
```

The `_id` uniquely identifies the document within that collection.

---

# 11. ObjectId

One very common identifier is:

```text
ObjectId
```

Example:

```javascript
ObjectId("507f1f77bcf86cd799439011")
```

So a document might look like:

```javascript
{
    "_id": ObjectId("507f1f77bcf86cd799439011"),
    "name": "John"
}
```

ObjectId is commonly used as a unique identifier.

If you don't explicitly provide an `_id`, the client/database ecosystem can generate one.

---

# 12. Don't confuse `_id` with your application ID

Suppose your application has:

```text
customerId = "CUST-1001"
```

You could use that as `_id`:

```json
{
  "_id": "CUST-1001",
  "name": "John"
}
```

Or you could have:

```json
{
  "_id": ObjectId("..."),
  "customerId": "CUST-1001",
  "name": "John"
}
```

Both represent different design choices.

We'll discuss data-modeling decisions in **Lesson 6**.

For now:

> `_id` is the database document identifier.

---

# 13. Fields

A document consists of **fields**.

Example:

```json
{
  "_id": 1001,
  "name": "John",
  "age": 30,
  "email": "john@example.com"
}
```

Fields are:

```text
_id
name
age
email
```

And their values are:

```text
1001
John
30
john@example.com
```

Think:

```text
Field → Value
```

---

# 14. Nested Documents

One of the most powerful features of a document model is **nesting**.

Instead of:

```json
{
  "name": "John",
  "city": "New Orleans",
  "state": "Louisiana",
  "zip": "70056"
}
```

you can model the address as an embedded document:

```json
{
  "name": "John",
  "address": {
    "city": "New Orleans",
    "state": "Louisiana",
    "zip": "70056"
  }
}
```

Now:

```text
John
 │
 └── address
       ├── city
       ├── state
       └── zip
```

This is called an:

> **Embedded document**

or

> **Nested document**

---

# 15. Why nested documents are useful

Consider a customer profile:

```json
{
  "_id": "C1001",
  "name": "John",
  "contact": {
    "email": "john@example.com",
    "phone": "504-555-1234"
  },
  "address": {
    "city": "New Orleans",
    "state": "LA"
  }
}
```

The data naturally represents:

```text
Customer
│
├── name
│
├── contact
│    ├── email
│    └── phone
│
└── address
     ├── city
     └── state
```

This is one of the fundamental differences between relational and document-oriented modeling.

---

# 16. Arrays

Documents can contain arrays.

Example:

```json
{
  "_id": "C1001",
  "name": "John",
  "skills": [
    "AWS",
    "Python",
    "DocumentDB"
  ]
}
```

The `skills` field is an array.

Conceptually:

```text
skills
│
├── AWS
├── Python
└── DocumentDB
```

Arrays can also contain objects.

---

# 17. Arrays of Documents

For example, a customer can have multiple orders:

```json
{
  "_id": "C1001",
  "name": "John",
  "orders": [
    {
      "orderId": "O1001",
      "amount": 150
    },
    {
      "orderId": "O1002",
      "amount": 75
    }
  ]
}
```

Now we have:

```text
Customer
│
└── orders
     │
     ├── Order 1
     │     ├── orderId
     │     └── amount
     │
     └── Order 2
           ├── orderId
           └── amount
```

This is extremely useful for representing hierarchical application data.

---

# 18. Documents can have different structures

Here's something very important.

Document databases generally allow flexible document structures.

Document 1:

```json
{
  "_id": 1,
  "name": "John",
  "age": 30
}
```

Document 2:

```json
{
  "_id": 2,
  "name": "Alice",
  "age": 25,
  "premium": true
}
```

Document 3:

```json
{
  "_id": 3,
  "name": "Bob",
  "skills": [
    "AWS",
    "Python"
  ]
}
```

They don't necessarily need to have exactly the same fields.

---

# 19. Flexible schema ≠ No structure

This is a common misconception.

People sometimes say:

> "NoSQL means there is no schema."

That's misleading.

DocumentDB allows flexible document structures, but **your application still needs a sensible data model**.

For example, if half your documents use:

```text
email
```

and the other half use:

```text
emailAddress
```

you can create unnecessary complexity.

So:

```text
Flexible schema
      ≠
No design
```

Good document database design is still extremely important.

---

# 20. Creating a Collection

Using a MongoDB-compatible shell/client, you can work with collections.

For example:

```javascript
use ecommerce
```

Then you can work with:

```javascript
db.customers
```

The collection is:

```text
customers
```

You don't have to think of collections exactly like SQL tables, but the analogy is useful initially.

---

# 21. Inserting a Document

A common MongoDB-compatible operation is:

```javascript
db.customers.insertOne({
    "_id": "C1001",
    "name": "John",
    "age": 30,
    "email": "john@example.com"
})
```

Conceptually:

```text
insertOne()
     ↓
customers collection
     ↓
new document
```

After insertion:

```text
customers

C1001 → John
```

---

# 22. Inserting Multiple Documents

You can insert multiple documents.

For example:

```javascript
db.customers.insertMany([
    {
        "_id": "C1001",
        "name": "John",
        "age": 30
    },
    {
        "_id": "C1002",
        "name": "Alice",
        "age": 25
    },
    {
        "_id": "C1003",
        "name": "Bob",
        "age": 40
    }
])
```

Now:

```text
customers
│
├── C1001 → John
├── C1002 → Alice
└── C1003 → Bob
```

We'll cover CRUD operations systematically in **Lesson 3**.

---

# 23. A realistic document

Let's create a more realistic customer document:

```json
{
  "_id": "C1001",
  "name": "John Smith",
  "email": "john@example.com",
  "age": 30,
  "active": true,

  "address": {
    "street": "123 Main Street",
    "city": "New Orleans",
    "state": "LA",
    "zip": "70112"
  },

  "skills": [
    "AWS",
    "Python",
    "DocumentDB"
  ],

  "orders": [
    {
      "orderId": "O1001",
      "amount": 150,
      "status": "SHIPPED"
    },
    {
      "orderId": "O1002",
      "amount": 75,
      "status": "PROCESSING"
    }
  ]
}
```

Look at how much information can naturally exist inside a single document.

---

# 24. Visualize that document

```text
Customer
│
├── _id
│
├── name
│
├── email
│
├── age
│
├── active
│
├── address
│    ├── street
│    ├── city
│    ├── state
│    └── zip
│
├── skills
│    ├── AWS
│    ├── Python
│    └── DocumentDB
│
└── orders
     │
     ├── Order
     │    ├── orderId
     │    ├── amount
     │    └── status
     │
     └── Order
          ├── orderId
          ├── amount
          └── status
```

This is the power of the document model.

---

# 25. Dot notation

When data is nested, you'll eventually need to reference nested fields.

For example:

```json
{
  "name": "John",
  "address": {
    "city": "New Orleans"
  }
}
```

The nested field can conceptually be represented as:

```text
address.city
```

So:

```text
address
   └── city
```

becomes:

```text
address.city
```

You'll use this heavily when querying nested documents.

---

# 26. Arrays and nested objects together

Consider:

```json
{
  "name": "John",
  "orders": [
    {
      "orderId": "O1001",
      "product": {
        "name": "Laptop",
        "price": 999
      }
    }
  ]
}
```

Now we have multiple levels:

```text
orders
   ↓
array
   ↓
document
   ↓
product
   ↓
name
```

Document-oriented applications often have structures like this.

---

# 27. Document size

Documents aren't infinitely large.

DocumentDB has limits on document size, so you shouldn't treat one document as an unlimited container for an application's entire dataset.

For practical architecture:

> A document should represent a useful unit of application data.

For example:

```text
Good:
Customer profile + appropriate customer information
```

rather than:

```text
Bad:
Entire company's customer database inside one document
```

We'll discuss how to decide what belongs together in **Lesson 6 — Data Modeling**.

---

# 28. When should data be embedded?

Suppose a customer has an address.

You might embed:

```json
{
  "name": "John",
  "address": {
    "city": "New Orleans",
    "state": "LA"
  }
}
```

This can make sense when the address naturally belongs to the customer and is usually accessed with the customer.

But if another part of the application needs to independently manage thousands of addresses, you might consider a different design.

The general decision is:

```text
Should this data live inside the document?
             │
       Depends on
             │
      Access patterns
             │
      Update patterns
             │
     Relationship size
             │
      Data ownership
```

We'll go deep into this in Lesson 6.

---

# 29. JSON/BSON Mental Model

Keep this picture:

```text
Application
     │
     │ JSON-like data
     ▼
MongoDB-compatible API
     │
     ▼
DocumentDB
     │
     ▼
BSON-style document representation
```

You typically work with documents through drivers/tools using JSON-like syntax.

You don't manually convert every document into binary BSON yourself.

The driver/database handles the representation.

---

# 30. Important distinction: JSON isn't BSON

Don't say:

> "BSON is just JSON."

A better answer is:

> **BSON is a binary, JSON-like document representation that supports additional data types.**

For an interview, that's a much stronger answer.

---

# 31. Example: E-commerce Product

Let's model a product:

```json
{
  "_id": "P1001",
  "name": "Laptop",
  "brand": "ExampleTech",
  "price": 1299.99,
  "inStock": true,

  "categories": [
    "electronics",
    "computers"
  ],

  "specifications": {
    "ram": "16GB",
    "storage": "1TB SSD",
    "screenSize": 15.6
  },

  "tags": [
    "business",
    "portable",
    "developer"
  ]
}
```

Identify the different data structures:

```text
_id             → String
name            → String
brand           → String
price           → Number
inStock         → Boolean
categories      → Array
specifications  → Embedded document
tags            → Array
```

This is exactly the kind of document you'll encounter in real applications.

---

# 32. Relational vs Document representation

Let's compare the same product.

### Relational approach

You might have:

```text
products
```

```text
product_id | name   | brand        | price
-------------------------------------------
P1001      | Laptop | ExampleTech  | 1299.99
```

Then perhaps additional tables:

```text
product_categories
product_specifications
product_tags
```

Potentially requiring relationships between tables.

### Document approach

You can represent the product as:

```json
{
  "_id": "P1001",
  "name": "Laptop",
  "brand": "ExampleTech",
  "price": 1299.99,
  "categories": [
    "electronics",
    "computers"
  ],
  "specifications": {
    "ram": "16GB",
    "storage": "1TB SSD"
  }
}
```

The related information can be represented together.

---

# 33. But embedding everything isn't always good

This is **very important**.

Don't take today's lesson and conclude:

> "DocumentDB means put everything inside one giant document."

No.

You need to balance:

```text
Embedding
     vs
Referencing
```

For example:

```text
Customer
   │
   ├── Profile
   │
   └── Orders
```

Should orders be embedded?

Maybe.

Should 10 million historical orders be embedded into one customer document?

Obviously not.

The correct answer depends on:

* How data is accessed
* How frequently it changes
* Document size
* Cardinality
* Read patterns
* Write patterns

We'll dedicate an entire lesson to this.

---

# 34. The most important concepts from Lesson 2

You should now understand:

```text
Document
   ↓
Individual record

Collection
   ↓
Group of documents

BSON
   ↓
Binary JSON-like representation

_id
   ↓
Document identifier

Embedded document
   ↓
Nested object

Array
   ↓
Multiple values

Flexible schema
   ↓
Documents can have different structures
```

---

# 🧠 Lesson 2 Interview Questions

### Q1. What is a document?

An individual JSON/BSON-style record stored in a DocumentDB collection.

---

### Q2. What is a collection?

A logical grouping of documents.

---

### Q3. What is BSON?

A binary, JSON-like document representation that supports additional data types beyond standard JSON.

---

### Q4. What is `_id`?

The unique identifier of a document within a collection.

---

### Q5. What is an embedded document?

A document nested inside another document.

Example:

```json
{
  "name": "John",
  "address": {
    "city": "New Orleans"
  }
}
```

---

### Q6. Can documents in the same collection have different fields?

Yes. DocumentDB supports flexible document structures.

---

### Q7. Is flexible schema the same as having no schema?

No.

It means the database allows flexibility in document structure, but the application should still follow a deliberate data model.

---

### Q8. What is an array of documents?

For example:

```json
{
  "orders": [
    {
      "orderId": "O1"
    },
    {
      "orderId": "O2"
    }
  ]
}
```

`orders` is an array containing embedded documents.

---

# 📝 Lesson 2 Mini Quiz

Try these yourself before checking your notes:

**1.** What is the difference between a collection and a document?

**2.** What does BSON stand for?

**3.** Why is BSON useful for database applications?

**4.** What is the purpose of `_id`?

**5.** What is an embedded document?

**6.** What is the difference between:

```json
{
  "city": "New Orleans"
}
```

and:

```json
{
  "address": {
    "city": "New Orleans"
  }
}
```

**7.** Can two documents in the same collection have different fields?

**8.** What is an array of documents?

**9.** Why shouldn't you put unlimited application data into one document?

**10.** Explain this structure:

```text
Database
   ↓
Collection
   ↓
Document
   ↓
Field
   ↓
Value
```

---

# 🎯 Lesson 2 — Final Mental Model

If someone asks you in an interview:

> **"Explain the DocumentDB data model."**

A strong answer is:

> **"Amazon DocumentDB uses a document-oriented data model. Data is organized into databases and collections, with individual records stored as JSON-like BSON documents.
Documents contain fields and can contain nested documents and arrays. Each document has an `_id` that identifies it within its collection.
The model is flexible, so documents in a collection don't necessarily have identical structures."**

That's the core of Lesson 2. 🔥

### Next: **Lesson 3 — CRUD Operations**

We'll move from **understanding documents** to **actually working with them**:

```text
INSERT
   ↓
READ
   ↓
UPDATE
   ↓
DELETE
```

We'll learn `insertOne`, `insertMany`, `find`, filters, updates, deletes, and practical DocumentDB examples.
