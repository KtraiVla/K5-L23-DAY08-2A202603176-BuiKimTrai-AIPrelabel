# Báo cáo Day 08 — 2A202603176 - Bùi Kim Trại

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26n, CPU, imgsz 640 (r01, r02) | 0.3 (mặc định; không chọn ngưỡng riêng) | Tôi chọn giữ YOLO26 cho các lớp COCO; các quan sát r01/r02 được ghi bên dưới. |
| three_wheeler | YOLOE-26s, prompt mặc định taxonomy (r03) | 0.3 (mặc định; không chọn ngưỡng riêng) | Tôi chọn giữ YOLOE để tìm xe ba bánh; các quan sát r03 được ghi bên dưới. |

- Lệnh tái lập cấu hình gộp theo hướng dẫn và hai nguồn lưu trong file merged (chưa có lịch sử terminal để xác nhận nguyên văn lệnh đã gõ):

```powershell
& 'D:\day08\.venv\Scripts\python.exe' scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --conf 0.3 --iou 0.6 --split 'batch,gold' --out work/prelabel/merged_review/pre.coco.json
```

Kết quả `submission/qc/threshold_sweep.md` trên 4 ảnh Gold với mục tiêu recall 90% không chọn được ngưỡng riêng cho lớp nào. Recall tối đa trong dải ngưỡng được xét: person khoảng 46% (35 vật Gold), motorbike 44% (16), car 88% (8), three_wheeler 86% (7). Bicycle 4, bus 1, truck 0: dưới tối thiểu 5 vật nên không chọn ngưỡng riêng. Vì vậy dùng mặc định 0.3 cho mọi lớp; đây là giá trị dự phòng theo quy trình, không phải ngưỡng đã chứng minh đạt recall mục tiêu. Các recall tối đa này không phải recall tại 0.3.
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Đã thử SAM 1 MobileSAM (`mobile_sam.pt`) ở r04; chọn SAM 2.1 B (`sam2.1_b.pt`) để thực hiện ba vật Part 5 theo lệnh mẫu repo. Kết quả và lỗi mask được ghi ở mục 5.
- Lỗi không sửa được chỉ bằng đổi ngưỡng: nhầm lớp phương tiện, gộp nhiều xe trong một box và box sai hình học. Thay conf chỉ lọc ứng viên, không đổi lớp hay tự tách/chỉnh box; tôi cần đối chiếu ảnh, sửa lớp/box và vẽ thêm vật đủ điều kiện khi review. Các ví dụ quan sát được ghi ở mục 2.
- Điểm vượt ngưỡng vẫn có thể sai: ở img_031, YOLO26 gán person cho vùng quần áo treo tại x≈457–619, y≈0–253 với score 0.513 (đối chiếu file y26_all trước cài bẫy). Box vẫn vượt conf 0.5 nhưng theo ảnh không có người thật tại vùng đó. Điểm máy không thay việc kiểm bằng mắt; đây cũng là giới hạn đáp án bẫy trình bày ở mục 3.
- Giới hạn: chỉ có 4 ảnh Gold, dùng cùng Gold để xem xét ngưỡng và đánh giá nên số đo có giới hạn in-sample/mẫu nhỏ. Không lớp nào đạt điều kiện chọn ngưỡng riêng; không suy rộng kết quả sang mọi ảnh hoặc coi ngưỡng mặc định là tối ưu.

### Nhật ký thực nghiệm Part 2

Các thông số, số box và thời gian dưới đây lấy từ `work/runs.csv`. Các nhận xét định tính được ghi từ quan sát trực tiếp của tôi trên `img_002.jpg`, `img_004.jpg` và `img_013.jpg`.

| Run ID | Model | Dữ liệu chạy | Conf detector | Số ảnh | Số box | Số mask | Giây/ảnh |
|---|---|---|---:|---:|---:|---:|---:|
| r01 | YOLO26n | pilot | 0.25 | 8 | 120 | 0 | 0.81 |
| r02 | YOLO26n | pilot | 0.50 | 8 | 54 | 0 | 0.51 |
| r03 | YOLOE-26s, chỉ giữ three_wheeler | pilot | 0.25 | 8 | 5 | 0 | 0.19 |
| r04 | YOLO26n + SAM 1 MobileSAM | img_004.jpg, img_013.jpg | 0.30 | 2 | 10 | 10 | 2.83 |

YOLO26 và YOLOE chạy trên CPU, `imgsz=640`. Thời gian r04 là thời gian của lượt detector kết hợp segmenter, không phải phép đo riêng tốc độ SAM.

### Quan sát ảnh của tôi

