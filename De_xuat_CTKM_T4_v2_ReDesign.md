# ĐỀ XUẤT CTKM T4/2026 — v3 REDESIGN

**Ngày:** 2026-04-13
**Dữ liệu:** Data lũy kế T4 (13 ngày, đến 2026-04-13)
**Mục đích:** Redesign 3 CTKM theo feedback — targeted cụ thể, không tăng AOV SC (chỉ Relax / UND / SP), nới tiêu chí NV để nhiều salon chạy được, tính cụ thể mục tiêu tăng AOV theo nhóm salon.

> **Nguyên tắc chung:** Mỗi CT phải trả lời được 4 câu hỏi
> 1. Nhắm vào **NHÓM KH** nào?
> 2. Chạy tại **SALON** nào?
> 3. **NHÂN SỰ** nào đủ khả năng chạy (không drop CR)?
> 4. Mục tiêu **AOV tăng** bao nhiêu (Relax / UND / SP)?

---

## HIỆN TRẠNG — INSIGHT TỪ DATA T4 (13 NGÀY)

### Bảng baseline 3 mảng không phải SC

| Chỉ số | Median hệ thống | Min | Max | Gap |
|--------|----------------|-----|-----|-----|
| **AOV Relax** | ~120K | 73K (58 NVC NA) | 172K (39 HBT, 595 BT TH) | 2.4x |
| **AOV UND** | ~26K | 9K (4 LQD BN) | 45K (1146 KVC) | 5.0x |
| **AOV SP** | ~13K | 3K (7 DTP LA) | 28K (145 THT) | 9.3x |
| **AOV SP tạo kiểu** (sáp/gel/xịt) | ~6.5K | 1.7K (236 DBT) | 16.3K (36 N1 BD) | 9.6x |
| **CR SP** | ~4.0% | 1.1% (7 DTP LA) | 10.9% (32 NAT, 145 THT) | 10x |
| **CR UND** | ~7.5% | 2.6% (4 LQD BN) | 13.6% (1146 KVC) | 5.2x |
| **BS SP** | ~300K | 166K (708 LTT) | 450K (36 N1 BD) | — |
| **BS UND** | ~320K | 228K (7 DTP LA) | 426K (68 LN NA) | — |

Gap giữa Max/Min rất lớn → **phần lớn salon còn room tăng AOV** mà không cần kéo CR toàn hệ.

---

## CT1 — SÁP GLANZEN PRIME SANDALWOOD (2 tầng, tập trung AOV SP tạo kiểu)

### Mục tiêu
- **AOV nhóm SP tạo kiểu** (sáp/gel/xịt) tại salon triển khai: từ ~6.5K/KH → **10K/KH** (+54%)
- **CR SP tạo kiểu** riêng (không phải CR SP tổng): từ ~2% → **3.5%** (+1.5pts)
- Không drop CR SP hiện tại trên các nhóm SP khác (skincare, haircare)

### Thông điệp mạnh

> **"Tóc khoẻ từ bàn tay stylist — giữ kiểu cả ngày với sáp thế hệ mới"**

Stylist vuốt thử sáp lên tóc KH sau khi cắt → pitch trực tiếp → offer mua tại quầy.
**Không có mẫu 10g tặng** — mọi offer đều gắn với giao dịch mua.

### Phân tầng mức giảm (2 tầng)

| Tầng | Nhóm KH | Nhận diện từ CRM | Mức giảm | Giá bán | Lý do |
|------|---------|------------------|----------|---------|-------|
| **A** | KH đã mua sáp/gel/xịt trong 6 tháng gần nhất | `last_styling_purchase <= 180d` | **10%** | 251,100đ | Đã có thói quen — giữ margin |
| **B** | KH chưa từng mua SP tạo kiểu, hoặc không mua trong 6 tháng | `last_styling_purchase > 180d OR NULL` | **20%** | 223,200đ | Acquisition — đổi trial lấy LTV |

