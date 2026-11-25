# Báo cáo Day 13 — Nguyễn Viết Tiến, 2A202603140

## 1. Kết luận trước
- **MSSV:** `2A202603140`
- **Task CVAT:** `D13-2A202603140`
- **10 frame của tôi:** `s01f36`, `s01f53`, `s02f37`, `s02f54`, `s03f00`, `s03f32`, `s04f33`, `s04f49`, `s05f04`, `s05f21`
- **Hệ điều hành:** Windows
- **Trình duyệt:** Chrome
- **Phiên bản CVAT:** Server `2.81.0`, UI `2.81.0`
- **MSSV bạn ngồi cạnh:** 2A202603059

Tổng cộng tôi đã gán **67 cuboid** trên 10 frame (55 `Car`, 8 `Cyclist`, 2 `Truck`, 1 `Bus`, 1 `Pedestrian`); frame có ít cuboid nhất là `s03f32` (**5 cuboid**) do lưu lượng xe thưa, dải phân cách cây bụi cao bên trái che khuất làn ngược chiều và phía sau khá thoáng. Phiếu tự kiểm `selfcheck` gắn cờ **21 / 67 cuboid** (17 cuboid lệch nhẹ kích thước so với khoảng chuẩn bảng 4.3, chủ yếu ở khoảng cách xa hoặc xe cỡ lớn, và 4 cuboid lệch độ cao đáy so với mặt đường). Tôi tin cậy nhất phần hướng (`yaw`) và phân loại (`class`) ở các vật gần (< 20 m) nhờ đối chiếu trực tiếp trên 7 góc camera; phần còn yếu nhất là ước lượng kích thước (`width`/`length`) và đáy hộp (`z_bottom`) ở các vật xa (> 25–30 m) khi điểm LiDAR thưa và mặt đường có độ dốc cục bộ.

Cách dùng CVAT:
- **Đổi góc nhìn:** kéo chuột trái để xoay, kéo chuột phải hoặc dùng phím mũi tên / `U, I, O, J, K, L` để dời khung nhìn; dùng nút chuyển góc nhìn phía trên khung 3D để về chế độ nhìn từ trên xuống (Top view).
- **Vẽ và chỉnh cuboid:** chọn công cụ vẽ cuboid (`Shape`), đặt hộp lên cụm điểm rồi nháy đúp vào cuboid để mở 3 khung chiếu `Top / Side / Front`, kéo cạnh/góc và xoay hộp sao cho ôm sát cụm điểm và chạm mặt đường.
- **Bật mũi tên hướng và kiểm tra ảnh camera:** bật hiển thị hướng cuboid (**Cuboid orientation**) để mũi tên chỉ đúng đầu xe, đồng thời kiểm tra độ khớp hộp 3D trên các ảnh camera (`cam01`–`cam09`) và ảnh kết xuất từ lệnh `preview`.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ (`s01f36`, `s01f53`). Ảnh: `t1-s01f36-3d.png`, `t1-s01f36-top.png`, `t1-s01f53-3d.png`, `t1-s01f53-top.png` (ảnh nhìn từ trên xuống đã đánh dấu phía trước `-y`, `+x`, `+y`).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f36` | Phía trước (-y) ở cạnh trên; +x chỉ sang trái, +y chỉ xuống dưới | Khá phẳng, thấp hơn LiDAR ~1,7 m | ~48 m (xe van phía trước) | Car, cây, cột đèn | Xe con trắng phía sau bên phải (~12 m, thấy ở cam08), xe máy đi sau bên trái (~9 m, thấy ở cam05 và cam09) | Cụm điểm nhỏ, thưa sát lề phải cách ~38 m |
| `s01f53` | Phía trước (-y) ở cạnh trên; +x chỉ sang trái, +y chỉ xuống dưới | Khá phẳng, thấp hơn LiDAR ~1,7 m | ~30 m (xe van phía trước) | Car, Bus, dải phân cách, tường | Xe tải nhỏ ở làn bên trái ngang xe (~6 m, thấy ở cam06), xe con đi ngay phía sau (~8 m, thấy ở cam09) | Cụm điểm rải rác gần gốc cây bên trái cách ~30 m |

Từ hai frame `s01f36` và `s01f53`:
- **3 điều chỉ camera cho biết:** (1) màu sắc, kiểu dáng và chi tiết bề mặt phương tiện (xe con trắng/đen/bạc, xe tải nhỏ, xe hai bánh); (2) vạch kẻ đường, mũi tên chỉ hướng làn, dải phân cách trồng cây và hệ thống đèn/biển báo giao thông (`cam01`, `cam03`); (3) nhận diện rõ đối tượng nhỏ bị lẫn vào nền cây cối/lề đường (các xe hai bánh phía sau dải cây bên phải ở `cam07`, `cam08`).
- **3 điều chỉ LiDAR cho biết:** (1) khoảng cách và tọa độ 3D chính xác theo mét đến từng vật (ví dụ xe con phía trước ở `s01f53` cách ~9,6–10,3 m, xe xa ở `s01f36` cách ~44,2–46,2 m); (2) kích thước hình học thật (`length`, `width`, `height` bằng mét) và cao độ đáy vật so với mặt đường (`z_bottom` ~ −1,7 m); (3) cấu trúc hình học 360° nhìn từ trên xuống (BEV) và vùng bóng che phía sau vật cản mà ảnh 2D khó ước lượng chiều sâu.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f37` | Cạnh trên (-y ngược chiều trục xanh lá) | |
| `s02f54` | Cạnh trên (-y ngược chiều trục xanh lá) | |
| `s03f00` | Cạnh trên (-y ngược chiều trục xanh lá) | |
| `s03f32` | Cạnh trên (-y ngược chiều trục xanh lá) | |
| `s04f33` | Cạnh trên (-y ngược chiều trục xanh lá) | |
| `s04f49` | Cạnh trên (-y ngược chiều trục xanh lá) | |
| `s05f04` | Cạnh trên (-y ngược chiều trục xanh lá) | |
| `s05f21` | Cạnh trên (-y ngược chiều trục xanh lá) | |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f36`, `#1` | `Car` | Có, `yaw = 275,6°` | Có, `z_bottom = −1,735 m` (`ground_gap = 0,08 m`) | Có | Có: `l = 4,934 m`, `w = 1,694 m`, `h = 1,526 m` |
| (b) xe buýt hoặc xe tải | `s01f36`, `#8` | `Truck` | Có, `yaw = 89,8°` | Có, `z_bottom = −1,125 m` (`ground_gap = −0,14 m`) | Có | Có: `l = 6,529 m`, `w = 2,486 m`, `h = 2,347 m` |
| (c) người đi bộ hoặc xe hai bánh | `s01f36`, `#5` | `Cyclist` | Có, `yaw = 266,6°` | Có, `z_bottom = −1,452 m` (`ground_gap = −0,08 m`) | Có | Có: `l = 1,720 m`, `w = 0,889 m`, `h = 1,957 m` |

