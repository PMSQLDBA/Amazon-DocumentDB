# Amazon DocumentDB — Top 50 Real-Time Scenario-Based FAQs

---

## 1. Your application suddenly gets `connection timeout` while connecting to DocumentDB. What do you check?

**Answer:**

1. Verify the DocumentDB endpoint, port (`27017`), and connection string.
2. Check whether the application and DocumentDB are in reachable VPC/network paths.
3. Verify the DocumentDB security group's inbound rule allows traffic from the application's security group.
4. Check subnet routing, NACLs, DNS resolution, and network connectivity.
5. If networking is correct, investigate TLS configuration, connection limits, and application connection pooling.

---

## 2. Your application gets `Authentication failed`. How do you troubleshoot it?

**Answer:**

1. Verify the database username and password.
2. Check whether the application is retrieving the latest credentials from Secrets Manager.
3. Verify the authentication mechanism and TLS configuration required by your environment.
4. Confirm the database user exists and has the required permissions.
5. Check whether a recent credential rotation or configuration change caused the failure.

---

## 3. A query that previously took 50 ms now takes 5 seconds. What would you do?

**Answer:**

1. Identify whether the slowdown affects one query or the entire workload.
2. Examine the query execution behavior and determine whether a collection scan is occurring.
3. Check whether an appropriate index exists and whether the query still matches the index design.
4. Review recent data growth, application changes, and index changes.
5. Check CloudWatch metrics for CPU, connections, memory/resource pressure, and overall database load.

---

## 4. Your `customers` collection has 100 million documents and an email lookup is slow. How do you fix it?

**Answer:**

1. Identify the lookup pattern, for example `{ email: "user@example.com" }`.
2. Create an appropriate index on `email`.
3. If email must be unique, consider an appropriate unique-index design supported by your DocumentDB version.
4. Test the query using production-like data volume.
5. Monitor latency and resource usage after the index is introduced.

---

## 5. Your application performs many reads but relatively few writes. How would you scale DocumentDB?

**Answer:**

1. Identify whether the primary is overloaded primarily by reads.
2. Add appropriate replica instances to the cluster.
3. Route suitable read traffic through the reader endpoint.
4. Keep writes directed to the appropriate writer/cluster endpoint.
5. Monitor read distribution and latency to confirm that replicas actually improve the workload.

---

## 6. Your primary DocumentDB instance fails during business hours. What happens?

**Answer:**

1. DocumentDB detects the primary failure.
2. An eligible replica can be promoted to become the new primary.
3. The cluster's writer role moves to the promoted instance.
4. Applications using the correct cluster endpoint can reconnect to the new primary.
5. Applications should have retry/reconnect logic because some requests may fail during the transition.

---

## 7. Your application hard-codes the IP address of a DocumentDB instance. Is this a good design?

**Answer:**

1. No, this creates an operational dependency on a specific instance.
2. Instance roles can change during failover.
3. IP addresses and topology can change.
4. Applications should use the appropriate DocumentDB endpoint.
5. Driver configuration should also support reconnecting after topology changes.

---

## 8. Your application has 500 EC2 instances and DocumentDB is receiving thousands of connections. What could be wrong?

**Answer:**

1. Each application instance may be creating too many database connections.
2. Connection pooling may be missing or incorrectly configured.
3. Autoscaling may be creating connection storms.
4. Configure sensible minimum and maximum pool sizes.
5. Monitor connection metrics and test the configuration under realistic load.

---

## 9. After a deployment, DocumentDB CPU increases to 95%. What would you investigate?

**Answer:**

1. Compare the workload before and after the deployment.
2. Identify newly introduced or changed queries.
3. Look for missing or ineffective indexes.
4. Check whether a query is scanning a large collection.
5. If the workload is legitimate and optimized, evaluate scaling the instance or adding read capacity.

---

## 10. Your application is receiving stale data from DocumentDB. What could cause it?

**Answer:**

1. The application may be reading from a replica or reader endpoint.
2. Replication introduces the possibility of replica lag.
3. Immediately-after-write reads may require the writer depending on consistency requirements.
4. Review whether the application actually requires strongly consistent read behavior.
5. Design read/write routing according to the application's consistency requirements.

---

