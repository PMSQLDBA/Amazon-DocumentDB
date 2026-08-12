# Lesson 3 — CRUD Operations

Welcome to **Lesson 3 of the Amazon DocumentDB course**. 🔥

So far:

* **Lesson 1:** DocumentDB Fundamentals
* **Lesson 2:** Documents, Collections & BSON
* **Lesson 3:** **CRUD Operations ← Today**

CRUD is the foundation of working with DocumentDB.

> **C = Create**
> **R = Read**
> **U = Update**
> **D = Delete**

We'll use a MongoDB-compatible shell/driver syntax throughout this lesson.

---

# 1. CRUD Mental Model

Imagine an `employees` collection:

```text
employees
│
├── Employee 1
├── Employee 2
├── Employee 3
└── Employee 4
```

CRUD lets your application:

```text
                 DocumentDB
                     │
       ┌─────────────┼─────────────┐
       │             │             │
     CREATE         READ         UPDATE
       │             │             │
    insert        find()        update()
                                   │
                                   │
                                 DELETE
                                   │
                                delete()
```

---

# 2. Our Example Database

For this lesson, imagine we're building an employee application.

We'll use:

```text
Database: company
Collection: employees
```

Our documents will look like:

```json
{
  "_id": "E1001",
  "name": "John Smith",
  "department": "Engineering",
  "age": 30,
  "salary": 90000,
  "active": true
}
```

---

# 3. CREATE — Inserting Documents

The first CRUD operation is **Create**.

The common operation is:

```javascript
insertOne()
```

Example:

```javascript
db.employees.insertOne({
    "_id": "E1001",
    "name": "John Smith",
    "department": "Engineering",
    "age": 30,
    "salary": 90000,
    "active": true
})
```

Conceptually:

```text
Application
     │
     │ INSERT
     ▼
employees collection
     │
     ▼
New document
```

After insertion:

```text
employees
│
└── E1001
     ├── name
     ├── department
     ├── age
     ├── salary
     └── active
```

---

# 4. Insert Another Employee

```javascript
db.employees.insertOne({
    "_id": "E1002",
    "name": "Alice Brown",
    "department": "Finance",
    "age": 28,
    "salary": 85000,
    "active": true
})
```

Now:

```text
employees
│
├── E1001 → John
└── E1002 → Alice
```

---

# 5. insertMany()

What if we need to insert many documents?

Use:

```javascript
insertMany()
```

Example:

```javascript
db.employees.insertMany([
    {
        "_id": "E1003",
        "name": "Bob Wilson",
        "department": "Engineering",
        "age": 35,
        "salary": 105000,
        "active": true
    },
    {
        "_id": "E1004",
        "name": "Sarah Davis",
        "department": "HR",
        "age": 32,
        "salary": 80000,
        "active": true
    }
])
```

Now our collection contains:

```text
employees
│
├── E1001 John
├── E1002 Alice
├── E1003 Bob
└── E1004 Sarah
```

---

# 6. READ — Finding Documents

Now we want to retrieve data.

The basic operation is:

```javascript
find()
```

For example:

```javascript
db.employees.find()
```

This asks:

> "Give me documents from the employees collection."

You can think of it as:

```text
SELECT * FROM employees;
```

in SQL.

---

# 7. Find One Document

Another useful operation is:

```javascript
findOne()
```

Example:

```javascript
db.employees.findOne({
    "_id": "E1001"
})
```

Conceptually:

```text
Search employees
       │
       ▼
_id = E1001
       │
       ▼
Return John
```

---

# 8. Find by a Field

You don't have to search only by `_id`.

Example:

```javascript
db.employees.find({
    "department": "Engineering"
})
```

This asks:

> Find employees whose department is Engineering.

Possible result:

```text
John
Bob
```

---

# 9. Find by Multiple Conditions

Suppose we want:

> Engineering employees who are older than 30.

We can use:

```javascript
db.employees.find({
    "department": "Engineering",
    "age": {
        "$gt": 30
    }
})
```