1. **Ảnh gốc img_002.jpg:** tôi thấy một tuk-tuk ở giữa ảnh và nhiều người; chưa gặp vật khó phân biệt trong lần quan sát này.
2. **r01 — YOLO26n, conf 0.25:** tuk-tuk ở giữa ảnh bị bỏ sót, cùng một ô tô phía phải. Một số box person chưa bao hết người, còn thiếu phần chân. Những tên lớp đã được gán mà tôi kiểm tra là phù hợp.
3. **r02 — YOLO26n, conf 0.50:** so với r01, người ngồi trên tuk-tuk không còn được gán nhãn; một người đi xe máy và chiếc xe máy cũng bị bỏ sót. Tôi chưa thấy tên lớp sai trong các box còn lại ở ảnh này.
4. **r03 — YOLOE:** trong img_002 có hai box three_wheeler. Một box tìm đúng tuk-tuk ở giữa ảnh; box còn lại, theo quan sát của tôi, gán nhầm xe bán hàng thành xe ba bánh.
5. **r04 — YOLO26n + MobileSAM, img_013.jpg:** các mask đã quan sát nằm trên đúng vật, tên lớp được quan sát là đúng, nhưng một mask person thiếu phần đầu. Xe ba bánh chưa có nhãn/mask.
6. **r04, img_004.jpg:** bỏ sót nhiều đối tượng và không có nhãn xe ba bánh. Có một trường hợp bánh xe ba bánh bị gán nhãn person.

Trong r04, `--classes "person,motorbike,car"` chỉ giữ ba lớp từ detector YOLO26. Vì vậy, việc thiếu đầu vào cho xe ba bánh và gán nhầm bánh xe thành person cần được phân biệt với lỗi biên mask của SAM; mask thiếu đầu là nhận xét về phần phân đoạn. SAM tạo mask từ các box được detector cung cấp.

Các nhận xét thiếu chân/đầu cần đối chiếu với phần cơ thể thực sự nhìn thấy theo guideline v1.0; không yêu cầu box hoặc mask bao phần bị che. Quan sát r01–r03 hiện chủ yếu trên img_002, còn r04 đã xem img_004 và img_013. Các ca này chưa đủ để suy ra precision/recall cho toàn bộ tập pilot.

### Hướng sử dụng đã chọn

Tôi giữ YOLO26 cho các lớp COCO và YOLOE để tìm xe ba bánh. Đã chạy threshold_sweep trên Gold ở Part 3; không lớp nào đủ điều kiện chọn ngưỡng riêng nên giữ mặc định 0.3. Tôi đã thử SAM 1 MobileSAM ở Part 2 và dùng SAM 2.1 B cho ba vật Part 5; xem mục 5.

## 2. AI error profile (từ `qc/review_stats.md`)

Số liệu sau vòng sửa bổ sung lấy từ `submission/qc/review_stats.json` và `.md`: 24 ảnh batch; 234 box máy và 328 đối tượng sau duyệt được tính trong bảng. Script đã loại 21 đối tượng nằm quanh các bẫy khỏi phép đo error profile; các mẫu số này không phải tổng box nguyên trạng trong file import hoặc tổng toàn bộ nhãn của task.

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---|---|
| accepted | 171 | 171/234 = 73.1% | 234 box máy |
| edited | 36 | 36/234 = 15.4% | 234 box máy |
| deleted | 27 | 27/234 = 11.5% | 234 box máy |
| added | 121 | 121/328 = 36.9% | 328 đối tượng sau duyệt |

Trong deleted + added, script nhận diện 2 trường hợp đổi lớp. Số đối tượng máy sót theo phép đối sánh này xấp xỉ 121 − 2 = 119; đây không phải kiểm chứng độc lập rằng nhãn sau duyệt không còn lỗi. Accepted được xác định bằng cùng lớp và IoU ≥ 0.9, không chứng minh bằng mắt rằng mọi box được giữ đều đúng.

Person và motorbike là hai lớp cần bổ sung nhiều nhất về số tuyệt đối: person added 62, motorbike added 38; mỗi lớp có 17 box edited. Three_wheeler có 9 box deleted, 13 accepted và 1 added, cho thấy vẫn có ứng viên thừa cần kiểm. Không dùng các số tuyệt đối khác mẫu số để kết luận một lớp có tỷ lệ lỗi cao nhất.

Trên 4 ảnh Gold có 71 đối tượng: TP = 28, FP = 7, FN = 43; precision của máy = 28/(28+7) = 80.0% (KTC 95% khoảng 64–90%), recall = 28/71 = 39.4% (KTC 95% khoảng 29–51%). Đây là số đo pre-label, không phải precision/recall của nhãn batch sau review. Gold được dùng trong quy trình xem xét ngưỡng và đánh giá, chỉ có 4 ảnh nên kết quả có giới hạn mẫu nhỏ và dùng chung dữ liệu. Không tính residual/new error vì Gold không nằm trong task review.

### Quan sát khi tự review 24 ảnh

Frame dưới đây là số frame CVAT bắt đầu từ 0; tên ảnh đã đối chiếu với dữ liệu task 2. Đây là ghi chú quan sát của tôi, không phải bảng đếm lỗi theo ID. Nhãn đưa vào task đã có bẫy nhân tạo; không quy mọi lỗi nhìn thấy cho model gốc. Việc đối chiếu loại bẫy được thực hiện sau khi export vòng đầu.

