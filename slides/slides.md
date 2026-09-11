---
marp: true
theme: default
paginate: true
size: 16:9
title: "MongoDB Schema Design"
footer: "MongoDB Schema Design | Major League Hacking - Global Hack Week"
---

# MongoDB Schema Design
## From Relational Mindset to High-Performance Document Modeling

**Facilitator:** Alberto Camarena  

<!--
Speaker Notes:
Welcome everyone! In this workshop, we are diving deep into MongoDB Schema Design.
Whether you come from a SQL background or are building your first web application, today you will learn how to structure your database for maximum speed, scalability, and developer velocity.
-->

---

# Workshop Agenda
## What We Will Cover Today

1. **Data Modeling Fundamentals**
2. **The Mindset Shift**
3. **Core Concepts**
4. **The 6 Rules of Thumb**
5. **The 4-Step MongoDB Schema Design Methodology**
6. **Architectural Schema Patterns**
7. **Antipatterns & Pitfalls**
8. **JSON Schema Validation, Mastering MongoDB & Skill Badges**

<!--
Speaker Notes:
We have a comprehensive roadmap. We start with fundamental concepts and mindset shifts before moving to practical patterns and real document examples.
-->

---

# Module 1
## Data Modeling Fundamentals

- What is a Data Model?
- The Three Stages of Modeling: Conceptual, Logical & Physical
- Why Schema Design Dictates Application Performance

<!--
Speaker Notes:
Before touching MongoDB specifics, let us align on what data modeling actually means across all database systems.
-->

---

# Data Modeling Defined
### Translating Real-World Domains into Software

> **Data Modeling** is the process of defining how data is collected, structured, stored, and accessed by an application.

### The Three Abstraction Layers:
1. **Conceptual Model:** High-level view for business logic and stakeholders (Entities and Business Rules).
2. **Logical Model:** Defines attributes, relationships, and constraints without tying to a specific engine.
3. **Physical Model:** The exact storage structures (Tables/Foreign Keys in SQL, Collections/BSON Documents in MongoDB).

<!--
Speaker Notes:
Every app starts with a conceptual model. But how you translate logical entities into physical database storage differs dramatically depending on your database engine!
-->

---

# Why Schema Design Matters
## The Direct Connection to System Performance

- **Query Latency:** Bad schemas force multiple sequential round-trips or disk reads.
- **WiredTiger RAM Working Set:** Storing bloated documents ejects frequently accessed data out of RAM memory.
- **Index Efficiency:** Unstructured or deeply unpredictable documents make indexing impossible or inefficient.
- **Developer Velocity:** Clean schemas make backend code easier to maintain and extend.

<!--
Speaker Notes:
In MongoDB, schema design directly impacts hardware utilization. If your working set fits in RAM, MongoDB reads take microseconds. If poor schemas bloat documents, MongoDB spends time paging to disk.
-->

---

# Module 2
## The Core Paradigm Shift: SQL vs. MongoDB

- The Relational Paradigm: *"The application takes the form of the data."*
- The Document Paradigm: *"Your data takes the shape of your application."*
- Normalization vs. Strategic Denormalization

<!--
Speaker Notes:
This is the single most important mental leap for developers transitioning from relational SQL to MongoDB.
-->

---

# Relational Mindset (SQL)
## "The Application Takes the Form of the Data"

```
[Users Table] ----1:N----> [Orders Table] ----1:N----> [Order Items Table] ----N:1----> [Products Table]
```

- **Third Normal Form (3NF):** Eliminate data redundancy at all costs.
- **Table Rigidities:** Tables represent fixed tabular entity types.
- **Query-Time Work:** Reconstruct object hierarchies on every read using `JOIN` operations.
- **Result:** Low storage overhead, but high CPU/IO costs during complex queries.

<!--
Speaker Notes:
In SQL, you break data down into its smallest atomic tables. When your application needs a user profile with orders, you join 4 or 5 tables together every single time.
-->

---

# Document Mindset (MongoDB)
## "Your Data Takes the Shape of Your Application"

