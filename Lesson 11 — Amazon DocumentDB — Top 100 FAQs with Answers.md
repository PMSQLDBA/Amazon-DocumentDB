# Amazon DocumentDB — Top 100 FAQs with Answers

Below is a **100-question interview + practical FAQ bank** covering the full 10-lesson Amazon DocumentDB course: 

fundamentals, BSON, CRUD, queries, indexes, modeling, AWS architecture, security, backups, monitoring, operations, and production design.

Each answer has **5 concise points** so you can use this as a revision sheet.

---

# 1. What is Amazon DocumentDB?

**Answer:**

1. Amazon DocumentDB is a **managed document database service from AWS**.
2. It is designed to be **compatible with MongoDB APIs and drivers**.
3. It stores data as flexible, JSON-like **documents**.
4. It separates database compute from distributed cluster storage.
5. AWS manages much of the underlying infrastructure, availability, backups, and maintenance.

---

# 2. Is Amazon DocumentDB the same as MongoDB?

**Answer:**

1. No, DocumentDB and MongoDB are **different database products**.
2. DocumentDB provides MongoDB compatibility for many common APIs and operations.
3. Not every MongoDB feature or behavior is necessarily supported identically.
4. Applications should therefore be tested specifically against DocumentDB.
5. Always check the current DocumentDB compatibility documentation before using advanced MongoDB features.

---

# 3. Why would you choose DocumentDB?

**Answer:**

1. It provides a managed document-database experience on AWS.
2. It is useful for applications that naturally represent data as documents.
3. It integrates with AWS networking, security, monitoring, IAM, KMS, and backup services.
4. It can provide high availability through replicas and failover.
5. It reduces the operational burden compared with managing database servers yourself.

---

# 4. What is a document database?

**Answer:**

1. A document database stores records as **documents**.
2. Documents commonly use JSON-like structures.
3. Documents can contain nested objects and arrays.
4. Different documents can have flexible structures.
5. This model is useful for application data that naturally maps to objects.

---

# 5. What is BSON?

**Answer:**

1. BSON means **Binary JSON**.
2. It is a binary representation of JSON-like documents.
3. BSON supports types beyond standard JSON, such as `ObjectId` and dates.
4. It is commonly associated with MongoDB-compatible document databases.
5. DocumentDB applications commonly work with BSON through MongoDB-compatible drivers and APIs.

---

# 6. What is a collection?

**Answer:**

1. A collection is a logical group of documents.
2. It is broadly comparable to a table in a relational database.
3. Documents within a collection can have flexible structures.
4. Example collections include `customers`, `products`, and `orders`.
5. Queries and indexes are generally created against collections.

---

# 7. What is a document?

**Answer:**

1. A document is an individual record stored in a collection.
2. It contains fields and values.
3. Values can include strings, numbers, arrays, objects, dates, and other supported types.
4. Documents can contain nested structures.
5. Example: one customer can be represented by one customer document.

---

# 8. What is `_id`?

**Answer:**

1. `_id` identifies a document within a collection.
2. It is commonly represented using `ObjectId`.
3. The `_id` field is automatically generated when appropriate.
4. It is indexed by default.
5. Applications can also use their own identifier values when suitable.

---

# 9. What is ObjectId?

**Answer:**

1. `ObjectId` is a BSON identifier type.
2. It is commonly used for document `_id` values.
3. It is designed to provide unique identifiers.
4. Drivers typically provide APIs for creating and handling ObjectIds.
5. Applications should consistently treat identifier types rather than mixing strings and ObjectIds accidentally.

---

# 10. How is DocumentDB different from a relational database?

**Answer:**

1. Relational databases organize data into tables and rows.
2. DocumentDB organizes data into collections and documents.
3. DocumentDB supports nested objects and arrays naturally.
4. Relational designs commonly emphasize normalized relationships and joins.
5. Document modeling instead emphasizes application access patterns and appropriate embedding/reference decisions.

---

# 11. What is CRUD?

**Answer:**

1. CRUD means **Create, Read, Update, Delete**.
2. Create inserts documents.
3. Read retrieves documents.
4. Update modifies documents.
5. Delete removes documents.

---

# 12. How do you insert one document?

**Answer:**