- **Vòng hiệu chuẩn (`D13-HC485087`):** Chưa tham gia vòng hiệu chuẩn chung; đã tự rà soát lại hướng yaw, đáy chạm đường và kích thước theo `GUIDELINE.md`.
- **Các vật bỏ qua vì không đủ cơ sở:** Đã bỏ qua cụm điểm thưa sát lề phải cách ~38 m ở s01f36 và cụm điểm gần gốc cây bên trái cách ~30 m ở s01f53 do không đủ 15–20% thể tích để suy luận (LOW_VISIBILITY).

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**. Frame mượn thì ghi rõ nguồn: task chung, hoặc MSSV của bạn cho mượn.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s05f04` / `t4-1-bong-che.png` / task cá nhân | Giả thuyết: xe con (`#2`, cách 3,31 m) bên phải và xe tải (`#6`, cách 13,56 m) phía sau chắn chùm tia LiDAR không truyền qua được | Xuất hiện vùng đen hình quạt trống điểm hoàn toàn phía sau thân xe `#2` và `#6` trên góc nhìn BEV | Vật nằm trong vùng bóng che bị mất một phần hoặc toàn bộ điểm; cần đối chiếu camera, bật `Occluded` và kéo cuboid bao trọn phần suy luận được nếu đủ cơ sở, không mặc định vùng trống là đường trống |
| 2. Thưa ở xa | `s01f36` (`#10`, 46,17 m) / `t4-2-thua-o-xa.png` / task cá nhân | Giả thuyết: góc phân giải giữa các tia LiDAR không đổi nên khoảng cách càng xa thì khoảng trống giữa các điểm càng lớn | Xe con `#10` ở 46,17 m chỉ còn 48 điểm rời rạc, không rõ đường biên khung xe (tương tự người đi bộ `#5` ở `s03f32` cách 38,78 m chỉ còn 22 điểm) | Theo quy tắc 15–20% thể tích: chỉ vẽ khi đủ cơ sở suy luận bằng camera và bảng 4.3; nếu quá thưa không đủ cơ sở thì bỏ qua và ghi `LOW_VISIBILITY` |
| 3. Phản xạ thấp / ít điểm bất thường | `s01f36` (`#9`, 21,58 m) / `t4-3-phan-xa-thap.png` / task cá nhân | Giả thuyết: bề mặt vật tối màu/kính hoặc góc tới lớn làm cường độ tia phản xạ về LiDAR yếu | Dù ở khoảng cách ~21,6 m nhưng chỉ ghi nhận 65 điểm (ít hơn nhiều so với mức ~300–350 điểm của các xe cùng khoảng cách) | Ghi nhận là giả thuyết (không khẳng định); tránh kéo hộp bị hụt kích thước bằng cách đối chiếu camera và bảng kích thước 4.3 |
| 4. Điểm lạ | `s03f32` (`#2`, 30,6 m) / `t4-4-diem-la.png` / task cá nhân | Giả thuyết: phản xạ đa đường từ kính ở trạm xe buýt hoặc nhiễu từ bụi cây gần đó | Xuất hiện các điểm nhiễu rời rạc quanh khu vực bụi cây và kính ở trạm xe buýt không thuộc thân vật thể | Không gán cuboid cho điểm lạ và không kéo rộng hộp để ôm các điểm nhiễu bám quanh thân xe |

