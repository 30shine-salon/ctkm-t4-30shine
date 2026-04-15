# PHƯƠNG PHÁP LUẬN: BÁO CÁO PHÂN TÍCH GIÁ SẢN PHẨM SALON
> Phiên bản: 2.0 | Kỳ áp dụng: Tháng 03/2026
> Tác giả: 30Shine Data Team
> Mục đích: Mô tả đầy đủ quy trình từ dữ liệu crawl đến báo cáo đề xuất giá cuối cùng

---

## 1. TỔNG QUAN QUY TRÌNH

```
[DSSP_enriched.xlsx] ──► product_enricher.py ──► Làm giàu thông tin SP
         │
         ▼
[DSSP_price_listings.xlsx] ◄── price_scanner_v3.py ──► Crawl giá từ 50+ nguồn
         │
         ▼
[DSSP_price_analysis_v3.xlsx] ──► Phân tích thống kê tự động
         │
         ▼
[BÁO CÁO CUỐI CÙNG] ──► 2 file Excel + 1 file Word
```

### Đầu vào (Input)
| File | Mô tả | Cột chính |
|------|--------|-----------|
| `DSSP_enriched.xlsx` | 144 SKU sản phẩm đã làm giàu thông tin | ItemId, Name, BRANDS, PRICE, Dung_Tich, Keyword_Search |
| `DSSP_price_analysis_v3_202603.xlsx` | Kết quả phân tích giá thị trường (43 cột) | Price_Original/Sale/Adjusted (Min/Max/Mean/Median), Mall/Normal breakdown, Gia_San/Tran, Danh_Gia, Hanh_Dong |
| `DSSP_price_listings.xlsx` | Chi tiết 6,500+ listings từ các sàn TMĐT | Price_Original, Price_Sale, Price_Adjusted, Platform, Shop_Type, Discount_%, Has_Voucher, Is_Flash_Sale |

### Đầu ra (Output) — 3 file
1. **`BAO_CAO_GIA_SALON_FINAL.xlsx`** — Báo cáo phân tích giá (5 sheets)
2. **`DE_XUAT_GIA_SAN_PHAM.xlsx`** — Đề xuất điều chỉnh giá (6 sheets)
3. **`Module_Quan_Tri_Gia_San_Pham_Salon.docx`** — Tài liệu Module quản trị giá (tham khảo, không cần tạo lại)

---

## 2. NGUỒN DỮ LIỆU VÀ CÁCH THU THẬP

### 2.1. Làm giàu thông tin sản phẩm (product_enricher.py)
- **Mô hình AI:** Gemini Pro (có Google Search)
- **Đầu vào:** Tên SP, Brand, Danh mục, Giá hiện tại
- **Đầu ra:** Tên chuẩn, dung tích, barcode, keyword tìm kiếm tối ưu
- **Cache:** Lưu kết quả để không gọi lại API cho SP đã xử lý

### 2.2. Scan giá thị trường (price_scanner_v3.py)
- **Nguồn thu thập:** Shopee Mall, Shopee, Lazada Mall, Lazada, TikTok Shop, Tiki, Sendo, Website chính hãng, Website độc lập
- **Mục tiêu:** Tối thiểu 30-50 listings/sản phẩm
- **Thông tin mỗi listing:**
  - `price_original`: Giá gốc/niêm yết (giá bị gạch ngang)
  - `price_sale`: Giá sau giảm (giá khách thực trả trên sàn)
  - `discount_percent`: % giảm giá
  - `has_voucher`, `voucher_value`: Thông tin voucher
  - `is_flash_sale`: Có phải giá sốc/flash sale
  - `platform`, `shop_type` (Mall/Official/Normal)
  - `sold_count`, `rating`

### 2.3. Hệ số điều chỉnh giá theo nền tảng (Platform Adjustment)
Giá hiển thị trên sàn chưa phản ánh giá thực tế khách trả (do voucher, mã giảm giá...). Áp dụng hệ số:

