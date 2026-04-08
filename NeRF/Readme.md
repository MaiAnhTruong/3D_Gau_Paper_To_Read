## Tài liệu tham khảo chính

- Blog:
  https://viblo.asia/p/tim-hieu-ve-giai-phap-digital-humans-phan-1-nerf-mo-hinh-tai-tao-cac-canh-3d-bang-mang-no-ron-dua-tren-truong-buc-xa-zXRJ8Ol54Gq

- Paper gốc:
  https://arxiv.org/pdf/2003.08934

## Lưu ý

Đây là nội dung mà anh em cần nắm chắc nhất, đây là base của base của base. Nó là xi măng đấy. Phải hiểu rõ từng công thức toán, thuật toán. Hãy hỏi AI để lấy ví dụ tính toán chi tiết để trực quan hơn.

# NeRF - Neural Radiance Fields 

Dùng các ảnh đầu vào cùng pose camera đã biết để học một **biểu diễn liên tục của cảnh 3D**, từ đó có thể **render các góc nhìn mới** với chất lượng rất cao.
Hiểu đơn giản là nó kiểu dạng 3D reconstruction ấy. Và ví dụ trong không gian 3D, bạn nhìn ở 1 góc độ mới bạn cũng biết nó như thế nào. (Mặc dù ảnh không có ở góc độ đó). 

NeRF là một phương pháp biểu diễn ngầm một cảnh 3D bằng mạng neural.

Thay vì lưu cảnh dưới dạng:
- point cloud rời rạc
- voxel grid
- mesh explicit

NeRF biểu diễn cảnh như một **hàm liên tục 5D**:

- đầu vào:
  - vị trí không gian `x = (x, y, z)`
  - hướng nhìn `d`
- đầu ra:
  - màu `c = (r, g, b)`
  - mật độ thể tích `σ`

`F_Θ : (x, d) -> (c, σ)`

Ý tưởng: cảnh 3D không còn được lưu trực tiếp bằng hình học explicit, mà được **mã hóa ngầm trong tham số của MLP**.

## NeRF thường cần:

- nhiều ảnh của cùng một cảnh
- camera intrinsics / extrinsics hoặc ít nhất là camera poses đã biết

Các thông tin này thường được lấy từ **SfM / COLMAP** (Ta gọi đây là foundation model)

Vì vậy, pipeline:

1. dùng **SfM** để:
   - tìm correspondence
   - ước lượng camera pose
   - khôi phục sparse point cloud

2. dùng **NeRF** để:
   - nhận ảnh + pose
   - học neural radiance field
   - tổng hợp novel views

Nói cách khác:

- **SfM** giải quyết phần hình học camera ban đầu
- **NeRF** giải quyết phần biểu diễn liên tục và render góc nhìn mới


### Với một ảnh 2D, ta có thể tưởng tượng có một hàm:

`f(x, y) -> c`

Trong đó:
- `(x, y)` là tọa độ pixel
- `c` là màu pixel

Nếu học được hàm đó, ta có thể suy ra màu tại các vị trí khác.

### Mở rộng sang 3D

Trong 3D, mỗi pixel không chỉ tương ứng với một điểm duy nhất, mà tương ứng với một **tia camera** đi vào không gian.

Do đó:
- từ mỗi pixel, ta bắn ra một **ray**
- lấy mẫu nhiều điểm dọc theo ray
- tại mỗi điểm mẫu, mạng dự đoán:
  - màu
  - mật độ
- sau đó tổng hợp các điểm đó bằng **volume rendering**
- cuối cùng thu được màu của pixel

## Các thành phần

### 1. Camera rays

Mỗi pixel của ảnh tương ứng với một tia xuất phát từ tâm camera đi vào không gian 3D.

NeRF không dự đoán màu pixel trực tiếp theo kiểu ảnh 2D thông thường.  
Thay vào đó, NeRF dự đoán nội dung của cảnh **dọc theo ray** rồi tổng hợp lại.

### 2. Neural radiance field

NeRF mô hình hóa cảnh bằng một MLP nhận:
- vị trí 3D
- hướng nhìn

và trả ra:
- `σ`: mật độ tại điểm đó
- `c`: màu phát xạ theo hướng nhìn đó

Điểm quan trọng:

- **density** chủ yếu gắn với vị trí trong không gian
- **color** có thể phụ thuộc cả vị trí lẫn hướng nhìn

Chính điều này giúp NeRF biểu diễn được các hiệu ứng **view-dependent** như phản xạ hoặc highlight.

### 3. Volume rendering

Sau khi lấy mẫu nhiều điểm dọc theo ray, NeRF không chọn một điểm duy nhất.

