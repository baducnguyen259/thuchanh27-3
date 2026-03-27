# Đặc tả yêu cầu phần mềm (SRS)

## Chức năng: Quản lý Sản phẩm (Product Information Management - PIM)

**Dự án:** Kochi Lens - Website bán thiết bị ngành ảnh  
**Mã chức năng:** PIM-01  
**Trạng thái:** Draft / Review  
**Người soạn thảo:** Duc  
**Vai trò:** Developer / Analyst

---

## 1. Mô tả tổng quan (Description)

Chức năng **Quản lý Sản phẩm (PIM - Product Information Management)** cung cấp cơ chế quản lý dữ liệu sản phẩm cho hệ thống thương mại điện tử Kochi Lens.  
Module này hỗ trợ quản lý thông tin sản phẩm, biến thể, giá bán, thuế VAT và tồn kho hiển thị realtime.

Chức năng PIM có nhiệm vụ chính:

- Quản lý sản phẩm và thông tin sản phẩm
- Quản lý biến thể sản phẩm (màu sắc, kích thước/phiên bản)
- Quản lý SKU và Barcode
- Thiết lập giá bán và thuế VAT
- Quản lý tồn kho theo từng biến thể
- Hiển thị tồn kho realtime cho khách hàng
- Cập nhật tồn kho bởi nhân viên kho

Module này đảm bảo dữ liệu sản phẩm luôn chính xác, hỗ trợ quá trình bán hàng, vận hành kho và kế toán.

---

## 2. Luồng nghiệp vụ (User Workflow)

### 2.1 Luồng quản lý sản phẩm (Admin Workflow)

| Bước | Hành động Admin                                    | Phản hồi hệ thống                                     |
| :--- | :------------------------------------------------- | :---------------------------------------------------- |
| 1    | Truy cập `/admin/products`                         | Hiển thị danh sách sản phẩm kèm tìm kiếm và bộ lọc.   |
| 2    | Nhấn nút **Thêm sản phẩm**                         | Hiển thị form tạo sản phẩm mới.                       |
| 3    | Nhập thông tin (Tên, Thương hiệu, Danh mục, Mô tả) | Kiểm tra dữ liệu bắt buộc.                            |
| 4    | Nhập giá bán và VAT                                | Validate giá >= 0, VAT từ 0-100.                      |
| 5    | Upload ảnh sản phẩm                                | Lưu ảnh và hiển thị preview.                          |
| 6    | Nhấn **Lưu**                                       | Tạo sản phẩm và sinh `product_id`.                    |
| 7    | Thêm biến thể (màu sắc, kích thước/phiên bản)      | Tạo dữ liệu variant và gán SKU/Barcode.               |
| 8    | Nhập tồn kho cho từng biến thể                     | Cập nhật tồn kho theo variant.                        |
| 9    | Hoàn tất                                           | Sản phẩm hiển thị trên website nếu trạng thái Active. |

---

### 2.2 Luồng xem sản phẩm và tồn kho (Customer Workflow)

| Bước | Hành động Customer                            | Phản hồi hệ thống                                    |
| :--- | :-------------------------------------------- | :--------------------------------------------------- |
| 1    | Truy cập `/products`                          | Hiển thị danh sách sản phẩm.                         |
| 2    | Click vào một sản phẩm                        | Hiển thị trang chi tiết sản phẩm.                    |
| 3    | Chọn biến thể (màu sắc, kích thước/phiên bản) | Truy vấn tồn kho realtime theo biến thể.             |
| 4    | Xem giá và tồn kho                            | Hiển thị số lượng còn lại hoặc trạng thái hết hàng.  |
| 5    | Nhấn **Thêm vào giỏ hàng**                    | Nếu còn hàng → thêm vào giỏ, nếu hết hàng → báo lỗi. |

---

### 2.3 Luồng cập nhật tồn kho (Warehouse Staff Workflow)

| Bước | Hành động Warehouse Staff       | Phản hồi hệ thống                         |
| :--- | :------------------------------ | :---------------------------------------- |
| 1    | Truy cập `/warehouse/stock`     | Hiển thị danh sách tồn kho theo SKU.      |
| 2    | Tìm kiếm theo SKU/Barcode       | Hệ thống lọc dữ liệu tương ứng.           |
| 3    | Chọn biến thể cần cập nhật      | Hiển thị form cập nhật tồn kho.           |
| 4    | Nhập số lượng nhập kho/xuất kho | Validate số lượng hợp lệ (>0).            |
| 5    | Nhấn **Xác nhận**               | Hệ thống cập nhật tồn kho và lưu log.     |
| 6    | Hoàn tất                        | Tồn kho cập nhật realtime cho khách hàng. |

---

## 3. Yêu cầu dữ liệu (Data Requirements)

### 3.1 Dữ liệu đầu vào (Input Fields)

#### 3.1.1 Product (Admin nhập)

