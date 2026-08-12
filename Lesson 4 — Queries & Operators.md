# Lesson 4 — Queries & Operators 🔎

Welcome to **Lesson 4 of the Amazon DocumentDB course**.

So far:

1. DocumentDB Fundamentals
2. Documents, Collections & BSON
3. CRUD Operations
4. **Queries & Operators ← Today**

This lesson is about becoming comfortable with **finding exactly the documents you want**.

---

# 1. What is a Query?

A **query** tells DocumentDB:

> "Which documents am I looking for?"

Suppose we have:

```text
employees
│
├── John   Engineering   age 30   salary 90000
├── Alice  Finance       age 28   salary 85000
├── Bob    Engineering   age 35   salary 105000
└── Sarah  HR            age 32   salary 80000
```

A query might ask:

> Find employees in Engineering.

```javascript
db.employees.find({
    "department": "Engineering"
})
```

Result:

```text
John
Bob
```

---

# 2. Query Anatomy

A basic query looks like:

```javascript
db.collection.find({
    field: value
})
```

For example:

```javascript
db.employees.find({
    "department": "Engineering"
})
```

Break it down:

```text
db
 ↓
employees
 ↓
find()
 ↓
filter
 ↓
department = Engineering
```

The object inside `find()` is called the **query filter**.

---

# 3. Equality Queries

The simplest query is equality.

```javascript
db.employees.find({
    "department": "Engineering"
})
```

Meaning:

```text
department == "Engineering"
```

Another:

```javascript
db.employees.find({
    "active": true
})
```

Meaning:

> Find all active employees.

And:

```javascript
db.employees.find({
    "age": 30
})
```

Meaning:

> Find employees whose age is 30.

---

# 4. Multiple Conditions — AND

Suppose we want:

> Engineering employees who are active.

We can write:

```javascript
db.employees.find({
    "department": "Engineering",
    "active": true
})
```

This means:

```text
department = Engineering
AND
active = true
```

Conceptually:

```text
Engineering?
   │
   ├── No → exclude
   │
   └── Yes
        │
        ▼
     Active?
        │
        ├── No → exclude
        │
        └── Yes → return
```

---

# 5. Comparison Operators

Now things get more powerful.

DocumentDB supports MongoDB-compatible query operators such as:

```text
$gt
$gte
$lt
$lte
$eq
$ne
$in
$nin
```

Let's learn them one by one.

---

# 6. `$gt` — Greater Than

`$gt` means:

> **greater than**

Suppose:

```text
John → age 30
Bob  → age 35
Sarah → age 32
```

Query:

```javascript
db.employees.find({
    "age": {
        "$gt": 30
    }
})
```

Meaning:

```text
age > 30
```

Result:

```text
Bob
Sarah
```

John is excluded because:

```text
30 > 30 = false
```

---

# 7. `$gte` — Greater Than or Equal

`$gte` means:

> **greater than or equal to**

```javascript
db.employees.find({
    "age": {
        "$gte": 30
    }
})
```

Meaning:

```text
age >= 30
```

Now John is included:

```text
John 30 ✓
Bob 35  ✓
Sarah 32 ✓
```

---

# 8. `$lt` — Less Than

`$lt` means:

> **less than**

```javascript
db.employees.find({
    "age": {
        "$lt": 30
    }
})
```

Meaning:

```text
age < 30
```

Alice:

```text
28
```

matches.

---

# 9. `$lte` — Less Than or Equal

`$lte` means:

> **less than or equal to**

```javascript
db.employees.find({
    "age": {
        "$lte": 30
    }
})
```

Meaning:

```text
age <= 30
```

John at age 30 matches.

---

# 10. `$eq` — Equal

You can explicitly use `$eq`.

```javascript
db.employees.find({
    "age": {
        "$eq": 30
    }
})
```

This means:

```text
age == 30
```

However, for simple equality, you usually don't need `$eq`.

This:

```javascript
{
    "age": 30
}
```

is simpler.

---

# 11. `$ne` — Not Equal

`$ne` means:

> **not equal**

For example:

```javascript
db.employees.find({
    "department": {
        "$ne": "HR"
    }
})
```

Meaning:

```text
department != HR
```

So Engineering and Finance employees can match.

---

# 12. `$in` — Match Any Value in a List

Suppose we want employees in either:

```text
Engineering
Finance
```

Use:

```javascript
db.employees.find({
    "department": {
        "$in": [
            "Engineering",
            "Finance"
        ]
    }
})
```

Meaning:

```text
department == Engineering
OR
department == Finance
```

This is extremely useful.

---

# 13. `$nin` — Not In

`$nin` means:

> value is not in the specified list.

Example:

```javascript
db.employees.find({
    "department": {
        "$nin": [
            "HR",
            "Finance"
        ]
    }
})
```

Meaning:

```text
department is NOT HR
AND
department is NOT Finance
```

---

# 14. Comparison Operator Cheat Sheet

Memorize this:

| Operator | Meaning                             |
| -------- | ----------------------------------- |
| `$gt`    | Greater than                        |
| `$gte`   | Greater than or equal               |
| `$lt`    | Less than                           |
| `$lte`   | Less than or equal                  |
| `$eq`    | Equal                               |
| `$ne`    | Not equal                           |
| `$in`    | Matches one of the specified values |
| `$nin`   | Doesn't match the specified values  |

---

# 15. Combining Conditions

Suppose we want:

> Engineering employees with salary greater than $90,000.

```javascript
db.employees.find({
    "department": "Engineering",
    "salary": {
        "$gt": 90000
    }
})
```

This means:

```text
department = Engineering
AND
salary > 90000
```

Possible result:

```text
Bob
```

---

# 16. `$and`

You can explicitly use `$and`.

```javascript
db.employees.find({
    "$and": [
        {
            "department": "Engineering"
        },
        {
            "salary": {
                "$gt": 90000
            }
        }
    ]
})
```

Meaning:

```text
Condition 1
    AND
Condition 2
```

For simple conditions, the shorter form is usually easier:

```javascript
db.employees.find({
    "department": "Engineering",
    "salary": {
        "$gt": 90000
    }
})
```

---

# 17. `$or`

Now suppose we want:

> Employees who are either in Engineering OR Finance.

Use:

```javascript
db.employees.find({
    "$or": [
        {
            "department": "Engineering"
        },
        {
            "department": "Finance"
        }
    ]
})
```

Conceptually:

```text
             Employee
                │
          ┌─────┴─────┐
          │           │
    Engineering    Finance
          │           │
          └─────┬─────┘
                │
              MATCH
```

---

# 18. `$nor`

`$nor` means:

> None of the specified conditions should match.

For example:

```javascript
db.employees.find({
    "$nor": [
        {
            "department": "HR"
        },
        {
            "department": "Finance"
        }
    ]
})
```

This excludes employees matching either condition.

---

# 19. Logical Operators Cheat Sheet

| Operator | Meaning                             |
| -------- | ----------------------------------- |
| `$and`   | All conditions must match           |
| `$or`    | At least one condition must match   |
| `$nor`   | None of the conditions should match |
| `$not`   | Negates a condition                 |

---

# 20. Nested Field Queries

Let's use:

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

To query city:

```javascript
db.employees.find({
    "address.city": "New Orleans"
})
```

Remember:

```text
address
   ↓
city

address.city
```

This is called **dot notation**.

---

# 21. Nested Field with Comparison

Suppose:

```json
{
  "name": "John",
  "address": {
    "zip": 70112
  }
}
```

You could query:

```javascript
db.employees.find({
    "address.zip": {
        "$gte": 70000
    }
})
```

Meaning:

```text
address.zip >= 70000
```

---

# 22. Querying Arrays

Suppose a document contains:

```json
{
  "_id": "E1001",
  "name": "John",
  "skills": [
    "AWS",
    "Python",
    "DocumentDB"
  ]
}
```

You can query for a value in the array:

```javascript
db.employees.find({
    "skills": "AWS"
})
```

