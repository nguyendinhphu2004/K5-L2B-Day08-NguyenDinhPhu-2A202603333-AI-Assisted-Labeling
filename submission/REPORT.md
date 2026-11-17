# Báo cáo Day 08 — 2A202603333 - Nguyễn Đình Phú

> Số liệu kèm mẫu số và file nguồn. `rXX` là dòng trong `model_log.csv`, `CAL-XX` là dòng trong `qc/calibration_decisions.csv`.

## 1. Model và recipe (Part 2–3)

| Lớp                                         | Model                                 | Ngưỡng đề xuất (sweep)       | Ngưỡng thực dùng khi merge                          | Vì sao                                                                                                       |
| ------------------------------------------- | ------------------------------------- | ---------------------------- | --------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| person, bicycle, motorbike, car, bus, truck | YOLO26m                               | `motorbike=0.1`, còn lại 0.3 | `person=0.35, motorbike=0.5, car=0.35`, còn lại 0.3 | Bắt chắc người, xe máy, ô tô rõ (r01); conf 0.5 làm mất 76/215 box, chủ yếu vật nhỏ (r02)                    |
| three_wheeler                               | YOLOE-26s-seg, prompt `three_wheeler` | 0.15                         | 0.3                                                 | YOLO26 không có lớp này; YOLOE bắt được tuk-tuk / xích lô mà YOLO26 bỏ sót (img_002 0.93, img_032 0.92, r03) |

- **Lệnh merge đã dùng** (khôi phục từ file kết quả, vì điểm thấp nhất mỗi lớp trong file là motorbike 0.50, person 0.35, car 0.36, three_wheeler 0.30):
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.35,motorbike=0.5,car=0.35" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Sai sót của tôi:** tôi dán nguyên lệnh mẫu của hướng dẫn thay vì chuỗi `motorbike=0.1,three_wheeler=0.15` của mình, và chỉ phát hiện khi rà soát cuối bài. Kết quả là 173 box `motorbike` (0.10–0.50) và 20 box `three_wheeler` (0.15–0.30) bị loại khỏi pre-label. Tôi không merge lại vì như vậy phải cài bẫy và review lại cả 24 ảnh, nên mọi số ở mục 2–4 phản ánh chuỗi đã dùng.
- **SAM chọn:** SAM 2.1 base. Viền mui, khung, bánh xe mượt hơn MobileSAM trên cùng click (r07 so với r08), chạy được trên CPU khoảng 10 s/lần và không cần token như SAM 3.
- **Lỗi không sửa được bằng ngưỡng:** YOLO26 không có lớp `three_wheeler`, nên tuk-tuk thành `car`/`truck` hoặc không có box (img_002, img_013), xích lô thành `bicycle` (img_032). Đổi `--conf` chỉ đổi số box chứ không sinh ra lớp mới. Cách sửa là thêm YOLOE và đặt nó **trước** YOLO26 khi merge (box trùng thì giữ box YOLOE).
- **Điểm số không tách được đúng / sai:** YOLO26 gọi xe bán tải là `truck` 0.73 (img_002) và xích lô là `bicycle` 0.70 (img_032). Ngược lại, YOLOE 0.48 đúng một chiếc xích lô (img_032), còn box 0.37 cùng lớp lại nằm trên xe bán tải (img_002).
- **Giới hạn của ngưỡng:**
  - Ngưỡng được chọn và chấm trên cùng 4 ảnh gold, nên số đo luôn lạc quan.
  - `bicycle` (4 vật), `truck` (1), `bus` (0) quá ít mẫu để chọn ngưỡng riêng.
  - `motorbike` (11 vật) chỉ chạm recall 90% ở mức đáy 0.1, với precision 10/22; `three_wheeler` dựa trên 7 vật.
  - `person` (tối đa 22/28) và `car` (7/8 ở mọi ngưỡng) không bao giờ đạt 90%: đó là vật model không thấy, hạ ngưỡng cũng không lấy lại được.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số  | Tỉ lệ | Mẫu số                  |
| ---------- | --- | ----- | ----------------------- |
| accepted   | 355 | 82.0% | 433 box máy             |
| edited     | 27  | 6.2%  | 433 box máy             |
| deleted    | 51  | 11.8% | 433 box máy             |
| added      | 72  | 15.9% | 454 đối tượng sau duyệt |

Đã loại 23 đối tượng quanh các bẫy vì đó là lỗi nhân tạo. Trong deleted + added có 7 ca chỉ là đổi lớp, nên máy sót thật khoảng 65 vật.

