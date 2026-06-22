# 05 · Hive — Lưu Trữ Dữ Liệu Local

> Hive là NoSQL database nhẹ, nhanh, thuần Dart — không phụ thuộc native. Lưu trữ dữ liệu dưới dạng key-value, hỗ trợ type-safe với adapter. Phù hợp cho game settings, scores, user data.

---

## 1. Setup

```yaml
# pubspec.yaml
dependencies:
  hive_flutter: ^1.1.0

dev_dependencies:
  hive_generator: ^2.0.1
  build_runner: ^2.4.9
```

---

## 2. Khởi Tạo Hive

```dart
// main.dart
import 'package:hive_flutter/hive_flutter.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // Khởi tạo Hive (tự tìm đường dẫn phù hợp trên từng nền tảng)
  await Hive.initFlutter();

  // Đăng ký adapter cho custom object
  Hive.registerAdapter(ScoreModelAdapter());
  Hive.registerAdapter(SettingsModelAdapter());
  Hive.registerAdapter(PlayerDataAdapter());

  // Mở các box cần dùng
  await Hive.openBox<ScoreModel>('scores');
  await Hive.openBox<SettingsModel>('settings');
  await Hive.openBox('misc'); // dynamic box — không type

  runApp(const ProviderScope(child: MyApp()));
}
```

---

## 3. Hive Type — Lưu Object Phức Tạp

Hive cần **adapter** để biết cách serialize/deserialize custom class. Dùng code generation để tự động tạo.

```dart
// score_model.dart
import 'package:hive/hive.dart';

part 'score_model.g.dart'; // file sẽ được generate

@HiveType(typeId: 0) // typeId phải duy nhất, không bao giờ thay đổi
class ScoreModel extends HiveObject {
  @HiveField(0)
  final String playerName;

  @HiveField(1)
  int score;

  @HiveField(2)
  final DateTime date;

  @HiveField(3)
  final int level;

  ScoreModel({
    required this.playerName,
    required this.score,
    required this.date,
    required this.level,
  });

  @override
  String toString() => '$playerName: $score điểm (Level $level)';
}

// settings_model.dart
@HiveType(typeId: 1)
class SettingsModel extends HiveObject {
  @HiveField(0)
  bool isMuted;

  @HiveField(1)
  double volume;

  @HiveField(2)
  bool isDarkMode;

  @HiveField(3)
  String playerName;

  SettingsModel({
    this.isMuted = false,
    this.volume = 0.8,
    this.isDarkMode = false,
    this.playerName = 'Player',
  });
}
```

```bash
# Chạy code generation
flutter pub run build_runner build --delete-conflicting-outputs
# Hoặc watch mode (tự động generate khi file thay đổi)
flutter pub run build_runner watch
```

---

## 4. CRUD Cơ Bản

```dart
// Lấy box (đã mở ở main)
final scoresBox = Hive.box<ScoreModel>('scores');
final settingsBox = Hive.box<SettingsModel>('settings');
final miscBox = Hive.box('misc');

// ===== CREATE / UPDATE =====

// add — thêm với key tự động (auto-increment integer)
await scoresBox.add(ScoreModel(
  playerName: 'Khoa',
  score: 1500,
  date: DateTime.now(),
  level: 5,
));

// put — thêm/cập nhật với key tùy chỉnh
await settingsBox.put('userSettings', SettingsModel(
  playerName: 'Khoa',
  volume: 0.7,
));

// Cập nhật object đã có (HiveObject)
final settings = settingsBox.get('userSettings')!;
settings.volume = 0.5;
await settings.save(); // save() tự biết key của nó

// Dynamic box
await miscBox.put('lastPlayedAt', DateTime.now().toIso8601String());
await miscBox.put('totalGamesPlayed', 42);

// ===== READ =====

// get bằng key
final settings = settingsBox.get('userSettings');
final totalGames = miscBox.get('totalGamesPlayed', defaultValue: 0);

// Lấy tất cả values
final allScores = scoresBox.values.toList();

// Lấy theo key
final score = scoresBox.getAt(0); // key = 0

// ===== DELETE =====

// Xóa bằng key
await scoresBox.delete('someKey');
await scoresBox.deleteAt(0); // xóa theo vị trí

// Xóa HiveObject trực tiếp
await score.delete();

// Xóa tất cả
await scoresBox.clear();
```

