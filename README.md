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

#### Event 3: Xử lý trả nợ và hoàn trả tài sản 
Viết Viết Store Procedure xử lý khi khách mang tiền đến: 
Nếu tài sản đã bị thanh lý (sau Deadline 2 và có cờ IsSold): Thông báo không thu tiền, 
không trả đồ. 
Nếu tài sản chưa bị thanh lý: Tính tổng nợ, trừ số tiền khách trả vào hệ thống. Nếu trả hết 
tiền, trả hết đồ và cập nhật trạng thái hợp đồng thành “Đã thanh toán đủ”; Nếu chưa trả 
hết tiền gốc+lãi: cập nhật trạng thái hợp đồng thành “Đang trả góp”, ghi nhận vào LOG số 
tiền đã trả, và số tiền còn nợ. 
Đưa ra danh sách gợi ý trả lại cho khách hàng này dựa trên điều kiện:  
Giá trị tài sản còn lại >= Dư nợ còn lại. 
```sql
CREATE PROC sp_ThanhToanHopDong
(
    @MaPhieuCam INT,
    @SoTienTra DECIMAL(18,2)
)
AS
BEGIN
    DECLARE
        @TongNo DECIMAL(18,2),
        @ConNo DECIMAL(18,2)
    -- Kiểm tra tài sản đã thanh lý chưa
    IF EXISTS
    (
        SELECT *
        FROM DoCam
        WHERE
            MaPhieuCam = @MaPhieuCam
            AND DaBanThanhLy = 1
    )
    BEGIN
        PRINT N'Tài sản đã bị thanh lý, không thể thanh toán'
        RETURN
    END
    -- Tính tổng nợ hiện tại
    SET @TongNo =
        dbo.fn_CalcMoneyContract
        (
            @MaPhieuCam,
            GETDATE()
        )
    -- Tính số nợ còn lại
    SET @ConNo =
        @TongNo - @SoTienTra
    -- Ghi log thanh toán
    INSERT INTO NhatKyDongTien
    (
        MaPhieuCam,
        SoTienKhachTra,
        SoTienNoConLai
    )
    VALUES
    (
        @MaPhieuCam,
        @SoTienTra,
        @ConNo
    )
    -- Cập nhật số nợ còn lại
    UPDATE PhieuCamDo
    SET TienConNo = @ConNo
    WHERE MaPhieuCam = @MaPhieuCam
    -- Nếu trả hết nợ
    IF @ConNo <= 0
    BEGIN
        -- Cập nhật trạng thái hợp đồng
        UPDATE PhieuCamDo
        SET TrangThaiPhieu = N'Đã thanh toán đủ'
        WHERE MaPhieuCam = @MaPhieuCam
        -- Trả tài sản cho khách
        UPDATE DoCam
        SET TrangThaiDo = N'Đã trả khách'
        WHERE MaPhieuCam = @MaPhieuCam
        PRINT N'Khách đã thanh toán đủ'
    END
    -- Nếu chưa trả hết
    ELSE
    BEGIN
        UPDATE PhieuCamDo
        SET TrangThaiPhieu = N'Đang trả góp'
        WHERE MaPhieuCam = @MaPhieuCam
        PRINT N'Khách chưa thanh toán hết'
    END
    -- Gợi ý tài sản có thể trả
    SELECT
        TenDo,
        GiaTriUocTinh
    FROM DoCam
    WHERE
        MaPhieuCam = @MaPhieuCam
        AND GiaTriUocTinh >= @ConNo
END
GO
```
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/23d10e19-f2e1-4461-9300-8a9e94e5431b" />

+ Test procedure
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/a2c2747b-ba82-4ba6-b7f4-fcbf32c60628" />

+ Xem log thanh toán
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/7226decd-65cc-4b8f-b552-307ab1027b29" />

+ Xem hợp đồng
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/3b32834e-db68-481a-9f02-3205d9f5b78e" />

+ Xem tài sản
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/e4b78741-297a-4a06-8b36-92e1605b7629" />

#### Event 4: Truy vấn danh sách nợ xấu (Nợ khó đòi) 
Xuất danh sách các khách hàng đã quá Deadline 1 mà chưa thanh toán. 
Yêu cầu các cột: Tên KH, Số điện thoại, Số tiền vay gốc, Số ngày quá hạn, Tổng tiền phải 
trả hiện tại (đến ngày hiện tại), Tổng số tiền phải trả sau 1 tháng nữa. 
```sql
CREATE VIEW vw_DanhSachNoXau
AS
SELECT
    nc.TenKhach,
    nc.SDT,
    pc.TienVayGoc,
    DATEDIFF
    (
        DAY,
        pc.HanLaiDon,
        GETDATE()
    ) AS SoNgayQuaHan,
    dbo.fn_CalcMoneyContract
    (
        pc.MaPhieuCam,
        GETDATE()
    ) AS TongTienHienTai,
    dbo.fn_CalcMoneyContract
    (
        pc.MaPhieuCam,
        DATEADD(MONTH, 1, GETDATE())
    ) AS TongTienSau1Thang
FROM PhieuCamDo pc
JOIN NguoiCamDo nc
ON pc.MaNguoiCam = nc.MaNguoiCam
WHERE
    GETDATE() > pc.HanLaiDon
    AND pc.TrangThaiPhieu
        <> N'Đã thanh toán đủ'
GO
```
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/07c53902-18b1-4778-a260-e57b23a4b1cf" />

