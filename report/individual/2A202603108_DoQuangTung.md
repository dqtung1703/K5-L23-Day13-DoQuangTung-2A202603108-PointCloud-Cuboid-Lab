# Báo cáo Day 13 — Đỗ Quang Tùng, 2A202603108

## 1. Kết luận trước
- **Thông tin:** MSSV `2A202603108`, task `D13-2A202603108`, 10 frame: `s01f01`, `s01f23`, `s02f08`, `s02f52`, `s03f15`, `s03f46`, `s04f07`, `s04f56`, `s05f08`, `s05f33`. Hệ điều hành Windows 11, trình duyệt Chrome/Edge, CVAT local bản `2.81.0` (chạy cổng `9080`). Bạn ngồi cạnh: làm bài cá nhân tại nhà/độc lập (chưa có MSSV bạn ngồi cạnh trực tiếp để đổi kiểm).
- **Tổng quan kết quả:** Tổng cộng đã vẽ **61 cuboid** (dạng Shape) trên 10 frame; các frame có ít cuboid nhất là `s02f08`, `s02f52`, `s03f15`, `s03f46`, `s04f07`, `s04f56`, `s05f08`, `s05f33` (mỗi frame 5 cuboid) vì ưu tiên gán các vật ở gần, cụm điểm rõ trong tầm 30 m thay vì đoán các vật quá thưa ở xa. Sau khi sửa 6 cuboid bị cờ quá rộng, `selfcheck` chỉ còn cờ **1/61 cuboid** (`s01f01 #5 Pedestrian`). Phần tự tin nhất là xác định hướng đầu xe (yaw), phân loại class và căn đáy hộp chạm mặt đường; phần còn yếu nhất là ước lượng kích thước với các vật ở xa (> 30 m) bị thưa điểm hoặc bị che khuất một phần.
- **Cách dùng CVAT 3D:**
  1. **Đổi góc nhìn:** Trong khung 3D (Perspective), kéo chuột trái để xoay (xoay thẳng xuống để nhìn toàn cảnh từ trên xuống), kéo chuột phải để dời khung nhìn, lăn con lăn để phóng to/thu nhỏ, nháy đúp chỗ trống để về góc nhìn mặc định.
  2. **Vẽ và chỉnh cuboid:** Chọn công cụ Cuboid → chọn class → bấm **Shape** (không bấm Track), đưa hộp tới cụm điểm và nháy đúp để đặt (phím `N` để vẽ tiếp cùng class). Chọn cuboid để chỉnh tâm, dài, rộng, góc yaw ở khung **Top**, và chỉ dời lên/xuống, chỉnh chiều cao đáy chạm đường ở khung **Side** và **Front** (không xoay ở Side/Front).
  3. **Bật hướng và kiểm trên ảnh camera:** Bật **Appearance → Cuboid orientation** để hiện vạch đỏ ngắn (`+X` của cuboid) và xoay vạch đỏ chỉ về đầu xe; sau khi bấm Save (`Ctrl+S`), chạy `python scripts/lab.py preview` để chiếu cuboid lên 7 ảnh camera và ảnh nhìn từ trên xuống trong `work/xem/`.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ. Ảnh: `t1-s01f01-3d.png`, `t1-s01f01-top.png`, `t1-s01f23-3d.png`, `t1-s01f23-top.png` (ảnh nhìn từ trên xuống đã đánh dấu phía trước `-y`, `+x`, `+y`).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f01` | Phía trước (`-y`) ở cạnh trên của ảnh; `+x` chỉ sang trái, `+y` chỉ xuống dưới (ra sau xe) | Mặt đường khá phẳng, thấp hơn LiDAR khoảng 1,7 m | Khoảng 40–45 m (xe tải và xe buýt lớn vẫn nhận ra khung thân) | Car, Truck, Bus, Pedestrian, Cyclist | Phía sau có nhiều xe con đi cùng chiều (`#2`, `#7`, `#8` ở khoảng 9–17 m); hai bên có người đi bộ (`#4`, `#5`), xe hai bánh (`#6`) và xe buýt phía bên phải (`#15`) | Một số cụm điểm nhỏ rời rạc trên vỉa hè/phía xa > 45 m rất thưa, không phân biệt rõ là người hay cột/cây |