---

## 5. Query & Sort

Hive không có query engine như SQL, nhưng ta filter bằng Dart.

```dart
final box = Hive.box<ScoreModel>('scores');

// Top 10 điểm cao nhất
List<ScoreModel> getTopScores(int limit) {
  final all = box.values.toList();
  all.sort((a, b) => b.score.compareTo(a.score));
  return all.take(limit).toList();
}

// Lọc theo player
List<ScoreModel> getScoresByPlayer(String name) {
  return box.values.where((s) => s.playerName == name).toList();
}

// Điểm cao nhất của player
int getHighScore(String playerName) {
  final playerScores = getScoresByPlayer(playerName);
  if (playerScores.isEmpty) return 0;
  return playerScores.map((s) => s.score).reduce(max);
}

// Scores trong 7 ngày gần nhất
List<ScoreModel> getRecentScores() {
  final oneWeekAgo = DateTime.now().subtract(const Duration(days: 7));
  return box.values
    .where((s) => s.date.isAfter(oneWeekAgo))
    .toList()
    ..sort((a, b) => b.date.compareTo(a.date));
}
```

---

## 6. Watch — Lắng Nghe Thay Đổi

```dart
// Watch một key cụ thể
final subscription = settingsBox.watch(key: 'userSettings').listen((event) {
  print('Settings thay đổi: ${event.value}');
  print('Bị xóa: ${event.deleted}');
});

// Watch toàn bộ box
final boxSub = scoresBox.watch().listen((event) {
  print('Box scores thay đổi: key=${event.key}');
});

// Hủy lắng nghe
await subscription.cancel();
await boxSub.cancel();

// Tích hợp với Riverpod — tự động cập nhật UI khi Hive thay đổi
final scoresProvider = StreamProvider<List<ScoreModel>>((ref) {
  final box = Hive.box<ScoreModel>('scores');

  return box.watch().map((_) => box.values.toList()
    ..sort((a, b) => b.score.compareTo(a.score)));
});
```

---

## 7. HiveService — Wrapper Pattern

```dart
// Tập trung logic Hive vào một service — dễ test và thay thế
class HiveService {
  static const _scoresBox = 'scores';
  static const _settingsBox = 'settings';

  static Future<void> init() async {
    await Hive.initFlutter();
    Hive.registerAdapter(ScoreModelAdapter());
    Hive.registerAdapter(SettingsModelAdapter());
    await Future.wait([
      Hive.openBox<ScoreModel>(_scoresBox),
      Hive.openBox<SettingsModel>(_settingsBox),
    ]);
  }

  Box<ScoreModel> get scoresBox => Hive.box<ScoreModel>(_scoresBox);
  Box<SettingsModel> get settingsBox => Hive.box<SettingsModel>(_settingsBox);
}

// ScoreRepository dùng HiveService
class ScoreRepositoryImpl implements ScoreRepository {
  final HiveService _hive;
  ScoreRepositoryImpl(this._hive);

  @override
  Future<void> saveScore(ScoreModel score) async {
    await _hive.scoresBox.add(score);
  }

  @override
  List<ScoreModel> getTopScores(int limit) {
    return _hive.scoresBox.values.toList()
      ..sort((a, b) => b.score.compareTo(a.score))
      ..take(limit).toList();
  }

  @override
  Future<void> clearAll() async {
    await _hive.scoresBox.clear();
  }
}

// SettingsRepository
class SettingsRepositoryImpl implements SettingsRepository {
  final HiveService _hive;
  static const _key = 'settings';

  SettingsRepositoryImpl(this._hive);

  @override
  SettingsModel getSettings() {
    return _hive.settingsBox.get(_key) ?? SettingsModel();
  }

  @override
  Future<void> saveSettings(SettingsModel settings) async {
    await _hive.settingsBox.put(_key, settings);
  }

  @override
  Future<void> updateVolume(double volume) async {
    final settings = getSettings();
    settings.volume = volume;
    await settings.save();
  }

  @override
  Future<void> updateTheme(bool isDark) async {
    final settings = getSettings();
    settings.isDarkMode = isDark;
    await settings.save();
  }
}
```

---

