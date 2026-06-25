# KIẾN TRÚC PHẦN MỀM — KÉO BÚA BAO

## 1. Tổng quan công nghệ (Tech Stack)
- **Frontend/Client:** Flutter (Dart) + Flame (Game Engine).
- **State Management:** Riverpod (Provider + StateNotifier + Future/AsyncNotifier).
- **Local Storage:** Hive (NoSQL, lưu offline: cài đặt, tiền, XP, lịch sử trận cục bộ).
- **Backend (Phase 3):** Firebase hoặc Supabase (Quyết định tại ADR-002).
  - **Auth:** Google/Apple Sign-In.
  - **Database:** Firestore / Supabase Postgres (Lưu hồ sơ, sync tiền/XP, lịch sử toàn cục).
  - **Realtime:** WebSocket (Socket.io) hoặc Firebase Realtime DB (Đồng bộ PvP, phòng chơi).

## 2. Kiến trúc phân lớp (Layered Architecture)
Áp dụng Clean Architecture thu gọn để tách biệt UI, Logic và Dữ liệu.