- **Accepted 82% không có nghĩa là máy tốt.** Tôi vẫn phải vẽ thêm 72 vật, khoảng 1/6 lô.
- **Lớp tệ nhất:**
  - `three_wheeler`: xoá 12/26 box và vẽ thêm 8. YOLOE gọi xe kéo chở rau là xe ba bánh (img_003), sinh box thừa (img_014, img_023), nhưng vẫn sót xích lô ở img_010.
  - `truck`: xoá 5/7 box, phần lớn là xe bán tải (guideline xếp vào `car`) hoặc tuk-tuk.
  - `motorbike` và `car` bị sót nhiều nhất: vẽ thêm 29 xe máy ở các ảnh xe dày (img_021, img_028, img_005; một phần do merge lọc ở 0.5) và 18 ô tô, gần như toàn bộ ở img_024, ảnh đêm có xe nhoè sáng.
  - `person` tốt nhất (221/248 box giữ nguyên), chỉ sót người ngồi trong tối ở chợ đêm (img_022).
- **So với gold:** precision 40/61 = 65.6%, recall 40/59 = 67.8% (KTC 95%: 53–76% và 55–78%). Vẫn là số lạc quan vì ngưỡng chọn trên chính gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- **Bắt bẫy 11/12 = 91.7%** (KTC 95%: 65–99%): `wrong_class` 3/3, `extra` 3/3, `loose` 3/3, `missing` 2/3. Tôi chỉ mở `traps_key.json` sau khi đã export kết quả review.
- **Bẫy lọt:** `missing` ở img_025, một người đứng trong vùng tối cuối hẻm, sát ngay một người tôi đã gán. Tôi bỏ sót ở **bước 1 của checklist (đếm trước khi nhìn box)**: đếm hai người đứng sát nhau thành một, nên khi bật nhãn thấy có box ở đó là yên tâm. Lỗi `missing` khó nhất vì màn hình không có gì nhắc rằng đang thiếu.
- **Accepted và bắt bẫy cùng cao** cho thấy tôi không bấm chấp nhận qua loa: các bẫy "trông tự nhiên" (`wrong_class`, `loose`) đều bị bắt. Điểm mù của tôi trùng với điểm yếu của máy: **vật không có box** ở vùng tối, chen chúc. Với chỉ 12 bẫy, khoảng tin cậy quá rộng để khẳng định tôi luôn đạt trên 80%.

## 4. Máy có đáng dùng không?

- **Thời gian**: gán tay 54.6 phút / 4 ảnh = **13.7 phút/ảnh**; duyệt có pre-label 75.3 phút / 24 ảnh = **3.1 phút/ảnh**. Tính theo vật: khoảng 0.87 so với 0.17 phút/vật.
- **Thiên lệch:** tôi gán gold trước nên đã quen cảnh, quen lớp và quen CVAT khi duyệt batch; số đo cũng không gồm thời gian chạy script và export.
- **Kết luận: có, nhưng chỉ như bản nháp.** Nhanh hơn khoảng 4 lần và 82% box giữ nguyên là lợi ích thật với `person`, `motorbike`, `car` ban ngày. Nhưng recall chỉ khoảng 68%, máy sót khoảng 65 vật, và lô vẫn phải Rework (mục 7). Với `three_wheeler`, ảnh đêm và xe đỗ trong tối, box máy chỉ là gợi ý.
- **Ba quy tắc giảm thiên kiến** (mỗi quy tắc đo được):
  1. **Đếm trước, nhìn box sau:** ẩn nhãn máy, ghi số người / xe đếm được, rồi mới bật nhãn. Đo bằng: bẫy `missing` đạt 3/3 ở lô sau.
  2. **Soi riêng từng box `three_wheeler`** bằng Filters, kiểm tra cạnh box dừng đúng ở mép xe của chính nó (CAL-02). Đo bằng: lỗi `sai_vi_tri_kich_thuoc` của lớp này trong QC bằng 0 (lô này 3/3 lỗi loại đó đều ở `three_wheeler`).
  3. **Phóng to mọi vùng tối và hàng xe đỗ** trước khi chuyển ảnh; vật cao ≥ 20 px phải có box hoặc nằm trong `ignore_region`. Đo bằng: lỗi `bo_sot` trong QC về 0 (hai vòng QC có 6 lỗi loại này, đều là xe đỗ trong tối).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh     | SAM          | Click          | Dính người lái / hàng?            | Nhận xét                                                                        | File                          |
