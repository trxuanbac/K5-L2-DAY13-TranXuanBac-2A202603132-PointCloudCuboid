# Báo cáo Day 13 — Trần Xuân Bắc, 2A202603132

## 1. Kết luận trước

Task `D13-2A202603132` chạy trên CVAT local 2.82.1, dùng Chrome trên macOS. Kết quả đồng bộ có **63 cuboid trên 10 frame**. Frame đầu `s01f13` có 8 hộp, `s03f10` có 7 hộp sau khi bổ sung một cuboid, tám frame còn lại có 6 hộp mỗi frame theo phân bổ đã chọn. Có 61 hộp trong 30 m và hai hộp trong khoảng 30–50 m. Phiếu tự kiểm có **1/63** hộp bị gắn cờ. Trong ba mẫu dưới đây, đáy của Car và Bus gần nền cục bộ nhất. Hướng mặt của người đi bộ và kích thước hộp ở xa vẫn cần xác nhận.

- **Góc nhìn.** Trong Perspective, kéo chuột trái để xoay, chuột phải để dời, dùng con lăn để thu phóng. Xoay nhìn thẳng xuống để quét quanh xe.
- **Cuboid.** Chọn label và `Shape`, nháy đúp vào cụm điểm. Chỉnh tâm, dài, rộng và yaw ở Top. Chỉnh chiều cao và đáy ở Side và Front.
- **Hướng.** Bật **Appearance → Cuboid orientation**. Đối chiếu vạch đỏ ngắn với đầu xe hoặc hướng đi của người. Dùng `python scripts/lab.py preview` để xem hình chiếu trên ảnh camera.

## 2. Đọc frame (Bước 2)

Độ cao mặt đường được ước lượng từ các điểm nền gần LiDAR.

| Frame và ảnh | Hướng trên ảnh từ trên xuống | Mặt đường | Vật đã đối chiếu xa nhất | Loại vật thấy | Vật hai bên hoặc phía sau | Cụm chưa rõ |
|---|---|---|---|---|---|---|
| `s01f13` · `t1-s01f13-3d.png`, `t1-s01f13-top.png` | Trước (`−y`) ở mép trên, `+x` về trái, `+y` về dưới | Gần phẳng, thấp hơn LiDAR khoảng 1,73 m | Car phía sau lệch phải, 36,02 m | Car, Bus, Pedestrian | Pedestrian phía sau phải, 9,58 m | Cụm rời bên phải khoảng 19 m |
| `s01f48` · `t1-s01f48-3d.png`, `t1-s01f48-top.png` | Trước (`−y`) ở mép trên, `+x` về trái, `+y` về dưới | Thấp hơn LiDAR khoảng 1,71 m, có độ nghiêng ngang nhẹ | Car phía sau trái, 22,44 m | Car ở cả trước và sau | Car phía sau trái | Cụm trước phải khoảng 15 m |

**Camera.** `cam06` của `s01f13` cho thấy sắc xanh sẫm của vùng thân xe Car gần, trong khi `cam03` của `s01f48` cho thấy một xe trước trái sáng hơn. `cam03` của `s01f13` cho thấy dáng xe buýt ở phía trước. Cũng trong `s01f13`, dáng người ở sau phải đọc được trên `cam08`.

**LiDAR.** Điểm nền cho phép ước lượng độ cao mặt đường ở hai frame. Tọa độ các cụm cho phép đo khoảng cách ngang tới vật, kể cả xe ở phía sau `s01f48`. LiDAR còn cho thấy độ thưa theo khoảng cách, chẳng hạn 2.630 điểm trong Car gần và 38 điểm trong Car xa ở `s01f13`.

| Frame đọc lướt | Phía trước trên ảnh từ trên xuống | Điều lạ ghi nhận |
|---|---|---|
| `s02f19` | Mép trên | Dải hụt điểm sau xe, xem mục 4 |
| `s02f53` | Mép trên | Cụm điểm nằm dưới nền, xem mục 4 |
| `s03f10` | Mép trên | |
| `s03f46` | Mép trên | |
| `s04f16` | Mép trên | |
| `s04f54` | Mép trên | |
| `s05f02` | Mép trên | |
| `s05f26` | Mép trên | |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ba ảnh ở `s01f13` ghi Perspective, Top, Side và Front của cuboid đang chọn. Class được đối chiếu bằng các góc camera trong bảng. Độ hở đáy dương nghĩa là đáy cao hơn nền ước lượng tại chỗ.

