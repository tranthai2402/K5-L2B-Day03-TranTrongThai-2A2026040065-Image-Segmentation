# Báo cáo thực hành Day 3 — điền trực tiếp trong fork của bạn

- Mã học viên theo lớp: **26A202604065**
- Ngày / CVAT local: **11/11/2026**, CVAT local (task `easy_task`, tạo 10:26 ngày 11/11 theo giờ UTC)
- Công cụ đã dùng: **CVAT local** với công cụ **Brush + Polygon vẽ tay**, không dùng SAM và không dùng Intelligent Scissors; dùng thêm `scripts/inspect_submissions.py` của repo và Python (Pillow + NumPy) để tự đo coverage / chồng lấn từ file export.

## 1. Bài đã nộp

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | `submissions/easy_semantic.zip` | 3 / 3 | 20 |
| medium_instance | chưa có | 0 / 3 | 32 |
| hard_panoptic | chưa có | 0 / 2 | 30 |
| cp1_holes | chưa có | 0 / 1 | 3 |
| cp2_slice | chưa có | 0 / 1 | 3 |
| cp5_occlusion | chưa có | 0 / 1 | 3 |
| cp3_thin | chưa có | 0 / 1 | 3 |
| cp4_curb | chưa có | 0 / 1 | 3 |
| cp6_coverage | chưa có | 0 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

Ghi chú trạng thái:

- `easy_semantic` đã vẽ đủ **3/3 ảnh**: `7ee6d192-89e2408b.jpg`, `817bca71-00000000.jpg`, `81ae7cbb-6bc63a4a.jpg`. Nhãn dùng đúng 5 lớp trong `data/tiers/easy_semantic/classes.json`: `road`, `sidewalk`, `building`, `vegetation`, `sky` (dán nguyên mảng JSON trong `cvat-labels.json` vào ô **Labels → Raw**).
- Trong 3 ảnh đã vẽ có **10 mask + 3 polygon** (sky vẽ bằng polygon vì biên trên gần, brush không ôm được đường chân trời).
- **Cảnh báo về định dạng export:** bản `easy_semantic.zip` đang nằm trong `submissions/` là bản export kiểu **CVAT for images 1.1** (chỉ chứa `annotations.xml`), nên `inspect_submissions.py` báo `[LỖI]`: không có PNG trong `SegmentationClass/` và thiếu `labelmap.txt`. Tôi đã export lại đúng **Segmentation mask 1.1** ở CVAT local và đổi tên thành `easy_semantic.zip`; **việc copy bản Segmentation mask 1.1 đó vào `submissions/` trên máy này là việc còn lại** (bản đang có trong `submissions/` vẫn là `annotations.xml`), nên trạng thái hiện tại là **đã sửa ở CVAT, chưa đồng bộ xong trên repo**.
- Chưa có nộp cho các task còn lại, nên không tạo ZIP rỗng cho những task đó.
- Chưa xem Summary tự đánh giá trên GitHub Actions (reference chưa được phát) → **chưa có điểm**.

## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: **chưa có** — tôi mới hoàn thành xong `easy_semantic` và **chưa bắt đầu `medium_instance`**, nên không có object Medium nào để mô tả. Tôi ghi lại một quyết định gán nhãn đã thực hiện ở `easy_semantic` thay cho phần này, theo đúng tinh thần câu hỏi (quyết định trước, không dùng gợi ý máy nào).
- Quyết định: ở ảnh `817bca71-00000000.jpg`, tôi vẽ **sky bằng Polygon** chạy theo đường chân trời (biên dưới từ khung phải, xuống đường chân trời ở giữa, rồi về khung trái), thay vì dùng Brush.
- Quy tắc tôi dùng để chọn biên: vùng `sky` là vùng **liên tục trên đường chân trời**, biên dưới của nó phải là đường chân trời thực tế (nơi hàng cây/tòa nhà chạm trời), không phải một đường thẳng cắt ngang ảnh. Vì vậy tôi dùng Polygon cho vùng này (biên có nhiều điểm gấp khúc ở chỗ cây nhô lên), còn `road`, `sidewalk`, `building`, `vegetation` vẽ bằng Brush vì biên của chúng là đường cong lớn, ít gấp khúc.
- Nếu dùng gợi ý sau đó: **không dùng** (không có SAM, không có Intelligent Scissors trên CVAT local của tôi). Việc kiểm biên sau khi vẽ là tự đối chiếu trên ảnh gốc bằng mắt, rồi dùng script đo coverage/chồng lấn.
- Một quyết định khác: ở ảnh `7ee6d192-89e2408b.jpg` không có `sidewalk` và `building` trong khung hình, tôi **không tạo mask nào cho hai lớp đó** thay vì tô bừa cho đủ 5 lớp — thiếu lớp thì thiếu, tô bừa thì sai cả vùng.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: **task `easy_semantic`, toàn bộ 3 ảnh**, phần **định dạng file nộp** (lỗi ở mức task chứ không phải một vùng ảnh cụ thể).
- Lỗi thuộc loại: **khác** — export sai format, không phải sai lớp / thiếu-thừa vật / gộp-tách / biên / phủ vùng.
- Bằng chứng tôi nhìn thấy: chạy `python scripts/inspect_submissions.py --dir submissions` in ra
  `[LỖI] easy_semantic: easy_semantic.zip` với 3 dòng cảnh báo: *"không có PNG trong SegmentationClass/; kiểm format Segmentation mask 1.1"*, *"thiếu mask ảnh: 7ee6d192-89e2408b.jpg, 817bca71-00000000.jpg, 81ae7cbb-6bc63a4a.jpg"*, *"cần đúng một labelmap.txt"*. Tôi mở ZIP ra kiểm tra thì nội dung chỉ có một file `annotations.xml` — đúng định dạng CVAT for images 1.1, không có thư mục `SegmentationClass/` và không có `labelmap.txt` như hợp đồng của `docs/SUBMISSION.md` yêu cầu cho task semantic.