- **Data Access First:** Design collections based on how your frontend/API queries and displays data.
- **Locality of Data:** Data accessed together should be stored together in a single document.
- **Embedded Hierarchies:** Arrays and nested objects directly reflect JSON objects in code.
- **Schema Flexibility:** Collections accommodate evolving schemas without downtime or heavy `ALTER TABLE` locks.

<!--
Speaker Notes:
In MongoDB, if your user profile page always displays user info along with their recent order summary, store that summary inside the user document! Zero joins needed.
-->

---

# SQL vs. MongoDB Mental Comparison

| Relational (SQL) | MongoDB (Document) |
| :--- | :--- |
| Database | Database |
| Table | Collection |
| Row / Record | BSON Document |
| Column | Field |
| Primary Key | Primary Key (`_id`) |
| Foreign Key / `JOIN` | Embedded Subdocument or `$lookup` |
| Normalized (3NF) | Denormalized / Workload-Optimized |

<!--
Speaker Notes:
Keep this table in mind. Rows become Documents, Tables become Collections, and JOINs are replaced by embedded documents or references.
-->
---

# Module 3
## Core Concepts: Entities, Cardinality & Storage Tradeoffs

- BSON Data Types & Entity Representation
- Quantifying Relationship Cardinality
- Embedding vs. Referencing Decision Matrix

<!--
Speaker Notes:
Now let us break down the basic building blocks of MongoDB documents: entity structures, cardinality levels, and when to embed versus reference.
-->

---

# Entities & BSON Data Types
## Rich Data Representation Inside Documents

MongoDB stores data as **BSON** (Binary JSON), extending JSON with native types:

- **Strings, Numbers, Booleans:** Standard primitives.
- **ObjectId (`_id`):** 12-byte unique identifier (timestamp, process ID, counter).
- **ISODate (`Date`):** 64-bit integer representing UTC milliseconds.
- **Embedded Documents:** Objects nested inside objects (`{ street: "123 Main St" }`).
- **Arrays:** Lists of scalar values or nested documents (`[ "tag1", "tag2" ]`).

<!--
Speaker Notes:
BSON allows rich nested structures natively. Unlike SQL, you can store arrays and objects right inside a field.
-->

---

# Quantifying Relationship Cardinality
## Not All 1:N Relationships Are Created Equal!

In SQL, every relationship is simply labeled `1:1`, `1:N`, or `N:M`. In MongoDB, we measure **cardinality depth**:

1. **One-to-One (`1:1`):** User to User Settings.
2. **One-to-Few (`1:10s`):** User to Addresses (2-5 addresses).
3. **One-to-Many (`1:1,000s`):** Product to Product Reviews (hundreds/thousands).
4. **One-to-Squillions (`1:1,000,000s`):** IoT Sensor to Temperature Log Entries (millions).
5. **Many-to-Many (`N:M`):** Students to Courses.

<!--
Speaker Notes:
The choice between embedding or referencing depends heavily on whether N means 5 items, 500 items, or 5 million items.
-->

---

# Embedding vs. Referencing
### The Core Design Dilemma

### Option A: Embedding (Subdocuments & Arrays)
- Storing related entities inside a single document.
- **Pros:** Excellent read performance (1 IO operation), atomic updates (`$push`, `$set`).
- **Cons:** Subject to 16MB document size limit; potential data duplication.

### Option B: Referencing (Document Links / `_id`)
- Storing related entities in separate collections linked by `_id`.
- **Pros:** Unbounded growth, no duplication, smaller individual document size.
- **Cons:** Requires multiple queries or pipeline `$lookup` joins.

<!--
Speaker Notes:
Embedding is your default choice in MongoDB for speed. Referencing is your solution when data grows infinitely or is accessed independently.
-->

---

# Module 4
## The 6 Rules of Thumb for MongoDB Schema Design

*(Engineering Guidelines from MongoDB Architecture Experts)*

1. Favor Embedding Unless There Is a Reason Not To
2. Standalone Access Calls for Separation
3. Watch Your Array Growth (Cardinality Rules)
4. Don't Fear Application-Level Joins
5. Factor In Read/Write Ratios
6. Schema Depends on Application Access Patterns