| Nền tảng | Hệ số | Lý do |
|----------|-------|-------|
| Shopee Mall | 0.92 | Thường có voucher 8% |
| Shopee | 0.88 | Hay giảm giá 12% |
| Lazada Mall / LazMall | 0.92 | Tương tự Shopee Mall |
| Lazada | 0.88 | Tương tự Shopee |
| TikTok Shop | 0.85 | Giảm mạnh để cạnh tranh |
| Tiki | 0.90 | Ít giảm hơn |
| Sendo | 0.85 | Giảm nhiều |
| Website chính hãng | 0.95 | Ít giảm |
| Website độc lập | 0.90 | Giảm trung bình |
| Mặc định | 0.90 | — |

**Quy tắc đặc biệt:**
- Nếu listing **đã có voucher** → hệ số = min(hệ số + 0.05, 1.0) (giảm bớt điều chỉnh)
- Nếu listing **là flash sale** → hệ số = 1.0 (không điều chỉnh thêm)

**Công thức:** `price_adjusted = price_sale × adjustment_factor`

---

## 3. PHƯƠNG PHÁP PHÂN TÍCH

### 3.1. Làm sạch dữ liệu listings
- Loại bỏ listing có giá sale < 20% hoặc > 300% so với giá tham khảo hiện tại
- Loại bỏ duplicate (cùng shop + cùng platform)
- Nếu không có price_sale → dùng price làm fallback
- Nếu không có price_original → lấy bằng price_sale

### 3.2. Tách Flash Sale và Thường ngày
- **Regular listings:** Không phải flash sale → dùng để tính giá thường ngày
- **Flash sale listings:** Đánh dấu riêng, không dùng để tính khung giá chuẩn
- Nếu tất cả đều flash sale → fallback dùng toàn bộ

### 3.3. Ba loại giá thống kê (cho mỗi SKU)

| Loại giá | Ý nghĩa | Cột |
|----------|---------|-----|
| **Price_Original** | Giá gốc/niêm yết trên sàn | Min, Max, Mean, Median |
| **Price_Sale** | Giá sau giảm hiển thị trên sàn | Min, Max, Mean, Median |
| **Price_Adjusted** | Giá thực tế ước tính (sau voucher + hệ số) | Min, Max, Mean, Median |

### 3.4. Phân tích Mall vs Normal

| Phân loại | Định nghĩa | Ý nghĩa |
|-----------|-----------|---------|
| **Mall/Official** | Shop_Type = "Mall" hoặc "Official" | Giá chính hãng, đáng tin cậy |
| **Normal** | Tất cả shop còn lại | Giá thị trường tự do |

Tính riêng `Adjusted_Min/Max/Mean` cho từng nhóm.

### 3.5. Công thức tính khung giá đề xuất

```
Gia_San (Giá sàn) = MIN(Mall_Adjusted_Mean, Price_Adjusted_Mean) × 0.95

Gia_Tran (Giá trần) = MAX(Mall_Adjusted_Mean, Price_Adjusted_Mean) × 1.15 × 1.10
                       (Hệ số GTGT = 1.10)

Gia_Canh_Tranh = Price_Adjusted_Median
                 (= median giá thực tế thị trường)

Gia_De_Xuat = (Mall_Adjusted_Mean + Price_Adjusted_Median) / 2
              (Nếu không có Mall → lấy Price_Adjusted_Median)
```

### 3.6. So sánh giá hiện tại với thị trường

```
Vs_Adjusted_Median_% = (Gia_Hien_Tai / Price_Adjusted_Median - 1) × 100
Vs_Sale_Median_%     = (Gia_Hien_Tai / Price_Sale_Median - 1) × 100
Vs_Original_Median_% = (Gia_Hien_Tai / Price_Original_Median - 1) × 100
```

### 3.7. Quy tắc đánh giá và khuyến nghị

| Điều kiện (Vs_Adjusted_Median_%) | Đánh giá | Hành động |
|----------------------------------|----------|-----------|
| > +15% | CAO hơn thị trường thực tế | Cân nhắc giảm giá |
| < -15% | THẤP hơn thị trường thực tế | Có thể tăng giá |
| Từ -15% đến +15% | PHÙ HỢP thị trường | Giữ nguyên |

---

## 4. CẤU TRÚC FILE ĐẦU RA

