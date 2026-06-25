# Feature Policy — Thêm chức năng mới

## Quy trình bắt buộc (theo thứ tự)

```
Bước 1 → Xác định phạm vi
Bước 2 → Kiểm tra phase
Bước 3 → Thiết kế interface trước
Bước 4 → Viết test trước (TDD)
Bước 5 → Implement
Bước 6 → Kiểm tra không break cái cũ
Bước 7 → Báo cáo
```

---

## Bước 1 — Xác định phạm vi

AI phải trả lời trước khi code:

- Feature này cần tạo file mới nào?
- Feature này cần **sửa** file cũ nào?
- Feature này có ảnh hưởng đến `GameState` không?
- Feature này có thêm route mới không?

Nếu sửa > 3 file cũ → hỏi lại người dùng trước khi tiến hành.

---

## Bước 2 — Kiểm tra phase

Đọc `architecture/boundaries.md` → xác nhận feature thuộc phase nào.

Nếu feature thuộc P2/P3 mà đang làm P1:
- Tạo **interface/abstract class** + comment `// TODO P2: implement`
- **Không implement logic thật**

---

## Bước 3 — Thiết kế interface trước

Trước khi viết implementation, AI phải định nghĩa:

```dart
// Viết signature trước, chưa có body
abstract class NewFeatureService {
  Future<Result> doSomething(Input input);
}

// Provider trả về gì
@riverpod
NewFeatureState newFeature(NewFeatureRef ref);
```

Hỏi người dùng confirm interface trước khi implement.

---

## Bước 4 — Viết test trước

```dart
// Viết test mô tả behavior mong muốn TRƯỚC khi có code
test('Khi player pick item attack, damage tăng 50%', () {
  final engine = RpsEngine();
  final result = engine.calculateDamage(
    result: RoundResult.win,
    playerItem: Item.attack(),
  );
  expect(result, equals(kBaseDamage * 1.5));
});
```

Test phải fail trước (vì chưa có code) → sau đó implement cho test pass.

---

## Bước 5 — Implement

Thứ tự implement:
1. Domain model (nếu có model mới)
2. Service/Engine (logic)
3. Repository (nếu cần persist)
4. Provider
5. Widget/Screen

Không skip bước nào, không implement từ UI xuống.

---

## Bước 6 — Kiểm tra không break cái cũ

Sau khi implement xong, chạy theo thứ tự:

```bash
flutter analyze
flutter test test/unit/
flutter test test/integration/
```

Nếu có test fail → **fix trước khi báo cáo done**, không để nợ.

---

## Bước 7 — Báo cáo

```
## Feature hoàn thành: [tên feature]

Files tạo mới:
- path/to/new_file.dart

Files đã sửa:
- path/to/existing.dart — lý do sửa

Tests đã viết:
- test/unit/xxx_test.dart — [N] test cases

Kết quả: flutter test ✅ [N] passed, 0 failed

Cần chú ý:
- [ghi chú nếu có thứ tạm thời hoặc sẽ thay đổi ở phase sau]
```

---

## Feature mới KHÔNG được làm

- ❌ Thêm field vào `GameState` mà không update `copyWith` và `==`
- ❌ Thêm route mới mà không khai báo trong `core/router.dart`
- ❌ Hard-code string hiển thị (dùng constants hoặc l10n key)
- ❌ Tự tạo Singleton mới khi chưa hỏi (đã có pattern qua Riverpod)