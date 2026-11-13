# Báo cáo thực hành Day 3 — điền trực tiếp trong fork của bạn

- Mã học viên theo lớp: **26A202604065**
- Ngày / CVAT local: **11/11/2026** (task `easy_task`, CVAT local), rồi **12/11 – 13/11/2026** (task `medium_instance` task id 41, task `hard_panoptic` task id 42 — CVAT local vẫn đang chạy, user `tranthai`).
- Công cụ đã dùng: **CVAT local** với **Brush + Polygon vẽ tay**, không dùng SAM và không dùng Intelligent Scissors; dùng thêm `scripts/inspect_submissions.py` của repo và Python (Pillow + NumPy + OpenCV) để tự đo coverage / chồng lấn, và để đọc lại nội dung ZIP đã export.

## 1. Bài đã nộp

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | `submissions/easy_semantic.zip` | 3 / 3 | 20 |
| medium_instance | `submissions/medium_instance.zip` | 3 / 3 | 32 |
| hard_panoptic | `submissions/hard_panoptic.zip` | 1 / 2 | 30 |
| cp1_holes | chưa có | 0 / 1 | 3 |
| cp2_slice | chưa có | 0 / 1 | 3 |
| cp5_occlusion | chưa có | 0 / 1 | 3 |
| cp3_thin | chưa có | 0 / 1 | 3 |
| cp4_curb | chưa có | 0 / 1 | 3 |
| cp6_coverage | chưa có | 0 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

Ghi chú trạng thái:

- `easy_semantic` — đã vẽ đủ **3/3 ảnh** (`7ee6d192-89e2408b.jpg`, `817bca71-00000000.jpg`, `81ae7cbb-6bc63a4a.jpg`), nhãn đúng 5 lớp trong `data/tiers/easy_semantic/classes.json` (`road`, `sidewalk`, `building`, `vegetation`, `sky`; dán nguyên mảng JSON trong `cvat-labels.json` vào ô **Labels → Raw**). Trong 3 ảnh có **10 mask + 3 polygon** (`sky` vẽ bằng polygon vì biên chân trời gấp khúc, brush không ôm được). ZIP đã export đúng **Segmentation mask 1.1** → có `SegmentationClass/*.png` (3 file) + `labelmap.txt` (6 dòng: `background`, `building`, `road`, `sidewalk`, `sky`, `vegetation`).
- `medium_instance` — đã vẽ đủ **3/3 ảnh**, export đúng **COCO 1.0**. Tổng **77 mask**: `000000181542.jpg` 29 object (19 `person`, 5 `motorcycle`, 4 `car`, 1 `bus`), `000000373353.jpg` 24 object (14 `car`, 9 `person`, 1 `bus`), `000000458325.jpg` 24 object (14 `car`, 10 `person`). **Không có** `bicycle` và `truck` trong 3 ảnh này nên tôi không tạo object nào cho 2 lớp đó.
- `hard_panoptic` — mới làm được **1/2 ảnh**: `000000350023.jpg` có **34 shape** = 4 mask stuff (`road`, `sidewalk`, `building`, `vegetation`) + 30 shape thing (`sky` 1, `car` 18, `traffic light` 4, `person` 3, `bicycle` 2, `bus` 1, `motorcycle` 1). **Ảnh `000000460147.jpg` chưa kịp vẽ** nên trong COCO 1.0 file này ảnh có mặt trong `images` nhưng **0 annotation** — tôi ghi rõ ở đây thay vì tô bừa, xem mục 4.
- Định dạng ZIP: cả 3 ZIP đã nộp đều là **đúng format** theo `docs/SUBMISSION.md` (semantic → Segmentation mask 1.1, instance/panoptic → COCO 1.0). Chạy `python scripts/inspect_submissions.py --dir submissions` ra `[OK]` cho cả 3 task; 6 task checkpoint còn lại báo `[THIẾU]` vì tôi chưa làm, nên tôi không tạo ZIP rỗng cho chúng.
- Chưa xem Summary tự đánh giá trên GitHub Actions (reference chưa được phát) → **chưa có điểm**.

## 2. Một quyết định trước khi dùng gợi ý