### 4.1. File 1: `BAO_CAO_GIA_SALON_FINAL.xlsx` (5 sheets)

#### Sheet 1: `1_Tong_Quan` — Dashboard tổng quan
Bảng key-value hiển thị:
- Tổng số SKU phân tích
- Tổng số listings thu thập
- Trung bình listings/SKU
- Số SKU giá CAO hơn thị trường (cần giảm)
- Số SKU giá THẤP hơn thị trường (có thể tăng)
- Số SKU PHÙ HỢP thị trường
- Nguồn dữ liệu (các sàn TMĐT)
- Ngày thu thập dữ liệu
- Phương pháp: Gemini AI + Google Search, hệ số điều chỉnh platform

| Cột | Mô tả |
|-----|-------|
| Chi_So | Tên chỉ số |
| Gia_Tri | Giá trị |
| Ghi_Chu | Giải thích thêm |

#### Sheet 2: `2_Can_Giam_Gia` — Sản phẩm cần giảm giá
- **Điều kiện lọc:** `Vs_Adjusted_Median_% > +15%` (hoặc `Danh_Gia == "CAO hơn thị trường thực tế"`)
- **Sắp xếp:** Theo `Vs_Adjusted_Median_%` giảm dần (vượt nhiều nhất lên đầu)

| Cột | Nguồn | Mô tả |
|-----|-------|-------|
| STT | Đánh số | Số thứ tự |
| Ten_SP | Name | Tên sản phẩm |
| Brand | BRANDS | Thương hiệu |
| Gia_HT | PRICE_Current | Giá hiện tại 30Shine |
| Gia_TT | Price_Adjusted_Median | Giá thị trường thực tế (median adjusted) |
| Gia_San | Gia_San | Giá sàn đề xuất |
| Gia_Tran | Gia_Tran | Giá trần đề xuất |
| Vuot_% | Vs_Adjusted_Median_% | % vượt so với thị trường |
| Khuyen_Nghi | — | Ghi chú đề xuất (tính toán cụ thể) |

**Cách tính Khuyen_Nghi:**
- Nếu vượt > 50%: "Giảm mạnh 25-40%, rà soát lại chiến lược giá"
- Nếu vượt 25-50%: "Giảm 15-25%"
- Nếu vượt 15-25%: "Giảm 5-15%, theo dõi phản hồi"

#### Sheet 3: `3_Co_The_Tang_Gia` — Sản phẩm có thể tăng giá
- **Điều kiện lọc:** `Vs_Adjusted_Median_% < -15%` (hoặc `Danh_Gia == "THẤP hơn thị trường thực tế"`)
- **Bao gồm cả SP có giá = 0** (chưa định giá)
- **Sắp xếp:** Theo `Vs_Adjusted_Median_%` tăng dần (thấp nhất lên đầu)

| Cột | Mô tả |
|-----|-------|
| (Tương tự sheet 2) | — |
| Khuyen_Nghi | Mức tăng đề xuất |

**Cách tính Khuyen_Nghi:**
- Nếu giá hiện tại = 0: "Cần định giá mới, đề xuất theo Gia_De_Xuat"
- Nếu thấp > 30%: "Có thể tăng 15-25%"
- Nếu thấp 15-30%: "Có thể tăng 5-15%"

#### Sheet 4: `4_Theo_Brand` — Phân tích theo thương hiệu
- **Gom nhóm:** Theo cột BRANDS
- **Tính cho mỗi brand:**

| Cột | Công thức |
|-----|-----------|
| Brand | Tên thương hiệu |
| So_SP | COUNT số SKU |
| Gia_30Shine_TB | MEAN(PRICE_Current) — giá trung bình 30Shine |
| Gia_TT_TB | MEAN(Price_Adjusted_Median) — giá thị trường TB |
| Chenh_% | (Gia_30Shine_TB / Gia_TT_TB - 1) × 100 |
| SP_OK | COUNT(Danh_Gia == "PHÙ HỢP") |
| SP_Vuot | COUNT(Danh_Gia == "CAO hơn...") |
| SP_Duoi | COUNT(Danh_Gia == "THẤP hơn...") |
| Ghi_Chu | Nhận xét tổng quan brand |

