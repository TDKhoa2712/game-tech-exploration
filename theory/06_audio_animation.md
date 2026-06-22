# 06 · flame_audio & Rive — Âm Thanh và Animation

---

## Phần A — flame_audio

> `flame_audio` là package âm thanh chính thức của Flame. Được xây dựng trên `audioplayers`, hỗ trợ BGM (nhạc nền), SFX (hiệu ứng âm thanh), preloading và volume control.

### A.1 Setup

```yaml
# pubspec.yaml
dependencies:
  flame_audio: ^2.1.1
```

```yaml
# Khai báo file âm thanh trong pubspec.yaml
flutter:
  assets:
    - assets/audio/bgm/
    - assets/audio/sfx/
```

**Cấu trúc thư mục:**
```
assets/audio/
├── bgm/
│   ├── main_menu.mp3
│   ├── gameplay.mp3
│   └── game_over.mp3
└── sfx/
    ├── coin_collect.wav
    ├── player_jump.wav
    ├── enemy_hit.wav
    ├── button_click.wav
    └── explosion.wav
```

---

### A.2 BGM — Nhạc Nền

```dart
import 'package:flame_audio/flame_audio.dart';

// Phát nhạc nền (loop tự động)
await FlameAudio.bgm.play('bgm/gameplay.mp3', volume: 0.6);

// Dừng
FlameAudio.bgm.stop();

// Tạm dừng / tiếp tục
FlameAudio.bgm.pause();
FlameAudio.bgm.resume();

// Kiểm tra trạng thái
bool isPlaying = FlameAudio.bgm.isPlaying;

// Thay đổi volume
await FlameAudio.bgm.audioPlayer.setVolume(0.8);

// Chuyển bài (fade out bài cũ, play bài mới)
await FlameAudio.bgm.stop();
await FlameAudio.bgm.play('bgm/main_menu.mp3', volume: 0.5);
```

---

### A.3 SFX — Hiệu Ứng Âm Thanh

```dart
// Play một lần (fire and forget)
await FlameAudio.play('sfx/coin_collect.wav');

// Play với volume tùy chỉnh
await FlameAudio.play('sfx/explosion.wav', volume: 1.0);

// Nhiều sound cùng lúc (không chờ nhau)
FlameAudio.play('sfx/coin_collect.wav');
FlameAudio.play('sfx/player_jump.wav');

// Play và lấy AudioPlayer để control
final player = await FlameAudio.playLongAudio('sfx/boss_music.mp3');
await player.pause();
await player.resume();
await player.stop();
```

---

### A.4 Preloading — Tải Trước Để Tránh Lag

```dart
// Preload trong onLoad của game (quan trọng!)
@override
Future<void> onLoad() async {
  // Preload tất cả SFX cùng một lúc
  await FlameAudio.audioCache.loadAll([
    'sfx/coin_collect.wav',
    'sfx/player_jump.wav',
    'sfx/enemy_hit.wav',
    'sfx/button_click.wav',
    'sfx/explosion.wav',
  ]);

  // Sau khi preload, play sẽ tức thì không có delay
}

// Xóa khỏi cache khi không cần
FlameAudio.audioCache.clear('sfx/boss_music.mp3');
FlameAudio.audioCache.clearAll();
```

---

### A.5 AudioService — Quản Lý Tập Trung

