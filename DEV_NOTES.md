# 🛠️ DEV NOTES — Savings Tracker (Hanh)

> **Mục đích file này:** Đọc file này trước khi sửa bất cứ thứ gì. Nó giải thích toàn bộ app hoạt động thế nào, tính năng nào đang có, bug nào từng xảy ra, và tại sao code viết theo cách đó. Tránh sửa lung tung gây lỗi cũ quay lại.

---

## 📁 File & Hosting

| Thứ | Chi tiết |
|---|---|
| **File chính** | `Personal/Saving/index.html` (monolithic ~800 dòng) |
| **URL live** | `https://hhanhng0211-crypto.github.io/hanh-savings/` |
| **Push code** | `cd ~/Desktop/Hanh-Antigravity/Personal/Saving && git add . && git commit -m "..." && git push` |
| **Cache bust** | Thêm `?v=N` vào URL để force iOS/browser load bản mới |

---

## 🏗️ Kiến trúc tổng quan

```
index.html
├── CSS (inline <style>)
├── Firebase SDK (app-compat, auth-compat, database-compat)
├── Data layer (localStorage key: hanh_finance_v4)
├── Firebase sync layer (shared path + user path)
└── UI render (vanilla JS, không dùng framework)
```

**Không có framework, không có build tool.** Mọi thứ viết thẳng trong 1 file HTML. Nếu ai đề nghị dùng React/Vue — không cần thiết với app này.

---

## 💾 Data Model (Object D)

```javascript
D = {
  incomes: [{id, name, amount}],
  expenses: [{id, name, amount}],
  assets: [{id, name, value}],              // Tài sản hiện có (tiền mặt, crypto, etc.)
  bankDeposits: [{                           // Gửi tiết kiệm ngân hàng
    id, amount, owner, date,                 // date = ngày gửi (YYYY-MM-DD)
    term,                                    // kỳ hạn (tháng)
    rate,                                    // lãi suất %/năm
    renewals: [{date, interest, newAmount}]  // lịch sử đáo hạn tự động
  }],
  goldHoldings: [{id, type, chi, price, date}], // type: '610' hoặc '9999'
  goldPrices: {g610: ..., g9999: ...},       // giá vàng hiện tại (tự fetch)
  goldPriceDate: "YYYY-MM-DD",
  tracking: { "YYYY-M": {gold, goldChi, depositDate, ...} }, // tracking cũ (legacy)
  monthTargets: { "YYYY-M": {bank, gold} }, // mục tiêu tháng
  dashTarget: 1000000000,                   // mục tiêu dashboard (mặc định 1 tỷ)
  cfg: { bankRate: 7.9 }
}
```

---

## ☁️ Firebase Sync

### Paths
```
users/{uid}/data     → Laptop (đăng nhập Google, real-time listener)
shared/021100/data   → Điện thoại (không cần login, mã cố định)
```

> ⚠️ **SHARE_CODE phải là hardcode `'021100'`** — không đọc từ localStorage. Bug cũ: nếu để đọc từ localStorage thì lần đầu chạy code là `''`, path thành `shared//data` → Firebase reject.

### Firebase Rules (phải set đúng như này)
```json
{
  "rules": {
    "users": {
      "$uid": { ".read": "$uid === auth.uid", ".write": "$uid === auth.uid" }
    },
    "shared": { ".read": true, ".write": true }
  }
}
```

### Luồng sync (QUAN TRỌNG — đọc kỹ trước khi sửa)

```
Laptop (đăng nhập):
  Lưu dữ liệu  →  save() → cloudSave() → ghi users/{uid}/data VÀ shared/021100/data
  Nhận realtime ← cloudListen() lắng nghe CẢ 2 paths song song

Điện thoại (không login):
  Lưu dữ liệu  →  save() → cloudSave() → chỉ ghi shared/021100/data
  Bấm ⬇️ Tải về → forceSync() → đọc shared/021100/data
  Bấm ⬆️ Đẩy lên → forceUpload() → ghi shared/021100/data

PWA mode (Add to Home Screen):
  Khi mở app → AUTO pull từ shared/ trước (pwaReady=false)
  Chặn cloudSave() cho đến khi pull xong (tránh ghi đè data mới bằng data cũ)
  Sau khi pull xong → mới cho phép lưu bình thường
```

