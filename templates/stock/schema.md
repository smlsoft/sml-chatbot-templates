# Database Schema - แผนกคลังสินค้า (Stock/Inventory)

**Database:** sml1
**Schema:** public
**Tables:** ic_inventory, ic_category, ic_trans_detail, ic_trans, ic_unit_use

---

## โครงสร้างตาราง

### Table: ic_inventory (สินค้า)

```sql
CREATE TABLE ic_inventory (
    -- Primary Key
    code                  VARCHAR(25)  PRIMARY KEY,  -- รหัสสินค้า

    -- ข้อมูลสินค้า
    name_1                VARCHAR(255),              -- ชื่อสินค้า
    item_category         VARCHAR(25),               -- รหัสหมวดสินค้า (FK -> ic_category.code)

    -- ข้อมูลหน่วยนับ
    unit_standard         VARCHAR(25),               -- รหัสหน่วยนับ
    unit_standard_name    VARCHAR(100),              -- ชื่อหน่วยนับ
);
```

### Table: ic_category (หมวดสินค้า)

```sql
CREATE TABLE ic_category (
    -- Primary Key
    code      VARCHAR(25)  PRIMARY KEY,  -- รหัสหมวดสินค้า

    -- ข้อมูลหมวดสินค้า
    name_1    VARCHAR(100),              -- ชื่อหมวดสินค้า
    status    INT2 DEFAULT 0,            -- สถานะ (0=ใช้งาน, 1=ไม่ใช้งาน)
);
```

### Table: ic_trans_detail (รายการธุรกรรมสินค้า)

```sql
CREATE TABLE ic_trans_detail (
    doc_no           VARCHAR(50),   -- เลขที่เอกสาร
    item_code        VARCHAR(25),   -- รหัสสินค้า
    wh_code          VARCHAR(25),   -- รหัสคลัง
    shelf_code       VARCHAR(25),   -- รหัสพื้นที่เก็บ
    qty              NUMERIC,       -- จำนวน
    trans_flag       INT2,          -- ประเภทธุรกรรม
    last_status      INT2,          -- สถานะ (0=ใช้งาน)
    doc_date_calc    DATE,          -- วันที่เอกสาร
    -- ... (more fields)
);
```

### Relationships

```
ic_inventory.item_category -> ic_category.code
ic_trans_detail.item_code -> ic_inventory.code
```

---

## Field Descriptions

### Table: ic_inventory

| Field | Type | Description |
|-------|------|-------------|
| **code** | VARCHAR(25) | รหัสสินค้า (Primary Key) |
| **name_1** | VARCHAR(255) | ชื่อสินค้า |
| **item_category** | VARCHAR(25) | รหัสหมวดสินค้า (Foreign Key -> ic_category.code) |
| **unit_standard** | VARCHAR(25) | รหัสหน่วยนับ |
| **unit_standard_name** | VARCHAR(100) | ชื่อหน่วยนับ |

### Table: ic_category

| Field | Type | Description |
|-------|------|-------------|
| **code** | VARCHAR(25) | รหัสหมวดสินค้า (Primary Key) |
| **name_1** | VARCHAR(100) | ชื่อหมวดสินค้า |
| **status** | INT2 | สถานะ: 0=ใช้งาน, 1=ไม่ใช้งาน |

### Table: ic_trans_detail

| Field | Type | Description |
|-------|------|-------------|
| **doc_no** | VARCHAR(50) | เลขที่เอกสาร |
| **item_code** | VARCHAR(25) | รหัสสินค้า |
| **wh_code** | VARCHAR(25) | รหัสคลัง |
| **shelf_code** | VARCHAR(25) | รหัสพื้นที่เก็บ (location) |
| **qty** | NUMERIC | จำนวน |
| **trans_flag** | INT2 | ประเภทธุรกรรม (34=จอง, 36=ส่ง, 6=รับ, etc.) |
| **last_status** | INT2 | สถานะ: 0=ใช้งาน, 1=ยกเลิก |
| **doc_date_calc** | DATE | วันที่เอกสาร |

---

## ตัวอย่าง SQL Queries

### 1. ดึงข้อมูลสินค้าพร้อมชื่อหมวดสินค้า

```sql
SELECT
  i.code,
  i.name_1,
  i.item_category,
  c.name_1 as category_name,
  i.unit_standard,
  i.unit_standard_name
FROM ic_inventory i
LEFT JOIN ic_category c ON i.item_category = c.code
ORDER BY i.code
LIMIT 100;
```

