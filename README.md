# MSS301 - E-Cinema Web Client

Hệ thống giao diện người dùng (Frontend) phục vụ nền tảng đặt vé xem phim trực tuyến. Ứng dụng được xây dựng theo kiến trúc **Feature-based**, thiết kế ánh xạ 1-1 với hệ thống Backend Microservices nhằm tối ưu hóa khả năng mở rộng, dễ dàng bảo trì và phân chia công việc trong nhóm.

## 🚀 Công nghệ sử dụng

* **Core Framework:** React (khởi tạo qua Vite để tối ưu tốc độ build) & TypeScript.
* **Quản lý State (Global State):** Zustand (Tối ưu cho luồng đặt vé qua nhiều bước) và React Context.
* **Giao tiếp mạng:** Axios (tích hợp cấu hình Interceptor để tự động xử lý JWT Token).
* **Định tuyến:** React Router DOM.
* **Môi trường:** Node.js.

## 📂 Kiến trúc thư mục (Feature-based Architecture)

Dự án phân tách rõ ràng các thành phần dùng chung (components, hooks, utils) và các luồng nghiệp vụ độc lập (features).

```text
mss301-emovie-ticket-booking-client/
├── public/                 # Các tài nguyên tĩnh (favicon, manifest...)
├── src/
│   ├── assets/             # Hình ảnh, icon, font chữ và global styles
│   ├── components/         # UI Components dùng chung (Button, Modal, Input...)
│   ├── config/             # Cấu hình môi trường, hằng số hệ thống
│   ├── features/           # Chứa UI và logic nhóm theo từng Domain/Microservice
│   │   ├── auth/           # Luồng Đăng nhập, Đăng ký, Quên mật khẩu
│   │   ├── booking/        # Luồng chọn ghế, thanh toán, giữ chỗ
│   │   ├── cinema/         # Danh sách hệ thống rạp
│   │   ├── movies/         # Danh sách phim, chi tiết phim
│   │   ├── showtime/       # Tra cứu lịch chiếu theo ngày/rạp
│   │   └── user/           # Quản lý hồ sơ cá nhân, lịch sử đặt vé
│   ├── hooks/              # Custom React Hooks (useDebounce, useClickOutside...)
│   ├── layouts/            # Cấu trúc khung trang (MainLayout, AuthLayout)
│   ├── pages/              # Lắp ráp các features thành trang hoàn chỉnh
│   ├── routes/             # Cấu hình đường dẫn URL
│   ├── services/           # Định nghĩa hàm gọi API qua Axios
│   ├── store/              # Quản lý Global State cho từng domain
│   ├── types/              # Khai báo TypeScript Interfaces/Types cốt lõi
│   └── utils/              # Các hàm hỗ trợ xử lý dữ liệu (format Date, Currency)
├── .env                    # Lưu trữ biến môi trường (URL API Gateway)
├── vite.config.ts          # Cấu hình trình biên dịch Vite
└── package.json            # Khai báo thư viện và script chạy dự án

```

## ⚙️ Quy chuẩn phát triển (Coding Convention)

1. **Ánh xạ Microservices:** Các file định nghĩa kiểu dữ liệu (`src/types/`) và cấu hình API (`src/services/`) bắt buộc phải bám sát cấu trúc của Backend (Ví dụ: `bookingApi.ts` giao tiếp với `booking-service`).


2. **Kiểm soát kiểu dữ liệu:** Mọi dữ liệu trả về từ Backend phải được ép kiểu thông qua interface `ApiResponse<T>` để đảm bảo IDE luôn gợi ý chính xác thuộc tính.
3. **Quản lý State cục bộ vs. Toàn cục:** Giới hạn việc sử dụng Global Store (`src/store/`). Chỉ đưa các dữ liệu cần chia sẻ giữa nhiều trang (như thông tin User đăng nhập, Giỏ hàng vé đang giữ chỗ) vào store. Các state của form nhập liệu nên được quản lý cục bộ tại component.

## 🛠 Hướng dẫn khởi chạy

1. Cài đặt các thư viện phụ thuộc:
```bash
npm install

```


2. Cấu hình biến môi trường:
* Tạo file `.env` từ file mẫu `.env.example` (nếu có).
* Đảm bảo `VITE_API_GATEWAY_URL` trỏ đúng về cổng của API Gateway dưới Backend.


3. Khởi chạy server ở chế độ phát triển:
```bash
npm run dev

```



---