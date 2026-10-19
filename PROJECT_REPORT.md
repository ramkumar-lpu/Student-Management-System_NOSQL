# Academic Project Report

## Course: Intelligent NoSQL Databases (CSE494)
## Project Title: Complete Student Management System Using MongoDB & PyMongo

---

### Executive Summary & Abstract
Traditional relational database management systems (RDBMS) enforce tabular, normalized schemas that introduce high query latency when calculating hierarchical academic metrics such as semester GPA, departmental grade distributions, and student rankings. These operations demand expensive multi-table `JOIN`s and strict database migrations whenever curriculum parameters evolve.

This project implements a complete, production-grade **Student Management System** leveraging **MongoDB** as a document-oriented NoSQL database and **PyMongo** as the Python database driver, paired with a modern **React (Vite + Tailwind CSS + Recharts)** frontend. The system manages student records, prevents duplicate registrations via unique BSON indexes, enforces data integrity using collection-level `$jsonSchema` validation, and computes complex academic statistics entirely inside MongoDB using multi-stage **Aggregation Pipelines**.

---

## 1. System Architecture & Tech Stack Justification

```
┌────────────────────────────────────────────────────────┐
│          Frontend (React 19 + Vite + Tailwind)         │
│  - Academic Dashboard (KPI Cards, Recharts, Recent)    │
│  - Student Directory (Search, Filter, Sort, Paging)    │
│  - Add / Edit Student Forms (Real-time Validation)     │
│  - Student Details Modal & Safe Deletion Dialog        │
│  - Analytics Page (5 Dedicated Recharts Charts)        │
└───────────────────────────▲────────────────────────────┘
                            │ REST API (JSON / HTTP)
┌───────────────────────────▼────────────────────────────┐
│              Backend (Python Flask + PyMongo)          │
│  - app.py: Application factory, CORS, error handling   │
│  - routes.py: CRUD & Analytics REST endpoints          │
│  - validators.py: Server-side validation logic         │
│  - aggregations.py: 5 Aggregation Pipelines & Stats    │
│  - database.py: PyMongo client, $jsonSchema, indexes   │
└───────────────────────────▲────────────────────────────┘
                            │ PyMongo Driver (BSON)
┌───────────────────────────▼────────────────────────────┐
│                   MongoDB Database                     │
│  - Database: student_management                        │
│  - Collection: students                                │
│  - Schema: $jsonSchema collection-level validation     │
│  - Indexes: unique(roll_no), unique(email), dept, sem  │
└────────────────────────────────────────────────────────┘
```

### Justification of Components:
1. **MongoDB**: High-performance document database storing data in rich, binary-encoded JSON (BSON). Eliminates object-relational impedance mismatch.
2. **PyMongo Driver**: Official low-level driver providing native access to BSON types, connection pooling, and full MongoDB Aggregation framework support without the abstraction overhead of ORMs.
3. **Flask**: Lightweight WSGI microframework offering minimal memory footprint, explicit routing, and clean error handling.
4. **React with Vite**: Fast hot module reloading (HMR) and lightweight component architecture.
5. **Recharts**: Declarative SVG chart library optimized for React applications.

---

## 2. Why NoSQL & MongoDB Over Relational Databases?

| Criteria | Relational Database (e.g., MySQL) | MongoDB (NoSQL Document Store) | Advantage in This Project |
| :--- | :--- | :--- | :--- |
| **Data Model** | Flat rows across multiple tables linked by foreign keys | Hierarchical BSON documents with embedded sub-documents | Marks are stored embedded inside the student profile; single read retrieves full profile |
| **Joins vs Embedding**| Requires multi-table `JOIN`s for student details and subject scores | Data is pre-joined; zero joins needed for standard read operations | Greatly reduces query latency and disk I/O |
| **Schema Flexibility**| Rigid schema; alterations require `ALTER TABLE` locks | Dynamic schema with optional `$jsonSchema` validation | Can support elective subjects or dynamic grading without table alterations |
| **Analytics** | SQL Group By queries; often slow across joined tables | In-memory Aggregation Pipelines (`$project`, `$group`, `$switch`) | Analytics execute directly inside the database engine |
| **Scalability** | Vertical scaling (larger servers); complex sharding | Native horizontal sharding and replica set clustering | Scalable to millions of student records across distributed nodes |

---

## 3. MongoDB Database Design & Schema Modeling

### Document Structure:
```json
{
  "_id": ObjectId("6ad5d5451e3176333086b99f"),
  "roll_no": "CSE2024001",
  "name": "Aarav Sharma",
  "email": "aarav.sharma@example.com",
  "department": "CSE",
  "semester": 6,
  "marks": {
    "DBMS": 92,
    "DSA": 96,
    "Python": 94
  },
  "created_at": ISODate("2026-10-19T04:31:01.953Z")
}
```

