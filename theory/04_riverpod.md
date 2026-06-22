# 04 · Riverpod — Quản Lý State

> Riverpod là giải pháp quản lý state hiện đại nhất cho Flutter. An toàn hơn Provider, testable hơn setState, linh hoạt hơn Bloc. Toàn bộ state sống trong **providers** — không phụ thuộc BuildContext.

---

## 1. Setup

```yaml
# pubspec.yaml
dependencies:
  flutter_riverpod: ^2.5.1
  riverpod_annotation: ^2.3.5

dev_dependencies:
  riverpod_generator: ^2.4.0
  build_runner: ^2.4.9
```

```dart
// main.dart — bọc toàn bộ app trong ProviderScope
void main() {
  runApp(const ProviderScope(child: MyApp()));
}
```

---

## 2. Provider Cơ Bản

### Provider — giá trị chỉ đọc

```dart
// Khai báo
final greetingProvider = Provider<String>((ref) {
  return 'Xin chào Khoa!';
});

// Đọc trong widget (dùng ConsumerWidget)
class GreetingWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final greeting = ref.watch(greetingProvider);
    return Text(greeting);
  }
}
```

### StateProvider — state đơn giản

```dart
// Khai báo
final counterProvider = StateProvider<int>((ref) => 0);
final themeProvider = StateProvider<ThemeMode>((ref) => ThemeMode.system);

// Đọc & ghi
class CounterWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);

    return Column(
      children: [
        Text('$count'),
        ElevatedButton(
          onPressed: () => ref.read(counterProvider.notifier).state++,
          child: const Text('Tăng'),
        ),
        ElevatedButton(
          onPressed: () => ref.read(counterProvider.notifier).state = 0,
          child: const Text('Reset'),
        ),
      ],
    );
  }
}
```

---

## 3. StateNotifierProvider — State Phức Tạp

Đây là cách phổ biến nhất để quản lý state có logic.

```dart
// Model
class GameState {
  final int score;
  final int lives;
  final GameStatus status;
  final int level;

  const GameState({
    this.score = 0,
    this.lives = 3,
    this.status = GameStatus.idle,
    this.level = 1,
  });

  GameState copyWith({
    int? score,
    int? lives,
    GameStatus? status,
    int? level,
  }) {
    return GameState(
      score: score ?? this.score,
      lives: lives ?? this.lives,
      status: status ?? this.status,
      level: level ?? this.level,
    );
  }
}

enum GameStatus { idle, playing, paused, gameOver }

// Notifier — chứa toàn bộ logic
class GameNotifier extends StateNotifier<GameState> {
  GameNotifier() : super(const GameState());

  void startGame() {
    state = state.copyWith(
      status: GameStatus.playing,
      score: 0,
      lives: 3,
    );
  }

  void addScore(int points) {
    if (state.status != GameStatus.playing) return;
    state = state.copyWith(score: state.score + points);

    // Lên level mỗi 100 điểm
    if (state.score >= state.level * 100) {
      state = state.copyWith(level: state.level + 1);
    }
  }

  void loseLife() {
    if (state.lives <= 1) {
      state = state.copyWith(lives: 0, status: GameStatus.gameOver);
    } else {
      state = state.copyWith(lives: state.lives - 1);
    }
  }

  void togglePause() {
    if (state.status == GameStatus.playing) {
      state = state.copyWith(status: GameStatus.paused);
    } else if (state.status == GameStatus.paused) {
      state = state.copyWith(status: GameStatus.playing);
    }
  }

  void resetGame() {
    state = const GameState();
  }
}

// Provider
final gameProvider = StateNotifierProvider<GameNotifier, GameState>((ref) {
  return GameNotifier();
});

// Sử dụng trong widget
class GameHud extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // watch — rebuild khi state thay đổi
    final gameState = ref.watch(gameProvider);

    // select — chỉ rebuild khi score thay đổi (tối ưu hơn)
    final score = ref.watch(gameProvider.select((s) => s.score));
    final lives = ref.watch(gameProvider.select((s) => s.lives));

    return Row(
      children: [
        Text('Điểm: $score'),
        Text('Mạng: $lives'),
        ElevatedButton(
          // read — không rebuild, chỉ đọc/gọi method
          onPressed: () => ref.read(gameProvider.notifier).togglePause(),
          child: const Text('Pause'),
        ),
      ],
    );
  }
}
```