1. Use the MongoDB-compatible `insertOne()` operation.
2. Pass a document containing the fields you want to store.
3. DocumentDB generates an `_id` when appropriate.
4. The operation returns information about the inserted document.
5. Validate required application fields before inserting.

Example:

```javascript
db.customers.insertOne({
  customerId: "C1001",
  name: "John",
  email: "john@example.com"
})
```

---

# 13. How do you insert multiple documents?

**Answer:**

1. Use `insertMany()`.
2. Provide an array of documents.
3. It is useful for batch-style inserts.
4. The application should handle errors appropriately.
5. Batch size should be designed according to workload and application requirements.

---

# 14. How do you read a document?

**Answer:**

1. Use `find()` to retrieve matching documents.
2. Use `findOne()` when you need one matching document.
3. Provide a filter document.
4. You can filter on nested fields and supported operators.
5. Index frequently queried fields for better performance.

---

# 15. How do you update a document?

**Answer:**

1. Use operations such as `updateOne()`.
2. Specify which document should be updated.
3. Use update operators such as `$set`.
4. Avoid unintentionally replacing the entire document.
5. Check the update result to confirm what changed.

---

# 16. How do you delete a document?

**Answer:**

1. Use `deleteOne()` to remove one matching document.
2. Use `deleteMany()` for multiple matching documents.
3. Always carefully verify the filter.
4. Broad delete operations can cause significant data loss.
5. Production applications should protect destructive operations with authorization and safeguards.

---

# 17. What is a query filter?

**Answer:**

1. A query filter specifies which documents should match.
2. Example: `{ customerId: "C1001" }`.
3. Filters can contain multiple conditions.
4. Operators such as `$gt`, `$lt`, `$in`, and `$exists` extend filtering.
5. Good filters combined with appropriate indexes improve query performance.

---

# 18. What is projection?

**Answer:**

1. Projection controls which fields are returned.
2. It can reduce unnecessary data transfer.
3. It can make application responses smaller.
4. It can also help avoid returning sensitive fields unnecessarily.
5. Projection should be designed around the application's response requirements.

---

# 19. What is `$eq`?

**Answer:**

1. `$eq` represents equality comparison.
2. It matches documents where a field equals a specified value.
3. Example: `{ price: { $eq: 100 } }`.
4. Simple equality filters can often be expressed without `$eq`.
5. Appropriate indexes can make equality searches efficient.

---

# 20. What is `$gt`?

**Answer:**

1. `$gt` means **greater than**.
2. It is used for range queries.
3. Example: `{ price: { $gt: 100 } }`.
4. It is useful for prices, dates, scores, and quantities.
5. Range queries should be considered when designing indexes.

---

# 21. What is `$gte`?

**Answer:**

1. `$gte` means **greater than or equal to**.
2. It includes the boundary value.
3. Example: `{ inventory: { $gte: 10 } }`.
4. It is useful for threshold-based filtering.
5. It can also be used in conditional atomic updates.

---

# 22. What is `$lt`?

**Answer:**

1. `$lt` means **less than**.
2. It is useful for range filtering.
3. Example: `{ price: { $lt: 50 } }`.
4. It can be combined with `$gte` for bounded ranges.
5. Index design should consider frequent range queries.

---

# 23. What is `$lte`?

**Answer:**

1. `$lte` means **less than or equal to**.
2. It includes the specified boundary.
3. Example: `{ price: { $lte: 50 } }`.
4. It is useful for maximum-value filtering.
5. It is commonly combined with other query conditions.

---

# 24. What is `$in`?

**Answer:**

1. `$in` matches a field against multiple possible values.
2. Example: `{ status: { $in: ["PLACED", "SHIPPED"] } }`.
3. It is useful when the application accepts a set of valid values.
4. Indexing may improve appropriate `$in` queries.
5. Very large `$in` lists should be evaluated carefully for performance.

---

# 25. What is `$exists`?

**Answer:**

1. `$exists` checks whether a field exists.
2. It is useful for flexible document structures.
3. Example: `{ phone: { $exists: true } }`.
4. It can help identify documents with optional fields.
5. Overusing schema flexibility can make application validation harder.

---

# 26. What are logical operators?

**Answer:**

1. Logical operators combine query conditions.
2. Common operators include `$and`, `$or`, and `$not`.
3. `$or` matches when at least one condition is satisfied.
4. `$and` requires multiple conditions.
5. Complex logical queries should be tested with realistic data volumes.

