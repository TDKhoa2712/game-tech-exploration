# BỐI CẢNH DỰ ÁN — KÉO BÚA BAO: SINH TỒN VẬT PHẨM

## 1. Tổng quan sản phẩm
- **Tên dự án:** Kéo Búa Bao: Sinh tồn Vật phẩm (Rock-Paper-Scissors: Item Survival).
- **Loại game:** Đối kháng theo lượt (turn-based), 2 người chơi (PvP/PvE), thời gian thực.
- **Nền tảng:** Mobile (Android & iOS) - Flutter.
- **Thời lượng mỗi trận:** 1-3 phút.
- **Concept:** Đối kháng Kéo-Búa-Bao theo thời gian thực, nơi thắng thua không chỉ phụ thuộc vào lựa chọn cử chỉ, mà còn vào khả năng ghi nhớ vị trí vật phẩm ẩn trên bàn đấu, sự phản xạ để khuếch đại sát thương hoặc tự bảo vệ --- pha trộn yếu tố hài hước "sỉ nhục" nhẹ nhàng và hệ thống tiến trình để giữ chân người chơi lâu dài.

## 2. Mục tiêu và vấn đề cốt lõi
- **Vấn đề:** Phần lớn game di động bị gỡ bỏ sớm do:
  1. Phụ thuộc vào "Button Mashing" (nhấn nhanh) -> mỏi tay, loại trừ người chơi kém vận động.
  2. Dựa hoàn toàn vào Ngẫu nhiên Đầu ra (Output Randomness) -> kết quả không liên quan đến quyết định của người chơi, gây bất công.
  3. Thiếu vòng lặp tiến trình dài hạn (Meta-progression) -> chán sau vài ván.
- **Mục tiêu:** Chuyển hóa Kéo Búa Bao từ "cầu may" thành kỹ năng ghi nhớ không gian, quản lý rủi ro và đọc tâm lý đối thủ (Input Randomness), đồng thời đáp ứng 3 nhu cầu tâm lý nền tảng của Thuyết Tự quyết (SDT): Tự chủ, Năng lực, Gắn kết.

## 3. Đối tượng người chơi (Target Audience)
- **Casual:** Tìm trò chơi nhanh, vui, chơi trong 1-3 phút.
- **Social/Party:** Thích game hài hước (Among Us, Fall Guys), chơi cùng bạn bè.
- **Competitive:** Muốn hệ thống PK, xếp hạng, phần thưởng để đầu tư lâu dài.

## 4. Phạm vi & Lộ trình (Roadmap)
Dựa trên kế hoạch dự án (v3 Cross-Training), lộ trình được chia làm 3 Phase, tổng thời gian **~19 tuần**:

- **Phase 1 - MVP (Tuần 1-4):**
  - Mục tiêu: Chơi được trận PvE đầu tiên (Core Loop).
  - Chức năng: Chọn RPS, Bàn 2 vật phẩm (Tấn công/Phòng thủ), HP 100, AI Random, Menu cơ bản.
- **Phase 2 - Alpha (Tuần 5-13):**
  - Mục tiêu: Phong phú & Hấp dẫn hơn.
  - Chức năng: Level Scaling (4-10+ ô), Trap, AI 3 cấp độ, Âm thanh, Animation, Nhân vật, Cửa hàng (Cosmetic), Meta-progression (XP/Tiền).
- **Phase 3 - Beta → Release (Tuần 14-19):**
  - Mục tiêu: PvP người thật, Rank & Hoàn thiện.
  - Chức năng: Đăng nhập (Google/Apple), Sync dữ liệu Server, Phòng riêng, Ghép trận MMR, Bảng xếp hạng, Chia sẻ, CI/CD & Lên Store.

## 5. Nguyên lý thiết kế chỉ đạo (Design Principles)
1. **Khước từ Button Mashing:** Mọi thao tác yêu cầu chiến lược, định thời gian, quản lý rủi ro.
2. **Quản trị thất bại qua Hài hước (Slapstick):** Dùng hiệu ứng visual hài hước để chuyển hóa cảm giác thua cuộc thành niềm vui, giảm cortisol.
3. **Bổ sung Input Randomness:** Vị trí vật phẩm được xáo trộn trước (ẩn), người chơi dùng trí nhớ và suy luận để chọn, giảm cảm giác may rủi thuần túy.
4. **Cân bằng (Balance):** Người thua không thể pick đồ công, người thắng không thể pick đồ thủ (tránh tình trạng "snowball" quá mức).