### 2. ค้นหาสินค้าตามชื่อ (พร้อมชื่อหมวดสินค้า)

```sql
SELECT
  i.code,
  i.name_1,
  c.name_1 as category_name,
  i.unit_standard,
  i.unit_standard_name
FROM ic_inventory i
LEFT JOIN ic_category c ON i.item_category = c.code
WHERE i.name_1 LIKE '%keyword%'
LIMIT 20;
```

### 3. ดึงสินค้าตามหมวดหมู่

```sql
SELECT
  i.code,
  i.name_1,
  c.name_1 as category_name,
  i.unit_standard,
  i.unit_standard_name
FROM ic_inventory i
LEFT JOIN ic_category c ON i.item_category = c.code
WHERE i.item_category = 'CATEGORY_CODE'
ORDER BY i.name_1
LIMIT 50;
```

### 4. นับจำนวนสินค้าแบ่งตามหมวดหมู่ (แสดงชื่อหมวด)

```sql
SELECT
  c.code as category_code,
  c.name_1 as category_name,
  COUNT(i.code) as item_count
FROM ic_category c
LEFT JOIN ic_inventory i ON c.code = i.item_category
WHERE c.status = 0
GROUP BY c.code, c.name_1
ORDER BY item_count DESC;
```

### 5. ดูรายการหมวดสินค้าทั้งหมด (ที่ใช้งาน)

```sql
SELECT code, name_1
FROM ic_category
WHERE status = 0
ORDER BY code;
```

### 6. ค้นหาสินค้าตามรหัส (พร้อมชื่อหมวดสินค้า)

```sql
SELECT
  i.code,
  i.name_1,
  i.item_category,
  c.name_1 as category_name,
  i.unit_standard,
  i.unit_standard_name
FROM ic_inventory i
LEFT JOIN ic_category c ON i.item_category = c.code
WHERE i.code = 'ITEM_CODE';
```

### 7. ค้นหาสินค้าตามชื่อหมวดสินค้า

```sql
SELECT
  i.code,
  i.name_1,
  c.name_1 as category_name,
  i.unit_standard,
  i.unit_standard_name
FROM ic_inventory i
LEFT JOIN ic_category c ON i.item_category = c.code
WHERE c.name_1 LIKE '%keyword%'
LIMIT 20;
```

### 8. ดูหน่วยนับที่ใช้ในระบบ

```sql
SELECT DISTINCT unit_standard, unit_standard_name
FROM ic_inventory
WHERE unit_standard IS NOT NULL AND unit_standard != ''
ORDER BY unit_standard;
```

---

## คำถามที่พบบ่อย (FAQ Queries)

| คำถาม | SQL Query |
|-------|-----------|
| มีสินค้าทั้งหมดกี่รายการ? | `SELECT COUNT(*) as total_items FROM ic_inventory` |
| มีหมวดสินค้ากี่หมวด? | `SELECT COUNT(*) as total_categories FROM ic_category WHERE status = 0` |
| หาสินค้าชื่อ "น้ำดื่ม" | `SELECT i.code, i.name_1, c.name_1 as category_name FROM ic_inventory i LEFT JOIN ic_category c ON i.item_category = c.code WHERE i.name_1 LIKE '%น้ำดื่ม%' LIMIT 20` |
| ดูข้อมูลสินค้ารหัส "P001" | `SELECT i.code, i.name_1, c.name_1 as category_name, i.unit_standard_name FROM ic_inventory i LEFT JOIN ic_category c ON i.item_category = c.code WHERE i.code = 'P001'` |
| หมวดสินค้าไหนมีสินค้ามากที่สุด? | `SELECT c.name_1, COUNT(i.code) as cnt FROM ic_category c LEFT JOIN ic_inventory i ON c.code = i.item_category WHERE c.status = 0 GROUP BY c.code, c.name_1 ORDER BY cnt DESC LIMIT 1` |
| แสดงสินค้าในหมวด "เครื่องดื่ม" | `SELECT i.code, i.name_1, i.unit_standard_name FROM ic_inventory i JOIN ic_category c ON i.item_category = c.code WHERE c.name_1 LIKE '%เครื่องดื่ม%' LIMIT 30` |

---

## หมายเหตุสำคัญ

### หลักการค้นหาชื่อ (Name Search Rules) ⚠️ สำคัญมาก

