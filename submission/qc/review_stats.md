# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 234; đối tượng sau duyệt: 328; ngưỡng accepted: IoU >= 0.9
- Đã **loại 21 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 171 | 171/234 = 73.1% | 234 box máy |
| edited (sửa) | 36 | 36/234 = 15.4% | 234 box máy |
| deleted (xoá) | 27 | 27/234 = 11.5% | 234 box máy |
| added (vẽ thêm) | 121 | 121/328 = 36.9% | 328 đối tượng sau duyệt |

Trong số deleted + added, **2 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 119.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 88 | 17 | 4 | 62 |
| bicycle | 4 | 0 | 1 | 3 |
| motorbike | 41 | 17 | 8 | 38 |
| car | 23 | 2 | 3 | 14 |
| bus | 2 | 0 | 0 | 3 |
| truck | 0 | 0 | 2 | 0 |
| three_wheeler | 13 | 0 | 9 | 1 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 71 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 28 | 7 | 43 | 80.0% (KTC 95%: 64–90%) | 39.4% (KTC 95%: 29–51%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 11/12 = 91.7% (KTC 95%: 65–99%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 2/3
  - missing: 3/3
  - loose: 3/3
  - extra: 3/3
  - lọt: img_031.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
