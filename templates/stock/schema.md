# Database Schema - แผนกคลังสินค้า (Stock/Inventory)

**Database:** sml1
**Schema:** public
**Tables:** ic_inventory, ic_category, ic_trans, ic_trans_detail, ic_unit_use, ar_customer, ap_supplier

---

## โครงสร้างตาราง

### Table: ic_trans (หัวเอกสารธุรกรรม - Transaction Header)

```sql
CREATE TABLE ic_trans (
    -- Primary Key
    doc_no           VARCHAR(50)  PRIMARY KEY,  -- เลขที่เอกสาร
    trans_flag       INT2,                      -- ธงประเภทธุรกรรม

    -- ข้อมูลเอกสาร
    trans_type       INT2,          -- ประเภทเอกสาร: 1=ขาย(ลูกหนี้), 2=ซื้อ(เจ้าหนี้)
    doc_date         DATE,          -- วันที่เอกสาร
    doc_time         TIME,          -- เวลาเอกสาร
    doc_ref          VARCHAR(50),   -- เลขที่เอกสารอ้างอิง
    doc_ref_date     DATE,          -- วันที่เอกสารอ้างอิง
    
    -- ข้อมูลภาษี
    tax_doc_no       VARCHAR(50),   -- เลขที่ใบกำกับภาษี
    tax_doc_date     DATE,          -- วันที่ใบกำกับภาษี
    vat_type         INT2,          -- ประเภทภาษี: 0=แยกนอก, 1=รวมใน, 2=0%, 3=ไม่กระทบ
    vat_rate         NUMERIC,       -- อัตราภาษีมูลค่าเพิ่ม
    
    -- ข้อมูลลูกค้า/ซัพพลายเออร์
    cust_code        VARCHAR(25),   -- รหัสลูกหนี้/เจ้าหนี้ (FK)
    
    -- ข้อมูลการขาย
    inquiry_type     INT2,          -- ประเภทการขาย: 0=เงินเชื่อ, 1=เงินสด, 2=เงินเชื่อ(บริการ), 3=เงินสด(บริการ)
    send_day         INT,           -- จำนวนวันนัดส่ง
    send_date        DATE,          -- วันที่ส่งสินค้า
    
    -- ข้อมูลการคำนวณ
    discount_word    VARCHAR(100),  -- ข้อความส่วนลด (เช่น "50%" หรือ "50.0")
    total_value      NUMERIC,       -- มูลค่ารวมก่อนส่วนลด
    total_discount   NUMERIC,       -- ยอดส่วนลดรวม
    total_before_vat NUMERIC,       -- มูลค่าก่อนภาษี
    total_vat_value  NUMERIC,       -- มูลค่าภาษี
    total_after_vat  NUMERIC,       -- มูลค่าหลังรวมภาษี
    total_except_vat NUMERIC,       -- มูลค่าไม่รวมภาษี
    total_amount     NUMERIC,       -- ยอดสุทธิ
    
    -- สถานะ
    last_status      INT2,          -- สถานะเอกสาร: 0=ปกติ, 1=ยกเลิก
);
```

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

### Table: ar_customer (ลูกหนี้/ลูกค้า)

```sql
CREATE TABLE ar_customer (
    -- Primary Key
    code           VARCHAR(25)  PRIMARY KEY,  -- รหัสลูกหนี้

    -- ข้อมูลลูกหนี้
    name_1         VARCHAR(255),              -- ชื่อลูกหนี้
    telephone      VARCHAR(150),              -- เบอร์โทรศัพท์
    address        VARCHAR(255),              -- ที่อยู่
    status         INT2 DEFAULT 0,            -- สถานะ (0=ใช้งาน, 1=ไม่ใช้งาน)
    -- ... (more fields)
);
```

### Table: ap_supplier (เจ้าหนี้/ผู้ขาย)

