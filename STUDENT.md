# Thông tin học viên

- Họ và tên: Nguyễn Đình Phú
- MSHV: 2A202603333
- Lớp (L2 / L3 / khác): L2B
- Tài khoản GitHub: https://github.com/nguyendinhphu2004
- Repo bài nộp (URL): https://github.com/nguyendinhphu2004/K5-L2B-Day08-NguyenDinhPhu-2A202603333-AI-Assisted-Labeling
- Commit chốt bài (hash):
- Máy: Windows 10, 16 GB RAM, GPU NVIDIA RTX 3050 Laptop 4 GB nhưng chạy model bằng **CPU** (torch bản CPU, không dùng CUDA)
- CVAT (Docker): v2.81.0 (Docker Desktop + WSL2, Nuclio 1.16.3 cho AI Tools)

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 355/433 = 82.0% / 27/433 = 6.2% / 51/433 = 11.8% (mẫu số: 433 box máy); added: 72/454 = 15.9% (mẫu số: 454 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 40/61 = 65.6% / 40/59 = 67.8% (59 đối tượng gold)
- Bắt bẫy: 11/12 = 91.7% (lọt 1 bẫy `missing` ở img_025)
- Thời gian: gán tay 54.6 phút / 4 ảnh = 13.7 phút/ảnh; duyệt có pre-label 75.3 phút / 24 ảnh = 3.1 phút/ảnh (working_time do CVAT ghi)
- Lỗi QC: vòng 1 (trước khi sửa) 33.3% theo ảnh (2/6), 8.9% theo đối tượng (7/79); vòng 2 (sau Rework 4 ảnh, mẫu mới seed 43) 33.3% theo ảnh (2/6), 2.9% theo đối tượng (4/139)
- **Quyết định:** Rework — lý do một dòng: sau một vòng sửa, mẫu mới vẫn có 2/6 ảnh lỗi (≫ 2%, cận trên Wilson 70%) và `bo_sot` xe đỗ trong tối / phía sau là lỗi hệ thống lan khắp lô, cần rà toàn bộ 24 ảnh theo recipe mới