This can match documents whose `skills` array contains `"AWS"`.

---

# 23. `$all`

Suppose we want employees who have **both**:

```text
AWS
Python
```

We can use:

```javascript
db.employees.find({
    "skills": {
        "$all": [
            "AWS",
            "Python"
        ]
    }
})
```

This asks for documents whose array contains all specified values.

---

# 24. `$size`

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

There are three skills.

A query using `$size` can match arrays of a specified length:

```javascript
db.employees.find({
    "skills": {
        "$size": 3
    }
})
```

Meaning:

```text
skills array contains exactly 3 elements
```

---

# 25. Arrays of Documents

Now consider:

```json
{
  "_id": "C1001",
  "name": "John",
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

You might query nested values such as:

```javascript
db.customers.find({
    "orders.orderId": "O1001"
})
```

The path is:

```text
orders
  ↓
orderId
```

represented as:

```text
orders.orderId
```

---

# 26. `$elemMatch`

Now imagine we want:

> Find customers who have an order where **amount > 100 AND status = SHIPPED**.

This is where `$elemMatch` can be useful when matching multiple conditions against the same array element.

Conceptually:

```javascript
db.customers.find({
    "orders": {
        "$elemMatch": {
            "amount": {
                "$gt": 100
            },
            "status": "SHIPPED"
        }
    }
})
```

The important idea is:

```text
orders array
     │
     ├── Order 1
     │    amount > 100 ✓
     │    status = SHIPPED ✓
     │
     └── Order 2
```

`$elemMatch` lets you express conditions that must apply to the **same array element**.

---

# 27. Why `$elemMatch` Matters

Without understanding array matching, you can accidentally write a query whose conditions aren't constrained to the same array element.

For example, imagine:

```json
{
  "orders": [
    {
      "amount": 200,
      "status": "PROCESSING"
    },
    {
      "amount": 50,
      "status": "SHIPPED"
    }
  ]
}
```

There is:

```text
an order with amount > 100
```

and:

```text
an order with status = SHIPPED
```

But there isn't one order satisfying **both**.

`$elemMatch` is the tool to express:

> Both conditions must be true for one array element.

---

# 28. `$exists`

You may want to find documents where a field exists.

Example:

```javascript
db.employees.find({
    "phone": {
        "$exists": true
    }
})
```

Meaning:

> Find documents where `phone` exists.

You can also test for non-existence:

```javascript
db.employees.find({
    "phone": {
        "$exists": false
    }
})
```

---

# 29. Null vs Missing

This is an important concept.

These aren't necessarily the same:

### Field exists with null

```json
{
  "phone": null
}
```

### Field doesn't exist

```json
{
  "name": "John"
}
```

There is no `phone` field at all.

When designing queries, understand whether you mean:

```text
field exists
```

or:

```text
field exists and contains null
```

This distinction becomes important in real applications.

---

# 30. Regular Expressions

You may sometimes need pattern-based searches.

For example:

> Find names beginning with "Jo".

A MongoDB-compatible query can use a regular expression:

```javascript
db.employees.find({
    "name": /^Jo/
})
```

Potential matches:

```text
John
Joseph
Jordan
```

Be careful with regex queries in production, especially when they aren't supported efficiently by an appropriate index.

We'll discuss performance and indexes in **Lesson 5**.

---

# 31. Sorting

Queries aren't only about filtering.

You may want to sort results.

For example:

```javascript
db.employees.find().sort({
    "salary": 1
})
```

Here:

```text
1 = ascending
```

So:

```text
80000
85000
90000
105000
```

---

# 32. Descending Sort

Use:

```text
-1
```

Example:

```javascript
db.employees.find().sort({
    "salary": -1
})
```

Result:

```text
105000
90000
85000
80000
```

Remember:

```text
1  → ascending
-1 → descending
```

---

# 33. Limiting Results

Suppose you only want the first five documents.

Use:

```javascript
db.employees.find().limit(5)
```

Conceptually:

```text
All matching documents
        ↓
     limit(5)
        ↓
