# BÀI TẬP 2 HỆ QUẢN TRỊ CƠ SỞ DỮ LIỆU SQL SERVER
# PHẦN MỞ ĐẦU
Thông tin cá nhân
- Họ và tên: Nguyễn Trung Kiên
- Lớp: K59KMT.K01
- Mã số sinh viên: K235480106037
- Học phần: Hệ quản trị Cơ sở dữ liệu 
- Đề tài thực hiện: Hệ thống Quản lý Sinh viên

## Yêu cầu đầu bài
- Thiết kế và khởi tạo cấu trúc dữ liệu cho một bài toán quản lý thực tế, bao gồm:

- Tạo Database theo định dạng quy định.

- Xây dựng ít nhất 03 bảng có quan hệ chặt chẽ.

- Sử dụng đa dạng kiểu dữ liệu và áp dụng các ràng buộc (PK, FK, CK).

- Tuân thủ quy tắc đặt tên CamelCase và sử dụng cặp ngoặc [] trong Script.

## Giới thiệu cách làm
Trong dự án này, em lựa chọn xây dựng hệ thống **Quản lý Sinh viên**. Cấu trúc được chia làm 3 tầng thực thể chính:

- Lớp học: Quản lý thông tin định danh của các lớp hành chính.

- Hồ sơ Sinh viên: Lưu trữ thông tin cá nhân chi tiết của sinh viên, liên kết với bảng Lớp học (Quan hệ 1-N).

- Kết quả học tập: Lưu trữ điểm số, tình trạng học phí và thời gian cập nhật, liên kết trực tiếp với mã sinh viên để đảm bảo tính đồng bộ.
# PHẦN 1: KHỞI TẠO CẤU TRÚC DỮ LIỆU
## 1.1. Script tạo Database

Tên Database được đặt theo cú pháp: [Tên đề tài]_[Mã SV].
```sql
-- Khởi tạo Database
CREATE DATABASE [QuanLySinhVien_K235480106037];
GO
USE [QuanLySinhVien_K235480106037];
GO
```

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/a8ef8059-cd98-4c5a-bba3-6d9bfe87b978" />

