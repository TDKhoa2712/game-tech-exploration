# Logging Policy — KBB

## Dùng AppLogger, không dùng print

```dart
// ❌ Cấm
print('game state: $state');
debugPrint('hp: $hp');

// ✅ Dùng
import '../../core/utils/logger.dart';
AppLogger.d('GameProvider: pickGesture called, phase=${state.phase}');
```

---

## 4 mức log và khi nào dùng

| Level | Method | Dùng khi |
|---|---|---|
| Debug | `AppLogger.d()` | Flow bình thường, dev only |
| Info | `AppLogger.i()` | Sự kiện quan trọng (game start, game over) |
| Warning | `AppLogger.w()` | Trạng thái bất thường nhưng không crash |
| Error | `AppLogger.e()` | Exception, lỗi cần xử lý |

---

## Chuẩn nội dung log

```dart
// Format: [ClassName]: [action] — [data ngắn gọn]
AppLogger.d('RpsEngine: determineResult — rock vs scissors → win');
AppLogger.i('GameProvider: game started — level=$level');
AppLogger.w('ItemService: generateItems called với level=0, dùng default=1');
AppLogger.e('HiveManager: openBox failed', error, stackTrace);
```

---

## Log bắt buộc ở đâu

```dart
// ✅ Game lifecycle
AppLogger.i('Game started — level=$level');
AppLogger.i('Round $round started');
AppLogger.i('Game over — playerWon=$playerWon, rounds=$round');

// ✅ Lỗi và exception
try {
  ...
} catch (e, st) {
  AppLogger.e('SettingsRepository: save failed', e, st);
}

// ✅ Warning khi state không hợp lệ
if (state.phase != GamePhase.pickGesture) {
  AppLogger.w('pickGesture called sai phase: ${state.phase}');
  return;
}
```

---

## Log KHÔNG được có

```dart
// ❌ Log data nhạy cảm
AppLogger.d('User token: $token');

// ❌ Log trong vòng lặp Flame (60fps = 60 log/s)
@override
void update(double dt) {
  AppLogger.d('update called');  // ❌ spam log
}

// ❌ Log quá nhiều chi tiết không cần thiết
AppLogger.d('Widget build called');  // ❌ vô nghĩa
```

---

## Production

- Debug log tự động tắt ở release build (cấu hình trong `logger.dart`)
- Chỉ `i`, `w`, `e` được giữ lại ở production nếu cần