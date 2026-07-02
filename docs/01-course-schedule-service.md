# Course & Schedule Service

## 1. Mục tiêu service

Course & Schedule Service phụ trách toàn bộ nghiệp vụ liên quan đến:

- Khóa học
- Lớp học
- Giáo viên
- Phân công giáo viên
- Lịch học
- Buổi học
- Sĩ số lớp
- Thời gian mở/đóng đăng ký lớp
- Thông tin lớp để Student Service đăng ký học viên

Service này phục vụ đủ 3 role:

- Admin
- Teacher
- Student


## Quy ước database

Course & Schedule Service sử dụng **một database riêng**:

```txt
CourseScheduleDb
```

Không tách thành read database và write database.


Service này chỉ quản lý các bảng thuộc nghiệp vụ khóa học, lớp học, giáo viên và lịch học. Service khác không được truy cập trực tiếp database này; nếu cần dữ liệu lớp/khóa học/lịch học thì gọi API của Course & Schedule Service.


## 2. Bảng dữ liệu thuộc service này

### Courses

| Field | Mô tả |
|---|---|
| Id | Mã khóa học |
| Code | Mã khóa học |
| Name | Tên khóa học |
| Description | Mô tả |
| Level | Cấp độ |
| DurationWeeks | Số tuần học |
| TotalSessions | Tổng số buổi |
| DefaultFee | Học phí mặc định |
| IsActive | Còn hoạt động hay không |
| CreatedAt | Ngày tạo |
| UpdatedAt | Ngày cập nhật |

### Teachers

| Field | Mô tả |
|---|---|
| Id | Mã giáo viên |
| UserId | Mã tài khoản đăng nhập trong UserAccounts |
| TeacherCode | Mã giáo viên |
| FullName | Họ tên |
| Email | Email |
| Phone | Số điện thoại |
| Specialty | Chuyên môn |
| Status | Active, Inactive |
| CreatedAt | Ngày tạo |
| UpdatedAt | Ngày cập nhật |

### Classes

| Field | Mô tả |
|---|---|
| Id | Mã lớp |
| CourseId | Mã khóa học |
| ClassCode | Mã lớp |
| Name | Tên lớp |
| TeacherId | Giáo viên phụ trách |
| StartDate | Ngày bắt đầu |
| EndDate | Ngày kết thúc |
| RegistrationStartDate | Ngày mở đăng ký |
| RegistrationEndDate | Ngày đóng đăng ký |
| MaxStudents | Sĩ số tối đa |
| CurrentStudents | Số học viên đã được duyệt |
| LearningMode | Offline, Online, Hybrid |
| Location | Phòng học/link online |
| Status | Draft, Open, InProgress, Completed, Cancelled |

### ClassSessions

| Field | Mô tả |
|---|---|
| Id | Mã buổi học |
| ClassId | Mã lớp |
| SessionNo | Số thứ tự buổi học |
| StudyDate | Ngày học |
| StartTime | Giờ bắt đầu |
| EndTime | Giờ kết thúc |
| Room | Phòng học |
| Topic | Chủ đề buổi học |
| Status | Scheduled, Done, Cancelled |

### ImportBatches và ImportErrors

Dùng cho chức năng import khóa học, giáo viên, lịch học.

## 3. Chức năng theo role

### Admin

Admin có quyền:

- Quản lý khóa học
- Import khóa học
- Quản lý giáo viên
- Import giáo viên
- Tạo tài khoản giáo viên khi tạo hồ sơ giáo viên
- Quản lý lớp học
- Phân công giáo viên cho lớp
- Quản lý lịch học
- Import lịch học
- Cập nhật sĩ số tối đa
- Cập nhật thời gian mở/đóng đăng ký
- Theo dõi lớp sắp bắt đầu, lớp gần đầy

### Teacher

Teacher có quyền:

- Xem lớp mình phụ trách
- Xem lịch dạy cá nhân
- Xem danh sách buổi học
- Cập nhật chủ đề buổi học nếu được cho phép
- Gửi yêu cầu đổi lịch/hủy buổi học nếu làm chức năng nâng cao

### Student

Student có quyền:

- Xem khóa học đang mở
- Xem lớp đang mở đăng ký
- Xem chi tiết lớp
- Xem lịch học của lớp
- Xem thông tin đăng ký: còn bao nhiêu chỗ, hạn đăng ký, học phí dự kiến, lịch học
- Xem lịch học cá nhân sau khi đã đăng ký lớp

## 4. Nghiệp vụ quan trọng