| Vật và ảnh | Frame, idx | Class | Hướng mũi tên yaw | Đáy chạm đường | Đối chiếu Top, Side, Front và camera | Kích thước theo bảng 4.3 |
|---|---|---|---|---|---|---|
| Xe con gần nhất · `t2-a-car.png` | `s01f13`, 1 | Car | 88,7° dọc thân, gần `+y`. Đầu xe còn cần xác nhận. | +0,11 m so với nền, gần chạm nền | Ảnh ghi đủ ba hướng. Đối chiếu class với `cam06`. | 4,562 × 1,820 × 1,472 m, trong khoảng sedan/hatchback. |
| Xe buýt · `t2-b-bus-truck.png` | `s01f13`, 2 | Bus | 284,0° về phía trước, phù hợp với chuyển động ước lượng qua hai frame liền kề. | +0,07 m so với nền, gần chạm nền | Ảnh ghi đủ ba hướng. Đối chiếu class với `cam03`. Hộp có `occluded`. | 9,869 × 2,715 × 2,576 m, ngắn và thấp hơn, rộng hơn khoảng xe buýt thường. |
| Người đi bộ · `t2-c-ped-cyclist.png` | `s01f13`, 5 | Pedestrian | 96,0°. Hướng mặt còn khó xác định. | +0,24 m so với nền, hơi cao | `cam08` cho thấy dáng người. Ảnh ghi đủ ba hướng. Đáy cần xem lại ở Side và Front. | 0,740 × 0,707 × 1,900 m, trong khoảng tham chiếu người lớn. |

Hai cụm chưa nhận ra ở mục 2 được để ngoài tập nhãn vì chưa đủ căn cứ xác định lớp (`UNCERTAIN_CLASS`). Hồ sơ hiện tại chưa ghi kết quả hiệu chuẩn chung về chênh lệch tâm, kích thước hoặc yaw giữa các bài.

## 4. Bốn hiện tượng chất lượng (Bước 4)

Cả bốn ảnh lấy từ task cá nhân. Vòng vàng đánh dấu vùng quan sát.

| Hiện tượng | Frame, ảnh và nguồn | Nguyên nhân giả thuyết | Quan sát trên point cloud | Hệ quả khi gán nhãn |
|---|---|---|---|---|
| Bóng che | `s02f19` · `t4-1-bong-che.png` · task cá nhân | Xe gần có thể chặn tia LiDAR. | Phía trước trái, xa hơn Car idx 4 ở 11,87 m có dải gần như trống điểm. Hai bên dải vẫn còn điểm. | Nếu thấy một phần vật phía sau, bật `occluded` và chỉ suy ra biên có căn cứ. |
| Thưa ở xa | `s01f13` · `t4-2-thua-o-xa.png` · task cá nhân | Khoảng cách và che khuất có thể cùng làm cụm thưa. | Car idx 7 phía sau xe ở 36,02 m có 38 điểm trong hộp. | Vẽ nếu thấy khoảng 15–20% thể tích hoặc có đủ căn cứ từ camera. Nếu chưa đủ, ghi `LOW_VISIBILITY`. |
| Phản xạ thấp | `s03f10` · `t4-3-phan-xa-thap.png` · task cá nhân | Vật liệu, góc tới hoặc che khuất có thể ảnh hưởng. | Car idx 3 ở 9,34 m có tín hiệu yếu hơn rõ so với Car idx 6 ở 11,25 m. Số điểm trong hộp lần lượt là 1.155 và 1.249. | Đối chiếu camera và biên LiDAR trước khi suy ra phần thiếu. |
| Điểm lạ | `s02f53` · `t4-4-diem-la.png` · task cá nhân | Có thể là phản hồi sai hoặc phản xạ nhiều đường. | Cụm rời phía sau lệch trái, cách khoảng 29 m và thấp hơn nền khoảng 12 m. | Bỏ qua cụm này khi gán nhãn. |

## 5. Số đo từ cuboid và tự kiểm (Bước 5)

Số liệu lấy từ `D13-2A202603132-bangA.csv` và `D13-2A202603132-tukiem.csv`. Đáy tính bằng `z − h/2`. Khoảng cách ngang là `dist_m = √(x² + y²)`, còn khoảng cách 3D tới tâm là `r = √(x² + y² + z²)`. Mốc mặt đường tham khảo là −1,7 m.