| Frame CVAT | Ảnh | Quan sát và cách diễn giải theo guideline |
|---:|---|---|
| 4 | img_008.jpg | Theo quan sát của tôi, xe bò/xe bán hàng bị nhận thành three_wheeler. Cần dựa trên cấu tạo phương tiện thực tế, không mặc định mọi xe bán hàng đều là xe ba bánh. |
| 6 | img_010.jpg | Xích lô được gán three_wheeler. Đây là đúng theo taxonomy: three_wheeler gồm tuk-tuk, xích lô và ba gác, không phải tên riêng của tuk-tuk. |
| 7 | img_013.jpg | Một box person dư vùng, một người bị bỏ sót; tìm đúng một tuk-tuk nhưng theo quan sát của tôi có một xe buýt bị gán three_wheeler. |
| 8 | img_014.jpg | Vải/vùng không phải ô tô bị gán car. Ảnh này có bẫy extra gán car; cần phân biệt đối tượng quan sát với bẫy theo tọa độ trước khi gọi đây là lỗi tự nhiên của model. |
| 9 | img_017.jpg | Tôi ghi nhận vùng có ba tuk-tuk bị gọi truck và một tuk-tuk thiếu nhãn. File import có một box truck lớn tại ảnh này, không phải ba box truck độc lập; cần tách nhãn theo từng xe thật. |
| 10 | img_018.jpg | Theo ghi chú của tôi, hai người bị gán motorbike; nhiều người và xe máy chưa có nhãn. |
| 12 | img_022.jpg | Khoảng trống bị gán car. Ảnh có bẫy extra car; ghi đây là lỗi của pre-label đã cài bẫy nếu đối tượng trùng vùng bẫy. |
| 13 | img_023.jpg | Hai tuk-tuk bị bao chung một box và một tuk-tuk thiếu nhãn. Ảnh có bẫy missing three_wheeler; không quy toàn bộ việc thiếu nhãn ở ảnh này cho model. |
| 14 | img_024.jpg | Một vùng chứa ô tô bị gán motorbike. Ảnh có bẫy extra motorbike; cần đối chiếu vùng bẫy trước khi kết luận lỗi tự nhiên của model. |
| 15 | img_025.jpg | Bỏ sót một người rõ và lớn; ảnh có bẫy missing person nên cần phân biệt nguyên nhân. Tôi thấy phương tiện giống xe đạp điện được gán motorbike; tôi xác nhận chưa gán nhãn phương tiện này trong vòng review; sau calibration đã bổ sung motorbike, xem CAL-02 ở mục 6. Ở thời điểm review đầu đây là câu hỏi chưa chốt; sau đó được xử lý theo CAL-02. Việc để trống nhãn lúc đầu không được coi là quyết định calibration. Phần sửa sau calibration được lưu riêng trong qc_fixed.json và áp dụng vào release; giữ số đo review trước calibration làm bằng chứng. |
| 16 | img_026.jpg | Nhiều xe máy, khó kiểm kê. Ghi chú này mô tả mật độ cảnh, chưa đủ để kết luận một lỗi nhãn cụ thể; áp dụng quy tắc phần nhìn thấy, kích thước 20 px và cụm ignore_region. |
| 17 | img_027.jpg | Theo quan sát của tôi, xích lô bị gán motorbike; lớp phù hợp cho xích lô theo taxonomy là three_wheeler. |
| 20 | img_031.jpg | Một xe đạp có hai nhãn; vùng quần áo trưng bày bị gán bicycle/person. Tôi loại box không có người thật; đây cũng là ca đáp án bẫy cần giải thích ở mục 3. |
| 22 | img_034.jpg | Nhãn bicycle nằm trên vùng người/xe và một người bị bao bởi hai box. Ảnh có bẫy wrong_class person → bicycle; phân biệt phần lỗi nhân tạo với lỗi tự nhiên. |
| 23 | img_035.jpg | Xích lô được gán three_wheeler: đúng theo taxonomy, không ghi thành lỗi nhận diện xích lô thành tuk-tuk. |

## 3. Thiên kiến tự động hóa và ảnh bẫy

Vòng review đầu đạt **8/12 = 66.7%**, dưới ngưỡng 80%: missing 3/3, extra 3/3, wrong_class 1/3, loose 1/3. Tôi xem kết quả, sửa bổ sung và export lại. Sau vòng sửa, script chấm **11/12 = 91.7%** (KTC 95% khoảng 65–99%): missing 3/3, extra 3/3, wrong_class 2/3, loose 3/3. Vòng sau đã biết danh sách bẫy lọt và có Codex hỗ trợ giải thích nên không phải phép đo mù độc lập.

Ca còn bị script chấm lọt là `img_031.jpg`, wrong_class. Đáp án lưu box gốc của máy có lớp person tại x≈457–619, y≈0–253 trên ảnh gốc 1024×682, sau đó đổi giả thành bicycle. Vùng này là quần áo treo/trưng bày, không thấy người thật cần gán nhãn. Theo guideline, tôi không khôi phục person chỉ để khớp đáp án. Script yêu cầu có person với IoU ≥ 0.5 tại vùng box đó nên vẫn báo lọt. Tôi giữ kết quả script 11/12 và ghi riêng giới hạn của bẫy tạo từ box máy sai; không tự sửa key hay cộng điểm thành 12/12.

Accepted vòng đầu 177/234 = 75.6% nhưng chỉ bắt 8/12 bẫy: tỷ lệ giữ box cao chưa chứng minh lượt duyệt đáng tin, nhất là về sai lớp và hình học box. Sau sửa, accepted là 171/234 = 73.1%, edited tăng từ 30 lên 36. Những thay đổi này phù hợp với việc cần kiểm lại box đã giữ, nhưng không đủ để suy ra mọi lỗi còn lại đã được loại bỏ. Tôi chưa có dữ liệu đủ để khẳng định cụ thể đã bỏ bước nào của checklist ở vòng đầu.