```sql
CREATE TABLE ap_supplier (
    -- Primary Key
    code           VARCHAR(25)  PRIMARY KEY,  -- รหัสเจ้าหนี้

    -- ข้อมูลเจ้าหนี้
    name_1         VARCHAR(100),              -- ชื่อเจ้าหนี้
    telephone      VARCHAR(150),              -- เบอร์โทรศัพท์
    address        VARCHAR(255),              -- ที่อยู่
    status         INT2 DEFAULT 0,            -- สถานะ (0=ใช้งาน, 1=ไม่ใช้งาน)
    -- ... (more fields)
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
    cust_code        VARCHAR(25),   -- รหัสลูกหนี้/เจ้าหนี้ (FK -> ar_customer.code หรือ ap_supplier.code)
    trans_type       INT2 DEFAULT 0 NOT NULL,  -- ประเภทเอกสาร: 1=ขาย(ลูกหนี้), 2=ซื้อ(เจ้าหนี้)
    -- ... (more fields)
);
```

### Relationships

```
ic_inventory.item_category -> ic_category.code
ic_trans_detail.item_code -> ic_inventory.code
ic_trans_detail.doc_no -> ic_trans.doc_no (รายละเอียดอ้างอิงหัวเอกสาร)
ic_trans_detail.cust_code -> ar_customer.code (เมื่อ trans_type=1)
ic_trans_detail.cust_code -> ap_supplier.code (เมื่อ trans_type=2)
ic_trans.cust_code -> ar_customer.code (เมื่อ trans_type=1)
ic_trans.cust_code -> ap_supplier.code (เมื่อ trans_type=2)
```

**ความสัมพันธ์หัวเอกสาร-รายละเอียด:**
- `ic_trans` = **Transaction Header** (หัวเอกสาร) - เก็บข้อมูลภาพรวมของเอกสาร
- `ic_trans_detail` = **Transaction Detail** (รายละเอียดเอกสาร) - เก็บข้อมูลรายการสินค้าแต่ละตัว
- เอกสาร 1 ใบ (1 doc_no) มีรายละเอียดได้หลายรายการ (1:N relationship)

---

## Field Descriptions

### Table: ic_trans (หัวเอกสาร)

| Field | Type | Description |
|-------|------|-------------|
| **doc_no** | VARCHAR(50) | เลขที่เอกสาร (Primary Key) |
| **trans_flag** | INT2 | ธงประเภทธุรกรรม (6=PO, 12=Purchase, 34=จอง, 28=Sale, etc.) |
| **trans_type** | INT2 | ประเภทเอกสาร: 1=ขาย(ลูกหนี้/AR), 2=ซื้อ(เจ้าหนี้/AP) |
| **doc_date** | DATE | วันที่เอกสาร |
| **doc_time** | TIME | เวลาเอกสาร |
| **doc_ref** | VARCHAR(50) | เลขที่เอกสารอ้างอิง |
| **doc_ref_date** | DATE | วันที่เอกสารอ้างอิง |
| **tax_doc_no** | VARCHAR(50) | เลขที่ใบกำกับภาษี |
| **tax_doc_date** | DATE | วันที่ใบกำกับภาษี |
| **vat_type** | INT2 | ประเภทภาษี: 0=แยกนอก, 1=รวมใน, 2=0%, 3=ไม่กระทบ |
| **vat_rate** | NUMERIC | อัตราภาษีมูลค่าเพิ่ม (เช่น 7, 10) |
| **cust_code** | VARCHAR(25) | รหัสลูกหนี้/เจ้าหนี้ (FK) |
| **inquiry_type** | INT2 | ประเภทการขาย: 0=เงินเชื่อ, 1=เงินสด, 2=เงินเชื่อ(บริการ), 3=เงินสด(บริการ) |
| **send_day** | INT | จำนวนวันนัดส่ง |
| **send_date** | DATE | วันที่ส่งสินค้า |
| **discount_word** | VARCHAR(100) | ข้อความส่วนลด (เช่น "50%" หรือ "50.0") |
| **total_value** | NUMERIC | มูลค่ารวมก่อนส่วนลด |
| **total_discount** | NUMERIC | ยอดส่วนลดรวม |
| **total_before_vat** | NUMERIC | มูลค่าก่อนภาษี |
| **total_vat_value** | NUMERIC | มูลค่าภาษี |
| **total_after_vat** | NUMERIC | มูลค่าหลังรวมภาษี |
| **total_except_vat** | NUMERIC | มูลค่าไม่รวมภาษี |
| **total_amount** | NUMERIC | ยอดสุทธิ |
| **last_status** | INT2 | สถานะเอกสาร: 0=ปกติ, 1=ยกเลิก |

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
| **cust_code** | VARCHAR(25) | รหัสลูกหนี้/เจ้าหนี้ (ใช้ร่วมกับ trans_type) |
| **trans_type** | INT2 | ประเภทเอกสาร: 1=ขาย(ลูกหนี้/AR), 2=ซื้อ(เจ้าหนี้/AP) |

