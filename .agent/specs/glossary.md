
# BẢNG THUẬT NGỮ (GLOSSARY) — KÉO BÚA BAO

**Mục đích:** Thống nhất ngôn ngữ giữa các thành viên team (Product, Dev, Art) và AI để tránh hiểu nhầm.

| Thuật ngữ (Term) | Định nghĩa / Giải thích |
| :--- | :--- |
| **RPS** | Viết tắt của Rock-Paper-Scissors (Kéo-Búa-Bao). Trò chơi oẳn tù tì cơ bản. |
| **Input Randomness** | Ngẫu nhiên xảy ra **TRƯỚC** khi người chơi ra quyết định. Ví dụ: vị trí vật phẩm được xáo trộn trước khi người chơi pick. Giúp người chơi có cảm giác kiểm soát (dùng trí nhớ/suy luận). |
| **Output Randomness** | Ngẫu nhiên xảy ra **SAU** khi người chơi ra quyết định. Ví dụ: việc ra Kéo/Búa/Bao đối đầu với đối thủ là ngẫu nhiên (không thể biết trước). Cần được bù đắp bằng Input Randomness. |
| **Core Loop** | Vòng lặp hành động cốt lõi của game. Trong dự án này: 1. Chọn cử chỉ + Pick vật phẩm → 2. Reveal kết quả → 3. Tính sát thương & cập nhật HP → 4. Làm mới bàn vật phẩm. Lặp lại cho đến khi hết HP. |
| **Meta-progression** | Hệ thống tiến trình dài hạn bên ngoài mỗi trận đấu. Bao gồm: lên Level, tích lũy XP, kiếm tiền, mở khóa nhân vật/cosmetic, và Rank. Giúp giữ chân người chơi. |
| **Slapstick / Sỉ nhục** | Phong cách hài hước vật lý, cường điệu. Trong game, là các hiệu ứng visual/hài hước nhẹ nhàng (ví dụ: bị dính trứng, vỏ chuối) khi người chơi pick trúng Trap, nhằm giảm căng thẳng khi thua cuộc. |
| **Catch-up Mechanic** | Cơ chế hỗ trợ người đang thua để tạo cơ hội lội ngược dòng. Ví dụ: Sau khi thua, người chơi được xem trước 1 vị trí vật phẩm cho lượt sau. |
| **Trap / Bẫy** | Loại ô vật phẩm đặc biệt. Nếu người chơi pick trúng, họ sẽ chịu hiệu ứng tiêu cực (mất máu nhẹ hoặc hiệu ứng hài hước) BẤT KỂ thắng hay thua RPS. |
| **MMR / Elo** | Hệ thống điểm số dùng để đánh giá kỹ năng người chơi (Matchmaking Rating). Dùng để ghép cặp (matchmaking) các trận PvP cân bằng. |
| **Cosmetic** | Trang bị thẩm mỹ, không ảnh hưởng đến sức mạnh (Damage/HP). Ví dụ: Skin nhân vật, hiệu ứng Emote, Sticker. Tránh Pay-to-Win. |
| **Hive** | Local database (NoSQL) dùng trong Flutter. Ở dự án này, Hive dùng để lưu Settings (âm thanh, theme), Stats (thắng thua), và inventory offline trước khi sync lên server. |
| **Riverpod** | Thư viện quản lý state cho Flutter. Ở đây dùng để kết nối UI (Flutter Widgets) với Game Logic (Domain) và Data (Hive/Backend). |
| **Flame** | Game Engine nhẹ chạy trên Flutter. Dùng để xử lý vòng lặp game (Game Loop), vẽ Animation, quản lý các Component (nhân vật, vật phẩm) và overlay trong trận đấu. |
| **Reveal** | Pha "lật bài" đồng thời. Cả người chơi và đối thủ cùng thấy kết quả RPS và vị trí vật phẩm mình đã chọn (thường kèm hiệu ứng animation). |
| **Boarding / Onboarding** | Quá trình người chơi học cách chơi. Ở game này, Level 1-3 được thiết kế để dạy luật chơi một cách tự nhiên (không có màn hướng dẫn riêng biệt cho người mới). |