First 5
```

This is extremely useful for pagination and APIs.

---

# 34. Combining Sort and Limit

Suppose we want:

> Top 3 highest-paid employees.

We can combine:

```javascript
db.employees.find()
    .sort({
        "salary": -1
    })
    .limit(3)
```

Conceptually:

```text
All employees
     ↓
Sort salary DESC
     ↓
105000
100000
95000
90000
85000
     ↓
limit(3)
     ↓
Top 3
```

---

# 35. Skip

You may also encounter:

```javascript
skip()
```

For example:

```javascript
db.employees.find()
    .sort({
        "name": 1
    })
    .skip(10)
    .limit(10)
```

Conceptually:

```text
Sort
 ↓
Skip first 10
 ↓
Return next 10
```

This is a basic pagination pattern.

For large datasets, however, pagination strategy and indexing matter greatly. We'll revisit that in Lesson 5.

---

# 36. Projection + Filtering

You can combine a filter with projection.

For example:

```javascript
db.employees.find(
    {
        "department": "Engineering"
    },
    {
        "name": 1,
        "salary": 1
    }
)
```

Meaning:

> Find Engineering employees but return only their name and salary.

Think:

```text
Filter
  ↓
Engineering employees
  ↓
Projection
  ↓
name + salary
```

---

# 37. A Real-World Query

Let's build one.

Requirement:

> Find active Engineering employees older than 30, show only their names and salaries, sort by salary descending, and return the top 5.

Query:

```javascript
db.employees.find(
    {
        "department": "Engineering",
        "active": true,
        "age": {
            "$gt": 30
        }
    },
    {
        "name": 1,
        "salary": 1
    }
)
.sort({
    "salary": -1
})
.limit(5)
```

Break it down:

```text
FILTER
│
├── department = Engineering
├── active = true
└── age > 30
        │
        ▼
PROJECTION
│
├── name
└── salary
        │
        ▼
SORT
│
└── salary DESC
        │
        ▼
LIMIT
│
└── 5
```

That's a very realistic application query.

---

# 38. Query Operators Cheat Sheet

Keep this table handy:

### Comparison

```text
$gt
$gte
$lt
$lte
$eq
$ne
$in
$nin
```

### Logical

```text
$and
$or
$nor
$not
```

### Array

```text
$all
$size
$elemMatch
```

### Element

```text
$exists
$type
```

You'll encounter additional operators as we progress.

---

# 39. Querying Nested Data — Important Pattern

Suppose:

```json
{
  "customer": {
    "name": "John",
    "address": {
      "city": "New Orleans"
    }
  }
}
```

To query the city:

```javascript
db.customers.find({
    "customer.address.city": "New Orleans"
})
```

The pattern is:

```text
object.field.subfield
```

For example:

```text
customer
   ↓
address
   ↓
city

customer.address.city
```

---

# 40. Querying Arrays — Important Pattern

Suppose:

```json
{
  "skills": [
    "AWS",
    "Python",
    "DocumentDB"
  ]
}
```

Simple membership:

```javascript
{
    "skills": "AWS"
}
```

Multiple required values:

```javascript
{
    "skills": {
        "$all": ["AWS", "Python"]
    }
}
```

Specific array length:

```javascript
{
    "skills": {
        "$size": 3
    }
}
```

Array of objects with conditions on the same element:

```javascript
{
    "orders": {
        "$elemMatch": {
            "amount": {
                "$gt": 100
            },
            "status": "SHIPPED"
        }
    }
}
```

These patterns are worth practicing.

---

# 41. Query Performance Warning ⚠️

This lesson is about **how queries work**.

But there's another question:

> **How efficiently does DocumentDB execute the query?**

That's where **indexes** come in.

Suppose you have:

```text
10 documents
```

Searching all of them might be fine.

But imagine:

```text
100 million documents
```

and you repeatedly query:

```javascript
{
    "email": "john@example.com"
}
```

Without an appropriate index, the database may have to inspect a large number of documents.

That's why our next lesson is:

> **Lesson 5 — Indexes & Query Performance**

---

# 42. Query vs Index

Keep these concepts separate.

### Query

Answers:

> **What data do I want?**

Example:

```javascript
{
    "email": "john@example.com"
}
```

### Index

Helps answer:

> **How can the database find that data efficiently?**

Think:

```text
Query
  ↓