Here:

```text
$gt
```

means:

> greater than

So we're asking for:

```text
department = Engineering
AND
age > 30
```

We'll study query operators in depth in **Lesson 4**.

---

# 10. Equality Queries

The simplest filter is equality.

```javascript
db.employees.find({
    "department": "Finance"
})
```

Meaning:

```text
department == Finance
```

Another example:

```javascript
db.employees.find({
    "active": true
})
```

Meaning:

> Find active employees.

---

# 11. Nested Document Queries

Remember our Lesson 2 nested documents?

Suppose we have:

```json
{
  "_id": "E1005",
  "name": "Mike",
  "address": {
    "city": "New Orleans",
    "state": "LA"
  }
}
```

We can query the nested field using dot notation:

```javascript
db.employees.find({
    "address.city": "New Orleans"
})
```

The path is:

```text
address
   ↓
city
```

represented as:

```text
address.city
```

---

# 12. Reading a Single Matching Document

Suppose we have:

```javascript
db.employees.findOne({
    "department": "Finance"
})
```

The database returns one matching document.

Important distinction:

```text
find()
   → returns matching documents

findOne()
   → returns one matching document
```

---

# 13. Projection — Choosing Fields

Sometimes you don't want the entire document.

Suppose:

```json
{
  "_id": "E1001",
  "name": "John",
  "department": "Engineering",
  "age": 30,
  "salary": 90000,
  "address": {
    "city": "New Orleans"
  }
}
```

Maybe your application only needs:

```text
name
department
```

You can use a projection.

For example:

```javascript
db.employees.find(
    {},
    {
        "name": 1,
        "department": 1
    }
)
```

Conceptually:

```text
Full document
     ↓
Projection
     ↓
Only requested fields
```

Projection becomes especially useful when documents contain a lot of information.

---

# 14. Excluding Fields

You can also exclude fields.

For example:

```javascript
db.employees.find(
    {},
    {
        "salary": 0
    }
)
```

This means:

> Return documents but don't include `salary`.

You may encounter inclusion and exclusion projection rules; be careful when mixing them, with `_id` being a notable special case.

---

# 15. UPDATE — Modifying Documents

Now we move to the third CRUD operation:

> **Update**

The commonly used operation is:

```javascript
updateOne()
```

Suppose John has:

```json
{
  "_id": "E1001",
  "name": "John Smith",
  "department": "Engineering",
  "age": 30,
  "salary": 90000
}
```

We want to increase his salary.

We can use:

```javascript
db.employees.updateOne(
    {
        "_id": "E1001"
    },
    {
        "$set": {
            "salary": 95000
        }
    }
)
```

Now:

```text
Before:
salary = 90000

       ↓ UPDATE

After:
salary = 95000
```

---

# 16. `$set`

One of the most important update operators is:

```text
$set
```

It means:

> Set this field to this value.

Example:

```javascript
{
    "$set": {
        "department": "Management"
    }
}
```

It changes the department.

---

# 17. Updating Multiple Fields

You can update multiple fields at once:

```javascript
db.employees.updateOne(
    {
        "_id": "E1001"
    },
    {
        "$set": {
            "department": "Management",
            "salary": 110000,
            "active": true
        }
    }
)
```

Result:

```text
department → Management
salary     → 110000
active     → true
```

---

# 18. Updating a Nested Field

Suppose:

```json
{
  "_id": "E1001",
  "name": "John",
  "address": {
    "city": "New Orleans",
    "state": "LA"
  }
}
```

We want to change the city.

Use dot notation:

```javascript
db.employees.updateOne(
    {
        "_id": "E1001"
    },
    {
        "$set": {
            "address.city": "Baton Rouge"
        }
    }
)
```

Now:

```text
address.city

New Orleans
     ↓
Baton Rouge
```

---

# 19. `$inc` — Incrementing Numbers

Another extremely useful operator is:

```text
$inc
```

Suppose John's salary is:

```text
90000
```