| `s01f23` | Phía trước (`-y`) ở cạnh trên của ảnh; `+x` chỉ sang trái, `+y` chỉ xuống dưới (ra sau xe) | Mặt đường tương đối phẳng, thấp hơn LiDAR khoảng 1,7 m | Khoảng 35–40 m | Car, Bus | Phía sau và bên trái có xe con (`#3`, `#6`) và xe buýt (`#5`, cách khoảng 13,8 m) không xuất hiện trong `cam01` | Cụm điểm lẻ nằm sâu dưới mặt đường (điểm lạ) và các cụm điểm xa bị tán xạ mờ |

- **3 điều chỉ camera cho biết:** (1) Màu sắc thân xe và nhận diện rõ đầu/đuôi xe (đèn pha, đèn hậu); (2) Phân biệt chi tiết loại phương tiện khi hình khối gần giống nhau (xe bán tải với SUV, người dắt xe với người lái xe); (3) Biển báo, đèn tín hiệu, vạch kẻ đường và vật ở góc khuất phản xạ kém.
- **3 điều chỉ LiDAR cho biết:** (1) Khoảng cách chính xác theo mét từ xe thu dữ liệu tới từng vật; (2) Kích thước 3D thực tế (dài, rộng, cao theo mét) và độ cao mặt đường (`z ≈ -1,7 m`); (3) Các vật nằm trong vùng mù giữa 7 camera nhờ góc quét liên tục 360°.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f08` | Cạnh trên (`-y` hướng lên, `+x` sang trái, `+y` xuống dưới) | Có xe buýt lớn ở gần gây bóng che phía sau |
| `s02f52` | Cạnh trên (`-y` hướng lên, `+x` sang trái, `+y` xuống dưới) | Các xe chạy hai chiều sát nhau, dễ nhầm hướng nếu không đối chiếu camera |
| `s03f15` | Cạnh trên (`-y` hướng lên, `+x` sang trái, `+y` xuống dưới) | Có xe tải ở làn kế bên gần tâm LiDAR |
| `s03f46` | Cạnh trên (`-y` hướng lên, `+x` sang trái, `+y` xuống dưới) | Có xe buýt ở xa ngoài 50 m (`~60,6 m`) điểm khá thưa |
| `s04f07` | Cạnh trên (`-y` hướng lên, `+x` sang trái, `+y` xuống dưới) | Xe tải ngược chiều gần bên trái có mật độ điểm dày |
| `s04f56` | Cạnh trên (`-y` hướng lên, `+x` sang trái, `+y` xuống dưới) | Có xe con ở phía sau khá xa (`~38,1 m`), mật độ điểm giảm mạnh |
| `s05f08` | Cạnh trên (`-y` hướng lên, `+x` sang trái, `+y` xuống dưới) | Đường nhiều xe con đi cả hai chiều ở tầm 10–25 m |
| `s05f33` | Cạnh trên (`-y` hướng lên, `+x` sang trái, `+y` xuống dưới) | Xe tải lớn ngược chiều bên trái có lượng điểm rất dày (> 6000 điểm) |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f01`, `#1` | Car | Đúng (`268.4°`, cùng chiều xe thu dữ liệu) | Đúng (`z_bottom = -1.908 m`) | Đã kiểm đủ 3 view + `preview` | Đúng (`4.770 × 2.035 × 1.660 m`, khớp SUV/minivan) |
| (b) xe buýt hoặc xe tải | `s01f23`, `#5` | Bus | Đúng (`270.0°`, cùng chiều xe thu dữ liệu) | Đúng (`z_bottom = -1.737 m`) | Đã kiểm đủ 3 view + `preview` | Dài (`12.497 m`) và cao (`3.501 m`) khớp bảng 4.3; rộng (`3.010 m`) hơi nhỉnh hơn `2.60 m` do gương/mép điểm |
| (c) người đi bộ hoặc xe hai bánh | `s01f01`, `#4` | Pedestrian | Đúng (`80.2°`, theo hướng di chuyển) | Đúng (`z_bottom = -1.758 m`) | Đã kiểm đủ 3 view + `preview` | Đúng (`0.713 × 0.575 × 1.773 m`, trong khoảng người lớn) |

