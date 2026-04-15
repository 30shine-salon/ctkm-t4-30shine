# PROMPT PHÂN TÍCH CHI PHÍ NHÂN SỰ (CPNS) — 30SHINE
**Version 5 | Cập nhật thêm góc phân tích Đơn Giá Thưởng Ăn Chia**

> **Thay đổi v5 so với v4:**
> - Bổ sung file đầu vào mới: `Query_1B_Don_gia_thuong_an_chia.xlsx` (từ PART B của SQL v2)
> - Thêm Data Dictionary cho bảng đơn giá (Mục II.C)
> - Thêm driver phân tích mới: Mục 2E — Đơn Giá Thưởng Ăn Chia
> - Cập nhật Mục 3.2 (salon vượt ngưỡng): thêm root cause "Đơn giá cao hơn chuẩn"
> - Thêm hành động chuẩn hóa đơn giá vào Mục 5

---

## I. BỐI CẢNH & FILE ĐẦU VÀO

**Hệ thống:** 30Shine — ~60 salon công ty (`IsFranchise=0`, `brand_id=1`), phân thành 8 vùng GDKV: Nguyễn Thanh Sang, Nguyễn Như Quỳnh, Nguyễn Thị Huệ, Phan Thị Hồng Gấm, Tống Xuân Quyền, Đoàn Trung Đức, Trịnh Văn Quang, Nguyễn Việt Cường. *(Vùng Hoàng Nguyên Hội đã giải thể — salon chuyển về vùng Nguyễn Như Quỳnh.)*

### File đầu vào mỗi kỳ

| File | Nội dung | Tần suất |
|---|---|---|
| `Query_1A_Chi_phi_luong_truc_tiep_[ky].xlsx` | Lương ST/SK/SUP chi tiết (Part A) | Mỗi kỳ |
| `Query_1B_Don_gia_thuong_an_chia.xlsx` | Đơn giá thưởng ăn chia toàn hệ thống (Part B) | Khi điều tra / mỗi tháng |
| `Query_2_DTT_chi_tiet_[ky].xlsx` | DTT theo nhóm dịch vụ | Mỗi kỳ |
| `Query_1A_..._ky_truoc.xlsx` + `Query_2_..._ky_truoc.xlsx` | Dữ liệu kỳ trước để so sánh | Mỗi kỳ |
| `Mục_tiêu_CPNS_[thang].xlsx` | Định mức %/DTT tháng hiện tại | Mỗi tháng |
| Tạm tính BV/SM/RSM/TTCL/BHXH | User cung cấp trực tiếp trong tin nhắn (xem Mục II.D) | Mỗi kỳ |

> **Lưu ý đặt tên file v5:** Query_1 cũ (v4) đổi tên thành `Query_1A_...` để phân biệt với Query_1B mới. Khi có cả hai file, tham chiếu đúng tên.

---

## II. DATA DICTIONARY — CỘT QUAN TRỌNG

### II.A — Query_1A: Lương trực tiếp

| Cột | Kiểu | Giá trị | Ghi chú |
|---|---|---|---|
| `Salon_ID` | int | | |
| `Salon_name` | str | "116 TK", "382 NT"... | |
| `GDKV` | str | "Đoàn Trung Đức", "Nguyễn Như Quỳnh"... | **Q1A đã có GDKV — không cần join file 60_salon** |
| `Vi_tri` | str | `Skinner` / `Stylist` / `Sup` | |
| `LevelId` | int | | |
| `Level_name` | str | xem bảng phân nhóm bên dưới | |
| `IsOverTime` | int | `0` = giờ thường / `1` = tăng ca | |
| `Nhom_DV_SP` | str | xem mapping bên dưới | |
| `TongLuong` | int | VNĐ | |

**Mapping Nhom_DV_SP trong Q1A:**