### Field Definitions & BSON Types:
- `_id`: 12-byte BSON ObjectId generated automatically by MongoDB.
- `roll_no`: String, unique identifier across institution.
- `name`: String, student full name.
- `email`: String, unique contact email.
- `department`: String, one of `['CSE', 'ECE', 'MECH', 'CIVIL', 'IT', 'AI&DS']`.
- `semester`: Integer bounded between 1 and 8.
- `marks`: Embedded document with integer/double scores for `DBMS`, `DSA`, and `Python` ($0 \le \text{mark} \le 100$).
- `created_at`: Native BSON Date object representing enrollment timestamp.

### Rationale for Nested Documents:
Embedding subject marks inside `marks` preserves **data locality**. When querying a student record, the marks are stored contiguously on disk with the student document. Fetching the record requires a single seek rather than querying a separate `marks` table.

---

## 4. Database Validation & Indexing Strategy

### A. Collection-Level `$jsonSchema` Validation
Enforced via `db.create_collection("students", validator=STUDENT_JSON_SCHEMA)` and `collMod`:
```json
{
  "$jsonSchema": {
    "bsonType": "object",
    "required": ["roll_no", "name", "email", "department", "semester", "marks", "created_at"],
    "properties": {
      "_id": { "bsonType": "objectId" },
      "roll_no": { "bsonType": "string" },
      "name": { "bsonType": "string" },
      "email": { "bsonType": "string", "pattern": "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$" },
      "department": { "bsonType": "string", "enum": ["CSE", "ECE", "MECH", "CIVIL", "IT", "AI&DS"] },
      "semester": { "bsonType": ["int", "long", "double"], "minimum": 1, "maximum": 8 },
      "marks": {
        "bsonType": "object",
        "required": ["DBMS", "DSA", "Python"],
        "properties": {
          "DBMS": { "bsonType": ["int", "long", "double"], "minimum": 0, "maximum": 100 },
          "DSA": { "bsonType": ["int", "long", "double"], "minimum": 0, "maximum": 100 },
          "Python": { "bsonType": ["int", "long", "double"], "minimum": 0, "maximum": 100 }
        },
        "additionalProperties": false
      },
      "created_at": { "bsonType": "date" }
    },
    "additionalProperties": false
  }
}
```

### B. Indexing Strategy:
1. `idx_unique_roll_no`: Unique index on `roll_no` ($O(\log N)$ lookup and prevents duplicates).
2. `idx_unique_email`: Unique index on `email`.
3. `idx_department`: Speeds up filtering by department and `$group` stage.
4. `idx_semester`: Speeds up semester filtering and cohort aggregation.
5. `idx_name`: Optimizes sorting by student name and regex searching.

---

## 5. Detailed Analysis of the 5 Aggregation Pipelines

All 5 aggregation pipelines execute directly on the database engine through PyMongo:

### Pipeline 1: Department-wise Student Count
- **Purpose**: Aggregates student enrollment per academic department.
- **Pipeline Definition**:
  ```python
  [
      {"$group": {"_id": "$department", "count": {"$sum": 1}}},
      {"$sort": {"count": -1, "_id": 1}},
      {"$project": {"_id": 0, "department": "$_id", "count": 1}}
  ]
  ```
- **Execution Mechanism**: The `$group` stage uses the `idx_department` index to partition records, sums occurrences using `{"$sum": 1}`, and `$sort` orders departments descending.

### Pipeline 2: Average Marks by Department
- **Purpose**: Computes institutional academic performance comparing disciplines.
- **Pipeline Definition**:
  ```python
  [
      {
          "$project": {
              "department": 1,
              "student_avg": {
                  "$divide": [
                      {"$add": ["$marks.DBMS", "$marks.DSA", "$marks.Python"]},
                      3.0
                  ]
              }
          }
      },
      {
          "$group": {
              "_id": "$department",
              "average_marks": {"$avg": "$student_avg"},
              "student_count": {"$sum": 1}
          }
      },
      {
          "$project": {
              "_id": 0,
              "department": "$_id",
              "average_marks": {"$round": ["$average_marks", 2]},
              "student_count": 1
          }
      },
      {"$sort": {"average_marks": -1}}
  ]
  ```

### Pipeline 3: Top 5 Students Ranked by Average Marks
- **Purpose**: Identifies top academic achievers and calculates letter grades.
- **Pipeline Definition**:
  ```python
  [
      {
          "$project": {
              "roll_no": 1, "name": 1, "email": 1, "department": 1, "semester": 1, "marks": 1,
              "average_marks": {
                  "$round": [
                      {"$divide": [{"$add": ["$marks.DBMS", "$marks.DSA", "$marks.Python"]}, 3.0]},
                      2
                  ]
              }
          }
      },
      {
          "$addFields": {
              "grade": {
                  "$switch": {
                      "branches": [
                          {"case": {"$gte": ["$average_marks", 80]}, "then": "A"},
                          {"case": {"$gte": ["$average_marks", 70]}, "then": "B"},
                          {"case": {"$gte": ["$average_marks", 60]}, "then": "C"},
                          {"case": {"$gte": ["$average_marks", 40]}, "then": "D"}
                      ],
                      "default": "F"
                  }
              }
          }
      },
      {"$sort": {"average_marks": -1, "name": 1}},
      {"$limit": 5}
  ]
  ```