- **Tên sản phẩm:** `string`, bắt buộc, tối đa 255 ký tự
- **Thương hiệu:** `string`, bắt buộc
- **Danh mục:** `string`, bắt buộc
- **Mô tả:** `text`, tùy chọn
- **Trạng thái:** `enum`, mặc định `Active`
- **Hình ảnh:** `file[]`, nhiều ảnh, định dạng `jpg/png/webp`

#### 3.1.2 Variant (Admin nhập)

- **Màu sắc:** `string`, bắt buộc
- **Kích thước/Phiên bản:** `string`, tùy chọn
- **SKU:** `string`, bắt buộc, không trùng lặp
- **Barcode:** `string`, tùy chọn, không trùng lặp
- **Giá bán:** `decimal`, bắt buộc, >= 0
- **VAT (%):** `float`, bắt buộc, từ 0-100
- **Tồn kho:** `int`, bắt buộc, >= 0

#### 3.1.3 Stock Adjustment (Warehouse Staff nhập)

- **Loại thao tác:** `enum` (IN / OUT)
- **Số lượng:** `int`, bắt buộc, > 0
- **Lý do:** `string`, tùy chọn
- **Ghi chú:** `text`, tùy chọn

---

### 3.2 Dữ liệu lưu trữ (Database Schema)

#### Bảng: `products`

| Field        | Type         | Constraint         | Mô tả           |
| ------------ | ------------ | ------------------ | --------------- |
| product_id   | INT          | PK, Auto Increment | ID sản phẩm     |
| product_name | VARCHAR(255) | NOT NULL           | Tên sản phẩm    |
| brand        | VARCHAR(255) | NOT NULL           | Thương hiệu     |
| category     | VARCHAR(255) | NOT NULL           | Danh mục        |
| description  | TEXT         | NULL               | Mô tả           |
| status       | ENUM         | DEFAULT Active     | Active/Inactive |
| created_at   | DATETIME     | NOT NULL           | Ngày tạo        |
| updated_at   | DATETIME     | NOT NULL           | Ngày cập nhật   |

---

#### Bảng: `product_variants`

| Field          | Type          | Constraint       | Mô tả             |
| -------------- | ------------- | ---------------- | ----------------- |
| variant_id     | INT           | PK               | ID biến thể       |
| product_id     | INT           | FK -> products   | Liên kết sản phẩm |
| sku            | VARCHAR(100)  | UNIQUE, NOT NULL | SKU duy nhất      |
| barcode        | VARCHAR(100)  | UNIQUE           | Barcode           |
| color          | VARCHAR(100)  | NOT NULL         | Màu sắc           |
| size_version   | VARCHAR(100)  | NULL             | Size/Phiên bản    |
| price          | DECIMAL(12,2) | NOT NULL         | Giá bán           |
| vat_percent    | FLOAT         | NOT NULL         | Thuế VAT (%)      |
| stock_quantity | INT           | NOT NULL         | Tồn kho           |
| status         | ENUM          | DEFAULT Active   | Active/Inactive   |
| created_at     | DATETIME      | NOT NULL         | Ngày tạo          |
| updated_at     | DATETIME      | NOT NULL         | Ngày cập nhật     |

---

#### Bảng: `product_images`

| Field      | Type     | Constraint | Mô tả                      |
| ---------- | -------- | ---------- | -------------------------- |
| image_id   | INT      | PK         | ID ảnh                     |
| product_id | INT      | FK         | Sản phẩm liên quan         |
| variant_id | INT      | FK, NULL   | Ảnh theo biến thể (nếu có) |
| image_url  | TEXT     | NOT NULL   | Link/đường dẫn ảnh         |
| created_at | DATETIME | NOT NULL   | Thời gian upload           |

---

#### Bảng: `stock_logs`

| Field       | Type         | Constraint | Mô tả                  |
| ----------- | ------------ | ---------- | ---------------------- |
| log_id      | INT          | PK         | ID log                 |
| variant_id  | INT          | FK         | Biến thể liên quan     |
| staff_id    | INT          | FK         | Nhân viên kho thao tác |
| action_type | ENUM         | NOT NULL   | IN / OUT               |
| quantity    | INT          | NOT NULL   | Số lượng thay đổi      |
| reason      | VARCHAR(255) | NULL       | Lý do                  |
| created_at  | DATETIME     | NOT NULL   | Thời gian tạo log      |

---

## 4. Yêu cầu chức năng (Functional Requirements)

### 4.1 Quản lý sản phẩm

- **FR-PIM-01:** Admin có thể tạo mới sản phẩm.
- **FR-PIM-02:** Admin có thể cập nhật thông tin sản phẩm.
- **FR-PIM-03:** Admin có thể xóa sản phẩm.
- **FR-PIM-04:** Admin có thể bật/tắt hiển thị sản phẩm (Active/Inactive).

### 4.2 Quản lý biến thể

- **FR-PIM-05:** Admin có thể tạo biến thể theo màu sắc, kích thước/phiên bản.
- **FR-PIM-06:** SKU của biến thể phải là duy nhất và không trùng lặp.
- **FR-PIM-07:** Barcode hỗ trợ việc quét mã trong kho.
- **FR-PIM-08:** Admin có thể cấu hình giá bán và VAT theo từng biến thể.