## 1.2. Script khởi tạo các bảng
Dưới đây là mã nguồn SQL để thiết lập các bảng với đầy đủ các ràng buộc, các khóa chính, khóa ngoại mà em đã liệt kể trong code
```sql
-- 1. Bảng Lớp Học
CREATE TABLE [LopHoc] (
    [MaLop] INT PRIMARY KEY IDENTITY(1,1), -- Khóa chính (PK): Tự động tăng
    [TenLop] NVARCHAR(50) NOT NULL,        -- Unicode cho tiếng Việt
    [NamNhapHoc] INT CONSTRAINT [CK_NamNhapHoc] CHECK ([NamNhapHoc] > 2000) -- Ràng buộc (CK)
);

-- 2. Bảng Hồ Sơ Sinh Viên
CREATE TABLE [HoSoSinhVien] (
    [MaSinhVien] NVARCHAR(20) PRIMARY KEY, -- Khóa chính (PK)
    [HoTen] NVARCHAR(100) NOT NULL,        -- Chuỗi Unicode
    [NgaySinh] DATE,                       -- Kiểu ngày tháng
    [Email] VARCHAR(100) UNIQUE,           -- Ràng buộc duy nhất
    [MaLop] INT,                           -- Khóa ngoại (FK)
    CONSTRAINT [FK_SinhVien_Lop] FOREIGN KEY ([MaLop]) REFERENCES [LopHoc]([MaLop])
);

-- 3. Bảng Kết Quả Học Tập
CREATE TABLE [KetQuaHocTap] (
    [MaKetQua] INT PRIMARY KEY IDENTITY(1,1),
    [MaSinhVien] NVARCHAR(20),             -- Khóa ngoại (FK) dẫn về bảng SinhVien
    [DiemTrungBinh] FLOAT CONSTRAINT [CK_DiemSo] CHECK ([DiemTrungBinh] BETWEEN 0 AND 10), -- Ràng buộc (CK) điểm 0-10
    [HocPhiDaDong] MONEY DEFAULT 0,        -- Kiểu tiền tệ
    [NgayCapNhat] DATETIME DEFAULT GETDATE(),
    CONSTRAINT [FK_KetQua_SinhVien] FOREIGN KEY ([MaSinhVien]) REFERENCES [HoSoSinhVien]([MaSinhVien])
);
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/6a52bad1-bb14-4f93-a312-d15fb45f2c3c" />

Giải thích các ràng buộc
- PK (Primary Key): Thiết lập trên [MaLop], [MaSinhVien], [MaKetQua] để đảm bảo tính duy nhất.

- FK (Foreign Key): * [MaLop] trong bảng Sinh viên tham chiếu đến bảng Lớp.

- [MaSinhVien] trong bảng Kết quả tham chiếu đến bảng Hồ sơ.

- CK (Check Constraint): * [NamNhapHoc] > 2000: Đảm bảo dữ liệu năm hợp lý.

- [DiemTrungBinh]: Ép dữ liệu nằm trong thang điểm từ 0 đến 10.

- Kiểu dữ liệu: Sử dụng NVARCHAR cho các trường có dấu, MONEY cho tài chính và DATETIME để lưu vết thời gian.

# Phần 2: Xây dựng Function
## 2.1. Các nhóm hàm Built-in phổ biến
- Hàm Chuỗi: LEN, LEFT, RIGHT, UPPER, LOWER, REPLACE, LTRIM, RTRIM.

- Hàm Ngày tháng: GETDATE, DATEPART, DATEDIFF, DATEADD.

- Hàm Hệ thống: ISNULL, COALESCE, CAST, CONVERT, IIF.

- Hàm Tổng hợp: SUM, AVG, COUNT, MIN, MAX.

## 2.2. Các hàm "Đặc sắc" và Ứng dụng vào Đề tài 
Dưới đây là 3 hàm (hoặc nhóm hàm) mà em chọn vì nó giúp dữ liệu trong bài Quản lý sinh viên trở nên thông minh hơn:

### A. Hàm DATEDIFF và GETDATE (Tính tuổi sinh viên)
Thay vì lưu số tuổi (vốn sẽ thay đổi theo từng năm), ta dùng hàm này để tính tuổi trực tiếp từ ngày sinh.

Góc nhìn: Giúp dữ liệu luôn "tươi" mà không cần cập nhật thủ công hàng năm.

### B. Hàm COALESCE (Xử lý dữ liệu trống)
Hàm này trả về giá trị đầu tiên không bị NULL trong danh sách.

Góc nhìn: Rất hay khi làm báo cáo học phí. Nếu sinh viên chưa đóng đồng nào (NULL), ta hiển thị là 0 thay vì để trống gây hiểu lầm.

### C. Hàm FORMAT (Định dạng tiền tệ và ngày tháng)
Giúp biến những con số thô kệch thành định dạng dễ đọc theo chuẩn Việt Nam.

Góc nhìn: Làm cho các bảng điểm hoặc biên lai học phí trông chuyên nghiệp hơn hẳn.

###2.3. SQL khai thác các hàm trên (Demo)

Em sử chèn thêm thông tin và sử dụng đoạn mã này để truy vấn dữ liệu sinh viên một cách "xịn xò" hơn 
```sql
-- 3. CHÈN DỮ LIỆU MẪU (4 SINH VIÊN)
INSERT INTO [LopHoc] ([TenLop], [NamNhapHoc]) VALUES (N'Công nghệ thông tin K23', 2023);

-- Chèn 4 sinh viên
INSERT INTO [HoSoSinhVien] ([MaSinhVien], [HoTen], [NgaySinh], [MaLop], [Email]) 
VALUES 
('K235480106037', N'Nguyễn Trung Kiên', '2005-10-15', 1, 'kien.nt@gmail.com'),
('K235480106038', N'Trần Thị B', '2004-02-20', 1, 'b.tt@gmail.com'),
('K235480106999', N'Lê Văn Tèo', '2005-12-30', 1, 'teo.lv@gmail.com'),
('K235480106888', N'Phạm Thị Nở', '2005-08-15', 1, 'no.pt@gmail.com');

