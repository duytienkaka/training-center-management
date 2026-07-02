# Student & Attendance Service

## 1. Mục tiêu service

Student & Attendance Service phụ trách toàn bộ nghiệp vụ liên quan đến:

- Hồ sơ học viên
- Tạo tài khoản đăng nhập cho học viên
- Đăng ký lớp học
- Duyệt/từ chối đăng ký
- Gán nhiều học viên vào lớp
- Điểm danh
- Tỷ lệ chuyên cần
- Điểm số
- Kết quả học tập

Service này phục vụ đủ 3 role:

- Admin
- Teacher
- Student


## Quy ước database

Student & Attendance Service sử dụng **một database riêng**:

```txt
StudentAttendanceDb
```

Không tách thành read database và write database.


Service này chỉ quản lý các bảng thuộc nghiệp vụ học viên, đăng ký lớp, điểm danh và kết quả học tập. Service khác không được truy cập trực tiếp database này; nếu cần dữ liệu học viên/enrollment/chuyên cần/kết quả thì gọi API của Student & Attendance Service.


## 2. Bảng dữ liệu thuộc service này

### Students

| Field | Mô tả |
|---|---|
| Id | Mã học viên |
| UserId | Mã tài khoản đăng nhập trong UserAccounts |
| StudentCode | Mã học viên |
| FullName | Họ tên |
| Email | Email |
| Phone | Số điện thoại |
| DateOfBirth | Ngày sinh |
| Address | Địa chỉ |
| Status | Active, Inactive |
| CreatedAt | Ngày tạo |
| UpdatedAt | Ngày cập nhật |

### Enrollments

| Field | Mô tả |
|---|---|
| Id | Mã đăng ký |
| StudentId | Mã học viên |
| CourseId | Mã khóa học |
| ClassId | Mã lớp |
| RequestedAt | Thời điểm đăng ký |
| ApprovedAt | Thời điểm duyệt |
| ApprovedByAdminId | Admin duyệt |
| RejectedAt | Thời điểm từ chối |
| RejectedReason | Lý do từ chối |
| Status | PendingApproval, Approved, Rejected, Studying, Completed, Dropped, Cancelled |
| AttendanceRate | Tỷ lệ chuyên cần |
| FinalStatus | Pass, Fail, NotYet |
| InvoiceCreated | Đã tạo invoice hay chưa |

### EnrollmentHistories

Lưu lịch sử thay đổi trạng thái đăng ký để hiển thị timeline cho Student.

### Attendances

| Field | Mô tả |
|---|---|
| Id | Mã điểm danh |
| EnrollmentId | Mã đăng ký |
| StudentId | Mã học viên |
| ClassId | Mã lớp |
| SessionId | Mã buổi học |
| Status | Present, Absent, Late, Excused |
| Note | Ghi chú |
| TakenByTeacherId | Giáo viên điểm danh |
| TakenAt | Thời gian điểm danh |

### AcademicResults

| Field | Mô tả |
|---|---|
| Id | Mã kết quả |
| EnrollmentId | Mã đăng ký |
| MidtermScore | Điểm giữa kỳ |
| FinalScore | Điểm cuối kỳ |
| AverageScore | Điểm trung bình |
| ResultStatus | Pass, Fail, NotYet |
| UpdatedBy | Người cập nhật |
| UpdatedAt | Ngày cập nhật |

### ImportBatches và ImportErrors

Dùng cho import học viên, đăng ký lớp, điểm danh, điểm số.

## 3. Chức năng theo role

### Admin

Admin có quyền:

- Quản lý học viên
- Import học viên
- Tạo tài khoản đăng nhập cho học viên
- Gán một học viên vào lớp
- Gán nhiều học viên vào lớp cùng lúc
- Import danh sách đăng ký lớp
- Xem đăng ký chờ duyệt
- Duyệt/từ chối đăng ký
- Duyệt/từ chối nhiều đăng ký cùng lúc
- Hủy đăng ký
- Xem điểm danh
- Xem kết quả học tập
- Chỉnh sửa điểm danh/kết quả nếu cần

