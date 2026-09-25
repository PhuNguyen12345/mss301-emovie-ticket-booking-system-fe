# Kiến trúc Tổng thể Hệ thống Đặt vé xem phim (E-Cinema System)

Tài liệu này mô tả chi tiết kiến trúc hệ thống và cấu trúc mã nguồn của toàn bộ dự án, bao gồm cả khối Backend (Microservices) và Frontend (React SPA). Hệ thống được thiết kế để đảm bảo tính mở rộng, khả năng bảo trì cao và sự độc lập giữa các module nghiệp vụ.

---

## 1. Kiến trúc Hệ thống (System Architecture)

Dự án áp dụng mô hình **Microservices Architecture** với luồng giao tiếp đồng bộ (Synchronous) thông qua HTTP/OpenFeign làm chủ đạo, kết hợp xử lý bất đồng bộ (Asynchronous) cho các tác vụ nền.

* **Client Layer:** Giao diện người dùng React (Web Client) giao tiếp với hệ thống thông qua một điểm chạm duy nhất.
* **Edge Layer (API Gateway):** Chịu trách nhiệm định tuyến (Routing) và làm trạm kiểm soát bảo mật (Xác thực JWT token) trước khi cho phép request đi vào hệ thống nội bộ.
* **Platform Layer:**
* `Service Discovery` (Netflix Eureka): Quản lý danh sách IP và trạng thái của các microservices.


* **Application Layer (Microservices):** Các dịch vụ nghiệp vụ độc lập, mỗi dịch vụ sở hữu một Database riêng biệt nhằm đảm bảo tính toàn vẹn (Bounded Context):
* `movie-service`: Quản lý thông tin phim (tích hợp Cloudinary).
* `cinema-service`: Quản lý rạp, phòng chiếu, và sơ đồ vật lý của ghế.
* `showtime-service`: Quản lý suất chiếu, xử lý logic chống trùng lịch chiếu.
* `user-service`: Quản lý định danh người dùng và lịch sử thành viên.
* `booking-service`: Dịch vụ trung tâm xử lý nghiệp vụ đặt vé, sử dụng **Redis** để thiết lập khóa phân tán (Distributed Lock) giải quyết tranh chấp (Concurrency), và tích hợp API thanh toán (SePay/PayOS).



---

## 2. Cấu trúc mã nguồn Backend (Spring Boot Microservices)

Toàn bộ Backend được quản lý theo mô hình **Multi-module Maven**. Ở vòng ngoài tuân thủ kiến trúc phân tán, ở vòng trong mỗi Microservice tuân thủ kiến trúc **Layered (MVC: Controller - Service - Repository)**.

### 2.1. Cấu trúc thư mục gốc (Root Structure)

```text
e-movie-ticket-booking-system/
├── .github/                # Cấu hình CI/CD workflows
├── api-gateway/            # Cổng định tuyến API & Filter bảo mật
├── discovery-server/       # Máy chủ đăng ký dịch vụ (Eureka)
├── common/                 # Module dùng chung (Thư viện nội bộ)
│   └── src/main/java/com/ecinema/common/
│       ├── dto/            # ApiResponse.java (Chuẩn hóa HTTP response)
│       ├── exception/      # GlobalExceptionHandler.java, ErrorCode.java
│       └── utils/          # Các tiện ích dùng chung
├── services/               # Khối chứa các Microservices nghiệp vụ
│   ├── booking-service/    
│   ├── cinema-service/     
│   ├── movie-service/      
│   ├── showtime-service/   
│   └── user-service/       
├── docs/                   # Tài liệu thiết kế hệ thống, sơ đồ kiến trúc
├── pom.xml                 # Maven Parent POM quản lý dependencies
└── README.md

```

### 2.2. Cấu trúc chi tiết một Microservice (Ví dụ: `booking-service`)