## 4. Máy có đáng dùng không?

Thời gian gán tay Gold là 30/4 = **7.5 phút/ảnh**. Tôi cung cấp thời gian review vòng đầu khoảng **45 phút/24 ảnh**, tương đương khoảng **1.88 phút/ảnh**; đây là ước tính, không trình bày như số đo bấm giờ chính xác. Vòng sửa bổ sung mất 10 phút theo thông tin tôi cung cấp. Tổng hai vòng là khoảng 45 + 10 = 55 phút/24 ảnh, tương đương khoảng 2.29 phút/ảnh. Tổng thời gian này có thành phần ước tính, không trình bày như số đo bấm giờ chính xác; việc sửa tiếp ở calibration/QC nếu phát sinh chưa nằm trong 55 phút. Gold và batch khác ảnh, đồng thời tôi đã quen cảnh sau khi làm Gold; so sánh thời gian có các giới hạn này.

Đánh giá sơ bộ: pre-label tạo điểm bắt đầu cho việc sửa nhãn, nhưng số added cao và recall máy trên Gold thấp cho thấy cần chủ động tìm vật thiếu. Giữ recipe YOLO26 cho các lớp COCO và YOLOE cho three_wheeler như lựa chọn Part 2; tính hữu ích cuối cùng còn phải xem cả thời gian sửa tiếp nếu có và QC, không suy ra từ accepted hoặc tốc độ vòng đầu đơn lẻ.

- **Ba quy tắc giảm thiên kiến của tôi** (đã xác nhận chọn cho lần sau):
  1. Tôi chọn cho lần sau: ẩn nhãn máy và tự quét/kiểm kê trước ở mọi ảnh; ảnh đông chia bốn vùng để hạn chế bỏ sót.
  2. Tôi chọn cho lần sau: kiểm tên lớp, bốn cạnh và box trùng trên từng vật, kể cả box có điểm cao; không dùng conf để bỏ qua kiểm tra.
  3. Tôi chọn cho lần sau: nghỉ sau khoảng 10 ảnh, ghi các ca can_review và quay lại chốt trước export.

## 5. Mask SAM cho 3 vật (Part 5)

Tôi dùng **SAM 2.1 bản B (`sam2.1_b.pt`)**, chạy CPU bằng `scripts/sam2_points.py`, theo lệnh mẫu Part 5 của repo. Ở img_013, tôi chạy MobileSAM trước, sau đó chạy lại SAM 2.1 B với cùng điểm và cùng tên file. Vì vậy ảnh nộp hiện tại của img_013 là kết quả SAM 2.1 B, không phải MobileSAM. Cả ba ảnh dưới đây đều là kết quả SAM 2.1 B theo log chạy tôi cung cấp.

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_003.jpg | SAM 2.1 B | 3 dương + 3 âm = 6 | Lấy thừa mặt đứa bé ngồi trên xe và hàng hóa trên xe | Phần còn lại tôi thấy mượt, không thấy bị bỏ sót | three-wheeler_img003.jpg |
| img_013.jpg | SAM 2.1 B | 2 dương + 2 âm = 4 | Tôi thấy ít lem hơn img_003; chưa mô tả riêng phần người/hàng lấy thừa | Một ít viền bánh trước chưa được mask hết | tuktuk_img013.jpg |
| img_017.jpg | SAM 2.1 B | 2 dương + 2 âm = 4 | Tôi thấy mask chuẩn hơn img_003 và img_013; chưa nêu phần lấy thừa riêng | Chưa nhận xét lỗi riêng còn lại; đánh giá này là định tính | tuktuk_img017.jpg |

**Quan sát của tôi, bổ sung riêng từng ảnh:**

- **img_003:** mask lem vào mặt đứa bé ngồi trên xe và hàng hóa trên xe. Phần còn lại tôi thấy mask mượt, không bị bỏ sót rõ rệt.
- **img_013:** tôi thấy mask ổn, ít lem hơn img_003, nhưng một ít viền bánh trước chưa được mask hết.
- **img_017:** tôi thấy mask chuẩn hơn hai ảnh trước; tôi chưa nêu lỗi cụ thể còn lại. Đây là so sánh bằng mắt, không phải số đo chứng minh mask chính xác 100%.

Ba nhận xét trên bổ sung cho mô tả chung lúc đầu; không tự gán mọi loại lỗi người/hàng hóa/nền/xe bên cạnh vào cả ba ảnh. Sau QC tôi xác định xe bán hàng ở img_003 ngoài phạm vi lớp detection và bỏ hai box; kết quả SAM vẫn được giữ như bằng chứng thử tương tác với vật đã chọn. Tên three-wheeler_img003.jpg là tên lệnh đã chạy, không chứng minh lớp detection của vật.

Các lệnh thực tế (Python trong môi trường D:\day08\.venv):

```powershell
& 'D:\day08\.venv\Scripts\python.exe' scripts/sam2_points.py --image img_013.jpg --pos "304,487" --pos "450,115" --neg "709,324" --neg "802,492" --sam sam2.1_b.pt --device cpu --out submission/masks --name tuktuk_img013
& 'D:\day08\.venv\Scripts\python.exe' scripts/sam2_points.py --image img_003.jpg --pos "431,139" --pos "595,385" --pos "389,547" --neg "328,324" --neg "417,277" --neg "314,359" --sam sam2.1_b.pt --device cpu --out submission/masks --name three-wheeler_img003
& 'D:\day08\.venv\Scripts\python.exe' scripts/sam2_points.py --image img_017.jpg --pos "329,338" --pos "339,485" --neg "492,482" --neg "202,404" --sam sam2.1_b.pt --device cpu --out submission/masks --name tuktuk_img017
```