- **Sắp xếp:** Theo `Chenh_%` giảm dần

#### Sheet 5: `5_Data` — Dữ liệu thô đầy đủ
- Toàn bộ 144 dòng dữ liệu
- **9 cột chính:**

| Cột | Nguồn |
|-----|-------|
| ItemId | DSSP_price_analysis |
| Name | DSSP_price_analysis |
| BRANDS | DSSP_price_analysis |
| PRICE_Current | DSSP_price_analysis |
| Price_Adjusted_Median | DSSP_price_analysis |
| Gia_San_Module | Gia_San |
| Gia_Tran_Module | Gia_Tran |
| Gia_Khuyen_Nghi | Gia_De_Xuat |
| Hanh_Dong_Module | Dựa trên Danh_Gia → "GIỮ NGUYÊN" / "CÂN NHẮC GIẢM" / "CÓ THỂ TĂNG" |

---

### 4.2. File 2: `DE_XUAT_GIA_SAN_PHAM.xlsx` (6 sheets)

#### Sheet 1: `1_Phuong_Phap_Luan` — Phương pháp luận
Bảng text mô tả:
- **Mục tiêu:** Tối ưu biên lợi nhuận, giữ cạnh tranh, tránh mất khách vì giá cao
- **Phạm vi:** 144 SKU sản phẩm bán tại salon 30Shine
- **Nguồn dữ liệu:** Shopee, Lazada, TikTok Shop, Tiki, Sendo, Website chính hãng
- **Phương pháp:** AI scan + hệ số điều chỉnh platform
- **Nguyên tắc đề xuất:**
  - Mức tăng tối đa: +4% (trần an toàn)
  - Mức giảm: Tùy mức vượt, phân bậc
  - Ưu tiên SP có doanh số cao (impact lớn)

#### Sheet 2: `2_Ket_Qua_Tong_Hop` — Kết quả tổng hợp
Bảng phân loại:

| Phân loại | Cách tính | Mô tả |
|-----------|-----------|-------|
| Đề xuất TĂNG giá | Vs_Adjusted_Median_% < -15% VÀ PRICE > 0 | Giá thấp hơn thị trường |
| GIỮ NGUYÊN | -15% ≤ Vs_Adjusted_Median_% ≤ +15% | Phù hợp |
| Đề xuất GIẢM giá | Vs_Adjusted_Median_% > +15% | Giá cao hơn thị trường |
| Cần ĐỊNH GIÁ MỚI | PRICE_Current = 0 | Chưa có giá |

Hiển thị: Số SKU, Tỷ lệ % cho mỗi nhóm.

#### Sheet 3: `3_De_Xuat_Gia` — Đề xuất giá chi tiết (144 dòng)

| Cột | Mô tả | Cách tính |
|-----|-------|-----------|
| Ma_SP | ItemId | — |
| Ten_SP | Name | — |
| Brand | BRANDS | — |
| Gia_HT | PRICE_Current | Giá hiện tại |
| Gia_TT | Price_Adjusted_Median | Giá thị trường |
| Gap_% | Vs_Adjusted_Median_% | % chênh lệch |
| Score | Điểm ưu tiên | Xem mục 5 bên dưới |
| Tang_Giam | Mức tăng/giảm (VNĐ) | Gia_Moi - Gia_HT |
| Tran_Tang | Trần tăng cho phép | Gia_HT × 1.04 (trần +4%) |
| Gia_Moi | Giá đề xuất mới | Xem mục 5 bên dưới |
| Nhom | Nhóm điều chỉnh | "+2% trần 4%", "Giảm 15%", "Giữ nguyên"... |
| Ghi_Chu | Ghi chú | — |

#### Sheet 4: `4_Top_SKU_Tang_LNG` — Top SKU tăng lợi nhuận gộp
- **Chỉ áp dụng cho SP được đề xuất TĂNG giá**
- **Cần dữ liệu bổ sung:** Số gói bán tháng gần nhất (nếu có)

