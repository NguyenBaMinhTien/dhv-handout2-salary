# Data Quality Report

Dataset: Ask A Manager Salary Survey 2021

## 1. Tổng Quan Dataset Gốc

Dataset gốc có 28,216 hàng và 18 cột. Đây là dữ liệu khảo sát tự khai báo nên có nhiều vấn đề thực tế: missing value hợp lệ, free-text không chuẩn hóa, salary có outlier cực mạnh, và nhiều categorical columns cần encoding.

| Chỉ số | Trước xử lý | Sau pipeline |
|---|---:|---:|
| Số hàng | 28,216 | 28,214 |
| Số cột | 18 | 47 |
| Tổng null | 87,320 | 79,349 |
| Duplicate toàn hàng | 0 | 0 |
| `annual_salary` dtype | object | float64 |
| `timestamp` dtype | object | datetime64 |

## 2. Vấn Đề Phát Hiện

| Vấn đề | Cột ảnh hưởng | Mức độ | Ví dụ cụ thể |
|---|---|---|---|
| Missing hợp lệ ở free-text | `currency_other`, `income_context`, `job_context` | Minor | Người không chọn Other currency không cần điền `currency_other` |
| Missing có ý nghĩa | `add_compensation` | Major | Null nhiều khả năng là không có bonus |
| Dtype sai | `annual_salary`, `timestamp` | Critical | `"55,000"` không thể tính mean nếu vẫn là string |
| Chuỗi không chuẩn | `country`, `job_title`, `industry` | Critical | `USA`, `US`, `U.S.` cùng là United States |
| Outlier salary | `annual_salary` | Major | USD salary bằng 0 hoặc hàng triệu |
| High-cardinality categorical | `country`, `city`, `job_title` | Major | One-hot toàn bộ country tạo nhiều cột sparse |

## 3. Quyết Định Xử Lý

| Vấn đề | Quyết định | Phương án khác | Rủi ro nếu không xử lý |
|---|---|---|---|
| `add_compensation` null | Fill 0 | Fill median | Median sẽ bịa bonus cho người không có bonus |
| `education`, `city`, `industry` null | Fill `Unknown` | Drop hàng | Mất dữ liệu không cần thiết vì tỉ lệ thiếu thấp |
| `country` null | Drop 2 hàng | Fill `Unknown` | Country là biến quan trọng cho location/currency |
| Timestamp parse lỗi | Tạo `survey_month_interp` bằng interpolate | Drop hàng | Có thể mất hàng dù chỉ lỗi timestamp |
| Salary dạng string | Xóa dấu phẩy/ký hiệu `$`, convert float | Giữ string | Không tính thống kê hoặc model được |
| Salary outlier USD | Winsorize p1/p99 | Drop outlier | Drop dễ làm mất nhóm lương cao/thấp hợp lệ |
| `country` high-cardinality | Binary `is_us` + target encoding demo | Full OHE | Dataset phình, nhiều cột sparse |

## 4. Quyết Định Khó

Vấn đề khó nhất là xử lý outlier trong `annual_salary`.

**Phương án 1: Drop outlier ngoài IQR hoặc p1/p99.**

Ưu điểm là phân phối sạch hơn và mean ít bị kéo bởi extreme values. Nhược điểm là có thể xóa các quan sát hợp lệ như executive, senior engineer, lawyer hoặc các nhóm compensation rất cao.

**Phương án 2: Winsorize tại 1st/99th percentile.**

Ưu điểm là giữ lại người trả lời và giảm ảnh hưởng của giá trị cực trị. Nhược điểm là làm méo nhẹ hai đuôi phân phối.

**Quyết định:** chọn winsorize cho salary USD. Lý do là dữ liệu tự khai báo không đủ bằng chứng để kết luận các giá trị cao đều là lỗi. Drop có thể gây bias khi phân tích salary theo ngành hoặc job title.

## 5. Hạn Chế Còn Lại

- Salary chưa quy đổi toàn bộ về cùng một currency, nên phân tích global salary vẫn cần lọc theo currency hoặc bổ sung tỷ giá.
- `job_title` là free-text rất đa dạng; mapping thủ công chỉ xử lý một phần các title tương đồng.
- Target Encoding trong notebook là minh họa. Nếu dùng cho mô hình thật cần tính trên train set hoặc dùng cross-validation để tránh leakage.
- `add_compensation = 0` là giả định hợp lý nhưng vẫn có thể sai với người bỏ trống vì không muốn khai báo.
- Survey là dữ liệu tự khai báo, vì vậy vẫn có lỗi nhập liệu không thể xác minh hoàn toàn.