Kết quả console của các lượt SAM 2.1 B:

| Ảnh | Diện tích mask (pixel) | Tỷ lệ diện tích ảnh | Số polygon |
|---|---:|---:|---:|
| img_013.jpg | 278890 | 39.9% | 1 |
| img_003.jpg | 112772 | 16.1% | 2 |
| img_017.jpg | 45174 | 6.5% | 1 |

MobileSAM ở img_013 trước đó trả 206362 pixel (29.5% ảnh), 1 polygon. Diện tích lớn hơn ở SAM 2.1 không tự chứng minh chính xác hơn; các tỷ lệ này là diện tích ảnh được mask, không phải điểm chất lượng. Hai polygon ở img_003 cũng không tự có nghĩa là đã tách được hai chiếc xe. Các số click trên là điểm được đưa vào lệnh cuối mỗi ảnh, không phải nhật ký đầy đủ của mọi lần thử.

Đối chiếu guideline: mục 4.2 yêu cầu bám biên phần nhìn thấy, không lấy bóng đổ và tách vật chạm nhau; mục 5 phân biệt xe với người ngồi trên xe. Viền bánh trước thiếu nhẹ là lỗi biên; người hoặc xe bên cạnh dính vào mask là lỗi tách đối tượng. Phạm vi người/hàng hóa trong mask được làm rõ ở CAL-01; không mở rộng kết luận sang mọi ca kính/người nhìn xuyên kính chưa được kiểm. Có thể thử thêm điểm âm ở vùng thừa, nhưng tôi chưa cung cấp kết quả một lượt sửa mask sau các lệnh trên nên không báo cáo rằng đã khắc phục.

Phần kết quả này dùng script offline. Không đưa kết quả thực nghiệm Interactor SAM trong CVAT vào báo cáo.

### Đối chiếu các mốc sản phẩm

| Mốc | File làm bằng chứng | Ảnh | Đối tượng thô | Ý nghĩa |
|---|---|---:|---:|---|
| Gold | labels/gold_manual.json | 4 | 71 | Bộ gán tay; giữ nguyên từ Part 1 |
| Review trước calibration | labels/batch_reviewed.json | 24 | 338 | Nguồn để tính review_stats; mẫu số 234/328 sau loại vùng bẫy không phải 338 |
| Trước QC vòng 1 | qc/round1_before_rework/final_det.coco.json | 24 | 338 | Đã thêm motorbike img_025 và xóa car dưới 20 px img_027; số lượng bằng bản review nhưng nội dung khác |
| Sau sửa QC vòng 1 | labels/qc_fixed.json và release/final_det.coco.json | 24 | 359 | Bản cuối dùng cho QC vòng 2 và release |

QC vòng 1 lấy 6 ảnh/79 đối tượng từ bản 338; QC vòng 2 lấy cùng 6 tên ảnh nhưng có 100 đối tượng từ bản 359. Không thay mẫu số lịch sử bằng số đối tượng của bản sau.

## 6. Calibration (Part 6)

- Đã ghi 4 quyết định trong `qc/calibration_decisions.csv`: CAL-01 (`loi_may`) làm rõ phạm vi mask xe dựa trên bộ ba ảnh SAM; CAL-02 (`cau_hoi_mo`) xử lý xe dạng xe máy ở mép phải cạnh ghế đỏ của img_025; CAL-03 (`bay_lot`) ghi giới hạn đáp án bẫy ở img_031; CAL-04 (`loi_may`) ghi quyết định học viên loại xe bán hàng giữa img_003 khỏi phạm vi lớp sau QC.
- CAL-02 đã được áp dụng: tôi thêm một box motorbike ở img_025 và export `labels/qc_fixed.json`. Bản sửa tại thời điểm trước QC vòng 1 gồm 24 ảnh, 338 đối tượng (bộ cuối sau rework là 24 ảnh, 359 đối tượng, xem mục 7). So với batch_reviewed (24 ảnh, 338 đối tượng), thêm một motorbike ở img_025 và xóa một car cao 17.12 px ở img_027; hình học các box khác giữ nguyên. Tổng số lượng bằng nhau nhưng nội dung hai bộ nhãn khác nhau. build_release áp dụng bản sửa; số đo review ở mục 2–3 vẫn là kết quả trước calibration, không phải số đo QC của release.
- `guidelines/guideline_v1.1_calibrated.md` chỉ chứa phần bổ sung/làm rõ. CAL-01 tách người và hàng hóa rời khỏi mask xe, giữ phần gắn cố định. CAL-02 làm rõ xe dạng mô tô/scooter hai bánh theo cấu tạo, không suy đoán xăng/điện; chưa khái quát mọi xe đạp trợ lực. CAL-03 áp dụng quy tắc v1.0 cho vùng không có người thật và ghi giới hạn phép đo, không tạo quy tắc lớp mới hoặc thay điểm bẫy.
- Nguyên nhân gốc gồm lỗi tách biên/đối tượng của máy, câu hỏi guideline cần làm rõ và đáp án bẫy xuất phát từ nhãn máy sai. Không sửa taxonomy.rules. Chưa báo cáo sửa lại mask sau ba lượt SAM đã ghi ở mục 5.
- Các câu hỏi v1.0 chưa chốt bổ sung được nêu trong guideline; nếu còn ảnh hưởng nhãn thật/QC thì cần giải quyết hoặc chọn Hold, không mặc định đã đạt chất lượng.