Thay vào đó, nó dùng **volume rendering** để cộng dồn đóng góp của tất cả các mẫu trên ray.

Hiểu đơn giản:
- điểm nào có mật độ lớn hơn thì có khả năng đóng góp mạnh hơn
- điểm phía trước có thể che khuất điểm phía sau
- màu cuối cùng của pixel là tổng có trọng số của các mẫu trên ray

Đây là cầu nối từ:
- biểu diễn liên tục trong không gian 3D
- sang ảnh 2D có thể so sánh với ground truth

## Công thức cốt lõi của NeRF

Với một camera ray:

`r(t) = o + td`

trong đó:
- `o` là tâm camera
- `d` là hướng ray
- `t` là tham số dọc theo ray

NeRF định nghĩa màu kỳ vọng của ray là tích phân thể tích dọc theo ray.

Khi rời rạc hóa để tính toán thực tế, màu pixel được viết thành tổng có trọng số trên các mẫu dọc tia.  
Các thành phần quan trọng cần hiểu gồm:

- `σ_i`: mật độ tại mẫu thứ `i`
- `δ_i`: khoảng cách giữa hai mẫu liên tiếp
- `T_i`: transmittance, thể hiện xác suất tia đi đến mẫu `i` mà chưa bị chặn trước đó
- `w_i`: trọng số đóng góp của mẫu thứ `i`
- `c_i`: màu tại mẫu thứ `i`

Trực giác:
- `T_i` càng lớn thì tia càng “đi xuyên” được đến điểm đó
- `σ_i` càng lớn thì điểm đó càng có xu hướng “chặn” tia
- `w_i` cho biết mẫu đó đóng góp bao nhiêu vào màu cuối cùng

## 4. Positional encoding

Một vấn đề lớn là MLP thuần thường học kém các chi tiết tần số cao.

Nếu đưa trực tiếp tọa độ `(x, y, z)` và hướng nhìn `d` vào mạng, kết quả thường bị:
- quá mượt
- mất texture
- mất chi tiết hình học nhỏ

Vì vậy paper gốc dùng **positional encoding** để ánh xạ tọa độ sang không gian nhiều chiều hơn trước khi đưa vào MLP.

Ý tưởng là biến một tọa độ gốc thành nhiều thành phần sin / cos ở các tần số khác nhau, giúp mạng biểu diễn tốt hơn:
- chi tiết cao tần
- biên sắc nét
- texture phức tạp

Đây là một trong những lý do NeRF đạt chất lượng hình ảnh rất cao.

## 5. Hierarchical sampling

Nếu lấy mẫu quá dày trên mọi ray ở mọi nơi, chi phí tính toán sẽ rất lớn.

Nhưng thực tế:
- có nhiều vùng trống không đóng góp gì
- có nhiều vùng bị che khuất
- chỉ một số vùng thực sự chứa nội dung quan trọng

Vì vậy NeRF dùng **hierarchical sampling**:

- một mạng coarse lấy mẫu sơ bộ
- từ đó xác định vùng nào quan trọng hơn
- mạng fine tập trung nhiều mẫu hơn vào các vùng có khả năng chứa bề mặt / nội dung nhìn thấy

Cách làm này giúp:
- tăng hiệu quả lấy mẫu
- cải thiện chất lượng render
- giảm lãng phí tính toán vào vùng free space

## Huấn luyện NeRF

Quá trình huấn luyện có thể hiểu theo pipeline như sau:

1. lấy một ảnh huấn luyện
2. chọn một tập pixel trong ảnh đó
3. với mỗi pixel:
   - tạo camera ray
   - lấy mẫu nhiều điểm dọc ray
   - đưa từng điểm vào MLP để lấy `(c, σ)`
   - dùng volume rendering để tính ra màu dự đoán của pixel
4. so sánh màu dự đoán với màu ground truth
5. tính loss
6. backprop để cập nhật tham số mạng

Lặp lại đủ lâu, mạng sẽ học được cách biểu diễn cảnh 3D sao cho khi render từ các góc nhìn đã biết hoặc góc nhìn mới, ảnh sinh ra khớp với dữ liệu quan sát.


## Hạn chế quan trọng của NeRF

Dù rất mạnh, NeRF gốc có một số hạn chế lớn:

- huấn luyện chậm
- render chậm
- phải lấy mẫu rất nhiều điểm trên mỗi ray
- phụ thuộc mạnh vào chất lượng pose
- khi số ảnh ít (sparse-view), bài toán trở nên khó hơn rất nhiều

Đây cũng chính là một trong những động lực lớn dẫn tới:
- các biến thể NeRF nhanh hơn
- sparse-view NeRF
- 3D Gaussian Splatting
- các phương pháp dùng depth prior / foundation models / geometric priors