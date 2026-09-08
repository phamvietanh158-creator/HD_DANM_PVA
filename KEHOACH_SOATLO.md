# KẾ HOẠCH RÀ SOÁT & SỬA — HD_DANM_PVA
> Cập nhật: 2026-09-08 | TS. Phạm Việt Anh

---

## 1. VẤN ĐỀ CỐT LÕI

### 1.1 Dữ liệu không liên thông xuyên suốt giữa các module
Sinh viên phải nhập lại thông số đã có ở module trước. Nguyên nhân:
- Một số trường chưa được auto-fill từ `localStorage`
- `pa_chon` (phương án 1/2 đã chốt ở module01) **không được hiển thị** ở module02+, SV không biết đang làm phương án nào
- Module04 đọc key **`nmong`** sai — đúng phải là **`nmong_p1`**
- Kích thước đáy móng chốt (b×l) từ module02 chưa được truyền sang module03 đủ context

### 1.2 Một số nội dung tính toán ẩn thay vì hiện
- TTGH1 (SCT Terzaghi) trong module02: nội dung không chạy ra → `buildTTGH1()` thất bại
- Các section bị `display:none` cần được cho hiện để SV điền

### 1.3 Khi chốt phương án → module tự nhận thông số
- PA1 (móng nông trực tiếp) hoặc PA2 (đệm cát): module02/03 cần tự hiển thị section tương ứng
- Chỉ cần SV chọn b×l hoặc hdc (chiều dày đệm cát), không nhập lại toàn bộ

---

## 2. LUỒNG DỮ LIỆU HIỆN TẠI (localStorage key: `nmong_p1`)

```
module01_phuong_an.html
  └─ goModule02() → lưu nmong_p1:
       soils[], hm, hm_sb, lop_dat, hdc, pa_chon
       N0tt/M0tt/Q0tt, N0tc/M0tc/Q0tc, bc, lc, mnn, de_so

module02_kiem_tra_nen.html
  ├─ window.onload → đọc nmong_p1: N0tc/M0tc/Q0tc, hm_sb, mnn, soils[]
  │    auto-fill: phi1, c1, gam1, E01, gam0, h1_val, hdc
  │    ✗ THIẾU: pa_chon (hiển thị), b1/l1 gợi ý từ pa_chon
  ├─ buildTTGH1() → ✗ KHÔNG CHẠY (lỗi im lặng)
  └─ goModule03() / saveProgress() → lưu nmong_p1:
       m02_b, m02_l, m02_S, m02_saved

module03_chieu_cao_cot_thep.html
  ├─ window.onload → đọc nmong_p1: N0tt/M0tt, hm_sb, bc/lc, m02_b/m02_l
  └─ saveM03() → lưu nmong_p1: m03_h, m03_mac_bt, m03_mac_thep, m03_thep_x, m03_thep_b

module04_ban_ve.html
  └─ window.onload → đọc 'nmong'  ← ✗ SAI KEY! Phải là 'nmong_p1'

phan3/p3_module01_phuong_an.html
  ├─ window.onload → đọc cả nmong_p1 (tải trọng, địa chất) + nmong_p3
  └─ saveM05() → lưu nmong_p3

phan2/ (bảng tổng hợp)
  └─ đọc nmong_p1 để điền bảng so sánh PA → cần kiểm tra riêng
```

---

## 3. DANH SÁCH LỖI THEO ƯU TIÊN

### 🔴 CRITICAL — Gây hỏng luồng chính

| # | Module | Vấn đề | Chi tiết |
|---|--------|---------|----------|
| C1 | module02 | `buildTTGH1()` không chạy | Không có b-final/l-final, hoặc `calcSCT` trả null → toàn bộ TTGH1 trống |
| C2 | module04 | Key localStorage sai: `'nmong'` → phải `'nmong_p1'` | module04 luôn load rỗng |
| C3 | module02 | `pa_chon` không hiển thị | SV không biết đang làm PA1 hay PA2 |