+ Xem danh sách nợ xấu
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/9af9a407-22d8-4628-bad5-f4f3379e2524" />

#### Event 5: Quản lý thanh lý tài sản 
1, Viết một Trigger tự động chuyển trạng thái hợp đồng sang "Quá hạn (nợ xấu)" sau khi hợp 
đồng đang ở trạng thái "Đang vay" mà ngày vượt quá Deadline 1. 
```sql
CREATE TRIGGER trg_ChuyenQuaHan
ON PhieuCamDo
AFTER INSERT, UPDATE
AS
BEGIN
    UPDATE PhieuCamDo
    SET TrangThaiPhieu = N'Quá hạn'
    WHERE
        GETDATE() > HanLaiDon
        AND TrangThaiPhieu = N'Đang vay'
END
GO
```
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/fe65f3b2-82f0-4f5d-81e7-369aab32e816" />

+ Test Trigger
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/5b7d134d-bbe8-4162-9348-0877c4e5c72c" />

2, Viết một Trigger tự động chuyển trạng thái tài sản sang "Sẵn sàng thanh lý" sau khi hợp 
đồng đang ở trạng thái "Quá hạn (nợ xấu)" mà ngày vượt quá Deadline 2. 
```sql
CREATE TRIGGER trg_SanSangThanhLy
ON PhieuCamDo
AFTER INSERT, UPDATE
AS
BEGIN
    UPDATE DoCam
    SET TrangThaiDo = N'Sẵn sàng thanh lý'
    WHERE MaPhieuCam IN
    (
        SELECT MaPhieuCam
        FROM PhieuCamDo
        WHERE
            GETDATE() > HanThanhLy
            AND TrangThaiPhieu = N'Quá hạn'
    )
END
GO
```
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/c4fa6083-432b-432a-9c88-337491d660fd" />

+ Test Trigger
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/eafcecd3-6a34-4cf0-92a9-5e51aebf8270" />

3, Viết một Trigger tự động chuyển trạng thái tài sản thành “Đã bán thanh lý” sau khi trạng 
thái của hợp đồng chuyển sang "Đã thanh lý". 
Chú ý: Mỗi tài sản cũng được theo dõi trạng thái: đang cầm cố, đã trả khách, đã bán thanh lý 
```sql
CREATE TRIGGER trg_DaBanThanhLy
ON PhieuCamDo
AFTER UPDATE
AS
BEGIN
    UPDATE DoCam
    SET
        TrangThaiDo = N'Đã bán thanh lý',
        DaBanThanhLy = 1
    WHERE MaPhieuCam IN
    (
        SELECT MaPhieuCam
        FROM inserted
        WHERE TrangThaiPhieu = N'Đã thanh lý'
    )
END
GO
```
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/48c7fd95-0d86-4b57-843d-42ae587894fa" />

+ Test Trigger
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/28a876a2-9469-4a1d-8f6b-b4fb1ed12398" />

4, Kết quả
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/10ed1929-2f7f-4064-a85f-09615b317a76" />
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/5dc16226-4b88-432e-9576-268f255c994f" />

### Các sự kiện bổ sung:
+ Tạo `sp_GiaHanHopDong`
```sql
CREATE PROC sp_GiaHanHopDong
(
    @MaPhieuCam INT,
    @SoNgayGiaHan INT
)
AS
BEGIN
    DECLARE
        @TienLai DECIMAL(18,2),
        @TienConNo DECIMAL(18,2)
    -- Tính tiền lãi hiện tại
    SET @TienLai =
        dbo.fn_CalcMoneyContract
        (
            @MaPhieuCam,
            GETDATE()
        )
    SELECT
        @TienConNo = TienVayGoc
    FROM PhieuCamDo
    WHERE MaPhieuCam = @MaPhieuCam
    -- Chỉ lấy phần lãi
    SET @TienLai =
        @TienLai - @TienConNo
    -- Ghi log gia hạn
    INSERT INTO NhatKyDongTien
    (
        MaPhieuCam,
        SoTienKhachTra,
        SoTienNoConLai
    )
    VALUES
    (
        @MaPhieuCam,
        @TienLai,
        @TienConNo
    )
    -- Dời deadline
    UPDATE PhieuCamDo
    SET
        HanLaiDon =
            DATEADD
            (
                DAY,
                @SoNgayGiaHan,
                HanLaiDon
            ),
        HanThanhLy =
            DATEADD
            (
                DAY,
                @SoNgayGiaHan,
                HanThanhLy
            ),
        TrangThaiPhieu = N'Đang vay'
    WHERE MaPhieuCam = @MaPhieuCam
    PRINT N'Gia hạn hợp đồng thành công'
END
GO
```
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/61846c7a-26b7-46bc-9b29-6e387c29457c" />

+ Test gia hạn
<img width="1917" height="1072" alt="image" src="https://github.com/user-attachments/assets/d96cc769-4d3d-4892-8bd9-fd4a2162281c" />

+ Kiểm tra hợp đồng
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/3db94596-03bf-4d6b-aa62-3a4f397e160f" />

+ Kiểm tra lịch sử thanh toán
<img width="1917" height="1077" alt="image" src="https://github.com/user-attachments/assets/39c544ca-9a4d-4e33-87d6-eecd5a78cad4" />