# Performance Scenarios

## 11. A query filters by `customerId` and sorts by `createdAt`. What index would you consider?

**Answer:**

1. Identify the exact query pattern.
2. Consider a compound index containing `customerId` and `createdAt`.
3. Put the fields in an order appropriate for the filter and sort pattern.
4. Verify that the index is actually used by the workload.
5. Test with realistic data volume rather than assuming the index will always improve performance.

---

## 12. You created 20 indexes and write performance became worse. Why?

**Answer:**

1. Every index requires maintenance when indexed data changes.
2. Additional indexes consume storage and database resources.
3. Some indexes may never be used.
4. Review index usage against actual production queries.
5. Remove unnecessary indexes carefully after validating their impact.

---

## 13. A collection contains large documents and reads are slow. What would you do?

**Answer:**

1. Measure document size and identify unnecessary fields.
2. Use projection when the application doesn't need every field.
3. Avoid unbounded embedded arrays.
4. Consider splitting large or independently accessed data into separate documents.
5. Review whether the current data model matches the application's access patterns.

---

## 14. Your `orders` documents contain thousands of items in one array. What problem could this cause?

**Answer:**

1. Documents can become excessively large.
2. Reads may transfer much more data than necessary.
3. Updates to large arrays can become expensive.
4. Indexing array contents can also increase resource usage.
5. Consider modeling very large or independently queried order-item data separately.

---

## 15. Your query uses `$or` with several conditions and performs poorly. What do you investigate?

**Answer:**

1. Examine each branch of the `$or`.
2. Determine whether suitable indexes exist for the relevant fields.
3. Check how many documents each condition matches.
4. Simplify the query if the business logic allows it.
5. Validate execution behavior with realistic data rather than optimizing based only on query syntax.

---

## 16. Your application frequently searches products by `category`, `brand`, and `price`. How do you design indexes?

**Answer:**

1. Start with the application's actual query patterns.
2. Identify equality fields such as category and brand.
3. Identify range fields such as price.
4. Consider an appropriate compound index based on the common query shape.
5. Test multiple candidate index designs with realistic workloads.

---

## 17. A query is fast in development but slow in production. Why?

**Answer:**

1. Development may contain only thousands of documents.
2. Production may contain millions or billions of documents.
3. Data distribution can be completely different.
4. Index effectiveness can change with production cardinality and workload.
5. Always performance-test with production-like data volumes and query distributions.

---

## 18. A developer says, "Adding an index will always make the query faster." Is that correct?

**Answer:**

1. No.
2. An index only helps when it matches a useful query pattern.
3. The optimizer may choose another access path depending on the workload.
4. Additional indexes can increase write and storage overhead.
5. Index decisions should be validated using actual workload measurements.

---

## 19. Your database has high CPU but low connection count. What does that suggest?

**Answer:**

1. The bottleneck may be query processing rather than connections.
2. Expensive queries may be consuming CPU.
3. Missing or inefficient indexes are possible causes.
4. Large scans, sorting, or complex operations should be investigated.
5. Review query behavior before simply increasing the number of connections.

---

## 20. CPU is normal, but application latency is high. What would you investigate?

**Answer:**

1. Check network latency between the application and DocumentDB.
2. Check connection acquisition and pool wait time.
3. Check database query latency.
4. Investigate replica lag or endpoint selection if reads are involved.
5. Check application-side serialization, deserialization, and downstream dependencies.

---

# Data Modeling Scenarios

## 21. You are designing an e-commerce order database. Would you embed order items?

**Answer:**

1. Usually, order items are closely related to the order.
2. An order is commonly retrieved together with its items.
3. Embedding can reduce the number of database operations.
4. Store historical values such as purchased quantity and price within the order.
5. If item collections can grow without bound, reconsider the model.

---

## 22. A customer can have millions of orders. Should all orders be embedded inside the customer document?

**Answer:**

1. No, this creates an unbounded document-growth problem.
2. Customer and order have different lifecycles.
3. Orders should generally be separate documents.
4. Store a `customerId` reference in each order.
5. Index the customer identifier for efficient order retrieval.

---

## 23. Product details change frequently, but orders must preserve historical product information. What would you do?

**Answer:**

