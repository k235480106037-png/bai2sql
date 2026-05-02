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

# Phần 3: Xây dựng Store Procedure

## Trong SQL Server có những SP có sẵn nào? nêu 1 vài system sp mà em tìm hiểu được, giải thích cách dùng chúng.
- System Store Procedure là gì?
Trong SQL Server, các System SP là những thủ tục được Microsoft viết sẵn để giúp người quản trị hệ thống (DBA) và lập trình viên kiểm tra, cấu hình và quản lý cơ sở dữ liệu một cách nhanh chóng. Các thủ tục này thường bắt đầu bằng tiền tố sp_.

- Một số System Store Procedure đặc sắc (Góc nhìn ứng dụng)
Dưới đây là 3 thủ tục mà mình thấy "quyền lực" nhất khi bạn làm việc với SQL:

### A. sp_help
Mục đích: Cung cấp thông tin chi tiết về bất kỳ đối tượng nào (bảng, hàm, thủ tục, view).

Tại sao nó đặc sắc: Thay vì phải vào cây thư mục bên trái để tìm xem bảng này có những cột nào, kiểu dữ liệu gì, khóa chính là ai, Kiên chỉ cần gõ 1 dòng lệnh.


### B. sp_helptext
Mục đích: Hiển thị mã nguồn (Script) của một thủ tục, hàm hoặc view đã được tạo.

Tại sao nó đặc sắc: Khi bạn làm việc nhóm hoặc thừa hưởng code từ người khác, nếu muốn xem bên trong một cái Function họ viết gì mà không muốn tìm file gốc, hãy dùng hàm này.


### C. sp_rename
Mục đích: Đổi tên một đối tượng (bảng hoặc tên cột) mà không cần phải xóa đi tạo lại.

Tại sao nó đặc sắc: Nó cực kỳ tiện lợi khi bạn lỡ đặt tên cột sai quy tắc CamelCase hoặc muốn đổi tên bảng cho chuyên nghiệp hơn mà vẫn giữ nguyên dữ liệu bên trong.

## Viết 01 Store Procedure đơn giản để thực hiện lệnh INSERT hoặc UPDATE dữ liệu, có kiểm tra điều kiện logic (SV TỰ NGHĨ RA YÊU CẦU CỦA SP VÀ VIẾT SP GIẢI QUYẾT NÓ)
### Ý tưởng logic cho Store Procedure
- Tên SP: sp_CapNhatDiemSinhVien

- Tham số:  @MaSV, @DiemMoi.

- Logic kiểm tra (Validation):

- Kiểm tra xem mã sinh viên có tồn tại trong hệ thống hay không.

- Kiểm tra điểm mới có nằm trong khoảng hợp lệ từ 0 đến 10 không.

- Nếu thỏa mãn cả hai: Thực hiện cập nhật và thông báo thành công.

- Nếu không thỏa mãn: Thông báo lỗi cụ thể và không thực hiện cập nhật.

```sql
 GO
CREATE PROCEDURE [dbo].[sp_CapNhatDiemSinhVien]
    @MaSV NVARCHAR(20),
    @DiemMoi FLOAT
AS
BEGIN
    -- 1. Kiểm tra mã sinh viên có tồn tại không
    IF NOT EXISTS (SELECT 1 FROM [HoSoSinhVien] WHERE [MaSinhVien] = @MaSV)
    BEGIN
        PRINT N'Lỗi: Không tìm thấy sinh viên có mã ' + @MaSV;
        RETURN;
    END

    -- 2. Kiểm tra logic điểm số (0 - 10)
    IF @DiemMoi < 0 OR @DiemMoi > 10
    BEGIN
        PRINT N'Lỗi: Điểm nhập vào (' + CAST(@DiemMoi AS NVARCHAR(5)) + N') không hợp lệ (Phải từ 0-10).';
        RETURN;
    END

    -- 3. Thực hiện cập nhật nếu các điều kiện trên đều thỏa mãn
    UPDATE [KetQuaHocTap]
    SET [DiemTrungBinh] = @DiemMoi,
        [NgayCapNhat] = GETDATE()
    WHERE [MaSinhVien] = @MaSV;

    -- Kiểm tra nếu sinh viên chưa có dòng nào trong bảng KetQuaHocTap thì INSERT mới
    IF @@ROWCOUNT = 0
    BEGIN
        INSERT INTO [KetQuaHocTap] ([MaSinhVien], [DiemTrungBinh])
        VALUES (@MaSV, @DiemMoi);
        PRINT N'Đã khởi tạo bản ghi mới và cập nhật điểm thành công.';
    END
    ELSE
    BEGIN
        PRINT N'Đã cập nhật điểm thành công cho sinh viên: ' + @MaSV;
    END
END;
GO
```