## 5. Số từ cuboid của tôi (Bước 5)
Số liệu lấy từ `submission/D13-2A202603140-bangA.csv` và cột `points_in` của `submission/D13-2A202603140-tukiem.csv`.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f36`, `#1`, `Car` | −0,972 m | 1,526 m | −1,735 m | 10,11 m | 10,15 m | 275,6° | Chạm đường (`ground_gap = 0,08 m`) |
| `s01f36`, `#7`, `Cyclist` | −0,175 m | 1,890 m | −1,120 m | 19,11 m | 19,11 m | 276,7° | Lơ lửng (`ground_gap = 0,55 m`, cao hơn mặt đường cục bộ) |
| `s05f21`, `#4`, `Car` | −1,106 m | 2,062 m | −2,137 m | 21,07 m | 21,10 m | 277,0° | Chìm (`ground_gap = −0,49 m`, thấp hơn mặt đường cục bộ) |

- **(a) Gần nhất và xa nhất:** Cuboid gần nhất là `s05f04, #2, Car` với `dist_m = 3,31 m` và `r = 3,42 m`; cuboid xa nhất là `s01f36, #10, Car` với `dist_m = 46,17 m` và `r = 46,18 m`.
- **(b) So sánh kích thước với bảng 4.3:**
  - Xe hai bánh `s03f32, #4, Cyclist` có `l = 2,311 m`, `w = 1,483 m`, `h = 2,105 m`: chiều rộng `w = 1,483 m` và chiều cao `h = 2,105 m` vượt khoảng bảng 4.3 (`w: 0,70–0,95 m`, `h: 1,40–1,80 m` của xe máy có người) vì vật ở khá xa (`26,77 m`), cụm điểm thưa nên hộp bị kéo rộng theo phương ngang. (Với người đi bộ duy nhất `s03f32, #5, Pedestrian`: `l = 0,806 m`, `w = 1,181 m`, `h = 2,129 m` — `l` nằm trong khoảng `0,4–0,9 m`, còn `w` và `h` vượt nhẹ bảng 4.3 `0,5–0,9 m` và `1,50–1,95 m` do vật ở xa `38,78 m`, chỉ còn 22 điểm thưa nên hộp kéo hơi dư biên).
  - Xe con `s04f49, #3, Car` có `l = 6,041 m`, `w = 2,687 m`, `h = 2,158 m`: chiều dài (`6,041 m > 6,0 m`) và chiều rộng (`2,687 m > 2,3 m`) vượt khoảng chuẩn của `Car` do ở khoảng cách `22,28 m` chỉ quan sát rõ một góc thân xe nên hộp bị kéo dư biên.
- **(c) Góc `yaw_deg` theo chiều đi:** Xe đi cùng chiều phía trước `s01f53, #2, Car` có `yaw_deg = 271,0°` (rất gần `270°`); xe đi ngược chiều `s02f54, #3, Car` có `yaw_deg = 91,5°` và `s05f21, #7, Car` có `yaw_deg = 89,7°` (rất gần `90°`).

