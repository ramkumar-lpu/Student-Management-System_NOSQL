# Academic Submission: Screenshots Checklist & Capture Guide
### Course: Intelligent NoSQL Databases (CSE494)
### Project: Student Management System Using MongoDB & PyMongo

This guide provides a structured checklist of all the essential screenshots you should capture from your browser, terminal, and MongoDB Compass for your college project report and viva presentation.

---

## Screenshot Checklist

| # | Screenshot Name | Application View | Description / What to Highlight |
| :- | :--- | :--- | :--- |
| **01** | `01_dashboard_overview.png` | Web Browser (`localhost:5173`) | **Main Dashboard View**: Highlight the 4 KPI cards (Total Students: 16, Departments: 6, Overall Average: 67.0%, Top Performer), the Department Distribution bar chart, and the Grade Distribution chart. |
| **02** | `02_students_table_view.png` | Web Browser (`localhost:5173`) | **Students Management Directory**: Shows the responsive table with Roll Numbers, Names, Emails, Departments, Semesters, Marks breakdown, Average scores, Grade badges, and pagination bar. |
| **03** | `03_students_search_filter.png`| Web Browser (`localhost:5173`) | **Live Filtering & Sorting**: Type a search query (e.g., `"Rahul"` or `"CSE"`), select a department filter (e.g., `CSE`), and sort by `Average Marks` descending. |
| **04** | `04_add_student_modal.png` | Web Browser (`localhost:5173`) | **Enroll New Student Modal**: Shows the clean form with Roll Number, Name, Email, Department dropdown, Semester select, and subject marks for DBMS, DSA, and Python. |
| **05** | `05_form_validation_error.png` | Web Browser (`localhost:5173`) | **Input Validation in Action**: Enter an invalid mark (e.g., `120` or `-5`) or an invalid email format and click save. Highlight the red validation error message. |
| **06** | `06_duplicate_key_error.png` | Web Browser (`localhost:5173`) | **Duplicate Key Protection**: Attempt to add a student with an existing roll number (e.g., `CSE2024001`). Highlight the MongoDB HTTP 409 Conflict toast notification. |
| **07** | `07_student_details_modal.png` | Web Browser (`localhost:5173`) | **Student Profile & Marks Modal**: Click "Details" on any student. Highlight the subject marks breakdown bars (DBMS, DSA, Python), overall average percentage, and Grade badge. |
| **08** | `08_edit_student_modal.png` | Web Browser (`localhost:5173`) | **Update Student Record**: Click the edit button, modify a student's marks, and observe the live estimated average calculation before saving. |
| **09** | `09_delete_confirm_dialog.png` | Web Browser (`localhost:5173`) | **Deletion Safeguard**: Click the trash icon on a student to show the confirmation modal showing student name and roll number before deletion. |
| **10** | `10_analytics_pipelines.png` | Web Browser (`localhost:5173`) | **MongoDB Analytics Page**: Scroll through the 5 Aggregation Pipeline charts (Department counts, Department averages, Grade distribution, Semester counts, and Top 5 Leaderboard). Highlight the stage explanation boxes (`$group`, `$sort`, `$project`, `$switch`). |
| **11** | `11_top_5_leaderboard.png` | Web Browser (`localhost:5173`) | **Top 5 Leaderboard Table**: Shows the top 5 ranked students with gold, silver, and bronze rank medals and grade A scores. |
| **12** | `12_mongodb_compass_docs.png` | MongoDB Compass / mongosh | **MongoDB Database Inspection**: Open MongoDB Compass, connect to `mongodb://localhost:27017/`, open database `student_management` and collection `students`. Show the documents with native BSON `ObjectId`, nested `marks`, and native `created_at` Date. |
| **13** | `13_mongodb_indexes.png` | MongoDB Compass | **Indexes Tab in Compass**: Show the indexes: `idx_unique_roll_no` (Unique), `idx_unique_email` (Unique), `idx_department`, and `idx_semester`. |
| **14** | `14_test_suite_passing.png` | PowerShell Terminal | **Automated Tests Passing**: Run `python -m unittest tests/test_api.py` and capture the terminal output showing `Ran 14 tests in 1.235s OK`. |

---

## Recommended Step-by-Step Instructions to Capture

1. **Start Backend**:
   ```powershell
   cd c:\Users\ramku\Downloads\NOSQL_SMS\backend
   .\venv\Scripts\python.exe app.py
   ```
2. **Start Frontend**:
   ```powershell
   cd c:\Users\ramku\Downloads\NOSQL_SMS\frontend
   npm run dev
   ```
3. Open Google Chrome or Microsoft Edge at `http://localhost:5173`.
4. Press `Windows Key + Shift + S` (Snipping Tool on Windows) to capture clear rectangular screenshots of each view listed above.
5. Save the images in an `images/` or `screenshots/` directory for your final report or presentation slides.