| Giá trị trong Q1A | Nhóm phân tích | Ghi chú |
|---|---|---|
| `1.DV_SC` | SC | |
| `2.Relax_Spa` | Relax | |
| `LRT` | LRT | |
| `3.DV_UND` | UND | |
| `Upsale_SUP` | Upsale_SUP | |
| `Thuong_Book_online` | **Loại khỏi phân tích mix** | Thưởng book online |
| `TIP_Ban_SP` | **Loại khỏi phân tích mix** | Tip bán sản phẩm |
| `TIP_Ban_TOPUP` | **Loại khỏi phân tích mix** | Tip topup |
| `Phat_KHL` | **Loại khỏi phân tích mix** | Phạt bill KHL (âm) |

**Phân nhóm Level:**

| Nhóm | Level_name thuộc về |
|---|---|
| Level 1 (thấp) | Level 1, SK Level 1A, SK Level 1B, SK Level 1C, ST Level 1A, ST Level 1B, ST Level 1C, Level CTV SK, Level CTV ST |
| Level 2 (trung) | Level 2, Level 2B |
| Level 3 (cao) | Level 3 |
| Level 4 (rất cao) | Level 4 |
| Khác (thử việc/CTV) | Level C, Level D, SK Level C01, SK Level D01, NaN |

---

### II.B — Query_2: DTT theo nhóm dịch vụ

| Cột | Giá trị `Nhom_DV_SP` | Ghi chú |
|---|---|---|
| `salon_ID` / `Salon_name` / `GDKV` | | |
| `Nhom_DV_SP` | `SC` / `Relax` / `LRT` / `UND` / `SP` | SP = sản phẩm, **không dùng để tính hệ số lương** |
| `DTT` | int | VNĐ |

> **Tên nhóm Q2 khác Q1A.** Mapping: `SC`↔`1.DV_SC`, `Relax`↔`2.Relax_Spa`, `LRT`↔`LRT`, `UND`↔`3.DV_UND`.

---

### II.C — Query_1B: Đơn giá thưởng ăn chia *(file mới từ v5)*

| Cột | Kiểu | Ghi chú |
|---|---|---|
| `SalonId` / `Salon_name` / `GDKV` | int/str | |
| `Vi_tri` | str | `Stylist` / `Skinner` |
| `LevelId` / `Level_name` | int/str | |
| `ItemId` | int | Mã dịch vụ cụ thể |
| `Item_name` | str | Tên dịch vụ (e.g., "ShineCombo 10 bước Ultra Care") |
| `Nhom_DV` | str | Nhóm DV: `1.DV_SC`, `2.Relax_Spa`, `LRT`, `3.DV_UND`... |
| `CustomerType` | str | `Khach BT` / `Thuong LK` / loại khác |
| `Don_Gia_Thuong` | int | VNĐ/lần dịch vụ = SSSalary × 3 |
| `TB_He_Thong` | float | Trung bình đơn giá tất cả salon cùng tổ hợp |
| `Min_He_Thong` / `Max_He_Thong` | int | Biên min/max hệ thống |
| `So_Salon_Tham_Khao` | int | Số salon được tổng hợp vào TB |
| `Delta_vs_TB` | int | `Don_Gia_Thuong − TB_He_Thong` (dương = cao hơn TB) |
| `Pct_vs_TB` | float | % chênh lệch so với TB |
| `Phan_loai` | str | `Cao hơn TB > 10%` / `Thấp hơn TB > 10%` / `Gần TB (±10%)` |

**Công thức đơn giá:** `SSSalary × 3 = đơn giá thực tế (VNĐ/lần dịch vụ)`

*Lý do ×3: bảng nguồn `test_ServiceSalarySatisfactionScoreV3` lưu 1/3 đơn giá thực (thiết kế hệ thống lương nội bộ).*

**Ví dụ thực tế (ItemId=513 "ShineCombo Ultra Care", ST Level 1, Khách BT):**
- Chuẩn (phần lớn salon): 8,600 × 3 = **25,800 VNĐ/lần**
- Salon 33: 10,918 × 3 = **32,754 VNĐ/lần** (+26,9% vs TB)
- Salon 54: 10,000 × 3 = **30,000 VNĐ/lần** (+16,3% vs TB)

---