### Table: ar_customer

| Field | Type | Description |
|-------|------|-------------|
| **code** | VARCHAR(25) | รหัสลูกหนี้ (Primary Key) |
| **name_1** | VARCHAR(255) | ชื่อลูกหนี้ |
| **telephone** | VARCHAR(150) | เบอร์โทรศัพท์ |
| **address** | VARCHAR(255) | ที่อยู่ |
| **status** | INT2 | สถานะ: 0=ใช้งาน, 1=ไม่ใช้งาน |

### Table: ap_supplier

| Field | Type | Description |
|-------|------|-------------|
| **code** | VARCHAR(25) | รหัสเจ้าหนี้ (Primary Key) |
| **name_1** | VARCHAR(100) | ชื่อเจ้าหนี้ |
| **telephone** | VARCHAR(150) | เบอร์โทรศัพท์ |
| **address** | VARCHAR(255) | ที่อยู่ |
| **status** | INT2 | สถานะ: 0=ใช้งาน, 1=ไม่ใช้งาน |

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

### 2.1 ค้นหาสินค้าทั้งรหัสและชื่อ (ยืดหยุ่น)

**ใช้เมื่อ:** user พิมพ์คำค้นที่อาจเป็นรหัสหรือชื่อสินค้า

```sql
SELECT
  i.code,
  i.name_1,
  c.name_1 as category_name,
  i.unit_standard,
  i.unit_standard_name
FROM ic_inventory i
LEFT JOIN ic_category c ON i.item_category = c.code
WHERE i.code LIKE '%keyword%' 
   OR i.name_1 LIKE '%keyword%'
ORDER BY 
  CASE 
    WHEN i.code = 'keyword' THEN 1  -- exact code match first
    WHEN i.code LIKE 'keyword%' THEN 2  -- code starts with
    WHEN i.name_1 LIKE 'keyword%' THEN 3  -- name starts with
    ELSE 4
  END
LIMIT 20;
```

**คำอธิบาย:**
- ค้นหาทั้งรหัสสินค้า (code) และชื่อสินค้า (name_1)
- เรียงลำดับผลลัพธ์: รหัสตรงทุกตัว > รหัสขึ้นต้นด้วย > ชื่อขึ้นต้นด้วย > อื่นๆ
- ใช้ได้ทั้งกรณีที่ user พิมพ์รหัสหรือชื่อ

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

### 9. ค้นหาข้อมูลลูกหนี้ (AR Customer)

```sql
SELECT code, name_1, telephone, address
FROM ar_customer
WHERE status = 0
  AND (code LIKE '%keyword%' OR name_1 LIKE '%keyword%')
LIMIT 20;
```

### 10. ค้นหาข้อมูลเจ้าหนี้ (AP Supplier)

```sql
SELECT code, name_1, telephone, address
FROM ap_supplier
WHERE status = 0
  AND (code LIKE '%keyword%' OR name_1 LIKE '%keyword%')
LIMIT 20;
```

### 11. ยอดคงเหลือสินค้า กรองตามลูกหนี้/เจ้าหนี้

