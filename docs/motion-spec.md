# Motion spec – Quản lý tài chính (Phase 1)

> Dành cho dev đưa thẳng vào AI coding assistant (Claude, Cursor, Copilot…) làm ngữ cảnh.
> Nguồn sự thật: prototype https://demo-personal-finance-pink.vercel.app (file `public/index.html`) và Figma "Tài chính thông minh", page Phase 1 (node 807:22203).
> Cập nhật: 27/11/2026 – đã đồng bộ với motion Home VTM 9.9 (Figma "VTM 9.9 (QLUD Only)", page "Home - Phase 1 (19.10.2026)", mục "4. Interaction / Micro-Animation / Haptic / Loading").
> Phần dùng chung (lớp 1–2) lấy số của Home; phần riêng Quản lý tài chính (donut, thu gọn header tổng số dư, Quản lý thu chi) giữ nguyên.

---

## 0. Cách dùng với AI

Dán file này vào ngữ cảnh rồi giao việc theo mẫu:

```text
Đọc docs/motion-spec.md. Hiện thực mục <ID, ví dụ M03> cho <màn/component> trên <nền tảng: iOS SwiftUI | Android Compose | Flutter | React Native>.
Yêu cầu:
- Dùng đúng token ở mục 2, không tự đặt thời lượng/easing khác.
- Tôn trọng "Quy tắc" và "Ngắt giữa chừng" của mục đó.
- Hỗ trợ Reduce Motion theo mục 1.4.
- Kết thúc bằng checklist "Tiêu chí nghiệm thu" của mục đó, đánh dấu cái nào đã đạt.
```

Mỗi mục M01–M22 tự đủ ý: trigger → thuộc tính đổi (từ → đến) → thời lượng/độ trễ/easing → quy tắc → tiêu chí nghiệm thu.

---

## 1. Quy ước

### 1.1 Đơn vị và hệ toạ độ
- Kích thước khung thiết kế: 375 × 812 pt. Mọi số px trong file là pt/dp ở khung này.
- Toạ độ y tính từ mép trên màn hình (đã gồm status bar 44 pt).
- Thời gian tính bằng ms. `delay` tính từ lúc trigger xảy ra (t = 0).
- `stagger i × N ms`: phần tử thứ i (i bắt đầu từ 0) trễ thêm i × N ms.

### 1.2 Easing
Dùng token ở mục 2. Cubic-bezier viết theo dạng `(x1, y1, x2, y2)`.
Ánh xạ gợi ý:
- iOS: `UICubicTimingParameters(controlPoint1:controlPoint2:)` hoặc SwiftUI `.timingCurve(x1, y1, x2, y2, duration:)`.
- Android Compose: `CubicBezierEasing(x1, y1, x2, y2)`.
- Flutter: `Cubic(x1, y1, x2, y2)`.
- React Native Reanimated: `Easing.bezier(x1, y1, x2, y2)`.

### 1.3 Hai loại animation
- **Theo thời gian** (time-based): chạy hết thời lượng khi có trigger.
- **Theo cuộn** (scroll-driven): giá trị là hàm của vị trí cuộn, KHÔNG có thời lượng. Ký hiệu `p` (0…1). Không dùng animation thời gian để giả lập.

### 1.4 Spring (theo Home)
Home dùng spring kiểu Apple với 2 tham số: `response` (giây, càng nhỏ càng nhanh) và `damping` ζ (1 = không nảy, < 1 = nảy).
- Công thức: `stiffness = (2π / response)²`, `damping_coeff = 4π·ζ / response`, mass = 1.
- iOS: `.spring(response:, dampingFraction:)`. Android Compose: `spring(dampingRatio = ζ, stiffness = (2π/response)²)`. Flutter: `SpringDescription(mass: 1, stiffness: …, damping: …)`. Web: easing `linear()` sinh từ spring (xem `springKF()` trong prototype).
- Spring luôn chạy tiếp từ giá trị + vận tốc hiện tại → ngắt được giữa chừng, không giật.
- Sau cử chỉ (vuốt, kéo): đích chọn theo điểm dừng dự kiến `project(v) = (v/1000)·d/(1−d)` (v px/s), rồi giao vận tốc tay cho spring (`v0 = v / quãng còn lại`).

