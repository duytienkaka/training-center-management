# Payment & Report Service

## 1. Mục tiêu service

Payment & Report Service phụ trách toàn bộ nghiệp vụ liên quan đến:

- Auth/JWT
- Tài khoản người dùng
- Role Admin/Teacher/Student
- Đổi mật khẩu lần đầu
- Học phí
- Invoice
- Thanh toán
- Công nợ
- Báo cáo doanh thu
- Báo cáo lớp/học viên

Service này phục vụ đủ 3 role:

- Admin
- Teacher
- Student


## Quy ước database

Payment & Report Service sử dụng **một database riêng**:

```txt
PaymentReportDb
```

Không tách thành read database và write database.


Service này chỉ quản lý các bảng thuộc nghiệp vụ tài khoản, xác thực, học phí, invoice, thanh toán, công nợ và báo cáo. Service khác không được truy cập trực tiếp database này; nếu cần tạo tài khoản, tạo invoice hoặc kiểm tra công nợ thì gọi API của Payment & Report Service.


## 2. Bảng dữ liệu thuộc service này

### UserAccounts

| Field | Mô tả |
|---|---|
| Id | Mã tài khoản |
| Username | Tên đăng nhập, nên dùng email |
| Email | Email |
| Phone | Số điện thoại |
| PasswordHash | Mật khẩu đã hash |
| FullName | Họ tên |
| Role | Admin, Teacher, Student |
| RelatedEntityId | StudentId hoặc TeacherId |
| RelatedEntityType | Student, Teacher, Admin |
| MustChangePassword | Có bắt buộc đổi mật khẩu lần đầu không |
| Status | Active, Locked |
| CreatedAt | Ngày tạo |
| UpdatedAt | Ngày cập nhật |

### RefreshTokens

Dùng nếu làm refresh token.

### TuitionFees

| Field | Mô tả |
|---|---|
| Id | Mã cấu hình học phí |
| CourseId | Mã khóa học |
| ClassId | Mã lớp nếu học phí theo lớp |
| Amount | Số tiền |
| Currency | VND |
| EffectiveFrom | Ngày áp dụng |
| IsActive | Đang áp dụng hay không |

### Invoices

| Field | Mô tả |
|---|---|
| Id | Mã invoice |
| InvoiceCode | Mã hóa đơn |
| StudentId | Mã học viên |
| EnrollmentId | Mã đăng ký |
| CourseId | Mã khóa học |
| ClassId | Mã lớp |
| TotalAmount | Tổng tiền |
| DiscountAmount | Giảm giá |
| FinalAmount | Số tiền phải thu |
| PaidAmount | Đã thanh toán |
| DebtAmount | Còn nợ |
| DueDate | Hạn thanh toán |
| Status | Unpaid, Partial, Paid, Overdue, Cancelled |
| CreatedSource | Manual, EnrollmentApproval, BulkGenerate, Import |
| CreatedAt | Ngày tạo |

### PaymentTransactions

| Field | Mô tả |
|---|---|
| Id | Mã giao dịch |
| InvoiceId | Mã invoice |
| Amount | Số tiền thanh toán |
| PaymentMethod | Cash, BankTransfer, Online |
| PaidAt | Thời điểm thanh toán |
| CollectedBy | Người ghi nhận |
| Note | Ghi chú |

### InvoiceHistories, ImportBatches, ImportErrors

Dùng cho lịch sử hóa đơn và import học phí/thanh toán.

## 3. Chức năng theo role

### Admin

Admin có quyền:

- Đăng nhập
- Quản lý tài khoản
- Tạo tài khoản Admin/Teacher/Student thủ công
- Reset mật khẩu
- Khóa/mở tài khoản
- Cấu hình học phí
- Import học phí
- Tạo invoice thủ công
- Tạo invoice từ đăng ký đã duyệt
- Tạo invoice hàng loạt
- Ghi nhận thanh toán
- Import thanh toán nếu bật
- Xem công nợ
- Xem báo cáo doanh thu
- Xem invoice quá hạn

### Teacher

Teacher có quyền:

- Đăng nhập
- Đổi mật khẩu lần đầu
- Xem báo cáo lớp mình dạy
- Xem tổng hợp chuyên cần/kết quả lớp
- Không xem dữ liệu tài chính nhạy cảm nếu không được phân quyền

### Student

Student có quyền:

- Đăng nhập
- Đổi mật khẩu lần đầu
- Xem học phí cá nhân
- Xem công nợ cá nhân
- Xem lịch sử thanh toán
- Xem biên lai
- Xem invoice mới sau khi đăng ký lớp được duyệt

## 4. Nghiệp vụ quan trọng

### 4.1 Tạo tài khoản cho Student

Student profile do Student Service tạo. Payment Service chỉ tạo UserAccount.

Luồng:

```txt
Student Service tạo Student profile
-> Gọi Payment Service tạo UserAccount role Student
-> Payment Service trả userId, username, temporaryPassword
-> Student Service lưu userId vào Students
```

### 4.2 Tạo tài khoản cho Teacher

Teacher profile do Course Service tạo. Payment Service chỉ tạo UserAccount.

Luồng:

```txt
Course Service tạo Teacher profile
-> Gọi Payment Service tạo UserAccount role Teacher
-> Payment Service trả userId, username, temporaryPassword
-> Course Service lưu userId vào Teachers
```

### 4.3 Đăng nhập lần đầu và đổi mật khẩu

Khi account được tạo tự động, `MustChangePassword = true`.

Luồng:

```txt
User login bằng mật khẩu tạm
-> Payment Service trả mustChangePassword = true
-> Frontend chuyển sang màn đổi mật khẩu
-> User đổi mật khẩu
-> MustChangePassword = false
-> User vào dashboard theo role
```

### 4.4 Tạo invoice sau khi duyệt đăng ký

Payment Service không tự tạo Enrollment.

Luồng:

```txt
Student Service duyệt Enrollment
-> Gọi Payment Service tạo invoice từ Enrollment
-> Payment Service kiểm tra invoice đã tồn tại chưa
-> Lấy học phí theo CourseId/ClassId
-> Tạo invoice Unpaid
-> DebtAmount = FinalAmount
-> Trả invoiceId cho Student Service
```

### 4.5 Ghi nhận thanh toán

Luồng:

```txt
Admin chọn invoice
-> Nhập số tiền thanh toán
-> Tạo PaymentTransaction
-> Cập nhật PaidAmount, DebtAmount
-> Nếu DebtAmount = 0 thì Status = Paid
```

## 5. API của Payment & Report Service

### 5.1 Auth API

| Method | Endpoint | Mục đích |
|---|---|---|
| POST | `/api/payment-report/auth/login` | Đăng nhập |
| GET | `/api/payment-report/auth/me` | Thông tin user hiện tại |
| POST | `/api/payment-report/auth/change-password` | Đổi mật khẩu |
| POST | `/api/payment-report/auth/refresh-token` | Làm mới token nếu có |
| POST | `/api/payment-report/auth/logout` | Đăng xuất nếu có refresh token |

### 5.2 API cho Admin

| Method | Endpoint | Mục đích |
|---|---|---|
| GET | `/api/payment-report/admin/dashboard-summary` | Tổng quan doanh thu/công nợ/invoice quá hạn |
| GET | `/api/payment-report/admin/users` | Danh sách tài khoản |
| GET | `/api/payment-report/admin/users/{userId}` | Chi tiết tài khoản |
| POST | `/api/payment-report/admin/users` | Tạo tài khoản thủ công |
| PATCH | `/api/payment-report/admin/users/{userId}/status` | Khóa/mở tài khoản |
| POST | `/api/payment-report/admin/users/{userId}/reset-password` | Reset mật khẩu |
| GET | `/api/payment-report/admin/tuition-fees` | Danh sách học phí |
| POST | `/api/payment-report/admin/tuition-fees` | Tạo học phí |
| PUT | `/api/payment-report/admin/tuition-fees/{feeId}` | Cập nhật học phí |
| GET | `/api/payment-report/admin/tuition-fees/import/template` | Tải template import học phí |
| POST | `/api/payment-report/admin/tuition-fees/import/preview` | Preview import học phí |
| POST | `/api/payment-report/admin/tuition-fees/import/confirm` | Xác nhận import học phí |
| GET | `/api/payment-report/admin/invoices` | Danh sách invoice |
| GET | `/api/payment-report/admin/invoices/{invoiceId}` | Chi tiết invoice |
| POST | `/api/payment-report/admin/invoices` | Tạo invoice thủ công |
| POST | `/api/payment-report/admin/invoices/generate-by-class/{classId}` | Tạo invoice hàng loạt theo lớp |
| PATCH | `/api/payment-report/admin/invoices/{invoiceId}/cancel` | Hủy invoice |
| GET | `/api/payment-report/admin/invoices/overdue` | Invoice quá hạn |
| POST | `/api/payment-report/admin/invoices/{invoiceId}/payments` | Ghi nhận thanh toán |
| GET | `/api/payment-report/admin/payments/import/template` | Tải template import thanh toán |
| POST | `/api/payment-report/admin/payments/import/preview` | Preview import thanh toán |
| POST | `/api/payment-report/admin/payments/import/confirm` | Xác nhận import thanh toán |
| GET | `/api/payment-report/admin/students/{studentId}/debt` | Công nợ học viên |
| GET | `/api/payment-report/admin/debts` | Danh sách công nợ |
| GET | `/api/payment-report/admin/receipts/{paymentId}` | Xem biên lai |
| GET | `/api/payment-report/admin/reports/revenue` | Báo cáo doanh thu |
| GET | `/api/payment-report/admin/reports/debt` | Báo cáo công nợ |
| GET | `/api/payment-report/admin/reports/classes/{classId}` | Báo cáo lớp |