## 7. QC và quyết định (Part 6)

### QC vòng 1 — trước sửa lỗi

- Mẫu: 6/24 ảnh batch, gồm 1 ca khó can_review và 5 ảnh ngẫu nhiên, seed 42; 79 đối tượng theo `qc/round1_before_rework/qc_sample.csv`. Ảnh: img_003, img_006, img_013, img_014, img_025, img_033. Người kiểm là tôi, cũng là người review nhãn, nên chưa có kiểm độc lập.
- Kết quả theo phiếu quan sát: **7/79 = 8.9% đối tượng lỗi**, **5/6 = 83.3% ảnh có lỗi**. Giữ mẫu số của script; không tự thêm đối tượng thiếu vào mẫu số. Các lỗi người ở xa được ghi theo xác nhận của tôi, cần đối chiếu điều kiện cao ≥20 px và phần nhìn thấy khi sửa; nếu không đủ điều kiện phải điều chỉnh phiếu kèm lý do thay vì vẽ để khớp số.

| Ảnh | Frame CVAT | Lỗi ghi nhận |
|---|---:|---|
| img_003 | 0 | 1 box thừa: #5 và #6 trên ảnh QC cùng bao một xe; ghi #6 là box trùng. Ở vòng 1 Codex diễn giải vật này là xe ba bánh bán hàng; học viên sau đó xác nhận vật ngoài phạm vi lớp và xóa cả hai box. Xem CAL-04 và ghi chú vòng 2; không dùng diễn giải cũ làm kết luận hiện tại. |
| img_006 | 3 | 3 người thiếu theo xác nhận của tôi: một người đứng ở xa và hai người ngồi ghế ở xa. |
| img_013 | 7 | 1 người áo hồng ngồi tại bàn chưa có box. |
| img_014 | 8 | 1 người áo trắng đi xe máy ở xa chưa có box. |
| img_025 | 15 | 1 người áo trắng đi bộ ở xa chưa có box. Xe cạnh ghế đỏ giữ motorbike theo CAL-02, không tính lỗi sai lớp. |
| img_033 | 21 | Tôi kiểm và không thấy lỗi. |

- Top lỗi: bo_sot **6/7 lỗi** và thua **1/7 lỗi**. Script gắn cờ bỏ sót lặp lại là lỗi hệ thống; cần siết bước quét ảnh trước, kiểm người ở xa/bị che thay vì chỉ tin box đã có. Không thêm loại lỗi chưa quan sát như sai_thuoc_tinh hoặc sai_mask vào phiếu detection này.
- `qc/round1_before_rework/qc_report_auto.md` cho cận trên 95% Wilson của tỷ lệ ảnh lỗi là khoảng **97%** với 5/6 ảnh lỗi. Mẫu nhỏ không chứng minh được tỷ lệ lỗi toàn bộ dữ liệu dưới 2%; cũng không coi 83.3% là tỷ lệ chính xác của toàn bộ 24 ảnh.
- Script kiểm cấu trúc, tên lớp, tọa độ và đếm lỗi dựa trên phiếu; không tự nhìn ra lớp sai, box chưa sát, bỏ sót chung của người/máy hay việc tôi có theo checklist thật. Codex đã đối chiếu ảnh QC có ID để hỗ trợ làm rõ ghi chú; đây không phải đánh giá hoàn toàn độc lập với hỗ trợ AI.

**Trạng thái theo script sau vòng 1: Rework**, ngày **2026-11-19**, theo kết quả script: ảnh lỗi 5/6 vượt 2% và bỏ sót 6/7 lỗi. Đây là kết quả trước khi sửa; kết quả sau sửa ở vòng 2 bên dưới, không phải trạng thái cuối của bài.

Sau vòng 1, tôi sửa nhãn trong CVAT, export lại qc_fixed.json, build release và tái kiểm. Tôi xác nhận xe bán hàng giữa img_003 ngoài phạm vi lớp nên xóa cả hai box; các chỉnh sửa khác được thể hiện ở bản nhãn vòng 2. Nếu phát hiện người quá nhỏ hoặc chưa đủ phần nhìn thấy thì ghi rõ điều chỉnh đánh giá. ID trên ảnh QC là ID của bản release, có thể khác ID trong CVAT; đối chiếu vị trí/loại lỗi khi sửa. Bằng chứng vòng đầu được lưu tại `submission/qc/round1_before_rework/`; không xóa hoặc thay điểm vòng đầu sau khi sửa. Vòng sau cùng seed có thể lặp ảnh nên không gọi là mẫu kiểm độc lập.

### QC vòng 2 — sau sửa