> **Ước tính tỷ lệ:** Tầng A ~30%, Tầng B ~70% (dựa theo % KH đã mua sáp trong 6 tháng).

### Cost & margin

```
COGS sáp Glanzen:   75,000đ
Giá gốc:           279,000đ

Tầng A (giảm 10% = 251,100đ):
  Margin/lượt:    176,100đ    Margin Rate: 70.1%
Tầng B (giảm 20% = 223,200đ):
  Margin/lượt:    148,200đ    Margin Rate: 66.4%
```

### Salon triển khai — nới tiêu chí để có 30+ salon

| Tiêu chí (trước) | Tiêu chí (sau, nới) |
|------------------|---------------------|
| Stylist cấp 4+ có cert SP | Stylist Level 2+ có điểm NL ≥ 4.0 (xếp loại A/B) **HOẶC** có chứng chỉ SP |
| Mix SC ≥ 35% | Bỏ điều kiện mix — tất cả salon triển khai được |
| 15 salon chọn lọc | **30-40 salon** có ≥ 3 stylist đủ tiêu chí |

Ưu tiên top 20 salon hiện có **AOV SP tạo kiểu thấp < 6K** (để tạo uplift lớn):

| Salon | ASM | AOV SP tạo kiểu hiện tại | Target sau CT |
|-------|-----|--------------------------|----------------|
| 236 DBT | Tống Xuân Quyền | 1.7K | 4-6K |
| 39 HBT | Phan Thị Hồng Gấm | 1.7K | 4-6K |
| 190 HB | Nguyễn Thanh Sang | 2.2K | 5-7K |
| 341 XVNT | Tống Xuân Quyền | 2.7K | 6-8K |
| 363 LNQ TN | Phan Thị Hồng Gấm | 2.7K | 6-8K |
| 7 DTP LA | Tống Xuân Quyền | 3.0K | 6-8K |
| 730 TL10 | Nguyễn Thanh Sang | 3.0K | 6-8K |
| 304 NT TH | Nguyễn Thị Huệ | 3.1K | 6-8K |
| 90 NH | Đoàn Trung Đức | 3.7K | 7-9K |
| 595 BT TH | Nguyễn Thị Huệ | 3.9K | 7-9K |
| 168 NVC | Nguyễn Như Quỳnh | 4.0K | 7-9K |
| 25 TD | Tống Xuân Quyền | 4.3K | 7-9K |
| 575 AC | Nguyễn Thanh Sang | 4.5K | 8-10K |
| 8 CVL | Nguyễn Thanh Sang | 4.6K | 8-10K |
| 290 QT TH | Nguyễn Thị Huệ | 4.7K | 8-10K |
| 148 TD | Phan Thị Hồng Gấm | ~5K | 8-10K |
| 50BT4 NPC LD | Phan Thị Hồng Gấm | ~5.4K | 8-10K |
| 99 TSN | Nguyễn Thanh Sang | ~5.5K | 8-10K |
| 420 HTP | Tống Xuân Quyền | ~5.7K | 8-10K |
| 202 DC | Phan Thị Hồng Gấm | ~5.7K | 8-10K |

### Nhân sự — tiêu chí nới

| Vị trí | Điều kiện (nới) | Nội dung brief |
|--------|-----------------|----------------|
| **Stylist** | Level 2+ điểm NL ≥ 4.0 | Brief 30 phút: vuốt thử, pitch 2 tầng A/B, xử lý từ chối |
| **Thu ngân** | Biết đọc CRM → áp đúng tầng A/B | Training 15 phút |
| **SM** | Monitor AOV SP tạo kiểu mỗi 3 ngày | Dashboard tuần, brief lại nếu CR SP drop > 2pts |

### KPI CT1