1. Keep the current product as its own document.
2. Store relevant historical information inside the order.
3. For example, preserve product name, SKU, quantity, and purchase price.
4. Don't depend entirely on the current product document for historical reporting.
5. This is an intentional form of denormalization.

---

## 24. Your application needs customer profile + recent orders on every request. How could you optimize the model?

**Answer:**

1. Determine how many orders are considered "recent."
2. Avoid embedding all historical orders.
3. Store orders separately.
4. Maintain only a bounded summary/recent-order representation if the access pattern justifies it.
5. Use indexes to retrieve recent orders efficiently.

---

## 25. You have a social-media application with users and followers. Would you store all followers in one user document?

**Answer:**

1. It depends on the expected follower count.
2. A small bounded list may be reasonable to embed.
3. Millions of followers create an unbounded-array problem.
4. Large follower relationships are better modeled as separate documents.
5. Query patterns such as "followers of user X" should drive the index design.

---

# Security Scenarios

## 26. A developer accidentally committed the DocumentDB password to Git. What do you do?

**Answer:**

1. Treat the credential as compromised.
2. Rotate/change the database credential immediately.
3. Remove the secret from active application configuration.
4. Move credential management to a service such as Secrets Manager.
5. Review logs and access history for suspicious usage.

---

## 27. A security engineer asks you to allow `0.0.0.0/0` to DocumentDB port 27017. Would you approve it?

**Answer:**

1. Normally, no.
2. It exposes the database to an unnecessarily broad network range.
3. Restrict access to the application's security group or required private networks.
4. Keep DocumentDB private whenever the architecture allows.
5. Follow least privilege at the network layer.

---

## 28. Your application needs to access DocumentDB from Lambda. What networking considerations matter?

**Answer:**

1. The Lambda function needs network connectivity to the DocumentDB VPC.
2. Configure Lambda networking appropriately for the VPC.
3. Ensure security groups permit the required traffic.
4. Ensure subnet/routing configuration supports the connection.
5. Configure connection reuse/pooling carefully because Lambda concurrency can create many connections.

---

## 29. Your security team requires encryption at rest. How would you configure DocumentDB?

**Answer:**

1. Enable encryption at rest when creating/configuring the cluster as supported.
2. Use AWS KMS for key management.
3. Consider a customer-managed KMS key when organizational requirements call for it.
4. Control access to the key through IAM/KMS policies.
5. Document the key lifecycle and recovery requirements.

---

## 30. Your security team says, "TLS is enabled, so the database is fully secure." Is that correct?

**Answer:**

1. No.
2. TLS protects data in transit.
3. It doesn't replace authentication or authorization.
4. It doesn't replace encryption at rest.
5. VPC isolation, security groups, IAM, secrets management, logging, and least privilege are also important.

---

# Backup & Disaster Recovery

## 31. A developer accidentally deletes thousands of documents. What recovery options do you investigate?

**Answer:**

1. Determine exactly when the deletion occurred.
2. Check the available automated backup/PITR window.
3. Identify the appropriate recovery point.
4. Restore into an appropriate recovery environment rather than blindly overwriting production.
5. Validate the restored data before using it for production recovery.

---

## 32. Your manager asks, "We have backups, so why do we need disaster-recovery testing?" How do you answer?

**Answer:**

1. A backup is only useful if it can actually be restored.
2. Restore procedures may fail due to configuration mistakes.
3. Application dependencies may be missing.
4. Recovery time may exceed the business RTO.
5. Regular restore testing validates both the technology and operational process.

---

## 33. Your business says it can tolerate only 5 minutes of data loss. What does that represent?

**Answer:**

1. This is an **RPO requirement**.
2. It means the business expects recovery to a point no more than roughly 5 minutes behind the failure scenario.
3. Backup/replication strategies must support the requirement.
4. Recovery mechanisms should be tested against it.
5. The actual achievable RPO depends on the architecture and service capabilities.

---

## 34. Your business requires the application to recover within 15 minutes. What is that?

**Answer:**

1. This is an **RTO requirement**.
2. It defines the maximum acceptable recovery time.
3. Database restoration time must be included.
4. Application, networking, secrets, DNS/endpoints, and validation must also be considered.
5. Conduct recovery drills to verify the 15-minute target is realistic.