- **Vòng hiệu chuẩn:** Theo hướng dẫn của lab coach trong buổi học, lớp không chạy phần tạo task chung `--code` của vòng hiệu chuẩn mà tập trung vẽ trực tiếp trên task cá nhân `D13-2A202603108` và tự kiểm theo guideline.
- **Vật bỏ qua vì không đủ cơ sở:** Một số cụm điểm quá thưa ở khoảng cách xa (> 40–50 m) trên các frame `s01f01`, `s04f56` chỉ còn vài điểm rời rạc, không thấy đủ 15–20% thể tích cuboid và khó xác nhận hình khối (`LOW_VISIBILITY` / `UNCERTAIN_SCOPE`) nên đã bỏ qua, không đoán.

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s01f01` / `t4-1-bong-che.png` / frame của bạn | Giả thuyết: thân xe buýt lớn nằm chắn tia LiDAR nên các tia không thể xuyên qua tới vùng phía sau xe | Ngay phía khuất sau thân xe buýt xuất hiện vùng trống hình quạt hầu như không có điểm, trong khi hai bên vẫn dày điểm | Vùng trống là vùng chưa quan sát được; nếu có vật bị che một phần thì bật `occluded` và vẽ cuboid bao trọn cả phần suy luận được |
| 2. Thưa ở xa | `s01f01` / `t4-2-thua-o-xa.png` / frame của bạn | Giả thuyết: các tia LiDAR phân kỳ theo góc quét nên ở khoảng cách xa (30–50 m) mật độ tia trúng bề mặt vật giảm mạnh | Xe ở xa chỉ còn vài chục điểm rời rạc theo từng vệt ngang, không hiện rõ đầy đủ mặt bên và nóc xe | Chỉ vẽ nếu nhìn thấy khoảng 15–20% thể tích hoặc có đủ cơ sở suy luận từ camera và bảng 4.3; nếu không đủ cơ sở thì bỏ qua và ghi `LOW_VISIBILITY` |
| 3. Phản xạ thấp / ít điểm bất thường | `s01f01` / `t4-3-phan-xa-thap.png` / frame của bạn | Giả thuyết: bề mặt xe tối màu hoặc phần kính hấp thụ/phản xạ lệch tia laser nên cường độ phản xạ yếu | Vật có ít điểm hơn hẳn và vùng thân/kính bị khuyết điểm so với xe sáng màu ở cùng tầm khoảng cách | Ghi nhận là giả thuyết (không khẳng định chắc chắn nguyên nhân); đối chiếu ảnh camera và bảng 4.3 để bao đủ thân xe, không cắt cụt hộp theo phần bị khuyết điểm |
| 4. Điểm lạ | `s01f23` / `t4-4-diem-la.png` / frame của bạn | Giả thuyết: phản xạ đa đường (multipath) từ mặt đường/bề mặt bóng hoặc nhiễu cảm biến khiến phép đo thời gian bay bị sai lệch | Xuất hiện cụm điểm nằm tách biệt hẳn bên dưới mặt phẳng mặt đường, không gắn với vật thể nào trên ảnh camera | Không gán cuboid thành vật thể thật, bỏ qua khi vẽ |

## 5. Số từ cuboid của tôi (Bước 5)

| Cuboid (frame, idx, class) | z (m) | h (m) | z đáy = z − h/2 (m) | dist_m (m) | r = √(x²+y²+z²) (m) | yaw_deg (°) | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f01`, `#1`, Car | `-1.079` | `1.660` | `-1.908` | `4.39` | `4.52` | `268.4°` | Chạm đường (lệch `0.21 m ≤ 0.3 m` so với `-1.7 m`) |
| `s01f01`, `#7`, Car | `-0.930` | `1.596` | `-1.727` | `14.19` | `14.22` | `268.4°` | Chạm đường (lệch `0.03 m` so với `-1.7 m`) |
| `s02f08`, `#2`, Bus | `-0.188` | `3.414` | `-1.895` | `18.03` | `18.03` | `85.9°` | Chạm đường (lệch `0.20 m ≤ 0.3 m` so với `-1.7 m`) |