| Chỉ số | Target | Kill Switch |
|--------|--------|-------------|
| Số lượt dùng CTKM | ≥ 5 lượt/salon/tuần × 30 salon × 4 tuần = 600 lượt | < 250 lượt |
| **AOV SP tạo kiểu** (toàn nhóm salon) | Tăng từ 6.5K → 10K (+54%) | Tăng < 15% |
| **CR SP tạo kiểu** | Từ 2% → 3.5% (+1.5pts) | Drop > 0.5pts |
| Margin Rate sau KM | ≥ 66% (đã check) | < 20% |
| Tỷ lệ tầng A / B | ~30% / 70% | — |
| CR SP tổng (bảo vệ) | ≥ baseline 30d trước | Drop > 2pts |

---

## CT2 — RELAX TẶNG SP CẬN DATE (3 bậc bill, tính chi tiết lãi lỗ)

### Vấn đề cần giải quyết
Phiên bản trước: tặng SP có vẻ hào phóng nhưng **chưa tính xem DT dịch vụ KH upgrade có bù được cost SP tặng hay không**.

### Nguyên tắc tài chính

```
LNG dịch vụ Relax = 60%
→ Với bill Relax = B, margin = 0.6 × B
→ Sau tặng SP (cost c), margin thực = 0.6B − c
→ Margin Rate = (0.6B − c) / B

Điều kiện Margin Rate ≥ 40% (Rule v3 DV):
  0.6B − c ≥ 0.4B
  c ≤ 0.2 × B   ← COST SP TẶNG TỐI ĐA = 20% BILL
```

### Bậc bill Relax và mức tặng tương ứng

| Bậc | Bill Relax | Cost SP tặng tối đa | Giá bán SP (cost 35%) | Gói SP cận date |
|-----|-----------|---------------------|----------------------|----------------|
| **Cơ bản** | 150K–250K | ≤ 50K | 150K–200K | 1 chai dầu gội L'Orsia / sáp nhỏ / serum mẫu |
| **Trung** | 250K–400K | ≤ 80K | 200K–300K | 1 dầu gội tea tree / serum dưỡng / tinh dầu |
| **Cao cấp** | ≥ 400K | ≤ 120K | 300K–500K | 1 kem tẩy tế bào chết / 1 combo 2 SP cận date |

> **Kiểm tra lãi lỗ mẫu (bậc Trung):**
> - KH đang dùng Relax 150K → upgrade lên 300K (bill tăng +150K)
> - Margin thêm từ upgrade: 150K × 60% = **+90K**
> - Cost SP tặng: 80K
> - **Lãi ròng thêm/KH: +10K**

### Phân theo 3 nhóm salon (theo AOV Relax baseline)

#### Nhóm S1 — Salon AOV Relax CAO (≥ 140K): ĐẨY BẬC CAO CẤP

**Salon target:** 39 HBT (172K), 595 BT TH (172K), 36 N1 BD (154K), 80 PL BD (150K), 1180 QT (150K), 12 LDT (148K), 363 LNQ TN (148K), 575 AC (149K), 113 THD AG (152K), 382 NT (141K), 50BT4 NPC LD (143K), 168 NVC (144K), 25 TD (142K), 10 TP + 168 NVC + 1146 KVC (136K+), 451 PVT (135K)

**Mục tiêu AOV Relax nhóm này:** **Từ ~145K → 170K (+17%)** — đã cao, target +17% đủ tham vọng nhưng khả thi

**Mechanic:** KH mua Shinecombo 3/4 hoặc Gội VIP (bill ≥ 400K) → tặng SP cận date 300-500K (cost 120K)

**Target lượt:** 6 lượt/salon/tuần × ~15 salon × 4 tuần ≈ 360 lượt

#### Nhóm S2 — Salon AOV Relax TRUNG (100–140K): ĐẨY BẬC TRUNG

