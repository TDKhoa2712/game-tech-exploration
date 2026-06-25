# Quản lý Dependencies

## Packages đã được duyệt (Phase 1)

| Package | Version | Dùng cho |
|---|---|---|
| `flame` | ^1.18.0 | Game engine |
| `flutter_riverpod` | ^2.5.1 | State management |
| `riverpod_annotation` | ^2.3.5 | Code generation |
| `hive_flutter` | ^1.1.0 | Local storage |
| `flame_audio` | ^2.1.1 | SFX + BGM |
| `go_router` | ^13.2.0 | Navigation |
| `logger` | ^2.3.0 | Logging |

**dev_dependencies:**
`build_runner` · `riverpod_generator` · `hive_generator` · `flutter_lints`

---

## Quy tắc thêm package mới

AI **không được tự thêm** package vào `pubspec.yaml` mà không hỏi.

Trước khi đề xuất package mới, AI phải trả lời:

1. Package này giải quyết vấn đề gì cụ thể?
2. Có thể dùng package đã có hoặc viết tay không?
3. Package có đang được maintain? (pub.dev score ≥ 100, last update < 6 tháng)
4. License có phù hợp không? (MIT/BSD/Apache)
5. Ảnh hưởng đến app size bao nhiêu?

Format đề xuất:
```
Đề xuất thêm: [tên package] ^[version]
Lý do: [1 câu]
Thay thế cho: [giải pháp hiện tại]
Pub score: [X/140]
```

---

## Packages bị cấm

| Package | Lý do |
|---|---|
| `get` / `getx` | Xung đột với Riverpod |
| `provider` | Đã dùng Riverpod |
| `bloc` / `flutter_bloc` | Xung đột state management |
| Bất kỳ package analytics nào | Chưa có privacy policy |

---

## Khi update version

- Không tự update package — báo cáo để test thủ công trước
- Không update nhiều package cùng lúc
- Sau khi update: chạy `flutter pub get` + `flutter analyze` + toàn bộ unit test