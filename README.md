# DataBricks-Project-1
### Notes from the Image

---

#### **Project 1: Incremental Data Load in Delta Table with Event-Driven Architecture**

---

### **Architecture Overview**
- **Platform**: **Databricks Workflow**
- **Objective**: Incrementally load data into a Delta table using an event-driven process.

---

### **Flow Components**

#### **1. Google Storage (Source & Archive Layer):**
- **Bucket 1 (Source)**:
  - Holds incoming files (e.g., `file1.csv`, `file2.csv`).
- **Client**:
  - Sends files to the source location in Google Storage.
  - Example files:
    - `file1.csv` → version_123, AC1
    - version_123, AC1 → Overwrite → version_123, AC1
    - version_123, AC2 → Archive (current file)

#### **2. Databricks Workflow**:
- **Task 1 (Load in Stage)**:
  - Loads data from the source (Google Storage) into the staging area.
  - Operation: **Overwrite** into **Stage Table**.
- **Task 2 (Merge in Target)**:
  - Merges data from the Stage Table into the Target Table.
  - Operation: **Merge (upsert)** into **Target Table**.

---

### **Tables Involved**:
- **Stage Table**
- **Target Table**

---

### **Data Movement Flow**:
1. **Client uploads files** → Google Storage **Bucket 1 (Source)**.
2. Databricks Task 1:
   - Loads data from **Source** to **Stage Table** (Overwrite mode).
3. Databricks Task 2:
   - Merges data from **Stage Table** into **Target Table** (Upsert/Merge mode).
4. After processing:
   - Archive the current file into an **Archive** folder.

---

## **Flow Diagram**

```
Client
   │
   │  Uploads files (file1.csv)
   ▼
Google Storage (Bucket 1)
 ├── Source Folder (file1.csv)
 └── Archive Folder (archived files)

   │
   │  Trigger / Event (Overwrite)
   ▼
Databricks Workflow
 ├── Task 1: Load in Stage
 │     - Overwrite Stage Table
 │
 └── Task 2: Merge in Target
       - Merge (upsert) into Target Table

   ▼
Tables
 ├── Stage Table
 └── Target Table
```

---

Let me know if you'd like a polished flowchart or diagram image!