### Teacher

Teacher có quyền:

- Xem học viên trong lớp mình dạy
- Điểm danh từng buổi
- Điểm danh hàng loạt
- Import điểm danh nếu được bật
- Nhập điểm
- Nhập điểm hàng loạt
- Import điểm số nếu được bật
- Xem tỷ lệ chuyên cần
- Xem cảnh báo học viên vắng nhiều

### Student

Student có quyền:

- Xem hồ sơ cá nhân
- Cập nhật thông tin cá nhân được phép
- Xem lớp đã đăng ký
- Gửi yêu cầu đăng ký lớp
- Hủy yêu cầu đăng ký khi còn PendingApproval
- Xem timeline trạng thái đăng ký
- Xem lịch sử điểm danh
- Xem tỷ lệ chuyên cần
- Xem điểm số và kết quả Pass/Fail

## 4. Nghiệp vụ quan trọng

### 4.1 Tạo học viên và tài khoản đăng nhập

Khi Admin tạo học viên, hệ thống có thể tự động tạo tài khoản đăng nhập cho học viên.

Luồng:

```txt
Admin tạo học viên
-> Student Service tạo Student profile
-> Student Service gọi Payment/Auth API tạo UserAccount role Student
-> Lưu UserId vào bảng Students
-> Trả username và mật khẩu tạm cho Admin
```

Nếu tạo Student thành công nhưng tạo UserAccount lỗi, hệ thống vẫn giữ Student profile và cho Admin bấm tạo lại tài khoản.

### 4.2 Student tự đăng ký lớp

Luồng:

```txt
Student chọn lớp đang mở
-> Student Service gọi Course Service kiểm tra điều kiện đăng ký
-> Kiểm tra học viên đã đăng ký lớp này chưa
-> Tạo Enrollment trạng thái PendingApproval
-> Chưa tăng sĩ số
-> Chưa tạo invoice
```

### 4.3 Admin duyệt đăng ký

Luồng:

```txt
Admin duyệt đăng ký
-> Student Service kiểm tra lại lớp còn chỗ qua Course Service
-> Chuyển Enrollment sang Approved hoặc Studying
-> Gọi Course Service tăng sĩ số
-> Gọi Payment Service tạo invoice/công nợ
-> Đánh dấu InvoiceCreated = true
```

### 4.4 Gán nhiều học viên vào lớp

Admin có thể chọn nhiều học viên và gán vào một lớp.

Luồng:

```txt
Admin chọn lớp
-> Chọn nhiều học viên
-> Student Service validate từng học viên
-> Kiểm tra lớp còn đủ chỗ
-> Tạo nhiều Enrollment
-> Tăng sĩ số theo số lượng thành công
-> Tạo invoice/công nợ cho từng học viên
```

### 4.5 Điểm danh và chuyên cần

Sau mỗi lần điểm danh, hệ thống tính lại tỷ lệ chuyên cần.

Rule gợi ý:

```txt
AttendanceRate = số buổi Present/Late hợp lệ / tổng số buổi đã học
```

Cảnh báo học viên vắng nhiều:

```txt
AttendanceRate < 80% hoặc vắng 3 buổi liên tiếp
```

### 4.6 Kết quả học tập

Rule gợi ý:

```txt
AverageScore = MidtermScore * 40% + FinalScore * 60%
Pass nếu AverageScore >= 5 và AttendanceRate >= 80%
```

## 5. API của Student & Attendance Service

### 5.1 API cho Admin