<!--
Speaker Notes:
MongoDB published these 6 Rules of Thumb to simplify schema decisions. Let us inspect each rule in detail.
-->

---

# Rule 1 & Rule 2
## Default to Embedding unless Standalone Access is Needed

### Rule 1: Favor embedding unless there is a compelling reason not to.
- If data is retrieved together and lifecycle-bound together (e.g., order and line items), **embed it**.

### Rule 2: Needing to access an object on its own is a compelling reason not to embed it.
- If an entity is queried independently without its parent context (e.g., searching for products directly outside of an order context), **store it in its own collection**.

<!--
Speaker Notes:
Rule 1 gives you performance out of the box. Rule 2 prevents you from burying independent entities deep inside other documents where searching them is awkward.
-->

---

# Rule 3
## Array Growth Rules: One-to-Few, Many & Squillions

- **1-to-Few (e.g., User addresses):** Embed array of subdocuments in parent.
  ```json
  { "name": "Alice", "addresses": [ { "city": "NYC" }, { "city": "SF" } ] }
  ```
- **1-to-Many (e.g., Task Board items):** Embed array of document IDs (Child References).
  ```json
  { "board": "Sprint 1", "tasks": [ ObjectId("..."), ObjectId("...") ] }
  ```
- **1-to-Squillions (e.g., Log streams):** Use **Parent Referencing** in child documents.
  ```json
  { "_id": ObjectId("log1"), "parent_sensor_id": ObjectId("sensor123"), "temp": 24.5 }
  ```

<!--
Speaker Notes:
Parent referencing is crucial for huge datasets. Instead of putting 1,000,000 log IDs in an array inside the sensor document, put 1 sensor ID inside each of the 1,000,000 log documents!
-->

---

# Rule 4 & Rule 5
## Joins, Read/Write Ratios & Access Patterns

- **Rule 4: Don't be afraid of application-level joins or `$lookup`.**
  - Modern drivers and network speeds execute application-level async fetches in parallel effortlessly.
- **Rule 5: Consider read/write ratio.**
  - **Read-Heavy (90% reads):** Denormalize and pre-compute aggressively to optimize reads.
  - **Write-Heavy (90% writes):** Keep documents normalized and light to minimize write amplification.

<!--
Speaker Notes:
Always analyze your access patterns first. Are you building an analytics dashboard with 99% read traffic, or an IoT ingress pipeline with 99% write traffic?
-->
---

# Rule 6
- **Rule 6: How you model data depends entirely on your access patterns.**
  - Design for query predicates, filter parameters, and view requirements.

---

# Module 5
### The MongoDB 4-Step Schema Design Process

```
[Step 1: Workload Analysis] ➔ [Step 2: Map Relationships] ➔ [Step 3: Apply Design Patterns] ➔ [Step 4: Index Strategy]
```

1. **Step 1: Identify Application Workload & Access Patterns**
   - List operations, read/write ratios, latency SLAs, and query volume.
2. **Step 2: Map Relationships & Cardinality**
   - Identify entities, relationship types (`1:1`, `1:N`, `N:M`), and quantifiable bounds.
3. **Step 3: Apply Schema Design Patterns**
   - Select appropriate architectural patterns (Polymorphic, Subset, Bucket, etc.).
4. **Step 4: Create Supporting Indexes**
   - Ensure ESR rule (Equality, Sort, Range) indexes back your schema queries.

<!--
Speaker Notes:
This 4-step framework is the standard methodology documented by MongoDB engineering.
-->
---

# Module 6
### Architectural Schema Design Patterns

- Practical patterns for high-scale applications
- Examples reference JSON files in `./examples/` directory:
  - `01_polymorphic_pattern.json`
  - `02_attribute_pattern.json`
  - `03_bucket_pattern.json`
  - `04_extended_reference.json`
  - `05_subset_pattern.json`
  - `06_computed_pattern.json`
  - `07_schema_versioning_pattern.json`

<!--
Speaker Notes:
Now let us examine the top architectural patterns used in production systems. We will look at real JSON document structures.
-->