### II.D — Tạm tính BV/SM/RSM/TTCL/BHXH (user cung cấp trong tin nhắn)

Dạng bảng kèm theo khi yêu cầu phân tích:

| Khoản mục | Tạm tính cả tháng (VNĐ) | Tạm tính kỳ báo cáo (VNĐ) |
|---|---|---|
| CP Lương Bảo vệ | | |
| CP Lương SM | | |
| CP Lương RSM | | |
| CP Thanh tra chất lượng | | |
| CP BHXH Salon+SM | | |
| CP BHXH RSM | | |

*(Tạm tính kỳ = Tạm tính tháng × số_ngày_kỳ / tổng_ngày_tháng)*

---

## III. NGUYÊN TẮC PHÂN TÍCH

**Thước đo chính:** So sánh trên **%/DTT**, không dùng số tuyệt đối. Mọi biến động quy về "+X điểm % so với ĐM tháng" và "+/- Y triệu đồng". **ĐM tham chiếu = định mức tháng hiện tại** (file target), không dùng ĐM đầu năm.

**Biến phí vs định phí:**
- Lương ST/SK/SUP = **biến phí thuần** → % không đổi theo DTT → **KHÔNG** giải thích "DTT thấp kéo lương trực tiếp lên %"
- BV / SM / RSM / BHXH = định phí → mới được nhận xét "DTT thấp → % cao"

**Đơn giá thưởng ăn chia = yếu tố cấu trúc (structural cost driver):**
- Đơn giá cao hơn chuẩn → salon đó có %CPNS cao hơn về cấu trúc, ngay cả khi vận hành tốt
- Phân biệt rõ: salon vượt ĐM do **vận hành** (TC cao, mix DV bất lợi) vs do **đơn giá bảng lương cao**
- Đơn giá không phải driver hàng tuần — chỉ phân tích khi điều tra outlier hoặc theo chu kỳ tháng/quý

---

## IV. NỘI DUNG PHÂN TÍCH CHI TIẾT

### MỤC 1 — TỔNG QUAN KỲ

**Bảng tổng quan — cấu trúc cột:**

| Cấu phần | TH kỳ này (%/DTT) | TH kỳ trước (%/DTT) | ĐM tháng (%/DTT) | Δ vs ĐM (điểm %) | Δ vs ĐM (tr.đ) | Dự báo cuối tháng (%/DTT) |
|---|---|---|---|---|---|---|

**Công thức cột Δ tr.đ — quan trọng, dễ sai:**
```python
delta_trieu = (th_pct - dm_pct) / 100 * dtt_ky
# th_pct và dm_pct là số dạng 18.44, 17.85 (không phải 0.1844)
# Đúng: (18.44 - 17.85) / 100 * 6_011_825_217 = +35.4 triệu
# Sai thường gặp: (18.44 - 17.85) / 100 / 100 * ... → nhỏ hơn 100x
```

**3 tầng nguồn data** (ghi trong note cuối bảng):
- ST / SK / SUP: data thực từ Query_1A
- BV / SM / RSM / TTCL / BHXH: tạm tính phân bổ theo ngày
- Thưởng CSLN: 0,5% × DTT kỳ. KD: KH tháng phân bổ tuyến tính. Phúc lợi: thực tế Larkform

**Nhận định:** (1) CPNS tổng bao nhiêu % so ĐM → vượt/đạt; (2) driver chính; (3) BV/SM/RSM: ngắn — cao/thấp so ĐM do DTT kỳ thấp/cao

> **Lưu ý phạm vi phân tích:** Báo cáo tập trung review 2 nhóm chính: **CP Lương trực tiếp** và **CP Thưởng & Phúc lợi**. Nhóm **BHXH** vẫn hiển thị trong bảng tổng quan nhưng **không cần bóc tách/nhận xét biến động %** — vì BHXH là chi phí cố định, biến động % hoàn toàn do DTT tăng/giảm, không phản ánh vấn đề quản lý.

---

### MỤC 2 — BÓC TÁCH BIẾN ĐỘNG LƯƠNG TRỰC TIẾP