-- Chèn kết quả (Sinh viên cuối cùng không chèn để test NULL)
INSERT INTO [KetQuaHocTap] ([MaSinhVien], [DiemTrungBinh], [HocPhiDaDong]) 
VALUES 
('K235480106037', 8.5, 5000000),
('K235480106038', 7.0, 4500000),
('K235480106999', 3.2, 2500000);
GO
```

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/0225218e-8619-440c-b0e6-46fb9a2a10d1" />

```sql
-- 4. KHAI THÁC HÀM BUILT-IN TRONG SQL SERVER
SELECT 
    [HoTen],
    -- Hàm DATEDIFF tính tuổi
    DATEDIFF(YEAR, [NgaySinh], GETDATE()) AS [TuoiHienTai],
    
    -- Hàm COALESCE xử lý dữ liệu trống (NULL)
    COALESCE([DiemTrungBinh], 0) AS [DiemSo],
    
    -- Hàm FORMAT định dạng tiền tệ Việt Nam
    FORMAT(COALESCE([HocPhiDaDong], 0), 'N0') + ' VND' AS [HocPhiFormatted],
    
    -- Hàm IIF để phân loại sinh viên nhanh
    IIF([DiemTrungBinh] >= 5, N'Đạt', N'Học lại/Chưa có điểm') AS [TrangThai]
FROM [HoSoSinhVien] sv
LEFT JOIN [KetQuaHocTap] kq ON sv.[MaSinhVien] = kq.[MaSinhVien];
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/7e964a85-17be-43d9-a776-22a74fa50c72" />

## 2.3. Mục đích của hàm tự viết (User-Defined Functions - UDF)
Mục đích chính là Tái sử dụng (Reuse) và Đóng gói logic (Encapsulation). Thay vì phải viết đi viết lại một công thức tính toán phức tạp ở 10 chỗ khác nhau, em chỉ cần viết một lần vào hàm và gọi nó ra ở bất cứ đâu.
### 2.4. Các loại Function trong SQL Server

| Loại Function | Đặc điểm | Khi nào dùng? |
| :--- | :--- | :--- |
| **Scalar Function** (Hàm vô hướng) | Trả về duy nhất **một giá trị** (số, chuỗi, ngày tháng). | Tính toán đơn giản. Ví dụ: Tính thuế học phí, viết hoa chữ cái đầu. |
| **Inline Table-Valued** (Hàm bảng đơn giản) | Trả về **một bảng** từ duy nhất một câu lệnh `SELECT`. | Dùng như một "View động" có tham số. Ví dụ: Lọc sinh viên theo mã lớp. |
| **Multi-statement Table-Valued** (Hàm bảng phức tạp) | Trả về **một bảng** nhưng chứa nhiều bước logic (IF/ELSE, vòng lặp). | Xử lý dữ liệu qua nhiều bước phức tạp trước khi xuất kết quả. |
## 2.5. Tại sao cần viết hàm riêng khi đã có System Function?
Mặc dù SQL Server cung cấp hàng trăm hàm hệ thống (như GETDATE, LEN, SUM), chúng ta vẫn cần hàm tự viết vì 3 lý do "sống còn":

- Xử lý nghiệp vụ đặc thù: SQL Server không thể biết công thức tính học bổng riêng của trường Kiên (ví dụ: Điểm > 8.0 VÀ không có môn nào dưới 5). Bạn phải tự viết logic này.

- Đơn giản hóa câu lệnh Query: Thay vì một câu SELECT dài 100 dòng với những phép tính loằng ngoằng, bạn đưa phép tính đó vào hàm. Câu SELECT lúc này chỉ còn 1 dòng gọi hàm, giúp code sạch sẽ và dễ bảo trì.

