# MINI-PROJECT SHORT TECHNICAL REPORT
**Course:** Cross-Platform Mobile App Development (VKU)  
**Mini-Project Title:** Mini-Project 1: VKU Field Survey PWA & Hybrid Mobile with Capacitor  
**Team / Student Name:** Nguyễn Thanh Sinh (GitHub: ThanhSinh-910)  
**Submission Date:** 14/10/2026  

---

## 1. GENERAL INFORMATION & DELIVERABLE LINKS
* **Team Members:**
  1. Nguyễn Thanh Sinh — Student ID: 23IT.B188 — Role: Fullstack Mobile Developer (PWA, Capacitor Native Bridge & Offline Architecture) — Contribution: 100%
* **🔗 Live Demo PWA (Web):** [https://thanhsinh-910.github.io/khaosat-mini/](https://thanhsinh-910.github.io/khaosat-mini/)
* **📱 Android APK Download (Release):** [Download vku-facility-check.apk (Release v1.0.0)](https://github.com/ThanhSinh-910/khaosat-mini/releases/download/v1.0.0/vku-facility-check.apk) *(Cũng có sẵn tại thư mục gốc repository: `./vku-facility-check.apk`)*
* **💻 GitHub Repository:** [https://github.com/ThanhSinh-910/khaosat-mini](https://github.com/ThanhSinh-910/khaosat-mini)
* **🎥 Video Demo (Optional):** [Cung cấp nếu có]

---

## 2. FEATURE IMPLEMENTATION CHECKLIST
| # | Required Feature | Status | Implementation Details & Acceptance Level |
|:---:|---|:---:|---|
| 1 | **Responsive Mobile Viewport** | ✅ Complete | Giao diện Mobile-First hoàn toàn responsive (`css/responsive.css`), hỗ trợ hoàn hảo mọi kích thước màn hình điện thoại & tablet. Hỗ trợ Dark/Light Theme tự động (`prefers-color-scheme`). |
| 2 | **PWA Standalone & Manifest** | ✅ Complete | Cấu hình đầy đủ `manifest.json` (`display: standalone`, `theme_color: #1565C0`, icons 192×192 & 512×512 maskable). Có thể cài đặt trực tiếp từ trình duyệt lên màn hình chính không có thanh URL. |
| 3 | **Local Offline Persistence (IndexedDB)** | ✅ Complete | Form khảo sát cơ sở vật chất (chọn tòa nhà, tầng, phòng, đánh giá 10 hạng mục thiết bị, ghi chú, đính kèm ảnh). Lưu trực tiếp vào `IndexedDB` (`VKUFacilityDB > surveys`) qua async CRUD API trong `js/db.js`, bảo đảm an toàn dữ liệu 100% khi offline. |
| 4 | **Service Worker & Cache-First Strategy** | ✅ Complete | `service-worker.js` triển khai đầy đủ vòng đời (Install → Activate → Fetch). Chiến lược **Cache-First** pre-cache toàn bộ App Shell (HTML, CSS, JS, Icons), fallback về `index.html` khi mất mạng. |
| 5 | **Automatic Background Sync & Queue** | ✅ Complete | `js/network.js` bắt sự kiện `online`/`offline`, hiển thị banner trạng thái. Phiếu tạo khi offline mang cờ `synced: false`. `js/sync.js` tự động đồng bộ hàng đợi lên máy chủ ngay khi có mạng trở lại. |
| 6 | **Capacitor Android Packaging (Hybrid Bridge)** | ✅ Complete | Tích hợp thành công **Capacitor 8.5** với cấu hình `capacitor.config.ts` (`appId: edu.vku.facilitycheck`). Cài đặt các core plugins: `@capacitor/camera`, `@capacitor/network`, `@capacitor/geolocation`, `@capacitor/local-notifications`. Biên dịch thành công file Android APK (`vku-facility-check.apk` ~9.1MB) cài đặt và chạy mượt mà trên thiết bị Android thật. |

---

## 3. TECHNICAL ARCHITECTURE & PROJECT STRUCTURE

### 3.1. Kiến trúc Hybrid Model (Capacitor Native Bridge)

```
┌────────────────────────────────────────────────────────┐
│          Web Application (HTML5 / CSS3 / JS)           │
│          SPA Shell chạy trong tối ưu hóa WebView       │
├───────────────────────────┬────────────────────────────┤
│   PWA Layer (Web/Browser) │   Capacitor Bridge Layer   │
│   • Service Worker        │   • TypeScript Plugin Calls│
│   • Cache-First Strategy  │   • Message Passing (JNI)  │
│   • IndexedDB Local Store │   • Hardware APIs Dispatch │
└───────────────────────────┴─────────────┬──────────────┘
                                          ▼
┌────────────────────────────────────────────────────────┐
│                Native Android Device APIs              │
│       Camera • Network • Geolocation • Local Notif     │
└────────────────────────────────────────────────────────┘
```

### 3.2. Cấu trúc thư mục
```text
khaosat-mini/
├── index.html            # SPA App Shell – giao diện chính
├── manifest.json         # Cấu hình PWA manifest (Standalone)
├── service-worker.js     # Cache-First Service Worker offline
├── capacitor.config.ts   # Cấu hình Capacitor Bridge (appId, scheme, splash)
├── capacitor.config.json # Cấu hình JSON cho Capacitor CLI
├── package.json          # Dependencies: @capacitor/core, camera, network, geolocation...
├── vku-facility-check.apk# Tệp cài đặt Android APK hoàn chỉnh (~9.1MB)
├── css/
│   ├── style.css         # Design system, theme VKU Blue
│   └── responsive.css    # Responsive mobile-first viewports
├── js/
│   ├── app.js            # Hash Router, điều hướng & Dashboard
│   ├── db.js             # IndexedDB wrapper (VKUFacilityDB)
│   ├── network.js        # Bắt sự kiện Online/Offline & banner
│   ├── sync.js           # Hàng đợi đồng bộ & gửi lên server/Google Sheets
│   ├── survey.js         # Logic form khảo sát cơ sở vật chất
│   ├── history.js        # Lịch sử phiếu khảo sát & bộ lọc
│   └── statistics.js     # Thống kê & biểu đồ tình trạng thiết bị
├── assets/icons/         # Biểu tượng PWA & Mobile launcher (192px, 512px)
├── android/              # Native Android project (Capacitor generated source)
└── README.md             # Hướng dẫn chi tiết chạy, test & build APK
```

### 3.3. Luồng dữ liệu & Xử lý lỗi
- **Offline-First:** Submit form ➔ Ghi vào `IndexedDB` ngay lập tức (`synced: false`) ➔ Kiểm tra mạng: nếu online thì gọi `sync.js` đẩy lên server; nếu offline thì giữ trong queue.
- **Tự động phục hồi mạng:** Sự kiện `online` kích hoạt quét hàng đợi và đồng bộ tự động, đánh dấu `synced: true`.
- **Hybrid Bridge:** Trên Web chạy thuần Cache API + IndexedDB; khi đóng gói qua Capacitor, WebView nạp từ tài nguyên cục bộ `https://localhost` an toàn, tích hợp camera và notification native.

---

## 4. EMPIRICAL EVIDENCE & SCREENSHOTS

*(Chèn 3–4 ảnh chụp màn hình ứng dụng thực tế trên điện thoại hoặc Android Studio/DevTools)*

| 1. Dashboard & Form khảo sát trên Mobile | 2. Cài đặt PWA & Native App Icon |
|:---:|:---:|
| *(Giao diện khảo sát: chọn tòa nhà, tầng, phòng; đánh giá 10 thiết bị phòng học)* | *(App icon xuất hiện trên màn hình chính Android, mở ở chế độ Standalone không có thanh URL)* |

| 3. Banner Offline & Lưu trữ IndexedDB | 4. Android APK chạy qua Capacitor Bridge |
|:---:|:---:|
| *(Mất mạng hiển thị banner cảnh báo, phiếu vẫn lưu an toàn vào IndexedDB)* | *(File APK `vku-facility-check.apk` chạy trực tiếp trên thiết bị Android, kết nối Native Bridge)* |

---

## 5. TECHNICAL CHALLENGES & RESOLUTIONS

### Thách thức 1: Đảm bảo dữ liệu khảo sát không bị mất khi mất kết nối mạng giữa chừng
* **Vấn đề:** Khi kiểm tra phòng học ở các khu vực khuất sóng trong trường, form gửi trực tiếp qua HTTP sẽ báo lỗi và mất sạch dữ liệu nhập liệu của người dùng.
* **Giải pháp:** Thiết kế theo nguyên tắc **IndexedDB-First**: Khi bấm Submit, phiếu luôn được ghi vào `IndexedDB` trước với trạng thái `synced: false`. Dữ liệu tồn tại bền bỉ trên bộ nhớ máy dù reload trang, đóng app hoặc hết pin.

### Thách thức 2: Đóng gói PWA thành Android APK và giải quyết xung đột đường dẫn tài nguyên
* **Vấn đề:** Khi dùng Capacitor đóng gói thư mục gốc web (`webDir: '.'`), Capacitor CLI báo lỗi `"." is not a valid value for webDir` và thiếu cấu hình Android SDK `sdk.dir` khi chạy Gradle.
* **Giải pháp:** 
  1. Cấu hình thư mục phân phối `www` chứa toàn bộ assets của PWA, cập nhật `capacitor.config.ts` trỏ đúng `webDir: 'www'`.
  2. Bổ sung `sdk.dir` vào `android/local.properties` trỏ đến Android SDK cục bộ.
  3. Chạy `gradlew assembleDebug`, tạo thành công gói cài đặt độc lập `vku-facility-check.apk` (dung lượng gọn nhẹ ~9.1MB, chạy mượt mà trên Android).
