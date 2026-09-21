# MINI-PROJECT SHORT TECHNICAL REPORT
**Course:** Cross-Platform Mobile App Development (VKU)  
**Mini-Project Title:** Mini-Project 1: VKU Field Survey – Offline Data Collection (PWA & Capacitor)  
**Team / Student Name:** Nguyễn Thanh Sinh  
**Submission Date:** 21/09/2026

---

## 1. GENERAL INFORMATION & DELIVERABLE LINKS
* **Team Members:**
  1. Nguyễn Thanh Sinh – Student ID: 23IT.B188 – Role: Fullstack Mobile Developer (PWA & Offline Architecture) – Contribution: 100%
* **📱 Live Demo URL:** https://thanhsinh-910.github.io/khaosat-mini/
* **📦 GitHub Repository:** https://github.com/ThanhSinh-910/khaosat-mini
* **🎥 Video Demo (Optional):** [Cung cấp nếu có]

---

## 2. FEATURE IMPLEMENTATION CHECKLIST
| # | Required Feature | Status | Implementation Details & Acceptance Level |
|:---:|---|:---:|---|
| 1 | **PWA Standalone Installation** | ✅ Complete | Cấu hình đầy đủ `manifest.json` (`display: standalone`, `theme_color: #1565C0`, bộ icons 192x192 & 512x512 maskable). Có thể cài đặt trực tiếp lên màn hình chính điện thoại Android (Chrome) và iOS (Safari). |
| 2 | **Offline Form & Local Persistence** | ✅ Complete | Form khảo sát kiểm tra cơ sở vật chất (Cơ sở, Tòa nhà, Tầng, Phòng, Đánh giá thiết bị, ghi chú, đính kèm ảnh). Lưu trữ trực tiếp vào `IndexedDB` (`VKUFacilityDB > surveys`) ngay khi tạo/sửa để không mất dữ liệu. |
| 3 | **Automatic Background Sync & Queue** | ✅ Complete | Lắng nghe trạng thái mạng (`network.js` với sự kiện `online`/`offline`), hiển thị banner thông báo. Dữ liệu khi offline được đánh dấu `synced: false`, tự động đồng bộ lên máy chủ và Google Sheets khi có kết nối trở lại. |
| 4 | **Service Worker & Cache-First** | ✅ Complete | Service Worker (`service-worker.js`) với chiến lược **Cache First** cho toàn bộ App Shell (HTML, CSS, JS, Icons), hỗ trợ mở và tương tác hoàn toàn offline. |
| 5 | **Capacitor Android Packaging** | ✅ Ready | Cấu trúc source code chuẩn tĩnh (SPA), sẵn sàng đóng gói thành file APK Android thông qua Capacitor Bridge. |

---

## 3. TECHNICAL ARCHITECTURE & PROJECT STRUCTURE
* **Kiến trúc ứng dụng (Client-First / Offline-First SPA):**
  - **Tầng Giao diện (Presentation Layer):** Single Page Application (SPA) xây dựng trên HTML5/CSS3/Vanilla JS theo hướng Mobile-First, responsive mượt mà trên mọi kích thước màn hình.
  - **Tầng Dịch vụ Mạng (Network & Sync Layer):** `network.js` bắt sự kiện kết nối; `sync.js` quản lý hàng đợi gửi dữ liệu (queue), hỗ trợ đẩy về Backend API và Google Sheets Script.
  - **Tầng Lưu trữ Cục bộ (Local Data Storage):** `db.js` bọc `IndexedDB` cung cấp các hàm async CRUD cho danh mục phiếu khảo sát.
  - **Tầng Offline & Cache (Service Worker):** `service-worker.js` đăng ký tiền bộ nhớ đệm (pre-cache) cho tất cả tài nguyên cốt lõi, fallback về `index.html` khi mất mạng.

* **Cấu trúc thư mục:**
```text
khaosat-mini/
├── index.html            # SPA App Shell giao diện chính
├── manifest.json         # Cấu hình PWA & thông tin cài đặt
├── service-worker.js     # Cache-First Service Worker offline
├── css/
│   ├── style.css         # Design system, theme VKU
│   └── responsive.css    # Responsive mobile viewports
├── js/
│   ├── app.js            # Router, điều hướng và Dashboard
│   ├── db.js             # IndexedDB wrapper (VKUFacilityDB)
│   ├── network.js        # Phát hiện Online/Offline
│   ├── sync.js           # Đồng bộ nền & gửi Google Sheets / API
│   ├── survey.js         # Xử lý form khảo sát cơ sở vật chất
│   ├── history.js        # Danh sách lịch sử khảo sát & lọc
│   └── statistics.js     # Thống kê trực quan tình trạng phòng
├── assets/icons/         # Biểu tượng PWA (192px, 512px)
├── server.py             # Server Python kiểm thử cục bộ
└── README.md             # Hướng dẫn chi tiết chạy và kiểm thử
```

---

## 4. EMPIRICAL EVIDENCE & SCREENSHOTS
1. **Màn hình Dashboard & Form khảo sát trên Mobile:**
   - Giao diện khảo sát trực quan, hỗ trợ chọn phòng, đánh giá tình trạng thiết bị (máy chiếu, điều hòa, bàn ghế, điện,...).
2. **Cài đặt PWA (Standalone Mode):**
   - Ứng dụng hiển thị nút/banner nhắc cài đặt, sau khi cài xuất hiện trên App Drawer như app native, mở lên không có thanh URL của trình duyệt.
3. **Thử nghiệm chế độ Offline & Đồng bộ khi có mạng:**
   - Bật chế độ Airplane / Offline trong DevTools: Ứng dụng vẫn tạo phiếu mới bình thường, lưu vào IndexedDB.
   - Khi bật lại mạng: Banner báo Online xuất hiện và các phiếu `synced: false` tự động được đồng bộ.

---

## 5. TECHNICAL CHALLENGES & RESOLUTIONS
* **Thách thức 1: Đảm bảo dữ liệu khảo sát không bị mất khi đang nhập liệu trong vùng mất sóng.**  
  *Giải pháp:* Tích hợp cơ chế tự động lưu nháp và lưu phiếu vào `IndexedDB` ngay khi bấm submit, đồng thời gán cờ `synced: false`. Dữ liệu luôn an toàn trên thiết bị dù reload trang hay tắt ứng dụng.
* **Thách thức 2: Đồng bộ ngầm khi người dùng di chuyển giữa vùng có và không có sóng.**  
  *Giải pháp:* Sử dụng `window.addEventListener('online', ...)` kết hợp vòng lặp kiểm tra hàng đợi (queue) trong `sync.js`, tự động gửi lần lượt các bản ghi chưa đồng bộ lên server khi phát hiện có mạng trở lại.
