# F5 — Game Screen & Providers

> **Tuần 3** · Ưu tiên: 🔴 Kết nối Flame + Flutter + State

---

## Mục tiêu

Tạo Riverpod providers quản lý game state, và GameScreen nhúng FlameGame vào Flutter UI.

---

## Files cần tạo

```
lib/features/game/presentation/providers/game_provider.dart
lib/features/game/presentation/providers/timer_provider.dart
lib/features/game/presentation/screens/game_screen.dart
lib/features/game/presentation/screens/result_screen.dart
lib/features/game/presentation/widgets/hp_bar_widget.dart
lib/features/game/presentation/widgets/gesture_picker_widget.dart
lib/features/game/presentation/widgets/item_board_widget.dart
```

---

## Vibe Code Prompt

```
Bạn là Flutter/Riverpod senior dev. Tạo presentation layer cho game Kéo Búa Bao.
Dùng flutter_riverpod ^2.5.1 với @riverpod code generation.
Đã có: GameState, RpsEngine, ItemService, LevelService, PveAI, RandomAI, KbbGame.

### game_provider.dart — GameStateNotifier
@riverpod class GameStateNotifier extends _$GameStateNotifier

State: GameState

Methods:
- startGame(int level): init GameState.initial(), generate items cho cả 2
- pickItem(int itemIndex): validate → update selectedItem → phase = pickGesture
- pickGesture(Gesture gesture): 
    1. AI cũng pick gesture (qua PveAI)
    2. Gọi RpsEngine.applyRound()
    3. phase = reveal
    4. Sau 2s (delay) → check gameOver hay tiếp tục
- resetGame(): về initial state
- ref.read(kbbGameProvider) để gọi KbbGame.triggerReveal()

### timer_provider.dart
@riverpod class TimerNotifier extends _$TimerNotifier

State: int (seconds còn lại)

- startTimer(int seconds): đếm ngược, mỗi giây emit state--
- Khi về 0: auto gọi gameProvider.pickGesture(randomGesture) (bị bắt buộc)
- cancelTimer(): dispose timer
- pauseTimer() / resumeTimer()

### game_screen.dart — GameScreen StatefulWidget
Layout (Column):
┌─────────────────────────┐
│  AI HP Bar              │  ← HpBarWidget(isPlayer: false)
│  AI Item Board (Flame)  │  ← phần trên GameWidget
├─────────────────────────┤
│  GameWidget (Flame)     │  ← Expanded, nhúng KbbGame
│  (RevealComponent ở đây)│
├─────────────────────────┤
│  Player Item Board      │  ← ItemBoardWidget
│  Gesture Picker + Timer │  ← GesturePickerWidget
│  Player HP Bar          │  ← HpBarWidget(isPlayer: true)
└─────────────────────────┘

- initState: khởi tạo KbbGame, gọi gameProvider.startGame(level)
- watch gameState, khi isGameOver → GoRouter.go('/result')
- watch gameState phase để enable/disable GesturePicker

### result_screen.dart
Nhận: playerWon (bool), playerHp, aiHp, round

Hiển thị:
- Text lớn: "Chiến Thắng! 🎉" hoặc "Thất Bại 😔"
- Stats: Số lượt đã chơi, HP còn lại
- 2 nút: "Chơi lại" (startGame lại) và "Menu" (GoRouter.go('/'))

### hp_bar_widget.dart
AnimatedContainer cho thanh HP:
- Màu: xanh (#27AE60) khi > 50%, vàng (#E67E22) khi 20-50%, đỏ (#C0392B) khi < 20%
- Label: "❤️ {hp}/100"
- Animation duration: 400ms

### gesture_picker_widget.dart
Row 3 nút + countdown timer ở giữa:
[🪨 Búa]  [⏱ 3s]  [✌️ Kéo]  [🤚 Bao]

- Disable toàn bộ khi phase != pickGesture
- Mỗi nút: InkWell với image asset + label tiếng Việt
- Timer hiển thị đếm ngược từ timerProvider

### item_board_widget.dart
Wrap các ItemCellWidget (Flutter widget, không phải Flame):
- Hiển thị N ô item dạng Row
- Ô chưa reveal: icon dấu ? + border
- Ô đã chọn: glow vàng
- onTap từng ô → gameProvider.pickItem(index)
- Disable khi phase != pickItem
```

---

## Định nghĩa Done

- [ ] GameScreen hiển thị không overflow trên màn hình 360x800
- [ ] Timer tự động pick gesture khi về 0
- [ ] HP bar animate mượt khi HP thay đổi
- [ ] Chuyển sang ResultScreen đúng sau khi game over
- [ ] Không có setState thủ công — toàn bộ qua Riverpod