### Em sẽ kiểm tra các trường hợp bằng cách chạy các lệnh dưới đây:

- Trường hợp 1: Cập nhật điểm thành công (cho chính bạn)

```sql
EXEC [dbo].[sp_CapNhatDiemSinhVien] @MaSV = 'K235480106037', @DiemMoi = 9.5;
```
<img width="1918" height="1075" alt="image" src="https://github.com/user-attachments/assets/e8ab7622-f419-4aa0-8180-90a41bc18b96" />

- Trường hợp 2: Nhập điểm sai quy định (Ví dụ: 11 điểm)

```sql
EXEC [dbo].[sp_CapNhatDiemSinhVien] @MaSV = 'K235480106037', @DiemMoi = 11.0;
-- Kết quả: Sẽ báo lỗi điểm không hợp lệ.
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/af97c7b2-f1dd-491d-b2a1-0de7e6812efb" />

- Trường hợp 3: Nhập mã sinh viên không tồn tại

```sql 
EXEC [dbo].[sp_CapNhatDiemSinhVien] @MaSV = 'SV_KHONG_TON_TAI', @DiemMoi = 8.0;
-- Kết quả: Sẽ báo lỗi không tìm thấy sinh viên.
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/bb109726-e6ff-4653-941e-5a37340916c7" />

## Viết 01 Store Procedure có sử dụng tham số OUTPUT để trả về một giá trị tính toán (SV TỰ NGHĨ RA YÊU CẦU CỦA SP VÀ VIẾT SP GIẢI QUYẾT NÓ, SP NÀY CÓ DÙNG THAM SỐ LOẠI OUTPUT)

### Ý tưởng logic: Thống kê số lượng sinh viên theo lớp
- Tên SP: sp_DemSinhVienTheoLop

- Tham số đầu vào (INPUT): @TenLop (Tên lớp cần đếm).

- Tham số đầu ra (OUTPUT): @TongSoSV (Trả về số lượng sinh viên đếm được).

- Logic: SP sẽ tìm mã lớp dựa trên tên lớp, sau đó đếm tổng số sinh viên thuộc lớp đó và gán vào biến đầu ra.

```sql
GO
CREATE PROCEDURE [dbo].[sp_DemSinhVienTheoLop]
    @TenLop NVARCHAR(50),          -- Tham số vào (Input)
    @TongSoSV INT OUTPUT           -- Tham số ra (Output)
AS
BEGIN
    -- Tính toán và gán giá trị vào tham số OUTPUT
    SELECT @TongSoSV = COUNT(sv.[MaSinhVien])
    FROM [HoSoSinhVien] sv
    JOIN [LopHoc] l ON sv.[MaLop] = l.[MaLop]
    WHERE l.[TenLop] LIKE '%' + @TenLop + '%';
    
    -- Nếu không tìm thấy lớp hoặc không có SV, mặc định là 0
    IF @TongSoSV IS NULL SET @TongSoSV = 0;
END;
GO
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/dc5da476-1e40-4afa-b3c0-c70dd9251aee" />


### Cách khai thác (Thực thi và nhận giá trị trả về)
Để nhận được giá trị từ tham số OUTPUT, em sẽ khai báo một biến bên ngoài để "hứng" kết quả. 

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/ae64f73f-3e4e-40ae-a9ba-4376cc676734" />
## Viết 01 Store Procedure trả về một tập kết quả (Result set) từ lệnh SELECT sau khi đã join nhiều bảng. (SV TỰ NGHĨ RA YÊU CẦU CỦA SP VÀ VIẾT SP GIẢI QUYẾT NÓ)

### Ý tưởng logic cho Store Procedure
- Tên SP: sp_BaoCaoKetQuaHocTapChiTiet

- Tham số vào: @MaLop (Dùng để lọc báo cáo theo từng lớp cụ thể).

- Nhiệm vụ: JOIN 3 bảng lại để lấy ra: Mã sinh viên, Họ tên, Tên lớp, Điểm trung bình và Học phí đã đóng.

```sql
GO
CREATE PROCEDURE [dbo].[sp_BaoCaoKetQuaHocTapChiTiet]
    @MaLop INT = NULL -- Nếu để NULL sẽ hiện tất cả sinh viên