```dart
class AudioService {
  bool _isMuted = false;
  double _sfxVolume = 0.8;
  double _bgmVolume = 0.5;

  // Singleton
  static final AudioService _instance = AudioService._internal();
  factory AudioService() => _instance;
  AudioService._internal();

  bool get isMuted => _isMuted;
  double get sfxVolume => _sfxVolume;
  double get bgmVolume => _bgmVolume;

  Future<void> init() async {
    await FlameAudio.audioCache.loadAll([
      'sfx/coin_collect.wav',
      'sfx/player_jump.wav',
      'sfx/enemy_hit.wav',
      'sfx/button_click.wav',
      'sfx/explosion.wav',
    ]);
  }

  // BGM
  Future<void> playBGM(String filename) async {
    if (_isMuted) return;
    await FlameAudio.bgm.play(filename, volume: _bgmVolume);
  }

  Future<void> stopBGM() => FlameAudio.bgm.stop();
  Future<void> pauseBGM() => FlameAudio.bgm.pause();
  Future<void> resumeBGM() => FlameAudio.bgm.resume();

  // SFX
  Future<void> playSFX(String filename) async {
    if (_isMuted) return;
    await FlameAudio.play(filename, volume: _sfxVolume);
  }

  // Mute/Unmute
  Future<void> toggleMute() async {
    _isMuted = !_isMuted;
    if (_isMuted) {
      await FlameAudio.bgm.pause();
    } else {
      await FlameAudio.bgm.resume();
    }
  }

  // Volume
  Future<void> setSFXVolume(double volume) async {
    _sfxVolume = volume.clamp(0.0, 1.0);
  }

  Future<void> setBGMVolume(double volume) async {
    _bgmVolume = volume.clamp(0.0, 1.0);
    if (FlameAudio.bgm.isPlaying) {
      await FlameAudio.bgm.audioPlayer.setVolume(_bgmVolume);
    }
  }
}

// Tích hợp với Riverpod
@riverpod
AudioService audioService(AudioServiceRef ref) {
  final service = AudioService();
  ref.onDispose(() => FlameAudio.bgm.stop());
  return service;
}
```

---

### A.6 Tích Hợp Vào Game

```dart
class MyGame extends FlameGame {
  final AudioService audio = AudioService();

  @override
  Future<void> onLoad() async {
    await audio.init();
    await audio.playBGM('bgm/gameplay.mp3');
  }
}

// Trong component
class PlayerComponent extends SpriteComponent with HasGameRef<MyGame> {
  void onJump() {
    gameRef.audio.playSFX('sfx/player_jump.wav');
    // ... logic jump
  }

  void onCollectCoin() {
    gameRef.audio.playSFX('sfx/coin_collect.wav');
    // ... logic collect
  }
}

// Trong Flutter overlay (pause menu)
class PauseOverlay extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final audioService = ref.read(audioServiceProvider);

    return Switch(
      value: !audioService.isMuted,
      onChanged: (value) => audioService.toggleMute(),
    );
  }
}
```

---

---

## Phần B — Rive Animation

> Rive là công cụ animation real-time mạnh mẽ. Tạo animation trong Rive editor (rive.app), export file `.riv`, tích hợp vào Flutter/Flame với state machine điều khiển trực tiếp từ code.

### B.1 Setup

```yaml
# pubspec.yaml
dependencies:
  rive: ^0.13.4
```

```yaml
# pubspec.yaml — khai báo assets
flutter:
  assets:
    - assets/animations/rive/
```

---

### B.2 Workflow: Rive Editor → Flutter

```
1. Thiết kế animation trong rive.app
2. Tạo State Machine với inputs:
   - Boolean: isRunning, isJumping
   - Number: speed, health
   - Trigger: hit, die, celebrate
3. Export file .riv
4. Đặt vào assets/animations/rive/
5. Dùng trong Flutter
```

---

### B.3 RiveAnimation Widget — Đơn Giản

```dart
import 'package:rive/rive.dart';

// Play animation đơn giản (không cần control)
RiveAnimation.asset(
  'assets/animations/rive/character.riv',
  fit: BoxFit.contain,
)

// Chỉ định animation cụ thể
RiveAnimation.asset(
  'assets/animations/rive/ui_effects.riv',
  animations: const ['idle'],
)

// Từ network
RiveAnimation.network(
  'https://cdn.rive.app/animations/vehicles.riv',
)
```

---

### B.4 State Machine — Điều Khiển Từ Code

