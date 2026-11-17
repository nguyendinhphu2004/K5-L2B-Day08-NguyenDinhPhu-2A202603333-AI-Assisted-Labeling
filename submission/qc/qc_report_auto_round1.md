# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 6 ảnh (1 ca khó + 5 ngẫu nhiên), 79 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 7/79 = 8.9% | 79 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 2/6 = 33.3% | 6 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 355/433 = 82.0% / 27/433 = 6.2% / 51/433 = 11.8% | 433 box máy |
| Máy: added | 72/454 = 15.9% | 454 đối tượng sau duyệt |
| Bắt lỗi bẫy | 11/12 = 91.7% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| sai_vi_tri_kich_thuoc | 3 | img_008.jpg |
| bo_sot | 3 | img_033.jpg |
| sai_lop | 1 | img_033.jpg |

## Giới hạn thống kê
- 2 ảnh lỗi trong 6 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **70%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 33.3% > 2%

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
