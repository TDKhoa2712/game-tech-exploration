# Testing Policy — KBB

## Bắt buộc test những gì

| Loại code | Phải test | Không bắt buộc |
|---|---|---|
| Domain service (RpsEngine, ItemService) | ✅ Unit test | |
| Provider logic | ✅ Unit test | |
| Game flow end-to-end | ✅ Integration test | |
| Widget UI | | ✅ Optional |
| Flame components | | ✅ Optional |
| Screen layout | | ✅ Optional |

---

## Cấu trúc test

```
test/
├── unit/
│   ├── rps_engine_test.dart       # P1 — bắt buộc
│   ├── item_service_test.dart     # P1 — bắt buộc
│   ├── level_service_test.dart    # P1
│   └── ai_test.dart               # P2
├── widget/
│   └── hp_bar_test.dart           # optional
└── integration/
    └── game_flow_test.dart        # P1 — bắt buộc
```

---

## Chuẩn viết unit test

```dart
// Tên test: [method] — [điều kiện] — [kết quả mong đợi]
test('determineResult — rock vs scissors — trả về win', () {
  // Arrange
  final engine = RpsEngine();

  // Act
  final result = engine.determineResult(Gesture.rock, Gesture.scissors);

  // Assert
  expect(result, equals(RoundResult.win));
});

// Group các test liên quan
group('RpsEngine.calculateDamage', () {
  late RpsEngine engine;

  setUp(() { engine = RpsEngine(); });

  test('win + không có item → BASE_DAMAGE', () { ... });
  test('win + item attack → BASE_DAMAGE * 1.5', () { ... });
  test('thua + item defense → BASE_DAMAGE * 0.5', () { ... });
  test('hòa → 0 damage', () { ... });
});
```

---

## Test phải cover những case này

### RpsEngine
- [ ] 3 × 3 = 9 combination Gesture (rock/paper/scissors vs rock/paper/scissors)
- [ ] Hòa = 0 damage
- [ ] Win damage cơ bản = kBaseDamage
- [ ] Item attack tăng damage
- [ ] Item defense giảm damage khi thua
- [ ] HP không xuống dưới 0
- [ ] HP không vượt quá kHpStart

### ItemService
- [ ] generateItems trả về đúng số lượng theo level
- [ ] Trap chỉ kích hoạt khi đúng điều kiện
- [ ] canPickItem = false khi item đã revealed
- [ ] canPickItem = false khi phase sai

### GameFlow (integration)
- [ ] Menu → ModeSelect → GameScreen navigate thành công
- [ ] Pick item → phase chuyển sang pickGesture
- [ ] Pick gesture → reveal → round tăng
- [ ] HP về 0 → isGameOver = true
- [ ] GameOver → ResultScreen navigate
- [ ] ResultScreen → "Chơi lại" → reset state

---

## Quy tắc mock

```dart
// ✅ Mock AI để test deterministic
class MockAI implements PveAI {
  final Gesture fixedGesture;
  MockAI(this.fixedGesture);

  @override
  Gesture chooseGesture(GameState state) => fixedGesture;

  @override
  Item? chooseItem(GameState state) => null;
}

// Dùng trong test
final container = ProviderContainer(overrides: [
  pveAIProvider.overrideWithValue(MockAI(Gesture.rock)),
]);
```

---

## Lệnh chạy test

```bash
# Chạy tất cả
flutter test

# Chỉ unit test
flutter test test/unit/

# 1 file cụ thể
flutter test test/unit/rps_engine_test.dart

# Với coverage
flutter test --coverage
genhtml coverage/lcov.info -o coverage/html
```

---

## Ngưỡng coverage tối thiểu (Phase 1)

| Layer | Coverage tối thiểu |
|---|---|
| Domain services | 90% |
| Providers | 70% |
| Widgets | Không bắt buộc |

Nếu coverage domain < 90% → không được merge.