
# AimeePowerBI

[![Power BI](https://img.shields.io/badge/Power%20BI-Dashboard-F2C811?logo=powerbi&logoColor=000)](https://powerbi.microsoft.com/)
[![DAX](https://img.shields.io/badge/DAX-Measures-5E5E5E)](#-dax-measures-and-calculations--dax-%E6%8E%AA%E6%96%BD%E4%B8%8E%E8%AE%A1%E7%AE%97)
[![Power Query](https://img.shields.io/badge/Power%20Query-M%20Language-2E77BC)](#-data-transformation-m-language--power-query--%E6%95%B0%E6%8D%AE%E8%BD%AC%E6%8D%A2)
[![Status](https://img.shields.io/badge/Status-Documentation%20Only-success)](#)

Demo video: https://www.youtube.com/watch?v=FwjaHCVNBWA  

---

## Project Overview (English)
This Power BI project provides interactive dashboards for analyzing **job postings**, **applicant funnel stages**, and **compensation trends**. It is designed to help users quickly explore key metrics and patterns for data-driven decision-making in recruitment and workforce planning.

## 项目概览（中文）
本项目使用 Power BI 构建交互式仪表板，用于分析**岗位发布信息**、**招聘漏斗阶段**以及**薪酬趋势**，帮助用户快速洞察关键指标与变化规律，从而支持招聘与人力规划的数据驱动决策。

---

## Data / 数据
Google Drive folder（数据下载）:  
https://drive.google.com/drive/folders/1DsLqC5OZt6fnUV8UksMhKnjWrNEG89xz?usp=drive_link

---

## File Descriptions / 文件说明
- **`MyfirstPBI.pbix`** — Entry-level project / 入门项目  
- **`AimeeadvancePBI.pbix`** — Advanced analytics / 高阶分析项目  
- **`AimeeData_Jobs_Dashboard_2.0.pbix`** — Main production dashboard / 主仪表板（生产版）

> Note: Large `.pbix` files may exceed GitHub web upload limits. Consider Git LFS or external storage links.

---

## 🗂️ Data Model Documentation / 数据模型说明

### Tables / 表
#### 1) `job_postings_flat`
**Purpose / 用途**: job posting details / 岗位发布明细  
**Typical columns / 常见字段（示例）**:
- `job_id` (PK candidate)  
- `company_id`, `company_name`
- `job_title`, `job_title_short`
- `job_location`, `job_country`, `search_location`
- `job_schedule_type`, `job_work_from_home`
- `job_posted_date`
- `salary_hour_avg`, `salary_year_avg`, `salary_rate`
- `job_via`
- `job_health_insurance`, `job_no_degree_mention`

#### 2) `waterfalldate`
**Purpose / 用途**: compensation components & amounts / 薪酬结构组件与金额  
**Typical columns / 常见字段（示例）**:
- `component`（薪酬组件）
- `amount`（金额）
- `date`（日期/期间，如有）

#### 3) `Table` (Applicants)
**Purpose / 用途**: recruitment funnel stages / 招聘漏斗阶段  
**Typical columns / 常见字段（示例）**:
- `stage`（阶段）
- `applicants`（人数）

### Keys & Relationships / 主外键与关系（示例模板）
> Adjust to match your actual model names/keys if different.

- `job_postings_flat[job_id]` — primary identifier / 主键候选  
- Dimension-style tables (e.g., schedule/company/date) should filter into `job_postings_flat` (single direction recommended).

**Recommended mapping / 建议关系：**
- Dimensions (1) → Fact (*) filtering  
- Avoid bidirectional filters unless required (prevents ambiguous paths and unexpected totals).

---

## 🔄 Data Transformation (M Language / Power Query) / 数据转换（Power Query / M）

### ETL Workflow (High-level) / ETL 流程（概览）
1. **Ingest / 导入**: Connect to CSV/Excel/Folder sources  
2. **Clean / 清洗**: remove nulls, trim text, standardize values  
3. **Type casting / 类型转换**: salary/date columns to numeric/date  
4. **Enrich / 派生**: create normalized fields (e.g., `job_title_short`)  
5. **Load / 加载**: load cleaned tables into the semantic model

### Example M Query (Cleaning & Types) / M 示例（清洗与类型）
```powerquery
let
    Source = Excel.CurrentWorkbook(){[Name="job_postings_flat"]}[Content],
    #"Trim Text" = Table.TransformColumns(Source, {
        {"job_title", Text.Trim, type text},
        {"job_location", Text.Trim, type text}
    }),
    #"Standardize Schedule" = Table.ReplaceValue(#"Trim Text","FT","Full-time",Replacer.ReplaceText,{"job_schedule_type"}),
    #"Change Types" = Table.TransformColumnTypes(#"Standardize Schedule",{
        {"job_posted_date", type date},
        {"salary_hour_avg", type number},
        {"salary_year_avg", type number}
    }),
    #"Remove Empty Titles" = Table.SelectRows(#"Change Types", each [job_title] <> null and [job_title] <> "")
in
    #"Remove Empty Titles"
```

### Step-by-step (Typical) / 分步说明（常见做法）
- **Promoted Headers / 提升标题行**  
- **Changed Type / 更改数据类型**（date/number/text）  
- **Replace Values / 替换值**（缩写、错别字、统一分类）  
- **Remove Errors / 删除错误**  
- **Remove Duplicates / 去重**  
- **Add Columns / 添加派生列**（如短标题、薪资区间、远程标记等）

---

## 📊 DAX Measures and Calculations / DAX 措施与计算

> Replace table/column names to match your model (e.g., `job_postings_flat`, `Table`, `waterfalldate`).

### 1) Total Jobs / 岗位总数
```DAX
Total Jobs =
DISTINCTCOUNT ( job_postings_flat[job_id] )
```

### 2) Average Salary (AVERAGEX) / 平均薪资（用 AVERAGEX）
Example: average of hourly average, ignoring blanks  
```DAX
Average Salary (Hour) =
AVERAGEX (
    FILTER ( job_postings_flat, NOT ISBLANK ( job_postings_flat[salary_hour_avg] ) ),
    job_postings_flat[salary_hour_avg]
)
```

### 3) Conversion Rate (Funnel) / 转化率（漏斗分析）
Example assumes `Table` has `[stage]` and `[applicants]`  
```DAX
Applicants (Total) =
SUM ( 'Table'[applicants] )

Applicants (Interview) =
CALCULATE ( [Applicants (Total)], 'Table'[stage] = "Interview" )

Applicants (Applied) =
CALCULATE ( [Applicants (Total)], 'Table'[stage] = "Applied" )

Conversion Rate (Applied→Interview) =
DIVIDE ( [Applicants (Interview)], [Applicants (Applied)] )
```

### 4) YoY Growth (Year-over-year) / 同比增长
Requires a proper Date table marked as Date table (recommended).  
```DAX
Jobs YoY % =
VAR CurrentJobs = [Total Jobs]
VAR PriorJobs =
    CALCULATE ( [Total Jobs], DATEADD ( 'Date'[Date], -1, YEAR ) )
RETURN
DIVIDE ( CurrentJobs - PriorJobs, PriorJobs )
```

### 5) Running Total (Time intelligence) / 累计值
```DAX
Jobs Running Total =
CALCULATE (
    [Total Jobs],
    FILTER ( ALL ( 'Date'[Date] ), 'Date'[Date] <= MAX ( 'Date'[Date] ) )
)
```

### 6) Total Compensation / 总薪酬（组件合计示例）
If `waterfalldate` stores compensation components in `[amount]`:  
```DAX
Total Compensation =
SUM ( waterfalldate[amount] )
```

### Calculated Column Example (Row Context) / 计算列示例（行上下文）
Example: salary band based on hourly salary  
```DAX
Salary Band =
VAR s = job_postings_flat[salary_hour_avg]
RETURN
SWITCH (
    TRUE (),
    ISBLANK ( s ), "Unknown",
    s < 25, "Low",
    s < 60, "Mid",
    "High"
)
```

---

## 🚀 Getting Started / 快速开始

### Prerequisites / 前置条件
- Power BI Desktop (latest recommended)
- Access to the dataset (Google Drive link above)

### Steps / 使用步骤
1. Download data from Google Drive / 下载数据  
2. Open a `.pbix` file / 打开任一 `.pbix`  
3. Update data source paths if prompted / 如提示，更新数据源路径  
4. Click **Refresh** / 点击刷新  
5. Use slicers/filters to explore:
   - Job title / 职位
   - Company / 公司
   - Schedule type / 工作类型
   - Location / 地点
   - Date range / 时间范围


---

## License / 许可
Provided for demonstration purposes. See `LICENSE` for details.  
本项目仅用于演示，请查看 `LICENSE` 获取更多信息。
```
