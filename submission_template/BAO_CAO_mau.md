# Báo cáo lab: chọn tracker cho 5 video

**Nhóm:** DangQuocHiep-2A202602755 **Thành viên:** Đặng Quốc Hiệp (MSSV: 2A202602755)

Detector cố định: `yolo26n.pt`, ảnh 640 px, Re-ID `osnet_x0_25_msmt17`. Không đổi các mục này trong bài nộp chính.

## 1. Cấu hình đã chọn

Mỗi video: tracker bạn nộp, `conf`, `iou`, điều bạn **nhìn thấy** trên video, và một cấu hình đã thử rồi loại.

| Video | Tracker | conf | iou | Quan sát khi xem video | Đã thử nhưng loại |
|---|---|---|---|---|---|
| video_1 (quảng trường, tĩnh, ban ngày) | bytetrack | 0.15 | 0.5 | Quảng trường rộng, camera tĩnh, người đi lại ổn định. Hạ conf xuống 0.15 giúp phát hiện thêm nhiều người ở xa với kích thước nhỏ, hai tầng liên kết của ByteTrack giữ track mượt mà, số lần nhảy ID rất thấp (IDSW = 13). | bytetrack (conf=0.3, iou=0.5) bị bỏ sót nhiều người đi bộ ở xa khiến số lượng False Negative cao, HOTA chỉ đạt 26.912 và MOTA 17.292. |
| video_2 (phố đêm, tĩnh, rất đông) | botsort | 0.25 | 0.5 | Phố đi bộ ban đêm mật độ cực kỳ đông đúc, ánh sáng yếu, che khuất liên tục khi người đi giao cắt. BoT-SORT kết hợp trích xuất đặc trưng ngoại hình Re-ID (OSNet) giúp nhận diện lại đúng người sau khi bước ra từ điểm che khuất, hạn chế tối đa nhảy ID. | bytetrack (conf=0.3, iou=0.5) thuần IoU chuyển động nên khi hai người che khuất nhau hoặc đi cắt mặt thì ID bị hoán đổi hoặc reset thành ID mới. |
| video_3 (camera di động, ảnh nhỏ) | botsort | 0.3 | 0.45 | Camera di động lia góc, FPS thấp, người ở xa kích thước nhỏ. Cơ chế Camera Motion Compensation (CMC) trong BoT-SORT giúp bù trừ chuyển động của nền camera, kết hợp Re-ID giữ vết ổn định không bị trôi hộp khi máy quay lia nhanh. | bytetrack (conf=0.3, iou=0.5) thiếu bù chuyển động camera nên dự đoán Kalman Filter bị lệch tọa độ khi camera quay nhanh, dẫn tới mất dấu hoặc nhảy track. |
| video_4 (trong nhà, camera di chuyển) | botsort | 0.35 | 0.5 | Không gian trong nhà có nhiều vách kính phản chiếu, camera tiến tới làm kích thước người thay đổi nhanh. Nâng conf lên 0.35 loại bỏ triệt để các bóng mờ phản chiếu trên kính (false positives); Re-ID giúp duy trì danh tính tốt khi scale thay đổi. | bytetrack (conf=0.2, iou=0.5) bắt nhầm bóng người phản chiếu trên vách kính làm xuất hiện nhiều bounding box giả nhấp nháy liên tục. |
| video_5 (trên xe bus, giao lộ đông) | botsort | 0.3 | 0.5 | Góc nhìn từ xe bus rung lắc mạnh khi xe di chuyển qua ngã tư đông đúc. BoT-SORT triệt tiêu rung chấn nhờ CMC, đồng thời Re-ID giữ track bền bỉ khi người đi bộ bị các phương tiện cơ giới che khuất thoáng qua. | ocsort (conf=0.3, iou=0.5) tuy xử lý chuyển động phi tuyến tốt nhưng thiếu CMC bù rung giật xe bus và thiếu Re-ID nên dễ mất track khi bị xe cộ che khuất. |

## 2. Số liệu video_1

Dán bảng HOTA / MOTA / IDF1 do `scripts/evaluate_practice.py` in ra.

```
HOTA: nop_bai_v1-pedestrian        HOTA      DetA      AssA      DetRe     DetPr     AssRe     AssPr     LocA      OWTA      HOTA(0)   LocA(0)   HOTALocA(0)
video_1                            27.314    15.855    47.144    16.109    81.962    49.462    84.484    84.136    27.544    33.187    80.779    26.808    
COMBINED                           27.314    15.855    47.144    16.109    81.962    49.462    84.484    84.136    27.544    33.187    80.779    26.808    

CLEAR: nop_bai_v1-pedestrian       MOTA      MOTP      MODA      CLR_Re    CLR_Pr    MTR       PTR       MLR       sMOTA     CLR_TP    CLR_FN    CLR_FP    IDSW      MT        PT        ML        Frag      
video_1                            18.314    81.945    18.384    19.019    96.769    11.29     16.129    72.581    14.881    3534      15047     118       13        7         10        45        34        
COMBINED                           18.314    81.945    18.384    19.019    96.769    11.29     16.129    72.581    14.881    3534      15047     118       13        7         10        45        34        

Identity: nop_bai_v1-pedestrian    IDF1      IDR       IDP       IDTP      IDFN      IDFP      
video_1                            26.987    16.146    82.147    3000      15581     652       
COMBINED                           26.987    16.146    82.147    3000      15581     652       

Count: nop_bai_v1-pedestrian       Dets      GT_Dets   IDs       GT_IDs    
video_1                            3652      18581     35        62        
COMBINED                           3652      18581     35        62        
```