## 8. Encrypted Box — Bảo Mật Dữ Liệu

```dart
import 'package:hive_flutter/hive_flutter.dart';
import 'dart:typed_data';

Future<void> openEncryptedBox() async {
  // Tạo hoặc lấy encryption key (lưu ở secure storage)
  const secureStorage = FlutterSecureStorage();
  String? keyString = await secureStorage.read(key: 'hive_key');

  Uint8List encryptionKey;
  if (keyString == null) {
    encryptionKey = Hive.generateSecureKey();
    await secureStorage.write(
      key: 'hive_key',
      value: base64Url.encode(encryptionKey),
    );
  } else {
    encryptionKey = base64Url.decode(keyString);
  }

  // Mở encrypted box
  await Hive.openBox<PlayerData>(
    'player_secure',
    encryptionCipher: HiveAesCipher(encryptionKey),
  );
}
```

---

## 9. TypeId Rules — Quan Trọng!

TypeId là định danh vĩnh viễn cho mỗi Hive type. Vi phạm các rule này gây mất dữ liệu hoặc crash.

```dart
// ✅ ĐÚNG — typeId tăng dần, không bao giờ thay đổi
@HiveType(typeId: 0) class ScoreModel {}       // mãi là 0
@HiveType(typeId: 1) class SettingsModel {}    // mãi là 1
@HiveType(typeId: 2) class PlayerData {}       // mãi là 2
@HiveType(typeId: 3) class LevelProgress {}   // mãi là 3

// ✅ ĐÚNG — thêm field mới dùng số mới
@HiveType(typeId: 0)
class ScoreModel {
  @HiveField(0) String playerName;   // field cũ — không đổi
  @HiveField(1) int score;           // field cũ — không đổi
  @HiveField(2) DateTime date;       // field cũ — không đổi
  @HiveField(3) String? region;      // field mới — thêm số mới
}

// ❌ SAI — đổi typeId hoặc HiveField index gây mất dữ liệu
// ❌ SAI — xóa HiveField cũ (để nguyên, không dùng nữa vẫn được)
// ❌ SAI — thay đổi kiểu dữ liệu của field
```

---

## 10. Lazy Box — Tối Ưu Bộ Nhớ

Dùng `LazyBox` khi box có nhiều dữ liệu lớn — chỉ load vào RAM khi cần.

```dart
// Mở lazy box
final lazyBox = await Hive.openLazyBox<ScoreModel>('bigScores');

// get phải await (vì đọc từ disk theo yêu cầu)
final score = await lazyBox.get('key123');

// Lấy tất cả (phải await)
final allKeys = lazyBox.keys.toList();
final allScores = await Future.wait(
  allKeys.map((key) => lazyBox.get(key)).toList(),
);
```

---

## Cheat Sheet Nhanh

| Thao tác | Code |
|----------|------|
| Mở box | `Hive.openBox<T>('name')` |
| Lấy box | `Hive.box<T>('name')` |
| Thêm auto key | `box.add(obj)` |
| Thêm custom key | `box.put('key', obj)` |
| Đọc | `box.get('key')` |
| Đọc với default | `box.get('key', defaultValue: ...)` |
| Tất cả values | `box.values.toList()` |
| Tất cả keys | `box.keys.toList()` |
| Xóa | `box.delete('key')` / `obj.delete()` |
| Cập nhật | `obj.field = value; obj.save()` |
| Xóa tất cả | `box.clear()` |
| Số lượng | `box.length` |
| Tồn tại? | `box.containsKey('key')` |

---

## Bài Tập Thực Hành

**Bài 1 — Model:** Tạo `PlayerData` với các field: name, totalGames, totalScore, bestScore, lastPlayedAt. Generate adapter.

**Bài 2 — CRUD:** Implement `PlayerRepository` với các method: `getPlayer()`, `savePlayer()`, `updateBestScore(int score)`.

**Bài 3 — Query:** Viết hàm lấy top 5 players theo `bestScore`, chỉ tính những ai đã chơi ít nhất 3 game.

**Bài 4 — Stream:** Tạo `StreamProvider` watch box settings, tự động cập nhật UI khi volume/theme thay đổi.

---

*Tiếp theo: [06 · flame_audio & Rive](06_audio_animation.md) — Âm Thanh và Animation*