**Salon target:** 109 TQH (119K), 116 TK (137K), 145 THT (134K), 173 TN (126K), 186 DTH (116K), 190 HB (112K), 202 DC (117K), 237 NTT (105K), 255 NAN (132K), 264 LLQ (111K), 312 LVS (123K), 32 NAT (125K), 341 XVNT (106K), 346 KT (121K), 362 NGT BN (128K), 401/1 NDT (114K), 483 TN (134K), 65 CD (136K), 76 PVH (118K), 80 TP TH (112K), 927 HG (145K gần biên), 99 TSN (125K), 103 TN (111K), 73 PL BD, 90 NH (128K), 55A NVL (114K), 29 HB (126K), 186 QT (104K), 290 QT TH (114K), 304 NT TH (109K), 68 LN NA (73K thấp), 8 CVL (103K), 708 LTT (117K), 730 TL10 (108K)

**Mục tiêu AOV Relax nhóm này:** **Từ ~118K → 140K (+19%)**

**Mechanic:** KH mua Relax CVG / Gội dưỡng sinh / Relax mini (bill ≥ 250K) → tặng SP cận date 200-300K (cost 80K)

**Target lượt:** 5 lượt/salon/tuần × ~30 salon × 4 tuần ≈ 600 lượt

#### Nhóm S3 — Salon AOV Relax THẤP (< 100K): ĐẨY BẬC CƠ BẢN

**Salon target:** 148 TD (104K), 11 PKT (85K), 62 LH (98K), 7 DTP LA (87K), 68 LN NA (73K), 58 NVC NA (78K)

**Mục tiêu AOV Relax nhóm này:** **Từ ~85K → 110K (+29%)** — gap lớn hơn

**Mechanic:** KH mua Gội + add Relax mini ngắn (bill ≥ 150K) → tặng SP cận date 150-200K (cost 50K)

**Target lượt:** 3 lượt/salon/tuần × 6 salon × 4 tuần ≈ 72 lượt

### Nhân sự — nới tiêu chí

| Vị trí | Điều kiện (nới) |
|--------|-----------------|
| **Skinner/Stylist Relax** | Level 1+ điểm NL ≥ 3.5 (xếp loại B+), đã training massage căn bản |
| **Lễ tân / Thu ngân** | Biết pitch combo tại quầy bill |
| **SM** | Check cost SP tặng/tuần, không vượt 5% DTT salon |

### KPI CT2

| Chỉ số | Target toàn CT | Kill Switch |
|--------|---------------|-------------|
| Tổng lượt CT2 (3 nhóm) | ~1,030 lượt | < 400 lượt |
| AOV Relax nhóm S1 (cao) | +17% (145K → 170K) | < +5% |
| AOV Relax nhóm S2 (trung) | +19% (118K → 140K) | < +5% |
| AOV Relax nhóm S3 (thấp) | +29% (85K → 110K) | < +10% |
| **Lãi ròng/lượt CTKM** (margin thêm − cost SP) | ≥ +10K/lượt | Âm (lỗ) |
| % tồn SP cận date 0-3T đã xả | ≥ 80% cuối T4 | < 40% sau 2 tuần |
| CR Relax (bảo vệ) | Không drop | Drop > 3pts |

---

## CT3 — UỐN TẶNG DẦU GỘI BOND, NHUỘM TẶNG DẦU GỘI TÍM

### Nguyên tắc tính bill ngưỡng

**Input:**
- LNG uốn/nhuộm ban đầu: **60%** (margin gốc)
- Cost SP tặng: **140,000đ/SP** (Bond hoặc Tím)
- Margin tối thiểu sau KM: **40%** (Rule v3)

**Tính toán:**

```
Với bill UND = B, margin gốc = 0.6 × B
Sau tặng 1 SP (cost 140K): margin còn = 0.6B − 140
Điều kiện margin ≥ 40% bill:
  0.6B − 140 ≥ 0.4B
  0.2B ≥ 140
  B ≥ 700,000đ
```

**Vậy bill ngưỡng tặng 1 SP (Bond HOẶC Tím) = 700K.**

Với combo 2 SP (Bond + Tím, cost 280K):
```
0.6B − 280 ≥ 0.4B → B ≥ 1,400,000đ
```

### Cơ chế 3 gói

