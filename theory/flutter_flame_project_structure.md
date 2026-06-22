# 📁 Cấu Trúc Dự Án Flutter + Flame

> **Tech Stack:** Dart · Flutter · Flame · Riverpod · Hive · flame_audio · Rive · Light/Dark Theme · GitHub Actions

---

## Tổng Quan Cấu Trúc Thư Mục

```
my_game/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── cd.yml
├── android/
├── ios/
├── web/
├── assets/
│   ├── audio/
│   │   ├── bgm/
│   │   │   ├── main_theme.mp3
│   │   │   └── game_theme.mp3
│   │   └── sfx/
│   │       ├── click.wav
│   │       ├── jump.wav
│   │       └── game_over.wav
│   ├── images/
│   │   ├── sprites/
│   │   │   ├── player.png
│   │   │   └── enemies/
│   │   ├── backgrounds/
│   │   └── ui/
│   │       ├── button.png
│   │       └── icons/
│   ├── animations/
│   │   └── rive/
│   │       ├── character.riv
│   │       └── ui_effects.riv
│   └── fonts/
│       └── GameFont.ttf
├── lib/
│   ├── main.dart
│   ├── app.dart
│   ├── core/
│   │   ├── constants/
│   │   │   ├── app_constants.dart
│   │   │   ├── asset_constants.dart
│   │   │   └── game_constants.dart
│   │   ├── theme/
│   │   │   ├── app_theme.dart
│   │   │   ├── light_theme.dart
│   │   │   └── dark_theme.dart
│   │   ├── router/
│   │   │   └── app_router.dart
│   │   └── utils/
│   │       ├── extensions.dart
│   │       └── helpers.dart
│   ├── data/
│   │   ├── models/
│   │   │   ├── player_model.dart
│   │   │   ├── score_model.dart
│   │   │   └── settings_model.dart
│   │   ├── repositories/
│   │   │   ├── score_repository.dart
│   │   │   └── settings_repository.dart
│   │   └── datasources/
│   │       └── local/
│   │           ├── hive_service.dart
│   │           └── hive_boxes.dart
│   ├── providers/
│   │   ├── game_provider.dart
│   │   ├── score_provider.dart
│   │   ├── settings_provider.dart
│   │   ├── audio_provider.dart
│   │   └── theme_provider.dart
│   ├── game/
│   │   ├── my_game.dart
│   │   ├── components/
│   │   │   ├── player/
│   │   │   │   ├── player_component.dart
│   │   │   │   └── player_animator.dart
│   │   │   ├── enemies/
│   │   │   │   ├── base_enemy.dart
│   │   │   │   └── enemy_spawner.dart
│   │   │   ├── world/
│   │   │   │   ├── background_component.dart
│   │   │   │   └── map_component.dart
│   │   │   └── ui/
│   │   │       ├── hud_component.dart
│   │   │       └── score_display.dart
│   │   ├── systems/
│   │   │   ├── collision_system.dart
│   │   │   ├── physics_system.dart
│   │   │   └── spawn_system.dart
│   │   └── overlays/
│   │       ├── pause_overlay.dart
│   │       ├── game_over_overlay.dart
│   │       └── level_complete_overlay.dart
│   ├── screens/
│   │   ├── splash/
│   │   │   └── splash_screen.dart
│   │   ├── home/
│   │   │   ├── home_screen.dart
│   │   │   └── home_widgets/
│   │   │       ├── play_button.dart
│   │   │       └── leaderboard_tile.dart
│   │   ├── game/
│   │   │   └── game_screen.dart
│   │   ├── settings/
│   │   │   └── settings_screen.dart
│   │   └── leaderboard/
│   │       └── leaderboard_screen.dart
│   └── services/
│       ├── audio_service.dart
│       └── storage_service.dart
├── test/
│   ├── unit/
│   │   ├── providers/
│   │   │   ├── game_provider_test.dart
│   │   │   └── score_provider_test.dart
│   │   └── repositories/
│   │       └── score_repository_test.dart
│   ├── widget/
│   │   ├── home_screen_test.dart
│   │   └── settings_screen_test.dart
│   └── game/
│       ├── player_component_test.dart
│       └── collision_system_test.dart
├── pubspec.yaml
└── README.md
```

---

## Chi Tiết Từng Thư Mục

### `.github/workflows/`

Chứa các file cấu hình CI/CD cho GitHub Actions.

| File | Mục đích |
|------|----------|
| `ci.yml` | Chạy tests, lint, build check khi có PR hoặc push |
| `cd.yml` | Build & deploy release lên store hoặc artifact khi merge vào main |

```yaml
# Ví dụ ci.yml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
      - run: flutter pub get
      - run: flutter analyze
      - run: flutter test
```

