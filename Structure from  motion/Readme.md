## Tài liệu tham khảo chính

- CMSC426 – Structure from Motion:  
  https://cmsc426.github.io/sfm/

# Structure from Motion

**Structure from Motion (SfM)** — Khôi phục **cấu trúc 3D của cảnh** và **pose của camera** từ nhiều ảnh 2D chụp ở các góc nhìn khác nhau.

Từ các ảnh đầu vào, thuật toán sẽ tìm các đặc trưng tương ứng giữa các ảnh, suy ra quan hệ hình học giữa các camera, ước lượng pose, tam giác hóa các điểm 3D và cuối cùng tinh chỉnh toàn bộ nghiệm để tạo ra một **sparse** 3D reconstruction.

- **Input:** nhiều ảnh 2D của cùng một cảnh
- **Output:**  
  - camera intrinsics / extrinsics
  - sparse point cloud 3D  
  - quan hệ hình học giữa các ảnh / camera

## SfM là nền tảng của:

- 3D reconstruction
- visual localization / SLAM
- NeRF
- 3D Gaussian Splatting
- sparse-view scene understanding

Hiện nay, **COLMAP/SfM** thường là bước khởi tạo để tạo:
- camera poses
- sparse point cloud
- tín hiệu hình học ban đầu cho các mô hình downstream

## Các thành phần

1. **Feature matching**  
   Tìm các điểm tương ứng giữa nhiều ảnh.

2. **Fundamental matrix & Epipolar geometry**  
   Mô tả ràng buộc hình học giữa hai ảnh.

3. **RANSAC**  
   Loại bỏ outlier trong quá trình matching.

4. **Essential matrix**  
   Suy ra quan hệ pose tương đối giữa hai camera khi đã biết intrinsics.

5. **Camera pose estimation**  
   Ước lượng rotation và translation của camera.

6. **Triangulation**  
   Từ các correspondence và pose, khôi phục điểm 3D.

7. **PnP (Perspective-n-Point)**  
   Ước lượng pose từ các điểm 3D–2D correspondence.

8. **Bundle Adjustment**  
   Tối ưu jointly camera poses và 3D points để giảm reprojection error.

## Structure from Motion được dùng làm nền tảng để hiểu:

- 3DGS thường phụ thuộc vào point cloud khởi tạo
- Chất lượng pose / point cloud ảnh hưởng mạnh đến kết quả render
- Sparse-view là bài toán khó khi SfM bị thiếu ràng buộc hình học
- khi nào cần thay thế / tăng cường SfM bằng foundation models, depth priors hoặc multi-view priors