| Method | Endpoint | Mục đích |
|---|---|---|
| GET | `/api/student-attendance/admin/dashboard-summary` | Tổng quan học viên/đăng ký/chuyên cần |
| GET | `/api/student-attendance/admin/students` | Danh sách học viên |
| GET | `/api/student-attendance/admin/students/{studentId}` | Chi tiết học viên |
| POST | `/api/student-attendance/admin/students` | Tạo học viên, có thể auto tạo account |
| PUT | `/api/student-attendance/admin/students/{studentId}` | Cập nhật học viên |
| PATCH | `/api/student-attendance/admin/students/{studentId}/status` | Khóa/mở học viên |
| POST | `/api/student-attendance/admin/students/{studentId}/create-account` | Tạo lại account cho học viên |
| GET | `/api/student-attendance/admin/students/import/template` | Tải template import học viên |
| POST | `/api/student-attendance/admin/students/import/preview` | Preview import học viên |
| POST | `/api/student-attendance/admin/students/import/confirm` | Xác nhận import học viên |
| GET | `/api/student-attendance/admin/enrollments` | Danh sách đăng ký |
| POST | `/api/student-attendance/admin/enrollments` | Gán một học viên vào lớp |
| GET | `/api/student-attendance/admin/enrollments/pending` | Đăng ký chờ duyệt |
| POST | `/api/student-attendance/admin/enrollments/{enrollmentId}/approve` | Duyệt đăng ký |
| POST | `/api/student-attendance/admin/enrollments/{enrollmentId}/reject` | Từ chối đăng ký |
| POST | `/api/student-attendance/admin/enrollments/approve-bulk` | Duyệt nhiều đăng ký |
| POST | `/api/student-attendance/admin/enrollments/reject-bulk` | Từ chối nhiều đăng ký |
| PATCH | `/api/student-attendance/admin/enrollments/{enrollmentId}/cancel` | Hủy đăng ký |
| POST | `/api/student-attendance/admin/classes/{classId}/enrollments/bulk` | Gán nhiều học viên vào lớp |
| GET | `/api/student-attendance/admin/enrollments/import/template` | Tải template import đăng ký |
| POST | `/api/student-attendance/admin/enrollments/import/preview` | Preview import đăng ký |
| POST | `/api/student-attendance/admin/enrollments/import/confirm` | Xác nhận import đăng ký |
| GET | `/api/student-attendance/admin/classes/{classId}/students` | Học viên trong lớp |
| GET | `/api/student-attendance/admin/classes/{classId}/attendance` | Điểm danh theo lớp |
| GET | `/api/student-attendance/admin/classes/{classId}/attendance-risk` | Học viên vắng nhiều |
| PUT | `/api/student-attendance/admin/attendance/{attendanceId}` | Chỉnh điểm danh |
| GET | `/api/student-attendance/admin/classes/{classId}/results` | Kết quả theo lớp |
| PUT | `/api/student-attendance/admin/results/{resultId}` | Cập nhật kết quả |

### 5.2 API cho Teacher

| Method | Endpoint | Mục đích |
|---|---|---|
| GET | `/api/student-attendance/teacher/dashboard-summary` | Tổng quan lớp dạy/điểm danh/cảnh báo |
| GET | `/api/student-attendance/teacher/classes/{classId}/students` | Học viên lớp mình dạy |
| GET | `/api/student-attendance/teacher/classes/{classId}/sessions/{sessionId}/attendance` | Điểm danh một buổi |
| POST | `/api/student-attendance/teacher/classes/{classId}/sessions/{sessionId}/attendance` | Điểm danh hàng loạt |
| PUT | `/api/student-attendance/teacher/attendance/{attendanceId}` | Cập nhật điểm danh |
| GET | `/api/student-attendance/teacher/classes/{classId}/attendance-summary` | Tổng hợp chuyên cần |
| GET | `/api/student-attendance/teacher/classes/{classId}/attendance-risk` | Học viên vắng nhiều |
| GET | `/api/student-attendance/teacher/classes/{classId}/results` | Kết quả lớp |
| POST | `/api/student-attendance/teacher/classes/{classId}/results` | Nhập điểm hàng loạt |
| PUT | `/api/student-attendance/teacher/results/{resultId}` | Cập nhật điểm |
| GET | `/api/student-attendance/teacher/attendance/import/template` | Tải template import điểm danh |
| POST | `/api/student-attendance/teacher/attendance/import/preview` | Preview import điểm danh |
| POST | `/api/student-attendance/teacher/attendance/import/confirm` | Xác nhận import điểm danh |
| GET | `/api/student-attendance/teacher/results/import/template` | Tải template import điểm số |
| POST | `/api/student-attendance/teacher/results/import/preview` | Preview import điểm số |
| POST | `/api/student-attendance/teacher/results/import/confirm` | Xác nhận import điểm số |

