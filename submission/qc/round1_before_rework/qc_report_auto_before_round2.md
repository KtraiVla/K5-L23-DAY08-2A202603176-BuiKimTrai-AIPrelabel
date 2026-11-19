# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 6 ảnh (1 ca khó + 5 ngẫu nhiên), 79 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 7/79 = 8.9% | 79 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 5/6 = 83.3% | 6 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 171/234 = 73.1% / 36/234 = 15.4% / 27/234 = 11.5% | 234 box máy |
| Máy: added | 121/328 = 36.9% | 328 đối tượng sau duyệt |
| Bắt lỗi bẫy | 11/12 = 91.7% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| bo_sot | 6 | img_006.jpg, img_013.jpg, img_014.jpg |
| thua | 1 | img_003.jpg |

**Lỗi hệ thống**: 'bo_sot' lặp lại -> sửa quy tắc / recipe, không chỉ sửa từng ảnh.

## Giới hạn thống kê
- 5 ảnh lỗi trong 6 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **97%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 83.3% > 2%
- lỗi hệ thống: 'bo_sot' chiếm 6/7 lỗi

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