### 4.3 Quản lý tồn kho

- **FR-PIM-09:** Warehouse Staff có thể cập nhật tồn kho theo SKU/Barcode.
- **FR-PIM-10:** Hệ thống phải hiển thị tồn kho realtime cho Customer.
- **FR-PIM-11:** Không cho phép tồn kho âm.
- **FR-PIM-12:** Hệ thống tự động trừ kho sau khi thanh toán thành công.

### 4.4 Quản lý hình ảnh

- **FR-PIM-13:** Admin có thể upload nhiều ảnh cho sản phẩm.
- **FR-PIM-14:** Hệ thống hỗ trợ ảnh riêng theo biến thể.
- **FR-PIM-15:** Customer có thể xem gallery ảnh sản phẩm.

### 4.5 Tìm kiếm và lọc

- **FR-PIM-16:** Customer có thể tìm kiếm theo tên hoặc SKU.
- **FR-PIM-17:** Customer có thể lọc theo danh mục, thương hiệu, mức giá.

---

## 5. Ràng buộc kỹ thuật & Bảo mật (Technical Constraints & Security)

### 5.1 Phân quyền

- Admin có toàn quyền quản lý sản phẩm, biến thể, giá bán.
- Warehouse Staff chỉ có quyền cập nhật tồn kho và xem dữ liệu sản phẩm.
- Customer chỉ có quyền xem sản phẩm và tồn kho.

### 5.2 Toàn vẹn dữ liệu

- SKU phải duy nhất.
- Tồn kho không được âm.
- Khi có nhiều người đặt hàng cùng lúc, hệ thống phải xử lý transaction/locking để tránh bán vượt tồn kho.

### 5.3 Hiệu năng

- Thời gian truy vấn tồn kho realtime <= 2 giây.
- Trang danh sách sản phẩm phải load <= 3 giây với khoảng 1000 sản phẩm.

### 5.4 Logging & Audit

- Mọi thao tác nhập/xuất kho đều phải lưu vào bảng `stock_logs`.
- Log phải chứa thông tin nhân viên thao tác, thời gian, số lượng thay đổi.

---

## 6. Trường hợp ngoại lệ & Xử lý lỗi (Edge Cases)

### SKU bị trùng

- **Trường hợp:** Admin nhập SKU đã tồn tại.
- **Xử lý:** Báo lỗi "SKU đã tồn tại trong hệ thống".

### Không đủ tồn kho

- **Trường hợp:** Customer chọn số lượng mua lớn hơn tồn kho.
- **Xử lý:** Báo lỗi "Số lượng vượt quá tồn kho hiện tại".

### Biến thể hết hàng

- **Trường hợp:** stock_quantity = 0.
- **Xử lý:** Hiển thị "Hết hàng" và disable nút Add to Cart.

### Upload ảnh sai định dạng

- **Trường hợp:** upload file không phải jpg/png/webp.
- **Xử lý:** Báo lỗi "Chỉ hỗ trợ định dạng jpg/png/webp".

### Lỗi do đặt hàng đồng thời

- **Trường hợp:** 2 khách đặt cùng variant khi tồn kho thấp.
- **Xử lý:** Khách sau bị từ chối và hiển thị "Sản phẩm vừa hết hàng".

### Warehouse Staff xuất kho vượt tồn kho

- **Trường hợp:** xuất kho số lượng lớn hơn tồn hiện tại.
- **Xử lý:** Hệ thống từ chối thao tác và báo lỗi.

---

## 7. Giao diện (UI/UX Requirements)

### 7.1 Trang quản trị Admin

- Danh sách sản phẩm có thanh tìm kiếm và bộ lọc.
- Có nút Add, Edit, Delete, Enable/Disable.
- Trang chi tiết sản phẩm có các tab:
  - Thông tin sản phẩm
  - Biến thể
  - Ảnh sản phẩm
  - Tồn kho

### 7.2 Trang khách hàng (Customer)

- Trang chi tiết sản phẩm hiển thị biến thể rõ ràng.
- Khi chọn biến thể sẽ cập nhật:
  - Giá bán
  - Hình ảnh
  - Tồn kho realtime
- Nếu hết hàng thì không cho thêm vào giỏ.

### 7.3 Responsive

- Hệ thống phải hiển thị tốt trên Desktop và Mobile.
- Dropdown chọn biến thể dễ thao tác trên điện thoại.

---

## 8. Kết luận (Conclusion)

Chức năng **PIM-01 - Quản lý sản phẩm** là thành phần quan trọng của hệ thống Kochi Lens.  
Nó đảm bảo quản lý chính xác thông tin sản phẩm, biến thể, giá bán, VAT và tồn kho realtime.  
Chức năng này hỗ trợ hiệu quả cho hoạt động bán hàng trực tuyến, kho vận và kế toán.