- Object Medium **đầu tiên** tôi tự vẽ, trước khi xem bất kỳ gợi ý nào: ở ảnh **`000000181542.jpg`**, **người đi bộ đứng sát bên trái ảnh**, class **`person`**, ở khoảng **x 38–98, y 163–266** (640×600). Đây là object tôi kê đầu tiên trong danh sách Objects của CVAT và cũng là object đầu tiên theo thứ tự đọc từ trái sang.
- Quy tắc tôi tự đặt trước khi vẽ: **mỗi object một mask riêng**, biên bám theo **phần nhìn thấy của vật thể** (vẽ ước lượng theo mắt, không cần pixel-perfect, không bám theo ảnh nền xung quanh). Với người bị che khuất, tôi vẽ phần thân/đầu còn nhìn thấy và **không** vẽ vùng bị che.
- Sau khi vẽ xong object đầu tiên, tôi **không dùng gợi ý nào** (không có SAM, không có Intelligent Scissors trên CVAT local của tôi). Cách tôi tự kiểm là đối chiếu lại trên ảnh gốc bằng mắt, rồi dùng script tự đo số object / coverage / chồng lấn.
- Quyết định còn giữ lại cho cả task Medium: ở ảnh `000000458325.jpg`, khi thấy một cụm xe ở giữa ảnh, tôi **vẽ từng chiếc một** (14 `car` riêng) thay vì vẽ một khối chung, vì đề bắt buộc "mỗi vật thể một mask riêng".
- Một quyết định khác ở `easy_semantic`: ở `7ee6d192-89e2408b.jpg` không có `sidewalk` và `building` trong khung hình, tôi **không tạo mask nào cho hai lớp đó** — thiếu lớp thì thiếu, tô bừa thì sai cả vùng.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: **cả 2 task mới (`medium_instance` và `hard_panoptic`)**, ở mức **định dạng file nộp + kiểu shape** (lỗi ở mức task, không phải một vùng ảnh cụ thể).
- Lỗi thuộc loại: **khác** — export sai format, không phải sai lớp / thiếu-thừa vật / gộp-tách / biên / phủ vùng.
- Bằng chứng tôi nhìn thấy:
  1. `python scripts/inspect_submissions.py --dir submissions` in ra `[LỖI] medium_instance` và `[LỖI] hard_panoptic` với cùng một dòng: *"cần đúng một COCO JSON trong annotations/"*. Tôi mở ZIP ra thì cả hai chỉ chứa **một file `annotations.xml`** — đúng định dạng **CVAT for images 1.1**, không phải COCO 1.0 như `docs/SUBMISSION.md` yêu cầu. (Lỗi này cũng còn đúng với `easy_semantic.zip` lúc đầu: ZIP là CVAT 1.1 thay vì Segmentation mask 1.1, đã nêu ở lần báo cáo trước.)
  2. Sau khi export thử đúng COCO 1.0 từ CVAT local, tôi đọc file JSON ra và thấy **toàn bộ 77 annotation của `medium_instance` có `"segmentation": []`**, và 30/34 annotation của `hard_panoptic` cũng rỗng. Nguyên nhân: trong CVAT, **mọi object tôi vẽ bằng công cụ Rectangle** (`engine_labeledshape.type = 'rectangle'`), mà COCO 1.0 chỉ mang `segmentation` cho polygon/mask — rectangle ra COCO thành `bbox` rỗng. Bộ chấm `lab_utils._rasterize` bỏ qua mask rỗng nên nếu giữ nguyên thì **0 object được tính**, không phải "vẽ hơi thô" mà là **không có mask nào cho điểm**.