Học viên export lại qc_fixed.json gồm 24 ảnh, **359 đối tượng**. So với bộ ứng viên 338 đối tượng đã kiểm ở vòng 1, chênh lệch ròng +21; đối sánh lớp và tọa độ cho 27 bản ghi mới/đổi và 6 bản ghi xóa/đổi, trong đó chỉnh hình học có thể xuất hiện ở cả hai phía. Không hiểu các số đối sánh này là 27 vật được thêm độc lập.

CAL-04: học viên xác nhận xe bán hàng giữa img_003 ngoài phạm vi lớp, nên xóa cả hai box cũ, không chỉ xóa một box trùng. Nhận xét trước của Codex rằng vật này chắc thuộc three_wheeler là quá chắc và được đính chính. Điểm QC vòng 1 7/79, 5/6 là ảnh chụp kết quả theo diễn giải lúc đó; không cập nhật ngược điểm đã lưu. Quyết định mới được ghi riêng, không dùng mô tả cũ để khẳng định xe đã được xác minh là ba bánh.

Nguồn vòng 2: `qc/qc_sample.csv`, `qc/qc_review.csv` và `qc/qc_report_auto.md` (đồng thời có bản lưu trong `qc/round2_after_rework/`). Đã build lại release và tạo phiếu vòng 2, seed 42: **6/24 ảnh, 100 đối tượng**, gồm 1 ca khó và 5 ngẫu nhiên. Học viên xác nhận không thấy lỗi trên cả sáu ảnh QC mới; đã ghi sáu dòng khong và chạy qc_report.py. Kết quả: **0/100 = 0.0% đối tượng lỗi, 0/6 = 0.0% ảnh lỗi**. Script đề xuất RELEASE theo tiêu chí lab. Đây là kết quả theo phiếu người kiểm, không phải máy tự chứng minh mọi nhãn đều đúng.

| Ảnh vòng 2 | Đối tượng | Cách lấy |
|---|---:|---|
| img_003.jpg | 20 | ngẫu nhiên |
| img_006.jpg | 22 | ngẫu nhiên |
| img_013.jpg | 10 | ngẫu nhiên |
| img_014.jpg | 16 | ngẫu nhiên |
| img_025.jpg | 14 | ca khó |
| img_033.jpg | 18 | ngẫu nhiên |

Đây là tái kiểm sau khi đã biết lỗi và có thể lặp cùng ảnh, không phải kiểm mù độc lập. Đã dùng bộ ảnh QC mới với ID mới và báo cáo tự động sinh từ phiếu vòng 2. Không dùng báo cáo vòng 1 làm kết quả vòng 2. Thời gian 55 phút ở mục 4 chỉ là review trước calibration/QC, chưa cộng thời gian sửa bổ sung sau QC chưa được cung cấp.

### Kết quả theo script và quyết định của tôi

**Script đề xuất Release theo tiêu chí kiểm mẫu của lab**, ngày **2026-11-19**: vòng 2 không ghi nhận lỗi (0/6 ảnh, 0/100 đối tượng), không còn lỗi hệ thống được ghi trong mẫu này; điểm bẫy sau sửa 11/12 = 91.7% vượt 80%. Giữ lịch sử vòng đầu Rework thay vì xóa kết quả cũ.

**Quyết định của tôi: Release cho bộ nhãn detection cuối.** Tôi đã sửa các lỗi ở vòng đầu, kiểm lại 6 ảnh và không thấy lỗi; phiếu vòng 2 ghi 0/6 ảnh lỗi và 0/100 đối tượng lỗi. Tôi tự review rồi tự QC, nên đây là kết quả tự kiểm của tôi, chưa có người kiểm độc lập. Tôi đồng ý với kết quả kiểm này sau khi thực hiện các bước sửa và tái kiểm nêu trên.


Giới hạn quan trọng: cùng người review và QC, tái kiểm cùng mẫu sau khi đã biết lỗi, có Codex hỗ trợ đối chiếu và biên tập. Cận trên 95% Wilson khi 0/6 ảnh lỗi vẫn khoảng **39%**; không chứng minh tỷ lệ lỗi toàn bộ dữ liệu ≤2%. Điểm bẫy 11/12 cũng là sau khi biết danh sách bẫy lọt. Nếu sau này phát hiện ca chưa rõ ảnh hưởng nhãn thật hoặc lỗi mới, cần Hold/Rework thay vì giữ Release chỉ vì script từng báo đạt. Bộ release cuối là detection 24 ảnh/359 đối tượng; ba ảnh SAM vẫn chứa các lỗi mask đã ghi ở mục 5, không gọi chúng là nhãn phân đoạn đã được QC sạch.

## 8. Ghi chú cá nhân