---

# 27. What is a nested document query?

**Answer:**

1. DocumentDB supports queries against nested document fields.
2. Dot notation is commonly used.
3. Example: `"address.city": "New Orleans"`.
4. Nested fields can be indexed when appropriate.
5. Query patterns should influence whether data is embedded or referenced.

---

# 28. How are arrays handled?

**Answer:**

1. Arrays can contain values or nested documents.
2. Applications can query array contents.
3. Arrays are useful for tags, order items, and other related values.
4. Large arrays can increase document size and update costs.
5. Multikey indexes can support appropriate array-field queries.

---

# 29. What is an index?

**Answer:**

1. An index is a data structure used to speed up searches.
2. It allows the database to avoid scanning every document for many queries.
3. Indexes are created on frequently queried fields.
4. Indexes consume storage and add write overhead.
5. Good indexing starts with actual query patterns.

---

# 30. Why are indexes important?

**Answer:**

1. Without a suitable index, a query may need to examine many documents.
2. Large collections can make such scans expensive.
3. Indexes can significantly reduce the amount of data examined.
4. They can improve query latency.
5. Too many indexes can hurt write performance, so balance is required.

---

# 31. What is a single-field index?

**Answer:**

1. It indexes one field.
2. Example: `{ email: 1 }`.
3. It is useful for simple equality or range queries.
4. It can improve lookups such as customer-by-email.
5. The direction can be ascending or descending depending on the index definition and use case.

---

# 32. What is a compound index?

**Answer:**

1. A compound index contains multiple fields.
2. Example: `{ customerId: 1, createdAt: -1 }`.
3. It is useful for queries involving multiple fields.
4. Field order matters.
5. The index should be designed around real query and sort patterns.

---

# 33. What is a multikey index?

**Answer:**

1. A multikey index can index array fields.
2. It is useful for documents containing arrays.
3. For example, a product's `tags` array may be queried.
4. The database can index array elements.
5. Multikey indexes should still be created only when useful to actual queries.

---

# 34. What is a unique index?

**Answer:**

1. A unique index enforces uniqueness for the indexed value or key combination.
2. It can prevent duplicate values.
3. Email addresses are a common example.
4. The application should handle duplicate-key errors.
5. Unique constraints should reflect actual business requirements.

---

# 35. How do you choose an index?

**Answer:**

1. Start with the application's most important queries.
2. Identify frequently filtered fields.
3. Consider sorting requirements.
4. Consider compound indexes for multi-field access patterns.
5. Validate the result using realistic performance testing and query-analysis tools supported by DocumentDB.

---

# 36. Should you index every field?

**Answer:**

1. No.
2. Each index consumes resources.
3. Indexes increase write and maintenance overhead.
4. Unused indexes provide little benefit.
5. Create indexes based on actual workload requirements.

---

# 37. What is a collection scan?

**Answer:**

1. A collection scan examines documents without using a suitable index.
2. It can be expensive on large collections.
3. High document examination can increase latency.
4. Query analysis can reveal whether scanning is occurring.
5. Adding an appropriate index may reduce unnecessary document examination.

---

# 38. How do you troubleshoot a slow query?

**Answer:**

1. Identify the exact slow query.
2. Check whether an appropriate index exists.
3. Analyze the query execution behavior.
4. Check CPU, memory, database load, and connections.
5. Optimize the query/index/data model before simply increasing infrastructure size.

---

# 39. What is data modeling in DocumentDB?

**Answer:**

1. Data modeling defines how application data is represented as documents.
2. It includes decisions about embedding and referencing.
3. Query patterns should strongly influence the model.
4. Document size and update frequency also matter.
5. A good model minimizes unnecessary database work.

---

# 40. What is embedding?

**Answer:**

1. Embedding stores related data inside the same document.
2. It allows related data to be retrieved together.
3. It can reduce the need for additional queries.
4. It works well for closely related data with manageable size.
5. Avoid embedding unbounded or extremely large structures.

---

# 41. What is referencing?

**Answer:**

1. Referencing stores an identifier pointing to another document.
2. It is useful for independently managed entities.
3. It can prevent very large documents.
4. It can reduce duplication when the related entity changes frequently.
5. The application may need multiple database operations to retrieve related information.