| Gói | Điều kiện bill | Tặng | Cost 30Shine | Giá trị cảm nhận KH |
|-----|---------------|------|--------------|---------------------|
| **Uốn chuẩn** | Bill uốn ≥ 700K | Laborie Bond 300ml | 140K | ~450K |
| **Nhuộm chuẩn** | Bill nhuộm ≥ 700K | Laborie Tím 300ml | 140K | ~400K |
| **Combo Uốn + Nhuộm** | Bill UND ≥ 1,400K | Bond + Tím | 280K | ~850K |

### Lãi/lỗ từng gói

| Gói | Bill | Margin gốc | Cost SP | Margin sau | Margin Rate |
|-----|------|-----------|---------|-----------|-------------|
| Uốn chuẩn | 700K | 420K | 140K | **280K** | **40.0%** ✓ |
| Uốn chuẩn | 900K | 540K | 140K | **400K** | **44.4%** ✓ |
| Nhuộm chuẩn | 700K | 420K | 140K | **280K** | **40.0%** ✓ |
| Combo | 1,400K | 840K | 280K | **560K** | **40.0%** ✓ |
| Combo | 1,800K | 1,080K | 280K | **800K** | **44.4%** ✓ |

### Tính khả thi bill UND hiện tại

Từ data T4 (13 ngày):
- **BS UND trung bình hệ thống:** ~320K
- **BS UND cao nhất:** 426K (68 LN NA), 425K (80 PL BD), 423K (58 NVC NA)

Bill ngưỡng 700K cao hơn BS UND trung bình. **Đây chính là điểm tốt:** CT kích KH upgrade gói UND cao cấp (nhuộm highlight, uốn cao cấp, combo uốn + nhuộm + dưỡng), không chỉ làm UND cơ bản.

### Salon triển khai — nới tiêu chí rộng

| Tiêu chí (trước) | Tiêu chí (sau, nới) |
|------------------|---------------------|
| Stylist nhuộm cấp 3+ có cert Laborie | Stylist Level 2+ điểm NL ≥ 4.0 (xếp loại A) đã qua training Laborie 1 buổi |
| UND < 8% DTT, AOV Relax > 130K | Bỏ điều kiện mix DTT. Mọi salon có ≥ 2 stylist đủ tiêu chí |
| 12 salon | **~40 salon** triển khai |

**Salon ưu tiên top 15 có AOV UND hiện thấp + BS UND cao (dễ upsell lên 700K):**

| Salon | ASM | BS UND | AOV UND | Cơ hội |
|-------|-----|--------|---------|--------|
| 80 PL BD | Nguyễn Việt Cường | 425K | 30K | Upsell lên 700K+ (cần +65% bill) |
| 58 NVC NA | Nguyễn Thị Huệ | 423K | 23K | Upsell lên 700K+ |
| 68 LN NA | Nguyễn Thị Huệ | 426K | 17K | Upsell (CR UND thấp, cần kích) |
| 401/1 NDT | Nguyễn Việt Cường | 402K | 20K | Upsell |
| 451 PVT | Nguyễn Việt Cường | 407K | 28K | Upsell |
| 304 NT TH | Nguyễn Thị Huệ | 412K | 37K | Upsell |
| 290 QT TH | Nguyễn Thị Huệ | 401K | 31K | Upsell |
| 236 DBT | Tống Xuân Quyền | 403K | 21K | Upsell |
| 25 TD | Tống Xuân Quyền | 393K | 42K | Upsell (CR UND 10.7%, đã quen UND) |
| 927 HG | Nguyễn Thanh Sang | 383K | 30K | Upsell |
| 312 LVS | Nguyễn Thanh Sang | 383K | 35K | Upsell |
| 1146 KVC | Nguyễn Việt Cường | 331K | 45K | CR UND cao nhất, upsell gói cao cấp |
| 8 CVL | Nguyễn Thanh Sang | 365K | 26K | Upsell |
| 99 TSN | Nguyễn Thanh Sang | 365K | 30K | Upsell |
| 80 TP TH | Nguyễn Thị Huệ | 348K | 17K | Upsell |