### 1.5 Reduce Motion
Khi hệ điều hành bật Reduce Motion:
- Tắt M01 (intro), M02 (đếm số), M07 (reveal): hiện ngay trạng thái cuối.
- Chuyển màn (M13) đổi sang fade 200 ms; đổi tab (M08) fade 100/150 ms, không blur.
- Nhấn (M22): không scale, đổi opacity 0.55 (thanh điều hướng 0.6).
- Giữ animation theo cuộn (M03) vì nó gắn với thao tác tay.

---

## 2. Tokens

```yaml
spring:            # [response giây, damping ζ] – lấy từ Home VTM 9.9
  press:   [0.18, 1.0]    # lún khi chạm
  bounce:  [0.38, 0.5]    # nảy về khi nhả
  bar-down:[0.25, 1.0]    # cả thanh điều hướng dưới lún
  bar-up:  [0.40, 0.6]
  tab-down:[0.20, 1.0]    # từng nút trong thanh điều hướng
  tab-up:  [0.35, 0.55]
  page:    [0.38, 1.0]    # bắt trang carousel, snap header
  indicator:[0.30, 1.0]   # viên chỉ báo tab
  dots:    [0.12, 1.0]    # chấm phân trang
  push:    [0.50, 1.0]    # màn con đẩy vào / ra
projection:
  page: 0.99         # carousel (Home)
  edge-swipe: 0.998  # vuốt cạnh trái để back (độ giảm tốc cuộn chuẩn iOS)
nav-delay: 100       # ms – nhả tay xong 100 ms mới mở màn/sheet (Home)
push-under-shift: -112   # pt – màn dưới lùi khi màn con push vào (Home)
push-dim: 0.10           # màn dưới tối 10% (Home)
axis-lock: 10            # pt – kéo quá 10 pt mới khoá trục ngang/dọc

easing:
  ease-out-standard: [0.2, 0.8, 0.2, 1]      # vào màn, trượt nội dung, toast
  ease-ios-sheet:    [0.32, 0.72, 0, 1]      # push, modal, bottom sheet, FAQ
  spring-pop:        [0.34, 1.56, 0.64, 1]   # icon bật ra, vượt đích nhẹ
  spring-soft:       [0.2, 0.9, 0.25, 1.06]  # popup onboarding
  spring-tabbar:     [0.2, 0.9, 0.25, 1.15]  # thanh điều hướng dưới vào màn
  spring-pill:       [0.3, 1.35, 0.5, 1]     # viên thuốc tab Tài sản/Phải trả
  ease-carousel:     [0.3, 0.7, 0.2, 1]      # banner, chấm phân trang, donut thu chi
  sweep:             [0.65, 0, 0.35, 1]      # cung donut quét vòng
  ease-in:           [0.42, 0, 1, 1]         # nội dung đi ra khi đổi tab
  ease-out-cubic:    "1 - (1 - t)^3"
  ease-out-quart:    "1 - (1 - t)^4"          # đếm số
  ease-out-back:     "1 + 2.25(t-1)^3 + 1.25(t-1)^2"   # snap header tab Tài sản (c1 = 1.25)
  spring-folder:     "1 - e^(-7t) * cos(9t)" # tab bìa hồ sơ Quản lý thu chi

duration:
  press: 150
  toast-in: 250        # opacity; translate 350
  toast-hold: 1800
  tab-out: 180
  tab-in: 340
  tab-pill: 500
  sheet-open: 520
  sheet-close: 340
  snap-back: 320

scale:
  press-card: 0.97     # thẻ, dòng, thanh điều hướng
  press-cta: 0.94      # nút CTA
  press-icon: 0.88     # nút icon, nút trong thanh điều hướng

layout:
  screen: [375, 812]
  content-top: 314           # y bắt đầu nội dung (cả 2 tab) khi chưa cuộn
  content-top-collapsed: 184 # y nội dung khi header đã thu gọn
  collapse-distance: 130     # = 314 - 184, dùng chung 2 tab
  header-bg: [375, 162]      # nền header khi thu gọn, bo dưới 24
  content-min-height: 942    # = 812 + 130, để tab ngắn vẫn giữ được header thu gọn
```