---

# 1. Polymorphic Pattern
## Storing Varying Entity Types in One Collection

- **Problem:** E-commerce stores sell bikes, helmets, and apparel. Each item type has distinct attributes.
- **Solution:** Use a shared collection with an `item_type` discriminator field and flexible subdocument specs.
- **Reference File:** `examples/01_polymorphic_pattern.json`

<!--
Speaker Notes:
In SQL, polymorphism requires multiple table joins or sparse columns full of NULLs. In MongoDB, documents naturally hold different fields cleanly.
-->

---

```json
[
  { "sku": "BIKE-01", "item_type": "BICYCLE", "specs": { "gears": 12, "wheel_size": 29 } },
  { "sku": "HELMET-01", "item_type": "HELMET", "specs": { "size": "M", "safety": ["DOT"] } }
]
```

---

# 2. Attribute Pattern
## Indexing Unpredictable Key-Value Pairs

- **Problem:** Products have hundreds of custom or user-defined specs (RAM size, color, voltage). Creating compound indexes on unpredictable field names is impossible.
- **Solution:** Transform fields into an array of `{ k: key, v: value }` subdocuments.
- **Reference File:** `examples/02_attribute_pattern.json`

- **Compound Index:** `{ "attributes.k": 1, "attributes.v": 1 }` indexes ALL attributes at once!

<!--
Speaker Notes:
With a single index on attributes.k and attributes.v, users can search across any product property instantly without creating hundreds of indexes.
-->
---

```json
{
  "product_name": "Pro Gaming Laptop",
  "attributes": [
    { "k": "ram_gb", "v": 32 },
    { "k": "gpu_model", "v": "NVIDIA RTX 4080" }
  ]
}
```

---

# 3. Bucket Pattern
## Streamlining Time-Series & High-Frequency IoT Data

- **Problem:** An IoT temperature sensor sending 1 reading per second creates 86,400 documents per day per sensor, causing high index overhead and RAM thrashing.
- **Solution:** Group readings into hourly/daily time bucket documents with pre-computed summary metrics.
- **Reference File:** `examples/03_bucket_pattern.json`

<!--
Speaker Notes:
The Bucket Pattern compresses thousands of documents into a few bucketed documents, reducing index size by 99% and enabling fast aggregations.
-->

---

```json
{
  "device_id": "SENSOR-TEMP-NYC-009",
  "bucket_start_date": "2026-09-11T10:00:00Z",
  "count": 60,
  "stats": { "min_temp_c": 21.2, "max_temp_c": 24.8, "avg_temp_c": 22.95 },
  "readings": [ { "timestamp": "...", "temp_c": 21.2 } ]
}
```

---

# 4. Extended Reference Pattern
## Eliminating `$lookup` JOINs for Frequent Read Queries

- **Problem:** An Order needs customer details. Storing only `customer_id` forces a `$lookup` join on every order list view. Storing the whole customer document duplicates data that changes often.
- **Solution:** Embed ONLY the frequently accessed, rarely changed fields (e.g., name, email) into the Order.
- **Reference File:** `examples/04_extended_reference.json`


<!--
Speaker Notes:
Notice how we only copy the name and email. The customer's full payment methods, address history, and security settings stay in the main customer collection.
-->

---

```json
{
  "order_number": "ORD-2026-9921",
  "customer": {
    "customer_id": "651a2b3c4d5e6f7a8b9c0d00",
    "name": "Alberto Camarena",
    "email": "alberto@example.com"
  }
}
```

---

# 5. Subset Pattern
## Working Set Optimization & Managing Large Arrays

- **Problem:** An e-commerce product has 1,000 reviews. Embedding all 1,000 reviews inflates document size to 1MB. 99% of users only read the top 5 reviews on the product detail page!
- **Solution:** Embed the top 10 most recent/popular reviews in the main product document, and store remaining reviews in a separate `reviews` collection.
- **Reference File:** `examples/05_subset_pattern.json`


<!--
Speaker Notes:
The Subset Pattern keeps the primary document small and fast so it fits neatly into WiredTiger RAM working set.
-->