---

## 35. Before a major production migration, what safety steps would you take?

**Answer:**

1. Create an appropriate recovery point/snapshot.
2. Validate automated backup configuration.
3. Test the migration in a non-production environment.
4. Prepare a rollback procedure.
5. Monitor the application and database closely during and after the migration.

---

# High Availability & Architecture

## 36. Your application must survive an Availability Zone failure. How would you design DocumentDB?

**Answer:**

1. Use a cluster architecture spanning appropriate Availability Zones.
2. Place instances/replicas across separate AZs.
3. Ensure the subnet group supports multi-AZ placement.
4. Configure the application to use appropriate cluster endpoints.
5. Test failover rather than assuming the architecture will work.

---

## 37. You have three DocumentDB instances. Does that automatically mean the application is highly available?

**Answer:**

1. Not necessarily.
2. Instance placement and roles matter.
3. The application must connect through appropriate endpoints.
4. Network configuration must support the intended architecture.
5. Failover and application reconnection must be tested.

---

## 38. Your company wants read scaling but cannot tolerate stale reads for a critical operation. What would you do?

**Answer:**

1. Separate critical reads from scalable read workloads.
2. Route operations requiring the necessary consistency to the appropriate writer/read path.
3. Use replicas for workloads where replica lag is acceptable.
4. Don't blindly route all reads to the reader endpoint.
5. Document consistency requirements for each application operation.

---

## 39. Your application experiences errors immediately after a DocumentDB failover. What should you change?

**Answer:**

1. Verify that the application uses an appropriate cluster endpoint.
2. Implement connection retry/reconnect logic.
3. Use sensible timeout settings.
4. Apply exponential backoff to transient failures.
5. Test the application by performing controlled failover exercises.

---

## 40. Your read replicas aren't reducing application latency. What do you investigate?

**Answer:**

1. Verify that read traffic is actually reaching the replicas.
2. Check whether the application is still sending reads to the writer.
3. Examine replica utilization and lag.
4. Check whether network/application latency is the real bottleneck.
5. Confirm that the workload is actually parallelizable across replicas.

---

# Production Operations

## 41. CloudWatch shows a sudden spike in database connections every morning at 9 AM. What could be happening?

**Answer:**

1. A scheduled batch job may be starting.
2. Application autoscaling may be occurring at that time.
3. Connection pools may be configured too aggressively.
4. Multiple services may simultaneously establish new connections.
5. Investigate application logs and correlate connection metrics with deployment/batch activity.

---

## 42. Your application is creating a new DocumentDB connection for every HTTP request. What's wrong?

**Answer:**

1. This creates unnecessary connection overhead.
2. It can rapidly increase database connection counts.
3. Connection establishment adds latency.
4. It can cause connection storms under high traffic.
5. Use the driver's connection pooling/reuse capabilities appropriately.

---

## 43. A deployment causes thousands of failed database operations and then the application starts retrying everything. What is the risk?

**Answer:**

1. Retries can amplify the original failure.
2. The database may receive even more load.
3. This can become a retry storm.
4. Implement bounded retries with exponential backoff.
5. Fix the underlying deployment/database issue rather than relying on retries indefinitely.

---

## 44. Your DocumentDB cluster is running out of storage. What do you investigate?

**Answer:**

1. Check data growth trends.
2. Identify unusually large documents or collections.
3. Review index storage consumption.
4. Investigate temporary or historical data that can be archived/removed according to business requirements.
5. Plan capacity before reaching a critical storage condition.

---

## 45. Your database load increases every month by 30%. What should you do?

**Answer:**

1. Establish a capacity baseline.
2. Forecast future CPU, memory, storage, connections, and workload growth.
3. Optimize expensive queries and indexes.
4. Evaluate vertical scaling and additional read replicas.
5. Create a capacity plan before the system reaches saturation.

---

# Real-Time Incident Scenarios

## 46. Production is down. You discover DocumentDB is healthy, but the application cannot connect. What is your approach?

**Answer:**

1. Check whether the failure is application-wide or limited to specific instances.
2. Test DNS resolution and network connectivity.
3. Verify security-group and routing changes.
4. Check recent deployments/configuration changes.
5. Compare application connection configuration against the known-good configuration.

