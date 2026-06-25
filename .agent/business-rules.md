# LUẬT NGHIỆP VỤ & CÔNG THỨC GAME — KÉO BÚA BAO

**LƯU Ý:** Đây là tài liệu tham chiếu chính cho lập trình viên và AI. Mọi thay đổi về cân bằng (Balance) phải được cập nhật tại đây.

## 1. Hệ thống Máu & Sát thương
- **HP khởi điểm:** `100` (cho cả người chơi và đối thủ/AI).
- **Damage cơ bản (Base):** `10` HP (khi thắng 1 ván RPS mà không có vật phẩm hỗ trợ).
- **Hòa (Tie):** Không ai mất HP từ RPS. Tuy nhiên, vật phẩm Trap vẫn có thể kích hoạt hiệu ứng tiêu cực cho người pick phải.

## 2. Hệ thống Vật phẩm & Điều kiện kích hoạt
| Loại | Ví dụ (Dân gian) | Hiệu ứng (Effect) | Điều kiện kích hoạt |
| :--- | :--- | :--- | :--- |
| **Tấn công** | Gậy gỗ, Búa sắt, Pháo đất | Cộng thêm `+5` đến `+15` damage vào sát thương cơ bản. | **CHỈ** có tác dụng nếu người chơi **THẮNG** ván RPS. |
| **Phòng thủ** | Nồi đất, Nón lá, Mẹt tre | Giảm `30%` - `50%` damage nhận vào (tùy cấp độ vật phẩm). | **CHỈ** có tác dụng nếu người chơi **THUA** ván RPS. |
| **Trap / Bẫy** | Vỏ chuối, Trứng thối, Bùn | Gây hiệu ứng tiêu cực (mất thêm `5-10` HP nhẹ hoặc hiệu ứng visual hài hước). | Kích hoạt **BẤT KỂ** thắng hay thua RPS. Người chơi chỉ bị nếu **PICK NHẦM** vào ô này. |

### Quy tắc "Pick nhầm" (Invalid Pick)
- **Thắng RPS mà pick Phòng thủ:** Vô tác dụng (bỏ lỡ cơ hội gây thêm sát thương).
- **Thua RPS mà pick Tấn công:** Vô tác dụng (không thể tấn công khi đang thua).
- **Pick vào ô Trap:** Luôn bị trừng phạt, không phụ thuộc kết quả RPS.

## 3. Cấu hình Level Scaling (Số ô vật phẩm)
Bàn vật phẩm mở rộng dựa trên Level của người chơi (hoặc cấp độ AI).

| Level | Số ô trên bàn | Nội dung/Thành phần |
| :--- | :--- | :--- |
| **1 → 3** | `2 ô` | 1 Tấn công + 1 Phòng thủ. (Phase dạy luật chơi) |
| **4 → 6** | `4 ô` | Thêm 1 ô Trap nhẹ. (Giới thiệu rủi ro) |
| **7 → 10** | `6 ô` | Thêm vật phẩm đặc biệt: Heal nhỏ (+5 HP) hoặc Hoán đổi vị trí (Swap). |
| **10+** | `8 ô+` | Bổ sung Trap mạnh hơn, khả năng kết hợp combo (cần cân bằng kỹ). |

## 4. Cơ chế Catch-up (Cân bằng cho người thua)
Để tránh cảm giác "mất kiểm soát" và tạo cơ hội lội ngược dòng:
- **Hiệu ứng:** Sau mỗi lượt **THUA**, người chơi được **xem trước** vị trí của **1 vật phẩm ngẫu nhiên** trên bàn cho lượt kế tiếp.
- *Lưu ý:* Tính năng này hiện đang ở trạng thái "Tạm bỏ qua" (GDD mục 2.2.5) để phát triển sau.

## 5. AI (Độ khó PvE)
- **AI Random:** Chọn ngẫu nhiên hoàn toàn. (Dùng cho MVP)
- **AI Trung bình:** Heuristic đơn giản. Có xu hướng lặp lại pattern dễ đoán.
- **AI Khó:** Pattern linh hoạt. Có khả năng chọn vật phẩm có chiến thuật (ví dụ: thường chọn Phòng thủ nếu biết sắp thua).

## 6. Meta-progression (Hệ thống Tiến trình)
- **Tiền (Coin):** Nhận được sau mỗi trận PvE/PvP. Dùng để mua Cosmetic (trang phục, nhân vật).
- **Kinh nghiệm (XP):** Tích lũy để lên Level. Mỗi Level mở khóa số ô vật phẩm mới.
- **Rank (Phase 3):** Hệ thống MMR (Elo) dùng để ghép trận PvP. Các cấp bậc: Đồng (Bronze) → Bạc (Silver) → Vàng (Gold) → Bạch kim (Platinum) → Kim cương (Diamond).

## 7. Nội dung "Sỉ nhục" (Slapstick) - Giới hạn
- ✅ Cho phép: Hiệu ứng visual hài hước, sticker chế có sẵn, animation bị dính bùn/trứng.
- ❌ Cấm: Văn bản tự do (free text) do người chơi nhập.
- ❌ Cấm: Hiệu ứng liên quan đến ngoại hình, giới tính, tôn giáo, hoặc đặc điểm cá nhân thật.