---

## 4. FutureProvider & AsyncNotifierProvider

```dart
// FutureProvider — fetch dữ liệu một lần
final topScoresProvider = FutureProvider<List<Score>>((ref) async {
  final repo = ref.read(scoreRepositoryProvider);
  return repo.getTopScores(10);
});

// Sử dụng
class LeaderboardScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final scoresAsync = ref.watch(topScoresProvider);

    return scoresAsync.when(
      loading: () => const Center(child: CircularProgressIndicator()),
      error: (error, stack) => Center(child: Text('Lỗi: $error')),
      data: (scores) => ListView.builder(
        itemCount: scores.length,
        itemBuilder: (_, i) => ScoreTile(score: scores[i]),
      ),
    );
  }
}

// AsyncNotifierProvider — async + có logic
class ScoreNotifier extends AsyncNotifier<List<Score>> {
  @override
  Future<List<Score>> build() async {
    return ref.read(scoreRepositoryProvider).getTopScores(10);
  }

  Future<void> addScore(Score score) async {
    await ref.read(scoreRepositoryProvider).saveScore(score);
    // Reload danh sách
    ref.invalidateSelf();
  }
}

final scoreNotifierProvider = AsyncNotifierProvider<ScoreNotifier, List<Score>>(
  ScoreNotifier.new,
);
```

---

## 5. Provider Dependencies — Provider Phụ Thuộc Nhau

```dart
// Repository provider (phụ thuộc vào HiveService)
final hiveServiceProvider = Provider<HiveService>((ref) => HiveService());

final scoreRepositoryProvider = Provider<ScoreRepository>((ref) {
  final hive = ref.read(hiveServiceProvider);
  return ScoreRepositoryImpl(hive);
});

final settingsRepositoryProvider = Provider<SettingsRepository>((ref) {
  final hive = ref.read(hiveServiceProvider);
  return SettingsRepositoryImpl(hive);
});

// Provider phụ thuộc vào provider khác
final gameProvider = StateNotifierProvider<GameNotifier, GameState>((ref) {
  final scoreRepo = ref.read(scoreRepositoryProvider);
  return GameNotifier(scoreRepo);
});

// Provider phụ thuộc vào state của provider khác
final isHighScoreProvider = Provider<bool>((ref) {
  final currentScore = ref.watch(gameProvider.select((s) => s.score));
  final topScores = ref.watch(topScoresProvider).valueOrNull ?? [];
  return topScores.isEmpty || currentScore > (topScores.first.value);
});
```

---

## 6. Family — Provider Có Tham Số

```dart
// Provider nhận tham số
final playerDetailProvider = FutureProvider.family<Player, int>((ref, playerId) async {
  return ref.read(playerRepositoryProvider).getPlayer(playerId);
});

// Sử dụng
final player = ref.watch(playerDetailProvider(42));

// StateNotifier.family
final levelProvider = StateNotifierProvider.family<LevelNotifier, LevelState, int>(
  (ref, levelId) => LevelNotifier(levelId),
);

ref.watch(levelProvider(3)); // level 3
```

---

## 7. AutoDispose — Tự Động Dọn Dẹp

```dart
// Provider tự hủy khi không có widget nào watch
final searchProvider = StateProvider.autoDispose<String>((ref) => '');

// FutureProvider.autoDispose — hủy khi navigate khỏi màn hình
final userDataProvider = FutureProvider.autoDispose<User>((ref) async {
  return fetchUser();
});

// Giữ sống một khoảng thời gian
final cachedDataProvider = FutureProvider.autoDispose<Data>((ref) async {
  ref.keepAlive(); // giữ sống mãi dù không có widget watch (cache)
  return fetchData();
});

// Cleanup khi dispose
final timerProvider = StateProvider.autoDispose<int>((ref) {
  final timer = Timer.periodic(Duration(seconds: 1), (t) {
    // cập nhật timer
  });

  ref.onDispose(() {
    timer.cancel(); // dọn dẹp khi provider bị hủy
  });

  return 0;
});
```

---

## 8. ConsumerWidget vs Consumer

