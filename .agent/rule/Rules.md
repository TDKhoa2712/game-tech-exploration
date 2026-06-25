# KBB — Quy tắc dự án (AI phải đọc file này trước khi làm bất cứ việc gì)

## Dự án là gì

Game mobile Flutter "Kéo Búa Bao" — dân gian VN, 2 người chơi, PvE + PvP.
Stack: Flutter · Flame · Riverpod · Hive · GoRouter.

---

## Trước khi viết bất kỳ dòng code nào, AI phải hỏi

1. **Task này thuộc loại nào?** → đọc file tương ứng
   - Thêm chức năng mới → `development/feature-policy.md`
   - Sửa bug → `development/bugfix-policy.md`
   - Refactor → `development/refactoring-policy.md`

2. **File nào sẽ bị chạm vào?** → đọc `architecture/boundaries.md`

3. **Code xong thì test gì?** → đọc `development/testing-policy.md`

---

## Những điều AI tuyệt đối không được làm

| Cấm | Lý do |
|---|---|
| Sửa file không liên quan đến task | Gây regression không kiểm soát |
| Xóa hoặc đổi tên public API/method | Có thể break provider hoặc test |
| Import framework vào domain layer | Vi phạm kiến trúc, domain phải thuần Dart |
| Tự thêm dependency vào pubspec.yaml | Phải hỏi người dùng trước |
| Viết logic trong widget | Logic thuộc về provider/service |
| Bỏ qua bước test sau khi sửa | Mọi thay đổi phải có test chạy pass |
| Giả định behavior của file không được attach | Hỏi lại hoặc yêu cầu attach file đó |

---

## Cam kết của AI sau mỗi task

Sau khi hoàn thành, AI phải tự báo cáo:

```
## Báo cáo task
- Files đã thay đổi: [danh sách]
- Files KHÔNG thay đổi nhưng có thể bị ảnh hưởng: [danh sách]
- Test cần chạy: [lệnh cụ thể]
- Rủi ro tiềm ẩn: [nếu có]
```

---

## Index tài liệu

| File | Đọc khi nào |
|---|---|
| `architecture/architecture.md` | Cần hiểu tổng thể hệ thống |
| `architecture/boundaries.md` | Cần biết file nào được phép gọi file nào |
| `architecture/dependencies.md` | Cần thêm/sửa dependency |
| `development/coding-standards.md` | Viết code mới |
| `development/feature-policy.md` | Thêm chức năng mới |
| `development/bugfix-policy.md` | Sửa bug |
| `development/refactoring-policy.md` | Refactor code |
| `development/testing-policy.md` | Viết hoặc chạy test |
| `development/logging-policy.md` | Thêm log |
| `quality/definition-of-done.md` | Kiểm tra task đã xong thật chưa |
| `quality/code-review-checklist.md` | Review code trước khi merge |
| `quality/regression-checklist.md` | Sau khi sửa bug hoặc refactor |