- **(a) Cuboid gần nhất và xa nhất:**
  - **Gần nhất:** `s01f23 #1 (Car)` có `dist_m = 4.21 m` và khoảng cách 3D `r = 4.35 m`.
  - **Xa nhất:** `s03f46 #4 (Bus)` có `dist_m = 60.65 m` và `r = 60.65 m` *(lưu ý: hộp này vượt ngưỡng 50 m nên nằm bin `>=50` và không được chấm; cuboid xa nhất trong phạm vi ≤ 50 m là `s01f01 #11 Truck` với `dist_m = 41.50 m`, `r = 41.50 m`)*.
- **(b) So kích thước với bảng 4.3:**
  - **Người đi bộ (`s01f01 #4, Pedestrian`):** `l = 0.713 m`, `w = 0.575 m`, `h = 1.773 m`. So với bảng 4.3 cho người lớn (`0.4–0.9 m × 0.5–0.9 m × 1.50–1.95 m`), cả 3 chiều đều nằm gọn trong khoảng tham chiếu.
  - **Xe con (`s01f01 #1, Car`):** `l = 4.770 m`, `w = 2.035 m`, `h = 1.660 m`. So với bảng 4.3 cho SUV/minivan (`4.4–5.3 m × 1.80–2.10 m × 1.65–2.10 m`), cả 3 chiều đều nằm trong khoảng chuẩn (nếu đối chiếu nhóm sedan `w = 1.70–1.90 m` thì chiều rộng nhỉnh hơn khoảng `0.13 m` do bao gồm hai gương chiếu hậu).
- **(c) Hướng `yaw_deg`:**
  - Xe cùng chiều phía trước `s02f08 #3 (Car)` có `yaw_deg = 270.7°` (rất gần `270°`, khớp hướng `-y`).
  - Xe đi ngược chiều `s02f52 #4 (Car)` có `yaw_deg = 89.8°` (rất gần `90°`, khớp hướng `+y`).