**วิธีใช้:** วิเคราะห์บริบทคำถามเพื่อกำหนด `trans_type`:
- คำถามเกี่ยวกับ **ลูกหนี้/ลูกค้า/ขาย** → ใช้ `trans_type = 1`
- คำถามเกี่ยวกับ **เจ้าหนี้/ซัพพลายเออร์/ซื้อ** → ใช้ `trans_type = 2`

**ตัวอย่าง Query (ยอดคงเหลือตามลูกหนี้):**
```sql
SELECT 
  itd.item_code,
  i.name_1 as item_name,
  ar.name_1 as customer_name,
  SUM(itd.calc_flag * ROUND((itd.qty * itd.stand_value) / itd.divide_value, 2)) as balance_qty,
  i.unit_standard_name
FROM ic_trans_detail itd
LEFT JOIN ic_inventory i ON itd.item_code = i.code
LEFT JOIN ar_customer ar ON itd.cust_code = ar.code
WHERE itd.last_status = 0
  AND itd.trans_type = 1
  AND (ar.code LIKE '%CUST_CODE_OR_NAME%' OR ar.name_1 LIKE '%CUST_CODE_OR_NAME%')
GROUP BY itd.item_code, i.name_1, ar.name_1, i.unit_standard_name
ORDER BY balance_qty DESC
LIMIT 30;
```

**ตัวอย่าง Query (ยอดคงเหลือตามเจ้าหนี้):**
```sql
SELECT 
  itd.item_code,
  i.name_1 as item_name,
  ap.name_1 as supplier_name,
  SUM(itd.calc_flag * ROUND((itd.qty * itd.stand_value) / itd.divide_value, 2)) as balance_qty,
  i.unit_standard_name
FROM ic_trans_detail itd
LEFT JOIN ic_inventory i ON itd.item_code = i.code
LEFT JOIN ap_supplier ap ON itd.cust_code = ap.code
WHERE itd.last_status = 0
  AND itd.trans_type = 2
  AND (ap.code LIKE '%SUPP_CODE_OR_NAME%' OR ap.name_1 LIKE '%SUPP_CODE_OR_NAME%')
GROUP BY itd.item_code, i.name_1, ap.name_1, i.unit_standard_name
ORDER BY balance_qty DESC
LIMIT 30;
```

### 12. ดูข้อมูลเอกสารพร้อมรายละเอียด (Header + Detail)

**ตัวอย่าง: ดูเอกสารจอง (trans_flag=34) พร้อมรายการสินค้า**
```sql
SELECT 
  t.doc_no,
  t.doc_date,
  t.trans_type,
  ar.name_1 as customer_name,
  t.total_amount,
  itd.item_code,
  i.name_1 as item_name,
  itd.qty,
  i.unit_standard_name
FROM ic_trans t
LEFT JOIN ic_trans_detail itd ON t.doc_no = itd.doc_no
LEFT JOIN ic_inventory i ON itd.item_code = i.code
LEFT JOIN ar_customer ar ON t.cust_code = ar.code
WHERE t.trans_flag = 34
  AND t.last_status = 0
  AND itd.last_status = 0
ORDER BY t.doc_date DESC
LIMIT 20;
```

### 13. สรุปยอดขายตามเอกสาร (ใช้ ic_trans)

```sql
SELECT 
  t.doc_no,
  t.doc_date,
  t.tax_doc_no,
  ar.name_1 as customer_name,
  t.total_before_vat,
  t.total_vat_value,
  t.total_amount
FROM ic_trans t
LEFT JOIN ar_customer ar ON t.cust_code = ar.code
WHERE t.trans_type = 1
  AND t.last_status = 0
  AND t.doc_date >= CURRENT_DATE - INTERVAL '30 days'
ORDER BY t.doc_date DESC
LIMIT 30;
```

### 14. ดูรายละเอียดเอกสารเฉพาะเลขที่

