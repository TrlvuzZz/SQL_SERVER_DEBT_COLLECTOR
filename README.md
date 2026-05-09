# Bài tập Hệ quản trị cơ sở dữ liệu (TEE560)
## Thông tin sinh viên:
+ **Họ và tên:** Trần Lâm Vũ
+ **Lớp:** K59KMT.K01
+ **Mã số sinh viên:** K235510205299
+ **Trường:** Đại học Kỹ thuật Công nghiệp Thái Nguyên
---
### Nhiệm vụ 1: Thiết kế CSDL

+ Tạo database `CamDo59KMT_ThaiNguyen`
```sql
CREATE DATABASE CamDo59KMT_ThaiNguyen
GO
USE CamDo59KMT_ThaiNguyen
GO
```
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/d610a92e-1834-4d3a-a2dd-6552cb53ebbf" />

+ Tạo bảng khách hàng `NguoiCamDo`
```sql
CREATE TABLE NguoiCamDo
(
    MaNguoiCam INT IDENTITY(1,1) PRIMARY KEY,
    TenKhach NVARCHAR(100),
    SDT VARCHAR(15),
    DiaChi NVARCHAR(200),
    CCCD VARCHAR(20)
)
```
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/ff4ff910-ae19-498b-9041-0f7abdc50ea7" />

+  Tạo bảng nhân viên - tức người thu tiền `NguoiThuTien`
```sql
CREATE TABLE NguoiThuTien
(
    MaNguoiThu INT IDENTITY(1,1) PRIMARY KEY,
    TenNhanVien NVARCHAR(100),
    SoDienThoai VARCHAR(15)
)
```
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/46a3d2bc-1f6a-4901-ac9f-091c020e790d" />

+ Tạo bảng hợp đồng cầm đồ - `PhieuCamDo`
```sql
CREATE TABLE PhieuCamDo
(
    MaPhieuCam INT IDENTITY(1,1) PRIMARY KEY,
    MaNguoiCam INT,
    TienVayGoc DECIMAL(18,2),
    NgayCam DATE DEFAULT GETDATE(),
    HanLaiDon DATE,
    HanThanhLy DATE,
    TrangThaiPhieu NVARCHAR(50)
        DEFAULT N'Đang giữ đồ',
    TienConNo DECIMAL(18,2),
    FOREIGN KEY (MaNguoiCam)
        REFERENCES NguoiCamDo(MaNguoiCam)
)
```
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/e4e7e7f8-0a35-4f98-b1e4-0c3402478008" />

+ Tạo bảng đồ đang cầm, hiện vật, tiền mặt - `DoCam`
```sql
CREATE TABLE DoCam
(
    MaDoCam INT IDENTITY(1,1) PRIMARY KEY,
    MaPhieuCam INT,
    TenDo NVARCHAR(200),
    GiaTriUocTinh DECIMAL(18,2),
    TrangThaiDo NVARCHAR(50)
        DEFAULT N'Đang giữ',
    DaBanThanhLy BIT DEFAULT 0,
    FOREIGN KEY (MaPhieuCam)
        REFERENCES PhieuCamDo(MaPhieuCam)
)
```
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/80047271-7089-4e23-bd4b-0449de892f9d" />

+ Tạo bảng nhật ký dòng tiền nhằm kiểm soát tiền đã thu và tiền còn nợ - `NhatKyDongTien`
```sql
CREATE TABLE NhatKyDongTien
(
    MaGiaoDich INT IDENTITY(1,1) PRIMARY KEY,
    MaPhieuCam INT,
    MaNguoiThu INT,
    NgayDong DATETIME
        DEFAULT GETDATE(),
    SoTienKhachTra DECIMAL(18,2),
    SoTienNoConLai DECIMAL(18,2),
    FOREIGN KEY (MaPhieuCam)
        REFERENCES PhieuCamDo(MaPhieuCam),
    FOREIGN KEY (MaNguoiThu)
        REFERENCES NguoiThuTien(MaNguoiThu)
)
```
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/14cca449-4e98-4eac-aabd-6c08d50137ab" />