- Tính nhất quán: Đảm bảo mọi phòng ban (Kế toán, Đào tạo) đều dùng chung một công thức tính điểm trung bình duy nhất do bạn định nghĩa, tránh mỗi người tính một kiểu.
## 2.6. Viết 01 Scalar Function (Hàm trả về một giá trị): Đưa ra 1 logic cho cơ sở dữ liệu của em, mà cần dùng đến function này. (SV TỰ NGHĨ RA YÊU CẦU CỦA HÀM VÀ VIẾT HÀM GIẢI QUYẾT NÓ)
Đối với đề tài Quản lý sinh viên của em, một logic cực kỳ thực tế mà các trường đại học hay dùng chính là Tính số tiền học phí còn nợ.Thông thường, học phí sẽ có một mức sàn cố định (định mức), và sinh viên có thể đã đóng một phần hoặc đóng dư. Việc viết một hàm để tính số tiền còn lại giúp bộ phận kế toán kiểm tra nhanh tình trạng tài chính của sinh viên đó.
-  Ý tưởng logic cho Hàm (Scalar Function)Tên hàm: fn_TinhHocPhiConNo
-  Tham số đầu vào: [MaSinhVien]
-  Logic xử lý: * Giả sử mức học phí cố định mỗi kỳ là 10,000,000 VND.
-  Hàm sẽ tìm trong bảng [KetQuaHocTap] số tiền sinh viên đã đóng.
-  Kết quả: Trả về số tiền còn nợ ($10,000,000 - Đã đóng$).
-  Nếu đóng dư, kết quả sẽ là số âm.
-  Nếu chưa đóng gì (NULL), coi như chưa đóng đồng nào.
  
```sql
  GO
CREATE FUNCTION [dbo].[fn_TinhHocPhiConNo] (@MaSV NVARCHAR(20))
RETURNS MONEY
AS
BEGIN
    DECLARE @DinhMuc MONEY = 10000000; -- Mức học phí cố định 10 triệu
    DECLARE @DaDong MONEY;
    DECLARE @ConNo MONEY;

    -- Lấy số tiền đã đóng từ bảng KetQuaHocTap
    SELECT @DaDong = ISNULL([HocPhiDaDong], 0) 
    FROM [KetQuaHocTap] 
    WHERE [MaSinhVien] = @MaSV;

    -- Nếu sinh viên chưa có hồ sơ trong bảng kết quả, coi như đã đóng 0
    IF @DaDong IS NULL SET @DaDong = 0;

    -- Tính toán
    SET @ConNo = @DinhMuc - @DaDong;

    RETURN @ConNo;
END;
GO
SELECT 
    [MaSinhVien],
    [HoTen],
    -- Gọi hàm tự viết để tính tiền nợ
    [dbo].[fn_TinhHocPhiConNo]([MaSinhVien]) AS [SoTienConNo],
    
    -- Kết hợp với IIF để đưa ra trạng thái thanh toán
    IIF([dbo].[fn_TinhHocPhiConNo]([MaSinhVien]) <= 0, N'Đã hoàn thành', N'Còn nợ') AS [TrangThaiHocPhi]
FROM [HoSoSinhVien];
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/8dae44e6-aa41-40d4-b94c-0860692cec29" />
Ở cột SoTienConNo, dữ liệu đang hiện dạng số thô (ví dụ: 5000000.00). em sẽ lồng thêm hàm FORMAT vào câu lệnh SELECT như sau:
```sql
SELECT 
    [MaSinhVien],
    [HoTen],
    FORMAT([dbo].[fn_TinhHocPhiConNo]([MaSinhVien]), 'N0') + ' VND' AS [SoTienConNo_Formatted],
    IIF([dbo].[fn_TinhHocPhiConNo]([MaSinhVien]) <= 0, N'Đã hoàn thành', N'Còn nợ') AS [TrangThaiHocPhi]
FROM [HoSoSinhVien];
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/b5604370-f7b7-49ca-bde3-4957c504dd60" />


