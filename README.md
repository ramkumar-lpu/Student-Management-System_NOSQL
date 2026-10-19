# Complete Student Management System (SMS)
### Intelligent NoSQL Databases (Course Code: CSE494)

A full-stack, production-grade, and beginner-friendly Student Management System built using **MongoDB**, **PyMongo**, **Python (Flask)**, and **React (Vite + Tailwind CSS + Recharts)**.

---

## Table of Contents
1. [Project Overview & Problem Statement](#1-project-overview--problem-statement)
2. [Objectives](#2-objectives)
3. [Technology Stack](#3-technology-stack)
4. [System Architecture](#4-system-architecture)
5. [MongoDB Database Design & Schema](#5-mongodb-database-design--schema)
6. [Database Validation & Indexing Strategy](#6-database-validation--indexing-strategy)
7. [The 5 MongoDB Aggregation Pipelines](#7-the-5-mongodb-aggregation-pipelines)
8. [REST API Endpoints](#8-rest-api-endpoints)
9. [Installation & Setup (Windows)](#9-installation--setup-windows)
10. [Sample Data Seeding](#10-sample-data-seeding)
11. [Running Automated Tests](#11-running-automated-tests)
12. [Project Deliverables](#12-project-deliverables)
13. [Top Viva Questions & Answers](#13-top-viva-questions--answers)

---

## 1. Project Overview & Problem Statement

Academic institutions often struggle with rigid, slow relational database systems that require expensive multi-table joins to compute student GPA, grade distributions, and department metrics. Changes to curriculum or exam criteria require complex schema migrations (`ALTER TABLE`).

This project solves this problem by using **MongoDB (NoSQL Document Store)** with **PyMongo**. Student profiles, semester standings, and subject-wise exam marks are stored in a unified, flexible document. Deep academic analytics and grade distributions are computed entirely inside MongoDB using **Aggregation Pipelines**, delivering near-instant responses with zero external data processing overhead.

---

## 2. Objectives

- **Full-Stack Implementation**: Functional, responsive web application connecting a React frontend with a Flask backend.
- **Pure PyMongo Integration**: All database queries, filters, CRUD mutations, and multi-stage aggregations are implemented via `PyMongo` without any ORMs/ODMs.
- **Native MongoDB Schema Validation**: Enforce strict data types, semester ranges ($1 \le \text{sem} \le 8$), and mark boundaries ($0 \le \text{mark} \le 100$) at the database engine level using `$jsonSchema`.
- **5 Mandatory Aggregation Pipelines**: Implement, verify, and document 5 multi-stage aggregation pipelines for academic statistics.
- **Viva Defense Ready**: Clean, readable code with extensive comments and documentation designed for student viva defense.

---

## 3. Technology Stack

| Layer | Technology | Purpose |
| :--- | :--- | :--- |
| **Database** | MongoDB 7.x / 8.x | Document database running locally (`mongodb://localhost:27017/`) or Atlas |
| **DB Driver** | PyMongo 4.8.0 | Python driver for native BSON document queries & aggregation |
| **Backend** | Python 3.11 + Flask 3.0 | Lightweight REST API server with Flask-CORS and Blueprint routing |
| **Frontend** | React 19 + Vite 6 | Fast modern frontend with component-based reactive architecture |
| **Styling** | Tailwind CSS 3.4 | Clean, minimal, academic administration interface |
| **Visualizations**| Recharts 2.15 | Responsive SVG charts (Bar charts, distributions, tooltips) |
| **Icons** | Lucide React | Modern, accessible iconography |
| **Testing** | Python `unittest` | Automated test suite running in an isolated test database |

---

## 4. System Architecture

```
┌────────────────────────────────────────────────────────┐
│          Frontend (React 19 + Vite + Tailwind)         │
│  - Academic Dashboard (KPIs, Charts, Recent Enrollees) │
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

---

## 5. MongoDB Database Design & Schema

### Database: `student_management`
### Collection: `students`

Each student record is stored as an independent BSON document:

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

### Why Store Marks as an Embedded Document?
1. **High Locality of Reference**: A student's marks are always retrieved whenever viewing the student profile. Embedding eliminates expensive `$lookup` / SQL `JOIN` operations.
2. **Atomic Updates**: Updating a student's marks along with semester or profile information happens in a single atomic document write (`update_one`).
3. **Simplicity**: No need for foreign key tables or separate mark sheets collections for standard academic evaluations.

---

## 6. Database Validation & Indexing Strategy

### A. Collection-Level `$jsonSchema` Validation
Enforced via MongoDB `create_collection` and `collMod` in `database.py`:
- `roll_no`: Required non-empty string.
- `name`: Required string (minimum 2 characters).
- `email`: Required string matching standard email regex.
- `department`: Required enum (`['CSE', 'ECE', 'MECH', 'CIVIL', 'IT', 'AI&DS']`).
- `semester`: Required integer bounded between `1` and `8`.
- `marks`: Required embedded document containing `DBMS`, `DSA`, and `Python` (each integer/double between `0` and `100`).
- `created_at`: Required native BSON Date.

### B. Indexing Strategy
To ensure optimal query performance ($O(\log N)$ via B-Tree indexes):
1. **`roll_no` (Unique Index)**: `idx_unique_roll_no` - Enforces uniqueness; prevents duplicate roll numbers.
2. **`email` (Unique Index)**: `idx_unique_email` - Enforces unique student email addresses.
3. **`department` (Index)**: `idx_department` - Speeds up department filtering and `$group` aggregations.
4. **`semester` (Index)**: `idx_semester` - Accelerates semester-based queries and analytics.
5. **`name` (Index)**: `idx_name` - Optimizes text sorting and prefix searching.

---

## 7. The 5 MongoDB Aggregation Pipelines

All pipelines are located in [`backend/aggregations.py`](backend/aggregations.py) with readable stage comments.

### Pipeline 1: Department-wise Student Count
- **Purpose**: Counts students per department and sorts by population descending.
- **Stages**:
  ```python
  [
      {"$group": {"_id": "$department", "count": {"$sum": 1}}},
      {"$sort": {"count": -1, "_id": 1}},
      {"$project": {"_id": 0, "department": "$_id", "count": 1}}
  ]
  ```
- **Sample Output**:
  ```json
  [
    {"department": "CSE", "count": 4},
    {"department": "ECE", "count": 3},
    {"department": "IT", "count": 3}
  ]
  ```

### Pipeline 2: Average Marks by Department
- **Purpose**: Computes each student's average `(DBMS + DSA + Python) / 3`, then calculates the department-wide average.
- **Stages**:
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
- **Purpose**: Computes student average marks, assigns letter grades via `$switch`, and limits to top 5 students.
- **Stages**:
  ```python
  [
      {
          "$project": {
              "roll_no": 1, "name": 1, "department": 1, "semester": 1, "marks": 1,
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
- **Purpose**: Categorizes students into grade buckets (`A`: $\ge 80$, `B`: $70-79$, `C`: $60-69$, `D`: $40-59$, `F`: $< 40$) and counts the frequency of each.
- **Stages**:
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
- **Purpose**: Groups students by semester (1 to 8) to show cohort distribution.
- **Stages**:
  ```python
  [
      {"$group": {"_id": "$semester", "count": {"$sum": 1}}},
      {"$sort": {"_id": 1}},
      {"$project": {"_id": 0, "semester": "$_id", "count": 1}}
  ]
  ```

---

## 8. REST API Endpoints

| Method | Endpoint | Description | Status Codes |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/health` | Verifies live MongoDB connection and total count | `200`, `503` |
| `GET` | `/api/dashboard/stats` | Consolidated metrics for dashboard KPI cards | `200`, `500` |
| `GET` | `/api/students` | List students with search, filters, sorting, pagination | `200`, `500` |
| `GET` | `/api/students/<id>` | Fetch single student by MongoDB ObjectId | `200`, `400`, `404` |
| `POST` | `/api/students` | Create new student document | `201`, `400`, `409` |
| `PUT` | `/api/students/<id>` | Update student profile or subject marks | `200`, `400`, `404`, `409`|
| `DELETE`| `/api/students/<id>` | Remove student document from collection | `200`, `400`, `404` |
| `GET` | `/api/analytics/department-counts` | Pipeline 1: Department distribution | `200` |
| `GET` | `/api/analytics/department-averages` | Pipeline 2: Academic average by dept | `200` |
| `GET` | `/api/analytics/top-students` | Pipeline 3: Top ranked students | `200` |
| `GET` | `/api/analytics/grade-distribution` | Pipeline 4: Grades count (A-F) | `200` |
| `GET` | `/api/analytics/semester-counts` | Pipeline 5: Enrollment per semester | `200` |

---

## 9. Installation & Setup (Windows)

### Prerequisites
- Python 3.10+ (Verified with Python 3.11)
- Node.js 18+ (Verified with Node v22.19)
- MongoDB Server running locally on `localhost:27017` (or MongoDB Atlas connection string)

### Step 1: Clone or Navigate to Directory
```powershell
cd c:\Users\ramku\Downloads\NOSQL_SMS
```

### Step 2: Configure & Start Backend
Open a terminal for the backend:
```powershell
cd backend

# Create virtual environment (if not already created)
python -m venv venv

# Activate virtual environment
.\venv\Scripts\Activate.ps1

# Install requirements
pip install -r requirements.txt

# Seed sample data (16 students across all departments and grades)
python seed.py

# Run the Flask backend
python app.py
```
*Backend will start on `http://127.0.0.1:5000`.*

### Step 3: Configure & Start Frontend
Open a second terminal for the frontend:
```powershell
cd frontend

# Install Node dependencies
npm install

# Start Vite development server
npm run dev
```
*Frontend will be accessible at `http://localhost:5173`.*

---

## 10. Sample Data Seeding

Run the seed script at any time to populate 16 realistic students across 6 departments:
```powershell
cd backend
.\venv\Scripts\python.exe seed.py
```
- Safe to rerun (idempotent upsert by `roll_no`).
- Populates all grade categories: Grade A (5), Grade B (3), Grade C (3), Grade D (3), Grade F (2).

---

## 11. Running Automated Tests

The automated test suite runs against an **isolated test database** (`student_management_test`) so your primary academic data in `student_management` is never touched or deleted.

```powershell
cd backend
.\venv\Scripts\python.exe -m unittest tests/test_api.py
```

### Verified Test Cases (14 Tests Passed):
- `test_health_check`: Verifies live MongoDB ping response.
- `test_create_student_success`: Creates student document with BSON timestamp.
- `test_duplicate_roll_number`: Rejects duplicate roll number with 409 Conflict.
- `test_duplicate_email`: Rejects duplicate email address with 409 Conflict.
- `test_invalid_marks_out_of_bounds`: Rejects marks $> 100$ and negative marks ($< 0$).
- `test_invalid_semester`: Rejects semester values outside $1-8$.
- `test_invalid_email_format`: Rejects invalid email patterns.
- `test_get_student_by_id`: Retrieves document and computes average and grade.
- `test_get_nonexistent_student`: Gracefully returns 404 for missing ObjectId.
- `test_search_and_filter_students`: Tests search regex and department filtering.
- `test_update_student`: Updates student marks and validates grade recalculation.
- `test_delete_student`: Deletes student and verifies 404 on subsequent lookups.
- `test_aggregation_pipelines`: Validates all 5 aggregation pipelines against live MongoDB.
- `test_dashboard_stats`: Tests KPI summary and highest-scoring student calculation.

---

## 12. Project Deliverables

This workspace contains full academic documentation for your CSE494 submission:
- [PROJECT_REPORT.md](PROJECT_REPORT.md): Comprehensive academic project report covering problem statement, NoSQL justification, document design, aggregation pipelines, and test results.
- [VIVA_QUESTIONS.md](VIVA_QUESTIONS.md): 18+ detailed viva questions and answers covering MongoDB, PyMongo, indexes, BSON, and aggregations.
- [SCREENSHOTS_GUIDE.md](SCREENSHOTS_GUIDE.md): Step-by-step checklist of screenshots to capture for submission.

---

## 13. Top Viva Questions & Answers

**Q1: Why choose MongoDB over a Relational Database (MySQL) for this project?**
> In student management, marks and profile data are inherently hierarchical. In MongoDB, subject marks are stored as an embedded sub-document inside the student record. This gives high locality of reference—all data is retrieved in a single disk read without complex multi-table SQL `JOIN`s.

**Q2: What is the difference between BSON and JSON?**
> JSON is text-based and supports only basic types (string, number, boolean, array, object, null). BSON (Binary JSON) is a binary serialization format used by MongoDB that extends JSON with additional data types such as `ObjectId`, native `Date`, `int32`, `int64`, and raw binary data.

**Q3: How does MongoDB enforce data integrity without rigid relational schemas?**
> MongoDB uses collection-level `$jsonSchema` validation. We defined rules requiring fields like `roll_no`, `semester` ($1-8$), and `marks` ($0-100$). Any insert or update violating these rules is rejected at the database level with write error code `121`.

**Q4: What is an aggregation pipeline in MongoDB?**
> An aggregation pipeline is a framework for multi-stage data processing modeled after Unix pipes. Documents enter a multi-stage pipeline where each stage (e.g., `$project`, `$group`, `$sort`, `$limit`, `$switch`) transforms the documents into aggregated results.