We want to increase it by:

```text
5000
```

We can use:

```javascript
db.employees.updateOne(
    {
        "_id": "E1001"
    },
    {
        "$inc": {
            "salary": 5000
        }
    }
)
```

Result:

```text
90000 + 5000
       ↓
95000
```

This is different from `$set`.

### `$set`

```javascript
"$set": {
    "salary": 95000
}
```

Means:

> Make salary exactly 95,000.

### `$inc`

```javascript
"$inc": {
    "salary": 5000
}
```

Means:

> Increase the existing value by 5,000.

---

# 20. `$unset`

What if you want to remove a field?

Use:

```text
$unset
```

Example:

```javascript
db.employees.updateOne(
    {
        "_id": "E1001"
    },
    {
        "$unset": {
            "temporaryField": ""
        }
    }
)
```

This removes:

```text
temporaryField
```

from the document.

---

# 21. updateMany()

`updateOne()` modifies one matching document.

What if we want to update multiple documents?

Use:

```javascript
updateMany()
```

Example:

```javascript
db.employees.updateMany(
    {
        "department": "Engineering"
    },
    {
        "$set": {
            "active": true
        }
    }
)
```

This says:

> Find Engineering employees and set `active` to `true`.

Conceptually:

```text
Engineering employees
│
├── John  → active = true
├── Bob   → active = true
└── Mike  → active = true
```

---

# 22. DELETE — Removing Documents

The fourth CRUD operation is:

> **Delete**

For one document:

```javascript
deleteOne()
```

Example:

```javascript
db.employees.deleteOne({
    "_id": "E1004"
})
```

This removes employee `E1004`.

---

# 23. deleteMany()

Suppose we want to remove inactive employees.

```javascript
db.employees.deleteMany({
    "active": false
})
```

This deletes all matching documents.

Conceptually:

```text
Inactive employees
       │
       ▼
deleteMany()
       │
       ▼
Documents removed
```

---

# 24. Be CAREFUL with deleteMany()

This is important in real systems.

Consider:

```javascript
db.employees.deleteMany({})
```

The empty filter:

```text
{}
```

matches all documents.

Therefore, this can delete the entire collection's contents.

So always ask yourself:

> "What documents does my filter match?"

before running destructive operations.

---

# 25. CRUD Summary

Here's your core cheat sheet:

| Operation   | Method         |
| ----------- | -------------- |
| Create one  | `insertOne()`  |
| Create many | `insertMany()` |
| Read many   | `find()`       |
| Read one    | `findOne()`    |
| Update one  | `updateOne()`  |
| Update many | `updateMany()` |
| Delete one  | `deleteOne()`  |
| Delete many | `deleteMany()` |

Memorize these.

---

# 26. Complete CRUD Example

Let's go through an entire lifecycle.

### Step 1 — Create

```javascript
db.employees.insertOne({
    "_id": "E2001",
    "name": "David",
    "department": "Engineering",
    "age": 29,
    "salary": 85000,
    "active": true
})
```

---

### Step 2 — Read

```javascript
db.employees.findOne({
    "_id": "E2001"
})
```

---

### Step 3 — Update

```javascript
db.employees.updateOne(
    {
        "_id": "E2001"
    },
    {
        "$set": {
            "salary": 90000
        }
    }
)
```

---

### Step 4 — Read again

```javascript
db.employees.findOne({
    "_id": "E2001"
})
```

You'll see:

```text
salary = 90000
```

---

### Step 5 — Delete

```javascript
db.employees.deleteOne({
    "_id": "E2001"
})
```

The document is removed.

That's CRUD.

---

# 27. CRUD Flow

Remember this lifecycle:

```text
              CREATE
                 │
                 ▼
              Document
                 │
                 ▼
               READ
                 │
                 ▼
              UPDATE
                 │
                 ▼
               READ
                 │
                 ▼
              DELETE
```

This is the fundamental interaction between your application and DocumentDB.

---

# 28. CRUD and Application Architecture