Mật độ theo khoảng cách (4 cuboid `Car` ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s03f00`, `#4`, `Car` | 4,98 m | 2461 |
| `s01f53`, `#2`, `Car` | 10,28 m | 904 |
| `s02f54`, `#4`, `Car` | 20,61 m | 351 |
| `s01f36`, `#3`, `Car` | 44,24 m | 63 |

Quy luật: khi khoảng cách tăng gấp ~2 lần (từ `4,98 m` lên `10,28 m`, rồi lên `20,61 m` và `44,24 m`), số điểm giảm rất nhanh xuống còn khoảng 1/3 đến 1/5 (từ `2461` xuống `904`, `351` và chỉ còn `63` điểm ở `44,24 m`). Xu hướng này khớp với đường cong ở slide bài giảng cho `Car` (trung bình ~1968 điểm ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m, 19 ở 60–80 m).

Phần tự kiểm (`preview` / `selfcheck`):
- **Hai cuboid đã sửa sau `preview` / `selfcheck`:** (1) `s03f32, #5, Pedestrian`: sửa chiều cao từ `3,11 m` xuống `2,13 m` và nâng đáy lên hết bị chìm (`0,33 m` → chạm đường); (2) `s05f04, #5, Car`: nâng đáy hộp lên hết bị chìm `0,29 m`; ngoài ra còn sửa `s01f36, #10, Car` mở rộng kích thước từ `2,12 m` lên `4,84 m` cho đúng kích thước xe con.
- **Các cảnh báo giữ nguyên (21 / 67 cuboid trong `-tukiem.csv`) kèm lý do:** 17 cảnh báo kích thước (`width`/`length`/`height` vượt ngưỡng nhẹ ở các xe xa > 20 m hoặc xe SUV/van/xe buýt như `s03f32 #2`, `s02f54 #1`, `s04f33 #4`) được giữ nguyên vì hộp đã ôm sát cụm điểm LiDAR thực tế và khớp trên ảnh camera; 4 cảnh báo lệch đáy (`s01f36 #7` lệch `+0,55 m`, `s01f36 #10` lệch `−0,30 m`, `s05f21 #4` lệch `−0,49 m`, `s05f21 #5` lệch `−0,38 m`) xuất hiện ở khu vực lề đường cao hoặc mặt đường có độ dốc ở xa nên cao độ mặt đường cục bộ khác mức chuẩn `−1,7 m`.

**Đổi kiểm:** Đã đổi kiểm trực tiếp trên màn hình với bạn ngồi cạnh (MSSV `2A202603059`), đối chiếu 2 con số ở Mục 5 (`dist_m = 3,31 m` của `s05f04, #2, Car` và `z_bottom = −1,735 m` của `s01f36, #1, Car`) với `submission/D13-2A202603140-bangA.csv` — cả 2 con số đều **khớp chính xác** với Bảng A (khi trao đổi thêm với bạn `2A202603059`, số đo tương ứng của bạn là `dist_m = 3,5 m` và `z_bottom = −1,705 m`, chênh lệch nhỏ nên không cần chỉnh sửa).

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
- **(a) Từ phía model học trên nhãn 3D:** Nhãn 3D phải đo bằng mét vì hệ tọa độ LiDAR phản ánh đúng vị trí, khoảng cách và kích thước hình học thực ngoài đời mà không bị biến dạng phối cảnh theo độ sâu như pixel trên ảnh 2D, giúp model dự đoán chính xác khoảng cách an toàn và quỹ đạo. Vật thưa điểm cần quy tắc gán riêng vì chỉ có vài chục điểm rời rạc không đủ xác định rõ biên hộp; nếu không thống nhất ngưỡng bỏ qua và cách ước lượng kích thước chuẩn thì nhãn sẽ nhiễu mạnh, làm giảm độ chính xác khi huấn luyện.
- **(b) Cách dùng AI:**
  - **Công cụ:** Trợ lý AI (Antigravity / Gemini).
  - **Câu hỏi chính:** Hỏi hướng dẫn từng bước làm lab, lệnh PowerShell/Python (`dry`, `task`, `sync`, `pack`, `check_submission.py`), cách thao tác CVAT 3D, khái niệm hệ tọa độ VF/yaw và rà soát cấu trúc báo cáo, tên file ảnh.
  - **Cách dùng kết quả:** Làm theo hướng dẫn thao tác và lệnh chạy trên máy cá nhân, tự gán nhãn trên CVAT, tự đối chiếu số liệu từ Bảng A/Phiếu tự kiểm và hoàn thiện báo cáo trước khi nộp.
- **(c) Điều còn chưa chắc:** Còn chưa thật chắc chắn về độ chính xác của chiều rộng và cao độ đáy đối với một vài xe ở xa trên 30–40 m (nơi mật độ điểm rất thưa và mặt đường có độ dốc cục bộ) cũng như một số xe hai bánh đi sát lề đường có cao độ nền chênh lệch so với làn xe chính.