| Cột | Mô tả |
|-----|-------|
| Ten_SP | Tên sản phẩm |
| So_Goi_T1 | Số gói bán (tháng gần nhất) |
| Muc_Tang | Mức tăng giá (VNĐ) = Gia_Moi - Gia_HT |
| Tran_Tang | Trần tăng cho phép |
| LNG_Tang_Them | Lợi nhuận gộp tăng thêm = Muc_Tang × So_Goi_T1 |
| Phan_Tram_Dong_Gop | % đóng góp = LNG_Tang_Them / Tổng LNG tăng thêm |

> **Lưu ý:** Nếu không có dữ liệu số gói bán → bỏ qua sheet này hoặc để trống cột So_Goi_T1 và LNG_Tang_Them.

#### Sheet 5: `5_SP_Giam_Gia` — Sản phẩm đề xuất giảm giá
- **Điều kiện:** Vs_Adjusted_Median_% > +25% (vượt đáng kể)
- **Sắp xếp:** Theo Gap_% giảm dần

| Cột | Mô tả |
|-----|-------|
| Ma_SP | ItemId |
| Ten_SP | Name |
| Gia_HT | PRICE_Current |
| Gia_TT | Price_Adjusted_Median |
| Gap_% | Vs_Adjusted_Median_% |
| De_Xuat | Mức giảm đề xuất |

**Quy tắc đề xuất mức giảm:**
- Gap > 60%: "Giảm 25-40%, kiểm tra lại nguồn hàng"
- Gap 40-60%: "Giảm 20-30%"
- Gap 25-40%: "Giảm 15-25%"

#### Sheet 6: `6_Ke_Hoach_Trien_Khai` — Kế hoạch triển khai
Bảng text mô tả phân kỳ:
- **Giai đoạn 1 (Tuần 1-2):** Giảm giá các SP vượt > 40% (ưu tiên cao)
- **Giai đoạn 2 (Tuần 3-4):** Tăng giá các SP dưới thị trường (tăng từ từ, tối đa +4%/lần)
- **Giai đoạn 3 (Tháng 2):** Điều chỉnh các SP còn lại, đo lường phản hồi
- **Theo dõi:** Doanh số, phản hồi khách hàng, so sánh lại giá thị trường sau 1 tháng

---

## 5. CÔNG THỨC TÍNH GIÁ ĐỀ XUẤT MỚI (CHI TIẾT)

### 5.1. Phân nhóm sản phẩm

| Nhóm | Điều kiện | Hành động |
|------|-----------|-----------|
| **A: Cần giảm mạnh** | Gap > +40% | Giảm về gần Gia_Tran |
| **B: Cần giảm vừa** | Gap +25% → +40% | Giảm 15-25% |
| **C: Cần giảm nhẹ** | Gap +15% → +25% | Giảm 5-15% |
| **D: Phù hợp** | Gap -15% → +15% | Giữ nguyên |
| **E: Có thể tăng nhẹ** | Gap -30% → -15% | Tăng +2% (trần +4%) |
| **F: Có thể tăng** | Gap < -30% | Tăng +4% (trần +4%) |
| **G: Chưa có giá** | PRICE = 0 | Đặt = Gia_De_Xuat |

### 5.2. Công thức Gia_Moi

```python
if PRICE_Current == 0:
    Gia_Moi = Gia_De_Xuat  # Từ analysis

elif Gap > 40%:
    Gia_Moi = min(PRICE_Current × 0.75, Gia_Tran)  # Giảm mạnh

elif Gap > 25%:
    Gia_Moi = min(PRICE_Current × 0.80, Gia_Tran)  # Giảm 20%

elif Gap > 15%:
    Gia_Moi = min(PRICE_Current × 0.90, Gia_Tran)  # Giảm 10%

elif Gap < -30%:
    Gia_Moi = min(PRICE_Current × 1.04, Gia_Tran)  # Tăng 4% (trần)

elif Gap < -15%:
    Gia_Moi = min(PRICE_Current × 1.02, Gia_Tran)  # Tăng 2%

else:
    Gia_Moi = PRICE_Current  # Giữ nguyên
```

### 5.3. Điểm ưu tiên (Score)
Dùng để sắp xếp thứ tự ưu tiên điều chỉnh:

```python
Score = abs(Gap_%) × weight

# weight:
#   1.5 nếu Gap > 0 (đang cao hơn TT → ưu tiên giảm, tránh mất khách)
#   1.0 nếu Gap < 0 (đang thấp hơn TT → tăng từ từ)
```

### 5.4. Làm tròn tâm lý (Psychological Rounding)
Áp dụng cho Gia_Moi cuối cùng:
- Giá < 100,000đ → làm tròn xuống bội số 9,000 gần nhất (VD: 87,500 → 79,000 hoặc 89,000)
- Giá ≥ 100,000đ → làm tròn xuống bội số 90,000 hoặc x9,000 gần nhất (VD: 385,000 → 379,000)
- Nguyên tắc: Đuôi giá luôn kết thúc bằng 9,000 hoặc 90,000

---

## 6. QUY TẮC XỬ LÝ ĐẶC BIỆT

### 6.1. Sản phẩm có giá = 0
- Chưa được định giá → phân vào nhóm "Cần ĐỊNH GIÁ MỚI"
- Gia_Moi = Gia_De_Xuat (từ kết quả scan)
- Ghi chú: "Sản phẩm mới, đề xuất giá theo thị trường"

### 6.2. Sản phẩm có ít listings (< 10)
- Đánh dấu "Dữ liệu không đủ tin cậy"
- Vẫn tính toán nhưng ghi chú cảnh báo
- Khuyến nghị: Giữ nguyên, cần thu thập thêm dữ liệu

### 6.3. Sản phẩm Flash Sale chiếm đa số
- Nếu > 50% listings là flash sale → ghi chú "Thị trường flash sale nhiều"
- Insight voucher: Nếu > 50% listings có voucher → cảnh báo giá thực tế có thể thấp hơn

---

## 7. TÓM TẮT CÁC BƯỚC THỰC HIỆN

Khi có dữ liệu đầu vào mới (3 file từ thư mục kỳ mới), thực hiện:

| Bước | Mô tả | Input | Output |
|------|-------|-------|--------|
| 1 | Đọc file `DSSP_price_analysis_v3_*.xlsx` | File analysis | DataFrame 144 dòng × 43 cột |
| 2 | Đọc file `DSSP_price_listings.xlsx` | File listings | DataFrame 6,500+ dòng |
| 3 | Đọc file `DSSP_enriched.xlsx` | File enriched | DataFrame 144 dòng (metadata bổ sung) |
| 4 | Tính các chỉ số tổng quan | Từ bước 1-3 | Số liệu cho sheet Tong_Quan |
| 5 | Phân loại SP theo Gap_% | Từ bước 1 | 4 nhóm: Tăng/Giữ/Giảm/Mới |
| 6 | Tính Gia_Moi cho từng SP | Công thức mục 5.2 | 144 dòng có giá mới |
| 7 | Làm tròn tâm lý | Mục 5.4 | Gia_Moi đã làm tròn |
| 8 | Phân tích theo Brand | Gom nhóm | Bảng brand summary |
| 9 | Lọc SP giảm/tăng/giữ | Điều kiện mục 3.7 | Các sheet riêng |
| 10 | Tính Top SKU tăng LNG | Nếu có data bán hàng | Sheet 4 của file 2 |
| 11 | Xuất 2 file Excel | Tất cả kết quả | 2 file output |

---

## 8. LƯU Ý QUAN TRỌNG

1. **Giá tham chiếu chính:** Luôn dùng `Price_Adjusted_Median` (giá thực tế ước tính) làm benchmark, KHÔNG dùng Price_Sale hay Price_Original
2. **Trần tăng giá an toàn:** Tối đa +4% mỗi lần điều chỉnh để tránh phản ứng tiêu cực từ khách hàng
3. **Giảm giá không giới hạn:** Giảm theo mức cần thiết để cạnh tranh
4. **Cập nhật định kỳ:** Nên scan lại giá thị trường mỗi tháng/quý
5. **Dữ liệu bán hàng:** Sheet `4_Top_SKU_Tang_LNG` cần dữ liệu số gói bán từ hệ thống nội bộ — nếu không có thì bỏ qua hoặc để placeholder
