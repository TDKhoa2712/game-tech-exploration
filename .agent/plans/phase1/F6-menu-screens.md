# F6 — Menu Screens

> **Tuần 4** · Ưu tiên: 🟡 Navigation wrapper cho toàn bộ app

---

## Mục tiêu

Tạo các màn menu: Main Menu, Mode Select, Settings, Help. Kết nối GoRouter hoàn chỉnh.

---

## Files cần tạo

```
lib/features/menu/presentation/screens/main_menu_screen.dart
lib/features/menu/presentation/screens/mode_select_screen.dart
lib/features/menu/presentation/screens/settings_screen.dart
lib/features/menu/presentation/screens/help_screen.dart
lib/features/menu/presentation/providers/settings_provider.dart
lib/shared/storage/hive_manager.dart
```

---

## Vibe Code Prompt

```
Bạn là Flutter/Riverpod senior dev. Tạo menu screens cho game Kéo Búa Bao.
Dùng GoRouter đã có ở core/router.dart. Theme: dân gian VN.

### settings_provider.dart
@riverpod class SettingsNotifier extends _$SettingsNotifier

State: Settings { bool sfxEnabled, bool bgmEnabled, bool isDarkMode, String locale }

- load(): đọc từ Hive box 'settings'
- toggleSfx(), toggleBgm(), toggleDarkMode()
- setLocale(String locale)
- Mỗi update tự động save vào Hive

### hive_manager.dart
class HiveManager {
  static Future<void> init() async {
    await Hive.initFlutter();
    // đăng ký adapters Phase 1: Settings adapter
    await Hive.openBox('settings');
  }
}

### main_menu_screen.dart
Layout full-screen:
- Background: gradient nâu gỗ → đỏ son nhẹ
- Logo game ở giữa trên (Text lớn: "Kéo Búa Bao" font bold)
- Tagline nhỏ: "Trò chơi dân gian Việt Nam"
- Nút "CHƠI NGAY" → GoRouter.go('/mode-select')
- Nút "CÀI ĐẶT" → GoRouter.go('/settings')  
- Nút "?" (Help) góc trên phải → GoRouter.go('/help')
- Version text góc dưới: "v0.1.0"

### mode_select_screen.dart
Chọn chế độ chơi:
- Card "PvE - Đấu AI" → GoRouter.go('/game', extra: {'mode': 'pve', 'level': 1})
- Card "PvP - Đấu bạn" → Disabled, badge "Sắp ra mắt" (P3)
- Card "Phòng riêng" → Disabled
- Card "Xếp hạng" → Disabled
- Back button

### settings_screen.dart
- SwitchListTile: Âm thanh (sfxEnabled)
- SwitchListTile: Nhạc nền (bgmEnabled)  
- SwitchListTile: Giao diện tối (isDarkMode)
- ListTile: Ngôn ngữ — chỉ hiện "Tiếng Việt" (P3 mới có EN)
- Tất cả tự động persist qua settingsProvider

### help_screen.dart
Scrollable Column:
- Tiêu đề: "Cách chơi"
- Section "Luật cơ bản": Kéo > Búa > Bao > Kéo, giải thích damage
- Section "Vật phẩm": attack / defense / trap
- Section "Vòng lặp 1 trận": Pick item → Pick gesture → Reveal → ...
- Có thể mở bất kỳ lúc nào từ in-game (GoRouter push)

Dùng PrimaryButton và AppScaffold từ shared/widgets.
Không có logic phức tạp — chỉ navigation và settings.
```

---

## Định nghĩa Done

- [ ] Navigate từ Menu → Mode Select → Game không lỗi
- [ ] Settings persist sau khi restart app (Hive hoạt động)
- [ ] Mode PvP/Room/Rank hiển thị "Sắp ra mắt" không crash
- [ ] Help screen scroll mượt, text không bị clip