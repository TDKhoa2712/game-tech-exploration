# 📚 Series Giảng Dạy — Flutter Game Tech Stack

> Tài liệu học tập toàn diện cho dự án Flutter + Flame game, bao gồm toàn bộ tech stack từ ngôn ngữ đến CI/CD.

---

## Danh Sách Tài Liệu

| # | Chủ đề | File | Nội dung chính |
|---|--------|------|----------------|
| 01 | **Dart** | `01_dart.md` | Types, Null Safety, OOP, Async/Await, Generics, Extensions |
| 02 | **Flutter** | `02_flutter.md` | Widgets, Layout, Navigation, Forms, Theme, Animation |
| 03 | **Flame** | `03_flame.md` | Game Loop, Components, Sprites, Input, Collision, Camera |
| 04 | **Riverpod** | `04_riverpod.md` | Providers, StateNotifier, FutureProvider, Testing |
| 05 | **Hive** | `05_hive.md` | Setup, Models, CRUD, Query, Watch, Encryption |
| 06 | **Audio & Animation** | `06_audio_animation.md` | flame_audio BGM/SFX, Rive State Machine |
| 07 | **GitHub Actions** | `07_github_actions.md` | CI pipeline, CD pipeline, Secrets, Optimize |

---

## Thứ Tự Học Khuyến Nghị

```
Người mới học Flutter:
  01 Dart → 02 Flutter → 04 Riverpod → 05 Hive → 03 Flame → 06 Audio/Rive → 07 CI/CD

Đã biết Flutter, học game dev:
  03 Flame → 04 Riverpod → 05 Hive → 06 Audio/Rive → 07 CI/CD

Chỉ cần CI/CD:
  07 GitHub Actions (độc lập)
```

---

## Tech Stack Overview

```
┌─────────────────────────────────────────────┐
│              Flutter App Shell              │
│  (MaterialApp, ThemeData, GoRouter)         │
├─────────────────────────────────────────────┤
│        State Layer (Riverpod)               │
│  gameProvider  scoreProvider  themeProvider │
├──────────────────┬──────────────────────────┤
│   Game Layer     │      UI Layer            │
│   (Flame)        │      (Flutter Screens)   │
│  - Components    │  - HomeScreen            │
│  - Collision     │  - SettingsScreen        │
│  - Camera        │  - LeaderboardScreen     │
├──────────────────┴──────────────────────────┤
│              Service Layer                  │
│    AudioService        StorageService       │
│    (flame_audio)       (Hive)               │
├─────────────────────────────────────────────┤
│           Animation Layer (Rive)            │
│    Character.riv    Button.riv    HUD.riv   │
├─────────────────────────────────────────────┤
│           CI/CD (GitHub Actions)            │
│    ci.yml: test + lint    cd.yml: release   │
└─────────────────────────────────────────────┘
```

---

## Bài Tập Tổng Hợp — Mini Game

Sau khi học xong toàn bộ series, build mini game hoàn chỉnh:

**Yêu cầu:**
- Nhân vật di chuyển bằng touch drag (Flame + Input)
- Sao rơi từ trên xuống, tránh bị chạm (Flame + Collision)
- Đếm điểm theo thời gian sống sót (Riverpod)
- Lưu điểm cao vào Hive, hiển thị bảng xếp hạng
- Nhạc nền và SFX khi collect/bị hit (flame_audio)
- Rive animation cho nhân vật
- Light/Dark mode toggle trong Settings
- CI pipeline tự động test khi push

**Màn hình cần có:**
1. Splash Screen (load assets)
2. Home Screen (Play, Leaderboard, Settings)
3. Game Screen (Flame GameWidget + HUD overlay)
4. Game Over Overlay (điểm, high score, play again)
5. Leaderboard Screen (top 10 từ Hive)
6. Settings Screen (âm thanh, theme, tên player)

---

*Chúc học tốt! Có câu hỏi cụ thể về bất kỳ phần nào, cứ hỏi thêm.*