Bóc 5 driver, mỗi driver ra điểm % và tr.đ, cộng lại = tổng biến động.

#### 2A. Dịch chuyển AOV Mix *(driver quan trọng nhất)*

**Hệ số lương mỗi nhóm DV** = lương nhóm đó (Q1A) / DTT nhóm đó (Q2), tính từ **kỳ trước làm baseline:**

```python
NHOM_MAP_Q1 = {'1.DV_SC':'SC','2.Relax_Spa':'Relax','LRT':'LRT','3.DV_UND':'UND','Upsale_SUP':'Upsale_SUP'}

q1_core = q1_truoc[q1_truoc['Nhom_DV_SP'].isin(NHOM_MAP_Q1)]
q1_core = q1_core.copy(); q1_core['Nhom'] = q1_core['Nhom_DV_SP'].map(NHOM_MAP_Q1)
luong_per_nhom = q1_core.groupby('Nhom')['TongLuong'].sum()

q2_core = q2_truoc[q2_truoc['Nhom_DV_SP'].isin(['SC','Relax','LRT','UND'])]
dtt_per_nhom = q2_core.groupby('Nhom_DV_SP')['DTT'].sum()

he_so = luong_per_nhom / dtt_per_nhom  # đơn vị: tỷ lệ, vd 0.4057 = 40.57%
```

*Hệ số tham khảo từ W1/T3-2026: SC≈40,6%, Relax≈34,5%, LRT≈39,5%, UND≈31,2%*

**Tác động mix của nhóm X:**
```python
ty_trong_X = dtt_X / dtt_total_core  # tỷ trọng trong tổng DTT core (bỏ SP)
tac_dong_X_pts = (ty_trong_X_ky_nay - ty_trong_X_ky_truoc) * he_so_X_truoc * 100
tac_dong_X_trieu = tac_dong_X_pts / 100 * dtt_ky_nay / 1e6
```

**Bảng AOV mix:**
| Nhóm DV | Tỷ trọng kỳ này (%) | Tỷ trọng kỳ trước (%) | Δ (pts) | Hệ số lương (kỳ trước) | Tác động (pts) | Tác động (tr.đ) |
|---|---|---|---|---|---|---|

#### 2B. Tỷ lệ Tăng Ca (TC)

**Phụ phí TC = 20% × Lương_TC** *(lương giờ TC = 1,25× BT → phụ phí = 0,25/1,25 = 20%, không phải 25%)*

```python
luong_BT = q1[q1['IsOverTime']==0].groupby('Vi_tri')['TongLuong'].sum()
luong_TC = q1[q1['IsOverTime']==1].groupby('Vi_tri')['TongLuong'].sum()
phu_phi_TC = luong_TC * 0.20
```

**Bảng TC** — kỳ này và kỳ trước cạnh nhau:
| Vị trí | Lương BT kỳ này (tr.đ) | Lương TC kỳ này (tr.đ) | %TC/Tổng kỳ này | Phụ phí TC kỳ này (tr.đ) | %TC/Tổng kỳ trước | Δ (pts) |
|---|---|---|---|---|---|---|

**Danh sách salon TC cao** (cho Section 5):
```python
# Lọc salon có tổng lương vị trí > 5 triệu (loại salon nhỏ)
# SK TC cao: lương TC Skinner / tổng lương Skinner salon > 40%
# ST TC cao: lương TC Stylist / tổng lương Stylist salon > 45%
```

#### 2C. Cơ cấu Level

```python
ty_trong_L34 = luong_L3L4 / luong_total_3vt * 100
# Tác động = (ty_trong_L34_ky_nay - ty_trong_L34_ky_truoc) × hệ_số_chênh_lệch
```

**Bảng level:**
| Nhóm Level | Tỷ trọng lương kỳ này (%) | Tỷ trọng lương kỳ trước (%) | Δ (pts) |
|---|---|---|---|
| Level 1 + biến thể | | | |
| Level 2 | | | |
| Level 3 | | | |
| Level 4 | | | |
| Khác (C/D/CTV) | | | |

