# Coding Standards — KBB

## Đặt tên

### Files
```
# Feature screens
game_screen.dart          ✅
gameScreen.dart           ❌
GameScreen.dart           ❌

# Widgets
hp_bar_widget.dart        ✅
gesture_picker_widget.dart ✅

# Providers
game_provider.dart        ✅

# Models
game_state.dart           ✅
```

### Classes
```dart
class GameScreen          // PascalCase
class RpsEngine           // PascalCase, viết tắt viết hoa hết
class GameStateNotifier   // Notifier suffix cho Riverpod notifier
```

### Variables & Methods
```dart
final int playerHp;           // camelCase
void pickGesture(Gesture g)   // động từ + danh từ
bool get isGameOver           // getter dùng is/has/can
```

### Constants
```dart
// core/constants.dart
const int kHpStart = 100;         // prefix k cho app-wide constants
const int kTimerSeconds = 5;
```

---

## Cấu trúc file chuẩn

```dart
// 1. Dart imports
import 'dart:math';

// 2. Flutter imports
import 'package:flutter/material.dart';

// 3. Package imports (theo alphabet)
import 'package:flame/game.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// 4. Local imports (relative path)
import '../domain/models/game_state.dart';
import '../../shared/widgets/primary_button.dart';

// 5. Part directives (nếu dùng code gen)
part 'game_provider.g.dart';
```

---

## Model phải có

```dart
class GameState {
  // ✅ immutable fields
  final int playerHp;
  final int round;

  const GameState({
    required this.playerHp,
    required this.round,
  });

  // ✅ copyWith bắt buộc
  GameState copyWith({
    int? playerHp,
    int? round,
  }) => GameState(
    playerHp: playerHp ?? this.playerHp,
    round: round ?? this.round,
  );

  // ✅ equality bắt buộc (dùng == và hashCode hoặc package equatable)
  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is GameState &&
          playerHp == other.playerHp &&
          round == other.round;

  @override
  int get hashCode => Object.hash(playerHp, round);

  // ✅ toString cho debug
  @override
  String toString() => 'GameState(hp: $playerHp, round: $round)';
}
```

---

## Widget phải làm

```dart
class HpBarWidget extends StatelessWidget {
  // ✅ nhận data đã tính sẵn, không tính trong widget
  const HpBarWidget({
    super.key,
    required this.hp,
    required this.maxHp,
  });

  final int hp;
  final int maxHp;

  // ✅ extract method khi build() > 30 dòng
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        _buildLabel(),
        _buildBar(),
      ],
    );
  }

  Widget _buildLabel() => Text('❤️ $hp/$maxHp');
  Widget _buildBar() => ...;
}
```

---

## Provider phải làm

```dart
@riverpod
class GameStateNotifier extends _$GameStateNotifier {
  // ✅ build() trả về initial state
  @override
  GameState build() => GameState.initial();

  // ✅ mỗi method làm 1 việc rõ ràng
  void pickGesture(Gesture gesture) {
    // validate trước
    if (state.phase != GamePhase.pickGesture) return;

    // gọi service, không viết logic ở đây
    final newState = ref.read(rpsEngineProvider).applyRound(
      state: state,
      playerGesture: gesture,
      aiGesture: _getAiGesture(),
    );

    state = newState;
  }
}
```

---

## Những thứ không được làm

```dart
// ❌ Magic number
if (hp < 20) { ... }

// ✅ Dùng constant
if (hp < kDangerHpThreshold) { ... }

// ❌ Comment giải thích code làm gì (code phải tự nói)
// Tăng round lên 1
state = state.copyWith(round: state.round + 1);

// ✅ Comment giải thích TẠI SAO (nếu không rõ ràng)
// AI luôn pick item trước player để tránh race condition
_aiPickItem();

// ❌ Nested ternary
final color = hp > 50 ? green : hp > 20 ? yellow : red;

// ✅ Rõ ràng
Color _hpColor(int hp) {
  if (hp > 50) return green;
  if (hp > 20) return yellow;
  return red;
}
```