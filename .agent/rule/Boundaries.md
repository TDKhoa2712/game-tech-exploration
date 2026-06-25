# Ranh giới giữa các layer — Ai được gọi ai

## Ma trận phụ thuộc

| Từ ↓ Gọi → | core | domain | data | presentation | shared | external |
|---|---|---|---|---|---|---|
| **core** | — | ❌ | ❌ | ❌ | ✅ | ❌ |
| **domain** | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| **data** | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ |
| **presentation** | ✅ | ✅ | ❌ | ✅ | ✅ | ❌ |
| **shared** | ✅ | ❌ | ❌ | ❌ | ✅ | ✅ |

> ✅ = được phép · ❌ = VI PHẠM KIẾN TRÚC

---

## Ranh giới cụ thể theo file

### Domain layer — KHÔNG được import
```dart
// ❌ SAI — domain không được biết Flutter/Flame tồn tại
import 'package:flutter/material.dart';
import 'package:flame/game.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:hive/hive.dart';

// ✅ ĐÚNG — chỉ thuần Dart
import 'dart:math';
import '../models/gesture.dart';
import '../../core/constants.dart';
```

### Presentation layer — KHÔNG được gọi thẳng data
```dart
// ❌ SAI — screen gọi thẳng Hive
final box = Hive.box('settings');
box.put('sfx', true);

// ✅ ĐÚNG — qua provider
ref.read(settingsProvider.notifier).toggleSfx();
```

### Widget — KHÔNG được chứa logic
```dart
// ❌ SAI
class HpBarWidget extends StatelessWidget {
  Widget build(BuildContext context) {
    final damage = (100 - hp) * 1.5; // logic trong widget
    ...
  }
}

// ✅ ĐÚNG — tính toán ở provider, widget chỉ hiển thị
class HpBarWidget extends StatelessWidget {
  final int hp;        // nhận data đã tính sẵn
  final double ratio;  // provider tính, widget render
}
```

### Feature A — KHÔNG được import trực tiếp Feature B
```dart
// ❌ SAI — game import shop
import '../../shop/domain/shop_item.dart';

// ✅ ĐÚNG — nếu cần share, đưa lên shared/ hoặc dùng provider
```

---

## Ranh giới theo Phase

AI không được tự ý implement file của Phase sau:

| Phase | Files được phép chạm | Files cấm |
|---|---|---|
| P1 | `core/`, `game/`, `menu/`, `ai/random`, `shared/` | `pvp/`, `auth/`, `rank/`, `social/` |
| P2 | + `profile/`, `shop/`, `character/`, `tutorial/`, `ai/heuristic` | `pvp/`, `auth/`, `rank/` |
| P3 | Tất cả | — |

Nếu task P1 cần thứ gì của P2/P3 → tạo **interface/placeholder** thay vì implement thật.