---

```json
{
  "title": "Legendary Trail Mountain Bike",
  "total_reviews_count": 842,
  "recent_top_reviews": [ { "author": "Maria S.", "rating": 5, "comment": "Best bike!" } ]
}
```

---

# 6. Computed Pattern
## Pre-aggregating Values on Write for Lightning Reads

- **Problem:** Calculating average movie rating across 100,000 votes during read queries consumes massive CPU and delays page loads.
- **Solution:** Compute and update summary statistics during write/update operations (`$inc`).
- **Reference File:** `examples/06_computed_pattern.json`

- When a new vote arrives:
  `db.movies.updateOne({ _id: id }, { $inc: { total_votes: 1, sum_of_ratings: rating }, $set: { avg_rating: newAvg } })`

<!--
Speaker Notes:
Shift the computation cost from read time to write time. Since most web applications have a 100:1 read-to-write ratio, pre-computing saves massive compute overhead.
-->

---

```json
{
  "movie_title": "Interstellar Journey",
  "total_votes": 12500,
  "sum_of_ratings": 112500,
  "avg_rating": 9.0
}
```

---

# 7. Schema Versioning Pattern
## Zero-Downtime Database Schema Migrations

- **Problem:** Changing document structures in SQL requires downtime or complex `ALTER TABLE` steps.
- **Solution:** Include a `schema_version` field. Your application code handles legacy versions on read and lazily updates documents to current versions on write.
- **Reference File:** `examples/07_schema_versioning_pattern.json`

<!--
Speaker Notes:
Schema versioning allows continuous delivery. Old code can write v1 documents, new code can read v1 or v2, and migrate documents seamlessly without system downtime.
-->

---

```json
{
  "schema_version": 2,
  "username": "johndoe",
  "contact": { "email": "john@example.com", "phone": "+1-555-0199" }
}
```

---

# Modeling Tree Structures in MongoDB
## 4 Approaches for Hierarchical & Categorical Data

1. **Parent References:** Sub-categories store `parent_id`. Excellent for immediate parent lookups.
2. **Child References:** Parent categories store `children: [ id1, id2 ]`.
3. **Array of Ancestors:** Sub-categories store ancestor chain `ancestors: [ "Electronics", "Computers", "Laptops" ]`. Fast subtree queries!
4. **Materialized Paths:** Path strings stored as fields `path: ",Electronics,Computers,Laptops,"`. Enables regex tree matching.

<!--
Speaker Notes:
Tree structures like e-commerce categories or organizational charts can be modeled cleanly using Array of Ancestors or Materialized Paths in MongoDB.
-->

---

# Module 7
## Schema Antipatterns & Pitfalls To Avoid

- Unbounded / Massive Arrays
- Relational Porting (Over-Referencing & Excessive `$lookup`)
- Bloated Documents & RAM Thrashing
- Unindexed Scans & Case-Insensitive Queries

<!--
Speaker Notes:
Knowing what NOT to do is just as critical as knowing design patterns. Let us inspect common antipatterns.
-->

---

# Antipattern 1: Unbounded Arrays
## The Silent Destroyer of Document Limits

- **Pitfall:** Pushing entries infinitely into a document array (e.g., logging activity logs or clicks directly in user documents).
- **Consequences:**
  - Exceeds MongoDB **16MB Document Limit**.
  - Document reallocation: As documents grow on disk, MongoDB must reallocate space, causing write latency spikes.
- **Reference Antipattern File:** `examples/08_antipattern_unbounded_array.json`
- **Fix:** Use Parent Referencing or the Bucket Pattern!

<!--
Speaker Notes:
Never let an array grow indefinitely without bounds. Always cap arrays or switch to parent referencing.
-->

---

# Antipattern 2: Relational Porting & Over-Referencing

- **Pitfall:** Treating MongoDB like a relational database by creating tiny 2-column collections and joining 8 collections per query with `$lookup`.
- **Consequences:** CPU spikes, slow query latencies, and loss of document model benefits.
- **Fix:** Embrace embedding and extended references for data accessed together!