---

### `assets/`

Tổ chức toàn bộ tài nguyên tĩnh của game.

| Thư mục | Nội dung |
|---------|----------|
| `audio/bgm/` | Nhạc nền (Background Music) — dùng với `flame_audio` |
| `audio/sfx/` | Hiệu ứng âm thanh (Sound Effects) |
| `images/sprites/` | Sprite sheet cho nhân vật, kẻ thù |
| `images/backgrounds/` | Hình nền màn chơi |
| `images/ui/` | Ảnh giao diện: nút bấm, icon, frame |
| `animations/rive/` | File `.riv` cho animation phức tạp với Rive |
| `fonts/` | Font chữ tùy chỉnh |

> **Lưu ý:** Khai báo đầy đủ trong `pubspec.yaml` dưới section `flutter: assets:`.

---

### `lib/core/`

Tầng nền tảng dùng chung toàn ứng dụng, không phụ thuộc business logic.

#### `constants/`

```dart
// app_constants.dart
class AppConstants {
  static const String appName = 'My Game';
  static const int targetFPS = 60;
}

// asset_constants.dart
class AssetConstants {
  static const String playerSprite = 'images/sprites/player.png';
  static const String mainThemeBGM = 'audio/bgm/main_theme.mp3';
  static const String characterRive = 'animations/rive/character.riv';
}

// game_constants.dart
class GameConstants {
  static const double gravity = 9.8;
  static const int maxLives = 3;
  static const double playerSpeed = 200.0;
}
```

#### `theme/`

Quản lý Light/Dark Mode toàn app.

```dart
// app_theme.dart
class AppTheme {
  static ThemeData get light => LightTheme.data;
  static ThemeData get dark => DarkTheme.data;
}

// light_theme.dart
class LightTheme {
  static ThemeData get data => ThemeData(
    brightness: Brightness.light,
    primaryColor: Colors.indigo,
    // ...
  );
}
```

#### `router/`

Cấu hình điều hướng màn hình (dùng `go_router` hoặc `Navigator`).

```dart
// app_router.dart
class AppRouter {
  static const splash = '/';
  static const home = '/home';
  static const game = '/game';
  static const settings = '/settings';
  static const leaderboard = '/leaderboard';
}
```

---

### `lib/data/`

Tầng dữ liệu: định nghĩa model, repository, và datasource (Hive).

#### `models/`

Các class model được đánh dấu `@HiveType` để Hive có thể serialize.

```dart
// score_model.dart
import 'package:hive/hive.dart';

part 'score_model.g.dart';

@HiveType(typeId: 0)
class ScoreModel extends HiveObject {
  @HiveField(0)
  final String playerName;

  @HiveField(1)
  final int score;

  @HiveField(2)
  final DateTime date;

  ScoreModel({
    required this.playerName,
    required this.score,
    required this.date,
  });
}
```

#### `repositories/`

Abstraction layer giữa business logic và datasource.

```dart
// score_repository.dart
abstract class ScoreRepository {
  Future<List<ScoreModel>> getTopScores(int limit);
  Future<void> saveScore(ScoreModel score);
}

class ScoreRepositoryImpl implements ScoreRepository {
  final HiveService _hiveService;
  ScoreRepositoryImpl(this._hiveService);

  @override
  Future<List<ScoreModel>> getTopScores(int limit) async {
    final box = _hiveService.getBox<ScoreModel>('scores');
    return box.values.toList()
      ..sort((a, b) => b.score.compareTo(a.score))
      ..take(limit).toList();
  }

  @override
  Future<void> saveScore(ScoreModel score) async {
    final box = _hiveService.getBox<ScoreModel>('scores');
    await box.add(score);
  }
}
```

#### `datasources/local/`

Khởi tạo và quản lý Hive boxes.

```dart
// hive_service.dart
class HiveService {
  static Future<void> init() async {
    await Hive.initFlutter();
    Hive.registerAdapter(ScoreModelAdapter());
    Hive.registerAdapter(SettingsModelAdapter());
  }

  Box<T> getBox<T>(String name) => Hive.box<T>(name);
}

// hive_boxes.dart
class HiveBoxes {
  static const String scores = 'scores';
  static const String settings = 'settings';
  static const String playerData = 'player_data';
}
```

---

### `lib/providers/`

Quản lý state toàn ứng dụng với **Riverpod**.