```dart
class CharacterAnimation extends StatefulWidget {
  const CharacterAnimation({super.key});

  @override
  State<CharacterAnimation> createState() => _CharacterAnimationState();
}

class _CharacterAnimationState extends State<CharacterAnimation> {
  // Controller
  StateMachineController? _controller;

  // Inputs — kết nối với biến trong Rive State Machine
  SMIBool? _isRunning;
  SMIBool? _isJumping;
  SMINumber? _speed;
  SMITrigger? _hitTrigger;
  SMITrigger? _dieTrigger;

  void _onRiveInit(Artboard artboard) {
    final controller = StateMachineController.fromArtboard(
      artboard,
      'CharacterStateMachine', // tên State Machine trong Rive editor
    );

    if (controller == null) return;

    artboard.addController(controller);
    _controller = controller;

    // Lấy input theo tên (phải khớp tên trong Rive editor)
    _isRunning = controller.findInput<bool>('isRunning') as SMIBool?;
    _isJumping = controller.findInput<bool>('isJumping') as SMIBool?;
    _speed = controller.findInput<double>('speed') as SMINumber?;
    _hitTrigger = controller.findInput<bool>('hit') as SMITrigger?;
    _dieTrigger = controller.findInput<bool>('die') as SMITrigger?;
  }

  @override
  void dispose() {
    _controller?.dispose();
    super.dispose();
  }

  // Điều khiển animation từ bên ngoài
  void startRunning() => _isRunning?.value = true;
  void stopRunning() => _isRunning?.value = false;
  void jump() => _isJumping?.value = true;
  void setSpeed(double speed) => _speed?.value = speed;
  void triggerHit() => _hitTrigger?.fire();
  void triggerDie() => _dieTrigger?.fire();

  @override
  Widget build(BuildContext context) {
    return RiveAnimation.asset(
      'assets/animations/rive/character.riv',
      stateMachines: const ['CharacterStateMachine'],
      onInit: _onRiveInit,
      fit: BoxFit.contain,
    );
  }
}
```

---

### B.5 Rive Trong Flame Game

```dart
// Rive component cho Flame
class RiveCharacterComponent extends PositionComponent with HasGameRef<MyGame> {
  late RiveFile _riveFile;
  late Artboard _artboard;
  StateMachineController? _controller;

  SMIBool? _isRunning;
  SMITrigger? _jump;

  @override
  Future<void> onLoad() async {
    // Load file .riv
    final data = await Flame.bundle.load('assets/animations/rive/character.riv');
    _riveFile = RiveFile.import(data);

    _artboard = _riveFile.mainArtboard.instance();
    final controller = StateMachineController.fromArtboard(
      _artboard,
      'PlayerStateMachine',
    );

    if (controller != null) {
      _artboard.addController(controller);
      _controller = controller;
      _isRunning = controller.findInput<bool>('isRunning') as SMIBool?;
      _jump = controller.findInput<bool>('jump') as SMITrigger?;
    }

    size = Vector2(128, 128);
  }

  @override
  void update(double dt) {
    super.update(dt);
    _artboard.advance(dt); // cập nhật animation theo delta time
  }

  @override
  void render(Canvas canvas) {
    // Vẽ Rive lên canvas
    final renderer = CanvasRenderer(canvas);
    _artboard.draw(renderer, size: Size(size.x, size.y));
  }

  @override
  void onRemove() {
    _controller?.dispose();
    super.onRemove();
  }

  // API điều khiển
  void startRunning() => _isRunning?.value = true;
  void stopRunning() => _isRunning?.value = false;
  void triggerJump() => _jump?.fire();
}
```

---

### B.6 Rive Cho UI — Button Animation

```dart
// Nút với Rive hover/press animation
class RiveButton extends StatefulWidget {
  final String label;
  final VoidCallback onTap;
  const RiveButton({super.key, required this.label, required this.onTap});

  @override
  State<RiveButton> createState() => _RiveButtonState();
}

class _RiveButtonState extends State<RiveButton> {
  SMIBool? _isHovered;
  SMIBool? _isPressed;

  void _onInit(Artboard artboard) {
    final ctrl = StateMachineController.fromArtboard(artboard, 'ButtonSM');
    if (ctrl == null) return;
    artboard.addController(ctrl);
    _isHovered = ctrl.findInput<bool>('isHovered') as SMIBool?;
    _isPressed = ctrl.findInput<bool>('isPressed') as SMIBool?;
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: widget.onTap,
      onTapDown: (_) => _isPressed?.value = true,
      onTapUp: (_) => _isPressed?.value = false,
      onTapCancel: () => _isPressed?.value = false,
      child: MouseRegion(
        onEnter: (_) => _isHovered?.value = true,
        onExit: (_) => _isHovered?.value = false,
        child: SizedBox(
          width: 200,
          height: 60,
          child: Stack(
            children: [
              RiveAnimation.asset(
                'assets/animations/rive/button.riv',
                stateMachines: const ['ButtonSM'],
                onInit: _onInit,
                fit: BoxFit.fill,
              ),
              Center(
                child: Text(
                  widget.label,
                  style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

### B.7 Loading Animation

```dart
// Màn hình loading với Rive
class LoadingOverlay extends StatelessWidget {
  const LoadingOverlay({super.key});