#### 2D. Yếu tố khác
TOPUP, Book online, chi phí đột xuất 1 lần — nêu nếu phát sinh đáng kể

#### 2E. Đơn Giá Thưởng Ăn Chia *(driver cấu trúc — mới từ v5)*

> Phân tích này **không chạy mỗi kỳ** — chỉ thực hiện khi:
> 1. Điều tra salon vượt ĐM kéo dài không giải thích được bằng TC/mix
> 2. Đầu tháng/quý để scan toàn hệ thống
> 3. Trước hoặc sau khi vận hành cập nhật bảng lương

**Mục đích:** Tách riêng phần chi phí lương cao do **cấu trúc đơn giá bảng lương cao hơn chuẩn** ra khỏi các driver vận hành (TC, mix DV).

**Câu hỏi cần trả lời từ Query_1B:**
- Salon nào có đơn giá thưởng > TB hệ thống > 10% cho nhóm dịch vụ core (SC, Relax)?
- Nhóm DV nào có biến thiên đơn giá lớn nhất giữa các salon?
- Với salon X đang vượt ĐM, bao nhiêu điểm % đến từ đơn giá cao (cấu trúc)?

**Logic phân tích:**

```python
# Bước 1: Lọc salon có ít nhất 1 tổ hợp đơn giá "Cao hơn TB > 10%"
# tập trung nhóm SC và Relax (2 nhóm chiếm ~80% DTT core)
outlier_salons = q1b[
    (q1b['Nhom_DV'].isin(['1.DV_SC', '2.Relax_Spa'])) &
    (q1b['Phan_loai'] == 'Cao hơn TB > 10%') &
    (q1b['CustomerType'] == 'Khach BT')
]['SalonId'].unique()

# Bước 2: Với mỗi salon outlier, tính "extra cost ước tính"
# Giả định số lượt dịch vụ ~ TongLuong(Q1A) / Don_Gia_Thuong(Q1B)
# → delta_luong_ky = (Don_Gia_Thuong - TB_He_Thong) × so_luot_uoc_tinh
# Chia cho DTT → ra "điểm % đến từ đơn giá cao"
```

**Bảng tóm tắt đơn giá (chỉ hiển thị khi có dữ liệu Q1B):**

| Salon | GDKV | Nhóm DV | Vi_tri | Level | Đơn giá salon (VNĐ) | TB hệ thống (VNĐ) | +% vs TB | Phan_loai |
|---|---|---|---|---|---|---|---|---|

**Bảng tổng hợp bóc tách (cập nhật khi có 2E):**
| Nguyên nhân | Tác động (điểm %) | Tác động (tr.đ) | % tổng |
|---|---|---|---|
| AOV mix | | | |
| Tăng ca | | | |
| Cơ cấu Level | | | |
| **Đơn giá cao hơn chuẩn** | | | |
| Yếu tố khác | | | |
| **TỔNG** | | | **100%** |

---

### MỤC 3 — PHÂN RÃ THEO CHIỀU CẦN THIẾT

#### 3.1 Phân tích theo GDKV

| GDKV | DTT (tỷ) | SC% / Relax% / LRT% DTT | Δ ST vs ĐM T3 (pts) | Δ SK vs ĐM T3 (pts) | Δ SUP vs ĐM T3 (pts) | Lương TT vs ĐM (pts) | Đánh giá |
|---|---|---|---|---|---|---|---|

Nhận định từng vùng gắn mix DTT với % lương. Vùng Relax cao → SK cao là tự nhiên. Vùng vượt xa không giải thích được bằng mix → flag điều tra TC.

#### 3.2 Salon vượt ngưỡng lương TT >40%/DTT

Gom theo GDKV, mỗi salon ghi root cause — **từ v5 thêm 1 loại root cause mới:**

| # | Root cause | Dấu hiệu nhận biết |
|---|---|---|
| 1 | **TC cao** | SK_TC% > 40% hoặc ST_TC% > 45% |
| 2 | **Mix DV bất lợi** | Relax tỷ trọng cao > benchmark vùng |
| 3 | **Đơn giá bảng lương cao hơn chuẩn** | Q1B: Phan_loai = "Cao hơn TB > 10%" cho dịch vụ core |

