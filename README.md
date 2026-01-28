# AimeePowerBI

[![Power BI](https://img.shields.io/badge/Power%20BI-Desktop-yellow)](https://powerbi.microsoft.com/)

**English** | [中文说明](#中文说明)

## 📊 Overview

This Power BI project analyzes job postings, applicant stages, and compensation trends through interactive dashboards to support data-driven recruitment and workforce planning. The project demonstrates advanced Power BI capabilities including data transformation with Power Query (M language) and sophisticated analytics using DAX.

**Video Tutorial:** https://www.youtube.com/watch?v=FwjaHCVNBWA

## 📁 Project Files

The repository contains three Power BI report files:

### 1. **MyfirstPBI.pbix**
- Entry-level Power BI project
- Basic visualizations and data connections
- Introduction to Power BI fundamentals

### 2. **AimeeadvancePBI.pbix**
- Advanced analytics and reporting
- Complex DAX calculations
- Advanced data modeling techniques

### 3. **AimeeData_Jobs_Dashboard_2.0.pbix** (Main Project)
- Comprehensive jobs and recruitment analytics dashboard
- Version 2.0 with enhanced features and optimizations
- Production-ready dashboard with full data model

## 📥 Data Source

**Google Drive folder:** https://drive.google.com/drive/folders/1DsLqC5OZt6fnUV8UksMhKnjWrNEG89xz?usp=drive_link

The data includes:
- Job posting details
- Applicant pipeline information
- Compensation and salary data
- Recruitment funnel metrics

## 🗂️ Data Model

The semantic model includes the following tables:

### 1) `job_postings_flat`
Detailed job posting information including:
- Job title and description
- Location and remote options
- Source and posting date
- Schedule type (full-time, part-time, etc.)
- Salary ranges
- Company information
- Required skills

### 2) `waterfalldate`
Compensation analysis table with:
- Compensation components (base salary, bonus, benefits, etc.)
- Amount breakdowns
- Time-based trends
- Pay structure analysis

### 3) `Table`
Applicant pipeline stages table featuring:
- Recruitment funnel stages
- Applicant counts at each stage
- Conversion metrics
- Pipeline performance indicators

## 🔄 Data Transformation (M Language / Power Query)

This project utilizes Power Query M language for ETL (Extract, Transform, Load) operations:

### Key Transformation Steps:

1. **Data Import and Connection**
   - Connect to external data sources
   - Load data from multiple files/tables

2. **Data Cleaning**
   - Remove duplicates
   - Handle null values
   - Standardize data formats
   - Trim whitespace

3. **Data Transformation**
   - Split columns (e.g., separating location into city/state)
   - Merge queries to combine related data
   - Unpivot columns for analysis
   - Create calculated columns

4. **Data Type Conversion**
   - Convert text to dates
   - Format currency fields
   - Standardize number formats

### Example M Queries:

```m
// Example: Clean and transform job postings data
let
    Source = Excel.Workbook(File.Contents("data.xlsx"), null, true),
    JobPostings = Source{[Item="job_postings",Kind="Sheet"]}[Data],
    PromotedHeaders = Table.PromoteHeaders(JobPostings),
    ChangedTypes = Table.TransformColumnTypes(PromotedHeaders, {
        {"posting_date", type date},
        {"salary_min", type number},
        {"salary_max", type number}
    }),
    RemovedDuplicates = Table.Distinct(ChangedTypes, {"job_id"}),
    FilteredRows = Table.SelectRows(RemovedDuplicates, each [posting_date] <> null)
in
    FilteredRows
```

## 📊 DAX Measures and Calculations

The project includes sophisticated DAX (Data Analysis Expressions) measures for analytics:

### Key DAX Measures:

#### 1. **Total Job Postings**
```dax
Total Jobs = COUNTROWS('job_postings_flat')
```

#### 2. **Average Salary**
```dax
Avg Salary = 
AVERAGE('job_postings_flat'[salary_avg])
```

#### 3. **Conversion Rate**
```dax
Conversion Rate = 
DIVIDE(
    CALCULATE(COUNT('Table'[Applicants]), 'Table'[Stage] = "Hired"),
    CALCULATE(COUNT('Table'[Applicants]), 'Table'[Stage] = "Applied"),
    0
)
```

#### 4. **YoY Growth**
```dax
YoY Growth % = 
VAR CurrentYear = CALCULATE([Total Jobs], YEAR('job_postings_flat'[posting_date]) = YEAR(TODAY()))
VAR PreviousYear = CALCULATE([Total Jobs], YEAR('job_postings_flat'[posting_date]) = YEAR(TODAY()) - 1)
RETURN
DIVIDE(CurrentYear - PreviousYear, PreviousYear, 0)
```

#### 5. **Time Intelligence - Running Total**
```dax
Running Total Jobs = 
CALCULATE(
    [Total Jobs],
    FILTER(
        ALL('job_postings_flat'[posting_date]),
        'job_postings_flat'[posting_date] <= MAX('job_postings_flat'[posting_date])
    )
)
```

#### 6. **Compensation Analysis**
```dax
Total Compensation = 
SUMX(
    'waterfalldate',
    'waterfalldate'[Amount]
)
```

### Calculated Columns:

```dax
// Example: Salary Range Category
Salary Range = 
SWITCH(
    TRUE(),
    'job_postings_flat'[salary_avg] < 50000, "Entry Level",
    'job_postings_flat'[salary_avg] < 100000, "Mid Level",
    'job_postings_flat'[salary_avg] < 150000, "Senior Level",
    "Executive Level"
)
```

## 🔗 Relationships and Data Model

The data model uses the following relationships:

- **One-to-Many:** `job_postings_flat` → `Table` (via job_id or applicant tracking)
- **One-to-Many:** Date dimension → fact tables (for time intelligence)
- **Star Schema:** Fact tables connected to dimension tables for optimal performance

## 🎯 Key Features

- ✅ Interactive dashboards with drill-through capabilities
- ✅ Advanced data transformations using M language
- ✅ Complex DAX measures and KPIs
- ✅ Time intelligence calculations
- ✅ Recruitment funnel analysis
- ✅ Compensation trend analysis
- ✅ Job market insights
- ✅ Filter and slicer interactions

## 🚀 Getting Started

### Prerequisites

- **Microsoft Power BI Desktop** (latest version recommended)
- Download from: https://powerbi.microsoft.com/desktop/

### Installation Steps

1. **Clone this repository:**
   ```bash
   git clone https://github.com/hahAI111/AimeePowerBI.git
   ```

2. **Download the data:**
   - Access the Google Drive folder linked above
   - Download all data files to your local machine

3. **Open the Power BI report:**
   - Launch Power BI Desktop
   - Open the desired `.pbix` file from the repository
   - Choose the main file: `AimeeData_Jobs_Dashboard_2.0.pbix`

4. **Update data connections:**
   - Go to Transform Data → Data Source Settings
   - Update file paths to point to your downloaded data location
   - Click "Refresh" to reload the data

5. **Explore the dashboard:**
   - Use slicers and filters to interact with the data
   - Click on visuals to cross-filter
   - Use drill-through features for detailed analysis

## 📖 Usage Guide

### Navigation
- Use the tabs at the bottom to switch between report pages
- Click on charts to filter related visuals
- Right-click on data points for drill-through options

### Filters
- Use date slicers to analyze specific time periods
- Apply location filters for geographic analysis
- Filter by job categories or compensation ranges

### Key Insights
- **Recruitment Funnel:** Track applicant progression through stages
- **Salary Trends:** Analyze compensation patterns over time
- **Job Market Analysis:** Identify trending skills and locations
- **Performance Metrics:** Monitor hiring KPIs and conversion rates

## 🛠️ Technical Details

### Power Query (M) Highlights
- Custom functions for data cleaning
- Query folding optimization
- Parameter-based data loading
- Advanced merge and append operations

### DAX Highlights
- Time intelligence functions (YTD, MTD, YoY)
- Context transition with CALCULATE
- Iterator functions (SUMX, AVERAGEX)
- Dynamic filtering and ranking

### Performance Optimization
- Star schema data model
- Aggregated tables for large datasets
- Optimized DAX measures
- Reduced visual complexity

## 📊 Dashboard Components

The main dashboard includes:
- **Overview Page:** High-level KPIs and trends
- **Job Analysis Page:** Detailed job posting metrics
- **Recruitment Funnel Page:** Applicant pipeline visualization
- **Compensation Page:** Salary analysis and waterfall charts
- **Trends Page:** Time-based analysis and forecasting

## 🤝 Contributing

This is a demonstration project. Feel free to:
- Fork the repository
- Experiment with the data model
- Add new visualizations
- Enhance DAX measures

## 📝 License

Provided for demonstration and educational purposes. See `LICENSE` for details.

---

## 中文说明

## 📊 项目概述

这是一个 Power BI 项目，用于分析职位发布、应聘者阶段和薪酬趋势，通过交互式仪表板支持数据驱动的招聘和人力规划。该项目展示了 Power BI 的高级功能，包括使用 Power Query（M 语言）进行数据转换和使用 DAX 进行复杂分析。

## 📁 项目文件说明

### 1. **MyfirstPBI.pbix**
- 入门级 Power BI 项目
- 基础可视化和数据连接
- Power BI 基础知识介绍

### 2. **AimeeadvancePBI.pbix**
- 高级分析和报告
- 复杂的 DAX 计算
- 高级数据建模技术

### 3. **AimeeData_Jobs_Dashboard_2.0.pbix**（主项目）
- 综合招聘分析仪表板
- 2.0 版本，具有增强功能和优化
- 生产就绪的仪表板，包含完整数据模型

## 🗂️ 数据模型

数据模型包含以下表：

1. **job_postings_flat** - 职位发布详细信息
2. **waterfalldate** - 薪酬组成分析
3. **Table** - 应聘者流程阶段

## 🔄 数据转换（M 语言）

项目使用 Power Query M 语言进行 ETL 操作：
- 数据导入和连接
- 数据清理（去重、处理空值、标准化格式）
- 数据转换（拆分列、合并查询、列转行）
- 数据类型转换

## 📊 DAX 度量值和计算

项目包含复杂的 DAX 度量值：
- 职位总数统计
- 平均薪资计算
- 转化率分析
- 同比增长率
- 累计统计
- 薪酬分析

## 🚀 使用说明

1. 下载并安装 Power BI Desktop
2. 克隆此仓库
3. 从 Google Drive 下载数据文件
4. 打开 .pbix 文件
5. 更新数据源连接
6. 刷新数据并开始分析

## 主要功能

- ✅ 交互式仪表板
- ✅ M 语言数据转换
- ✅ DAX 度量值和 KPI
- ✅ 时间智能计算
- ✅ 招聘漏斗分析
- ✅ 薪酬趋势分析

---

**Author:** Aimee  
**Last Updated:** 2026-01
