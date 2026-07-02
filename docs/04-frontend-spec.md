# Frontend VueJS Specification

## 1. Mục tiêu frontend

Frontend xây dựng giao diện quản lý trung tâm đào tạo cho 3 role:

* Admin
* Teacher
* Student

Frontend sử dụng VueJS và gọi API thông qua API Gateway. Frontend chỉ gọi Public API, không gọi Internal API giữa các service.

Hệ thống backend gồm 3 service:

* Course & Schedule Service
* Student & Attendance Service
* Payment & Report Service

Frontend cần đảm bảo:

* Đăng nhập được theo role
* Hiển thị menu đúng role
* Điều hướng đúng màn hình
* Gọi đúng API theo từng service
* Hiển thị dữ liệu rõ ràng
* Có loading, error, empty state
* Có mock data để làm song song với backend

---

## 2. Công nghệ sử dụng

Frontend sử dụng stack đề xuất:

```txt
Vue 3
Vite
TypeScript
Vue Router
Pinia
Axios
Element Plus hoặc Vuetify
```

Có thể chọn UI library khác nếu nhóm quen dùng, nhưng nên thống nhất từ đầu để giao diện đồng bộ.

---

## 3. Quy tắc gọi API

Frontend gọi tất cả API thông qua API Gateway.

Mô hình:

```txt
VueJS Frontend
      |
      v
API Gateway Ocelot
      |
      |-- Course & Schedule Service
      |-- Student & Attendance Service
      |-- Payment & Report Service
```

Frontend chỉ cần cấu hình một base URL:

```env
VITE_API_BASE_URL=https://localhost:7000
```

Khi deploy:

```env
VITE_API_BASE_URL=https://api.training-center.com
```

Frontend không gọi các API dạng:

```txt
/api/.../internal/...
```

Internal API chỉ dùng cho service gọi service.

---

## 4. Quy tắc JWT và Authorization

Sau khi đăng nhập thành công, frontend lưu access token.

Tất cả request sau login cần gửi header:

```txt
Authorization: Bearer {accessToken}
```

Axios interceptor cần tự động gắn token vào request.

Nếu API trả về `401 Unauthorized`:

```txt
Token không hợp lệ hoặc hết hạn
-> Xóa token
-> Redirect về /login
```

Nếu API trả về `403 Forbidden`:

```txt
Người dùng không có quyền
-> Redirect về /unauthorized
```

---

## 5. Auth flow

### 5.1 Đăng nhập

API:

```http
POST /api/payment-report/auth/login
```

Request:

```json
{
  "username": "student@example.com",
  "password": "Abc@123456"
}
```

Response:

```json
{
  "accessToken": "jwt-token",
  "refreshToken": "refresh-token",
  "user": {
    "userId": 1001,
    "fullName": "Nguyễn Văn A",
    "email": "student@example.com",
    "role": "Student",
    "relatedEntityId": 15,
    "relatedEntityType": "Student",
    "mustChangePassword": true
  }
}
```

Sau khi login:

```txt
Nếu mustChangePassword = true -> chuyển sang /change-password
Nếu role = Admin -> /admin/dashboard
Nếu role = Teacher -> /teacher/dashboard
Nếu role = Student -> /student/dashboard
```

---

### 5.2 Đổi mật khẩu lần đầu

API:

```http
POST /api/payment-report/auth/change-password
```

Request:

```json
{
  "oldPassword": "Abc@123456",
  "newPassword": "NewPassword@123"
}
```

Response:

```json
{
  "success": true,
  "message": "Password changed successfully",
  "mustChangePassword": false
}
```

Sau khi đổi mật khẩu thành công:

```txt
Cập nhật authStore.mustChangePassword = false
Redirect user về dashboard theo role
```

---

## 6. Role-based routing

Frontend cần route guard theo role.

Quy tắc:

```txt
Chưa login -> /login
Đã login nhưng mustChangePassword = true -> /change-password
Không đúng role -> /unauthorized
Đúng role -> cho vào trang
```

Role:

```txt
Admin
Teacher
Student
```

---

## 7. Public routes

