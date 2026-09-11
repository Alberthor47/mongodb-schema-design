# 🍃 MongoDB Schema Design Workshop
> **Global Hack Week | Major League Hacking (MLH)**  
> **Facilitator:** Alberto Camarena

Welcome to the **MongoDB Schema Design Workshop** repository! This workshop transitions software developers from a traditional relational (SQL) normalization mindset to modern, high-performance document schema design in MongoDB.

---

## 🎯 Learning Objectives

By completing this workshop, participants will be able to:
1. Shift from the SQL paradigm (*"the app takes the form of the data"*) to the document paradigm (*"your data takes the shape of your application"*).
2. Quantify relationship cardinality (`1:1`, `1:few`, `1:many`, `1:squillions`, `N:M`).
3. Apply the **6 Rules of Thumb for MongoDB Schema Design**.
4. Execute MongoDB's official **4-Step Schema Design Methodology**.
5. Implement production schema design patterns (Polymorphic, Attribute, Bucket, Extended Reference, Subset, Computed, Schema Versioning).
6. Avoid fatal schema antipatterns like unbounded arrays, over-referencing, and document bloat.
7. Implement database-level schema validation using `$jsonSchema`.
8. Earn an official **MongoDB Skill Badge**.

---

## 📂 Repository Structure

```
.
├── slides/
│   ├── slides.md     # Marp Markdown source presentation (680+ lines with speaker notes)
│   ├── slides.html   # Compiled standalone HTML presentation
│   └── readme.md     # Instructions for Marp slide compilation
└── examples/
    ├── 01_polymorphic_pattern.json           # Varying entity schemas in one collection
    ├── 02_attribute_pattern.json             # Searchable key-value pairs for unpredictable specs
    ├── 03_bucket_pattern.json                # Time-series & high-frequency IoT readings
    ├── 04_extended_reference.json            # Embedding frequent fields to eliminate $lookup
    ├── 05_subset_pattern.json                # Working set optimization for large arrays (reviews)
    ├── 06_computed_pattern.json              # Pre-computed statistics on write
    ├── 07_schema_versioning_pattern.json     # Zero-downtime schema evolution
    └── 08_antipattern_unbounded_array.json   # Anti-pattern: growing array vs. Parent Referencing
```

---

## 🖥️ Viewing & Generating Presentation Slides

The presentation slides are built using [Marp CLI](https://marp.app).

### Option 1: View in Browser
Open `slides/slides.html` directly in any web browser to deliver or view the presentation with slide controls.

### Option 2: Recompile Slides
To regenerate `slides/slides.html` after editing `slides/slides.md`:

```bash
npx -y @marp-team/marp-cli slides/slides.md --output slides/slides.html
```

---

## 📚 Essential Resources & Reading

1. **MongoDB University & Skill Badges:**  
   Earn free official credentials to display on LinkedIn & GitHub:  
   👉 [https://learn.mongodb.com/](https://learn.mongodb.com/)

2. **Official Documentation & Guides:**
   - [MongoDB Schema Design Process](https://www.mongodb.com/es/docs/manual/data-modeling/schema-design-process/)
   - [MongoDB Data Modeling Overview](https://www.mongodb.com/es/docs/manual/data-modeling/)
   - [6 Rules of Thumb for MongoDB Schema Design](https://www.mongodb.com/company/blog/mongodb/6-rules-of-thumb-for-mongodb-schema-design)
   - [Schema Design Patterns & Antipatterns Course](https://learn.mongodb.com/courses/schema-design-patterns-and-antipatterns)

3. **Recommended Book:**
   - *Mastering MongoDB 8.0* by Packt Publishing (Deep-dive into WiredTiger storage engine mechanics, aggregation pipelines, and sharded cluster architectures).

---

## 🎓 License & Community

Created for **Major League Hacking (MLH) Global Hack Week**. Feel free to use and adapt these slides and schema examples for technical workshops and community events!
