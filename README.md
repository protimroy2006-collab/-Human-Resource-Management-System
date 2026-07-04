# Enigma's management - Human Resource Management System (HRMS)  .

Every workday, perfectly aligned. A high-performance, secure, and visually stunning web application built for workforce coordination.

---

## 🚀 Industry-Grade Feature Classification  .



### 1. Identity & Access Management (IAM)
- **Role-Based Access Control (RBAC)**: Enforces hard boundaries between two primary roles:
  - **Admin / HR Officer**: Full management, approval, creation, deletion, and compensation visibility.
  - **Employee**: Limited self-service capabilities (viewing personal profiles, managing attendance, submitting leave requests).
- **Console-Based Multi-Factor Authentication (MFA)**: Verification codes are dynamically printed to the terminal console during registrations to simulate secure out-of-band email verification.
- **Dynamic Image CAPTCHA Verification**: Built-in automated image CAPTCHAs generated dynamically using Pillow to prevent brute-force attacks on signup and signin forms.
- **Application-Level Row-Level Security (RLS)**: Enforces data isolation where the backend validates the user context (`session['user_id']`, `session['role']`) before responding to database queries, preventing horizontal privilege escalations.

### 2. Core Workforce Management & Core HR
- **Employee ID Auto-Generation**: Signup automatically assigns a unique, non-colliding `EMP####` identifier if left blank.
- **Interactive Directory Operations (Admin CRUD)**: HR managers have dedicated local APIs to register new employees and run cascade deletions that clean up related profile details, leaves, payroll records, and attendance.
- **Limited Employee Profile Self-Service**: Employees can edit limited fields (address, phone, profile picture), while Admins maintain editing access over job titles, departments, salaries, and joining dates.

### 3. Time & Attendance Tracking
- **Skeuomorphic Clock Dashboard**: Interactive check-in and check-out tracking buttons with tactile 3D physical states.
- **Status Markers**: Visual indicators highlighting working status: `Present`, `Absent`, `Half-day`, and `Leave`.
- **Granular Views**: Daily and weekly records restricted to own view for employees, and search-filterable company-wide lists for admins.

### 4. Leave & Time-Off Management
- **Request Pipeline**: Employees can request `Paid`, `Sick`, or `Unpaid` leave by selecting start and end dates with optional remarks.
- **HR Review Workflows**: Admins can approve or reject pending leave requests with custom comments.
- **Cascading Attendance Updates**: Once an Admin approves a leave request, the system automatically writes `Leave` status markers in the attendance database for the entire date range.

### 5. Compensation & Benefits (Payroll)
- **Salary Structure Visibility**: Displays details for basic salary, allowances, and deductions.
- **Read-Only Access for Employees**: Payroll summaries are strictly read-only for employees.
- **Admin Control Panel**: Full access for HR officers to adjust basic salary, allowances, and deductions, with automatic calculation of net pay.

---

## 🛠️ Tech Stack & Dependencies
- **Backend Framework**: Python Flask
- **Database Engine**: SQLite (Embedded)
- **Styling Core**: Vanilla CSS (Skeuomorphic depth + Frosted Glassmorphism blur)
- **Image CAPTCHA engine**: Pillow (PIL Fork)
- **Package Manager**: UV (CPython 3.14.3)

---

## 📦 Installation & Setup

1. **Prerequisites**: Ensure you have [uv](https://github.com/astral-sh/uv) installed.
2. **Environment Initialization**:
   ```bash
   uv venv
   .venv\Scripts\activate
   ```
3. **Install Dependencies**:
   ```bash
   uv pip install flask pillow
   ```
4. **Launch the Local server**:
   ```bash
   python app.py
   ```
   The application will start on `http://127.0.0.1:5000/`.

---

## 🧪 Local Verification & Tests
A complete automated test suite is provided to verify authorization logic, CAPTCHAs, ID generation, and Admin CRUD. Run:
```bash
.venv\Scripts\python.exe -m unittest test_app.py
```

---

## 📄 License
This project is licensed under the **MIT License**.

```text
MIT License

Copyright (c) 2026 Enigma's management

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
