# 03 · Flame — Game Engine Trên Flutter

> Flame là game engine 2D được xây dựng trên Flutter. Nó cung cấp game loop, component system, collision detection, camera, và nhiều tính năng game quan trọng.

---

## 1. Game Loop — Vòng Lặp Game

Flame tự động chạy một vòng lặp liên tục gọi `update()` và `render()` mỗi frame (~60 FPS).

```
Mỗi frame:
  update(dt)  →  cập nhật vị trí, logic, physics
  render(canvas) →  vẽ tất cả lên màn hình
```

```dart
import 'package:flame/game.dart';

class MyGame extends FlameGame {
  // Chạy một lần khi game khởi động — load assets
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    // Load sprites, audio, thêm component...
    add(PlayerComponent());
    add(BackgroundComponent());
  }

  // Chạy mỗi frame — dt = delta time (giây kể từ frame trước)
  @override
  void update(double dt) {
    super.update(dt);
    // Logic game: đếm điểm, check win/lose,...
  }

  // Vẽ thứ gì đó thẳng vào canvas (hiếm dùng — thường để cho Component làm)
  @override
  void render(Canvas canvas) {
    super.render(canvas);
  }
}

// Tích hợp vào Flutter
void main() {
  runApp(GameWidget(game: MyGame()));
}
```

---

## 2. Component System

Flame sử dụng **Component** để đại diện cho mọi object trong game: nhân vật, kẻ thù, đạn, UI,...

```dart
// Component cơ bản
class StarComponent extends PositionComponent {
  late Sprite _sprite;

  StarComponent({super.position, super.size});

  @override
  Future<void> onLoad() async {
    _sprite = await Sprite.load('star.png');
  }

  @override
  void update(double dt) {
    super.update(dt);
    position.y += 100 * dt; // rơi xuống 100 px/giây

    // Xóa khi ra khỏi màn hình
    if (position.y > gameRef.size.y) removeFromParent();
  }

  @override
  void render(Canvas canvas) {
    _sprite.render(canvas, size: size);
  }
}

// Thêm component vào game
game.add(StarComponent(
  position: Vector2(100, 0),
  size: Vector2(32, 32),
));
```

### Lifecycle của Component

```dart
class MyComponent extends PositionComponent with HasGameRef<MyGame> {
  @override
  Future<void> onLoad() async {
    // Khởi tạo: load asset, tạo hitbox,...
  }

  @override
  void onMount() {
    // Đã được add vào component tree
  }

  @override
  void update(double dt) {
    // Chạy mỗi frame
  }

  @override
  void render(Canvas canvas) {
    // Vẽ lên canvas
  }

  @override
  void onRemove() {
    // Sắp bị xóa khỏi game
  }
}
```

---

## 3. Sprites & Animation

```dart
// Sprite — một ảnh tĩnh
class PlayerComponent extends SpriteComponent with HasGameRef<MyGame> {
  @override
  Future<void> onLoad() async {
    sprite = await gameRef.loadSprite('player.png');
    size = Vector2(64, 64);
    anchor = Anchor.center;
  }
}

// SpriteAnimation — nhiều frame liên tiếp
class AnimatedPlayerComponent extends SpriteAnimationComponent
    with HasGameRef<MyGame> {

  @override
  Future<void> onLoad() async {
    // Load từ sprite sheet (8 frame, mỗi frame 64x64)
    animation = await gameRef.loadSpriteAnimation(
      'player_run.png',
      SpriteAnimationData.sequenced(
        amount: 8,
        stepTime: 0.1,          // 0.1 giây mỗi frame → 10 FPS
        textureSize: Vector2(64, 64),
      ),
    );

    size = Vector2(64, 64);
    anchor = Anchor.center;
  }
}

// Nhiều animation (idle, run, jump)
class CharacterComponent extends SpriteAnimationGroupComponent<CharacterState>
    with HasGameRef<MyGame> {

  @override
  Future<void> onLoad() async {
    final idle = await gameRef.loadSpriteAnimation(
      'char_idle.png',
      SpriteAnimationData.sequenced(amount: 4, stepTime: 0.2, textureSize: Vector2(64, 64)),
    );
    final run = await gameRef.loadSpriteAnimation(
      'char_run.png',
      SpriteAnimationData.sequenced(amount: 8, stepTime: 0.1, textureSize: Vector2(64, 64)),
    );

    animations = {
      CharacterState.idle: idle,
      CharacterState.running: run,
    };
    current = CharacterState.idle;
  }

  void startRunning() => current = CharacterState.running;
  void stopRunning() => current = CharacterState.idle;
}

enum CharacterState { idle, running, jumping }
```

---

## 4. Input — Xử Lý Đầu Vào