### 4.1 Tạo giáo viên và tài khoản đăng nhập

Khi Admin tạo giáo viên, hệ thống có thể tự động tạo tài khoản đăng nhập cho giáo viên.

Luồng:

```txt
Admin tạo giáo viên
-> Course Service tạo Teacher profile
-> Course Service gọi Payment/Auth API để tạo UserAccount role Teacher
-> Lưu UserId vào bảng Teachers
-> Trả username và mật khẩu tạm cho Admin
```

Nếu tạo Teacher thành công nhưng tạo UserAccount lỗi, hệ thống vẫn giữ Teacher profile và cho Admin bấm tạo lại tài khoản.

### 4.2 Kiểm tra điều kiện đăng ký lớp

Khi Student Service cần đăng ký học viên vào lớp, Course Service cung cấp API kiểm tra:

- Lớp có tồn tại không
- Lớp có đang mở đăng ký không
- Lớp còn chỗ không
- Lớp đã bị hủy/chưa hoàn thành không
- Hạn đăng ký còn hiệu lực không

### 4.3 Cập nhật sĩ số lớp

Không tăng sĩ số khi Student mới gửi yêu cầu đăng ký.

Chỉ tăng sĩ số khi Admin đã duyệt đăng ký thành công.

```txt
Student đăng ký lớp -> PendingApproval -> chưa tăng sĩ số
Admin duyệt -> tăng sĩ số
Admin hủy đăng ký đã duyệt -> giảm sĩ số
```

## 5. API của Course & Schedule Service

### 5.1 API cho Admin

| Method | Endpoint | Mục đích |
|---|---|---|
| GET | `/api/course-schedule/admin/dashboard-summary` | Tổng quan khóa học/lớp/lớp gần đầy |
| GET | `/api/course-schedule/admin/courses` | Danh sách khóa học |
| GET | `/api/course-schedule/admin/courses/{courseId}` | Chi tiết khóa học |
| POST | `/api/course-schedule/admin/courses` | Tạo khóa học |
| PUT | `/api/course-schedule/admin/courses/{courseId}` | Cập nhật khóa học |
| PATCH | `/api/course-schedule/admin/courses/{courseId}/status` | Bật/tắt khóa học |
| GET | `/api/course-schedule/admin/courses/import/template` | Tải template import khóa học |
| POST | `/api/course-schedule/admin/courses/import/preview` | Preview import khóa học |
| POST | `/api/course-schedule/admin/courses/import/confirm` | Xác nhận import khóa học |
| GET | `/api/course-schedule/admin/teachers` | Danh sách giáo viên |
| GET | `/api/course-schedule/admin/teachers/{teacherId}` | Chi tiết giáo viên |
| POST | `/api/course-schedule/admin/teachers` | Tạo giáo viên, có thể auto tạo account |
| PUT | `/api/course-schedule/admin/teachers/{teacherId}` | Cập nhật giáo viên |
| PATCH | `/api/course-schedule/admin/teachers/{teacherId}/status` | Khóa/mở giáo viên |
| POST | `/api/course-schedule/admin/teachers/{teacherId}/create-account` | Tạo lại account cho giáo viên |
| GET | `/api/course-schedule/admin/teachers/import/template` | Tải template import giáo viên |
| POST | `/api/course-schedule/admin/teachers/import/preview` | Preview import giáo viên |
| POST | `/api/course-schedule/admin/teachers/import/confirm` | Xác nhận import giáo viên |
| GET | `/api/course-schedule/admin/classes` | Danh sách lớp |
| GET | `/api/course-schedule/admin/classes/{classId}` | Chi tiết lớp |
| POST | `/api/course-schedule/admin/classes` | Tạo lớp |
| PUT | `/api/course-schedule/admin/classes/{classId}` | Cập nhật lớp |
| PATCH | `/api/course-schedule/admin/classes/{classId}/teacher` | Phân công giáo viên |
| PATCH | `/api/course-schedule/admin/classes/{classId}/status` | Đổi trạng thái lớp |
| PATCH | `/api/course-schedule/admin/classes/{classId}/capacity` | Cập nhật sĩ số tối đa |
| PATCH | `/api/course-schedule/admin/classes/{classId}/registration-window` | Cập nhật thời gian đăng ký |
| GET | `/api/course-schedule/admin/classes/{classId}/sessions` | Lịch học của lớp |
| POST | `/api/course-schedule/admin/classes/{classId}/sessions` | Tạo một buổi học |
| POST | `/api/course-schedule/admin/classes/{classId}/sessions/bulk` | Tạo lịch học hàng loạt |
| PUT | `/api/course-schedule/admin/classes/{classId}/sessions/{sessionId}` | Cập nhật buổi học |
| DELETE | `/api/course-schedule/admin/classes/{classId}/sessions/{sessionId}` | Hủy buổi học |
| GET | `/api/course-schedule/admin/classes/{classId}/sessions/import/template` | Tải template import lịch học |
| POST | `/api/course-schedule/admin/classes/{classId}/sessions/import/preview` | Preview import lịch học |
| POST | `/api/course-schedule/admin/classes/{classId}/sessions/import/confirm` | Xác nhận import lịch học |
| GET | `/api/course-schedule/admin/calendar` | Lịch toàn trung tâm |