```dart
// ConsumerWidget — widget toàn bộ là consumer (phổ biến nhất)
class ScoreDisplay extends ConsumerWidget {
  const ScoreDisplay({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final score = ref.watch(gameProvider.select((s) => s.score));
    return Text('$score', style: const TextStyle(fontSize: 32));
  }
}

// ConsumerStatefulWidget — khi cần lifecycle (initState, dispose,...)
class GameScreen extends ConsumerStatefulWidget {
  @override
  ConsumerState<GameScreen> createState() => _GameScreenState();
}

class _GameScreenState extends ConsumerState<GameScreen> {
  late MyGame _game;

  @override
  void initState() {
    super.initState();
    _game = MyGame(ref: ref);
    // Có thể dùng ref.read() ở đây
    ref.read(gameProvider.notifier).startGame();
  }

  @override
  Widget build(BuildContext context) {
    // Dùng ref.watch() trong build
    final isPaused = ref.watch(gameProvider.select((s) => s.status == GameStatus.paused));
    return GameWidget(game: _game);
  }
}

// Consumer — chỉ rebuild một phần widget tree (tối ưu hóa)
class HeavyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        ExpensiveWidget(), // không rebuild
        Consumer(          // chỉ phần này rebuild
          builder: (context, ref, child) {
            final score = ref.watch(gameProvider.select((s) => s.score));
            return Text('Điểm: $score');
          },
        ),
      ],
    );
  }
}
```

---

## 9. ref.listen — Side Effects

```dart
// Lắng nghe thay đổi và chạy side effect (không rebuild)
class GameScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Hiện dialog khi game over
    ref.listen<GameStatus>(
      gameProvider.select((s) => s.status),
      (previous, next) {
        if (next == GameStatus.gameOver) {
          showDialog(
            context: context,
            builder: (_) => const GameOverDialog(),
          );
        }
      },
    );

    // Hiện snackbar khi lên level
    ref.listen<int>(
      gameProvider.select((s) => s.level),
      (previous, next) {
        if (previous != null && next > previous) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(content: Text('Level $next!')),
          );
        }
      },
    );

    return GameWidget(game: myGame);
  }
}
```

---

## 10. Testing với Riverpod

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  group('GameNotifier', () {
    late ProviderContainer container;

    setUp(() {
      container = ProviderContainer();
    });

    tearDown(() {
      container.dispose();
    });

    test('trạng thái ban đầu', () {
      final state = container.read(gameProvider);
      expect(state.score, 0);
      expect(state.lives, 3);
      expect(state.status, GameStatus.idle);
    });

    test('startGame', () {
      container.read(gameProvider.notifier).startGame();
      expect(container.read(gameProvider).status, GameStatus.playing);
    });

    test('addScore cộng điểm', () {
      container.read(gameProvider.notifier).startGame();
      container.read(gameProvider.notifier).addScore(50);
      expect(container.read(gameProvider).score, 50);
    });

    test('loseLife → gameOver khi hết mạng', () {
      container.read(gameProvider.notifier).startGame();
      container.read(gameProvider.notifier).loseLife();
      container.read(gameProvider.notifier).loseLife();
      container.read(gameProvider.notifier).loseLife();
      expect(container.read(gameProvider).status, GameStatus.gameOver);
    });
  });

  // Test với mock dependency
  test('với mock repository', () {
    final container = ProviderContainer(
      overrides: [
        scoreRepositoryProvider.overrideWithValue(MockScoreRepository()),
      ],
    );
    // ... test logic
    container.dispose();
  });
}
```

---

## Bài Tập Thực Hành

**Bài 1 — StateProvider:** Tạo provider quản lý `ThemeMode`. Thêm nút toggle theme trong Settings screen.

**Bài 2 — StateNotifier:** Tạo `AudioNotifier` với state `{isMuted: bool, volume: double}`. Implement `toggleMute()` và `setVolume()`.

**Bài 3 — FutureProvider:** Load danh sách top 10 điểm từ Hive và hiển thị với loading/error state.

**Bài 4 — ref.listen:** Lắng nghe `gameProvider` để tự động pause/resume audio khi game paused/resumed.

---

*Tiếp theo: [05 · Hive](05_hive.md) — Lưu trữ dữ liệu local*