---

## 3. Đặc tả từng chuyển động

### M01 – Intro khi vào màn Quản lý tài chính
- **Trigger:** màn xuất hiện lần đầu, hoặc đổi tầng người dùng (M09).
- **Tổng thời gian:** ~1,7 s.

| Phần tử | Từ → Đến | Delay | Duration | Easing |
|---|---|---|---|---|
| Tiêu đề "Quản lý tài chính" | Không animation, hiện ngay | – | – | – |
| Tab Tài sản / Phải trả | opacity 0→1, y −10→0 | 100 | 600 | ease-out-standard |
| Nhãn "Tổng tài sản" + icon mắt | opacity 0→1, y −10→0 | 200 | 600 | ease-out-standard |
| Số tiền tổng | opacity 0→1, y −10→0; đồng thời đếm số (M02) | 250 | 600 | ease-out-standard |
| Dòng "Đồng bộ lúc…", tiêu đề nhóm, từng dòng tài sản/nguồn tiền, các khối bên dưới | opacity 0→1, y +8→0 (Home) | 80 + min(i, 5) × 40 (Home) | 600 | ease-out-standard |
| Vòng nền donut | opacity 0→1, scale 0.82→1, rotate −25°→0° | 50 | 800 | ease-out-standard |
| Cung màu donut | quét 0°→360° theo chiều kim đồng hồ (mask) | 250 | 1200 | sweep |
| Icon sản phẩm quanh donut | opacity 0→1, scale 0.3→1 | 1000 + i × 70 | 600 | spring-pop |
| Nhãn % và đường nối | opacity 0→1 | 1200 + i × 60 | 450 | ease |
| Thanh điều hướng dưới | opacity 0→1, y +60→0, scale 0.9→1 | 900 | 700 | spring-tabbar |

- **Quy tắc (Home):** mọi khối vào cùng một lượt, so le tối đa 6 bậc (i ≤ 5); không có khối nào hiện riêng khi cuộn tới.
- **Nghiệm thu:**
  - [ ] Tiêu đề màn hiện ngay, không rơi.
  - [ ] Icon donut bật sau khi cung đã quét gần xong (~1 s).
  - [ ] Reduce Motion: mọi phần tử hiện ngay.

### M02 – Đếm số tiền tổng
- **Trigger:** cùng M01 (delay 250), đổi tab (M08, 700 ms), đổi tầng (M09, 900 ms).
- **Giá trị:** 0 → số thật trong 1300 ms, easing ease-out-quart, delay 250 ms. Định dạng `#,###` (dấu phẩy), số dạng tabular (không nhảy bề ngang).
- **Quy tắc:**
  - Mỗi tab chỉ đếm 1 lần trong một phiên vào màn. Quay lại tab đã đếm → hiện ngay số cuối.
  - Không đếm khi header đang thu gọn (p = 1) hoặc đang ẩn số dư (M10).
- **Nghiệm thu:**
  - [ ] Vào lại tab Tài sản lần 2 trong cùng phiên không đếm lại.
  - [ ] Đang ẩn số dư: hiện "******", không đếm.

### M03 – Thu gọn header theo cuộn (scroll-driven)
- **Đầu vào:** `p = clamp(scrollY / 130, 0, 1)`. Áp dụng như nhau cho tab Tài sản và Phải trả.
- **Thuộc tính (nội suy tuyến tính theo p trừ khi ghi khác):**