| ------- | ------------ | -------------- | --------------------------------- | ------------------------------------------------------------------------------- | ----------------------------- |
| img_013 | SAM 2.1 base | 2 dương + 1 âm | Có: mặt, chân, bàn chân người lái | 1 điểm âm chỉ khoét được thân áo (mask 45.4% ảnh)                               | `tuktuk_img013_sam2.jpg`      |
| img_013 | MobileSAM    | 2 + 1          | Ít hơn, vẫn dính mặt, bàn chân    | Tách thân người tốt hơn nhưng viền thô, có đốm thủng (35.2%)                    | `tuktuk_img013_sam1.jpg`      |
| img_013 | SAM 2.1 base | 2 + 5          | Không                             | Quá nhiều điểm âm: mask sụp còn mui và khung (12.4%)                            | `tuktuk_img013_sam2_neg5.jpg` |
| img_013 | SAM 2.1 base | 4 + 3          | Gần như không, còn bàn chân       | Tốt nhất (40.2%), nhưng lan sang xe máy đỏ phía sau                             | `tuktuk_img013_sam2_v3.jpg`   |
| img_004 | SAM 2.1 base | 2 + 1          | Không                             | Ôm khít xe, nhưng điểm dương sát mặt đường làm dính bóng gầm                    | `tuktuk_img004_sam2.jpg`      |
| img_017 | SAM 2.1 base | 2 + 1          | Không                             | Điểm âm rơi ngay trên mui nên vô tác dụng; mask gồm cả nền nhìn xuyên khoang hở | `tuktuk_img017_sam2.jpg`      |

- **SAM 2.1 so với MobileSAM:** SAM 2.1 cho viền đẹp hơn nhưng dính người nhiều hơn khi chỉ có 1 điểm âm; MobileSAM nhanh hơn khoảng 1.6 lần. Tôi chọn SAM 2.1 vì phần dính người sửa được bằng thêm click, còn viền thô thì không.
- **Bài học về click:** 1 điểm âm không tách được người ngồi chồng lên xe, nhưng dồn nhiều điểm âm thì mask sụp. Cách hiệu quả là thêm điểm dương ở bánh và đuôi xe để giữ thân, rồi mới đặt điểm âm lên người.
- **Đối chiếu guideline 4.2:** quy tắc 4.2.3 (bóng không thuộc vật) và 4.2.4 (vùng chạm thuộc vật phía trước) giúp nhận ra lỗi ở img_004 và img_013. Còn thiếu: bàn chân người lái trên sàn xe, nền nhìn xuyên khoang hở, kính chắn gió (đã chốt ở CAL-05), và cách đặt click (không đặt điểm âm lên chính vật).

## 6. Calibration (Part 6)

- **6 quyết định** (CAL-01 → 06): 1 `can_review`, 2 `loi_may`, 2 `cau_hoi_mo`, 1 `bay_lot`. Quan trọng nhất là **CAL-02** (img_008): YOLOE vẽ box `three_wheeler` trùm sang xe bên cạnh khi các xe đẩy đứng sát nhau. Lỗi này gây 3/7 lỗi QC vòng 1 và tôi cũng không bắt được khi review, nên cần thành quy tắc thay vì chỉ sửa tay.
- **Quy tắc mới trong v1.1:**
  - Xe ba gác, xe kéo hoặc đẩy chở hàng có thùng gắn bánh là `three_wheeler` (CAL-01, img_003).
  - Vật cùng lớp đứng sát nhau: mỗi vật một box, không trùm sang vật bên cạnh (CAL-02, img_008).
  - Xe đỗ dày gần camera: gán từng xe khi cao ≥ 20 px và thấy riêng bánh hoặc yên; phần xa gom `ignore_region` (CAL-03, img_026).
  - Ở vùng tối, đếm theo số đầu người trước khi bật nhãn máy (CAL-04, img_025).
  - Mask: kính thuộc xe, khoang hở lớn bỏ khỏi mask, bàn chân người lái thuộc `person` (CAL-05, img_017, img_013).
  - `three_wheeler` không gồm xe thân kín hoặc ≥ 4 bánh; minibus là `bus` (CAL-06, img_018).
  - Câu hỏi mở 1, 3, 4, 6, 7 chưa chốt vì lô không có ca làm căn cứ.
- **Nguyên nhân gốc:** guideline mơ hồ (CAL-01, 03, 05); máy sai (CAL-02, 06); tôi sai dù quy tắc đã rõ (CAL-04, các xe đỗ bị sót ở img_033, img_035, và box 11.9 px còn trong gold img_001).

