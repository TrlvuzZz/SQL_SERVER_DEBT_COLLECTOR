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

+ Add một vài khách hàng có nhu cầu cầm đồ
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/df0464cb-ad4a-4479-b3c9-30ab5bcb388a" />

### Nhiệm vụ 2: Cài đặt SQL (Yêu cầu viết Scripts)
#### Event 1: Đăng ký hợp đồng mới (Vay tiền)
Viết Store Procedure tiếp nhận hợp đồng: Lưu thông tin khách hàng, danh sách tài sản 
(kèm giá trị định giá), số tiền vay gốc và thiết lập 2 mốc Deadline1, Deadline2.

+Tạo `sp_TaoHopDong` - lưu thông tin khách hàng, danh sách tài sản (kèm định giá), số tiền vay gốc và thiết lập 2 mốc Deadline1, Deadline2.
```sql
CREATE PROC sp_TaoHopDong
(
    @MaNguoiCam INT,
    @TienVayGoc DECIMAL(18,2),
    @HanLaiDon DATE,
    @HanThanhLy DATE,
    @TenDo NVARCHAR(200),
    @GiaTriUocTinh DECIMAL(18,2)
)
AS
BEGIN
    -- Tạo phiếu cầm đồ
    INSERT INTO PhieuCamDo
    (
        MaNguoiCam,
        TienVayGoc,
        HanLaiDon,
        HanThanhLy,
        TrangThaiPhieu,
        TienConNo
    )
    VALUES
    (
        @MaNguoiCam,
        @TienVayGoc,
        @HanLaiDon,
        @HanThanhLy,
        N'Đang giữ đồ',
        @TienVayGoc
    )
    -- Lấy mã phiếu vừa tạo
    DECLARE @MaPhieuCamMoi INT
    SET @MaPhieuCamMoi = SCOPE_IDENTITY()
    -- Thêm tài sản cầm cố
    INSERT INTO DoCam
    (
        MaPhieuCam,
        TenDo,
        GiaTriUocTinh,
        TrangThaiDo
    )
    VALUES
    (
        @MaPhieuCamMoi,
        @TenDo,
        @GiaTriUocTinh,
        N'Đang giữ'
    )
    PRINT N'Tạo hợp đồng thành công'
END
GO
```
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/2e4e67c0-6bfd-407c-b596-c75f2be3861f" />

+ Kiểm tra Hợp đồng + Đồ cầm
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/83e86b91-8f35-490e-9c38-78621789f60c" />

#### Event 2: Tính toán công nợ thời gian thực 
1, Viết một Function fn_CalcMoneyTransaction(TransactionID, TargetDate) để tính số tiền 
phải trả của TransactionID này cho đến ngày TargetDate
```sql
CREATE FUNCTION fn_CalcMoneyContract
(
    @ContractID INT,
    @TargetDate DATE
)
RETURNS DECIMAL(18,2)
AS
BEGIN
    DECLARE
        @TienGoc DECIMAL(18,2),
        @NgayCam DATE,
        @Deadline1 DATE,
        @SoNgay INT,
        @SoNgayQuaHan INT,
        @LaiDon DECIMAL(18,2),
        @TongNo DECIMAL(18,2)
    SELECT
        @TienGoc = TienVayGoc,
        @NgayCam = NgayCam,
        @Deadline1 = HanLaiDon
    FROM PhieuCamDo
    WHERE MaPhieuCam = @ContractID
    -- Nếu chưa quá hạn lãi đơn
    IF @TargetDate <= @Deadline1
    BEGIN
        SET @SoNgay =
            DATEDIFF
            (
                DAY,
                @NgayCam,
                @TargetDate
            )
        -- Lãi đơn:
        -- 5.000 / 1.000.000 / ngày
        -- = 0.005
        SET @LaiDon =
            @TienGoc * 0.005 * @SoNgay
        SET @TongNo =
            @TienGoc + @LaiDon
    END
    -- Nếu đã quá Deadline1
    ELSE
    BEGIN
        -- Tính lãi đơn trước
        SET @SoNgay =
            DATEDIFF
            (
                DAY,
                @NgayCam,
                @Deadline1
            )
        SET @LaiDon =
            @TienGoc * 0.005 * @SoNgay
        -- Số ngày quá hạn
        SET @SoNgayQuaHan =
            DATEDIFF
            (
                DAY,
                @Deadline1,
                @TargetDate
            )
        -- Lãi kép
        SET @TongNo =
            (@TienGoc + @LaiDon)
            * POWER(1.005, @SoNgayQuaHan)
    END
    RETURN @TongNo
END
GO
```

+ Test function
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/340856c8-5a5c-484a-83ff-c334b0c9e4a5" />

2, Viết một Function fn_CalcMoneyContract(ContractID, TargetDate) để tính tổng số tiền 
khách(ContractID) phải trả (Gốc + Lãi đơn + Lãi kép) tính đến ngày TargetDate.
```sql
CREATE FUNCTION fn_CalcMoneyTransaction
(
    @TransactionID INT,
    @TargetDate DATE
)
RETURNS DECIMAL(18,2)
AS
BEGIN
    DECLARE @ContractID INT
    DECLARE @TongNo DECIMAL(18,2)
    SELECT
        @ContractID = MaPhieuCam
    FROM NhatKyDongTien
    WHERE MaGiaoDich = @TransactionID
    SET @TongNo =
        dbo.fn_CalcMoneyContract
        (
            @ContractID,
            @TargetDate
        )
    RETURN @TongNo
END
GO
```
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/a269d7e5-f86f-4f22-8935-7731f637bcfc" />

+ Test function
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/bc88b577-a61d-4064-bf87-6a1fd125f7e6" />