```dart
// Tap
class TapGame extends FlameGame with TapCallbacks {
  @override
  void onTapDown(TapDownEvent event) {
    final position = event.canvasPosition;
    add(BulletComponent(position: position));
  }
}

// Drag (kéo nhân vật)
class DraggablePlayer extends SpriteComponent with DragCallbacks {
  @override
  void onDragUpdate(DragUpdateEvent event) {
    position += event.localDelta;
  }
}

// Keyboard (PC/Web)
class KeyboardGame extends FlameGame with KeyboardEvents {
  bool _goLeft = false, _goRight = false;

  @override
  KeyEventResult onKeyEvent(KeyEvent event, Set<LogicalKeyboardKey> keys) {
    _goLeft = keys.contains(LogicalKeyboardKey.arrowLeft);
    _goRight = keys.contains(LogicalKeyboardKey.arrowRight);
    return KeyEventResult.handled;
  }

  @override
  void update(double dt) {
    super.update(dt);
    if (_goLeft)  player.position.x -= 200 * dt;
    if (_goRight) player.position.x += 200 * dt;
  }
}

// Joystick (mobile)
class JoystickGame extends FlameGame {
  late JoystickComponent joystick;
  late PlayerComponent player;

  @override
  Future<void> onLoad() async {
    joystick = JoystickComponent(
      knob: CircleComponent(radius: 20, paint: Paint()..color = Colors.white54),
      background: CircleComponent(radius: 50, paint: Paint()..color = Colors.white24),
      margin: const EdgeInsets.only(left: 32, bottom: 32),
    );

    player = PlayerComponent();
    addAll([joystick, player]);
  }

  @override
  void update(double dt) {
    super.update(dt);
    if (!joystick.delta.isZero()) {
      player.position += joystick.relativeDelta * 200 * dt;
    }
  }
}
```

---

## 5. Collision Detection — Phát Hiện Va Chạm

```dart
// Mixin HasCollisionDetection cho game
class CollisionGame extends FlameGame with HasCollisionDetection {}

// Component với hitbox
class PlayerComponent extends SpriteComponent
    with CollisionCallbacks, HasGameRef<CollisionGame> {

  @override
  Future<void> onLoad() async {
    sprite = await gameRef.loadSprite('player.png');
    size = Vector2(48, 48);

    // Hình chữ nhật
    add(RectangleHitbox());

    // Hoặc hình tròn
    // add(CircleHitbox());

    // Hoặc đa giác tùy chỉnh
    // add(PolygonHitbox([Vector2(0,0), Vector2(48,0), Vector2(24,48)]));
  }

  @override
  void onCollisionStart(Set<Vector2> intersectionPoints, PositionComponent other) {
    super.onCollisionStart(intersectionPoints, other);

    if (other is EnemyComponent) {
      // Bị kẻ thù đâm vào
      gameRef.read(gameProvider.notifier).loseLife();
    }

    if (other is CoinComponent) {
      other.removeFromParent();
      gameRef.read(scoreProvider.notifier).addScore(10);
    }
  }

  @override
  void onCollisionEnd(PositionComponent other) {
    super.onCollisionEnd(other);
    // Không còn va chạm nữa
  }
}
```

---

## 6. Camera & World

```dart
// World — không gian game lớn hơn màn hình
class ScrollingGame extends FlameGame {
  @override
  Future<void> onLoad() async {
    final world = World();
    final camera = CameraComponent(world: world);

    // Camera follow nhân vật
    camera.follow(player, maxSpeed: 300);

    // Giới hạn camera không ra ngoài map
    camera.setBounds(Rectangle.fromLTWH(0, 0, 2000, 1000));

    addAll([world, camera]);
    world.add(player);
  }
}

// Zoom
camera.viewfinder.zoom = 2.0; // zoom 2x

// Di chuyển camera tức thì
camera.moveTo(Vector2(500, 300));

// Shake effect
camera.shake(intensity: 10, duration: 0.5);
```

---

## 7. Game Overlays — Flutter Widget Trong Game

Overlay là Flutter widget bình thường hiển thị đè lên game canvas. Dùng cho: HUD, pause menu, game over screen.

```dart
// Khai báo overlay trong GameWidget
GameWidget(
  game: myGame,
  overlayBuilderMap: {
    'Hud': (context, game) => HudOverlay(game: game as MyGame),
    'Pause': (context, game) => PauseOverlay(game: game as MyGame),
    'GameOver': (context, game) => GameOverOverlay(game: game as MyGame),
  },
  initialActiveOverlays: const ['Hud'],
)

// Bật/tắt overlay từ trong game
game.overlays.add('Pause');     // hiện pause menu
game.overlays.remove('Pause');  // ẩn pause menu

// HUD overlay (hiển thị điểm, máu,...)
class HudOverlay extends ConsumerWidget {
  final MyGame game;
  const HudOverlay({super.key, required this.game});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final score = ref.watch(scoreProvider);
    final lives = ref.watch(livesProvider);

    return Padding(
      padding: const EdgeInsets.all(16),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Row(children: List.generate(lives, (_) => const Icon(Icons.favorite, color: Colors.red))),
          Text('$score', style: const TextStyle(fontSize: 24, color: Colors.white, fontWeight: FontWeight.bold)),
        ],
      ),
    );
  }
}
```