`video_2` đến `video_5` không có nhãn trong gói lab. Không điền số cho các video đó.

## 3. Phân tích

Với **ít nhất hai video** (nên gồm một video bạn chỉ đánh giá bằng mắt), viết 3–5 câu:

- **Phân tích video_1 (Quảng trường, camera tĩnh, ban ngày):**
  Trong cảnh này, camera hoàn toàn cố định và mật độ người đi bộ ở mức vừa phải, các quỹ đạo chuyển động diễn ra trơn tru theo đường thẳng hoặc cong nhẹ. Thuật toán **ByteTrack** phát huy tối đa thế mạnh của cơ chế liên kết hai tầng (two-stage association): hộp có độ tin cậy thấp (low-score detection) vẫn được tận dụng để ghép nối với tracklet đang có thay vì bị vứt bỏ, giúp tăng độ bao phủ (DetRe tăng lên 16.109%) mà độ chính xác vẫn rất cao (Precision đạt tới 96.77%). Do không có nhiều hiện tượng che khuất dài hạn hay chuyển động máy quay phức tạp, việc chỉ dùng Kalman Filter kết hợp IoU giúp thuật toán đạt tốc độ xử lý nhanh vượt trội, số lần đổi danh tính rất thấp (chỉ 13 lần ID switch trên toàn bộ chuỗi), đạt chỉ số HOTA cao nhất (27.314) so với các cấu hình khác.

- **Phân tích video_2 (Phố đêm, camera tĩnh trên cao, mật độ rất đông):**
  Khác với video 1, video 2 là thử thách lớn về hiện tượng che khuất (occlusion) liên tục và chồng chéo giữa các cá nhân trong điều kiện ánh sáng đêm phức tạp. Nếu chỉ sử dụng tracker dựa trên chuyển động thuần như ByteTrack, các hộp dự đoán bị chồng lấn liên tục khiến Hungarian matching gán nhầm ID qua lại hoặc làm gián đoạn quỹ đạo sinh ra vô số ID mới. Khi chuyển sang **BoT-SORT** kết hợp mô hình trích xuất đặc trưng ngoại hình `osnet_x0_25_msmt17`, tracker duy trì một vector embedding cho mỗi người; khi hai người giao cắt và tách ra, độ tương đồng cosine của vector ngoại hình giúp khôi phục chính xác ID ban đầu. Nhờ đó, trên video preview ta quan sát thấy các vệt ID có màu sắc ổn định, không bị hiện tượng nhảy màu nhấp nháy khi người đi bộ lướt qua nhau.

- **Phân tích bổ sung video_5 (Trên xe bus, giao lộ đông, rung lắc mạnh):**
  Ở cảnh quay từ trên xe bus, rung lắc cơ học của xe tạo ra chuyển động toàn cảnh đột ngột (egomotion), khiến giả định vận tốc tuyến tính của Kalman Filter thông thường bị phá vỡ hoàn toàn. **BoT-SORT** vượt trội nhờ module bù chuyển động camera (Camera Motion Compensation - CMC) sử dụng khớp đặc trưng toàn cục để ước lượng ma trận affine biến đổi giữa các khung hình, đưa tọa độ về hệ quy chiếu chuẩn trước khi dự đoán. Kết hợp cùng Re-ID, BoT-SORT giữ vững bám đuổi ngay cả khi xe bus xóc nảy hoặc rẽ góc ở ngã tư đông đúc.

## 4. Nếu có thêm thời gian

- Thử nghiệm các mô hình Re-ID có năng lực biểu diễn mạnh mẽ hơn như OSNet kích thước đầy đủ (`osnet_ibn_x1_0_msmt17`) hoặc FastReID/CLIP-ReID để phân biệt người tốt hơn nữa trong điều kiện đêm tối, giảm tỷ lệ gán nhầm khi trang phục tương đồng.
- Fine-tune bộ detector YOLO26n với ảnh độ phân giải cao hơn hoặc thêm dữ liệu augmentation chuyên biệt cho người ở xa và người bị che khuất một phần để nâng cao độ nhạy phát hiện (Detection Recall).
- Quét tham số chi tiết theo dạng Grid Search cho các ngưỡng nội tại của tracker như `track_high_thresh`, `track_buffer` (tăng số frame lưu nhớ track khi bị che khuất lâu) và `appearance_thresh` để tối ưu hóa sự cân bằng giữa tính năng chuyển động và ngoại hình.