AS
BEGIN
    SELECT 
        sv.[MaSinhVien],
        sv.[HoTen],
        l.[TenLop],
        ISNULL(kq.[DiemTrungBinh], 0) AS [DiemSo],
        FORMAT(ISNULL(kq.[HocPhiDaDong], 0), 'N0') + ' VND' AS [HocPhi]
    FROM [HoSoSinhVien] sv
    INNER JOIN [LopHoc] l ON sv.[MaLop] = l.[MaLop]
    LEFT JOIN [KetQuaHocTap] kq ON sv.[MaSinhVien] = kq.[MaSinhVien]
    WHERE (@MaLop IS NULL OR l.[MaLop] = @MaLop)
    ORDER BY l.[TenLop], sv.[HoTen] ASC;
END;
GO
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/7ea231bf-f037-49f5-9af3-8d4705582182" />

### Cách khai thác (Thực thi SP)
Có thể chạy SP này theo hai cách:

- Cách 1: Xem báo cáo của tất cả sinh viên trong hệ thống
```sql
EXEC [dbo].[sp_BaoCaoKetQuaHocTapChiTiet];
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/ce8fd5bb-3afb-4729-88e9-90d767f9dc05" />

- Cách 2: Chỉ xem báo cáo của lớp có mã số 1

```sql 
EXEC [dbo].[sp_BaoCaoKetQuaHocTapChiTiet] @MaLop = 1;
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/bb1adf95-58f2-4f9b-b6fa-53a8d1cc3a0b" />

# Phần 4: Trigger và Xử lý logic nghiệp vụ 

## Ý tưởng logic Trigger thực tế (Phần đầu)
- Khi thêm một sinh viên mới vào bảng [HoSoSinhVien] (Bảng A), hệ thống phải tự động tạo một dòng trống bên bảng [KetQuaHocTap] (Bảng B) cho sinh viên đó với điểm số bằng 0.

- Mục tiêu: Đảm bảo mọi sinh viên mới đều có hồ sơ học tập ngay lập tức mà không cần nhập tay hai lần.


```sql
GO
CREATE TRIGGER [trg_AutoCreateRecord]
ON [HoSoSinhVien]
AFTER INSERT
AS
BEGIN
    INSERT INTO [KetQuaHocTap] ([MaSinhVien], [DiemTrungBinh], [HocPhiDaDong])
    SELECT [MaSinhVien], 0, 0 FROM inserted;
    PRINT N'==> Trigger A: Đã tự động tạo hồ sơ học tập cho sinh viên mới.';
END;
GO
```

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/8c9f69cc-6c40-468c-8a7f-29e2701af99d" />

## Thử nghiệm Trigger "Đuổi bắt" (Bảng A tác động B, B tác động ngược lại A)
Hãy tạo thêm một Trigger trên bảng B, logic là: Khi cập nhật bảng B thì cập nhật lại một thông tin nào đó ở bảng A.

```sql
GO
CREATE TRIGGER [trg_UpdateBackToA]
ON [KetQuaHocTap]
AFTER UPDATE
AS
BEGIN
    -- Khi điểm ở bảng B thay đổi, cập nhật Email ở bảng A (ví dụ để đánh dấu đã update)
    UPDATE [HoSoSinhVien]
    SET [Email] = [Email] -- Chỉ là lệnh gán giả vờ để kích hoạt logic cập nhật
    FROM [HoSoSinhVien] sv
    JOIN inserted i ON sv.[MaSinhVien] = i.[MaSinhVien];
    
    PRINT N'==> Trigger B: Đã cập nhật ngược lại bảng A.';
END;
GO
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/ff8d9b42-d350-41a6-8b6d-2ba08e0aa6f6" />

## Quan sát hệ thống và Giải thích
Tình huống: Nếu bật cả 2 Trigger này và thực hiện một lệnh Update điểm ở bảng B, chuyện gì sẽ xảy ra?

Thông báo hệ thống: Thông thường, nếu SQL Server được cấu hình mặc định, bạn sẽ thấy các dòng PRINT lặp đi lặp lại một số lần, hoặc tệ hơn là lỗi:

"Maximum stored procedure, function, trigger, or view nesting level exceeded (limit 32)."

## Giải thích: 
-  Lệnh Update ở B kích hoạt Trigger B.
-  Trigger B thực hiện lệnh Update ở A.
- Lệnh Update ở A (nếu có Trigger Update ở A) lại kích hoạt ngược lại B.
-  Cứ thế, chúng tạo thành một cái vòng lặp không hồi kết. SQL Server có một "cầu chì" bảo vệ, nếu vòng lặp này quá 32 lần, nó sẽ tự ngắt để tránh làm treo máy chủ (Crash server).


### Cách bật/tắt một Trigger cụ thể
Nếu đã viết code tạo Trigger rồi nhưng nó không chạy, có thể nó đang ở trạng thái bị vô hiệu hóa (Disable). Dùng lệnh sau:

```sql
-- Bật Trigger trên bảng HoSoSinhVien
ENABLE TRIGGER [trg_AutoCreateRecord] ON [HoSoSinhVien];