| Route              | Màn hình           | Mô tả                |
| ------------------ | ------------------ | -------------------- |
| `/login`           | LoginPage          | Đăng nhập            |
| `/change-password` | ChangePasswordPage | Đổi mật khẩu lần đầu |
| `/unauthorized`    | UnauthorizedPage   | Không có quyền       |
| `/not-found`       | NotFoundPage       | Không tìm thấy trang |

---

## 8. Admin routes

| Route                                 | Màn hình                 | Service chính                       |
| ------------------------------------- | ------------------------ | ----------------------------------- |
| `/admin/dashboard`                    | AdminDashboardPage       | Tổng hợp nhiều service              |
| `/admin/courses`                      | CourseListPage           | Course & Schedule                   |
| `/admin/courses/create`               | CourseCreatePage         | Course & Schedule                   |
| `/admin/courses/import`               | CourseImportPage         | Course & Schedule                   |
| `/admin/courses/:id`                  | CourseDetailPage         | Course & Schedule                   |
| `/admin/teachers`                     | TeacherListPage          | Course & Schedule                   |
| `/admin/teachers/create`              | TeacherCreatePage        | Course & Schedule + Payment/Auth    |
| `/admin/teachers/import`              | TeacherImportPage        | Course & Schedule                   |
| `/admin/teachers/:id`                 | TeacherDetailPage        | Course & Schedule                   |
| `/admin/classes`                      | ClassListPage            | Course & Schedule                   |
| `/admin/classes/create`               | ClassCreatePage          | Course & Schedule                   |
| `/admin/classes/:id`                  | ClassDetailPage          | Course & Schedule + Student         |
| `/admin/classes/:id/sessions`         | ClassSessionPage         | Course & Schedule                   |
| `/admin/classes/:id/sessions/import`  | SessionImportPage        | Course & Schedule                   |
| `/admin/classes/:id/enrollments/bulk` | BulkEnrollmentPage       | Student & Attendance                |
| `/admin/students`                     | StudentListPage          | Student & Attendance                |
| `/admin/students/create`              | StudentCreatePage        | Student & Attendance + Payment/Auth |
| `/admin/students/import`              | StudentImportPage        | Student & Attendance                |
| `/admin/students/:id`                 | StudentDetailPage        | Student & Attendance + Payment      |
| `/admin/enrollments`                  | EnrollmentListPage       | Student & Attendance                |
| `/admin/enrollments/pending`          | EnrollmentApprovalPage   | Student & Attendance                |
| `/admin/attendance`                   | AttendanceManagementPage | Student & Attendance                |
| `/admin/results`                      | ResultManagementPage     | Student & Attendance                |
| `/admin/users`                        | UserAccountPage          | Payment & Report                    |
| `/admin/tuition-fees`                 | TuitionFeePage           | Payment & Report                    |
| `/admin/tuition-fees/import`          | TuitionFeeImportPage     | Payment & Report                    |
| `/admin/invoices`                     | InvoiceListPage          | Payment & Report                    |
| `/admin/invoices/:id`                 | InvoiceDetailPage        | Payment & Report                    |
| `/admin/payments`                     | PaymentManagementPage    | Payment & Report                    |
| `/admin/debts`                        | DebtListPage             | Payment & Report                    |
| `/admin/reports/revenue`              | RevenueReportPage        | Payment & Report                    |
| `/admin/reports/classes`              | ClassReportPage          | Payment & Report                    |

---

## 9. Teacher routes

| Route                             | Màn hình                | Service chính               |
| --------------------------------- | ----------------------- | --------------------------- |
| `/teacher/dashboard`              | TeacherDashboardPage    | Tổng hợp nhiều service      |
| `/teacher/classes`                | TeacherClassListPage    | Course & Schedule           |
| `/teacher/classes/:id`            | TeacherClassDetailPage  | Course & Schedule + Student |
| `/teacher/classes/:id/sessions`   | TeacherClassSessionPage | Course & Schedule           |
| `/teacher/schedule`               | TeacherSchedulePage     | Course & Schedule           |
| `/teacher/classes/:id/attendance` | TeacherAttendancePage   | Student & Attendance        |
| `/teacher/classes/:id/results`    | TeacherResultPage       | Student & Attendance        |
| `/teacher/reports/classes/:id`    | TeacherClassReportPage  | Payment & Report            |