### 5.3 API cho Student

| Method | Endpoint | Mục đích |
|---|---|---|
| GET | `/api/student-attendance/student/dashboard-summary` | Tổng quan học viên |
| GET | `/api/student-attendance/student/profile` | Hồ sơ cá nhân |
| PUT | `/api/student-attendance/student/profile` | Cập nhật thông tin cá nhân |
| POST | `/api/student-attendance/student/classes/{classId}/register` | Gửi yêu cầu đăng ký lớp |
| PATCH | `/api/student-attendance/student/enrollments/{enrollmentId}/cancel-request` | Hủy yêu cầu đăng ký |
| GET | `/api/student-attendance/student/enrollments` | Danh sách đăng ký/lớp của tôi |
| GET | `/api/student-attendance/student/enrollments/{enrollmentId}` | Chi tiết đăng ký |
| GET | `/api/student-attendance/student/enrollments/{enrollmentId}/timeline` | Timeline đăng ký |
| GET | `/api/student-attendance/student/attendance` | Lịch sử điểm danh |
| GET | `/api/student-attendance/student/enrollments/{enrollmentId}/attendance-rate` | Tỷ lệ chuyên cần |
| GET | `/api/student-attendance/student/results` | Kết quả học tập |
| GET | `/api/student-attendance/student/enrollments/{enrollmentId}/result` | Kết quả theo lớp |

### 5.4 API nội bộ cho service khác

| Method | Endpoint | Consumer | Mục đích |
|---|---|---|---|
| GET | `/api/student-attendance/internal/students/{studentId}` | Payment | Lấy thông tin học viên |
| GET | `/api/student-attendance/internal/enrollments/{enrollmentId}` | Payment | Lấy thông tin đăng ký |
| GET | `/api/student-attendance/internal/classes/{classId}/students` | Payment, Report | Học viên trong lớp |
| GET | `/api/student-attendance/internal/students/{studentId}/enrollments` | Payment | Enrollment của học viên |
| GET | `/api/student-attendance/internal/classes/{classId}/results` | Report | Kết quả theo lớp |
| GET | `/api/student-attendance/internal/classes/{classId}/attendance-summary` | Report | Chuyên cần theo lớp |
| PATCH | `/api/student-attendance/internal/enrollments/{enrollmentId}/invoice-created` | Payment | Đánh dấu đã tạo invoice |

## 6. API cần gọi từ service khác

Student Service cần gọi Course Service:

```http
GET /api/course-schedule/internal/classes/{classId}/registration-check
PATCH /api/course-schedule/internal/classes/{classId}/increase-student-count
PATCH /api/course-schedule/internal/classes/{classId}/decrease-student-count
GET /api/course-schedule/internal/classes/{classId}/sessions
```

Student Service cần gọi Payment/Auth Service:

```http
POST /api/payment-report/internal/users/create-student-account
POST /api/payment-report/internal/invoices/generate-from-enrollment
POST /api/payment-report/internal/invoices/generate-bulk-from-enrollments
GET /api/payment-report/internal/students/{studentId}/debt-status
```

## 7. UX cần hỗ trợ

- Import wizard cho học viên/đăng ký/điểm danh/điểm số
- Bulk enroll học viên vào lớp
- Bulk approve/reject đăng ký
- Enrollment timeline cho Student
- Teacher điểm danh nhanh: Mark all Present, selected Absent/Late
- Bảng nhập điểm giống spreadsheet
- Cảnh báo học viên vắng nhiều
- Form tạo học viên có checkbox tự tạo tài khoản
- Badge Account Status: Created, NotCreated, Locked, MustChangePassword