---

# 42. Embedding vs referencing — which is better?

**Answer:**

1. Neither is universally better.
2. Embed when data is usually accessed together.
3. Reference when data has independent lifecycle or size concerns.
4. Consider update frequency and duplication.
5. Design around actual application access patterns.

---

# 43. Why might you embed order items?

**Answer:**

1. An order normally needs its line items together.
2. Retrieving an order can therefore retrieve its items in one document.
3. Items are conceptually part of the order.
4. Historical price/name information can be preserved.
5. This can simplify order retrieval.

---

# 44. Why might an order reference a customer?

**Answer:**

1. Customers are independent entities.
2. Customer data may be updated independently.
3. One customer can have many orders.
4. Duplicating the entire customer document in every order can create unnecessary duplication.
5. `customerId` provides a simple relationship.

---

# 45. Why store product price inside an order?

**Answer:**

1. An order represents a historical transaction.
2. Product prices can change later.
3. The order should retain the price actually charged.
4. Storing the unit price makes historical reporting simpler.
5. It avoids depending on the current product price.

---

# 46. What is denormalization?

**Answer:**

1. Denormalization intentionally duplicates some information.
2. It can improve read performance.
3. It can simplify queries.
4. It introduces consistency considerations.
5. Document databases commonly use selective denormalization.

---

# 47. What is the maximum document size?

**Answer:**

1. DocumentDB has a maximum supported BSON document size.
2. Applications must stay within the service's current documented limit.
3. Very large documents can also create performance and operational problems before reaching the hard limit.
4. Unbounded arrays should therefore be treated carefully.
5. Always verify the current AWS DocumentDB documentation for the exact limit.

---

# 48. Why should documents not grow indefinitely?

**Answer:**

1. Large documents consume more storage.
2. Large reads increase network transfer.
3. Large updates can become expensive.
4. Very large arrays can become difficult to manage.
5. Unbounded data is often better modeled as separate documents.

---

# 49. What is the VPC role in DocumentDB?

**Answer:**

1. DocumentDB clusters are deployed within an Amazon VPC.
2. The VPC provides network isolation.
3. Subnets determine where resources are placed.
4. Security groups control network access.
5. Private networking is commonly used for production databases.

---

# 50. What is a DB subnet group?

**Answer:**

1. A DB subnet group identifies subnets that a DocumentDB cluster can use.
2. It is associated with a VPC.
3. It normally includes subnets across multiple Availability Zones.
4. It supports appropriate placement of DocumentDB instances.
5. It is an important part of the cluster's network architecture.

---

# 51. What is a Security Group?

**Answer:**

1. A security group acts as a virtual firewall.
2. It controls allowed network traffic to the database resources.
3. DocumentDB commonly listens on TCP port `27017`.
4. You can restrict access to the application's security group.
5. Avoid unnecessarily broad inbound rules.

---

# 52. Why should DocumentDB not normally be exposed directly to the internet?

**Answer:**

1. Databases are high-value targets.
2. Public exposure increases the attack surface.
3. Private networking reduces unnecessary network access.
4. Application servers can communicate with the database internally.
5. Security groups provide another layer of access control.

---

# 53. What is authentication?

**Answer:**

1. Authentication determines **who the user is**.
2. DocumentDB database users authenticate using supported database authentication mechanisms.
3. Credentials must be protected.
4. Authentication is different from AWS IAM permissions.
5. Successful authentication does not automatically mean unlimited database privileges.

---

# 54. What is authorization?

**Answer:**

1. Authorization determines **what an authenticated identity can do**.
2. Database roles/permissions control database operations.
3. AWS IAM controls permissions for AWS resources and APIs.
4. Least privilege should be applied.
5. Authentication and authorization are separate security concepts.

---

# 55. What is least privilege?

**Answer:**

1. Give users only the permissions they need.
2. Don't use administrative credentials for routine application operations.
3. Separate application identities where appropriate.
4. Restrict IAM permissions as well.
5. Least privilege reduces the impact of compromised credentials.

---

# 56. What is IAM's role with DocumentDB?

**Answer:**

1. IAM controls access to AWS resources and APIs.
2. IAM can control what an AWS identity can do with DocumentDB-related resources.
3. IAM is also useful for accessing services such as Secrets Manager and KMS.
4. IAM does not simply replace database authentication.
5. AWS authorization and database authorization are separate layers.