| Phần tử | Công thức |
|---|---|
| Tiêu đề "Quản lý tài chính" (y = 53) | opacity = clamp(1 − 2.5p, 0, 1); translateY = −8p |
| Tab Tài sản/Phải trả (x = 64) | y = 112 − 64p (112 → 48, nằm giữa navbar) |
| Nhãn "Tổng tài sản" | x: căn giữa → 20; y = 176 − 84p (→ 92) |
| Số tiền tổng (32 pt) | x: căn giữa → 20; y = 204 − 84p (→ 120); scale = 1 − 0.3125p (32 → 22 pt) |
| Dòng đồng bộ giữa màn (y = 258) | opacity = clamp(1 − 2.5p, 0, 1) |
| Dòng đồng bộ góc phải (2 dòng, right 11, y = 107) | opacity = clamp((p − 0.7) / 0.3, 0, 1); translateY = 6(1 − p) |
| Nền header 375×162, bo dưới 24 | opacity = clamp((p − 0.5) × 2, 0, 1); translateY = −14(1 − p) |

- Pseudo-code:
```text
onScroll(y):
  p = clamp(y / 130, 0, 1)
  title.alpha   = clamp(1 - 2.5p)
  tabs.y        = lerp(112, 48, p)
  label.y       = lerp(176, 92, p);  label.x  = lerp(centerX(label), 20, p)
  amount.y      = lerp(204, 120, p); amount.x = lerp(centerX(amount), 20, p); amount.scale = lerp(1, 0.6875, p)
  syncCenter.alpha  = clamp(1 - 2.5p)
  syncCorner.alpha  = clamp((p - 0.7) / 0.3); syncCorner.translateY = 6(1 - p)
  headerBg.alpha    = clamp((p - 0.5) * 2);   headerBg.translateY  = -14(1 - p)
```
- **Nghiệm thu:**
  - [ ] Cuộn chậm từng px thì header đổi từng px, không có bước nhảy.
  - [ ] Ở y = 130 nội dung đầu tiên nằm đúng y = 184.

### M04 – Snap header khi thả tay
- **Trigger:** ngừng cuộn 140 ms và 0 < p < 1.
- **Đích:** theo điểm dừng dự kiến `scrollY + project(v, 0.99)` (v = vận tốc cuộn gần nhất, px/s): ≥ 65 → 130 (thu gọn hẳn), < 65 → 0.
- **Chuyển động:** spring `page` (0.38, ζ 1) nhận vận tốc cuộn hiện tại, không nảy. Hai tab như nhau.
- **Quy tắc:** không snap khi người dùng đang giữ tay hoặc khi một animation cuộn khác đang chạy.

### M05 – Ẩn/hiện thanh điều hướng dưới khi cuộn
- Cuộn xuống > 4 px mỗi lần và scrollY > 40 → ẩn: translateY 0→110, opacity 1→0.
- Cuộn lên > 4 px → hiện lại.
- translate 450 ms ease-out-standard; opacity 300 ms.

### M06 – Thẻ tài sản co giãn theo vận tốc cuộn (chỉ tab Tài sản)
- `v` = vận tốc cuộn (px/ms), làm mượt mỗi frame: `sv += (v − sv) × 0.2; v *= 0.8`.
- `pull = clamp(sv / 1.1, −1, 1) × (p < 1 ? 1 : 0.45)`.
- Áp: mép trên thẻ dịch −12 × pull; nội dung thẻ +5 × pull; donut +7 × pull.
- Bóng đổ thẻ: offsetY −(6 + 10|pull|), blur 14 + 20|pull|, alpha 0.03 + 0.07|pull|.
- Ngừng cuộn → về 0. Nhấn giữ lên thẻ → |pull| = 0.6 (cảm giác "túm" thẻ).

### M07 – Khối bên dưới (Tổng quan tháng này, Hôm nay có gì mới…)
- Đã bỏ "hiện khi cuộn tới" để giống Home: Home không hiện từng phần.
- Các khối này vào cùng lượt intro (M01): opacity 0→1, y +8→0, trễ 80 + min(i, 5) × 40 ms (thực tế 280 ms vì i ≥ 5).

### M08 – Đổi tab Tài sản ↔ Phải trả
- **Trigger:** chạm tab.
- **Trình tự:**

