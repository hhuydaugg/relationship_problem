# Relationship Problem

Project này dùng để tìm các cặp doanh nghiệp có liên hệ thông qua:
- cùng một cá nhân làm việc ở nhiều doanh nghiệp
- quan hệ gia đình giữa các cá nhân đang làm ở các doanh nghiệp khác nhau

Đồng thời project có thể lọc bỏ các quan hệ đã khai báo trong `nclq.csv`.


## Input dữ liệu gốc

### `leader.csv`

Các cột được dùng:
- `cif_khcn`
- `name_khcn`
- `cif_khdn`
- `job_title`
- `name_khdn` (không bắt buộc)

Ghi chú:
- Nếu thiếu `name_khdn`, pipeline vẫn chạy được.
- Khi thiếu `name_khdn`, phần `description` sẽ dùng `cif_khdn` để hiển thị thay cho tên doanh nghiệp.
- Nếu có `name_khdn`, phần `description` sẽ ưu tiên dùng `name_khdn`.

### `nclq.csv`

Các cột được dùng:
- `CIF_NO`
- `CIF_NO_REL`
- `Mô tả MQH` hoặc `Mã MQH`

Ưu tiên xử lý quan hệ gia đình:
- Nếu có `Mã MQH`, pipeline ưu tiên dùng mã.
- Nếu không có `Mã MQH`, pipeline fallback sang `Mô tả MQH`.

Mapping hiện tại:
- `201` = `La vo, chong, cha, me, con voi ca nhan d`
- `211` = `La anh, chi, em voi ca nhan do`

## Hỗ trợ file Excel

Notebook raw-schema hỗ trợ cả:
- `leader.csv` / `nclq.csv`
- `leader.xlsx` / `nclq.xlsx`

Luồng xử lý:
- nếu có file `.csv` thì dùng trực tiếp
- nếu không có `.csv` mà có `.xlsx` thì tự chuyển sang `.csv`
- sau đó pipeline chạy trên `.csv`

## Logic nghiệp vụ

### TH1: `same_person`

Giữ các cặp doanh nghiệp khi cùng một cá nhân xuất hiện ở cả hai doanh nghiệp.

Điều kiện:
- chỉ cần `job_title` hợp lệ
- loại các giá trị rỗng / `nan`
- không giới hạn phải là chức vụ cao

### TH2: `family`

Giữ các cặp doanh nghiệp khi hai cá nhân có quan hệ gia đình.

Chỉ lấy các nhóm:
- `La vo, chong, cha, me, con voi ca nhan d`
- `La anh, chi, em voi ca nhan do`

Điều kiện thêm:
- cả hai người đều phải có chức vụ cao

Các nhóm chức vụ cao đang dùng:
- `Giam doc`
- `Pho giam doc`
- `Nguoi dai dien theo phap luat`
- `Nguoi duoc uy quyen cua giam doc/pho giam doc`
- `Ban lanh dao`
- `HDTV`
- `Ban giam doc`

Pipeline có xử lý dữ liệu chức danh bẩn:
- khác dấu
- viết tắt
- thừa khoảng trắng
- khác cách viết nhẹ

## Lọc quan hệ đã khai báo

Pipeline sinh ra 2 bộ output:

- `all_*`: toàn bộ quan hệ phát hiện được trước khi lọc khai báo
- `hidden_*`: quan hệ còn lại sau khi loại các quan hệ đã khai báo

Rule lọc:
- `company -> company` trong `nclq.csv` sẽ loại đúng cặp doanh nghiệp có hướng đó trong `hidden_*`
- `person -> person` trong `nclq.csv` sẽ loại các link `family` tương ứng trong `hidden_*`

Lưu ý:
- `A -> B` và `B -> A` được giữ là hai quan hệ khác nhau
- nếu chỉ khai báo `A -> B` thì không tự động loại `B -> A`

## Output

### Theo cặp doanh nghiệp

- `all_relationships_pairs_raw_schema.csv`
- `hidden_relationships_pairs_raw_schema.csv`

Cột chính:
- `cif_khdn_A`, `name_khdn_A`
- `cif_khdn_B`, `name_khdn_B`
- `shared_count`
- `shared_individuals`
- `shared_individual_names`
- `roles_in_A`, `roles_in_B`
- `high_roles_in_A`, `high_roles_in_B`
- `high_role_groups_in_A`, `high_role_groups_in_B`
- `connection_types`
- `family_relationships`
- `description`

### Theo cá nhân

- `all_relationships_by_person_raw_schema.csv`
- `hidden_relationships_by_person_raw_schema.csv`

Cột chính:
- `cif_khcn`
- `name_khcn`
- `danh_sach_doanh_nghiep`
- `so_luong_doanh_nghiep`
- `high_role_groups`
- `description`

### File chẩn đoán

- `nclq_declared_diagnostics_raw_schema.csv`

File này cho biết mỗi dòng trong `nclq.csv` được resolve thành loại nào:
- `company_company`
- `person_person`
- `company_person`
- `person_company`
- `ambiguous_or_unmatched`

## Cách chạy trên máy mới

Thư viện cần cài:
- `pandas`
- `openpyxl`
- `jupyter` nếu chạy notebook

Lệnh cài:

```bash
pip install pandas openpyxl jupyter
```

## Cách dùng khuyến nghị

Với dữ liệu gốc chưa mã hóa:
1. đặt `leader.csv` và `nclq.csv` cùng thư mục notebook
2. nếu chỉ có Excel thì đặt `leader.xlsx` và `nclq.xlsx`
3. chạy [relationship_pipeline_end_to_end_raw_schema copy.ipynb]
