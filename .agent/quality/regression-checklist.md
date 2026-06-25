# Regression Checklist — Sau khi sửa bug hoặc refactor

> Chạy checklist này để đảm bảo không có thứ gì bị vỡ.

---

## Mức 1 — Sau mọi thay đổi (bắt buộc)

```bash
flutter analyze
flutter test test/unit/
```

```
[ ] 0 error từ flutter analyze
[ ] Tất cả unit test pass
```

---

## Mức 2 — Khi sửa game logic (RpsEngine, ItemService, GameProvider)

```bash
flutter test test/unit/rps_engine_test.dart
flutter test test/unit/item_service_test.dart
flutter test test/integration/game_flow_test.dart
```

Kiểm tra thủ công thêm:
```
[ ] Damage tính đúng: thắng = -10HP đối thủ, thua = -10HP mình
[ ] HP không xuống dưới 0
[ ] Round tăng đúng sau mỗi lượt
[ ] Game over đúng khi HP = 0
[ ] "Chơi lại" reset state về đúng initial
```

---

## Mức 3 — Khi sửa navigation hoặc router

```
[ ] Menu → Mode Select → Game Screen (không lỗi)
[ ] Game Screen → Result Screen (sau game over)
[ ] Result → "Chơi lại" → Game Screen (state reset)
[ ] Result → "Menu" → Main Menu
[ ] Back button hoạt động đúng ở mỗi màn
[ ] Deep link (nếu đã implement) vẫn đúng
```

---

## Mức 4 — Khi sửa Hive / Storage

```
[ ] Settings persist sau khi tắt/mở app
[ ] Không bị crash khi box chưa tồn tại (first launch)
[ ] Không bị crash khi data cũ không tương thích schema mới
[ ] Xóa app install lại → không crash
```

---

## Mức 5 — Trước khi release / kết thúc phase

Chạy toàn bộ:
```bash
flutter clean
flutter pub get
flutter analyze
flutter test
flutter build apk --debug
```

Manual test trên device thật:
```
[ ] Cài APK sạch (không có data cũ)
[ ] Chạy đủ 1 trận PvE từ đầu đến cuối
[ ] Tắt app giữa chừng → mở lại → không crash
[ ] Xoay màn hình (nếu không lock portrait) → không crash
[ ] Chơi 5 trận liên tiếp → không memory leak rõ ràng
```