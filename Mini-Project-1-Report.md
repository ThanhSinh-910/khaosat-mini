# MINI-PROJECT SHORT TECHNICAL REPORT
**Course:** Cross-Platform Mobile App Development (VKU)  
**Mini-Project Title:** Mini-Project 1: VKU Field Survey PWA  
**Team / Student Name:** Nguyễn Thanh Sinh (GitHub: ThanhSinh-910)  
**Submission Date:** 14/10/2026  

---

## 1. GENERAL INFORMATION & DELIVERABLE LINKS
* **Team Members:**
  1. Nguyễn Thanh Sinh — Student ID: 23IT.B188 — Role: Fullstack Mobile Developer (PWA & Offline Architecture) — Contribution: 100%
* **🔗 Live Demo URL:** [https://thanhsinh-910.github.io/khaosat-mini/](https://thanhsinh-910.github.io/khaosat-mini/)
* **💻 GitHub Repository:** [https://github.com/ThanhSinh-910/khaosat-mini](https://github.com/ThanhSinh-910/khaosat-mini)
* **🎥 Video Demo (Optional):** [Cung cấp nếu có]

---

## 2. FEATURE IMPLEMENTATION CHECKLIST
| # | Required Feature | Status | Implementation Details & Acceptance Level |
|:---:|---|:---:|---|
| 1 | **Responsive Mobile Viewport** | ✅ Complete | Giao diện Mobile-First hoàn toàn responsive (`css/responsive.css`), hỗ trợ mọi kích thước màn hình điện thoại và máy tính bảng. Hỗ trợ Dark/Light tự động theo hệ thống (`prefers-color-scheme`). |
| 2 | **PWA Installable (Standalone Mode)** | ✅ Complete | Cấu hình đầy đủ `manifest.json` (`display: standalone`, `theme_color: #1565C0`, bộ icons 192×192 & 512×512 maskable). Có thể cài đặt lên màn hình chính điện thoại Android (Chrome) và iOS (Safari) như app native. |
| 3 | **Local Offline Persistence (IndexedDB)** | ✅ Complete | Toàn bộ phiếu khảo sát được lưu vào `IndexedDB` (`VKUFacilityDB > surveys`) qua module `js/db.js` với API async CRUD đầy đủ. Dữ liệu tồn tại vĩnh viễn trên thiết bị, không mất khi reload hay mất điện. |
| 4 | **Service Worker & Cache-First Strategy** | ✅ Complete | `service-worker.js` thực hiện vòng đời đầy đủ (Install → Activate → Fetch). Chiến lược **Cache-First** pre-cache toàn bộ App Shell (HTML, CSS, JS, Icons). Fallback về `index.html` khi offline. Cập nhật cache khi có phiên bản mới (`VKU-CACHE-v2`). |
| 5 | **Automatic Background Sync** | ✅ Complete | `js/network.js` lắng nghe sự kiện `online`/`offline`, hiển thị banner trạng thái kết nối. Các phiếu khi offline được đánh dấu `synced: false`. `js/sync.js` tự động đồng bộ hàng đợi lên server khi phát hiện có mạng trở lại. |

---

## 3. TECHNICAL ARCHITECTURE & PROJECT STRUCTURE

### 3.1. Kiến trúc hệ thống (Offline-First SPA)

```
┌─────────────────────────────────────────┐
│           PWA Frontend (SPA)            │
│  index.html – Hash Router (#page)       │
├─────────────────────────────────────────┤
│           JavaScript Modules            │
│  app.js       → Router, Dashboard       │
│  db.js        → IndexedDB abstraction   │
│  network.js   → Online/offline detect   │
│  sync.js      → Queue & Sync service    │
│  survey.js    → Form tạo/sửa phiếu     │
│  history.js   → Lịch sử, filter        │
│  statistics.js → Thống kê, chart       │
├─────────────────────────────────────────┤
│           Service Worker                │
│  Cache-First cho tất cả static assets   │
│  Offline fallback → index.html          │
├─────────────────────────────────────────┤
│           IndexedDB Storage             │
│  VKUFacilityDB > surveys                │
└─────────────────────────────────────────┘
```

### 3.2. Cấu trúc thư mục
```text
khaosat-mini/
├── index.html            # SPA App Shell – giao diện chính
├── manifest.json         # Cấu hình PWA & thông tin cài đặt
├── service-worker.js     # Cache-First Service Worker offline
├── css/
│   ├── style.css         # Design system, theme VKU Blue
│   └── responsive.css    # Responsive mobile-first viewports
├── js/
│   ├── app.js            # Hash Router, điều hướng & Dashboard
│   ├── db.js             # IndexedDB wrapper (VKUFacilityDB)
│   ├── network.js        # Phát hiện Online/Offline & banner
│   ├── sync.js           # Hàng đợi đồng bộ & gửi lên server
│   ├── survey.js         # Logic form khảo sát cơ sở vật chất
│   ├── history.js        # Danh sách lịch sử khảo sát & lọc
│   └── statistics.js     # Thống kê & biểu đồ tình trạng phòng
├── assets/icons/         # Biểu tượng PWA (192px, 512px maskable)
├── server.py             # Dev server Python để test local
└── README.md             # Hướng dẫn chi tiết chạy & kiểm thử
```

### 3.3. Luồng dữ liệu & Exception Handling
- **Luồng Offline → Sync:**
  `Người dùng nhập phiếu` → `IndexedDB (synced: false)` → *(có mạng)* → `sync.js gửi lên server` → `synced: true`
- **Exception handling:** Nếu request sync thất bại (timeout, lỗi server), bản ghi vẫn ở `synced: false` và được thử lại ở lần online tiếp theo, đảm bảo không mất dữ liệu.
- **Cache miss handling:** Khi offline và tài nguyên chưa được cache, Service Worker trả về `index.html` (SPA shell) thay vì 404.

---

## 4. EMPIRICAL EVIDENCE & SCREENSHOTS

*(Chèn 3–4 ảnh chụp màn hình ứng dụng thực tế trên điện thoại hoặc DevTools)*

| 1. Dashboard & Form khảo sát trên Mobile | 2. Cài đặt PWA – Standalone Mode |
|:---:|:---:|
| *(Giao diện khảo sát: chọn tòa nhà, tầng, phòng; đánh giá thiết bị bằng 3 trạng thái)* | *(Nút "Thêm vào màn hình chính" trên Chrome Android, sau khi cài không còn thanh URL)* |

| 3. Banner Offline & Phiếu lưu vào IndexedDB | 4. Lighthouse PWA Audit |
|:---:|:---:|
| *(Banner đỏ hiện khi mất mạng, app vẫn tạo và lưu phiếu bình thường)* | *(Chrome DevTools → Lighthouse → PWA score ≥ 90, Service Worker ✅, Offline ✅, Installable ✅)* |

---

## 5. TECHNICAL CHALLENGES & RESOLUTIONS

### Thách thức 1: Dữ liệu không bị mất khi mất mạng giữa chừng khi nhập liệu
* **Vấn đề:** Người dùng kiểm tra phòng học tại vùng sóng yếu — nếu form chỉ POST lên server thì mất toàn bộ dữ liệu khi offline.
* **Giải pháp:** Áp dụng chiến lược **IndexedDB-first**: Mỗi phiếu được ghi vào `IndexedDB` với trạng thái `synced: false` **ngay khi bấm Submit**, trước khi thực hiện bất kỳ request network nào. Đảm bảo 100% dữ liệu không bị mất dù reload trang hay tắt ứng dụng.

### Thách thức 2: Đồng bộ tự động khi chuyển vùng có/không có sóng
* **Vấn đề:** Người dùng di chuyển liên tục qua nhiều phòng, sóng không ổn định. Cần đồng bộ ngầm mà không ảnh hưởng trải nghiệm.
* **Giải pháp:** Kết hợp `window.addEventListener('online', ...)` trong `network.js` với hàm `syncPendingSurveys()` trong `sync.js`. Khi phát hiện kết nối phục hồi, hệ thống tự động duyệt toàn bộ hàng đợi `synced: false` và gửi tuần tự lên server, cập nhật trạng thái `synced: true` khi thành công — hoàn toàn trong nền, không làm gián đoạn người dùng.
