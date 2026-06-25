# F4 — Flame Game & Components

> **Tuần 3** · Ưu tiên: 🔴 Core visual của game

---

## Mục tiêu

Tạo FlameGame chính và các Component cơ bản. GameScreen Flutter sẽ nhúng `GameWidget` vào.

---

## Files cần tạo

```
lib/features/game/flame/kbb_game.dart
lib/features/game/flame/components/item_cell_component.dart
lib/features/game/flame/components/reveal_component.dart
```

---

## Vibe Code Prompt

```
Bạn là Flutter/Flame senior dev. Tạo Flame layer cho game Kéo Búa Bao.
Flame version: ^1.18.0. Đã có GameState, Item, Gesture từ domain.

### kbb_game.dart — KbbGame extends FlameGame
Fields:
- late ItemBoardComponent playerBoard
- late ItemBoardComponent aiBoard
- late RevealComponent revealComponent
- GameState gameState (nhận từ bên ngoài, update qua method)

onLoad():
- Thêm playerBoard (bottom half màn hình)
- Thêm aiBoard (top half, flip ngược)
- Thêm revealComponent (center)
- Background: màu truyền thống VN (#2C1810 — nâu gỗ)

Methods:
- updateGameState(GameState newState): cập nhật visual từ state mới
- triggerReveal(Gesture player, Gesture ai): chạy RevealComponent animation
- onItemCellTapped(int index): callback ra Flutter (dùng Function?)

### components/item_cell_component.dart — ItemCellComponent extends PositionComponent
Là 1 ô vật phẩm hình vuông bo góc.

States:
- hidden: hiển thị mặt sau (dấu ?)
- revealed: flip animation → hiển thị icon item
- selected: glow border vàng

onTapDown(): gọi onTapped callback

Flip animation: dùng SequenceEffect với ScaleEffect theo trục X (giả lập card flip)
- 150ms thu lại → đổi sprite → 150ms mở ra

Kích thước: 64x64px, margin 8px giữa các ô

### components/reveal_component.dart — RevealComponent extends PositionComponent
Hiển thị kết quả RPS đồng thời ở center màn hình.

showReveal(Gesture playerGesture, Gesture aiGesture, RoundResult result):
1. Fade in 2 icon gesture (player trái, AI phải) — 300ms
2. Hiển thị text kết quả (Thắng!/Thua!/Hòa!) với màu tương ứng
3. Pause 1.2 giây
4. Fade out toàn bộ — 200ms
5. Gọi onRevealComplete callback

Dùng Flame effects: OpacityEffect, SequenceEffect, DelayedEffect
```

---

## Định nghĩa Done

- [ ] `KbbGame` load không crash trên Android emulator
- [ ] `ItemCellComponent` flip animation mượt 60fps
- [ ] `RevealComponent` complete callback hoạt động đúng thứ tự
- [ ] Không hardcode kích thước — dùng `size` từ parent