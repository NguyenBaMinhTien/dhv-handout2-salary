# DHV Handout 2 - Salary Survey Data Cleaning

Repo này hoàn thành bài Data Handling & Visualization - Handout 2: Xử lý & Làm sạch dữ liệu.

## Files

| File | Mô tả |
|---|---|
| `salary_cleaning.ipynb` | Notebook xử lý đầy đủ: missing values, duplicates, dtype, outliers, string normalization, normalization, encoding, feature engineering, pipeline |
| `data_quality_report.md` | Data Quality Report 5 phần theo handout |
| `data/salary_cleaned.csv` | Dataset sạch export từ `clean_data(df_raw)` |
| `data/Ask A Manager Salary Survey 2021 (Responses) - Form Responses 1.csv` | Dataset gốc |

## Cách Chạy

1. Mở `salary_cleaning.ipynb`.
2. Chọn `Restart Kernel`.
3. Chạy `Run All`.
4. Cell cuối sẽ chạy unit tests và export `data/salary_cleaned.csv`.

## Nội Dung Đã Xử Lý

- Missing values: phân loại MCAR/MAR/MNAR, dùng fillna, drop và interpolate.
- Duplicates: kiểm tra duplicate toàn hàng và duplicate theo soft key.
- Data types: convert salary sang `float64`, timestamp sang `datetime64`, industry sang `category`.
- Outliers: phát hiện bằng IQR và Z-score cho 2 cột numeric, xử lý salary USD bằng winsorize p1/p99.
- String normalization: chuẩn hóa country, job title, industry, city; dùng mapping dict và fuzzy matching.
- Normalization: so sánh Min-Max, StandardScaler và RobustScaler.
- Encoding: ordinal encoding, binary flags, one-hot top industry và target encoding cho country.
- Feature engineering: `salary_per_exp_year`, `total_comp`, `age_group`, `survey_hour`, `is_weekend`, `survey_day_of_week`.
- Pipeline: hàm `clean_data(df)` có docstring, không sửa input in-place, có unit tests.

## Kết Quả Pipeline

Pipeline cuối tạo `df_clean` với 28,214 hàng và 47 cột. Các unit tests kiểm tra shape, null quan trọng, dtype, encoding columns và feature engineering columns đều pass.
