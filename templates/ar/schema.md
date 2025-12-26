# Database Schema - แผนกลูกหนี้ (Accounts Receivable)

**Database:** sml1
**Schema:** public
**Tables:** ar_customer, ar_invoice

---

## โครงสร้างตาราง

### Table: ar_customer (ลูกค้า)

```sql
CREATE TABLE ar_customer (
    code      VARCHAR(25)  PRIMARY KEY,  -- รหัสลูกค้า
    name_1    VARCHAR(255),              -- ชื่อลูกค้า
);
```

### Table: ar_invoice (ใบแจ้งหนี้)

```sql
CREATE TABLE ar_invoice (
    doc_no         VARCHAR(50)  PRIMARY KEY,  -- เลขที่ใบแจ้งหนี้
    customer_code  VARCHAR(25),               -- รหัสลูกค้า
    amount         NUMERIC,                   -- จำนวนเงิน
);
```

---

## ตัวอย่าง SQL Queries

### 1. ดึงข้อมูลลูกค้า

```sql
SELECT code, name_1
FROM ar_customer
ORDER BY code
LIMIT 100;
```

### 2. ค้นหาลูกค้าตามชื่อ

```sql
SELECT code, name_1
FROM ar_customer
WHERE name_1 LIKE '%keyword%'
LIMIT 20;
```

---

**Version:** 2.0.0
**Format:** Markdown
**Last Updated:** December 26, 2025