Mật độ theo khoảng cách (4 cuboid `Car` ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m (m) | Số điểm (`points_in`) |
|---|---|---|
| `s01f01`, `#1`, Car | `4.39` | `2977` |
| `s01f01`, `#7`, Car | `14.19` | `648` |
| `s02f08`, `#4`, Car | `28.09` | `221` |
| `s04f56`, `#5`, Car | `38.14` | `85` |

- **Quy luật mật độ:** Khi khoảng cách tăng khoảng 3,2 lần (từ `4.39 m` lên `14.19 m`), số điểm giảm khoảng 4,6 lần (từ `2977` xuống `648` điểm); khi tăng tiếp gấp đôi lên `28.09 m`, số điểm giảm gần 3 lần còn `221` điểm, và tới `38.14 m` chỉ còn `85` điểm — bám rất sát đường cong trung vị trên slide bài giảng (khoảng `1968` điểm ở 0–10 m, `718` ở 10–20 m, `214` ở 20–30 m, `72` ở 30–40 m).
- **Tự kiểm và sửa cuboid:** Sau lần `sync` đầu tiên có 7/60 hộp bị cờ, đã mở CVAT sửa gọn chiều rộng (`width`) cho 6 cuboid bị quá rộng: `s02f08 #2 (Bus)` giảm từ `3.30 m` xuống `2.600 m`, `s03f46 #4 (Bus)` giảm từ `3.60 m` xuống `3.200 m`, `s05f33 #5 (Truck)` giảm từ `3.46 m` xuống `2.600 m`, `s02f52 #3 (Car)` giảm từ `2.31 m` xuống `2.000 m`, `s04f07 #4 (Car)` giảm từ `2.46 m` xuống `1.990 m`, và `s05f08 #5 (Car)` giảm từ `2.35 m` xuống `2.100 m`, sau đó Save và chạy lại `sync`. Cảnh báo duy nhất giữ nguyên là `s01f01 #5 (Pedestrian)` (`46 điểm sát ngay ngoài mép`) vì hộp đã ôm vừa kích thước người đi bộ (`0.651 × 0.493 × 1.590 m`), các điểm sát ngoài mép thuộc vật/người kế bên nên giữ nguyên không nới hộp.
- **Đổi kiểm:** Làm bài độc lập sau giờ học trên lớp nên chưa có bạn ngồi cạnh đối chiếu chéo trực tiếp; đã tự kiểm lại toàn bộ các con số trong Mục 5 đối chiếu trực tiếp với `submission/D13-2A202603108-bangA.csv` và `submission/D13-2A202603108-tukiem.csv` (100% khớp).

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
- **(a) Ý nghĩa của nhãn đối với model 3D:**
  - Nhãn 3D đo bằng **mét** (thay vì pixel) vì xe tự hành cần biết khoảng cách, kích thước thực và vận tốc thực của vật cản trong không gian vật lý để lập kế hoạch quỹ đạo và phanh an toàn; cùng một chiếc xe nhưng ở xa hay gần sẽ có số pixel khác hẳn nhau trên ảnh 2D, còn kích thước theo mét là bất biến.
  - Vật thưa điểm cần quy tắc riêng (ngưỡng 15–20% thể tích, không đoán bừa) vì nếu gán nhãn phỏng đoán sai tâm/hướng/kích thước hoặc gán nhầm nhiễu thành vật, model học trên dữ liệu đó sẽ tăng tỉ lệ báo động giả (false positive) và dự đoán lệch hộp 3D nghiêm trọng.
- **(b) Cách dùng AI:**
  - **Công cụ:** Trợ lý AI trong IDE (Antigravity).
  - **Câu hỏi chính:** Tóm tắt các bước làm bài lab, lệnh PowerShell chạy môi trường/CVAT cổng 9080, cách xác định hệ tọa độ VF (`+x`, `+y`, `-y`), ý nghĩa các cờ cảnh báo của `selfcheck`, và hỗ trợ tổng hợp định dạng báo cáo từ file Bảng A/phiếu tự kiểm công khai.
  - **Cách sử dụng kết quả:** Tự thực hiện toàn bộ việc đọc frame, vẽ và chỉnh sửa 61 cuboid trên CVAT local, chụp 11 ảnh màn hình, kiểm tra lại các số liệu đối chiếu với `bangA.csv` và `tukiem.csv` trước khi nộp; không gửi ảnh camera hay dữ liệu `data-VF` cho AI.
- **(c) Điều còn chưa chắc:**
  - Cuboid `s03f46 #4 (Bus)` có `dist_m = 60.65 m` (vượt ranh 50 m, không được chấm) và `s01f01 #15 (Bus)` có `width = 2.993 m` hơi rộng so với bảng 4.3 do cụm điểm ở xa hơn 32 m hơi tán xạ.
  - Một số xe bị che khuất một phần trong 10 frame chưa được bật cờ `occluded`, sẽ cần chú ý kiểm tra thuộc tính này kỹ hơn ở các bài lab gán nhãn theo track (Ngày 24–26).
