# BÁO CÁO TIẾN ĐỘ XÂY DỰNG ỨNG DỤNG SMART ATTENDANCE

## 1. Tổng quan

**Đề tài:** Smart Attendance – Ứng dụng quản lý và chấm công bằng vân tay  
**Nền tảng:** Android  
**Công nghệ:** React Native + TypeScript  
**Backend dữ liệu:** Firebase Realtime Database  
**Xác thực:** Firebase Authentication  
**Phần cứng:** ESP32 + cảm biến vân tay

---

# PHẦN 1. NGHIÊN CỨU CÁC CHỨC NĂNG CẦN THIẾT

## 1.1. Mục tiêu

Giai đoạn đầu tập trung phân tích yêu cầu và xác định các chức năng cần thiết trước khi triển khai ứng dụng.

Kiến trúc được xác định gồm:

```text
React Native + TypeScript
          │
          ├── Firebase Authentication
          │
          └── Firebase Realtime Database
                         │
                         └── ESP32 + cảm biến vân tay
```

Firebase Realtime Database được sử dụng làm nguồn dữ liệu trung tâm.

## 1.2. Các chức năng đã nghiên cứu

### Đăng nhập

Ứng dụng sử dụng Firebase Authentication với Email/Password.

Sau khi đăng nhập:

```text
Firebase Authentication
        ↓
Auth UID
        ↓
/users/{uid}
        ↓
Kiểm tra role + active
        ↓
Admin / Employee
```

Hai role chính:

- `admin`
- `employee`

Password không được lưu trong Realtime Database.

### Quản lý nhân viên

Admin quản lý:

- Mã nhân viên
- Họ tên
- Số điện thoại
- Phòng ban
- Chức vụ
- Email
- Trạng thái hoạt động
- Fingerprint ID

Dữ liệu nhân viên nằm tại:

```text
/employees/{employeeId}
```

### Đăng ký vân tay

Quy trình:

```text
Admin
 ↓
Nhập thông tin nhân viên
 ↓
Tạo Firebase Authentication account
 ↓
Tạo /users/{uid}
 ↓
Tạo /employees/{employeeId}
 ↓
Tạo /enrollment_requests/{requestId}
 ↓
ESP32 xử lý
 ↓
Cảm biến vân tay
 ↓
fingerprintId thực tế
 ↓
SUCCESS / FAILED
```

Các trạng thái:

```text
PENDING
PROCESSING
SUCCESS
FAILED
CANCELLED
```

### Chấm công

Quy trình:

```text
Nhân viên đặt vân tay
        ↓
ESP32 nhận fingerprintId
        ↓
Xác định employeeId
        ↓
Lấy thời gian
        ↓
Ghi Firebase
        ↓
/attendance/{date}/{employeeId}
        ↓
Ứng dụng nhận dữ liệu realtime
```

### Dashboard và lịch sử

Nghiên cứu các chức năng:

- Tổng quan chấm công
- Danh sách nhân viên
- Trạng thái chấm công
- Lịch sử chấm công
- Quản lý ca
- Trạng thái đăng ký vân tay

---

# PHẦN 2. XÂY DỰNG GIAO DIỆN UI VÀ CÁC CHỨC NĂNG

## 2.1. Mục tiêu

Sau khi xác định chức năng, nhóm triển khai giao diện và các chức năng cơ bản đã nghiên cứu ở tuần trước.

Mục tiêu là tạo được luồng sử dụng hoàn chỉnh ở tầng giao diện trước khi kết nối toàn bộ dữ liệu thực tế.

## 2.2. Các màn hình đã triển khai

### Màn hình Login

Gồm:

- Email
- Password
- Nút đăng nhập
- Thông báo lỗi
- Điều hướng sau đăng nhập

### Dashboard

Hiển thị các thông tin tổng quan như:

- Tổng số nhân viên
- Tình hình chấm công
- Có mặt
- Đi muộn
- Thông tin trong ngày

### Quản lý nhân viên

Các chức năng:

- Xem danh sách
- Xem chi tiết
- Thêm nhân viên
- Chỉnh sửa
- Quản lý trạng thái
- Đăng ký vân tay

### Quản lý ca

Các thông tin:

- Tên ca
- Giờ bắt đầu
- Giờ kết thúc
- Grace period
- Mô tả
- Trạng thái hoạt động

### Đăng ký vân tay

Luồng giao diện:

```text
Chọn nhân viên
      ↓
Chọn thiết bị
      ↓
Gửi yêu cầu
      ↓
Chờ ESP32
      ↓
PROCESSING
      ↓
SUCCESS / FAILED
```

### Lịch sử chấm công

Hiển thị:

- Nhân viên
- Ngày
- Giờ vào
- Giờ ra
- Trạng thái

---

# PHẦN 3. KẾT NỐI FIREBASE VÀ ĐĂNG NHẬP ADMIN

## 3.1. Mục tiêu

Giai đoạn hiện tại chuyển từ giao diện sang sử dụng dữ liệu thực tế trên Firebase.

Hai thành phần chính:

1. Firebase Authentication.
2. Firebase Realtime Database.

## 3.2. Khởi tạo Firebase

Ứng dụng sử dụng Firebase Client SDK:

```ts
const app = initializeApp(firebaseConfig);

const auth = getAuth(app);

const db = getDatabase(app);
```

Không sử dụng:

- Firebase Admin SDK
- Firestore
- Backend riêng
- Service Account
- Private Key

## 3.3. Đăng nhập Admin

Luồng hiện tại:

```text
Admin nhập email + password
          ↓
signInWithEmailAndPassword()
          ↓
Firebase Authentication
          ↓
currentUser.uid
          ↓
/users/{uid}
          ↓
Kiểm tra role
          ↓
role = admin
          ↓
Dashboard
```

Ngoài `role`, tài khoản còn được kiểm tra trạng thái:

```text
active == true
```

## 3.4. Theo dõi session

Sử dụng:

```ts
onAuthStateChanged(auth, callback)
```

để theo dõi trạng thái đăng nhập và khôi phục session.

Đăng xuất:

```ts
signOut(auth)
```

## 3.5. Đọc dữ liệu Firebase

Luồng kiến trúc:

```text
Firebase RTDB
      ↓
Firebase Service
      ↓
Mapper / Validator
      ↓
TypeScript Model
      ↓
UI
```

Các node chính:

```text
/attendance
/attendance_test
/devices
/employees
/settings
/users
/shifts
/enrollment_requests
```

## 3.6. Ghi dữ liệu Firebase

Ví dụ Admin tạo yêu cầu đăng ký vân tay:

```text
/enrollment_requests/{requestId}
```

Request ban đầu:

```json
{
  "employeeId": "NV001",
  "deviceId": "ESP32_01",
  "fingerprintId": null,
  "status": "PENDING"
}
```

Sau đó ESP32 xử lý:

```text
PENDING
   ↓
PROCESSING
   ↓
SUCCESS / FAILED
```

Nếu thành công, ESP32 trả về `fingerprintId` thực tế.

---

