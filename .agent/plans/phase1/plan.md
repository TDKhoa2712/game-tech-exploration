# Phase 1 — MVP Plan (Tuần 1–4)

> **Mục tiêu:** Chơi được trận PvE đầu tiên — vòng lặp hoàn chỉnh Menu → PvE → Kết quả → Chơi lại

---

## Checklist mục tiêu kiểm chứng

- [ ] Core loop: Kéo/Búa/Bao + HP 100 + 2 ô vật phẩm
- [ ] PvE với AI random
- [ ] Vòng lặp đầy đủ: Menu → PvE → Kết quả → Chơi lại
- [ ] Flame GameWidget nhúng vào Flutter thành công
- [ ] Unit test cho RPS engine pass
- [ ] Cả 2 thành viên hiểu Flame + Riverpod cơ bản

---

## Thứ tự build — Phase 1

```
Tuần 1  →  [F1] Project Setup + Core
Tuần 2  →  [F2] Domain Layer (RPS Engine + Models)
            [F3] AI Random
Tuần 3  →  [F4] Flame Game (kbb_game + components)
            [F5] Game Screen + Providers
Tuần 4  →  [F6] Menu Screens
            [F7] Audio Manager (SFX cơ bản)
            [F8] Kết nối toàn bộ + Integration Test
```

---

## Danh sách chức năng Phase 1

| ID | Chức năng | File plan |
|---|---|---|
| F1 | Project Setup & Core | [F1-project-setup.md](F1-project-setup.md) |
| F2 | Domain — Models & RPS Engine | [F2-domain-rps-engine.md](F2-domain-rps-engine.md) |
| F3 | AI — Random Strategy | [F3-ai-random.md](F3-ai-random.md) |
| F4 | Flame Game & Components | [F4-flame-game.md](F4-flame-game.md) |
| F5 | Game Screen & Providers | [F5-game-screen.md](F5-game-screen.md) |
| F6 | Menu Screens | [F6-menu-screens.md](F6-menu-screens.md) |
| F7 | Audio Manager | [F7-audio-manager.md](F7-audio-manager.md) |
| F8 | Integration & Test | [F8-integration-test.md](F8-integration-test.md) |

---

## Dependencies giữa các chức năng

```
F1 (Setup)
  └─► F2 (Domain/Models)
        └─► F3 (AI)
        └─► F4 (Flame)
              └─► F5 (Game Screen)
                    └─► F6 (Menu)
                          └─► F8 (Integration)
      F7 (Audio) ──────────► F8
```

---

## Files tạo mới trong Phase 1

```
main.dart · app.dart
core/constants.dart · core/theme.dart · core/router.dart
core/errors/ · core/utils/

game/domain/models/gesture.dart
game/domain/models/item.dart
game/domain/models/game_state.dart
game/domain/models/player.dart
game/domain/services/rps_engine.dart
game/domain/services/item_service.dart
game/domain/services/level_service.dart

game/flame/kbb_game.dart
game/flame/components/item_cell_component.dart
game/flame/components/reveal_component.dart

game/presentation/providers/game_provider.dart
game/presentation/providers/timer_provider.dart
game/presentation/screens/game_screen.dart
game/presentation/screens/result_screen.dart
game/presentation/widgets/hp_bar_widget.dart
game/presentation/widgets/gesture_picker_widget.dart
game/presentation/widgets/item_board_widget.dart

menu/presentation/screens/main_menu_screen.dart
menu/presentation/screens/mode_select_screen.dart
menu/presentation/screens/settings_screen.dart
menu/presentation/screens/help_screen.dart
menu/presentation/providers/settings_provider.dart

ai/pve_ai.dart
ai/random_ai.dart

shared/audio/audio_manager.dart
shared/storage/hive_manager.dart
shared/widgets/primary_button.dart · app_scaffold.dart · loading_overlay.dart · error_view.dart

test/unit/rps_engine_test.dart
test/unit/item_service_test.dart
test/integration/game_flow_test.dart
```