-- Bật Trigger trên bảng KetQuaHocTap
ENABLE TRIGGER [trg_UpdateBackToA] ON [KetQuaHocTap];
(Ngược lại, nếu muốn tắt để sửa dữ liệu mà không bị dính logic tự động, Kiên thay ENABLE bằng DISABLE nhé).
```

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/3ea5ebe9-3b54-4367-8df0-bea247b8a07e" />




### Cách bật tính năng "Vòng lặp" (Recursive Triggers)
Mặc định, SQL Server thường tắt tính năng Trigger bảng này gọi Trigger bảng kia một cách gián tiếp để bảo vệ hệ thống. Để thấy được hiện tượng lặp (và cả thông báo lỗi tràn tầng), cần bật cấu hình này lên cho Database của mình:

```sql
-- Cho phép Trigger hoạt động bắc cầu (A -> B -> A)
ALTER DATABASE [QuanLySinhVien_K235480106037] 
SET RECURSIVE_TRIGGERS ON;
GO
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/5160e86e-f7cc-4ec1-8987-2d7e103b4fe6" />


### Cách "Kích hoạt" để quan sát thông báo
Sau khi đã chạy lệnh tạo 2 Trigger và bật cấu hình ở trên, Kiên hãy thực hiện một hành động để "châm ngòi".
```sql
-- Cập nhật điểm cho chính bạn để xem Trigger B gọi A, rồi A lại gọi B...
UPDATE [KetQuaHocTap] 
SET [DiemTrungBinh] = 10 
WHERE [MaSinhVien] = 'K235480106037';
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/d975efc6-4d21-426c-9544-33936f5b2ca2" />


### Nhận xét cuối cùng 
Về tính thực tế: Trigger rất mạnh mẽ trong việc đảm bảo tính đồng bộ dữ liệu tự động (như việc tự tạo hồ sơ ở bảng B khi bảng A có dữ liệu mới).

Về lỗi vòng lặp (Recursive Trigger): Việc thiết kế các Trigger tác động chéo lẫn nhau (A gọi B, B gọi A) là một sai lầm nguy hiểm trong thiết kế cơ sở dữ liệu. Nó gây ra tình trạng Lặp vô tận, làm tốn tài nguyên hệ thống và dẫn đến lỗi tràn tầng lặp (nesting level exceeded).

### Giải pháp:  
- Hạn chế tối đa việc dùng Trigger cập nhật ngược xuôi giữa 2 bảng.

- Sử dụng Store Procedure để xử lý logic tuần tự thay vì dùng Trigger nếu nghiệp vụ quá phức tạp.

- Nếu bắt buộc phải dùng, cần sử dụng hàm TRIGGER_NESTLEVEL() để kiểm tra và ngắt vòng lặp kịp thời.


# Phần 5: Cursor và Duyệt dữ liệu

## Viết một đoạn script sử dụng CURSOR để duyệt qua danh sách của 1 câu lệnh SQL dạng SELECT, duyệt qua từng bản ghi, xử lý riêng từng bản ghi (THEO LOGIC SV TỰ ĐẶT RA: SAO CHO HỢP LÝ VÀ THUYẾT PHỤC)

Trong SQL, Cursor giống như một vòng lặp for trong lập trình (C#, Java), cho phép ta duyệt qua từng dòng một của kết quả SELECT để xử lý những logic mà một câu lệnh SQL thuần túy khó làm được.
1. Ý tưởng logic: Tự động gửi thông báo học tập
2. Kịch bản: Kiên cần duyệt qua danh sách tất cả sinh viên.
3. Logic xử lý: Với mỗi sinh viên, ta sẽ kiểm tra điểm số:
4. Nếu điểm $\ge 5.0$: In ra lời chúc mừng kèm tên sinh viên.
5. Nếu điểm $< 5.0$: In ra lời nhắc nhở yêu cầu sinh viên ôn tập lại.
6. Mục tiêu: Mô phỏng hệ thống gửi tin nhắn cá nhân hóa cho từng người dựa trên kết quả học tập.

```sql
GO
-- 1. Khai báo các biến để chứa dữ liệu từng dòng
DECLARE @TenSV NVARCHAR(100);
DECLARE @Diem FLOAT;

