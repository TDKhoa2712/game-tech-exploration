# Kéo Búa Bao — Cấu trúc dự án

## Legend

| Badge | Ý nghĩa |
|---|---|
| `[core]` | Bắt buộc ngay |
| `[P1]` | Phase 1 — MVP (Tuần 1–4) |
| `[P2]` | Phase 2 — Alpha (Tuần 5–13) |
| `[P3]` | Phase 3 — Beta/Release (Tuần 14–19) |

---

## Kiến trúc theo Layer

```
┌─────────────────────────────────────────────────────────┐
│  Presentation Layer                                     │
│  Screens · Widgets · Riverpod Providers · Flame · Router│
└───────────────────────┬─────────────────────────────────┘
        Providers inject│↕
┌───────────────────────▼─────────────────────────────────┐
│  Domain Layer  (thuần Dart, không phụ thuộc framework)  │
│  Models · RPS Engine · Item Service · Level Service · AI│
└───────────────────────┬─────────────────────────────────┘
       Repository pattern│↕
┌───────────────────────▼─────────────────────────────────┐
│  Data Layer                                             │
│  Hive Repos · Remote Repos · Audio Manager · Share Svc  │
└───────────────────────┬─────────────────────────────────┘
              Calls to  │↓
┌───────────────────────▼─────────────────────────────────┐
│  External / Infrastructure                              │
│  Hive · Firebase/Supabase · flame_audio · Rive          │
│  WebSocket · GitHub Actions                             │
└─────────────────────────────────────────────────────────┘

Dependency rule: outer layers phụ thuộc vào inner layers, không ngược lại
```

---

## Cấu trúc thư mục

