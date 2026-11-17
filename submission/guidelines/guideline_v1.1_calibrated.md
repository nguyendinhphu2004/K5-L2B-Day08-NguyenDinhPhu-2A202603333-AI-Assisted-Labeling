# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603333 - Nguyễn Đình Phú

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày đặc **gần camera**: từng xe hay `ignore_region`? | Gán **từng xe** khi xe cao ≥ 20 px **và** nhìn thấy riêng bánh hoặc yên của xe đó (tách được). Phần hàng xe phía xa, chồng lấp đến mức không thấy riêng bánh / yên → **một** `ignore_region` bao phần đó. | img_026 (xe gần gán từng chiếc, hai cụm xa là `ignore_region`) | CAL-03 |
| 5 | **Kính chắn gió / cửa sổ xe**: thuộc mask xe không? | **Có**: kính thuộc mask xe (kể cả khi nhìn xuyên thấy nội thất). Nhưng **khoang hở lớn** (không có kính) nhìn xuyên thấy nền / cửa hàng phía sau, chiếm > 5% diện tích xe → **bỏ khỏi mask** theo 4.2.2. Bàn chân người lái đặt trên sàn xe thuộc mask `person`, không thuộc mask xe (4.2.4: vùng chạm thuộc vật phía trước). | img_017 (SAM phủ cả nền trong khoang), img_013 (mask dính bàn chân người lái) | CAL-05 |

**Chưa chốt** (không có ca trong lô 24 ảnh batch + 4 gold để làm căn cứ, giữ nguyên v1.0):
- Câu 1 (xe đạp điện): không gặp xe đạp điện nhận ra được trong lô.
- Câu 3 (hàng nhô ra): một phần được xử lý ở CAL-01 (thùng hàng gắn xe thuộc box xe); hàng nhô xa (tre, rổ dài) chưa gặp.
- Câu 4 (người trong xe ba bánh có mui, ngưỡng 20%): chưa đo được độ lệch giữa hai người đánh giá vì làm bài một mình.
- Câu 6 (box máy chồng nhau ban đêm): img_024 xử lý bằng `ignore_region` cho phần xe nhoè không tách được; chưa đủ ca để chốt quy tắc gộp box.
- Câu 7 (ngưỡng tin cậy chấp nhận box máy không cần sửa): **không đặt**. Bẫy `wrong_class` / `loose` có `conf` 0.45–0.8 giống box thật, nên `conf` không thay được việc nhìn ảnh.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_003 | Xe kéo chở rau có thùng gỗ và mái che, có người ngồi trên; không đếm đủ số bánh (`can_review`) | YOLOE-26s gán `three_wheeler`; YOLO26m không có lớp này | Một box `three_wheeler` ôm cả thùng, mái, bánh; mỗi người trên xe một box `person` | **Mục 2, lớp `three_wheeler`**: xe ba gác / xe kéo hoặc đẩy chở hàng có **thùng gắn bánh** (có hoặc không có mái) → `three_wheeler`; không đếm được bánh vẫn gán `three_wheeler` và tick `can_review` (CAL-01). Áp dụng cho các xe đẩy trái cây ở img_008. |
| img_008 | Ba xe đẩy trái cây đứng sát nhau | YOLOE-26s vẽ box `three_wheeler` lệch / trùm sang xe bên cạnh (QC: #102, #99, #96 vượt 160–170 px) | Mỗi xe một box, cạnh box dừng ở mép nhìn thấy của chính xe đó | **Thêm 4.1.8**: vật cùng lớp đứng sát / chồng nhau → mỗi vật một box; không box nào được chứa trọn bánh hoặc thân của vật bên cạnh. Khi review, kiểm riêng từng box `three_wheeler` của YOLOE (CAL-02). |
| img_018 | Xe khách nhỏ (minibus) ở ngã tư, nhìn từ trên cao (QC vòng 2) | YOLOE-26s gán `three_wheeler` (#178) | `bus` (minibus chở khách theo định nghĩa `bus` của v1.0) | **Mục 2, lớp `three_wheeler`, thêm cột "Không gồm"**: xe có thân kín nhiều cửa sổ hoặc ≥ 4 bánh không bao giờ là `three_wheeler` (minibus → `bus`, bán tải → `car`). Recipe: thêm `bus`, `car` vào prompt YOLOE cùng `three_wheeler` (CAL-06). |
| img_025 | Người đứng trong vùng tối cuối hẻm, sát một người khác đã có box (bẫy `missing` lọt) | Box máy của người này bị xoá (bẫy); khi duyệt tôi không thêm lại | Người cao ≥ 20 px trong vùng tối hoặc đứng sát người khác vẫn là một box `person` riêng (bổ sung khi Rework) | **Bổ sung quy trình review** (sau mục 3): ở vùng tối, đông người, ẩn nhãn máy rồi **đếm theo số đầu người**, sau đó mới đối chiếu box (CAL-04). |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Mở rộng định nghĩa `three_wheeler` cho xe ba gác / xe kéo, đẩy chở hàng có thùng gắn bánh; thêm 4.1.8 (vật cùng lớp đứng sát: mỗi vật một box); chốt câu hỏi mở 2 (tiêu chí tách xe đỗ dày gần camera) và câu 5 (kính thuộc mask, khoang hở lớn bỏ khỏi mask, bàn chân người lái thuộc person); thêm bước đếm đầu người ở vùng tối vào quy trình review; thêm "Không gồm" cho `three_wheeler` (xe thân kín / ≥ 4 bánh, minibus → `bus`) | CAL-01 (img_003), CAL-02 (img_008), CAL-03 (img_026), CAL-04 (img_025), CAL-05 (img_017, img_013), CAL-06 (img_018) |