Ngoài top 15 trên, ~25 salon khác đủ điều kiện chạy CT với target thấp hơn.

### Mục tiêu AOV UND theo nhóm salon

| Nhóm | Salon | AOV UND hiện | Target AOV UND | Uplift |
|------|-------|-------------|----------------|--------|
| **UND-HIGH** (CR UND ≥ 10%, BS UND ≥ 380K) | 25 TD, 1146 KVC, 927 HG, 382 NT, 103 TN, 116 TK, 25 TD, 312 LVS | ~32K | **45K** | +40% |
| **UND-MID** (CR UND 5-10%, BS UND 300-380K) | ~20 salon | ~23K | **32K** | +40% |
| **UND-LOW** (CR UND < 5%, BS UND < 300K) | ~12 salon | ~13K | **20K** | +54% |

### Nhân sự — nới tiêu chí

| Vị trí | Điều kiện (nới) | Training |
|--------|-----------------|----------|
| **Stylist nhuộm** | Level 2+ điểm NL ≥ 4.0 (xếp loại A) | Training Laborie 1 buổi (demo + pitch) |
| **Stylist uốn** | Level 2+ điểm NL ≥ 4.0 | Training tương tự |
| **Tư vấn** | Biết phân loại bill 700K / 1.4M | Training 20 phút: đọc bill → bật gói |
| **SM** | Monitor % KH qualify gói CTKM / tổng UND | Weekly check |

### KPI CT3

| Chỉ số | Target | Kill Switch |
|--------|--------|-------------|
| Số gói CTKM / salon / tuần | ≥ 3 gói × 40 salon × 4 tuần = 480 gói | < 200 gói |
| AOV UND nhóm UND-HIGH | +40% (32K → 45K) | < +10% |
| AOV UND nhóm UND-MID | +40% (23K → 32K) | < +10% |
| AOV UND nhóm UND-LOW | +54% (13K → 20K) | < +15% |
| Tỷ lệ upsell lên gói Combo (1.4M+) | ≥ 15% tổng gói CTKM | < 5% |
| Margin Rate sau KM (tất cả gói) | ≥ 40% (kiểm thủ công) | < 30% |
| Tồn Laborie Bond/Tím | Giảm theo plan kho | — |
| CR UND (bảo vệ) | Không drop | Drop > 2pts |

---

## TÓM TẮT 3 CT v3

| | CT1 Sáp Glanzen | CT2 Relax + SP cận date | CT3 UND + Laborie |
|---|---|---|---|
| **Mảng target** | SP tạo kiểu | Relax | UND (uốn + nhuộm) |
| **Nhóm KH** | Tầng A (đã mua) / B (chưa mua) | Theo bậc bill Relax | KH upgrade bill UND ≥ 700K |
| **Salon** | ~30-40 salon (nới Stylist L2+, NL ≥ 4.0) | ~50 salon chia 3 nhóm S1/S2/S3 | ~40 salon (nới, Stylist L2+ A) |
| **Nhân sự** | Stylist L2+ NL ≥ 4.0 hoặc cert SP | Skinner/Stylist L1+ NL ≥ 3.5 | Stylist L2+ NL ≥ 4.0 + training Laborie |
| **Mục tiêu AOV** | SP tạo kiểu: 6.5K → 10K (+54%) | Relax: +17% đến +29% theo nhóm | UND: +40% đến +54% theo nhóm |
| **CR protection** | CR SP không drop > 2pts | CR Relax không drop > 3pts | CR UND không drop > 2pts |
| **Margin Rate ≥** | 66% | 40% | 40% |
| **Target lượt T4** | 600 lượt | 1,030 lượt | 480 gói |

---

**Người đề xuất:** BP Thương mại
**Ngày:** 2026-04-13
**Trạng thái:** Draft v3 — trình sếp review
