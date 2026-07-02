# 🎓 Training Center Management System

**Training Center Management System** là hệ thống quản lý trung tâm đào tạo, hỗ trợ quản lý học viên, khóa học, lớp học, lịch học, điểm danh, học phí và báo cáo.

Dự án được xây dựng theo mô hình chia module/service để nhiều nhóm có thể phát triển song song mà không ảnh hưởng trực tiếp đến nhau.

---

## 📌 Prerequisites

Before you continue, ensure you meet the following requirements:

* You have installed **Git**.
* You have installed **Node.js** and **npm** for frontend development.
* You have installed **Java JDK** for backend services.
* You have installed **Maven** or **Gradle**, depending on each backend service.
* You have a basic understanding of **REST API**.
* You have a basic understanding of **Git workflow**.
* You are using **Windows, Linux, or macOS**.

---

## 🧱 Project Structure

```text
training-center-management/
├─ frontend/
│  └─ vue-client/
│
├─ gateway/
│
├─ services/
│  ├─ student-attendance-service/
│  ├─ course-schedule-service/
│  └─ payment-report-service/
│
├─ .gitignore
└─ README.md
```

---

## 🧩 Modules

### 🖥️ Frontend

Path:

```text
frontend/vue-client/
```

Main responsibilities:

* Build user interfaces.
* Call APIs from backend services.
* Display data for students, courses, schedules, attendance, payments, and reports.

Expected technologies:

* Vue.js
* Vue Router
* Axios
* UI library depending on the frontend team

---

### 🚪 Gateway

Path:

```text
gateway/
```

Main responsibilities:

* Receive requests from frontend.
* Route requests to the correct backend service.
* Handle CORS configuration.
* Can be extended to support authentication and authorization.

---

### 👨‍🎓 Student & Attendance Service

Path:

```text
services/student-attendance-service/
```

Main responsibilities:

* Manage student information.
* Register students into classes.
* Manage attendance.
* View attendance history.
* Calculate attendance percentage.
* Manage learning results.

---

### 📚 Course & Schedule Service

Path:

```text
services/course-schedule-service/
```

Main responsibilities:

* Manage courses.
* Manage classes.
* Manage schedules.
* Manage classrooms.
* Provide class and schedule data for other services.

---

### 💰 Payment & Report Service

Path:

```text
services/payment-report-service/
```

Main responsibilities:

* Manage tuition fees.
* Track payment status.
* Generate revenue statistics.
* Generate reports.

---

## ⚙️ Installation

### 1. Clone the repository

```bash
git clone https://github.com/duytienkaka/training-center-management.git
```

### 2. Move into the project folder

```bash
cd training-center-management
```

### 3. Pull the latest code

```bash
git pull origin main
```

### 4. Open the project in your editor

```bash
code .
```

---

## 🚀 Usage

Each team works inside its assigned folder.

Example:

```text
Frontend team:
frontend/vue-client/

Gateway team:
gateway/

Student & Attendance team:
services/student-attendance-service/

Course & Schedule team:
services/course-schedule-service/

Payment & Report team:
services/payment-report-service/
```

Before starting work, always pull the latest code:

```bash
git pull origin main
```

After finishing work:

```bash
git status
git add .
git commit -m "Describe your changes"
git push origin main
```

---

## 🌿 Git Workflow

To avoid conflicts, each team should work in its own folder and should not modify files from other teams unless necessary.

Recommended workflow:

```bash
git pull origin main
```

Create a new branch:

```bash
git checkout -b feature/your-feature-name
```

Commit your changes:

```bash
git add .
git commit -m "Add your feature"
```

Push your branch:

```bash
git push origin feature/your-feature-name
```

Then create a Pull Request to merge into `main`.

---

## 🤝 How to Contribute

To contribute to this project:

1. Pull the latest code from `main`.
2. Create a new branch for your feature.
3. Work only inside your assigned module folder.
4. Commit your changes with a clear message.
5. Push your branch to GitHub.
6. Create a Pull Request.
7. Wait for review before merging.

Please avoid pushing directly to `main` if the team decides to use Pull Requests.

---

## 👥 Contributors

| Name                      | Role                           |
| ------------------------- | ------------------------------ |
| Team Frontend             | Frontend Development           |
| Team Gateway              | API Gateway Development        |
| Team Student & Attendance | Student and Attendance Service |
| Team Course & Schedule    | Course and Schedule Service    |
| Team Payment & Report     | Payment and Report Service     |

---

## 📄 API Contract

The services communicate with each other through agreed API contracts.

Important rules:

* Do not change API endpoints without notifying other teams.
* Do not change request or response format without team agreement.
* If an API contract changes, update the related documentation.
* Frontend should follow the agreed API contract when building screens.

---

## 📁 About `.gitkeep`

Git does not track empty folders.

The `.gitkeep` file is used to keep empty folders in the repository.

Example:

```text
gateway/.gitkeep
services/course-schedule-service/.gitkeep
services/payment-report-service/.gitkeep
```

When a folder already contains real source code, `.gitkeep` can be removed or kept. It does not affect the project.

---

## 🙈 About `.gitignore`

The `.gitignore` file is used to prevent unnecessary files from being pushed to GitHub.

Examples:

```text
node_modules/
target/
.env
*.log
dist/
build/
```

Do not commit sensitive information such as:

* Database passwords
* Secret keys
* Access tokens
* Private configuration files

---

## 🔥 Future Improvements

Possible improvements for this project:

* Add authentication and authorization.
* Add Docker support.
* Add centralized API documentation.
* Add database migration scripts.
* Add CI/CD pipeline.
* Add unit tests and integration tests.
* Add screenshots of the final system UI.

---

## 📸 Screenshots

Screenshots will be added after the frontend UI is completed.

Example:

```text
docs/screenshots/home-page.png
docs/screenshots/student-management.png
docs/screenshots/course-schedule.png
```

---

## 📬 Contact

For questions or collaboration, please contact the project owner or team leader.

GitHub Repository:

```text
https://github.com/duytienkaka/training-center-management
```

---

## 📜 License

This project is currently used for learning and academic purposes.

License information can be updated later if the project is published as an open-source project.
