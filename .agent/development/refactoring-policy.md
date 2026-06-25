# Refactoring Policy — Dọn dẹp code

## Nguyên tắc số 1

> Refactor = chỉ đổi cách viết, không đổi behavior. Nếu test fail sau refactor → đã làm sai.

---

## Khi nào được refactor

✅ Được phép:
- File > 300 dòng cần tách
- Method > 30 dòng cần extract
- Logic trùng lặp ở 2+ chỗ cần gộp
- Tên không rõ nghĩa cần đổi

❌ Không được refactor khi:
- Đang giữa phase đang fix bug
- Chưa có test cho code sắp refactor
- Sát deadline release

---

## Quy tắc vàng: không refactor + feature cùng lúc

```
❌ Sai:
"Tôi sẽ refactor RpsEngine và thêm luôn logic item mới"

✅ Đúng:
Commit 1: refactor RpsEngine (behavior không đổi, test vẫn pass)
Commit 2: thêm logic item mới
```

---

## Quy trình refactor

### 1. Xác nhận có test bao phủ
Trước khi refactor, chạy:
```bash
flutter test [file liên quan]
```
Nếu chưa có test → viết test trước, sau đó mới refactor.

### 2. Refactor từng bước nhỏ

Không làm tất cả trong 1 lần:
```
Lần 1: đổi tên variable → chạy test → pass
Lần 2: extract method → chạy test → pass
Lần 3: tách file → chạy test → pass
```

### 3. Sau mỗi bước phải pass test

```bash
flutter test  # phải green sau mỗi bước
```

---

## Các loại refactor cụ thể

### Đổi tên (Rename)
```dart
// Nếu đổi tên public method/field, phải tìm và update TẤT CẢ nơi gọi
// Dùng IDE rename để không miss

// ❌ Sai: đổi tên nhưng để lại alias
Gesture get gesture => move;  // tạo confusion

// ✅ Đúng: đổi hết, xóa cái cũ
```

### Tách file (Extract)
```
Khi tách game_state.dart thành game_state.dart + round_result.dart:
- Tạo file mới
- Chuyển code
- Update tất cả import
- Chạy flutter test
- Xóa code cũ
```

### Gộp logic trùng lặp (DRY)
```dart
// Tìm pattern lặp ở 2+ chỗ
// Extract thành 1 method/class ở shared/
// Update cả 2 chỗ dùng
// Test cả 2 flow
```

---

## Báo cáo refactor

```
## Refactor: [mô tả]

Lý do: [tại sao cần refactor]
Files thay đổi: [danh sách]
Behavior thay đổi: KHÔNG (nếu có → đây không phải refactor)

Kết quả: flutter test ✅ [N] passed — số test không giảm
```