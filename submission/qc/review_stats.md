# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 433; đối tượng sau duyệt: 454; ngưỡng accepted: IoU >= 0.9
- Đã **loại 23 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 355 | 355/433 = 82.0% | 433 box máy |
| edited (sửa) | 27 | 27/433 = 6.2% | 433 box máy |
| deleted (xoá) | 51 | 51/433 = 11.8% | 433 box máy |
| added (vẽ thêm) | 72 | 72/454 = 15.9% | 454 đối tượng sau duyệt |

Trong số deleted + added, **7 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 65.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 221 | 10 | 17 | 10 |
| bicycle | 4 | 1 | 4 | 3 |
| motorbike | 73 | 10 | 6 | 29 |
| car | 43 | 2 | 6 | 18 |
| bus | 2 | 0 | 1 | 1 |
| truck | 2 | 0 | 5 | 3 |
| three_wheeler | 10 | 4 | 12 | 8 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 59 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 40 | 21 | 19 | 65.6% (KTC 95%: 53–76%) | 67.8% (KTC 95%: 55–78%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 11/12 = 91.7% (KTC 95%: 65–99%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 2/3
  - wrong_class: 3/3
  - extra: 3/3
  - loose: 3/3
  - lọt: img_025.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
