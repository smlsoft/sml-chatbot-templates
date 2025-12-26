# Database Schema - แผนกเจ้าหนี้ (Accounts Payable)

**Database:** sml1
**Schema:** public
**Tables:** ap_vendor, ap_purchase

---

## โครงสร้างตาราง

### Table: ap_vendor (ผู้ขาย)

```sql
CREATE TABLE ap_vendor (
    code      VARCHAR(25)  PRIMARY KEY,  -- รหัสผู้ขาย
    name_1    VARCHAR(255),              -- ชื่อผู้ขาย
);
```

### Table: ap_purchase (ใบสั่งซื้อ)

```sql
CREATE TABLE ap_purchase (
    doc_no       VARCHAR(50)  PRIMARY KEY,  -- เลขที่ใบสั่งซื้อ
    vendor_code  VARCHAR(25),               -- รหัสผู้ขาย
    amount       NUMERIC,                   -- จำนวนเงิน
);
```

---

## ตัวอย่าง SQL Queries

### 1. ดึงข้อมูลผู้ขาย

```sql
SELECT code, name_1
FROM ap_vendor
ORDER BY code
LIMIT 100;
```

### 2. ค้นหาผู้ขายตามชื่อ

```sql
SELECT code, name_1
FROM ap_vendor
WHERE name_1 LIKE '%keyword%'
LIMIT 20;
```

---

**Version:** 2.0.0
**Format:** Markdown
**Last Updated:** December 26, 2025