A typical application might look like:

```text
                 Web / Mobile App
                        │
                        ▼
                  Application API
                        │
             ┌──────────┼──────────┐
             │          │          │
           CREATE      READ      UPDATE
             │          │          │
             └──────────┼──────────┘
                        │
                        ▼
                   DocumentDB
```

For example:

```text
User registers
     ↓
insertOne()

User views profile
     ↓
findOne()

User changes email
     ↓
updateOne()

User deletes account
     ↓
deleteOne()
```

---

# 29. CRUD vs SQL

If you're familiar with SQL, this comparison helps:

| SQL      | DocumentDB                     |
| -------- | ------------------------------ |
| `INSERT` | `insertOne()` / `insertMany()` |
| `SELECT` | `find()` / `findOne()`         |
| `UPDATE` | `updateOne()` / `updateMany()` |
| `DELETE` | `deleteOne()` / `deleteMany()` |
| Table    | Collection                     |
| Row      | Document                       |
| Column   | Field                          |

But don't assume the systems behave identically.

The query model and data-modeling philosophy are different.

---

# 30. A Practical Example: Customer Application

Suppose we have:

```json
{
  "_id": "C1001",
  "name": "John",
  "email": "john@example.com",
  "active": true,
  "address": {
    "city": "New Orleans",
    "state": "LA"
  }
}
```

### Create customer

```javascript
db.customers.insertOne({
    "_id": "C1001",
    "name": "John",
    "email": "john@example.com",
    "active": true,
    "address": {
        "city": "New Orleans",
        "state": "LA"
    }
})
```

### Read customer

```javascript
db.customers.findOne({
    "_id": "C1001"
})
```

### Change email

```javascript
db.customers.updateOne(
    {
        "_id": "C1001"
    },
    {
        "$set": {
            "email": "john.smith@example.com"
        }
    }
)
```

### Change city

```javascript
db.customers.updateOne(
    {
        "_id": "C1001"
    },
    {
        "$set": {
            "address.city": "Metairie"
        }
    }
)
```

### Delete customer

```javascript
db.customers.deleteOne({
    "_id": "C1001"
})
```

You've now performed the complete lifecycle.

---

# 31. Important Update Operators

For now, focus on these:

### `$set`

Change/create a field.

```javascript
{
    "$set": {
        "name": "Alice"
    }
}
```

### `$inc`

Increment a numeric value.

```javascript
{
    "$inc": {
        "age": 1
    }
}
```

### `$unset`

Remove a field.

```javascript
{
    "$unset": {
        "temporary": ""
    }
}
```

There are many more operators, especially for arrays and conditional updates. We'll encounter them as we progress.

---

# 32. What happens if the filter doesn't match?

Suppose:

```javascript
db.employees.updateOne(
    {
        "_id": "DOES_NOT_EXIST"
    },
    {
        "$set": {
            "active": false
        }
    }
)
```

If no document matches:

```text
Matched documents = 0
Modified documents = 0
```

Nothing is updated.

This is why your filter is extremely important.

---

# 33. Why filters matter so much

Consider:

```javascript
db.employees.updateMany(
    {
        "department": "Engineering"
    },
    {
        "$set": {
            "salary": 100000
        }
    }
)
```

This potentially changes **every Engineering employee**.

Compare:

```javascript
db.employees.updateOne(
    {
        "_id": "E1001"
    },
    {
        "$set": {
            "salary": 100000
        }
    }
)
```

Only one document is targeted.

So when writing updates/deletes:

> **Always understand your filter before executing the operation.**

---

# 34. CRUD + `_id`

The `_id` field is extremely useful for targeted operations.

For example:

```javascript
db.customers.findOne({
    "_id": "C1001"
})
```

```javascript
db.customers.updateOne(
    {
        "_id": "C1001"
    },
    {
        "$set": {
            "active": false
        }
    }
)
```

```javascript
db.customers.deleteOne({
    "_id": "C1001"
})
```

The pattern is:

```text
_id
 ↓
Identify document
 ↓
Read / Update / Delete
```

---

# 35. The Three Most Important Things to Remember

### 1. CRUD

```text
Create → Read → Update → Delete
```

### 2. Methods

```text
insertOne()
findOne()
updateOne()
deleteOne()
```

And their multi-document versions:

```text
insertMany()
find()
updateMany()
deleteMany()
```

### 3. Filter

The filter determines **which documents are affected**.

```javascript
{
    "_id": "E1001"
}
```

or:

```javascript
{
    "department": "Engineering"
}
```

---

# 🧠 Lesson 3 Interview Questions

### Q1. What does CRUD stand for?

**Create, Read, Update, Delete.**

---

### Q2. How do you insert one document?

```javascript
db.collection.insertOne({...})
```

---

### Q3. How do you insert multiple documents?

```javascript
db.collection.insertMany([...])
```

---

### Q4. Difference between `find()` and `findOne()`?

`find()` is used to retrieve matching documents, while `findOne()` returns a single matching document.

---

### Q5. Difference between `updateOne()` and `updateMany()`?

`updateOne()` updates one matching document, while `updateMany()` can update all matching documents.

---

### Q6. What does `$set` do?

It sets a field to a specified value.

---

### Q7. What does `$inc` do?

It increments a numeric field by a specified amount.

---

### Q8. What does `$unset` do?

It removes a field from a document.

---

### Q9. How do you delete one document?

```javascript
db.collection.deleteOne({
    "_id": "E1001"
})
```

---

### Q10. Why should you be careful with `deleteMany()`?

Because the filter determines how many documents are deleted. An overly broad filter can delete many documents, potentially the entire collection.

---

# 📝 Lesson 3 Mini Quiz

Try these without looking back.

### 1.

Write a command to insert this document:

```json
{
  "_id": "P1001",
  "name": "Laptop",
  "price": 1000
}
```

into the `products` collection.

### 2.

How would you retrieve the document with `_id = "P1001"`?

### 3.

How would you change its price to `1200`?

### 4.

How would you increase its price by `100`?

### 5.

How would you remove the `price` field?

### 6.

How would you delete the product?

### 7.

What's the difference between:

```javascript
"$set": {
    "price": 1200
}
```

and:

```javascript
"$inc": {
    "price": 200
}
```

### 8.

What is dangerous about:

```javascript
db.products.deleteMany({})
```

---

# 🎯 Lesson 3 Final Mental Model

You should now be able to visualize an application talking to DocumentDB like this:

```text
                    APPLICATION
                         │
             ┌───────────┼───────────┐
             │           │           │
             ▼           ▼           ▼
          CREATE        READ       UPDATE
             │           │           │
             │           │           │
             └───────────┼───────────┘
                         │
                         ▼
                  DOCUMENTDB
                         │
                    Collection
                         │
                     Documents
                         │
                     Fields
                         │
                         ▼
                      DELETE
```

And your core commands are:

```text
CREATE
  insertOne()
  insertMany()

READ
  find()
  findOne()

UPDATE
  updateOne()
  updateMany()

DELETE
  deleteOne()
  deleteMany()
```

## 🔥 One interview-level takeaway

If you're asked:

> **"How does an application perform CRUD operations in Amazon DocumentDB?"**

A strong answer is:

> **"An application uses DocumentDB's MongoDB-compatible APIs through a driver or compatible client. It can create documents with `insertOne` or `insertMany`, retrieve them with `find` or `findOne`, modify them with `updateOne` or `updateMany` using update operators such as `$set` and `$inc`, and remove them with `deleteOne` or `deleteMany`. Query filters determine which documents are affected."**

### Next: **Lesson 4 — Queries & Operators** 🔎

We'll go much deeper into **how to search DocumentDB data**: `$gt`, `$gte`, `$lt`, `$lte`, `$in`, `$nin`, `$and`, `$or`, `$ne`, arrays, nested fields, regular expressions, sorting, limiting, and more.