- **Việc tôi tự làm:** tự gán tay Gold trước khi xem pre-label, chạy các lệnh model, chọn điểm click SAM, xem và sửa nhãn trong CVAT, cung cấp nhận xét từng ảnh, export kết quả và xác nhận QC vòng 2 không thấy lỗi. Tôi xác nhận chọn recipe YOLO26/YOLOE, ba quy tắc ở mục 4 và các quyết định calibration được ghi theo quá trình trong mục 6–7.
- **Hỗ trợ của Codex:** cài/kiểm tra môi trường, thiết lập CVAT riêng, giải thích lệnh và sửa lỗi dấu nháy PowerShell; bổ sung bộ nhãn vào task và import file pre-label. Codex còn đọc script, đối chiếu các file và số đo, xem một số ảnh sau export, soạn/biên tập bản nháp báo cáo và guideline từ trao đổi với tôi. Phần hỗ trợ có cả gợi ý diễn giải số đo và bản nháp kết luận theo script, không chỉ sửa chính tả. Tôi đã xác nhận kết luận cuối dựa trên việc tự sửa lỗi và tái kiểm; Codex biên tập câu chữ từ xác nhận đó. Tôi vẫn cần tự đọc lại và giải thích được nội dung/quy tắc theo RULES trước khi nộp.
- **Gold:** theo xác nhận của tôi, đã gán xong 4 ảnh và export trước khi xem kết quả AI. Codex chỉ rà Gold sau export/commit Part 1 (446b2f8). Audit xác nhận JSON Gold hiện không thay đổi so với commit đó; kiểm hash không tự chứng minh độc lập toàn bộ quá trình gán tay. Các thử nghiệm bus.jpg phục vụ cài đặt trước đó không phải ảnh Gold của lab.
- **Nguồn kiểm chứng:** work/runs.csv và run.json cho r01–r06; console tôi cung cấp cho SAM click; các JSON/CSV/Markdown do script sinh cho threshold, review, QC và release. Ba lượt SAM 2.1 không có runtime được đo nên để trống, không tự bổ sung số. Lượt MobileSAM của img_013 đã bị lượt SAM 2.1 ghi đè cùng tên ảnh và được mô tả rõ ở mục 5.
- **Review và bẫy:** tôi export xong vòng đầu rồi mới xem kết quả bẫy. Vòng đầu 8/12, sau xem danh sách lọt và sửa là 11/12; không coi lượt sau là đo mù độc lập. Các file pre_traps và key không sửa tay; audit tái tạo lại khớp theo MSHV và seed.
- **Hai vòng QC:** Codex ghi phiếu theo nhận xét tôi cung cấp, đối chiếu ID và chạy script; không tự đặt câu trả lời không lỗi trước khi tôi xác nhận. Vòng đầu ghi Rework; tôi sửa trong CVAT và export lại; vòng 2 tôi xác nhận không thấy lỗi ở sáu ảnh mới. File cũ được lưu riêng, không dùng phiếu mới để xóa lịch sử lỗi. Khi xem lại img_003, tôi xác định xe ngoài phạm vi và xóa cả hai box; giải thích trước của Codex về lớp xe đã được đính chính ở CAL-04.
- **Thời gian:** Gold 4 ảnh/30 phút giữ nguyên. Review đầu khoảng 45 phút là ước tính tôi cung cấp; thêm 10 phút sửa bẫy thành khoảng 55 phút, chưa bao gồm calibration, kiểm QC và sửa sau QC. Chưa có số đo đồng hồ đầy đủ cho lượt review đầu theo yêu cầu RULES; không trình bày số ước tính như số đo chính xác hoặc tự bịa thời gian còn thiếu. Cần làm rõ cách xử lý thiếu số đo này với giảng viên/coach.
- **Máy và phần mềm:** Windows 11 Pro; AMD Ryzen 7 5800H; RAM 16 GB; Python 3.13.15 trong D:\day08\.venv; PyTorch 2.15.0+cpu; Ultralytics 8.4.215; transformers 5.23.0; huggingface_hub 2.4.0; CVAT 2.81.0 tại http://localhost:8090. Chạy CPU. Cảnh báo không GPU/chưa đăng nhập Hugging Face không cản phần bắt buộc đã làm.
- **Cache:** không dùng data/cache để thay các lượt tự chạy r05/r06. Metadata và nguồn merge chỉ tới work/prelabel/y26_all và work/prelabel/yoloe_all.
- **Lỗi đã gặp:** kết nối CVAT từ chối khi dịch vụ chưa sẵn sàng; task thiếu bộ nhãn; Docker/ổ C thiếu dung lượng; proxy mất cổng 8090; dấu nháy bị mất khi truyền Python -c; Copy-Item tìm .jpg vì SamName trống dù kết quả đã ghi trực tiếp vào submission/masks. Đã được hỗ trợ xử lý; không sửa mã nguồn script của lab hoặc taxonomy.rules để làm đẹp số đo.
- **Rà soát sau khi làm:** tái tính review_stats, threshold, merge, traps, release và QC trong thư mục audit riêng khớp đầu ra nộp; dữ liệu gốc, attribution và script giữ nguyên. Không chạy lại suy luận SAM nặng trong audit vì tài nguyên máy thấp; số pixel/polygon dựa trên console tôi đã cung cấp, không gọi đó là tái suy luận độc lập.
- **Một điều tôi chọn làm khác lần sau:** thực hiện ba quy tắc đã xác nhận ở mục 4, nhất là tự quét người ở xa và kiểm box trùng; giữ bằng chứng từng vòng và bấm giờ ngay từ đầu.
- **Trước khi nộp:** tôi đã xác nhận quyết định cuối ở mục 7 và ba quy tắc ở mục 4; cần đọc lại để giải thích được nội dung, làm rõ việc dùng AI xem ảnh theo điều kiện của lớp nếu chưa có xác nhận cho phép, xử lý thiếu số đo thời gian trung thực, commit công việc thật và điền hash trong STUDENT. Không nhận rằng các việc này đã hoàn tất khi chưa có bằng chứng.