---

# 57. Why use AWS Secrets Manager?

**Answer:**

1. It provides centralized secret storage.
2. Database credentials don't need to be hard-coded in source code.
3. Applications can retrieve secrets using IAM permissions.
4. It supports secret-management and rotation capabilities.
5. It reduces the risk of credentials being exposed through source repositories or configuration files.

---

# 58. What is TLS in DocumentDB?

**Answer:**

1. TLS protects data while it travels between the application and database.
2. It encrypts the network connection.
3. It helps protect credentials and database traffic.
4. Clients/drivers need appropriate TLS configuration.
5. TLS protects data in transit, not stored data.

---

# 59. What is encryption at rest?

**Answer:**

1. Encryption at rest protects stored database data.
2. DocumentDB supports encryption at rest.
3. AWS KMS is used as part of the encryption architecture.
4. Encryption helps protect data if underlying storage is accessed improperly.
5. It is separate from TLS encryption.

---

# 60. What is AWS KMS?

**Answer:**

1. KMS stands for **Key Management Service**.
2. It manages cryptographic keys used by AWS services.
3. DocumentDB can use KMS for encryption at rest.
4. Customer-managed keys provide additional control in appropriate environments.
5. Key permissions and lifecycle should be managed carefully.

---

# 61. What is defense in depth?

**Answer:**

1. Defense in depth uses multiple security layers.
2. VPC controls network boundaries.
3. Security groups restrict connectivity.
4. Authentication and authorization control database access.
5. TLS, KMS, Secrets Manager, monitoring, and auditing add further protection.

---

# 62. What is high availability in DocumentDB?

**Answer:**

1. DocumentDB clusters can contain multiple instances.
2. Replicas can provide additional availability.
3. A suitable replica can be promoted during failover.
4. Applications should use cluster endpoints appropriately.
5. High availability reduces the impact of instance failures.

---

# 63. What is a DocumentDB replica?

**Answer:**

1. A replica is an additional database instance in the cluster.
2. It maintains replicated data from the primary.
3. Replicas can provide high availability.
4. They can also support read scaling.
5. Replica configuration should match the workload and availability requirements.

---

# 64. What is a writer/primary instance?

**Answer:**

1. The primary handles the cluster's write workload.
2. It also serves reads depending on the application's connection design.
3. Replicas maintain replicated copies.
4. During failover, an eligible replica can become primary.
5. Applications should not hard-code a specific instance as their permanent writer target.

---

# 65. What is a reader endpoint?

**Answer:**

1. A reader endpoint is intended for read traffic.
2. It can distribute read connections across eligible replicas.
3. It is useful for read-heavy applications.
4. It should not be used for operations that require the primary.
5. Application architecture should separate read and write requirements where appropriate.

---

# 66. What happens during failover?

**Answer:**

1. The current primary becomes unavailable.
2. DocumentDB can promote an eligible replica.
3. The cluster's primary role moves to the promoted instance.
4. Applications may experience temporary connection errors.
5. Proper endpoint usage and retry/reconnect logic help applications recover.

---

# 67. Why should applications use cluster endpoints?

**Answer:**

1. Cluster endpoints represent the appropriate cluster target.
2. Failover can change which instance is primary.
3. Hard-coded instance endpoints can cause application problems after topology changes.
4. Cluster endpoints simplify failover handling.
5. Driver connection behavior should also be configured appropriately.

---

# 68. What is scaling in DocumentDB?

**Answer:**

1. Scaling means increasing the system's ability to handle workload.
2. Vertical scaling uses a larger instance.
3. Horizontal read scaling can use additional replicas.
4. Query and index optimization can also increase effective capacity.
5. Scaling decisions should be based on measured workload and bottlenecks.

---

# 69. What is vertical scaling?

**Answer:**

1. Vertical scaling means moving to a larger instance size.
2. It provides more compute and memory resources.
3. It can help when the workload is resource constrained.
4. Operational impact must be considered when changing instance configuration.
5. Query optimization should be investigated before scaling blindly.

---

# 70. What is horizontal read scaling?

**Answer:**