| Bước | Phần tử | Từ → Đến | Delay | Duration | Easing |
|---|---|---|---|---|---|
| Ra | Nội dung tab cũ (cả khối) | opacity 1→0, blur 0→2 pt | 0 | 180 | ease-out bậc 3 |
| Ra | Nhãn + số tiền tổng | opacity 1→0 | 0 | 180 | ease-out bậc 3 |
| Đổi | Ẩn tab cũ, hiện tab mới, đặt scrollY (xem Quy tắc) | – | 180 | 0 | – |
| Vào | Nội dung tab mới (cả khối) | opacity 0→1, blur 2→0 pt | 180 | 260 | ease-out bậc 3 |
| Vào | Nhãn ("Tổng tài sản" ↔ "Dư nợ tháng này") + số tiền | opacity 0→1 | 180 | 260 | ease-out bậc 3 |
| Song song | Viên chỉ báo tab | trượt sang tab mới | 0 | spring indicator (0.30, ζ 1) | – |
| Song song | Màu chữ tab | đổi màu | 0 | 300 | ease |

  Theo tab Đề xuất/Yêu thích của Home: không trượt ngang, không cho từng dòng chạy lẻ, không chạy lại donut.
- **Quy tắc giữ header (quan trọng, tránh giật):**
  1. Header KHÔNG đổi trạng thái khi đổi tab: p trước và sau khi đổi phải bằng nhau.
  2. Mỗi tab nhớ scrollY riêng (`tabY[tab]`).
  3. Lúc đổi: `keepY = min(scrollY, 130)`.
     - Nếu `keepY = 130` (đang thu gọn): scrollY tab mới = `max(130, tabY[tabMới])`.
     - Nếu không: scrollY tab mới = `keepY`.
  4. Nội dung mỗi tab cao tối thiểu 942 (812 + 130) để tab ngắn (Phải trả) vẫn cuộn được tới 130.
- **Ngắt giữa chừng:** bỏ qua chạm tab khi đang đổi tab (khoá trong 180 ms đầu).
- **Nghiệm thu:**
  - [ ] Đang thu gọn ở Tài sản, chạm Phải trả: thanh tab đứng yên trong navbar, header vẫn thu gọn, 2 thẻ Phải trả nằm ở y = 184.
  - [ ] Từ Phải trả quay lại Tài sản: về đúng vị trí cuộn cũ của Tài sản.
  - [ ] Đang cuộn dở (0 < p < 1): đổi tab không làm header nhảy.

### M09 – Đổi tầng người dùng (khi dữ liệu thay đổi / tải lại)
1. Nội dung cũ: opacity 1→0, y 0→10, 160 ms.
2. Đặt scrollY = 0, hiện lại thanh điều hướng.
3. Nội dung mới: opacity 0→1, y 16→0, 380 ms ease-out-standard.
4. Chạy lại M01; đếm số 900 ms.

### M10 – Ẩn/hiện số dư (icon mắt)
- **Phạm vi ẩn:** số tiền tổng, số tiền và chênh lệch từng tài sản, số dư nguồn tiền. KHÔNG ẩn số ở "Tổng quan tháng này" (theo BRD và Figma).
- **Hiệu ứng trên từng số bị ẩn:** 300 ms: blur 0→3→0, opacity 1→0.4→1. Đổi text giữa số thật và "******" tại 120 ms.
- **Icon mắt:** nét gạch chéo dài 0→19 (ẩn) / 19→0 (hiện), xoay −40°, 250 ms ease-out-standard.
- Đang ẩn: không chạy M02.

### M11 – Nút đồng bộ
- Chạm icon reload cạnh "Đồng bộ lúc…": icon xoay −360° (ngược chiều kim đồng hồ), 800 ms, easing (0.4, 0, 0.2, 1).
- Đề xuất khi gắn API (BRD chưa mô tả, cần BA/PO xác nhận): xoay lặp tới khi có kết quả → cập nhật giờ đồng bộ → toast (M16). Khoá nút khi đang tải.

