# F2 — Domain: Models & RPS Engine

> **Tuần 2** · Ưu tiên: 🔴 Core logic, không phụ thuộc framework nào

---

## Mục tiêu

Viết toàn bộ business logic game dưới dạng thuần Dart — không import Flutter, không import Flame. Có thể test độc lập hoàn toàn.

---

## Files cần tạo

```
lib/features/game/domain/models/gesture.dart
lib/features/game/domain/models/item.dart
lib/features/game/domain/models/game_state.dart
lib/features/game/domain/models/player.dart
lib/features/game/domain/services/rps_engine.dart
lib/features/game/domain/services/item_service.dart
lib/features/game/domain/services/level_service.dart

test/unit/rps_engine_test.dart
test/unit/item_service_test.dart
```

---

## Vibe Code Prompt

```
Bạn là Dart senior dev. Tạo domain layer cho game Kéo Búa Bao — thuần Dart, không import Flutter/Flame.

### gesture.dart
enum Gesture { rock, paper, scissors }

extension GestureExt on Gesture {
  // Trả về true nếu this thắng other
  bool beats(Gesture other)
  // Tên tiếng Việt
  String get displayName
  // Asset path icon
  String get iconPath
}

### item.dart
enum ItemType { attack, defense, trap }

class ItemEffect {
  final int damageBonus;    // thêm damage khi thắng
  final int damageReduce;   // giảm damage khi thua
  final int trapDamage;     // damage khi trap kích hoạt
  final bool isTrap;
}

class Item {
  final String id;
  final String name;         // đòn gánh, nón lá, vỏ chuối...
  final ItemType type;
  final ItemEffect effect;
  final String assetPath;
  final bool isRevealed;    // ô đã lật hay chưa
}

### game_state.dart
enum GamePhase { pickItem, pickGesture, reveal, roundEnd, gameOver }
enum RoundResult { win, lose, draw }

class GameState {
  final int playerHp;
  final int aiHp;
  final int round;
  final GamePhase phase;
  final List<Item> playerItems;   // tối đa 6 ô
  final List<Item> aiItems;
  final Item? playerSelectedItem;
  final Item? aiSelectedItem;
  final Gesture? playerGesture;
  final Gesture? aiGesture;
  final RoundResult? lastRoundResult;
  final bool isGameOver;
  final bool playerWon;

  GameState copyWith({...})
  static GameState initial()   // HP=100, round=1, phase=pickItem
}

### player.dart
class Player {
  final String id;
  final String name;
  final bool isAI;
}

### rps_engine.dart
class RpsEngine {
  // Xác định kết quả 1 lượt
  RoundResult determineResult(Gesture player, Gesture ai)

  // Tính damage sau khi áp dụng item effects
  int calculateDamage({
    required RoundResult result,
    required Item? playerItem,
    required Item? aiItem,
  })

  // Áp dụng kết quả vào GameState, trả về state mới
  GameState applyRound({
    required GameState state,
    required Gesture playerGesture,
    required Gesture aiGesture,
  })
}

### item_service.dart
class ItemService {
  // Sinh danh sách item ngẫu nhiên cho 1 lượt
  List<Item> generateItems(int level)

  // Validate player có thể pick item này không
  bool canPickItem(Item item, GameState state)

  // Trigger trap nếu điều kiện thỏa (gọi sau reveal)
  GameState triggerTrapIfNeeded(GameState state)
}

### level_service.dart
class LevelService {
  // Số ô item theo level (2 ô ở level 1, tăng dần)
  int itemCountForLevel(int level)

  // HP AI theo level
  int aiHpForLevel(int level)

  // Timer countdown theo level (5s level 1 → 3s level 5+)
  int timerSecondsForLevel(int level)
}

### test/unit/rps_engine_test.dart
Viết test đầy đủ cho RpsEngine:
- Kéo thắng Búa, Búa thắng Bao, Bao thắng Kéo
- Hòa không mất máu
- Damage cơ bản = BASE_DAMAGE
- Item attack tăng damage khi thắng
- Item defense giảm damage khi thua
- Trap kích hoạt đúng điều kiện

### test/unit/item_service_test.dart
- generateItems trả về đúng số lượng theo level
- canPickItem trả false khi item đã reveal
- triggerTrapIfNeeded không thay đổi state khi không có trap
```

---

## Định nghĩa Done

- [ ] Tất cả models có `copyWith` và `==` override
- [ ] `flutter test test/unit/rps_engine_test.dart` — 100% pass
- [ ] `flutter test test/unit/item_service_test.dart` — 100% pass
- [ ] Không có import nào từ Flutter/Flame trong domain/