## 2.7. Viết 01 Inline Table-Valued Function: Trả về danh sách các bản ghi theo một điều kiện lọc cụ thể (SV TỰ NGHĨ RA YÊU CẦU CỦA HÀM VÀ VIẾT HÀM GIẢI QUYẾT NÓ
Lấy danh sách sinh viên theo tên lớp học. Thay vì phải viết câu lệnh JOIN phức tạp mỗi khi muốn xem sinh viên của một lớp nào đó, em sẽ gọi hàm và truyền tên lớp vào.
### Ý tưởng logic cho Hàm (Table-Valued Function)
- Tên hàm: fn_DanhSachSinhVienTheoLop

- Tham số đầu vào: @TenLop (NVARCHAR)

- Kết quả trả về: Một bảng gồm các cột: MaSinhVien, HoTen, NgaySinh, TenLop.

 ### Cách khai thác Hàm
- Điểm đặc biệt của hàm trả về bảng là Kiên không dùng SELECT [dbo].[TenHam], mà phải dùng SELECT * FROM [dbo].[TenHam](...) (coi hàm như một cái bảng thực thụ).
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/4ccba412-a503-40e1-8fb8-096a650c5a78" />

## 2.8. Viết 01 Multi-statement Table-Valued Function: Thực hiện xử lý logic phức tạp bên trong (có sử dụng biến bảng) trước khi trả về kết quả. (SV TỰ NGHĨ RA YÊU CẦU CỦA HÀM VÀ VIẾT HÀM GIẢI QUYẾT NÓ)
### Ý tưởng logic: Phân loại tình trạng học tập chi tiết
Mình sẽ viết một hàm tên là fn_BaoCaoChiTietHocTap. Hàm này sẽ duyệt qua danh sách sinh viên và tính toán để đưa ra một bảng báo cáo "thông minh"  gồm:

- Thông tin sinh viên.

- Điểm trung bình.

- Nhận xét chuyên môn: (Dựa trên điểm số: Giỏi, Khá, Trung bình, Yếu).

- Trạng thái học phí: (Đã đóng đủ, Còn nợ, hoặc Chưa có dữ liệu).

### Sau đó mình sẽ khai thác hàm
```sql
GO
CREATE FUNCTION [dbo].[fn_BaoCaoChiTietHocTap]()
RETURNS @BangKetQua TABLE (
    [MSSV] NVARCHAR(20),
    [TenSinhVien] NVARCHAR(100),
    [Diem] FLOAT,
    [XepLoai] NVARCHAR(50),
    [TinhTrangTaiChinh] NVARCHAR(50)
)
AS
BEGIN
    -- Chèn dữ liệu vào biến bảng @BangKetQua với các logic xử lý phức tạp
    INSERT INTO @BangKetQua
    SELECT 
        sv.[MaSinhVien],
        sv.[HoTen],
        ISNULL(kq.[DiemTrungBinh], 0),
        -- Logic 1: Phân loại học lực
        CASE 
            WHEN kq.[DiemTrungBinh] >= 8.0 THEN N'Xuất sắc/Giỏi'
            WHEN kq.[DiemTrungBinh] >= 6.5 THEN N'Khá'
            WHEN kq.[DiemTrungBinh] >= 5.0 THEN N'Trung bình'
            WHEN kq.[DiemTrungBinh] < 5.0 AND kq.[DiemTrungBinh] IS NOT NULL THEN N'Yếu (Cảnh báo)'
            ELSE N'Chưa có điểm'
        END,
        -- Logic 2: Phân loại tài chính (Dựa trên mức sàn 10 triệu)
        CASE 
            WHEN kq.[HocPhiDaDong] >= 10000000 THEN N'Đã hoàn thành'
            WHEN kq.[HocPhiDaDong] > 0 THEN N'Còn nợ (Đã đóng một phần)'
            ELSE N'Chưa đóng phí'
        END
    FROM [HoSoSinhVien] sv
    LEFT JOIN [KetQuaHocTap] kq ON sv.[MaSinhVien] = kq.[MaSinhVien];

    -- Bạn có thể viết thêm các câu lệnh UPDATE hoặc logic bổ sung tại đây trước khi RETURN
    
    RETURN;
END;
GO
SELECT * FROM [dbo].[fn_BaoCaoChiTietHocTap]();
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/27034220-7f96-4bc3-bd19-cffdfea30c2e" />