### M12 – Banner "Hôm nay có gì mới" (carousel)
- Thẻ giữa 343×130; hai thẻ bên scale 0.75; bước trượt 308.125.
- Scale thẻ giữa/bên đổi cùng lúc với vị trí.
- Tự chạy mỗi 5 s, lặp vô hạn liền mạch; dừng khi người dùng đang kéo, chạy lại sau khi thả.
- Tự chạy: chuyển thẻ bằng spring `page` (0.38, ζ 1).
- Kéo (Home): khoá trục sau 10 pt; đi dọc trước → nhường cho cuộn trang. Thẻ bám ngón tay.
- Thả: trang đích = làm tròn `(dx + project(v, 0.99)) / bước`, tối đa ±1 trang; spring `page` nhận vận tốc tay.
- Chấm phân trang: chấm hiện tại rộng 4→16, màu #E9E9E9 → đỏ thương hiệu, spring `dots` (0.12, ζ 1).

### M13 – Màn con: push và modal
| Kiểu | Mở | Đóng | Màn dưới |
|---|---|---|---|
| Push (từ phải) | x 100%→0, spring push (0.5, ζ 1) | spring push | tối tới 10%, lùi x 0 → −112 pt cùng spring (Home) |
| Modal (từ dưới) | y 100%→0, spring push | spring push | tối tới 10%, không lùi |

- Mở bằng chạm: nhả tay xong 100 ms mới bắt đầu (Home `nav-delay`), để kịp thấy nút nảy về.
- Nội dung màn con: từng khối opacity 0→1, y 14→0, 520 ms, delay 140 + i × 45 ms.
- Vuốt cạnh trái để quay lại (chỉ push): bắt đầu trong 28 pt sát mép trái, nhận khi kéo ngang > 6. Màn con và màn dưới (−112 → 0) bám ngón tay.
- Thả: đóng nếu `dx + project(v, 0.998) > 187.5`; spring push nhận vận tốc tay (`v0 = v / (375 − dx)`). Không đủ → spring trả về.

### M14 – Bottom sheet
- Mở 520 ms, easing (0.2, 1.1, 0.3, 1) (nảy nhẹ); đóng 340 ms ease-ios-sheet.
- Lớp phủ: hiện 360 ms, ẩn 300 ms.
- Từng dòng trong sheet: opacity 0→1, y +12→0, 420 ms, delay 120 + i × 35 ms.
- Kéo xuống: đóng khi kéo > 140 hoặc vuốt nhanh xuống; không thì trả về 320 ms.

### M15 – Popup onboarding
- Lớp phủ đen 60%, 300 ms.
- Popup: y (100% + 90)→0, 550 ms spring-soft.
- Checkbox khi tick: bật scale, 200 ms, easing (0.2, 0.9, 0.3, 1.3).

### M16 – Toast
- Vào: opacity 250 ms, y +12→0 350 ms ease-out-standard. Giữ 1800 ms. Ra: ngược lại.

### M17 – Quản lý thu chi
| Phần tử | Từ → Đến | Delay | Duration | Easing |
|---|---|---|---|---|
| Tab Chi tiêu/Thu nhập (bìa hồ sơ) | nền tab chọn biến hình sang tab kia | 0 | 520 | spring-folder |
| Donut | quét 0°→360° khi vào màn, đổi tab, đổi tháng | 0 | 900 | ease-carousel |
| Dòng danh mục/giao dịch | opacity 0→1, y +12→0 | 80 + i × 40 | 380 | ease-out-standard |
| Nhãn tháng | tháng cũ ra, tháng mới vào ±12 theo hướng bấm | 0 | 300 | ease-out-standard |
| Mũi tên đổi tháng | press scale 0.85 | 0 | 150 | ease |

### M18 – Bộ chọn tháng (date picker, bottom sheet)
- Mở 560 ms easing (0.2, 1.15, 0.3, 1); đóng 320 ms. Lớp phủ 320 / 260 ms.
- Nút trong sheet: scale 0.6→1, 460 ms easing (0.3, 1.5, 0.5, 1).
- Bánh xe kiểu iOS, hàng cao 32. Với d = khoảng cách (số hàng) tới hàng đang chọn:
  `rotateX = −d × 20°`, `scale = 1 − |d| × 0.09`, `opacity = 1 − |d| × 0.26`. Thả tay → khớp hàng gần nhất.

### M19 – Hub vay
- Navbar: scrollY > 6 → nền trong suốt → trắng 72% + blur 24, 300 ms; về đầu trang thì ngược lại.
- Mascot: nhún y ±4, chu kỳ 3.2 s, lặp vô hạn, ease-in-out.

