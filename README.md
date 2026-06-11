# Hướng dẫn triển khai Depot Manager PWA

## Tổng quan

Depot Manager là Progressive Web App (PWA) chạy trên trình duyệt Chrome/Safari của máy tính bảng.
Dữ liệu đồng bộ qua Google Sheets — miễn phí, không cần server riêng.

---

## Bước 1 — Tạo Google Sheet và Apps Script (làm 1 lần duy nhất)

### 1.1 Tạo Google Sheet

1. Mở **sheets.google.com** → đăng nhập bằng tài khoản Google của công ty
2. Nhấn **+ Bảng tính trống**
3. Đặt tên file: `Depot Container`
4. Nhấp đúp vào tab `Trang tính 1` ở dưới → đổi tên thành `containers`

### 1.2 Tạo Apps Script

1. Trong Google Sheet, vào menu **Tiện ích mở rộng → Apps Script**
2. Xoá toàn bộ code mặc định trong editor
3. Dán đoạn code sau vào:

```javascript
function doGet(e) {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sh = ss.getSheetByName('containers');
  var act = e.parameter.action;

  if (act === 'load') {
    var data = sh.getDataRange().getValues();
    return ContentService
      .createTextOutput(JSON.stringify(data))
      .setMimeType(ContentService.MimeType.JSON);
  }

  if (act === 'save') {
    var rows = JSON.parse(e.parameter.data);
    sh.clearContents();
    rows.forEach(function(r) { sh.appendRow(r); });
    return ContentService.createTextOutput('ok');
  }
}
```

4. Nhấn biểu tượng **💾 Lưu** (hoặc Ctrl+S)
5. Đặt tên project: `DepotManager` → OK

### 1.3 Deploy Web App

1. Nhấn nút **Deploy** (góc trên phải) → **New deployment**
2. Nhấn biểu tượng ⚙️ bên cạnh "Select type" → chọn **Web app**
3. Điền thông tin:
   - Description: `Depot Manager v1`
   - Execute as: **Me** (tài khoản của bạn)
   - Who has access: **Anyone**
4. Nhấn **Deploy**
5. Google sẽ yêu cầu cấp quyền → nhấn **Authorize access** → chọn tài khoản → Allow
6. **Copy URL** hiện ra (dạng: `https://script.google.com/macros/s/AKfy.../exec`)
   → Lưu URL này lại, cần dùng ở bước tiếp theo

---

## Bước 2 — Host file ứng dụng lên GitHub Pages (miễn phí)

### 2.1 Tạo tài khoản GitHub (nếu chưa có)

Truy cập **github.com** → Sign up → tạo tài khoản miễn phí

### 2.2 Tạo repository và upload file

1. Đăng nhập GitHub → nhấn **New repository**
2. Repository name: `depot-manager`
3. Chọn **Public**
4. Nhấn **Create repository**
5. Nhấn **uploading an existing file**
6. Kéo 3 file vào: `index.html`, `manifest.json`, `sw.js`
7. Nhấn **Commit changes**

### 2.3 Bật GitHub Pages

1. Vào **Settings** của repository
2. Mục **Pages** (thanh trái)
3. Source: **Deploy from a branch** → Branch: **main** → / (root)
4. Nhấn **Save**
5. Sau 1-2 phút, URL sẽ xuất hiện dạng:
   `https://[tên-account].github.io/depot-manager/`

---

## Bước 3 — Cài ứng dụng lên máy tính bảng

### Trên Android (Chrome)

1. Mở Chrome → truy cập URL GitHub Pages ở trên
2. Nhấn menu **⋮** (3 chấm góc phải) → **Add to Home screen**
3. Đặt tên: `Depot` → **Add**
4. Icon xuất hiện trên màn hình chính như app thật

### Trên iPad/iPhone (Safari)

1. Mở Safari → truy cập URL GitHub Pages
2. Nhấn biểu tượng **Chia sẻ** (hình vuông có mũi tên lên)
3. Chọn **Add to Home Screen**
4. Đặt tên: `Depot` → **Add**

---

## Bước 4 — Cài đặt lần đầu trong app

1. Lần đầu mở app, màn hình **Cài đặt** sẽ hiện ra
2. Dán **URL Apps Script** (đã copy ở Bước 1.3) vào ô
3. Nhấn **Lưu & Kiểm tra kết nối**
4. Nếu hiện `✓ Kết nối thành công` → Hoàn thành!
5. Trên các máy tính bảng khác: cài app → nhập cùng URL → dữ liệu tự đồng bộ

---

## Cấu hình depot (tuỳ chỉnh cho depot của bạn)

Mặc định app cài sẵn: **5 dãy (A-E)**, **3 tầng**, **10 bay/slot**.

Để thay đổi, mở file `index.html`, tìm đoạn:

```javascript
const CFG = {
  rows: ['A','B','C','D','E'],   // ← thêm/bớt dãy
  tiers: [1,2,3],                // ← số tầng
  bays: 10                       // ← số bay mỗi slot
};
```

Ví dụ depot có 8 dãy, 4 tầng, 15 bay:
```javascript
const CFG = {
  rows: ['A','B','C','D','E','F','G','H'],
  tiers: [1,2,3,4],
  bays: 15
};
```

Sau khi sửa, upload lại file `index.html` lên GitHub → app tự cập nhật.

---

## Tính năng

| Tính năng | Mô tả |
|-----------|-------|
| **Sơ đồ depot** | Xem toàn bộ vị trí container, nhấn vào để xem chi tiết |
| **Lấy ra nhanh** | Nhập số container → lấy ra trong 2 giây |
| **Lấy ra thủ công** | Chọn Dãy → Tầng → Bay |
| **Nhập vào** | Nhập đầy đủ thông tin container + vị trí |
| **Tìm kiếm** | Tìm theo số container, khách hàng, dãy |
| **Lịch sử** | Ghi log tất cả thao tác trong ngày |
| **Đồng bộ** | Nhấn ⟳ Sync để đồng bộ với Google Sheets |
| **Offline** | Vẫn dùng được khi mất mạng, sync khi có mạng lại |

---

## Lưu ý quan trọng

- **Số container** phải đúng chuẩn ISO: 4 chữ cái + 7 số (VD: `TCKU1234567`)
- **Đồng bộ**: nhấn nút ⟳ Sync sau mỗi ca làm việc để đảm bảo dữ liệu nhất quán
- **Backup**: Google Sheets tự lưu lịch sử phiên bản — có thể khôi phục nếu cần
- **Nhiều máy**: nếu 2 người thao tác cùng lúc, người dùng sau nên Sync trước khi thao tác

---

## Hỗ trợ

Nếu gặp sự cố, kiểm tra theo thứ tự:
1. URL Apps Script có đúng không? (phải kết thúc bằng `/exec`)
2. Deploy có chọn "Anyone" chưa?
3. Tên sheet có đúng là `containers` (chữ thường) không?
