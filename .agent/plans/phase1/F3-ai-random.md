# F3 — AI: Random Strategy

> **Tuần 2** · Ưu tiên: 🟡 Cần trước khi có game screen

---

## Mục tiêu

Tạo interface AI và implement RandomAI — AI chọn ngẫu nhiên Gesture và Item. Đủ để chạy PvE Phase 1.

---

## Files cần tạo

```
lib/features/ai/pve_ai.dart
lib/features/ai/random_ai.dart
```

---

## Vibe Code Prompt

```
Bạn là Dart senior dev. Tạo AI layer cho game Kéo Búa Bao.
Đã có: Gesture (enum), Item, GameState từ domain layer.

### pve_ai.dart — Interface
abstract class PveAI {
  // AI chọn Gesture cho lượt này
  Gesture chooseGesture(GameState state);

  // AI chọn Item từ danh sách items của nó
  Item? chooseItem(GameState state);

  // Tên AI hiển thị
  String get displayName;

  // Mức độ khó
  AIDifficulty get difficulty;
}

enum AIDifficulty { random, heuristic, adaptive }

### random_ai.dart — RandomAI implements PveAI
- chooseGesture: Random().nextInt(3) → Gesture.values[n]
- chooseItem: random pick từ state.aiItems chưa reveal
  - 30% chance bỏ qua item (không pick item)
- displayName: 'Đối thủ May Rủi'
- difficulty: AIDifficulty.random

Dùng Dart's Random, không import framework nào.
```

---

## Định nghĩa Done

- [ ] `PveAI` là abstract class, dễ mock trong test
- [ ] `RandomAI` không bao giờ crash khi aiItems rỗng
- [ ] Có thể inject vào GameProvider dễ dàng (truyền qua constructor)