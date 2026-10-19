# Retail Operations: KPI Dictionary & Data Quality Contract

## 📌 Project Overview
This project translates ambiguous e-commerce and retail operations data requirements into measurable business definitions, automated data profiling checks, and a formal Data Quality Contract. 

Using raw transactional order data (`retail-orders-raw.csv`), this project profiles data anomalies across completeness, uniqueness, validity, and consistency, establishing automated failure thresholds and escalation protocols for data engineering and business teams.

---

## 🔗 Quick Links
* **Google Colab Executable Notebook:** [View Interactive Colab Notebook](https://colab.research.google.com/drive/1Kd_2y_X98zIU0en5PWzW9Xq6kTVbVsbM?usp=sharing)
* **Dataset Files:** `retail-orders-raw.csv`, `retail-data-dictionary.csv`

---

## 📊 Business KPI Dictionary
Below are the key operational metrics defined to track business performance and order fulfillment health:

| Metric Name | Formula / Logic | Grain | Filters / Conditions | Owner | Refresh Cadence |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Gross Order Revenue** | `SUM(quantity * unit_price)` | Per Order ID | Exclude canceled/refunded orders | Sales Lead | Daily |
| **Net Revenue** | `SUM((quantity * unit_price) * (1 - discount_pct/100))` | Per Order ID | Payment Status = 'Paid' | Finance Lead | Daily |
| **Average Order Value (AOV)** | `Net Revenue / COUNT(DISTINCT order_id)` | Overall Aggregate | Successful orders only | Marketing Lead | Weekly |
| **Order Completion Rate** | `(COUNT(Status = 'Paid') / COUNT(order_id)) * 100` | Monthly | All recorded order attempts | Operations Lead | Daily |
| **Discount Leakage Rate** | `SUM(discount_pct * unit_price * quantity) / Gross Revenue` | Category Level | Discount > 0% | Pricing Team | Weekly |
| **Refund Rate** | `(COUNT(Status = 'Refunded') / COUNT(order_id)) * 100` | Regional / City | All processed orders | Support Lead | Monthly |
| **Customer Segment Yield** | `Net Revenue / COUNT(DISTINCT customer_segment)` | Customer Segment | Valid segment entries | Growth Lead | Weekly |
| **Unit Sales Volume** | `SUM(quantity)` | Category / Product | Valid numeric quantities (> 0) | Supply Chain | Daily |

---

## 🔍 Data Quality Profiling Results
Automated Python profiling checks identified several critical data anomalies in `retail-orders-raw.csv`:

1. **Completeness Issues:**
   * Missing `order_date` (`RT-1011`)
   * Missing `city` (`RT-1005`)
   * Missing `discount_pct` (`RT-1003`)
2. **Uniqueness Violations:**
   * Exact duplicate `order_id` records detected (`RT-1004` appears twice)
3. **Validity Errors:**
   * Invalid negative quantity (`RT-1006` with `quantity = -1`)
   * Invalid non-numeric string (`RT-1008` with `quantity = "two"`)
   * Out-of-bounds discount rate (`RT-1007` with `discount_pct = 105.0%`)
   * Date syntax and logical anomalies (`RT-1002` formatted as `03/01/2026`, `RT-1006` with invalid month `2026-13-10`)
4. **Consistency Anomalies:**
   * Un-normalized casing in `customer_segment` (`"student"` vs `"Student"`)
   * Un-normalized casing in `payment_status` (`"paid"` vs `"Paid"`)

---

## 📄 Data Quality Contract & Escalation Plan

### 1. Quality Thresholds Agreement
* **Primary Key Uniqueness:** **0% tolerance** for duplicate `order_id` values. Duplicate records immediately trigger pipeline halt.
* **Completeness:** **0% tolerance** for missing critical fields (`order_date`, `city`, `payment_status`). Missing optional fields (e.g., `discount_pct`) must remain **< 2%** and default to `0.0`.
* **Validity:** 
  * `quantity` must strictly be a positive integer greater than zero (`> 0`).
  * `discount_pct` must strictly fall between `0.0` and `100.0`.
  * `order_date` must conform strictly to standard ISO format (`YYYY-MM-DD`).
* **Consistency:** Categorical attributes (`customer_segment` and `payment_status`) must be normalized to Title Case prior to downstream ingestion.

### 2. Automated Escalation Matrix
| Severity Level | Trigger Condition | Automated Action | Business Escalation Target |
| :--- | :--- | :--- | :--- |
| **Critical (Blocker)** | Duplicate `order_id`, missing `order_date`, or invalid date syntax | Halt ETL pipeline & quarantine raw batch | Data Engineering Lead |
| **High** | Negative `quantity`, discount $> 100\%$, or non-numeric strings in numeric columns | Route affected rows to exception quarantine table | Operations & Billing Support |
| **Low / Warning** | Un-normalized casing (`"paid"` vs `"Paid"`) or missing `discount_pct` | Auto-correct casing & impute default missing values | Automated Pipeline Logger |

---

## 🛠️ How to Run the Code
1. Clone or download this repository.
2. Open the Google Colab Notebook via the link above or execute the `.ipynb` notebook locally in Jupyter.
3. Ensure `retail-orders-raw.csv` and `retail-data-dictionary.csv` are uploaded in the root execution directory.
4. Run all code cells sequentially to view profiling outputs and data validation assertions.