1. Horizontal scaling adds additional instances for read capacity.
2. DocumentDB replicas can support read workloads.
3. Applications can use the reader endpoint.
4. It is particularly useful for read-heavy systems.
5. Adding replicas does not automatically solve a write bottleneck.

---

# 71. What is connection pooling?

**Answer:**

1. Connection pooling reuses database connections.
2. It avoids creating a new connection for every request.
3. It reduces connection establishment overhead.
4. It helps control the number of simultaneous connections.
5. Pool size should be tuned according to application and database capacity.

---

# 72. Why can too many connections be dangerous?

**Answer:**

1. Each connection consumes database and application resources.
2. Large application fleets can create connection storms.
3. Excessive connections can increase resource pressure.
4. Connection pooling helps control connection creation.
5. Connection limits should be monitored and capacity-planned.

---

# 73. What is a retry storm?

**Answer:**

1. A retry storm occurs when many clients repeatedly retry failing operations.
2. Retries can increase load on an already unhealthy database.
3. Immediate infinite retries are dangerous.
4. Exponential backoff helps reduce retry pressure.
5. Applications should retry only appropriate transient failures.

---

# 74. What is exponential backoff?

**Answer:**

1. Exponential backoff increases the delay between retries.
2. Example: 100 ms, 200 ms, 400 ms, 800 ms.
3. It reduces pressure during temporary failures.
4. A maximum retry count or timeout should be used.
5. It is especially useful during transient connectivity or failover events.

---

# 75. What is an atomic update?

**Answer:**

1. An atomic update performs a modification as one database operation.
2. It can help avoid race conditions.
3. `$inc` is useful for counters and inventory.
4. Conditions can be included in the update filter.
5. Atomic operations are often preferable to separate read-then-write sequences.

---

# 76. How would you safely decrease inventory?

**Answer:**

1. Include the product identifier in the filter.
2. Check that sufficient inventory exists.
3. Use `$inc` with a negative value.
4. Check whether the update actually matched a document.
5. This helps prevent overselling under concurrent requests.

Example:

```javascript
db.products.updateOne(
  {
    productId: "P1001",
    inventory: { $gte: 2 }
  },
  {
    $inc: { inventory: -2 }
  }
)
```

---

# 77. When should you use transactions?

**Answer:**

1. Use transactions when multiple operations need transactional consistency.
2. They can help maintain business invariants across multiple documents.
3. They add complexity and overhead.
4. Not every database operation needs a transaction.
5. Verify the exact transaction capabilities and limitations supported by the DocumentDB engine/version you use.

---

# 78. What are automated backups?

**Answer:**

1. Automated backups provide ongoing data protection.
2. They support recovery capabilities such as point-in-time recovery.
3. Backup retention is configurable within supported limits.
4. They help recover from accidental or operational data changes.
5. Backup configuration should match business recovery requirements.

---

# 79. What is a manual snapshot?

**Answer:**

1. A manual snapshot is an explicitly created recovery point.
2. It can be useful before risky changes.
3. It can support migration or release safety.
4. It can be retained according to your organization's requirements.
5. Snapshots should still be tested through actual restore procedures.

---

# 80. What is point-in-time recovery?

**Answer:**

1. Point-in-time recovery allows recovery to an appropriate earlier time.
2. It is based on the available automated backup history.
3. It is useful after accidental changes.
4. The available recovery window depends on the configured backup retention and service capabilities.
5. Recovery should be validated after restoration.

---

# 81. What is RPO?

**Answer:**

1. RPO means **Recovery Point Objective**.
2. It defines the amount of data loss the business can tolerate.
3. A smaller RPO generally requires stronger recovery mechanisms.
4. Backup and replication strategies influence achievable RPO.
5. RPO is a business requirement, not merely a database setting.

---

# 82. What is RTO?

**Answer:**

1. RTO means **Recovery Time Objective**.
2. It defines how quickly service should be restored.
3. Restore speed affects RTO.
4. Automation and tested recovery procedures can reduce recovery time.
5. RTO should be validated through disaster-recovery exercises.

---

# 83. What is the difference between HA and backup?

**Answer:**

1. High availability focuses on keeping service available during infrastructure failures.
2. Backups focus on recovering data.
3. HA can help with instance failures.
4. Backups help with accidental deletion or bad data changes.
5. A production system generally needs both.

---

# 84. Why should backups be tested?

**Answer:**

