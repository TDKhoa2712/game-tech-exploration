# F7 — Audio Manager

> **Tuần 4** · Ưu tiên: 🟢 Nice-to-have nhưng nên có trước release P1

---

## Mục tiêu

Wrapper cho `flame_audio` — phát SFX và nhạc nền, tích hợp với settings.

---

## Files cần tạo

```
lib/shared/audio/audio_manager.dart
assets/audio/bgm/       ← placeholder file .mp3
assets/audio/sfx/       ← placeholder files .mp3
```

---

## Vibe Code Prompt

```
Bạn là Flutter senior dev. Tạo AudioManager cho game Kéo Búa Bao dùng flame_audio ^2.1.1.

### audio_manager.dart — Singleton
class AudioManager {
  static final AudioManager _instance = AudioManager._();
  factory AudioManager() => _instance;

  bool _sfxEnabled = true;
  bool _bgmEnabled = true;

  Future<void> init() async {
    // Preload các SFX thường dùng
    await FlameAudio.audioCache.loadAll([
      'sfx/win.mp3',
      'sfx/lose.mp3', 
      'sfx/pick_gesture.mp3',
      'sfx/pick_item.mp3',
      'sfx/trap_trigger.mp3',
      'sfx/draw.mp3',
    ]);
  }

  // SFX
  void playWin()          { if (_sfxEnabled) FlameAudio.play('sfx/win.mp3'); }
  void playLose()         { if (_sfxEnabled) FlameAudio.play('sfx/lose.mp3'); }
  void playPickGesture()  { if (_sfxEnabled) FlameAudio.play('sfx/pick_gesture.mp3'); }
  void playPickItem()     { if (_sfxEnabled) FlameAudio.play('sfx/pick_item.mp3'); }
  void playTrap()         { if (_sfxEnabled) FlameAudio.play('sfx/trap_trigger.mp3'); }
  void playDraw()         { if (_sfxEnabled) FlameAudio.play('sfx/draw.mp3'); }

  // BGM
  Future<void> playBgm(String trackName) async {
    if (!_bgmEnabled) return;
    await FlameAudio.bgm.play('bgm/$trackName.mp3', volume: 0.4);
  }
  void stopBgm() => FlameAudio.bgm.stop();
  void pauseBgm() => FlameAudio.bgm.pause();
  void resumeBgm() { if (_bgmEnabled) FlameAudio.bgm.resume(); }

  // Settings sync (gọi từ SettingsNotifier)
  void setSfxEnabled(bool val) { _sfxEnabled = val; }
  void setBgmEnabled(bool val) {
    _bgmEnabled = val;
    if (!val) stopBgm();
  }
}

Gọi AudioManager().init() trong main.dart sau Hive.initFlutter().
Phase 1 có thể dùng audio miễn phí từ freesound.org làm placeholder.
```

---

## Assets placeholder cần tạo

Tạo file README trong mỗi thư mục audio:

`assets/audio/sfx/README.md`:
```
Cần chuẩn bị:
- win.mp3       — tiếng hoan hô/trống
- lose.mp3      — tiếng ủ rũ
- pick_gesture.mp3 — tiếng nhấn nút
- pick_item.mp3    — tiếng lật bài
- trap_trigger.mp3 — tiếng bẫy kích hoạt
- draw.mp3      — tiếng hòa

Nguồn gợi ý: freesound.org (CC0 license)
```

---

## Định nghĩa Done

- [ ] `AudioManager().init()` không crash khi file chưa có (graceful fail)
- [ ] Tắt SFX từ Settings → không nghe thấy tiếng
- [ ] BGM không bị phát chồng lên nhau khi navigate
- [ ] Volume BGM mặc định 40% (không quá to)