## 7. QC và quyết định (Part 6)

Tôi làm QC **hai vòng**. Phiếu vòng 1 lưu ở các file `qc/*_round1.*`, file không có hậu tố là vòng 2. Mỗi vòng gồm 6 ảnh (1 ảnh `can_review` và 5 ảnh ngẫu nhiên). Tôi tự kiểm, có AI hỗ trợ phóng to và đo toạ độ. **Hạn chế:** tôi vừa duyệt vừa QC nên dễ lặp lại điểm mù của chính mình.

|                    | Vòng 1 (seed 42)                                                                                                                                      | Vòng 2 (seed 43, sau khi sửa)                                                                                            |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Lỗi theo đối tượng | 7/79 = 8.9%                                                                                                                                           | 4/139 = 2.9%                                                                                                             |
| Lỗi theo ảnh       | 2/6 = 33.3%                                                                                                                                           | 2/6 = 33.3%                                                                                                              |
| Lỗi chính          | `sai_vi_tri_kich_thuoc` 3 (img_008, box xe đẩy trùm nhau); `bo_sot` 3 (img_033, xe đỗ trong tối); `sai_lop` 1 (img_033, người đạp xe gán `motorbike`) | `bo_sot` 3 (img_035, xe đỗ phía sau); `sai_lop` 1 (img_018, minibus gán `three_wheeler`). Script gắn cờ **lỗi hệ thống** |

- **Giữa hai vòng, tôi sửa 4 ảnh** rồi export `labels/qc_fixed.json` và chạy lại `build_release.py` (472 → 476 đối tượng): kéo lại 3 box ở img_008; đổi lớp và thêm 4 xe đỗ ở img_033; thêm người bị sót ở img_025; xoá ô tô 17 px ở img_027.
- **Giới hạn mẫu nhỏ:** 2/6 ảnh lỗi cho cận trên Wilson 95% là 70%. Ngay cả mẫu sạch 0/6 vẫn cho cận trên khoảng 39%, nên 6 ảnh không bao giờ chứng minh được lô đạt ≤ 2%, chỉ phát hiện được lỗi lớn.
- **Script không kiểm được** box có ôm sát không, vật mà cả máy lẫn người cùng bỏ sót, hay tôi có làm đủ checklist không. Nó chỉ đếm theo phiếu tôi điền.

**Quyết định: Rework** (2026-11-17), xét theo Hold → Rework → Release:

1. **Không Hold:** các ca mơ hồ đã chốt trong v1.1, không có ảnh hỏng.
2. **Rework:** sau một vòng sửa vẫn có 2/6 ảnh lỗi (≫ 2%), và vòng 2 tìm ra lỗi mới ở ảnh chưa sửa. Lỗi lan khắp lô, không phải vài ảnh lẻ.

**Việc tiếp theo:** sửa img_035 và img_018; rà cả 24 ảnh theo quy tắc 2 và 3 ở mục 4; với lô sau, thêm `bus`, `car` vào prompt YOLOE và dùng `motorbike=0.1`; sau đó QC lại với mẫu lớn hơn (khoảng 12 ảnh).

## 8. Ghi chú cá nhân

- **Dùng AI:** có, dùng Claude (Claude Code) để:
  - đọc và tóm tắt tài liệu của bài (file hướng dẫn HTML/MD, `RUBRIC.md`, `SUBMISSION.md`, guideline v1.0) cùng mã nguồn các script trong repo, để hiểu yêu cầu từng pha, cách `review_stats.py` / `inject_traps.py` / `qc_report.py` tính số và các quy định chấm;
  - cài môi trường, CVAT, Nuclio và các model AI Tools;

  **Không dùng AI** để gán gold hay review batch, và AI không mở `traps_key.json` trước khi review xong. Tôi kiểm lại mọi con số với file gốc (`review_stats.md`, `threshold_sweep.csv`, `qc_report_auto.md`, `work/runs.csv`), mở lại ảnh để xác nhận nhận xét, và chạy `validate_labels.py`, `check_submission.py`.

- **Máy và cài đặt:** Windows 10, 16 GB RAM, chạy model bằng CPU; CVAT 2.81 trên Docker Desktop + WSL2. Vướng mắc: WSL thiếu RAM khi bật SAM, timeout của Nuclio và nginx quá ngắn cho CPU, Nuclio đổi cổng sau mỗi lần khởi động lại Docker, ổ đĩa đầy vì Docker. Tất cả đã xử lý.
