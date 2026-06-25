# Code Review Checklist

> Dùng trước khi merge bất kỳ thay đổi nào vào main branch.
> AI tự review output của mình trước khi trả lời người dùng.

---

## Kiến trúc

```
[ ] File nằm đúng folder theo architecture.md
[ ] Không vi phạm boundaries.md (domain không import Flutter, v.v.)
[ ] Feature mới không vượt phase hiện tại
[ ] Không tự thêm package mới vào pubspec.yaml
```

---

## Code quality

```
[ ] Không có magic number — dùng constant
[ ] Không có nested ternary > 2 cấp
[ ] Method không quá 30 dòng
[ ] File không quá 300 dòng
[ ] Tên biến/method nói lên ý nghĩa, không cần comment giải thích
[ ] Không có code trùng lặp (DRY)
```

---

## State & Data

```
[ ] Model mới có copyWith + == + toString
[ ] State chỉ thay đổi qua provider, không qua widget
[ ] Không có shared mutable state ngoài Riverpod
[ ] Hive box mới đã đăng ký trong hive_manager.dart
```

---

## Test

```
[ ] Có test cho logic mới
[ ] Test có tên mô tả rõ behavior
[ ] Không có test bị skip mà không có lý do
[ ] flutter test pass 100%
```

---

## Những dấu hiệu cần hỏi lại người dùng

Nếu AI thấy bất kỳ điều nào sau đây → dừng và hỏi trước khi tiến hành:

- Cần sửa > 3 file không liên quan trực tiếp
- Cần đổi tên public method/class
- Cần thêm dependency mới
- Logic phức tạp có thể hiểu theo 2 cách
- Không chắc behavior mong muốn là gì