1. A backup existing does not guarantee a successful recovery.
2. Restore procedures may contain operational mistakes.
3. Applications may have dependencies that aren't documented.
4. Testing reveals recovery-time problems.
5. Regular restore exercises increase confidence in the disaster-recovery plan.

---

# 85. What is Amazon CloudWatch?

**Answer:**

1. CloudWatch is AWS's monitoring and observability service.
2. It collects and displays metrics and supports alarms.
3. DocumentDB exposes operational metrics to CloudWatch.
4. Teams can monitor resource usage and workload behavior.
5. CloudWatch alarms can notify operators about important conditions.

---

# 86. What DocumentDB metrics should you monitor?

**Answer:**

1. CPU utilization.
2. Memory/resource pressure.
3. Database connections.
4. Database workload and latency.
5. Storage, replication, network, and error-related indicators relevant to the cluster.

---

# 87. What is CloudTrail?

**Answer:**

1. CloudTrail records AWS API activity.
2. It helps identify who performed AWS actions.
3. It can help investigate configuration changes.
4. It supports security auditing and compliance.
5. It complements CloudWatch rather than replacing it.

---

# 88. CloudWatch vs CloudTrail?

**Answer:**

1. CloudWatch focuses on monitoring and operational telemetry.
2. CloudTrail focuses on AWS API activity and auditing.
3. CloudWatch answers: **"How is the system behaving?"**
4. CloudTrail helps answer: **"Who performed this AWS action?"**
5. Using both provides better operational visibility.

---

# 89. What is a CloudWatch alarm?

**Answer:**

1. An alarm evaluates a metric against configured conditions.
2. It can detect abnormal operational behavior.
3. It can notify an operations team through supported integrations.
4. Examples include sustained resource pressure or unusual activity.
5. Alarms should be actionable rather than excessively numerous.

---

# 90. How do you troubleshoot high CPU?

**Answer:**

1. Confirm whether CPU is consistently high or only temporarily elevated.
2. Examine database load and workload patterns.
3. Identify expensive queries.
4. Check indexes and query execution behavior.
5. If optimization is insufficient, evaluate instance scaling.

---

# 91. How do you troubleshoot a connection timeout?

**Answer:**

1. Check the DocumentDB endpoint and port.
2. Verify VPC/subnet/network connectivity.
3. Check security-group rules.
4. Verify DNS and routing behavior.
5. If the network path works, investigate TLS, authentication, connection limits, and application configuration.

---

# 92. How do you troubleshoot an authentication failure?

**Answer:**

1. Verify the username.
2. Verify the password or retrieved secret.
3. Confirm the correct authentication configuration.
4. Verify TLS/client configuration where required.
5. Check that the database user exists and has appropriate permissions.

---

# 93. What would you do if queries suddenly became slow?

**Answer:**

1. Identify whether the problem affects all queries or specific queries.
2. Check CloudWatch resource and workload metrics.
3. Investigate recent application or query changes.
4. Check indexes and query execution behavior.
5. Determine whether the issue is query design, data growth, resource pressure, or networking.

---

# 94. How would you design a production DocumentDB application?

**Answer:**

1. Place DocumentDB in a private VPC/subnet architecture.
2. Restrict access through security groups.
3. Use TLS, encryption at rest, KMS, and Secrets Manager.
4. Design documents and indexes around application access patterns.
5. Add backups, monitoring, failover handling, and tested recovery procedures.

---

# 95. What should you do before a major database migration?

**Answer:**

1. Test the migration in a non-production environment.
2. Validate application compatibility.
3. Review query and index performance.
4. Create an appropriate recovery point/snapshot strategy.
5. Prepare rollback and recovery procedures before making production changes.

---

# 96. What should you never store in source code?

**Answer:**

1. Database passwords.
2. Secrets Manager credentials.
3. Private keys.
4. API secrets or tokens.
5. Sensitive connection strings containing credentials.

---

# 97. What are common DocumentDB production mistakes?

**Answer:**

1. Exposing the database too broadly through security groups.
2. Using administrative credentials in applications.
3. Creating too few or inappropriate indexes.
4. Ignoring connection pooling and failover behavior.
5. Having backups without ever testing restoration.

---

# 98. What are the most important DocumentDB interview topics?

**Answer:**