```dart
// theme_provider.dart
final themeProvider = StateNotifierProvider<ThemeNotifier, ThemeMode>((ref) {
  return ThemeNotifier(ref.watch(settingsRepositoryProvider));
});

class ThemeNotifier extends StateNotifier<ThemeMode> {
  final SettingsRepository _repo;
  ThemeNotifier(this._repo) : super(ThemeMode.system) {
    _loadTheme();
  }

  void _loadTheme() {
    final saved = _repo.getTheme();
    state = saved ?? ThemeMode.system;
  }

  void toggleTheme() {
    state = state == ThemeMode.light ? ThemeMode.dark : ThemeMode.light;
    _repo.saveTheme(state);
  }
}

// audio_provider.dart
final audioProvider = Provider<AudioService>((ref) => AudioService());

// game_provider.dart
final gameProvider = StateNotifierProvider<GameNotifier, GameState>((ref) {
  return GameNotifier();
});

enum GameStatus { idle, playing, paused, gameOver }

class GameState {
  final GameStatus status;
  final int score;
  final int lives;

  const GameState({
    this.status = GameStatus.idle,
    this.score = 0,
    this.lives = 3,
  });

  GameState copyWith({GameStatus? status, int? score, int? lives}) {
    return GameState(
      status: status ?? this.status,
      score: score ?? this.score,
      lives: lives ?? this.lives,
    );
  }
}
```

---

### `lib/game/`

**Core game logic** — toàn bộ code liên quan đến Flame Engine.

#### `my_game.dart` — Game chính

```dart
class MyGame extends FlameGame
    with HasCollisionDetection, HasKeyboardHandlerComponents {
  late final PlayerComponent player;

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    camera.viewfinder.anchor = Anchor.topLeft;
    add(BackgroundComponent());
    player = PlayerComponent();
    add(player);
    add(EnemySpawner());
    overlays.add('Hud');
  }
}
```

#### `components/`

Mỗi thực thể trong game là một `Component`.

```dart
// player/player_component.dart
class PlayerComponent extends SpriteAnimationComponent
    with CollisionCallbacks, HasGameRef<MyGame> {
  
  @override
  Future<void> onLoad() async {
    animation = await gameRef.loadSpriteAnimation(
      AssetConstants.playerSprite,
      SpriteAnimationData.sequenced(amount: 8, stepTime: 0.1, textureSize: Vector2(64, 64)),
    );
    add(RectangleHitbox());
  }

  @override
  void onCollisionStart(Set<Vector2> points, PositionComponent other) {
    if (other is EnemyComponent) {
      // xử lý va chạm
    }
  }
}
```

#### `systems/`

Tách các logic hệ thống ra khỏi component để dễ test và tái sử dụng.

| File | Chức năng |
|------|-----------|
| `collision_system.dart` | Xử lý logic va chạm tập trung |
| `physics_system.dart` | Gravity, velocity, movement |
| `spawn_system.dart` | Quy tắc sinh ra enemy/item |

#### `overlays/`

Các lớp UI hiển thị **đè lên** game canvas (dùng Flutter widget, không phải Flame component).

```dart
// pause_overlay.dart
class PauseOverlay extends ConsumerWidget {
  static const id = 'Pause';

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return Center(
      child: Column(
        children: [
          Text('PAUSED'),
          ElevatedButton(
            onPressed: () => ref.read(gameProvider.notifier).resume(),
            child: Text('Resume'),
          ),
        ],
      ),
    );
  }
}
```

---

### `lib/services/`

Các service độc lập, không thuộc tầng data hay UI.

```dart
// audio_service.dart — wrapper cho flame_audio
class AudioService {
  bool _isMuted = false;

  Future<void> playBGM(String filename) async {
    if (_isMuted) return;
    await FlameAudio.bgm.play(filename, volume: 0.5);
  }

  Future<void> playSFX(String filename) async {
    if (_isMuted) return;
    await FlameAudio.play(filename);
  }

  void toggleMute() {
    _isMuted = !_isMuted;
    if (_isMuted) {
      FlameAudio.bgm.pause();
    } else {
      FlameAudio.bgm.resume();
    }
  }
}
```

---

### `lib/screens/`

Các màn hình Flutter thuần (ngoài game canvas).

| Màn hình | Mô tả |
|----------|-------|
| `splash_screen.dart` | Load assets, init Hive, chuyển hướng |
| `home_screen.dart` | Menu chính: Play, Settings, Leaderboard |
| `game_screen.dart` | Wrapper chứa `GameWidget` của Flame |
| `settings_screen.dart` | Điều chỉnh âm thanh, theme, tên player |
| `leaderboard_screen.dart` | Hiển thị bảng điểm từ Hive |