---

## 10. Student routes

| Route                           | Màn hình               | Service chính          |
| ------------------------------- | ---------------------- | ---------------------- |
| `/student/dashboard`            | StudentDashboardPage   | Tổng hợp nhiều service |
| `/student/profile`              | StudentProfilePage     | Student & Attendance   |
| `/student/courses`              | StudentCourseListPage  | Course & Schedule      |
| `/student/classes/open`         | OpenClassListPage      | Course & Schedule      |
| `/student/classes/:id`          | StudentClassDetailPage | Course & Schedule      |
| `/student/classes/:id/register` | ClassRegistrationPage  | Student & Attendance   |
| `/student/enrollments`          | MyEnrollmentPage       | Student & Attendance   |
| `/student/schedule`             | MySchedulePage         | Course & Schedule      |
| `/student/attendance`           | MyAttendancePage       | Student & Attendance   |
| `/student/results`              | MyResultPage           | Student & Attendance   |
| `/student/tuition`              | MyInvoicePage          | Payment & Report       |
| `/student/payments`             | MyPaymentHistoryPage   | Payment & Report       |
| `/student/debt`                 | MyDebtPage             | Payment & Report       |

---

## 11. Layout theo role

### 11.1 Admin layout

Menu Admin:

```txt
Dashboard
Khóa học
Giáo viên
Lớp học
Lịch học
Học viên
Đăng ký lớp
Duyệt đăng ký
Điểm danh
Kết quả học tập
Tài khoản
Học phí
Invoice
Thanh toán
Công nợ
Báo cáo
```

### 11.2 Teacher layout

Menu Teacher:

```txt
Dashboard
Lớp tôi dạy
Lịch dạy
Điểm danh
Nhập điểm
Báo cáo lớp
```

### 11.3 Student layout

Menu Student:

```txt
Dashboard
Hồ sơ cá nhân
Khóa học
Lớp đang mở
Đăng ký của tôi
Lịch học
Chuyên cần
Kết quả
Học phí
Thanh toán
Công nợ
```

---

## 12. API mapping theo service

### 12.1 Course & Schedule API

Frontend dùng service này cho:

* Quản lý khóa học
* Quản lý giáo viên
* Quản lý lớp học
* Quản lý lịch học
* Xem lớp đang mở
* Xem lịch dạy/lịch học

API tiêu biểu:

```http
GET /api/course-schedule/admin/courses
POST /api/course-schedule/admin/courses
POST /api/course-schedule/admin/courses/import/preview
POST /api/course-schedule/admin/courses/import/confirm

GET /api/course-schedule/admin/teachers
POST /api/course-schedule/admin/teachers
POST /api/course-schedule/admin/teachers/import/preview
POST /api/course-schedule/admin/teachers/import/confirm

GET /api/course-schedule/admin/classes
POST /api/course-schedule/admin/classes
GET /api/course-schedule/admin/classes/{classId}/sessions
POST /api/course-schedule/admin/classes/{classId}/sessions/bulk

GET /api/course-schedule/teacher/classes
GET /api/course-schedule/teacher/schedule

GET /api/course-schedule/student/classes/open
GET /api/course-schedule/student/classes/{classId}/registration-info
GET /api/course-schedule/student/schedule
```

---

### 12.2 Student & Attendance API

Frontend dùng service này cho:

* Quản lý học viên
* Import học viên
* Đăng ký lớp
* Duyệt đăng ký
* Gán nhiều học viên vào lớp
* Điểm danh
* Nhập điểm
* Xem chuyên cần/kết quả

API tiêu biểu:

```http
GET /api/student-attendance/admin/students
POST /api/student-attendance/admin/students
POST /api/student-attendance/admin/students/import/preview
POST /api/student-attendance/admin/students/import/confirm

GET /api/student-attendance/admin/enrollments/pending
POST /api/student-attendance/admin/enrollments/{enrollmentId}/approve
POST /api/student-attendance/admin/enrollments/{enrollmentId}/reject
POST /api/student-attendance/admin/enrollments/approve-bulk
POST /api/student-attendance/admin/classes/{classId}/enrollments/bulk

GET /api/student-attendance/teacher/classes/{classId}/students
POST /api/student-attendance/teacher/classes/{classId}/sessions/{sessionId}/attendance
POST /api/student-attendance/teacher/classes/{classId}/results

POST /api/student-attendance/student/classes/{classId}/register
GET /api/student-attendance/student/enrollments
GET /api/student-attendance/student/enrollments/{enrollmentId}/timeline
GET /api/student-attendance/student/attendance
GET /api/student-attendance/student/results
```