```text
services/booking-service/
└── src/main/java/com/ecinema/booking/
    ├── BookingServiceApplication.java
    ├── config/             # Cấu hình hạ tầng (RedisConfig, FeignConfig, AsyncConfig)
    ├── controller/         # REST API Endpoints (Giao tiếp với Frontend qua Gateway)
    ├── dto/                # Data Transfer Objects (Request/Response Payloads)
    ├── entity/             # Các lớp JPA ánh xạ tới bảng Database
    ├── repository/         # Spring Data JPA Interfaces
    ├── client/             # Các OpenFeign Interfaces gọi sang các service khác
    ├── service/            # Tầng logic nghiệp vụ cốt lõi (Business logic)
    ├── notification/       # Tác vụ nền bất đồng bộ (@Async gửi email/thông báo)
    └── exception/          # Các ngoại lệ đặc thù (VD: SeatAlreadyBookedException)

```

---

## 3. Cấu trúc mã nguồn Frontend (React + TypeScript)

Frontend áp dụng kiến trúc **Feature-based**, phân chia thư mục ánh xạ 1-1 với cấu trúc Microservices của Backend để tối ưu hóa việc quản lý mã nguồn và bảo trì.

### Cấu trúc dự án

```text
mss301-emovie-ticket-booking-client/
├── public/                 # Tài nguyên tĩnh (favicon, index.html)
├── docs/                   # Tài liệu nội bộ của Frontend
├── src/
│   ├── assets/             # Hình ảnh, global CSS/SCSS
│   ├── components/         # UI Components dùng chung (Button, Modal, Loader)
│   ├── config/             # Biến môi trường, hằng số cấu hình
│   ├── features/           # Nhóm UI và logic theo Domain (Ánh xạ Backend)
│   │   ├── auth/           # Giao diện Đăng nhập / Đăng ký
│   │   ├── booking/        # Giao diện chọn ghế, thanh toán
│   │   ├── cinema/         # Giao diện tra cứu rạp
│   │   ├── movies/         # Danh sách phim, chi tiết phim
│   │   ├── showtime/       # Lịch chiếu
│   │   └── user/           # Hồ sơ cá nhân
│   ├── hooks/              # Custom React Hooks
│   ├── layouts/            # Khung giao diện chính (Header, Footer, Sidebar)
│   ├── pages/              # Lắp ghép component thành trang hoàn chỉnh (Routing targets)
│   ├── routes/             # Cấu hình React Router
│   ├── services/           # Axios API Clients (movieApi.ts, bookingApi.ts...)
│   ├── store/              # Global State (Zustand) quản lý luồng dữ liệu liên trang
│   ├── types/              # Khai báo TypeScript Interfaces (api.types.ts, booking.types.ts)
│   ├── utils/              # Các hàm tiện ích (format Date, currency)
│   ├── App.tsx             # Root component bọc Provider
│   └── main.tsx            # Entry point
├── .env                    # Lưu trữ biến môi trường (API Gateway URL)
├── vite.config.ts          # Cấu hình Vite bundler
├── tsconfig.json           # Cấu hình TypeScript
└── package.json

```

---

## 4. Quy chuẩn Giao tiếp & Dữ liệu (Communication & Data Conventions)

1. **Response Format:** Toàn bộ API Endpoints từ Backend bắt buộc trả về một định dạng JSON duy nhất thông qua class `ApiResponse<T>` từ module `common`. Frontend sử dụng file `src/types/api.types.ts` để ép kiểu dữ liệu tương ứng.
2. **Inter-service Call:** Các Microservices tuyệt đối không chọc thẳng vào Database của nhau. Mọi nhu cầu lấy dữ liệu chéo phải được thực hiện thông qua **OpenFeign Client** (gọi HTTP nội bộ). Chiều gọi dữ liệu quy định: Consumer $\rightarrow$ Provider.
3. **Authentication Flow:** Token JWT do `user-service` cấp phát. API Gateway thực hiện xác thực chữ ký của Token. Axios Interceptor ở Frontend (`src/services/apiClient.ts`) có trách nhiệm tự động đính kèm Header `Authorization: Bearer <token>` vào mọi Request cần xác thực.