- Quy tắc và hành động sửa: (a) export lại đúng format trong CVAT — Job menu → **Export job dataset** → `Segmentation mask 1.1` cho `easy_task`, `COCO 1.0` cho `medium_instance` và `hard_panoptic`; (b) đổi kiểu shape từ **Rectangle sang Polygon** ngay trong dữ liệu CVAT, **giữ nguyên 4 toạ độ góc của rectangle** nên hình học mask không đổi (chỉ đổi cách lưu), rồi Save và export lại lần nữa. Tôi **không sửa ZIP bằng tay** — đúng như `GUIDE.md` dặn.
- Sau sửa đã Save và export lại chưa: **đã Save và đã export lại ở CVAT local**, và 3 ZIP trong `submissions/` trên máy này đã là bản export mới.
- Kết quả tự đánh giá liên quan: **chưa có điểm** (reference chưa được phát, GitHub Actions chưa có Summary). Bằng chứng thay bằng điểm số của tôi là output của `inspect_submissions.py` và của chính file COCO đã export:
  - `easy_semantic` → `[OK]`, có `SegmentationClass/` (3 PNG) + `labelmap.txt`.
  - `medium_instance` → `[OK]`, `annotations: 77 {'polygon': 77}`.
  - `hard_panoptic` → `[OK]`, `annotations: 34 {'polygon': 30, 'RLE': 4}`.
  - Tự đo bằng script trên file COCO đã export: coverage mask `medium_instance` = 60,6% / 20,6% / 37,3% theo 3 ảnh; `hard_panoptic` ảnh 1 = 69,8% ảnh, chồng lấn toàn ảnh chỉ 816 px (0,27%).

  Một lỗi thứ hai tôi phát hiện khi tự đo (đang để ở mục 4 vì chưa sửa xong): **chồng lấn giữa các lớp semantic** — ở `817bca71-00000000.jpg`, `sky ∩ vegetation` = 129 643 px (14,1% ảnh) và `building ∩ sky` = 123 656 px (13,4% ảnh); ở `81ae7cbb-6bc63a4a.jpg`, `sky ∩ vegetation` = 132 076 px (14,3%) và `building ∩ sky` = 67 388 px (7,3%). Semantic chỉ được có 1 lớp cho mỗi pixel nên đây là lỗi cần sửa bằng cách sửa lại biên (không phải sửa bằng cách tô đè).

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| 1 | `medium_instance` / `000000458325.jpg` — cụm xe ở giữa ảnh. Cách hiểu A: có **2 chiếc xe riêng biệt** đứng cạnh nhau (tôi vẽ 14 `car`). Cách hiểu B: một số "xe" tôi vẽ thực chất là **cùng một chiếc xe bị vẽ trùng** (vì biên các ô vuông chồng lên nhau rất nhiều). | Khi tự đo bằng script, cặp ô vuông có **IoU = 0,71**: `[99,183,192,261]` và `[109,183,203,252]` — hai ô gần như lồng nhau, nghĩa là cùng một vật thể. Ngoài ra `car` tự chồng nhau 57 235 px (20,9% ảnh) và có cặp IoU = 0,56 `[355,195,436,256]` vs `[338,184,424,250]`. | Tôi **chưa sửa xong**, vì sửa thì phải xem lại trên ảnh từng cặp và quyết định cái nào là vật thể thật. Câu hỏi cho coach: **với các xe chồng khuất, có được vẽ thành 2 object riêng (mỗi object một mask) không, hay phải gộp lại thành 1 object với biên bao trọn cả cụm?** Trong `medium_instance` mỗi vật thể một mask riêng thì tôi đang làm theo hướng tách hết, nhưng nếu coach nói phải gộp thì số object của tôi đang thừa. |
| 2 | `hard_panoptic` / `000000350023.jpg` — vùng trời phía trên. Cách hiểu A: `sky` là **một vùng liên tục chạy hết chiều ngang ảnh** (tôi vẽ một shape `sky` phủ từ x=0 đến x=640, y=0 đến y≈313, z_order = −2 để nằm dưới tất cả). Cách hiểu B: phần trời bị cây/tòa nhà che **không tính vào `sky`** nên vùng trời phải bị cắt theo tán cây và theo đường chân trời. | Mặc dù tôi đặt `sky` z_order = −2 để nó nằm dưới mọi thứ khác, khi đo thì `sky` vẫn **chồng** với `building` 234 px và với `vegetation` (tổng chồng lấn giữa 5 lớp stuff chỉ 373 px ≈ 0,12% ảnh — nhỏ, nhưng vẫn chồng). Biên của `sky` là **đường thẳng ngang** chứ không ôm theo tán cây. | Tôi giữ cách hiểu A (sky là nền phía sau, vẽ trọn vùng trời rồi xếp z_order thấp) vì đây là cách nhanh và giảm rủi ro sót vùng. Câu hỏi cho coach: **trong panoptic, `sky` có được vẽ là vùng bao trọn phía sau các vật thể (kể cả phần bị cây che không?), hay bắt buộc phải cắt theo đường viền thật của tán cây/tòa nhà?** |
| 3 | `hard_panoptic` / `000000460147.jpg` — **ảnh thứ hai của task, tôi chưa kịp vẽ gì cả** (0 shape). Cách hiểu A: để trống và ghi rõ trong report (tôi đang làm vậy) vì ảnh chủ yếu là cảnh đường tối, đèn đường và hàng cây, tôi không chắc chọn đúng ranh giới `road`/`sidewalk`/`vegetation`. Cách hiểu B: cứ vẽ `road` + `sky` + `building` sơ lược để "có điểm" cho ảnh này. | Ảnh 640×424 này rất tối (vùng dưới trung bình RGB ≈ 20/255), sau khi tăng gamma mới thấy được: nền trời sáng ở góc trên-trái, một dãy cây bên trái, mặt đường ở giữa, tòa nhà có cửa sổ ở bên phải, và vài xe đèn trước. Tôi **không dám chắc** chỗ nào là `sidewalk` và chỗ nào là `road`, và không có `car`/`person` nào tôi xác định được chắc chắn. | Tôi chọn **cách hiểu A**: để ảnh này **không có annotation** thay vì vẽ sai hàng loạt vùng, vì trong `hard_panoptic` mask sai chắc chắn bị tính là FP và còn làm giảm PQ của những lớp đó. Câu hỏi cho coach: **nếu không kịp vẽ 1 trong 2 ảnh thì có nên nộp ảnh đó với 0 annotation (như tôi đang làm), hay bỏ hẳn ảnh đó khỏi file COCO?** Tôi hiện để ảnh trong `images` của file COCO (đúng tên ảnh) nhưng không có annotation nào. |

Ghi chú thêm cho các task chưa làm: tôi sẽ bắt đầu `cp1_holes`… bằng đúng nguyên tắc đã rút ra ở trên — dùng Brush cho vùng biên lớn, **Polygon cho mọi object** (để COCO 1.0 mang được `segmentation`), và **kiểm tra số object trong danh sách Objects sau khi Save** trước khi export, vì với COCO thì số dòng `annotations` trong file xuất là số mask đã nộp chứ không phải số object đúng.