```dart
// game_screen.dart
class GameScreen extends ConsumerStatefulWidget {
  @override
  ConsumerState<GameScreen> createState() => _GameScreenState();
}

class _GameScreenState extends ConsumerState<GameScreen> {
  late final MyGame _game;

  @override
  void initState() {
    super.initState();
    _game = MyGame();
  }

  @override
  Widget build(BuildContext context) {
    return GameWidget(
      game: _game,
      overlayBuilderMap: {
        'Hud': (context, game) => HudOverlay(game: game as MyGame),
        PauseOverlay.id: (context, game) => PauseOverlay(),
        'GameOver': (context, game) => GameOverOverlay(game: game as MyGame),
      },
    );
  }
}
```

---

### `lib/main.dart` & `lib/app.dart`

```dart
// main.dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await HiveService.init();
  await Hive.openBox<ScoreModel>(HiveBoxes.scores);
  await Hive.openBox<SettingsModel>(HiveBoxes.settings);
  runApp(const ProviderScope(child: MyApp()));
}

// app.dart
class MyApp extends ConsumerWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final themeMode = ref.watch(themeProvider);
    return MaterialApp(
      title: AppConstants.appName,
      theme: AppTheme.light,
      darkTheme: AppTheme.dark,
      themeMode: themeMode,
      initialRoute: AppRouter.splash,
      routes: {
        AppRouter.splash: (_) => const SplashScreen(),
        AppRouter.home: (_) => const HomeScreen(),
        AppRouter.game: (_) => const GameScreen(),
        AppRouter.settings: (_) => const SettingsScreen(),
        AppRouter.leaderboard: (_) => const LeaderboardScreen(),
      },
    );
  }
}
```

---

### `test/`

Cấu trúc test song song với `lib/`.

| Thư mục | Loại test |
|---------|-----------|
| `unit/providers/` | Test Riverpod providers (StateNotifier, logic) |
| `unit/repositories/` | Test repository với Hive mock |
| `widget/` | Test Flutter widget (render, tap, state) |
| `game/` | Test Flame component & system logic |

---

## `pubspec.yaml` — Dependencies

```yaml
name: my_game
description: Flutter Flame Game

environment:
  sdk: '>=3.0.0 <4.0.0'
  flutter: '>=3.10.0'

dependencies:
  flutter:
    sdk: flutter

  # Core Game Engine
  flame: ^1.17.0
  flame_audio: ^2.1.1

  # State Management
  flutter_riverpod: ^2.5.1
  riverpod_annotation: ^2.3.5

  # Local Storage
  hive_flutter: ^1.1.0

  # Animation
  rive: ^0.13.4

  # Navigation
  go_router: ^14.0.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  hive_generator: ^2.0.1
  build_runner: ^2.4.9
  riverpod_generator: ^2.4.0
  flutter_lints: ^4.0.0

flutter:
  uses-material-design: true
  assets:
    - assets/images/sprites/
    - assets/images/backgrounds/
    - assets/images/ui/
    - assets/animations/rive/
    - assets/audio/bgm/
    - assets/audio/sfx/
  fonts:
    - family: GameFont
      fonts:
        - asset: assets/fonts/GameFont.ttf
```

---

## Luồng Dữ Liệu (Data Flow)

```
UI (Screens / Overlays)
       │
       ▼
  Providers (Riverpod)
  ┌────────────────────────────┐
  │ gameProvider               │
  │ scoreProvider              │
  │ themeProvider              │
  │ audioProvider              │
  └────────────────────────────┘
       │                   │
       ▼                   ▼
  Repositories          Services
  (ScoreRepo,        (AudioService)
   SettingsRepo)         │
       │                  ▼
       ▼             flame_audio
  HiveService
  (Local Storage)
```

---

## Quy Ước Đặt Tên

| Loại | Convention | Ví dụ |
|------|-----------|-------|
| File | `snake_case.dart` | `player_component.dart` |
| Class | `PascalCase` | `PlayerComponent` |
| Provider | `camelCase + Provider` | `scoreProvider` |
| Hive TypeId | Tăng dần từ 0 | `@HiveType(typeId: 0)` |
| Asset path | Khai báo trong `AssetConstants` | `AssetConstants.playerSprite` |
| Overlay id | `static const String id` trong class | `PauseOverlay.id` |

---

## CI/CD Pipeline (GitHub Actions)

```
Push / PR
   │
   ├─► CI Job
   │     ├── flutter pub get
   │     ├── dart analyze
   │     ├── flutter test --coverage
   │     └── flutter build apk --debug (kiểm tra build)
   │
   └─► CD Job (chỉ chạy khi merge vào main)
         ├── flutter build apk --release
         ├── flutter build ios --release (macOS runner)
         └── Upload artifact / Deploy to store
```

---

*Tài liệu này mô tả cấu trúc chuẩn cho dự án Flutter + Flame. Có thể điều chỉnh tùy theo quy mô và yêu cầu cụ thể của game.*