---

### 12.3 Payment & Report API

Frontend dùng service này cho:

* Login
* Đổi mật khẩu
* Quản lý tài khoản
* Học phí
* Invoice
* Thanh toán
* Công nợ
* Báo cáo

API tiêu biểu:

```http
POST /api/payment-report/auth/login
GET /api/payment-report/auth/me
POST /api/payment-report/auth/change-password

GET /api/payment-report/admin/users
POST /api/payment-report/admin/users/{userId}/reset-password

GET /api/payment-report/admin/tuition-fees
POST /api/payment-report/admin/tuition-fees
POST /api/payment-report/admin/tuition-fees/import/preview
POST /api/payment-report/admin/tuition-fees/import/confirm

GET /api/payment-report/admin/invoices
GET /api/payment-report/admin/invoices/{invoiceId}
POST /api/payment-report/admin/invoices/{invoiceId}/payments

GET /api/payment-report/student/invoices
GET /api/payment-report/student/payments
GET /api/payment-report/student/debt

GET /api/payment-report/admin/reports/revenue
GET /api/payment-report/admin/reports/debt
```

---

## 13. Component dùng chung

Frontend nên tạo các component dùng chung:

```txt
AppLayout
RoleSidebar
PageHeader
DataTable
StatusBadge
ConfirmDialog
ToastNotification
LoadingSkeleton
EmptyState
ImportWizard
BulkActionBar
CalendarView
SearchFilterBar
EnrollmentTimeline
InvoiceStatusCard
DashboardCard
```

---

## 14. Component theo nghiệp vụ

### Course components

```txt
CourseForm
CourseTable
CourseDetail
ClassForm
ClassTable
ClassDetail
SessionTable
SessionCalendar
TeacherForm
TeacherTable
```

### Student components

```txt
StudentForm
StudentTable
StudentDetail
EnrollmentTable
EnrollmentApprovalTable
BulkEnrollmentForm
EnrollmentTimeline
```

### Attendance components

```txt
AttendanceTable
AttendanceStatusSelect
AttendanceSummaryCard
ScoreInputTable
ResultTable
```

### Payment components

```txt
InvoiceTable
InvoiceDetail
PaymentForm
DebtTable
TuitionFeeForm
ReceiptView
```

### Report components

```txt
RevenueChart
DashboardSummaryCards
ClassReportTable
DebtReportTable
```

---

## 15. Mock data cần chuẩn bị

Frontend có thể code song song với backend bằng mock data.

Nên tạo:

```txt
src/mocks/auth.mock.ts
src/mocks/courses.mock.ts
src/mocks/classes.mock.ts
src/mocks/teachers.mock.ts
src/mocks/students.mock.ts
src/mocks/enrollments.mock.ts
src/mocks/attendance.mock.ts
src/mocks/results.mock.ts
src/mocks/invoices.mock.ts
src/mocks/payments.mock.ts
src/mocks/reports.mock.ts
```

Mock data phải bám sát response contract backend.

---

## 16. Enum/status dùng trong UI

### Role

```txt
Admin
Teacher
Student
```

### AccountStatus

```txt
Active
Locked
MustChangePassword
NotCreated
```

### ClassStatus

```txt
Draft
Open
InProgress
Completed
Cancelled
```

### LearningMode

```txt
Offline
Online
Hybrid
```

### EnrollmentStatus

```txt
PendingApproval
Approved
Rejected
Studying
Completed
Dropped
Cancelled
```

### AttendanceStatus

```txt
Present
Absent
Late
Excused
```

### ResultStatus

```txt
Pass
Fail
NotYet
```

### InvoiceStatus

```txt
Unpaid
Partial
Paid
Overdue
Cancelled
```

### ImportStatus