  @override
  Widget build(BuildContext context) {
    return const ColoredBox(
      color: Colors.black54,
      child: Center(
        child: SizedBox(
          width: 150,
          height: 150,
          child: RiveAnimation.asset(
            'assets/animations/rive/loading_spinner.riv',
            animations: ['spin'],
            fit: BoxFit.contain,
          ),
        ),
      ),
    );
  }
}
```

---

### B.8 Flame + Rive + Riverpod — Full Integration

```dart
// Kết hợp 3 công nghệ: Flame component, Rive animation, Riverpod state
class PlayerComponent extends PositionComponent with HasGameRef<MyGame> {
  late RiveFile _riveFile;
  late Artboard _artboard;
  StateMachineController? _smController;
  SMIBool? _isRunning;
  SMINumber? _healthInput;

  @override
  Future<void> onLoad() async {
    // Load Rive
    final data = await Flame.bundle.load('assets/animations/rive/player.riv');
    _riveFile = RiveFile.import(data);
    _artboard = _riveFile.mainArtboard.instance();
    final sm = StateMachineController.fromArtboard(_artboard, 'PlayerSM');
    if (sm != null) {
      _artboard.addController(sm);
      _smController = sm;
      _isRunning = sm.findInput<bool>('isRunning') as SMIBool?;
      _healthInput = sm.findInput<double>('health') as SMINumber?;
    }
    size = Vector2(96, 96);
    anchor = Anchor.center;

    // Đồng bộ health từ Riverpod → Rive
    gameRef.ref.listen(
      gameProvider.select((s) => s.lives),
      (_, lives) => _healthInput?.value = lives.toDouble(),
    );
  }

  void setRunning(bool running) => _isRunning?.value = running;

  @override
  void update(double dt) {
    super.update(dt);
    _artboard.advance(dt);
  }

  @override
  void render(Canvas canvas) {
    _artboard.draw(CanvasRenderer(canvas), size: Size(size.x, size.y));
  }

  @override
  void onRemove() {
    _smController?.dispose();
    super.onRemove();
  }
}
```

---

## Tóm Tắt So Sánh

| | flame_audio | Rive |
|--|-------------|------|
| **Dùng cho** | BGM + SFX | Character/UI animation |
| **File format** | .mp3, .wav, .ogg | .riv |
| **Preload** | `audioCache.loadAll()` | Load trong `onLoad()` |
| **Điều khiển** | play/pause/stop/volume | State Machine inputs |
| **Flame support** | Native (FlameAudio.play) | Qua CanvasRenderer |
| **Flutter support** | Native | RiveAnimation.asset widget |

---

## Bài Tập Thực Hành

**Bài 1 — Audio:** Implement AudioService đầy đủ với Riverpod. UI Settings screen có 2 slider điều chỉnh BGM/SFX volume và toggle mute.

**Bài 2 — Rive basic:** Tạo idle animation đơn giản trong Rive (hoặc tải file mẫu từ rive.app), tích hợp vào Flutter widget.

**Bài 3 — State Machine:** Tạo nhân vật với 2 trạng thái `idle` ↔ `running` điều khiển bằng Boolean input khi bấm nút.

**Bài 4 — Full:** Kết hợp: Flame component dùng Rive animation, phát SFX khi chuyển state, đồng bộ với Riverpod provider.

---

*Tiếp theo: [07 · GitHub Actions](07_github_actions.md) — CI/CD Tự Động*