- **ห้ามใช้ `=` สำหรับการค้นหาชื่อ** (name_1, category name, etc.)
- **ต้องใช้ `LIKE '%keyword%'` เสมอ**
- **เหตุผล:**
  - User อาจพิมพ์ไม่ครบ (พิมพ์ "BT 1" แต่ในฐานข้อมูลเป็น "หมวดสินค้า BT 1")
  - ชื่อในฐานข้อมูลอาจมีคำเพิ่มเติมข้างหน้าหรือข้างหลัง
  - ป้องกันกรณีหาไม่เจอ (Better to find too many than find nothing)
- **ตัวอย่าง:**
  - ❌ **ผิด:** `WHERE c.name_1 = 'BT 1'` (จะหาไม่เจอถ้าชื่อจริงคือ "หมวดสินค้า BT 1")
  - ✅ **ถูก:** `WHERE c.name_1 LIKE '%BT 1%'` (จะหาเจอทุกกรณี)
  - ✅ **ถูก:** `WHERE i.name_1 LIKE '%น้ำดื่ม%'` (ค้นหาสินค้าที่มีคำว่า "น้ำดื่ม")

### การใช้ Fields

- ใช้ **code** เป็น Primary Key ในการอ้างอิง
- ใช้ **name_1** สำหรับชื่อสินค้าหลัก
- ใช้ **item_category** เป็น Foreign Key อ้างอิงไปยัง **ic_category.code**
- **เมื่อต้องการแสดงชื่อหมวดสินค้า ต้อง JOIN กับ ic_category เสมอ**
- ใช้ **unit_standard** และ **unit_standard_name** สำหรับข้อมูลหน่วยนับ

### การใช้ JOIN

- **ใช้ LEFT JOIN** เมื่อต้องการแสดงสินค้าทั้งหมด แม้ไม่มีหมวดสินค้า
- **ใช้ INNER JOIN** เมื่อต้องการเฉพาะสินค้าที่มีหมวดสินค้า
- **ตัวอย่าง:**
  ```sql
  FROM ic_inventory i
  LEFT JOIN ic_category c ON i.item_category = c.code
  ```

### การแสดงผล

- จำกัดผลลัพธ์ด้วย `LIMIT` เสมอ (แนะนำ 20-30 แถว สูงสุด 100 แถว)
- ใช้ `ORDER BY` เพื่อเรียงลำดับที่เหมาะสม
- ใช้ `LIKE '%keyword%'` สำหรับการค้นหาแบบ partial match
- ใช้ `IS NOT NULL AND field != ''` เพื่อกรองข้อมูลว่าง

### การค้นหาหมวดสินค้า

- **ห้ามใช้ item_category โดยตรง** เพราะเป็นเพียงรหัส
- **ต้อง JOIN กับ ic_category.name_1** เพื่อดึงชื่อหมวดสินค้า
- **ตัวอย่างที่ถูก:** `WHERE c.name_1 LIKE '%keyword%'`
- **ตัวอย่างที่ผิด:** `WHERE i.item_category LIKE '%keyword%'`

---

## 🎯 Custom Filter Application Points

เมื่อ Channel มี `customSystemPrompt` ที่กำหนด filter เช่น warehouse, category, date range ให้ใช้กับ:

### 1. Warehouse Filter (wh_code, warehouse)

**Apply to tables:**
- `ic_trans_detail.wh_code`
- `final.warehouse` (ใน special queries)

**Example:**
```sql
-- Original
FROM ic_trans_detail WHERE trans_flag = 34

-- With Warehouse Filter "DEDE"
FROM ic_trans_detail WHERE trans_flag = 34 AND wh_code = 'DEDE'
```

### 2. Category Filter (item_category)

**Apply to tables:**
- `ic_inventory.item_category`
- JOIN กับ `ic_category.name_1`

**Example:**
```sql
-- With Category Filter "เครื่องดื่ม"
WHERE c.name_1 LIKE '%เครื่องดื่ม%'
```

### 3. Date Filter (doc_date, doc_date_calc)

**Apply to tables:**
- `ic_trans_detail.doc_date_calc`
- `ic_trans.doc_date`

**Example:**
```sql
-- With Date Filter "3 months"
WHERE doc_date_calc >= CURRENT_DATE - INTERVAL '3 months'
```

---

## 🔧 Filter Combination Rules

เมื่อมีหลาย filters พร้อมกัน:
```sql
WHERE wh_code = 'DEDE'                                -- Warehouse filter
  AND item_category IN (...)                          -- Category filter
  AND doc_date_calc >= CURRENT_DATE - INTERVAL '...'  -- Date filter
```

---

**Version:** 2.0.0
**Format:** Markdown
**Last Updated:** December 26, 2025