```
BAD:  [User Collection] <->$lookup-> [Email Collection] <-$lookup-> [Address Collection]
GOOD: Single User Document containing email array and address array
```

<!--
Speaker Notes:
Do not port SQL tables 1-to-1 into MongoDB. If you find yourself writing $lookup chains everywhere, redesign your schema.
-->

---

# Antipattern 3: Bloated Documents
- Storing massive binary blobs or historical data in hot collections ejects active working sets from WiredTiger RAM memory.
- **Fix:** Use GridFS or S3 for large binaries; use Subset Pattern for historical data.

<!--
Speaker Notes:
Always inspect queries using explain("executionStats") in MongoDB Compass or mongosh to ensure indexes back your schemas.
-->

---

# Antipattern 4: Unindexed & Case-Insensitive Scans
- Running `{ username: /^alberto$/i }` without collation or indexes forces a **COLLSCAN** (scanning every document in the collection).
- **Fix:** Use case-insensitive indexes with MongoDB Collation or normal exact match indexes.


---

# Module 8
## Schema Validation, Mastering MongoDB & Skill Badges

- Enforcing Quality with `$jsonSchema`
- Book Recommendation: *Mastering MongoDB 8.0*
- Claim Your Free Official MongoDB Skill Badges!

<!--
Speaker Notes:
To wrap up our workshop, let us explore database-level validation, reference books, and official skill certification.
-->

---

# Database-Level Schema Validation
## Combining Document Flexibility with Enterprise Rules

MongoDB allows enforcing structural constraints at the collection level via **JSON Schema (`$jsonSchema`)**:

<!--
Speaker Notes:
JSON Schema validation ensures invalid documents are rejected at write time while preserving flexible subdocument structures.
-->

---

```javascript
db.createCollection("users", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: [ "username", "email", "schema_version" ],
      properties: {
        username: { bsonType: "string", description: "must be a string" },
        email: { bsonType: "string", pattern: "^.+@.+$" },
        schema_version: { bsonType: "int", minimum: 1 }
      }
    }
  }
});
```

---

# Essential Reading Shoutout
## Mastering MongoDB 8.0

> *"Mastering MongoDB 8.0"* is the definitive guide to designing scalable document schemas, optimizing performance, and mastering MongoDB Atlas.

### Key Takeaways from the Book:
- Deep-dive into WiredTiger storage engine mechanics and cache management.
- Production-grade aggregation pipelines and multi-document ACID transactions.
- Advanced sharding strategy and global database architecture.

<!--
Speaker Notes:
If you want to master MongoDB engineering, "Mastering MongoDB 8.0" is a must-read book.
-->

---

# 🎓 Earn Your Official MongoDB Skill Badge!

Take your MongoDB knowledge further and add official credentials to your resume & LinkedIn profile!

### 🌟 Free Official Skill Badges:
- **Badge Course:** *Data Modeling for MongoDB / Schema Design*
- **URL:** [https://learn.mongodb.com/](https://learn.mongodb.com/)

```
  [ Learn MongoDB University ] ➔ [ Take Free Course ] ➔ [ Earn Verified Skill Badge ]
```

- Learn for free from MongoDB engineers
- Complete hands-on lab challenges
- Claim your shareable digital badge on LinkedIn & GitHub!

<!--
Speaker Notes:
Visit learn.mongodb.com today! Complete the Data Modeling course and earn your official MongoDB Skill Badge to showcase on your profile.
-->

---

# Workshop Summary & Q&A
## Core Takeaways

1. **Mindset:** Your data takes the shape of your application.
2. **Rules of Thumb:** Default to embedding; reference when growth is unbounded or entities are accessed standalone.
3. **Patterns:** Leverage Polymorphic, Attribute, Bucket, Subset, Extended Reference, and Computed patterns.
4. **Get Certified:** Claim your skill badge at [learn.mongodb.com](https://learn.mongodb.com/)!

### 🚀 Thank you for attending Global Hack Week!
*Presentation slides and JSON schema samples are in `./slides` and `./examples`.*

<!--
Speaker Notes:
Thank you everyone for joining! We are now open for Q&A.
-->