```
keo_bua_bao/
├── pubspec.yaml                        # flame, riverpod, hive, flame_audio, rive, go_router, i18n
├── main.dart                           # entry point, ProviderScope, ThemeMode init
├── app.dart                            # MaterialApp.router, GoRouter setup, Light/Dark theme
├── analysis_options.yaml
├── .github/workflows/ci.yml            # GitHub Actions: test + build APK/IPA [P3]
│
├── lib/
│   ├── core/                           # [core]
│   │   ├── constants.dart              # HP_START=100, TIMER_SECONDS=5, damage configs
│   │   ├── theme.dart                  # AppTheme light/dark, color tokens
│   │   ├── router.dart                 # GoRouter: tất cả routes của app
│   │   ├── errors/
│   │   │   ├── app_exception.dart
│   │   │   └── failure.dart
│   │   └── utils/
│   │       ├── extensions.dart         # String, List ext
│   │       └── logger.dart
│   │
│   ├── features/                       # [core]
│   │   │
│   │   ├── game/                       # [P1]
│   │   │   ├── domain/
│   │   │   │   ├── models/
│   │   │   │   │   ├── gesture.dart    # enum Gesture { rock, paper, scissors }
│   │   │   │   │   ├── item.dart       # ItemType { attack, defense, trap }, ItemEffect
│   │   │   │   │   ├── game_state.dart # HP, round, items, phase, result
│   │   │   │   │   └── player.dart     # Player local model
│   │   │   │   └── services/
│   │   │   │       ├── rps_engine.dart     # logic thắng/thua/hòa thuần Dart
│   │   │   │       ├── item_service.dart   # damage calc, trap trigger, pick validate
│   │   │   │       └── level_service.dart  # level scaling, số ô vật phẩm theo level
│   │   │   ├── presentation/
│   │   │   │   ├── providers/
│   │   │   │   │   ├── game_provider.dart      # GameStateNotifier (Riverpod)
│   │   │   │   │   └── timer_provider.dart     # countdown 3–5s per round
│   │   │   │   ├── screens/
│   │   │   │   │   ├── game_screen.dart        # FlameGameWidget embedded trong Flutter
│   │   │   │   │   └── result_screen.dart      # màn kết quả trận
│   │   │   │   └── widgets/
│   │   │   │       ├── hp_bar_widget.dart          # thanh máu 2 player
│   │   │   │       ├── gesture_picker_widget.dart  # 3 nút Kéo/Búa/Bao + timer
│   │   │   │       ├── item_board_widget.dart       # bàn N ô vật phẩm ẩn
│   │   │   │       ├── round_history_widget.dart    # N lượt gần nhất [P2]
│   │   │   │       └── wrong_pick_alert_widget.dart # cảnh báo pick nhầm [P2]
│   │   │   └── flame/
│   │   │       ├── kbb_game.dart                       # FlameGame chính (extends FlameGame)
│   │   │       └── components/
│   │   │           ├── item_cell_component.dart         # ô vật phẩm flip animation
│   │   │           ├── reveal_component.dart            # reveal đồng thời kết quả RPS
│   │   │           ├── character_component.dart         # nhân vật + Rive animation [P2]
│   │   │           └── slapstick_component.dart         # hiệu ứng Trap/sỉ nhục [P2]
│   │   │
│   │   ├── menu/                       # [P1]
│   │   │   └── presentation/screens/
│   │   │       ├── main_menu_screen.dart    # nút Play, Settings, GoRouter nav
│   │   │       ├── mode_select_screen.dart  # PvE / PvP / Room / Rank
│   │   │       ├── settings_screen.dart     # SFX, nhạc, Light/Dark, i18n
│   │   │       ├── help_screen.dart         # luật chơi xem bất kỳ lúc nào
│   │   │       └── providers/
│   │   │           └── settings_provider.dart  # Hive lưu settings
│   │   │
│   │   ├── ai/                         # [P1]
│   │   │   ├── pve_ai.dart             # interface AIStrategy
│   │   │   ├── random_ai.dart          # AI random [P1]
│   │   │   ├── heuristic_ai.dart       # đọc pattern trung bình [P2]
│   │   │   └── adaptive_ai.dart        # AI khó, pattern linh hoạt [P2]
│   │   │
│   │   ├── profile/                    # [P2]
│   │   │   ├── domain/player_stats.dart                    # wins, losses, XP, currency
│   │   │   └── data/
│   │   │       ├── profile_hive_repository.dart            # local Hive
│   │   │       └── profile_remote_repository.dart          # Firestore sync [P3]
│   │   │   └── presentation/
│   │   │       ├── providers/profile_provider.dart
│   │   │       └── screens/profile_screen.dart
│   │   │
│   │   ├── character/                  # [P2]
│   │   │   ├── domain/character.dart               # tên, xuất thân dân gian, màu, unlock
│   │   │   └── presentation/
│   │   │       ├── screens/character_select_screen.dart
│   │   │       └── providers/character_provider.dart
│   │   │
│   │   ├── shop/                       # [P2]
│   │   │   ├── domain/shop_item.dart               # cosmetic, emote, sticker (không pay-to-win)
│   │   │   ├── data/shop_hive_repository.dart
│   │   │   └── presentation/
│   │   │       ├── screens/shop_screen.dart
│   │   │       └── providers/shop_provider.dart
│   │   │
│   │   ├── tutorial/                   # [P2]
│   │   │   ├── tutorial_provider.dart          # trigger Level 1-3 onboarding
│   │   │   └── tutorial_overlay_widget.dart
│   │   │
│   │   ├── auth/                       # [P3]
│   │   │   ├── domain/auth_repository.dart
│   │   │   ├── data/firebase_auth_service.dart     # Google/Apple Sign-In
│   │   │   └── presentation/
│   │   │       ├── providers/auth_provider.dart
│   │   │       └── screens/login_screen.dart
│   │   │
│   │   ├── pvp/                        # [P3]
│   │   │   ├── domain/room.dart                    # Room model, RoomState
│   │   │   ├── data/
│   │   │   │   ├── realtime_service.dart            # Firebase Realtime / WebSocket
│   │   │   │   └── matchmaking_service.dart         # MMR queue
│   │   │   └── presentation/
│   │   │       ├── screens/
│   │   │       │   ├── room_screen.dart             # tạo/join phòng, mã mời
│   │   │       │   └── pvp_game_screen.dart
│   │   │       └── providers/pvp_provider.dart
│   │   │
│   │   ├── rank/                       # [P3]
│   │   │   ├── domain/rank.dart                    # Bronze→Silver→Gold, MMR
│   │   │   ├── data/rank_repository.dart
│   │   │   └── presentation/
│   │   │       ├── screens/leaderboard_screen.dart
│   │   │       └── providers/rank_provider.dart
│   │   │
│   │   └── social/                     # [P3]
│   │       ├── share_service.dart          # RepaintBoundary → lưu ảnh kết quả
│   │       ├── deep_link_service.dart      # Firebase Dynamic Links mời phòng
│   │       └── sticker_service.dart        # gửi/nhận sticker chê
│   │
│   └── shared/                         # [core]
│       ├── widgets/
│       │   ├── primary_button.dart
│       │   ├── app_scaffold.dart
│       │   ├── loading_overlay.dart
│       │   └── error_view.dart
│       ├── audio/
│       │   └── audio_manager.dart          # flame_audio wrapper: SFX + nhạc nền
│       ├── storage/
│       │   ├── hive_manager.dart           # init Hive, đăng ký adapters
│       │   └── adapters/
│       │       ├── settings_adapter.dart
│       │       ├── player_stats_adapter.dart
│       │       └── shop_adapter.dart
│       └── l10n/
│           ├── app_vi.arb                  # Tiếng Việt [P3]
│           └── app_en.arb                  # English [P3]
│
├── assets/
│   ├── audio/
│   │   ├── bgm/        # nhạc nền dân gian VN (CC)
│   │   └── sfx/        # thắng/thua/pick/trap
│   ├── images/
│   │   ├── characters/ # sprites nhân vật dân gian VN
│   │   ├── items/      # đòn gánh, nón lá, vỏ chuối...
│   │   ├── ui/         # icons, buttons, backgrounds
│   │   └── stickers/   # sticker chê [P2]
│   ├── animations/
│   │   └── *.riv       # Rive: idle, win, lose mỗi nhân vật [P2]
│   └── fonts/
│
└── test/
    ├── unit/
    │   ├── rps_engine_test.dart        # logic thắng/thua/hòa [P1]
    │   ├── item_service_test.dart      # damage, trap, pick validate [P1]
    │   ├── level_service_test.dart
    │   └── ai_test.dart                # heuristic/adaptive AI [P2]
    ├── widget/
    │   └── hp_bar_test.dart
    └── integration/
        └── game_flow_test.dart         # Menu→PvE→Kết quả→Chơi lại [P1]
```

---

## Roadmap tóm tắt

| Phase | Thời gian | Mục tiêu chính |
|---|---|---|
| **P1 — MVP** | Tuần 1–4 | Chơi được trận PvE đầu tiên |
| **P2 — Alpha** | Tuần 5–13 | Phong phú & hấp dẫn hơn |
| **P3 — Beta/Release** | Tuần 14–19 | Backend + PvP + Store |

Chi tiết từng phase → xem [`phase1/PLAN.md`](phase1/PLAN.md)