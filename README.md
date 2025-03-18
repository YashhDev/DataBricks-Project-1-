# DataBricks-Project-1
---

# 🚀 Incremental Data Load in Delta Table using Event-Driven Architecture

This project demonstrates **Incremental Data Load and Merge** in **Delta Tables** using an **Event-Driven Architecture** approach. It's designed to work seamlessly with **Google Cloud Storage**, **Databricks Workflows**, and **Delta Lake** for efficient data processing.

---

## 📚 Table of Contents
- [Project Overview](#project-overview)
- [Architecture Diagram](#architecture-diagram)
- [Process Flow](#process-flow)
- [Merge Logic](#merge-logic)
- [SCD Type 1 & Type 2](#scd-type-1--type-2)
- [Transaction Aggregation Example](#transaction-aggregation-example)
- [Tech Stack](#tech-stack)
- [Key Takeaways](#key-takeaways)

---

## 📌 Project Overview

This project focuses on:
- Loading data incrementally from **Google Storage Buckets**.
- Performing **Merge (Upsert)** operations into **Delta Tables** using **Databricks**.
- Supporting **Slowly Changing Dimensions (SCD1 & SCD2)**.
- Automating workflows through **Databricks Workflows**.

---

## 🏗️ Architecture Diagram

```
┌──────────────┐
│   CLIENT     │
└─────┬────────┘
      │
      ▼
┌───────────────────────────┐
│ Google Storage (BUCKET-1) │
│  ┌────────┐    ┌───────┐  │
│  │ SOURCE │    │ARCHIVE│  │
└─────┬─────────────────────┘
      │
      ▼
┌───────────────────────────────┐
│      DATABRICKS WORKFLOW      │
│  ┌────────┐       ┌────────┐  │
│  │ TASK 1 │       │ TASK 2 │  │
│  │ Load   │       │ Merge  │  │
│  │ in     │──────▶│ Target │  │
│  │ Stage  │       │ Table  │  │
└─────┬────────────────────────┘
      │
      ▼
┌──────────────────────────┐
│   Stage Table (Delta)    │
└──────────────────────────┘
      │
      ▼
┌──────────────────────────┐
│   Target Table (Delta)   │
└──────────────────────────┘
```

---

## 🔄 Process Flow

1. **Client Uploads Files**  
   - Files are uploaded to **Google Cloud Storage** → `/source` folder.

2. **Databricks Workflow Triggers**
   - **Task 1:** Loads data from `/source` to **Staging Delta Table**.
   - **Task 2:** Merges data from **Stage Table** to **Target Table**.

3. **Data Movement**
   - Processed files are moved to the `/archive` folder in Google Storage.

---

## 📝 Merge Logic

Example: **MERGE INTO** Target Table (Incremental Load with Conditions)

### Source & Target Table Example

| eid | name | dept | salary | status |
|-----|------|------|--------|--------|
| 1   | A    | HR   | 500    | Y      |
| 2   | B    | IT   | 600    | Y      |
| 3   | C    | HR   | 300    | N      |
| 4   | D    | IT   | 500    | Y      |

### Merge SQL Query
```sql
MERGE INTO target
USING (SELECT * FROM source)
ON source.eid = target.eid

WHEN MATCHED AND source.status = 'Y' AND source.dept = 'HR' THEN
  UPDATE SET target.salary = source.salary + 1000

WHEN MATCHED AND source.status = 'Y' AND source.dept = 'IT' THEN
  UPDATE SET target.salary = source.salary + 500

WHEN MATCHED AND source.status = 'N' THEN
  DELETE

WHEN NOT MATCHED THEN
  INSERT (*)
```

---

## 🗂️ SCD Type 1 & Type 2

### 🔹 SCD Type 1 - Overwrite  
- Directly updates the existing record.
- No historical tracking.

---

### 🔸 SCD Type 2 - History Tracking  
- Maintains full history of data changes.
- Uses **valid_from** and **valid_to** for tracking.

#### SCD2 Example Table
| custid | name | address | age | active | valid_from   | valid_to     |
|--------|------|---------|-----|--------|--------------|--------------|
| 100    | A    | XY-2    | 30  | Y      | 12-Jan-2024  | 14-Jan-2024 |
| 100    | A    | XY-2    | 32  | Y      | 15-Jan-2024  | 9999        |

#### SCD2 Logic (SQL Concept)
```sql
target.valid_to = source.valid_from - 1
```

---

## 💰 Transaction Aggregation Example

### Transaction Table
| mid  | trans_amt | status  |
|------|-----------|---------|
| M001 | 100       | Success |
| M001 | 200       | Success |
| M001 | 100       | Failed  |
| M002 | 50        | Success |
| M002 | 100       | Success |
| M001 | 50        | Failed  |

### Aggregated Output
| mid  | success_amt | failed_amt |
|------|-------------|------------|
| M001 | 300         | 150        |
| M002 | 150         | 0          |

#### Logic
```text
Success Amount = SUM(CASE WHEN status = 'Success' THEN trans_amt ELSE 0 END)
Failed Amount  = SUM(CASE WHEN status = 'Failed' THEN trans_amt ELSE 0 END)
```

---

## 🛠️ Tech Stack
- **Google Cloud Storage** (Source / Archive Buckets)
- **Databricks Workflow** (Orchestration)
- **Delta Lake / Delta Tables** (Data Store)
- **SQL Merge Statements** (Upsert, Delete, Insert)
- **Event-Driven Processing**

---

## ✅ Key Takeaways
- Implements **Incremental Loads** in **Delta Tables**.
- Real-time **event-driven** ingestion architecture.
- **SCD1 & SCD2** for historical and non-historical data management.
- End-to-End **Databricks Workflow Automation**.
- Efficient **Merge & Aggregation** handling for dynamic datasets.

---
