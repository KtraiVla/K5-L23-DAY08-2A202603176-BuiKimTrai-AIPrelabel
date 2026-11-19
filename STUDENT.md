# Thông tin học viên

- Họ và tên: Bùi Kim Trại
- MSHV: 2A202603176
- Lớp: L23
- Tài khoản GitHub: KtraiVla
- Repo bài nộp (URL): https://github.com/KtraiVla/K5-L23-DAY08-2A202603176-BuiKimTrai-AIPrelabel
- Commit chốt bài (hash): Chưa chốt; bổ sung sau khi hoàn thành bài.
- Máy: Windows 11 Pro; AMD Ryzen 7 5800H; RAM 16 GB; Python 3.13.15 trong D:\day08\.venv; chạy model bằng CPU (PyTorch 2.15.0+cpu), không sử dụng CUDA/MPS cho lab.
- CVAT (Docker): v2.81.0, chạy bằng Docker Desktop/WSL2 trên máy tại http://localhost:8090.

## Kết quả tóm tắt (phân biệt số đo review, QC vòng 1 và QC vòng 2)

- Máy (trước calibration/QC): accepted 171/234 = 73.1%; edited 36/234 = 15.4%; deleted 27/234 = 11.5% (mẫu số: 234 box máy, đã loại vùng bẫy); added 121/328 = 36.9% (328 đối tượng sau review được tính). Không dùng 359 đối tượng release làm mẫu số cho các chỉ số review này.
- So với Gold: precision của máy 28/35 = 80.0%; recall 28/71 = 39.4% (4 ảnh Gold, 71 đối tượng; không phải chất lượng nhãn cuối).
- Bắt bẫy: vòng đầu 8/12 = 66.7%; sau sửa 11/12 = 91.7%. Ca còn bị script chấm lọt img_031 có đáp án máy gốc trên quần áo; vòng sau đã biết danh sách bẫy.
- Thời gian: gán tay 30/4 = 7.5 phút/ảnh; review hai lượt khoảng (45 + 10)/24 = 2.29 phút/ảnh. 45 phút đầu là ước tính; chưa gồm calibration/QC và sửa sau QC.
- Lỗi QC: vòng 1 (nguồn qc/round1_before_rework/) ghi 7/79 = 8.9% theo đối tượng, 5/6 = 83.3% theo ảnh. Vòng 2 sau sửa (nguồn qc/ và qc/round2_after_rework/): 0/100 = 0.0% theo đối tượng, 0/6 = 0.0% theo ảnh; cùng mẫu tái kiểm, cận trên Wilson 95% theo ảnh khoảng 39%, không chứng minh chất lượng toàn bộ ≤2%.
- **Quyết định của tôi: Release cho bộ nhãn detection cuối** — tôi đã sửa các lỗi vòng đầu, kiểm lại 6 ảnh và không thấy lỗi (0/6 ảnh, 0/100 đối tượng). Tôi tự review và tự QC, chưa có người kiểm độc lập; giữ giới hạn mẫu nhỏ và lịch sử Rework trong REPORT. Bộ cuối: 24 ảnh, 359 đối tượng; các mask thử nghiệm SAM vẫn có lỗi đã nêu ở mục 5.

Lưu ý trước nộp: tôi đã xác nhận quyết định trên; kết quả tự kiểm mẫu nhỏ không chứng minh chất lượng toàn bộ dữ liệu dưới ngưỡng 2%. Review 45 phút đầu là ước tính, chưa đáp ứng đầy đủ yêu cầu đo đồng hồ; không tự đổi thành số đo. Hash commit chốt và phần nộp GitHub chỉ hoàn tất sau khi commit/push thật.
