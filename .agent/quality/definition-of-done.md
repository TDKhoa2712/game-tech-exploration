# Definition of Done — Khi nào task mới được coi là xong

## Task chưa xong nếu còn bất kỳ điều nào sau đây

---

## Checklist chung (mọi task)

```
[ ] flutter analyze — 0 error (warning ≤ 5 toàn project)
[ ] flutter test — 0 failed
[ ] Không có TODO/FIXME mới nào không có issue đi kèm
[ ] Không có commented-out code
[ ] Không có print() hoặc debugPrint() còn sót
[ ] Tất cả public method có đủ tham số và return type rõ ràng
```

---

## Checklist thêm feature mới

```
[ ] Có ít nhất 1 unit test cho business logic mới
[ ] Test cover happy path + ít nhất 1 edge case
[ ] File mới đặt đúng folder theo architecture.md
[ ] Import đúng theo coding-standards.md
[ ] Nếu thêm GameState field → copyWith và == đã update
[ ] Nếu thêm route → đã khai báo trong core/router.dart
[ ] Nếu thêm Hive box → đã đăng ký trong hive_manager.dart
```

---

## Checklist sửa bug

```
[ ] Có test tái hiện bug (test fail trước fix, pass sau fix)
[ ] Nguyên nhân gốc rễ đã được fix, không phải che triệu chứng
[ ] Chạy regression: flutter test test/integration/ — pass
[ ] Các file không liên quan không bị chạm vào
```

---

## Checklist refactor

```
[ ] Số lượng test không giảm
[ ] flutter test pass 100% trước và sau refactor
[ ] Behavior không thay đổi (có thể verify bằng test)
[ ] Không có feature mới lẫn vào commit refactor
```

---

## Checklist Phase 1 Done (toàn bộ MVP)

```
[ ] flutter test — tất cả pass
[ ] flutter analyze — 0 error
[ ] Manual test checklist trong F8_integration_test.md — ✅ hết
[ ] Chạy được trên Android thật (không chỉ emulator)
[ ] Cả 2 thành viên demo được trên điện thoại của nhau
[ ] git tag v0.1.0-phase1-mvp đã tạo
```