| Cuboid | z (m) | h (m) | Đáy (m) | dist_m (m) | r (m) | Yaw (°) | Nhận xét đáy |
|---|---:|---:|---:|---:|---:|---:|---|
| `s01f13` idx 1 · Car | −0,914 | 1,472 | −1,650 | 5,07 | 5,15 | 88,7 | Gần nền |
| `s01f13` idx 2 · Bus | −0,395 | 2,576 | −1,683 | 28,50 | 28,51 | 284,0 | Gần nền |
| `s01f13` idx 5 · Pedestrian | −0,531 | 1,900 | −1,481 | 9,58 | 9,60 | 96,0 | Hơi cao so với nền |

**Khoảng cách và kích thước.** Gần nhất là Car `s03f10` idx 4 với `dist_m` 3,89 m và `r` 4,01 m. Xa nhất là Car `s01f13` idx 7 với `dist_m` 36,02 m và `r` 36,03 m. Dài, rộng và cao của Car idx 1 và Pedestrian idx 5 trong mục 3 đều nằm trong các khoảng tham chiếu tương ứng.

**Yaw.** Trong `s02f19`, Car idx 2 ở phía trước cùng chiều có yaw 270,0° và Car idx 4 đi ngược chiều có yaw 90,0°. Chuyển động ước lượng qua hai frame liền kề phù hợp với hai hướng này.

| Bốn Car ở `s01f13` | dist_m (m) | points_in |
|---|---:|---:|
| idx 1 | 5,07 | 2.630 |
| idx 4 | 18,22 | 496 |
| idx 3 | 20,21 | 304 |
| idx 7 | 36,02 | 38 |

Trong nhóm này, hộp xa nhất cách khoảng 7,1 lần hộp gần nhất và còn khoảng 1,4% số điểm. Các mức 2.630, 496, 304 và 38 điểm cùng xu hướng giảm với các trung vị 1.968, 718, 214 và 72 điểm ở bốn khoảng tương ứng trong slide. Che khuất và góc nhìn cũng ảnh hưởng số điểm.

**Hai lần chỉnh.** Ở `s01f13` idx 4, yaw Car đổi từ 88,0° sang 268,0° sau khi đối chiếu chuyển động. Ở `s01f13` idx 5, chiều dài Pedestrian giảm từ 1,140 m xuống 0,740 m và tâm dịch khoảng 0,30 m theo `−y` để hộp ôm cụm điểm hơn. Phiếu cuối gắn cờ Car `s03f10` idx 7 vì đáy cao hơn nền ước lượng 0,51 m. Cần kiểm ở Side và Front để xác nhận vị trí đáy so với nền quanh vật.

**Đổi kiểm.** Đã đổi kiểm với người ngồi cạnh, MSSV `2A202603205`. Khi tự đối chiếu lại hàng `s01f13` idx 1 với Bảng A, cả hai số đều khớp. Khoảng cách ngang là **5,07 m** và đáy cuboid là **−1,650 m**.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

Từ mỗi cuboid, mô hình học cách nhận ra loại vật và dự đoán vị trí, kích thước, hướng trong không gian. Mét giữ kích thước ở cùng một thang đo dù khoảng cách thay đổi, còn quy tắc cho vật thưa điểm giúp hạn chế đưa các biên hộp thiếu căn cứ vào dữ liệu huấn luyện.

- **Công cụ.** Codex và YOLOX chạy trên máy.
- **Câu hỏi chính.** Cài CVAT, tạo task, đọc trục, gán nhãn, tự kiểm và soạn báo cáo.
- **Cách dùng kết quả.** Codex hỗ trợ chụp các góc nhìn CVAT, đối chiếu số liệu và soạn báo cáo. YOLOX và LiDAR được dùng để tạo nhãn tự động. Bản cuối có 60 hộp tạo tự động và 3 hộp đặt thủ công. Một trong hai hộp đặt thủ công ban đầu được chỉnh kích thước trong lần rà soát có hỗ trợ. Việc gán tự động và soạn báo cáo bằng AI vượt phạm vi cho phép trong nội quy lab.

**Điều còn chưa chắc.** Car xa ở `s01f13` idx 7 có chiều dài 3,120 m, dưới khoảng 3,8–5,0 m của sedan/hatchback. Với 38 điểm và trạng thái bị che, cần đối chiếu lại ảnh để quyết định có đủ căn cứ suy ra chiều dài hay nên bỏ hộp này.