```sql
-- ดูหัวเอกสาร
SELECT 
  t.doc_no,
  t.doc_date,
  t.trans_type,
  t.trans_flag,
  CASE WHEN t.trans_type = 1 THEN ar.name_1 ELSE ap.name_1 END as partner_name,
  t.total_amount,
  t.vat_type,
  t.last_status
FROM ic_trans t
LEFT JOIN ar_customer ar ON t.cust_code = ar.code AND t.trans_type = 1
LEFT JOIN ap_supplier ap ON t.cust_code = ap.code AND t.trans_type = 2
WHERE t.doc_no = 'DOC_NO_HERE';

-- ดูรายละเอียดสินค้า
SELECT 
  itd.item_code,
  i.name_1 as item_name,
  itd.qty,
  itd.wh_code,
  i.unit_standard_name
FROM ic_trans_detail itd
LEFT JOIN ic_inventory i ON itd.item_code = i.code
WHERE itd.doc_no = 'DOC_NO_HERE'
  AND itd.last_status = 0
ORDER BY itd.item_code;
```

---

## คำถามที่พบบ่อย (FAQ Queries)

| คำถาม | SQL Query |
|-------|-----------|
| มีสินค้าทั้งหมดกี่รายการ? | `SELECT COUNT(*) as total_items FROM ic_inventory` |
| มีหมวดสินค้ากี่หมวด? | `SELECT COUNT(*) as total_categories FROM ic_category WHERE status = 0` |
| หาสินค้าตามชื่อ (คำเดียว) | `SELECT i.code, i.name_1, c.name_1 as category_name FROM ic_inventory i LEFT JOIN ic_category c ON i.item_category = c.code WHERE i.name_1 LIKE '%<ชื่อสินค้า>%' LIMIT 20` |
| หาสินค้าตามชื่อ (หลายคำ) | `SELECT i.code, i.name_1, c.name_1 as category_name FROM ic_inventory i LEFT JOIN ic_category c ON i.item_category = c.code WHERE i.name_1 LIKE '%<คำ1>%' OR i.name_1 LIKE '%<คำ2>%' OR i.name_1 LIKE '%<คำ3>%' LIMIT 20` |
| ดูข้อมูลสินค้าตามรหัส | `SELECT i.code, i.name_1, c.name_1 as category_name, i.unit_standard_name FROM ic_inventory i LEFT JOIN ic_category c ON i.item_category = c.code WHERE i.code = '<รหัสสินค้า>'` |
| ค้นหาสินค้า (รหัสหรือชื่อ) | `SELECT i.code, i.name_1, c.name_1 as category_name FROM ic_inventory i LEFT JOIN ic_category c ON i.item_category = c.code WHERE i.code LIKE '%<คำค้น>%' OR i.name_1 LIKE '%<คำค้น>%' LIMIT 20` |
| หมวดสินค้าไหนมีสินค้ามากที่สุด? | `SELECT c.name_1, COUNT(i.code) as cnt FROM ic_category c LEFT JOIN ic_inventory i ON c.code = i.item_category WHERE c.status = 0 GROUP BY c.code, c.name_1 ORDER BY cnt DESC LIMIT 1` |
| แสดงสินค้าในหมวดหมู่ | `SELECT i.code, i.name_1, i.unit_standard_name FROM ic_inventory i JOIN ic_category c ON i.item_category = c.code WHERE c.name_1 LIKE '%<ชื่อหมวด>%' LIMIT 30` |

---

## หมายเหตุสำคัญ

### ความสัมพันธ์ ic_trans และ ic_trans_detail

**โครงสร้างเอกสาร:**
- `ic_trans` = **หัวเอกสาร (Header)** - เก็บข้อมูลภาพรวม 1 ใบ
  - เลขที่เอกสาร (doc_no)
  - วันที่ (doc_date)
  - ลูกค้า/ซัพพลายเออร์ (cust_code)
  - ยอดรวม (total_amount)
  - ภาษี (vat_type, vat_rate, total_vat_value)
  
- `ic_trans_detail` = **รายละเอียดเอกสาร (Detail)** - เก็บรายการสินค้าแต่ละตัว
  - รายการสินค้า (item_code)
  - จำนวน (qty)
  - คลัง (wh_code)
  - **เชื่อมกับหัวผ่าน doc_no**

