# F8 — Integration & Test

> **Tuần 4** · Ưu tiên: 🔴 Validate toàn bộ Phase 1 hoạt động

---

## Mục tiêu

Kết nối tất cả F1–F7 lại, viết integration test cho luồng chính, kiểm tra không có lỗi runtime.

---

## Files cần tạo/hoàn thiện

```
test/integration/game_flow_test.dart
test/unit/level_service_test.dart
```

---

## Checklist kết nối

Trước khi viết test, kiểm tra:

- [ ] `main.dart` gọi `HiveManager.init()` và `AudioManager().init()`
- [ ] `core/router.dart` có đủ routes: `/`, `/mode-select`, `/game`, `/result`, `/settings`, `/help`
- [ ] `GameScreen` nhận `level` từ GoRouter `extra`
- [ ] `ResultScreen` nhận `playerWon`, `playerHp`, `aiHp`, `round` từ GoRouter `extra`
- [ ] `SettingsNotifier` kết nối với `AudioManager` (sfx/bgm toggle)
- [ ] `GameProvider` gọi `AudioManager` đúng thời điểm

---

## Vibe Code Prompt

```
Bạn là Flutter senior dev. Viết tests cho game Kéo Búa Bao Phase 1.

### test/integration/game_flow_test.dart
Dùng flutter_test và ProviderContainer để test luồng không cần UI.

testWidgets('Full game flow: Menu → PvE → Kết quả', (tester) async {
  // 1. App khởi động, GoRouter ở route '/'
  // 2. Pump MainMenuScreen
  // 3. Tap nút "CHƠI NGAY" → navigate đến ModeSelectScreen
  // 4. Tap card "PvE - Đấu AI" → navigate đến GameScreen
  // 5. GameState initial: playerHp=100, aiHp=100, round=1
  // 6. Tap 1 ô item → phase = pickGesture
  // 7. Tap gesture "Búa" → AI pick ngẫu nhiên → reveal
  // 8. Sau reveal → round=2 hoặc isGameOver=true
  // 9. Simulate 10 rounds đến game over
  // 10. Kiểm tra ResultScreen hiển thị đúng
});

test('GameProvider: pick item → pick gesture → state update', () async {
  final container = ProviderContainer(overrides: [
    // override AI để dùng mock trả về Gesture.rock cố định
  ]);
  
  final notifier = container.read(gameStateNotifierProvider.notifier);
  await notifier.startGame(1);
  
  // Pick item index 0
  await notifier.pickItem(0);
  expect(container.read(gameStateNotifierProvider).phase, GamePhase.pickGesture);
  
  // Pick gesture
  await notifier.pickGesture(Gesture.rock);
  final state = container.read(gameStateNotifierProvider);
  // HP phải thay đổi
  expect(state.playerHp != 100 || state.aiHp != 100, true);
});

### test/unit/level_service_test.dart
test('Level 1: 2 items, 5s timer, aiHp=100')
test('Level 5: 4 items, 3s timer, aiHp=150')
test('itemCountForLevel không vượt quá MAX_ITEMS_PER_BOARD=6')
```

---

## Manual Test Checklist (chạy trên device thật)

Trước khi đánh dấu Phase 1 Done:

```
[ ] Mở app → thấy Main Menu
[ ] Tap "CHƠI NGAY" → Mode Select
[ ] Tap "PvE" → vào Game
[ ] Tap 1 ô item → ô highlight
[ ] Tap "Búa" → reveal animation chạy
[ ] HP bar thay đổi sau mỗi lượt
[ ] Đánh đủ → vào Result Screen
[ ] Tap "Chơi lại" → game reset về round 1
[ ] Tap "Menu" → về Main Menu
[ ] Vào Settings → toggle SFX → tiếng mất/có
[ ] Vào Help → đọc luật cuộn mượt
[ ] Rotate màn hình → không crash (portrait lock nếu cần)
```

---

## Định nghĩa Done — Phase 1 Complete

- [ ] `flutter test` — tất cả tests pass
- [ ] Manual test checklist ✅ hết
- [ ] `flutter analyze` — 0 error, ≤ 5 warning
- [ ] Cả 2 thành viên demo được game trên điện thoại của nhau
- [ ] Tạo tag git: `v0.1.0-phase1-mvp`