### 5.2 API cho Teacher

| Method | Endpoint | Mục đích |
|---|---|---|
| GET | `/api/course-schedule/teacher/classes` | Lớp mình dạy |
| GET | `/api/course-schedule/teacher/classes/{classId}` | Chi tiết lớp mình dạy |
| GET | `/api/course-schedule/teacher/classes/{classId}/sessions` | Buổi học của lớp |
| GET | `/api/course-schedule/teacher/schedule` | Lịch dạy cá nhân |
| GET | `/api/course-schedule/teacher/calendar` | Lịch dạy dạng calendar |
| PATCH | `/api/course-schedule/teacher/classes/{classId}/sessions/{sessionId}/topic` | Cập nhật chủ đề buổi học |
| POST | `/api/course-schedule/teacher/classes/{classId}/sessions/{sessionId}/change-request` | Yêu cầu đổi lịch/hủy buổi học |

### 5.3 API cho Student

| Method | Endpoint | Mục đích |
|---|---|---|
| GET | `/api/course-schedule/student/courses` | Khóa học đang mở |
| GET | `/api/course-schedule/student/courses/{courseId}` | Chi tiết khóa học |
| GET | `/api/course-schedule/student/classes/open` | Lớp đang mở đăng ký |
| GET | `/api/course-schedule/student/classes/{classId}` | Chi tiết lớp |
| GET | `/api/course-schedule/student/classes/{classId}/registration-info` | Thông tin đăng ký lớp |
| GET | `/api/course-schedule/student/classes/{classId}/sessions` | Lịch học của lớp |
| GET | `/api/course-schedule/student/schedule` | Lịch học cá nhân |
| GET | `/api/course-schedule/student/calendar` | Lịch học cá nhân dạng calendar |

### 5.4 API nội bộ cho service khác

| Method | Endpoint | Consumer | Mục đích |
|---|---|---|---|
| GET | `/api/course-schedule/internal/courses/{courseId}` | Student, Payment | Lấy thông tin khóa học |
| GET | `/api/course-schedule/internal/classes/{classId}` | Student, Payment | Lấy thông tin lớp |
| GET | `/api/course-schedule/internal/classes/{classId}/registration-check` | Student | Kiểm tra điều kiện đăng ký |
| GET | `/api/course-schedule/internal/classes/{classId}/capacity` | Student | Kiểm tra sĩ số |
| PATCH | `/api/course-schedule/internal/classes/{classId}/increase-student-count` | Student | Tăng sĩ số sau khi duyệt đăng ký |
| PATCH | `/api/course-schedule/internal/classes/{classId}/decrease-student-count` | Student | Giảm sĩ số khi hủy đăng ký đã duyệt |
| GET | `/api/course-schedule/internal/classes/{classId}/sessions` | Student | Lấy buổi học để điểm danh |
| GET | `/api/course-schedule/internal/teachers/{teacherId}` | Student, Payment | Lấy thông tin giáo viên |

## 6. API cần gọi từ service khác

Course Service cần gọi Payment/Auth API khi tạo tài khoản giáo viên:

```http
POST /api/payment-report/internal/users/create-teacher-account
```

Request:

```json
{
  "teacherId": 20,
  "fullName": "Trần Thị B",
  "email": "teacherb@example.com",
  "phone": "0911111111"
}
```

## 7. UX cần hỗ trợ

- Dashboard Admin: tổng khóa học, lớp đang mở, lớp gần đầy, lớp sắp bắt đầu
- Calendar view cho Admin/Teacher/Student
- Import wizard có template, preview lỗi, confirm import
- Status badge cho lớp, khóa học, buổi học
- Cảnh báo lớp gần đầy
- Modal xác nhận khi hủy buổi học, hủy lớp, đổi trạng thái lớp
- Form tạo giáo viên có checkbox tự tạo tài khoản