"What?"
  ↓
email = John

Index
  ↓
"How quickly?"
```

We'll study this next.

---

# 🧠 Lesson 4 Interview Questions

### Q1. What is a query filter?

The condition that determines which documents should be returned or affected.

---

### Q2. What does `$gt` mean?

Greater than.

```javascript
{
    "age": {
        "$gt": 30
    }
}
```

means:

```text
age > 30
```

---

### Q3. `$gt` vs `$gte`?

```text
$gt  → >
$gte → >=
```

---

### Q4. What does `$in` do?

Matches documents where a field contains one of the specified values.

Example:

```javascript
{
    "department": {
        "$in": ["Engineering", "Finance"]
    }
}
```

---

### Q5. `$or` vs `$and`?

```text
$and → all conditions must match
$or  → at least one condition must match
```

---

### Q6. How do you query a nested field?

Use dot notation:

```javascript
{
    "address.city": "New Orleans"
}
```

---

### Q7. What does `$elemMatch` do?

It allows multiple conditions to be applied to the **same element of an array**, which is particularly useful for arrays of embedded documents.

---

### Q8. What does `$exists` do?

Tests whether a field exists.

---

### Q9. How do you sort descending?

```javascript
.sort({
    "salary": -1
})
```

---

### Q10. How do you return only five results?

```javascript
.limit(5)
```

---

# 📝 Lesson 4 Mini Quiz

Try these before checking your notes.

### 1.

Find employees whose salary is greater than `100000`.

### 2.

Find employees whose age is between 25 and 35 inclusive.

### 3.

Find employees in either Engineering or Finance.

### 4.

Find employees who aren't in HR.

### 5.

Given:

```json
{
  "name": "John",
  "address": {
    "city": "New Orleans"
  }
}
```

Write a query that finds John based on the city.

### 6.

Given:

```json
{
  "skills": ["AWS", "Python", "DocumentDB"]
}
```

Find documents containing both AWS and Python.

### 7.

Find documents where `phone` exists.

### 8.

Return employees sorted by salary from highest to lowest.

### 9.

Return only the top 10 highest-paid employees.

### 10.

Explain the difference between:

```text
Query
```

and:

```text
Index
```

---

# 🎯 Lesson 4 Final Mental Model

When you're writing a DocumentDB query, think in layers:

```text
                QUERY
                  │
        ┌─────────┴─────────┐
        │                   │
     FILTER             PROJECTION
        │                   │
   "Which docs?"       "Which fields?"
        │
        ▼
     SORT
        │
        ▼
     LIMIT
```

And the filter itself can use:

```text
Equality
   │
   ├── department: "Engineering"
   │
Comparison
   │
   ├── $gt
   ├── $gte
   ├── $lt
   ├── $lte
   │
Logical
   │
   ├── $and
   ├── $or
   └── $nor
   │
Arrays
   │
   ├── $all
   ├── $size
   └── $elemMatch
   │
Nested fields
   │
   └── dot notation
```

## 🔥 The interview takeaway

If asked:

> **"How do you query data in Amazon DocumentDB?"**

A strong answer is:

> **"DocumentDB provides MongoDB-compatible query APIs. Queries use filters to match documents and support equality, comparison, logical, array, and nested-field operators. Results can also be projected, sorted, skipped, and limited. For example, `$gt` performs greater-than comparisons, `$in` matches values from a list, `$or` combines alternatives, and dot notation accesses nested fields. Appropriate indexes are then used to make frequently executed queries efficient."**

**Next → Lesson 5: Indexes & Query Performance ⚡**

That's where we'll connect today's queries to **how DocumentDB actually finds the data efficiently**, including **single-field indexes, compound indexes, index selection, `explain()`, and common performance mistakes**.
