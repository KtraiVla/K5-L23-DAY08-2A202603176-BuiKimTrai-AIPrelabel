# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603176 - Bùi Kim Trại

Chỉ ghi bổ sung và làm rõ so với v1.0. Bốn quyết định CAL-01–CAL-04 đã được ghi vào calibration_decisions.csv; nhãn sửa ở CAL-02 đã có trong qc_fixed.json và bộ release candidate. QC vòng 2 sau sửa được học viên xác nhận không thấy lỗi ở 6 ảnh/100 đối tượng; quyết định kiểm mẫu và giới hạn được ghi trong REPORT, không suy rộng chất lượng toàn bộ dữ liệu. Không thay đổi taxonomy.rules.

## 1. Câu hỏi mở và phạm vi đã chốt

| Câu hỏi v1.0 | Quyết định | Ảnh ví dụ | Ca |
|---|---|---|---|
| Câu 1: phân biệt xe dạng xe máy và xe đạp trong ca img_025 | Xe cạnh ghế đỏ bên phải thuộc motorbike theo cấu tạo; đã bổ sung nhãn. Không suy đoán nhiên liệu; chưa khái quát cho mọi xe đạp trợ lực điện. | img_025 | CAL-02 |
| Câu 3: hàng hóa và phạm vi mask xe | Mask xe không gồm người lái, hành khách, hàng hóa rời, nền hoặc xe bên cạnh. Giữ khung, mái và bộ phận/giỏ gắn cố định thuộc xe. Quy tắc này được học viên xác nhận; nhận xét đã được bổ sung riêng: img_003 lấy thừa mặt trẻ và hàng hóa; img_013 thiếu nhẹ viền bánh trước và ít lem hơn; img_017 được học viên đánh giá chuẩn hơn hai ảnh trước. Không suy ra cả ba cùng có mọi loại lấy thừa. | img_003, img_013, img_017 | CAL-01 |

Phần box hàng hóa giữ quy tắc v1.0 mục 4.1.5 về hàng nhô hơn khoảng 20% chiều dài xe; không tự thêm một ngưỡng số mới cho mask. Phần bị che không được đoán để vẽ bù. Kính xe và phần người nhìn xuyên kính chưa được bổ sung quy tắc mới trong quyết định này.

Câu 1 được làm rõ trong phạm vi ca đã xem: xe ở mép phải cạnh ghế đỏ của img_025 có cấu tạo xe máy hai bánh, gán motorbike mà không suy đoán xăng hay điện. Học viên đã thêm box motorbike x=930.72, y=249.94, rộng=93.28, cao=234.94 và export qc_fixed.json. Xe đạp chỉ đạp chân vẫn là bicycle; ca không đủ căn cứ chọn lớp dùng can_review theo v1.0, không bỏ qua vật đủ điều kiện. Quyết định này không khẳng định đã phân loại được mọi dạng xe đạp điện có bàn đạp/trợ lực chưa quan sát (CAL-02).

Các câu 2, 4, 5, 6, 7 chưa có quyết định bổ sung riêng được học viên xác nhận; giữ v1.0 và rà lại nếu phát sinh trong nhãn cuối/QC. Nếu một câu chưa rõ còn ảnh hưởng nhãn thật thì phải giải quyết hoặc nêu Hold, không coi việc chưa có quyết định là đã đạt QC.

## 2. Ca khó và cách áp dụng

| Ca | Ảnh | Tình huống | Cách xử lý | Loại thay đổi |
|---|---|---|---|---|
| CAL-01 | img_003, img_013, img_017 | img_003 lem vào mặt đứa bé và hàng hóa; phần còn lại mượt, không thấy thiếu. img_013 ít lem hơn nhưng thiếu nhẹ viền bánh trước. img_017 được học viên đánh giá chuẩn hơn hai ảnh trước | Đối chiếu biên vật thật, tách người và vật khác; có thể dùng điểm âm để loại vùng thừa. Chưa báo cáo rằng đã chạy lượt sửa mask mới. | Làm rõ phạm vi mask và lỗi áp dụng, không đổi ngưỡng hình học |
| CAL-02 | img_025 | Xe bên phải cạnh ghế đỏ chưa được gán nhãn trong vòng review | Đã thêm motorbike trong qc_fixed.json; giữ bản batch_reviewed và thống kê review trước calibration | Làm rõ câu hỏi mở trong phạm vi ca và sửa nhãn thực tế |
| CAL-03 | img_031 | Box máy gốc trên quần áo trưng bày bị gọi person, sau đó biến thành bẫy wrong_class; script yêu cầu khôi phục person để được điểm | Không thêm person tại vùng không có người thật. Giữ điểm script 11/12 và giải thích bằng ảnh; không sửa đáp án hoặc tự cộng thành 12/12. | Áp dụng quy tắc v1.0 loại hình in/tượng/vật không thuộc lớp; ghi hạn chế đo bẫy, không tạo lớp mới |

### Bổ sung CAL-04 — img_003 sau QC vòng 1

Học viên xác nhận xe bán hàng giữa img_003 nằm ngoài các lớp của bài nên đã xóa cả hai box three_wheeler. Áp dụng quy tắc v1.0: vật ngoài phạm vi không gán và không tự vẽ ignore_region. Không tự suy ra một xe là three_wheeler chỉ từ tên xe bán hàng; quyết định phải theo cấu tạo quan sát và phạm vi taxonomy. Đây là quyết định áp dụng của học viên, không phải Codex đã đếm chắc số bánh. Trước đó Codex diễn giải quá chắc trường hợp này thuộc three_wheeler; đã ghi đính chính. Những xe ba bánh chở hàng thực sự vẫn thuộc three_wheeler theo v1.0.

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Bằng chứng |
|---|---|---|
| v1.0 | Quy tắc gốc của giảng viên | guideline_v1.0_label_contract.md |
| v1.1 — đã ghi kết quả QC trong REPORT | Làm rõ phần xe trong mask, ghi giới hạn đáp án bẫy; CAL-02 đã bổ sung motorbike cạnh ghế đỏ của img_025 | CAL-01–CAL-04 theo xác nhận học viên và các bản export; xem kết quả/giới hạn QC ở REPORT |

Áp dụng v1.0 mục 4.1.6 trước QC: học viên đã xóa box car cao 17.12 px ở img_027 và export lại qc_fixed. Đây là sửa lỗi theo ngưỡng 20 px có sẵn, không phải tạo quy tắc mới hoặc thay taxonomy.rules.


Đây là bản biên tập các quyết định từ quá trình làm bài đã trao đổi; người học cần tự rà và diễn đạt/chốt quy tắc theo RULES trước nộp. Không tự nhận Codex đã xác minh độc lập mọi nhãn hoặc đếm chắc số bánh ở CAL-04.