---

## 47. DocumentDB is healthy, but API response times increased from 200 ms to 2 seconds. How do you isolate the problem?

**Answer:**

1. Break the API latency into application, database, network, and downstream portions.
2. Measure actual DocumentDB query latency.
3. Check connection-pool wait time.
4. Check CloudWatch database metrics.
5. Determine whether the bottleneck is database-side or application-side before changing database capacity.

---

## 48. A production query suddenly starts scanning millions of documents. What could have happened?

**Answer:**

1. An index may have been removed or changed.
2. The query shape may have changed after an application deployment.
3. Data distribution may have changed significantly.
4. A previously useful index may no longer be appropriate.
5. Investigate the query and index configuration before adding random indexes.

---

## 49. You need to migrate an existing MongoDB application to Amazon DocumentDB. What is your approach?

**Answer:**

1. Inventory the MongoDB features, drivers, queries, indexes, and data types being used.
2. Compare them against the DocumentDB version's compatibility/support matrix.
3. Test the application against DocumentDB in a non-production environment.
4. Validate performance, transactions, indexes, aggregation behavior, and operational features.
5. Plan data migration, cutover, rollback, and post-migration monitoring.

---

## 50. You're asked to design a complete production DocumentDB solution for an e-commerce application. What would your architecture look like?

**Answer:**

1. **Application layer:** Use application servers/services in the VPC with properly managed connection pools.
2. **Database layer:** Use a DocumentDB cluster with an appropriate writer and replicas for availability/read scaling.
3. **Data model:** Separate customers/products/orders appropriately, embedding bounded order-item data where it makes sense.
4. **Security:** Private subnets, restrictive security groups, TLS, KMS encryption, Secrets Manager, and least-privilege access.
5. **Operations:** CloudWatch monitoring, CloudTrail auditing, automated backups/PITR, snapshots where appropriate, failover testing, and defined RPO/RTO.

---

# 🔥 Top 15 Scenarios to Master for Interviews

If you're short on time, make these **15 scenarios your priority**:

| #  | Scenario                   | Main Skill Tested             |
| -- | -------------------------- | ----------------------------- |
| 1  | Connection timeout         | VPC + Security Group          |
| 2  | Authentication failure     | Credentials + Security        |
| 3  | Slow query                 | Indexing + Query Optimization |
| 4  | 100M-document collection   | Index Design                  |
| 5  | Read-heavy workload        | Replicas + Reader Endpoint    |
| 6  | Primary failure            | Failover                      |
| 7  | Hard-coded DB endpoint     | AWS Architecture              |
| 8  | Thousands of connections   | Connection Pooling            |
| 9  | 95% CPU                    | Performance Troubleshooting   |
| 10 | Huge embedded arrays       | Data Modeling                 |
| 11 | Secret committed to Git    | Security Incident             |
| 12 | Accidental deletion        | PITR + Recovery               |
| 13 | AZ failure                 | High Availability             |
| 14 | MongoDB → DocumentDB       | Migration + Compatibility     |
| 15 | Complete e-commerce system | End-to-End Architecture       |

## 🧠 The Production Troubleshooting Formula

When an interviewer gives you **any DocumentDB production incident**, think in this order:

```text
                    INCIDENT
                       │
                       ▼
              1. IS DATABASE UP?
                       │
                       ▼
             2. CAN APP CONNECT?
                       │
           ┌───────────┴───────────┐
           ▼                       ▼
        Network                 Security
       VPC / SG               TLS / Auth
           │                       │
           └───────────┬───────────┘
                       ▼
                3. QUERY HEALTH
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
          Indexes              Queries
             │                   │
             └─────────┬─────────┘
                       ▼
                4. RESOURCE LOAD
                       │
          CPU / Memory / Connections
                       │
                       ▼
                 5. SCALE / FIX
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Optimize     Replicas      Bigger
        Query       Read Load     Instance
                       │
                       ▼
                6. RECOVERY
                       │
             Backup / PITR / DR
                       │
                       ▼
                 7. PREVENTION
                       │
       Monitoring + Alarms + Testing
```

**This is the key difference between knowing DocumentDB commands and being able to handle DocumentDB as a production AWS engineer.**