```txt
NotUploaded
Previewing
HasErrors
ReadyToImport
Imported
Failed
```

---

## 17. Import Wizard UX

Các màn import dùng chung component `ImportWizard`.

Flow:

```txt
Tải template
Upload file
Preview dữ liệu
Hiển thị dòng hợp lệ/lỗi
Download file lỗi nếu có
Xác nhận import
```

Các import cần hỗ trợ:

```txt
Import khóa học
Import giáo viên
Import lịch học
Import học viên
Import đăng ký lớp
Import điểm danh
Import điểm số
Import học phí
```

Với import học viên/giáo viên cần có checkbox:

```txt
Tự động tạo tài khoản đăng nhập sau khi import
```

---

## 18. Dashboard theo role

### Admin Dashboard

Hiển thị:

```txt
Tổng số khóa học
Tổng số lớp đang mở
Tổng số học viên
Số đăng ký chờ duyệt
Số lớp gần đầy
Doanh thu tháng
Tổng công nợ
Invoice quá hạn
```

### Teacher Dashboard

Hiển thị:

```txt
Số lớp đang dạy
Lịch dạy hôm nay
Buổi chưa điểm danh
Lớp chưa nhập điểm
Học viên vắng nhiều
```

### Student Dashboard

Hiển thị:

```txt
Lớp đang học
Lịch học sắp tới
Trạng thái đăng ký mới nhất
Tỷ lệ chuyên cần
Kết quả gần nhất
Học phí còn nợ
Hạn thanh toán gần nhất
```

---

## 19. UX bắt buộc

Frontend cần có:

```txt
Loading state
Empty state
Error state
Toast notification
Confirm modal
Status badge
Pagination
Search/filter
Role-based sidebar
Import preview
Bulk action
Enrollment timeline
Calendar view
Change password flow
```

---

## 20. Quy tắc hiển thị account

Ở bảng học viên và giáo viên, nên có cột:

```txt
Account Status
```

Giá trị:

```txt
Created
NotCreated
Locked
MustChangePassword
```

Action:

```txt
Tạo tài khoản
Reset mật khẩu
Khóa tài khoản
Mở tài khoản
```

---

## 21. Quy tắc response/error

Frontend giả định backend trả response thành công dạng:

```json
{
  "success": true,
  "message": "Success",
  "data": {}
}
```

Response lỗi:

```json
{
  "success": false,
  "message": "Validation failed",
  "errors": [
    {
      "field": "email",
      "message": "Email không hợp lệ"
    }
  ]
}
```

Nếu backend không dùng wrapper `success/message/data`, nhóm backend và frontend phải thống nhất lại trước khi code.

---

## 22. Thứ tự code frontend

Thứ tự ưu tiên:

```txt
1. Setup Vue 3 + Vite + TypeScript
2. Setup Vue Router
3. Setup Pinia auth store
4. Setup Axios httpClient
5. Làm LoginPage
6. Làm ChangePasswordPage
7. Làm route guard theo role
8. Làm layout/menu Admin/Teacher/Student
9. Làm component chung
10. Làm Admin Dashboard
11. Làm Course/Class/Teacher screens
12. Làm Student management screens
13. Làm Enrollment approval và Bulk enroll
14. Làm Attendance và Result screens
15. Làm Invoice/Payment/Debt screens
16. Làm Student dashboard và các trang cá nhân
17. Tích hợp API thật qua Gateway
18. Test full flow
```

---

## 23. Tiêu chí hoàn thành frontend

Frontend được xem là hoàn thành khi:

```txt
Đăng nhập được theo 3 role
Bắt đổi mật khẩu lần đầu nếu mustChangePassword = true
Mỗi role thấy đúng menu
Admin quản lý được khóa học, giáo viên, lớp, học viên
Admin duyệt được đăng ký
Admin gán nhiều học viên vào lớp
Teacher điểm danh được
Teacher nhập điểm được
Student đăng ký lớp được
Student xem được chuyên cần, kết quả, học phí
Admin ghi nhận thanh toán được
Import wizard hoạt động với preview lỗi
Có loading, empty, error state
Có confirm modal và toast notification
Tất cả API gọi qua Gateway
```