### M20 – FAQ (accordion)
- Chạm câu hỏi: chiều cao mở/đóng 350 ms ease-ios-sheet; mũi tên xoay 90°→−90°; câu trả lời fade in sau 80 ms.

### M21 – Tab màn Sản phẩm
- Chạm tab → nội dung cuộn ngang mượt (smooth scroll hệ thống) tới trang của tab.

### M22 – Trạng thái nhấn (dùng chung, theo Home)
- Lún ngay khi chạm (pointer-down), spring `press` (0.18, ζ 1):
  - Thẻ, dòng, thẻ tổng quan, thẻ khoản vay: 0.97.
  - Nút CTA (Kích hoạt, Tôi đồng ý, Vay ngay…): 0.94.
  - Nút icon (back, ?, mũi tên tháng, đồng bộ, chevron 20 pt): 0.88.
- Nhả tay: nảy về 1 bằng spring `bounce` (0.38, ζ 0.5). Mở màn/sheet sau 100 ms.
- Tay đi quá 10 pt (đang cuộn/vuốt) hoặc huỷ → về 1 bằng spring `press`, không mở gì.
- Thanh điều hướng dưới: cả thanh lún 0.97 (`bar-down` / `bar-up`), nút được chạm lún 0.88 (`tab-down` / `tab-up`); nhả → rung nhẹ (haptic light) + mở màn.
- Reduce Motion: không scale, opacity 0.55 (thanh điều hướng 0.6).

---

## 4. Quy tắc chung khi hiện thực
1. Animation theo cuộn (M03, M06) phải chạy trên luồng render/UI mỗi frame, không debounce.
2. Animation mới được phép ngắt animation cũ của cùng phần tử, bắt đầu từ giá trị + vận tốc hiện tại (không reset về điểm đầu). Ưu tiên spring cho mọi thứ người dùng chạm/kéo được.
5. Khi dữ liệu đổi (đồng bộ, làm mới ngầm): crossfade cả khối 240 ms, không đổi lẻ từng chữ/số (quy tắc Home); chỉ đếm số ở lần vào đầu.
3. Không chạy intro/đếm số lại khi chỉ quay về từ màn con (M13).
4. Tất cả chạm trong lúc màn con đang mở/đóng bị bỏ qua cho tới khi animation xong.

## 5. Câu hỏi còn mở
- Haptic: đã có cho thanh điều hướng (theo Home). Snap header, đổi tab, tick checkbox: Home chưa quy định, chưa thêm.
- Snap header vẫn theo cuộn (scrub) vì header Quản lý tài chính có số tiền lớn thu nhỏ dần; Home dùng ngưỡng 40/24 pt + spring cho thanh top. Cần designer Home xác nhận giữ khác biệt này.
- Thời lượng khối vào (M01, 600 ms): Home chỉ ghi trễ 80 + 40·i và lệch 8 pt, chưa ghi thời lượng – cần đối chiếu file tokens của Home.
- Trạng thái đang tải (skeleton) khi đồng bộ: chưa có trên prototype/Figma.
- Hành vi chi tiết của nút đồng bộ (M11) khi lỗi / giới hạn số lần bấm: BRD chưa mô tả.

## 6. Tham chiếu
- Prototype: https://demo-personal-finance-pink.vercel.app — tham số `?tang=1|2a|2b|3|3b` để xem từng tầng người dùng.
- Figma page Phase 1: 807:22203. Bảng spec trực quan: section "Spec animation" trong page này.
- Motion Home VTM 9.9: Figma owTkAz1FHxoyqdukaqmWDy, page "Home - Phase 1 (19.10.2026)" (thanh top, thanh điều hướng, tab dịch vụ, banner, loading).
- Component liên quan (page Component): Navbar (Chưa scroll / Đã scroll), Tổng số dư (Trạng thái Chưa scroll / Đã scroll), Thẻ tài sản, Tổng quan tháng này, Hôm nay có gì mới, Thanh điều hướng dưới.