### Hàm sync quan trọng

| Hàm | Làm gì |
|---|---|
| `cloudSave()` | Ghi data lên Firebase (cả 2 paths). Chạy tự động sau mỗi `save()`. Có flag `pwaReady` để chặn khi mới vào PWA |
| `cloudListen()` | Mở 2 real-time listener (users/ và shared/). Chỉ chạy khi đã login |
| `cloudStop()` | Tắt cả 2 listener |
| `applyCloudData(v)` | Apply data từ cloud: `D = Object.assign(def(), v)` — bắt buộc dùng `Object.assign` để điền default cho các field bị thiếu |
| `forceSync()` | Pull 1 lần từ shared/ về (nút ⬇️) |
| `forceUpload()` | Push 1 lần từ local lên shared/ (nút ⬆️) |

---

## 🔐 Authentication

- **Login desktop**: `signInWithPopup` (không dùng redirect trên desktop — nó load lại trang)  
- **Login mobile**: `signInWithRedirect` (popup bị Safari chặn trên iOS)  
- **Detect mobile**: `/iPhone|iPad|Android/i.test(navigator.userAgent)`  
- **Sau redirect**: `getRedirectResult()` bắt kết quả → auto gọi `forceSync()`

> 💡 App vẫn hoạt động đầy đủ khi **không đăng nhập Google**. Sync vẫn được qua `shared/` path.

---

## 📱 UI & Navigation

- **Desktop**: Sidebar trái cố định (250px), nội dung bên phải
- **Mobile**: Ẩn sidebar, hiện bottom nav bar (`.mob-nav`)
- **Bottom nav buttons**: Tổng quan | Thu chi | Tài sản | TK&Vàng | Forecast | Login | ⬇️ Tải về | ⬆️ Đẩy lên
- **PWA**: file có `<meta name="apple-mobile-web-app-capable">` → khi Add to Home Screen trên iOS sẽ fullscreen, không thanh địa chỉ

---

## ✨ Tính năng đã có (đừng xóa/vỡ)

### Dashboard (Tổng quan)
- Tổng quan tài chính: thu nhập, chi phí, tiết kiệm/tháng
- Progress bar đến mục tiêu tài chính (`dashTarget`)
- Tiết kiệm theo mục (bank + vàng)
- Forecast đến mục tiêu 1 tỷ/300 triệu

### Thu nhập & Chi phí
- Thêm/sửa/xóa khoản thu & chi
- Tính tự động số tiền tiết kiệm được/tháng

### Tài sản hiện có
- Track các tài sản lỏng (tiền mặt, crypto, v.v.)

### Tiết kiệm & Vàng (Tab chính)
- Gửi tiết kiệm ngân hàng: nhập số tiền, người gửi (Hạnh/Quân), ngày gửi, kỳ hạn, lãi suất
- Tính lãi kép theo số tháng đã gửi
- **Auto đáo hạn (tính năng mới)**: `checkMaturity()` chạy khi load app — tự cộng lãi vào gốc khi đến hạn, lưu lịch sử renewals, hiện toast thông báo
- Mua vàng: nhập loại (610/9999), số chỉ, giá mua
- Auto fetch giá vàng từ API (BTMC) mỗi khi vào app

### Overview
- Bảng tổng hợp theo năm

### Forecast 1 tỷ
- Dự báo thời gian đạt mục tiêu với các milestone

### Import/Export
- Export JSON để backup
- Import JSON để restore hoặc chuyển thiết bị

---

## 🐛 Bug History (chẩn bệnh nhanh)