**การใช้งาน:**
- ถ้าต้องการ**ข้อมูลยอดเงิน, ภาษี, ลูกค้า** → ใช้ `ic_trans`
- ถ้าต้องการ**รายการสินค้า, จำนวน, คลัง** → ใช้ `ic_trans_detail`
- ถ้าต้องการ**ทั้งสองอย่าง** → JOIN ทั้งสองตาราง

**ตัวอย่าง:**
```sql
-- ดูเอกสารพร้อมรายการสินค้า
FROM ic_trans t
LEFT JOIN ic_trans_detail itd ON t.doc_no = itd.doc_no
WHERE t.doc_no = 'XXX'
```

### หลักการค้นหาชื่อ (Name Search Rules) ⚠️ สำคัญมาก

- **ห้ามใช้ `=` สำหรับการค้นหาชื่อ** (name_1, category name, etc.)
- **ต้องใช้ `LIKE '%keyword%'` เสมอ**
- **เมื่อมีหลายคำค้นหา ใช้ OR ไม่ใช่ AND**
  - ✅ **ถูก:** `WHERE name_1 LIKE '%คำ1%' OR name_1 LIKE '%คำ2%'` (ค้นหาได้กว้าง ตัดคำถามได้มาก)
  - ❌ **ผิด:** `WHERE name_1 LIKE '%คำ1%' AND name_1 LIKE '%คำ2%'` (เข้มงวดเกิน หาไม่เจอง่าย)
  - **เหตุผล:** OR ทำให้มีโอกาสหาข้อมูลเจอมากกว่า (มีคำใดคำหนึ่งก็ถือว่าเจอ)
- **เหตุผล:**
  - User อาจพิมพ์ไม่ครบ (พิมพ์ "BT 1" แต่ในฐานข้อมูลเป็น "หมวดสินค้า BT 1")
  - ชื่อในฐานข้อมูลอาจมีคำเพิ่มเติมข้างหน้าหรือข้างหลัง
  - ป้องกันกรณีหาไม่เจอ (Better to find too many than find nothing)
- **ตัวอย่าง:**
  - ❌ **ผิด:** `WHERE c.name_1 = '<ชื่อหมวด>'` (จะหาไม่เจอถ้าชื่อจริงมีคำเพิ่มเติม)
  - ✅ **ถูก:** `WHERE c.name_1 LIKE '%<ชื่อหมวด>%'` (จะหาเจอทุกกรณี)
  - ✅ **ถูก:** `WHERE i.name_1 LIKE '%<ชื่อสินค้า>%'` (ค้นหาสินค้าที่มีคำที่ระบุ)
  - ✅ **ถูก:** `WHERE i.name_1 LIKE '%สาย%' OR i.name_1 LIKE '%100M%'` (หาสินค้าที่มี "สาย" หรือ "100M")
- **ห้าม hardcode ชื่อสินค้าเฉพาะ** - ต้องค้นหาได้ทุกสินค้า ทุกร้าน

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

-- With Warehouse Filter (XXX = รหัสคลังของร้าน)
FROM ic_trans_detail WHERE trans_flag = 34 AND wh_code = 'XXX'
```

### 2. Category Filter (item_category)

**Apply to tables:**
- `ic_inventory.item_category`
- JOIN กับ `ic_category.name_1`

**Example:**
```sql
-- With Category Filter (<ชื่อหมวด> = ชื่อหมวดหมู่ของร้าน)
WHERE c.name_1 LIKE '%<ชื่อหมวด>%'
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
WHERE wh_code = 'XXX'                                 -- Warehouse filter (XXX = รหัสคลัง)
  AND item_category IN (...)                          -- Category filter
  AND doc_date_calc >= CURRENT_DATE - INTERVAL '...'  -- Date filter
```

---

**Version:** 2.0.0
**Format:** Markdown
**Last Updated:** December 26, 2025
