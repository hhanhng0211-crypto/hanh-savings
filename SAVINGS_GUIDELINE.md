# 💰 Savings Tracker — Hướng Dẫn Sử Dụng

## 🌐 Mở App

### Trên Laptop
- Mở trình duyệt → vào **https://hhanhng0211-crypto.github.io/hanh-savings/**
- Hoặc double-click file `index.html` trong folder `~/Desktop/Hanh-Antigravity/Personal/Saving/`

### Trên Điện Thoại
- Mở link **https://hhanhng0211-crypto.github.io/hanh-savings/**
- Để thêm icon app: Safari → Share → **"Add to Home Screen"**

---

## ☁️ Đồng Bộ Dữ Liệu (Firebase)

### Lần đầu tiên
1. **Laptop**: Mở file local → **Export Data** (cuối trang) → lưu file `.json`
2. **Laptop**: Mở link online → **Import Data** → chọn file `.json`
3. Bấm **"Đăng nhập Google"** (sidebar hoặc tab Sync trên mobile)
4. ✅ Dữ liệu đẩy lên Firebase

### Các lần sau
- Mở link trên bất kỳ thiết bị → **Đăng nhập Google** → data tự tải về
- Nhập liệu trên 1 thiết bị → các thiết bị khác **tự động cập nhật real-time**

### Đăng xuất
- Laptop: sidebar → **"🚪 Đăng xuất"**
- Mobile: tab **"Sync"** ở bottom bar

---

## 📱 Giao Diện Mobile
- Bottom navigation bar: Tổng quan | Thu chi | Tài sản | TK&Vàng | Forecast | Sync
- Layout tự động responsive

---

## 🔧 Push Code Lên GitHub (sau khi sửa file)

Mở **Terminal** và chạy:
```bash
cd ~/Desktop/Hanh-Antigravity/Personal/Saving
git add .
git commit -m "mô tả thay đổi"
git push
```
Chờ 1-2 phút → trang online tự cập nhật.

---

## 📁 Cấu Trúc File

| File | Mô tả |
|---|---|
| `index.html` | App chính (HTML + CSS + JS) |
| `SAVINGS_GUIDELINE.md` | File hướng dẫn này |

---

## 🔑 Tài Khoản & Config

| Service | Thông tin |
|---|---|
| **Firebase Project** | `hanh---savings` |
| **Firebase Console** | https://console.firebase.google.com |
| **GitHub Repo** | https://github.com/hhanhng0211-crypto/hanh-savings |
| **GitHub Pages** | https://hhanhng0211-crypto.github.io/hanh-savings/ |
| **Storage Key** | `hanh_finance_v4` (localStorage) |

---

## ⚠️ Lưu Ý
- Dữ liệu lưu trong **localStorage** + **Firebase Realtime Database**
- Nếu chưa đăng nhập Google → data chỉ lưu local (mỗi trình duyệt riêng)
- Đăng nhập Google → data sync giữa tất cả thiết bị
- **Export thường xuyên** để backup phòng trường hợp mất data