### BUG 1 — Sync crash: `Cannot read properties of undefined (reading 'reduce')`
- **Khi nào**: Bấm Sync trên điện thoại → app crash/toast lỗi
- **Nguyên nhân**: Data trên Firebase thiếu field `assets` (data cũ chưa có field này). Khi `D = v` → `D.assets = undefined` → `render()` gọi `D.assets.reduce()` → crash
- **Fix**: Thay `D = v` bằng `D = Object.assign(def(), v)` ở tất cả chỗ apply cloud data (`applyCloudData`, `forceSync`, `forceUpload`)
- **Chỗ fix**: hàm `applyCloudData()` và `forceSync()`

### BUG 2 — Firebase path sai: `shared//data` (double slash)
- **Khi nào**: Data không lên Firebase, console có lỗi "invalid path"
- **Nguyên nhân**: `SHARE_CODE` là `''` (rỗng) vì đọc từ `localStorage` chưa được set → path = `shared//data`
- **Fix**: `const SHARE_CODE = '021100'` (hardcode), không đọc từ localStorage
- **Chỗ cần check**: dòng khai báo `SHARE_CODE`

### BUG 3 — Login không được trên iPhone
- **Khi nào**: Bấm Login trên điện thoại → bị block, lỗi `disallowed_useragent`
- **Nguyên nhân**: `signInWithPopup` bị Safari/in-app browser chặn
- **Fix**: Detect mobile → dùng `signInWithRedirect`, desktop → `signInWithPopup`
- **Chỗ fix**: hàm `fbLogin()`

### BUG 4 — PWA ghi đè data mới bằng data cũ
- **Khi nào**: Dùng app từ màn hình chính (Add to Home Screen) → data bị reset về cũ
- **Nguyên nhân**: iOS PWA storage **TÁCH BIỆT** với Safari browser. Khi PWA khởi động, `localStorage` của nó có thể trống hoặc rất cũ. `checkMaturity()` chạy → `save()` → `cloudSave()` → **ghi đè Firebase bằng data rỗng/cũ**
- **Fix**: Detect `window.navigator.standalone` → nếu PWA thì pull từ Firebase trước (`pwaReady=false`), chặn `cloudSave()` cho đến khi pull xong
- **Chỗ fix**: đoạn init sau `rFbAuth()`

### BUG 5 — Laptop không nhận update từ điện thoại
- **Khi nào**: Điện thoại sửa data + đẩy lên → laptop không tự cập nhật
- **Nguyên nhân**: `cloudListen()` chỉ lắng nghe `users/{uid}/data`. Điện thoại (không login) chỉ ghi vào `shared/021100/data` → laptop không biết
- **Fix**: Thêm `fbSharedListener` lắng nghe `shared/` song song với listener auth
- **Chỗ fix**: hàm `cloudListen()` và `cloudStop()`

### BUG 6 — Nút Login và Nút Sync trùng nhau
- **Khi nào**: Bottom nav mobile chỉ có 1 nút, vừa là Login vừa là Sync
- **Nguyên nhân**: Logic ban đầu gộp chung 1 nút, toggle theo auth state
- **Fix**: Tách thành 3 nút riêng: `mobLogin`, `mobSync` (⬇️), `mobUpload` (⬆️). `rFbAuth()` ẩn/hiện tùy trạng thái

---

## ⚠️ Những thứ KHÔNG được làm (anti-patterns)

1. **Đừng** dùng `D = cloudData` trực tiếp — luôn dùng `Object.assign(def(), cloudData)` để tránh thiếu field
2. **Đừng** để `SHARE_CODE` đọc từ `localStorage` — phải hardcode
3. **Đừng** gọi `cloudSave()` trước khi `pwaReady = true` trong PWA mode
4. **Đừng** chỉ lắng nghe 1 path Firebase — phải nghe cả `users/` và `shared/`
5. **Đừng** dùng `signInWithPopup` trên mobile — dùng `signInWithRedirect`
