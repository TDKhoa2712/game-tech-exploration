# F1 — Project Setup & Core

> **Tuần 1** · Ưu tiên: 🔴 Bắt buộc đầu tiên

---

## Mục tiêu

Dựng khung dự án đủ để các thành viên clone về và chạy được `flutter run` không lỗi.

---

## Files cần tạo

```
pubspec.yaml
main.dart
app.dart
lib/core/constants.dart
lib/core/theme.dart
lib/core/router.dart
lib/core/errors/app_exception.dart
lib/core/errors/failure.dart
lib/core/utils/extensions.dart
lib/core/utils/logger.dart
lib/shared/widgets/primary_button.dart
lib/shared/widgets/app_scaffold.dart
lib/shared/widgets/loading_overlay.dart
lib/shared/widgets/error_view.dart
```

---

## Vibe Code Prompt

```
Bạn là Flutter senior dev. Tạo khung dự án Flutter tên "keo_bua_bao" theo clean architecture.

### pubspec.yaml
Thêm dependencies:
- flame: ^1.18.0
- flutter_riverpod: ^2.5.1
- riverpod_annotation: ^2.3.5
- hive_flutter: ^1.1.0
- flame_audio: ^2.1.1
- go_router: ^13.2.0
- flutter_localizations (sdk: flutter)
- logger: ^2.3.0

dev_dependencies:
- build_runner
- riverpod_generator
- hive_generator
- flutter_test

### main.dart
- Khởi tạo WidgetsFlutterBinding
- Await Hive.initFlutter()
- Wrap app trong ProviderScope
- Gọi runApp(const KbbApp())

### app.dart — KbbApp widget
- MaterialApp.router với GoRouter từ core/router.dart
- Hỗ trợ Light/Dark theme từ core/theme.dart
- Locale mặc định: vi

### core/constants.dart
Định nghĩa constants:
- HP_START = 100
- TIMER_SECONDS = 5
- BASE_DAMAGE = 10
- TRAP_DAMAGE = 20
- MAX_ITEMS_PER_BOARD = 6

### core/theme.dart
- AppTheme.light() và AppTheme.dark()
- Color scheme lấy cảm hứng dân gian VN: đỏ son (#C0392B), vàng đất (#E67E22), xanh lá tre (#27AE60)
- TextTheme dùng Google Font Nunito

### core/router.dart
GoRouter với các routes placeholder (trả về Scaffold trống):
- '/' → MainMenuScreen
- '/mode-select' → ModeSelectScreen
- '/game' → GameScreen
- '/result' → ResultScreen
- '/settings' → SettingsScreen
- '/help' → HelpScreen

### core/errors/
- AppException: abstract class với message, code
- NetworkException, LocalStorageException extends AppException
- Failure: sealed class (ServerFailure, CacheFailure, UnknownFailure)

### core/utils/extensions.dart
- extension StringExt on String: capitalize(), isNullOrEmpty
- extension ListExt<T> on List<T>: randomElement() dùng Random

### core/utils/logger.dart
- Singleton AppLogger dùng package logger
- Methods: d(), i(), w(), e()

### shared/widgets/
- PrimaryButton: ElevatedButton với style theo AppTheme, nhận onPressed + label
- AppScaffold: Scaffold với background gradient nhẹ
- LoadingOverlay: Stack với CircularProgressIndicator overlay
- ErrorView: Column icon + message + retry button

Tất cả file phải có đúng import, không placeholder TODO, compile được ngay.
```

---

## Định nghĩa Done

- [ ] `flutter pub get` không lỗi
- [ ] `flutter run` ra màn hình trắng (route '/' placeholder)
- [ ] `flutter analyze` 0 error
- [ ] Shared widgets hiển thị đúng trong `flutter test`