### 🟠 HIGH — Phải nhập lại thủ công thứ đã có

| # | Module | Vấn đề | Chi tiết |
|---|--------|---------|----------|
| H1 | module02 | Chưa hiển thị `pa_chon`, `hdc` (đệm cát) ngay khi load | SV phải tự nhớ từ module01 |
| H2 | module02 | Các section TTGH2 lún bị ẩn → cần hiện để SV tính và điền | Không cần ẩn |
| H3 | module03 | `b`, `l` từ m02 load được nhưng không có nhãn/context rõ ràng | SV không biết đây là kết quả đã kiểm tra |
| H4 | module03 | `pa_chon` không được đọc → không auto-skip section đệm cát | Hiện cả 2 section dù SV chỉ làm 1 PA |

### 🟡 MEDIUM — UX / chính xác kết quả

| # | Module | Vấn đề | Chi tiết |
|---|--------|---------|----------|
| M1 | module02 | `calcSCT`: khi `phi=0 AND c=0` trả null im lặng | Cần thông báo rõ lý do không tính được |
| M2 | module02 | `b2`, `l2`, `b3`, `l3` là hidden input → SV không điền được nếu muốn thử nhiều cặp | Cần input visible hoặc cơ chế add row |
| M3 | module01 | `isDinh` = `WL!==''&&WP!==''` → nếu chỉ có 1 trong 2 → sai | Cần `WL!==''&&WP!==''` hoặc rõ hơn |
| M4 | module04 | Một số field đọc từ `nmong` (sai) nên luôn rỗng | Sau fix C2, cần kiểm tra từng field |
| M5 | module02 | Không validate b>0, l>0 trước khi tính → Infinity/NaN | Cần guard |
| M6 | tất cả | Decimal dấu phẩy `,` → `parseFloat` bị cắt → NaN | Normalize input: replace(',','.') |

### 🟢 LOW — Giao diện / nhỏ

| # | Module | Vấn đề |
|---|--------|---------|
| L1 | module01 | Placeholder một số field gây hiểu nhầm |
| L2 | module02 | Contrast `.btn-ghost` thấp trên nền sáng |
| L3 | tất cả | Thiếu `type="button"` trên một số `<button>` |
| L4 | module01 | `min-width:0` thiếu đơn vị |

---

## 4. KẾ HOẠCH SỬA THEO MODULE (thứ tự thực hiện)

### Phase A — Fix luồng data liên thông (1–2 ngày)

#### A1. Module02: Hiển thị thông tin PA đã chốt từ module01
- Đọc `pa_chon` từ nmong_p1 → hiện banner "Phương án đã chọn: PA1 – Móng nông / PA2 – Đệm cát"
- Nếu PA2: tự fill `hdc`, show section đệm cát; nếu PA1: ẩn section đệm cát
- Hiện readonly badge với: `hm`, `phi1`, `c1`, `gam1`, `E01`, `pa_chon` (SV không cần gõ lại)

#### A2. Module02: Fix buildTTGH1()
- Debug lý do không chạy: kiểm tra `b-final`, `l-final` có trong HTML không
- Nếu thiếu: thêm field hoặc đổi cách lấy giá trị b/l từ input b1/l1
- Đảm bảo section TTGH1 và TTGH2 LUÔN VISIBLE (không hide bằng JS)
- Fix `calcSCT` khi phi=0 AND c=0: thêm cảnh báo thay vì return null im lặng

#### A3. Module04: Fix localStorage key
- Đổi tất cả `localStorage.getItem('nmong')` → `'nmong_p1'`
- Kiểm tra từng field sau khi fix

#### A4. Module03: Truyền context rõ ràng
- Sau khi load m02_b/m02_l: hiện label "b×l đã kiểm tra TTGH1 = X×Y m"
- Đọc `pa_chon`: nếu PA1 ẩn section đệm cát; nếu PA2 show

### Phase B — Đồng bộ thông số xuyên suốt (ngày 2–3)