1. DocumentDB vs MongoDB compatibility.
2. Documents, collections, BSON, CRUD, and queries.
3. Indexes and document data modeling.
4. VPC, security groups, encryption, IAM, and Secrets Manager.
5. High availability, backups, monitoring, scaling, and disaster recovery.

---

# 99. How would you answer: "Design a production DocumentDB system"?

**Answer:**

1. Start with the application's access patterns and data model.
2. Deploy the cluster in a private VPC architecture across appropriate Availability Zones.
3. Restrict connectivity using security groups and secure credentials with Secrets Manager.
4. Use indexes, connection pooling, appropriate replicas, backups, TLS, and KMS encryption.
5. Add CloudWatch monitoring, CloudTrail auditing, failover handling, tested restore procedures, and defined RPO/RTO.

---

# 100. What is the complete Amazon DocumentDB mental model?

**Answer:**

1. **Data:** Documents, collections, BSON, CRUD, queries.
2. **Performance:** Indexes, query patterns, connection pooling, scaling.
3. **Modeling:** Embed or reference based on access patterns and document lifecycle.
4. **AWS:** VPC, subnets, security groups, instances, replicas, endpoints.
5. **Production:** TLS, KMS, Secrets Manager, backups, CloudWatch, CloudTrail, failover, recovery, and operational discipline.

---

# 🏆 The 100-FAQ Master Revision Map

If you're preparing for an **AWS/Cloud/Database interview**, organize the 100 questions into these 10 buckets:

| Area                |    FAQ Range | Core Things                                      |
| ------------------- | -----------: | ------------------------------------------------ |
| Fundamentals        |         1–10 | DocumentDB, MongoDB compatibility, BSON          |
| CRUD                |        11–16 | Insert, read, update, delete                     |
| Queries             |        17–28 | Filters, operators, arrays, nested documents     |
| Indexes             |        29–38 | Single, compound, multikey, performance          |
| Data Modeling       |        39–48 | Embed, reference, denormalization                |
| AWS Architecture    | 49–52, 62–70 | VPC, subnets, SG, replicas, scaling              |
| Security            |        53–61 | IAM, authentication, TLS, KMS, Secrets           |
| Operations          |        71–77 | Pooling, retries, transactions, atomic updates   |
| Backup & Monitoring |        78–93 | PITR, snapshots, RPO/RTO, CloudWatch, CloudTrail |
| Production          |       94–100 | Architecture, troubleshooting, interviews        |

---

# 🔥 20 Questions You Should Know Absolutely Cold

If an interviewer gives you only a few minutes, make sure you can answer these immediately:

```text
1.  What is Amazon DocumentDB?
2.  DocumentDB vs MongoDB?
3.  What is BSON?
4.  What are documents and collections?
5.  Explain CRUD.
6.  What are MongoDB query operators?
7.  What is an index?
8.  Single vs compound index?
9.  How do you troubleshoot a slow query?
10. Embedding vs referencing?
11. How does DocumentDB work inside a VPC?
12. What does a security group do?
13. Authentication vs authorization?
14. IAM vs database user?
15. TLS vs encryption at rest?
16. Why use Secrets Manager?
17. How does DocumentDB provide high availability?
18. Automated backup vs snapshot vs PITR?
19. CloudWatch vs CloudTrail?
20. Design a production DocumentDB architecture.
```

## 🧠 One final formula

```text
                AMAZON DOCUMENTDB
                        │
       ┌────────────────┼────────────────┐
       ▼                ▼                ▼
      DATA          PERFORMANCE         AWS
       │                │                │
 Documents          Indexes            VPC
 Collections        Queries            Subnets
 BSON               Pooling            SG
 CRUD               Scaling            Replicas
       │                │                │
       └────────────────┼────────────────┘
                        ▼
                    SECURITY
                        │
             IAM + TLS + KMS
             Secrets + Least Privilege
                        │
                        ▼
                   OPERATIONS
                        │
       ┌────────────────┼────────────────┐
       ▼                ▼                ▼
    Backups          Monitoring       Failover
    Snapshots        CloudWatch        Recovery
    PITR             CloudTrail        RPO/RTO
```

**If you can confidently explain these 100 FAQs, you have a strong end-to-end foundation in Amazon DocumentDB—from basic document operations all the way to production architecture and troubleshooting.**