---

## 8. Particles & Effects

```dart
// Particle effect đơn giản
game.add(
  ParticleSystemComponent(
    particle: Particle.generate(
      count: 20,
      lifespan: 1.0,
      generator: (i) => AcceleratedParticle(
        speed: Vector2(
          (Random().nextDouble() - 0.5) * 200,
          -Random().nextDouble() * 300,
        ),
        acceleration: Vector2(0, 500), // gravity
        child: CircleParticle(
          radius: 4,
          paint: Paint()..color = Colors.yellow,
        ),
      ),
    ),
    position: explodePosition,
  ),
);

// Sprinkle effect khi collect coin
void collectCoin(Vector2 position) {
  game.add(
    ParticleSystemComponent(
      position: position,
      particle: Particle.generate(
        count: 8,
        lifespan: 0.6,
        generator: (i) {
          final angle = i * (2 * pi / 8);
          return MovingParticle(
            to: Vector2(cos(angle) * 40, sin(angle) * 40),
            child: CircleParticle(
              radius: 3,
              paint: Paint()..color = Colors.amber,
            ),
          );
        },
      ),
    ),
  );
}
```

---

## 9. Flame + Riverpod Integration

Flame game không phải Flutter widget nên cần cách đặc biệt để truy cập Riverpod.

```dart
// Inject ProviderContainer vào game
class MyGame extends FlameGame {
  final WidgetRef ref;
  MyGame({required this.ref});
}

// Trong GameScreen
class GameScreen extends ConsumerStatefulWidget {
  @override
  ConsumerState<GameScreen> createState() => _GameScreenState();
}

class _GameScreenState extends ConsumerState<GameScreen> {
  late MyGame _game;

  @override
  void initState() {
    super.initState();
    _game = MyGame(ref: ref); // truyền ref vào game
  }

  @override
  Widget build(BuildContext context) {
    return GameWidget(game: _game, ...);
  }
}

// Trong component — dùng game.ref để đọc/ghi state
class PlayerComponent extends PositionComponent with HasGameRef<MyGame> {
  void onCollectCoin() {
    gameRef.ref.read(scoreProvider.notifier).addScore(10);
  }

  void onHit() {
    gameRef.ref.read(livesProvider.notifier).loseLife();
    if (gameRef.ref.read(livesProvider) <= 0) {
      gameRef.overlays.add('GameOver');
      gameRef.paused = true;
    }
  }
}
```

---

## 10. Tiled Map — Bản Đồ Tile

```dart
// pubspec.yaml: flame_tiled: ^1.18.0

class TiledMapGame extends FlameGame {
  @override
  Future<void> onLoad() async {
    // Load file .tmx từ Tiled editor
    final tiledMap = await TiledComponent.load(
      'level_01.tmx',
      Vector2.all(16), // tile size
    );

    add(tiledMap);

    // Lấy object layer để spawn nhân vật
    final objectLayer = tiledMap.tileMap.getLayer<ObjectGroup>('Objects');
    for (final obj in objectLayer?.objects ?? []) {
      if (obj.type == 'SpawnPoint') {
        player.position = Vector2(obj.x, obj.y);
      }
      if (obj.type == 'Enemy') {
        add(EnemyComponent(position: Vector2(obj.x, obj.y)));
      }
    }

    // Lấy tile layer để tạo collision
    final collisionLayer = tiledMap.tileMap.getLayer<TileLayer>('Collision');
    // ...
  }
}
```

---

## Bài Tập Thực Hành

**Bài 1 — Component:** Tạo `FallingStarComponent` rơi từ trên xuống với tốc độ ngẫu nhiên, tự xóa khi ra khỏi màn hình.

**Bài 2 — Input:** Làm player di chuyển bằng touch drag. Đảm bảo player không ra khỏi màn hình.

**Bài 3 — Collision:** Tạo game đơn giản: tránh kẻ thù rơi từ trên xuống. Đếm điểm theo thời gian sống sót. Hiện game over overlay khi bị chạm.

**Bài 4 — Animation:** Tạo character với 3 state: `idle`, `running`, `jumping`. Đổi animation theo trạng thái khi player di chuyển.

---

*Tiếp theo: [04 · Riverpod](04_riverpod.md) — Quản lý State toàn ứng dụng*