#### B1. Module01 → Module02: bổ sung các trường còn thiếu
Cần lưu thêm vào nmong_p1 khi goModule02():
- `phi1`, `c1`, `gam1`, `E01` (lớp chịu tải) — hiện đã auto-fill nhưng cần xác nhận
- `gam0`, `h1_val` (lớp trên) — cần xác nhận đủ
- `pa1h`, `pa2h`, `pa1t`, `pa2t` (thông số PA1/PA2 đề xuất từ module01)

#### B2. Module02 → Module03: bổ sung các trường
Cần lưu thêm khi goModule03() / saveProgress():
- `m02_b_chon`, `m02_l_chon` (kích thước chốt cuối cùng)
- `m02_SCT_ok` (đã pass TTGH1?), `m02_S_ok` (đã pass TTGH2?)
- `pgh` (sức chịu tải nền Terzaghi)

#### B3. Module03 → Module04: kiểm tra chain
Sau fix key, xác nhận module04 nhận đủ:
- `m03_h` (chiều cao móng), `m03_thep_x`, `m03_thep_b`
- `m02_b`, `m02_l` (kích thước)
- `hm`, `N0tt`, `M0tt`, `bc`, `lc`

#### B4. Phan3: kiểm tra chain từ p1
- p3_module01 đọc nmong_p1 cho tải trọng + địa chất → xác nhận đủ
- Đảm bảo sinh viên không phải nhập lại tải trọng khi chuyển sang Phần 3

### Phase C — Input validation & UX (ngày 3–4)

#### C1. Normalize decimal dấu phẩy
Thêm event listener chung:
```js
document.querySelectorAll('input[type=number], input[type=text]').forEach(function(el){
  el.addEventListener('input',function(){ this.value=this.value.replace(',','.'); });
});
```

#### C2. Guard chia cho 0
- Trong `calcSCT`: kiểm tra b>0 && l>0
- Trong TTGH2 lún: kiểm tra b>0

#### C3. Bổ sung `type="button"` cho tất cả button không phải submit

#### C4. Cải thiện UX placeholder: thêm ví dụ thực tế thay vì con số trần

---

## 5. TIÊU CHÍ HOÀN THÀNH

| Tiêu chí | Cách kiểm tra |
|----------|---------------|
| Module02 hiển thị đúng PA đã chốt | Load module02 sau module01 → thấy banner PA1/PA2 |
| TTGH1 (SCT) chạy ra kết quả | Nhập b=1.5, l=1.5 → bấm Kiểm tra → có bảng kết quả |
| Module04 nhận đủ data | Mở module04 → thấy hm, b, l, thep đúng |
| Không phải nhập lại tải trọng | Toàn bộ N0/M0/Q0, hm chỉ nhập 1 lần ở module01 |
| Decimal phẩy hoạt động | Nhập "18,5" cho γ → hiển thị 18.5, tính đúng |
| Console F12 không có ReferenceError | Mở từng module → F12 → Console: 0 lỗi đỏ |

---

## 6. CÁC FILE CẦN SỬA (ưu tiên cao → thấp)

1. `phan1/module02_kiem_tra_nen.html` — nhiều nhất, TTGH1 + A1+A2
2. `phan1/module04_ban_ve.html` — fix key localStorage (C2)
3. `phan1/module01_phuong_an.html` — bổ sung dữ liệu save
4. `phan1/module03_chieu_cao_cot_thep.html` — truyền context PA
5. `phan3/p3_module01_phuong_an.html` — kiểm tra chain
6. `phan2/*.html` — kiểm tra bảng tổng hợp

---

## 7. GHI CHÚ KỸ THUẬT

- **Key dùng chung:** `nmong_p1` (phan1+phan2), `nmong_p3` (phan3)
- **Không dùng KaTeX** — quyết định kiến trúc vĩnh viễn
- **Không dùng framework** — vanilla HTML/JS
- **Không cần internet sau khi tải** — DB địa chất nhúng inline
- **Unit:** tải trọng kN, kích thước m, áp lực kPa, γ kN/m³