-- 2. Khai báo Cursor để duyệt danh sách sinh viên và điểm
DECLARE cur_ThongBaoHocTap CURSOR FOR
    SELECT sv.[HoTen], ISNULL(kq.[DiemTrungBinh], 0)
    FROM [HoSoSinhVien] sv
    LEFT JOIN [KetQuaHocTap] kq ON sv.[MaSinhVien] = kq.[MaSinhVien];

-- 3. Mở Cursor
OPEN cur_ThongBaoHocTap;

-- 4. Lấy dòng dữ liệu đầu tiên
FETCH NEXT FROM cur_ThongBaoHocTap INTO @TenSV, @Diem;

-- 5. Vòng lặp duyệt qua từng bản ghi cho đến khi hết (@@FETCH_STATUS = 0)
WHILE @@FETCH_STATUS = 0
BEGIN
    -- Logic xử lý riêng cho từng bản ghi
    IF @Diem >= 5.0
        PRINT N'CHÚC MỪNG: Sinh viên [' + @TenSV + N'] đã đạt môn với điểm số: ' + CAST(@Diem AS NVARCHAR(5));
    ELSE
        PRINT N'CẢNH BÁO: Sinh viên [' + @TenSV + N'] cần chú ý ôn tập. Điểm hiện tại: ' + CAST(@Diem AS NVARCHAR(5));

    -- Lấy dòng dữ liệu tiếp theo
    FETCH NEXT FROM cur_ThongBaoHocTap INTO @TenSV, @Diem;
END;

-- 6. Đóng và giải phóng Cursor
CLOSE cur_ThongBaoHocTap;
DEALLOCATE cur_ThongBaoHocTap;
GO
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/9721ee3d-c8da-43a2-9722-da9544c0f8d0" />

## Tìm cách không sử dụng CURSOR để giải quyết bài toán mà em đã dùng CURSOR mới giải quyết được ở trên. thử so sánh tốc độ giữa có dùng cursor và không dùng cursor (nếu cùng kết quả) thì thời gian xử lý cái nào nhanh hơn, cần ảnh chụp màn hình minh chứng.

### Giải pháp không sử dụng CURSOR (Dùng SELECT thuần túy)
Thay vì duyệt từng dòng và dùng IF...ELSE, chúng ta sử dụng biểu thức CASE WHEN. Đây là cách tiếp cận Set-based (xử lý cả bảng cùng lúc).

```
-- Cách tiếp cận tập hợp (Không dùng Cursor)
SELECT 
    sv.[HoTen],
    kq.[DiemTrungBinh],
    CASE 
        WHEN ISNULL(kq.[DiemTrungBinh], 0) >= 5.0 
        THEN N'CHÚC MỪNG: Sinh viên [' + sv.[HoTen] + N'] đã đạt môn.'
        ELSE N'CẢNH BÁO: Sinh viên [' + sv.[HoTen] + N'] cần chú ý ôn tập.'
    END AS [ThongBao]
FROM [HoSoSinhVien] sv
LEFT JOIN [KetQuaHocTap] kq ON sv.[MaSinhVien] = kq.[MaSinhVien];
```

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/a0f6b8a3-dc74-42a0-9527-df813b5e1e8e" />

### So sánh tốc độ xử lý
Để so sánh chính xác, hãy bật tính năng đo thời gian và tài nguyên của SQL Server bằng hai câu lệnh này trước khi chạy:

```sql
SET STATISTICS TIME ON;
SET STATISTICS IO ON;
GO
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/1df2ff2f-07b9-4a5d-99b6-50c4c894ce7d" />