### Pipeline 4: Academic Grade Distribution
- **Purpose**: Groups students into canonical letter grades (`A`, `B`, `C`, `D`, `F`) based on marks.
- **Pipeline Definition**:
  ```python
  [
      {
          "$project": {
              "average_marks": {
                  "$divide": [{"$add": ["$marks.DBMS", "$marks.DSA", "$marks.Python"]}, 3.0]
              }
          }
      },
      {
          "$project": {
              "grade": {
                  "$switch": {
                      "branches": [
                          {"case": {"$gte": ["$average_marks", 80]}, "then": "A"},
                          {"case": {"$gte": ["$average_marks", 70]}, "then": "B"},
                          {"case": {"$gte": ["$average_marks", 60]}, "then": "C"},
                          {"case": {"$gte": ["$average_marks", 40]}, "then": "D"}
                      ],
                      "default": "F"
                  }
              }
          }
      },
      {"$group": {"_id": "$grade", "count": {"$sum": 1}}},
      {"$project": {"_id": 0, "grade": "$_id", "count": 1}}
  ]
  ```

### Pipeline 5: Semester-wise Student Count
- **Purpose**: Counts enrollments across semesters 1 to 8.
- **Pipeline Definition**:
  ```python
  [
      {"$group": {"_id": "$semester", "count": {"$sum": 1}}},
      {"$sort": {"_id": 1}},
      {"$project": {"_id": 0, "semester": "$_id", "count": 1}}
  ]
  ```

---

## 6. PyMongo Backend & CRUD Implementation

### Key Implementation Details:
1. **Connection Pooling**: `get_client()` maintains a single, reusable `MongoClient` instance with timeout safeguards.
2. **Duplicate Key Handling**: PyMongo raises `DuplicateKeyError` (code `11000`) when a write violates `idx_unique_roll_no` or `idx_unique_email`. The backend intercepts this error and returns HTTP 409 Conflict with field-specific guidance.
3. **Schema Violation Handling**: MongoDB returns `OperationFailure` (code `121`) when `$jsonSchema` rules are violated. The backend transforms this into HTTP 400 Bad Request.
4. **BSON Serialization**: `serialize_student()` converts native BSON `ObjectId` and `datetime` into JSON-compliant strings and calculates `average_marks` and `grade`.

---

## 7. Automated Testing & Verification Report

### Test Execution Strategy:
Tests execute against a dedicated, isolated test database (`student_management_test`) created dynamically during test suite initialization and torn down upon completion. This ensures production/sample data in `student_management` is never wiped.

### Test Execution Command:
```powershell
python -m unittest tests/test_api.py
```

### Results Summary:
```
Ran 14 tests in 1.235s
OK
```

| Test Case Name | Objective | Result |
| :--- | :--- | :--- |
| `test_health_check` | MongoDB ping & status check | **PASSED** |
| `test_create_student_success` | Full document insertion with BSON date | **PASSED** |
| `test_duplicate_roll_number` | Rejects duplicate roll_no (HTTP 409) | **PASSED** |
| `test_duplicate_email` | Rejects duplicate email (HTTP 409) | **PASSED** |
| `test_invalid_marks_out_of_bounds` | Rejects marks $>100$ and $<0$ | **PASSED** |
| `test_invalid_semester` | Rejects semester $>8$ | **PASSED** |
| `test_invalid_email_format` | Rejects non-email string | **PASSED** |
| `test_get_student_by_id` | Document lookup by ObjectId | **PASSED** |
| `test_get_nonexistent_student` | Returns HTTP 404 for missing ID | **PASSED** |
| `test_search_and_filter_students` | Regex name search & department filter | **PASSED** |
| `test_update_student` | Updates marks and recalculates grade | **PASSED** |
| `test_delete_student` | Deletes document and confirms 404 | **PASSED** |
| `test_aggregation_pipelines` | Validates all 5 aggregation pipelines | **PASSED** |
| `test_dashboard_stats` | Validates KPI counts and top scorer | **PASSED** |

---

## 8. Conclusion & Viva Defense Summary

The project demonstrates the practical application of NoSQL document database concepts for Intelligent Databases (CSE494):
1. **Embedded Data Modeling**: Proves higher read throughput and simpler document management than relational multi-table normalization.
2. **PyMongo Driver Proficiency**: Demonstrates low-level BSON handling, indexing, and pipeline execution.
3. **Database-Level Integrity**: Combines collection-level `$jsonSchema` validation with unique B-Tree indexes.
4. **Data Aggregation**: Successfully computes real-time academic statistics without moving raw data outside MongoDB.
