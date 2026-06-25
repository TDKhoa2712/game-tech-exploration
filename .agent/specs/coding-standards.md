# QUY ƯỚC CODE & TIÊU CHUẨN LẬP TRÌNH — KÉO BÚA BAO

## 1. Quy tắc đặt tên (Naming Conventions)
Áp dụng chuẩn [Dart Official Style Guide](https://dart.dev/guides/language/effective-dart/style).

| Loại | Quy ước | Ví dụ |
| :--- | :--- | :--- |
| **Thư mục (Folders)** | `snake_case` (chữ thường + gạch dưới) | `game_engine/`, `player_profile/` |
| **File (Dart)** | `snake_case` | `game_state.dart`, `hp_bar.dart` |
| **Class / Enum** | `UpperCamelCase` | `GameEngine`, `PlayerStatus`, `ItemType` |
| **Biến / Hàm / Tham số** | `camelCase` | `playerHealth`, `calculateDamage()`, `isWinner` |
| **Hằng số (Constants)** | `camelCase` (hoặc `SCREAMING_SNAKE` nếu là top-level) | `const baseDamage = 10;` hoặc `const int MAX_HP = 100;` |
| **Private members** | Bắt đầu bằng `_` | `_updateBoard()`, `_currentRound` |

## 2. Cấu trúc thư mục (Folder Structure)
Tuân thủ nghiêm ngặt cấu trúc đã định nghĩa trong `architecture.md`:

- `lib/core/`: Code dùng chung (utils, constants, routes, themes).
- `lib/data/`: Models, Hive Adapters, Remote Services.
- `lib/domain/`: Business logic thuần Dart (không phụ thuộc Flutter/UI).
- `lib/presentation/`: UI (Screens, Widgets, Providers).
- `lib/game/`: Flame Game Loop, Components, Overlays.

**Nguyên tắc:** Code ở `domain` **KHÔNG ĐƯỢC** import `flutter` hay `flame`. Chỉ nhận đầu vào là các class đơn giản và trả về kết quả (Use Cases).

## 3. Quản lý State (Riverpod)
- Sử dụng `@riverpod` annotation (code generation) nếu có thể để giảm boilerplate.
- Mỗi StateNotifier chỉ nên quản lý **duy nhất 1 phần state** (ví dụ: `gameProvider` không quản lý `settings`).
- Luôn sử dụng `ref.watch` trong ConsumerWidget để UI tự động cập nhật.
- **AsyncValue:** Sử dụng `AsyncValue` để xử lý loading/error/data khi gọi API (Phase 3).

## 4. Git & Commit (Conventional Commits)
Áp dụng chuẩn [Conventional Commits](https://www.conventionalcommits.org/) để dễ dàng sinh CHANGELOG và quản lý version.

- `feat:` Thêm tính năng mới (ví dụ: `feat: add HP bar widget`)
- `fix:` Sửa lỗi (ví dụ: `fix: timer not resetting after round`)
- `chore:` Thay đổi công cụ, cấu hình, CI/CD (ví dụ: `chore: update flutter sdk`)
- `docs:` Thay đổi tài liệu (ví dụ: `docs: update architecture.md`)
- `test:` Thêm hoặc sửa test (ví dụ: `test: add unit test for damage calculation`)

## 5. Linting & Formatting
- Sử dụng `flutter_lints` (hoặc `very_good_analysis`) làm lint rule mặc định.
- Bắt buộc chạy `dart format .` trước mỗi commit để code được định dạng đều.
- CI/CD sẽ tự động kiểm tra lint và test trước khi cho phép merge Pull Request.

## 6. Xử lý lỗi (Error Handling)
- Sử dụng `try-catch` khi gọi API hoặc đọc file.
- Ghi log lỗi rõ ràng (có thể dùng `debugPrint` trong quá trình phát triển).
- Không bỏ qua exception. Luôn có fallback UI (ví dụ: `CircularProgressIndicator`, màn hình lỗi).

## 7. Bảo mật (Security)
- **Không hardcode key/secret** trong code. Sử dụng file `.env` và `--dart-define` hoặc `flutter_config`.
- Tất cả logic damage, thắng thua **phải được tính lại trên Server** ở Phase 3. Client chỉ gửi lựa chọn của người chơi.