- Quy tắc và hành động sửa: task semantic bắt buộc export **Segmentation mask 1.1** (task instance/panoptic mới dùng COCO 1.0). Tôi đã quay lại CVAT local, mở Job `easy_task`, vào **Job menu → Export job dataset** và chọn lại đúng format `Segmentation mask 1.1`, sau đó đổi tên file kết quả thành `easy_semantic.zip`. Việc dán nhầm `classes.json` (metadata của bài lab) vào ô Raw cũng đã tránh: tôi dán mảng JSON trong `cvat-labels.json`.
- Sau sửa đã Save và export lại chưa: **đã Save và đã export lại đúng format ở CVAT local**; **còn 1 việc chưa xong là copy bản `Segmentation mask 1.1` đó vào thư mục `submissions/` trên máy này** — bản đang nằm trong `submissions/` vẫn là `annotations.xml`, nên chạy lại script vẫn báo `[LỖI]`. Tôi sẽ copy và chạy lại `inspect_submissions.py` để xác nhận `[OK]` trước khi push lên fork.
- Kết quả tự đánh giá liên quan: **chưa có điểm** (reference chưa được phát, GitHub Actions chưa có Summary). Bản thay bằng chứng của tôi là output của `inspect_submissions.py` trước/sau, không phải điểm số.

  Một lỗi thứ hai tôi phát hiện khi tự đo bằng script (đang để ở mục 4 vì chưa sửa xong): **chồng lấn giữa các lớp semantic** — ở `817bca71-00000000.jpg`, `sky ∩ vegetation` = 129 643 px (14,1% ảnh) và `building ∩ sky` = 123 656 px (13,4% ảnh); ở `81ae7cbb-6bc63a4a.jpg`, `sky ∩ vegetation` = 132 076 px (14,3%) và `building ∩ sky` = 67 388 px (7,3%). Semantic chỉ được có 1 lớp cho mỗi pixel nên đây là lỗi cần sửa bằng cách sửa lại biên (không phải sửa bằng cách tô đè).

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| 1 | `817bca71-00000000.jpg` — vùng cây ở nửa trên ảnh (x 0–1279, y 0–579). Cách hiểu A: cây là `vegetation` cho tới hết chiều cao cây, phần trên đó là `sky`. Cách hiểu B: `sky` là nền sau cây nên cây vẫn là vegetation nhưng biên trên của vegetation phải ôm theo mép lá, không được tràn lên vùng trời. | Đo được `sky ∩ vegetation` = 129 643 px (14,1% ảnh) → hai mask đang chồng lên nhau ở tán cây. Biên trên của vùng trời phải đi theo đường viền của tán cây. | Tôi chưa sửa xong. Định sửa bằng cách cắt lại biên `vegetation` theo đường viền tán cây (giữ `sky` là nền, `vegetation` chỉ chiếm đúng phần lá). Nhờ coach xác nhận: trong semantic của lớp này, **cây ở giữa ảnh có được tính là `vegetation` không**, hay là `sky` phủ lên cây (vì `sky` ở đây là "vùng trời" theo nghĩa pixel)? |
| 2 | `817bca71-00000000.jpg` và `81ae7cbb-6bc63a4a.jpg` — vùng "chân trời" phía xa (trong ảnh 817bca71: `building` x[0..1279] y[131..481] hòa vào `sky`; trong ảnh 81ae7cbb: `building` x[0..1279] y[263..471]). Cách hiểu A: `building` chỉ chiếm phần tường nhìn thấy, phần trên là `sky`. Cách hiểu B: `building` bao trọn cả khối (bao gồm phần "mất" sau cây/rõ mây), vì đây là vùng liên tục. | `building ∩ sky` lần lượt là 123 656 px (13,4% ảnh) và 67 388 px (7,3% ảnh) → mask building của tôi đang vẽ cả lên vùng trời ở đường chân trời. | Tôi chưa sửa xong, sẽ cắt lại `building` theo đường chân trời để không lấn sang `sky`. Câu hỏi cho coach: **một tòa nhà bị cây che một phần thì phần "không nhìn thấy" có được vẽ vào mask building không, hay chỉ vẽ phần nhìn thấy?** |
| 3 | `7ee6d192-89e2408b.jpg` — dải trống giữa `vegetation` (kết thúc ở y≈443) và `road` (bắt đầu ở y≈361), tức vùng chưa gán nhãn nằm ở **y 246–509**, chiếm **14,3% ảnh** (131 648 px). Cách hiểu A: vùng này là phần `road` xa (mặt đường chạy tới chân trời) nên phải gộp vào road. Cách hiểu B: vùng này là `sidewalk`/`building` ở giữa ảnh, tôi đã bỏ sót nên không có lớp nào phủ. | Trong ảnh này tôi không tạo mask `sidewalk` và `building` nào. Biên dưới của `vegetation` (y≈443) và biên trên của `road` (y≈361) không khớp nhau nên để hở một dải. | Tôi đã dừng ở đây để hỏi coach thay vì tô bừa: **dải trống giữa hàng cây và mặt đường trong ảnh này thuộc lớp nào** (`road` xa, hay `sidewalk`), và trong `easy_semantic` có bắt buộc phủ kín 100% pixel không (task `cp6_coverage` mới là trạm kiểm coverage, nhưng rubric của `easy_semantic` có ghi "phủ đúng vùng")? |

Ghi chú thêm cho các task chưa làm: tôi sẽ bắt đầu `medium_instance` với nguyên tắc đã rút ra ở `easy_semantic` — dùng Brush cho vùng biên lớn, Polygon cho vùng có biên gấp khúc, và **kiểm tra số object trong danh sách Objects sau khi Save** trước khi export COCO 1.0, vì với COCO thì số dòng `annotations` trong file xuất là số mask đã nộp chứ không phải số object đúng.