### 5.3 API cho Teacher

| Method | Endpoint | Mục đích |
|---|---|---|
| GET | `/api/payment-report/teacher/dashboard-summary` | Tổng quan giáo viên |
| GET | `/api/payment-report/teacher/reports/classes` | Báo cáo các lớp mình dạy |
| GET | `/api/payment-report/teacher/reports/classes/{classId}` | Báo cáo lớp |
| GET | `/api/payment-report/teacher/reports/classes/{classId}/attendance` | Báo cáo chuyên cần |
| GET | `/api/payment-report/teacher/reports/classes/{classId}/results` | Báo cáo kết quả |

### 5.4 API cho Student

| Method | Endpoint | Mục đích |
|---|---|---|
| GET | `/api/payment-report/student/dashboard-summary` | Tổng quan học phí/công nợ |
| GET | `/api/payment-report/student/invoices` | Học phí cá nhân |
| GET | `/api/payment-report/student/invoices/{invoiceId}` | Chi tiết invoice cá nhân |
| GET | `/api/payment-report/student/invoices/overdue` | Invoice quá hạn của mình |
| GET | `/api/payment-report/student/payments` | Lịch sử thanh toán |
| GET | `/api/payment-report/student/debt` | Công nợ cá nhân |
| GET | `/api/payment-report/student/receipts/{paymentId}` | Biên lai |

### 5.5 API nội bộ cho service khác

| Method | Endpoint | Consumer | Mục đích |
|---|---|---|---|
| POST | `/api/payment-report/internal/users/create-student-account` | Student | Tạo account cho học viên |
| POST | `/api/payment-report/internal/users/create-teacher-account` | Course | Tạo account cho giáo viên |
| GET | `/api/payment-report/internal/users/{userId}` | Gateway/Services | Lấy user/role |
| GET | `/api/payment-report/internal/students/{studentId}/debt-status` | Student | Kiểm tra nợ quá hạn |
| POST | `/api/payment-report/internal/invoices/generate-from-enrollment` | Student | Tạo invoice sau khi duyệt đăng ký |
| POST | `/api/payment-report/internal/invoices/generate-bulk-from-enrollments` | Student | Tạo invoice hàng loạt |
| GET | `/api/payment-report/internal/invoices/by-enrollment/{enrollmentId}` | Student | Kiểm tra invoice theo enrollment |

## 6. API cần gọi từ service khác

Payment Service cần gọi Student Service khi báo cáo hoặc tạo invoice:

```http
GET /api/student-attendance/internal/students/{studentId}
GET /api/student-attendance/internal/enrollments/{enrollmentId}
GET /api/student-attendance/internal/classes/{classId}/students
GET /api/student-attendance/internal/classes/{classId}/results
```

Payment Service cần gọi Course Service khi cần thông tin khóa/lớp:

```http
GET /api/course-schedule/internal/courses/{courseId}
GET /api/course-schedule/internal/classes/{classId}
```

## 7. UX cần hỗ trợ

- Login
- Đổi mật khẩu lần đầu
- Quản lý tài khoản
- Reset mật khẩu
- Badge trạng thái tài khoản: Active, Locked, MustChangePassword
- Dashboard doanh thu/công nợ
- Cảnh báo invoice quá hạn
- Ghi nhận thanh toán bằng modal xác nhận
- Xem biên lai
- Import học phí/thanh toán có preview lỗi
- Student xem học phí ngay sau khi đăng ký được duyệt