Ghi rõ salon nào thuộc loại nào. Nếu salon vừa có TC cao vừa có đơn giá cao → ghi cả 2.

---

### MỤC 4 — KẾT LUẬN

Tổng kết ngắn gọn tình hình CPNS kỳ báo cáo:
- CPNS tổng bao nhiêu %/DTT, vượt/đạt ĐM bao nhiêu pts
- Nguyên nhân chính (gắn driver từ Mục 2)
- So sánh với kỳ trước: cải thiện hay xấu đi
- Đánh giá rủi ro nếu xu hướng tiếp tục

> **Lưu ý:** Dự báo cuối tháng đã nằm ở cột cuối bảng Mục 1. Các giả định dự báo ghi note ngay dưới bảng tổng quan (Mục 1), KHÔNG đặt ở Mục 4.

---

### MỤC 5 — HÀNH ĐỘNG KỲ TỚI

Tạo bảng hành động với tiêu đề cột, **để trống nội dung** cho người dùng điền sau:

| # | Hành động | Đối tượng (Salon / GDKV) | Chỉ số cần cải thiện | Deadline | Người phụ trách |
|---|---|---|---|---|---|
| | | | | | |

Gợi ý các hành động thường gặp (người dùng tự chọn và điền):
1. Rà soát TC Skinner cao — salon SK_TC > 40%
2. Rà soát TC Stylist cao — salon ST_TC > 45%
3. Tăng Relax + UND + SP để pha loãng SC
4. Phân bổ thưởng KD vào các tuần còn lại
5. **[Mới v5]** Đề xuất chuẩn hóa đơn giá thưởng về mức chuẩn hệ thống — salon có Phan_loai "Cao hơn TB > 10%" kéo dài
6. *(Không thêm "theo dõi lũy kế" chung chung)*

---

## V. FORMAT BÁO CÁO

- **Tool:** `python-docx` *(không dùng JS docx library)*
- **Output:** `/mnt/user-data/outputs/BaoCao_CPNS_[ngay_ky]_[thang]_[nam].docx`
- **Hướng trang:** Portrait A4, margin 2,5 cm
- **Cấu trúc 5 mục:** Tổng quan → Bóc tách → Phân rã → Kết luận → Hành động
- **Bảng:** Header teal (#2E75B6), dòng tổng highlight nhạt (#D6E4F0)
- **Số:** VNĐ = `1,234,567` ; % = `18,44%` ; điểm % = `+1,28 pts`
- **Ngôn ngữ:** Tiếng Việt

---

## VI. LƯU Ý KỸ THUẬT

- **Không join Q1A và Q2 cùng 1 query** → 1 bill nhiều vị trí → nhân đôi/ba DTT
- Q1A đã có cột `GDKV` → file 60_salon không cần thiết cho phân tích chính
- `Nhom_DV_SP` trong Q1A và Q2 dùng tên khác nhau — phải map trước khi ghép
- Bỏ `Thuong_Book_online`, `TIP_Ban_SP`, `TIP_Ban_TOPUP`, `Phat_KHL` khỏi phân tích cơ cấu DV
- Hệ số lương per nhóm DV: dùng **kỳ trước** làm baseline (không phải kỳ hiện tại)
- Công thức Δ tr.đ: `(th_pct - dm_pct) / 100 * DTT` — th_pct dạng 18.44, không phải 0.1844
- **[Mới v5]** Q1B không có filter ngày — là snapshot bảng lương tại thời điểm chạy query. Ghi chú ngày chạy vào tên file để truy vết.
- **[Mới v5]** `SSSalary × 3 = đơn giá thực` — không dùng SSSalary thô để so sánh
- **[Mới v5]** Khi Q1B không được cung cấp, bỏ qua Mục 2E và root cause "Đơn giá cao" trong Mục 3.2. Không được tự suy luận đơn giá từ Q1A.
