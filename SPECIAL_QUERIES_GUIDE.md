# Special Queries Reference - สำหรับ Stock Template

เอกสารนี้อธิบายวิธีจัดการกับ Special Queries ที่มี Business Logic ซับซ้อน

---

## ⚠️ ปัญหาที่พบ

Special Queries เช่น:
- ยอดค้างจอง (Book Out)
- ยอดค้างส่ง (Accrued Out)  
- ยอดค้างรับ (Accrued In)
- ยอดคงเหลือแยกคลัง (Stock Balance)

มี Business Logic ที่ซับซ้อนมาก:
- ใช้ CTE (WITH clause) หลายชั้น
- Subqueries ซ้อนกัน
- Calculation ที่ซับซ้อน
- ต้องมี 2 กรณี (ระบุรหัส / ไม่ระบุรหัส)

**ไม่สามารถใส่ใน customQueries ได้** เพราะ format ปัจจุบันรองรับแค่:
```json
{
  "keyword": "...",
  "description": "...",
  "sql": "..."
}
```

---

## 💡 Solutions

### Solution 1: ใส่ตัวอย่างใน systemPrompt (Current)

ใส่ตัวอย่าง SQL พร้อมคำอธิบายใน systemPrompt:

```json
{
  "systemPrompt": "...\n\n## Special Queries ที่ต้องใช้\n\n### ยอดค้างจอง\nเมื่อ user ถามเกี่ยวกับ 'ยอดค้างจอง', 'book out', 'จองสินค้า'\n\n**กรณีที่ 1: ระบุรหัสสินค้า**\n```sql\nWITH bookout as (\n  select item_code, sum(bookout_qty_balance) as book_out_qty from (\n    ...\n  ) as temp3 group by item_code\n)\nselect * from bookout\n```\nแทนที่ 'ITEM_CODE_HERE' ด้วยรหัสสินค้า\n\n**กรณีที่ 2: ไม่ระบุรหัส**\n```sql\nWITH bookout as (...)\nSELECT b.item_code, i.name_1, ROUND(b.book_out_qty, 2), i.unit_standard_name\nFROM bookout b\nLEFT JOIN ic_inventory i ON b.item_code = i.code\nWHERE b.book_out_qty <> 0\nORDER BY b.book_out_qty DESC\nLIMIT 30\n```"
}
```

**ข้อดี:**
- ไม่ต้องแก้ code
- AI เรียนรู้จากตัวอย่าง

**ข้อเสีย:**
- systemPrompt ยาวมาก
- AI อาจจำไม่ได้ทั้งหมด
- Token usage สูง

---

### Solution 2: สร้าง Stored Procedures (Recommended)

สร้าง Functions/Stored Procedures ในฐานข้อมูล:

```sql
-- Function สำหรับยอดค้างจอง
CREATE OR REPLACE FUNCTION get_bookout_qty(
  item_codes TEXT[] DEFAULT NULL
)
RETURNS TABLE (
  item_code VARCHAR(25),
  item_name VARCHAR(255),
  book_out_qty NUMERIC,
  unit_name VARCHAR(100)
) AS $$
BEGIN
  -- Complex CTE logic here
  RETURN QUERY
  WITH bookout as (
    -- ... original complex query ...
  )
  SELECT 
    b.item_code,
    i.name_1,
    ROUND(b.book_out_qty, 2),
    i.unit_standard_name
  FROM bookout b
  LEFT JOIN ic_inventory i ON b.item_code = i.code
  WHERE b.book_out_qty <> 0
  AND (item_codes IS NULL OR b.item_code = ANY(item_codes))
  ORDER BY b.book_out_qty DESC
  LIMIT 30;
END;
$$ LANGUAGE plpgsql;
```

แล้วใน template:

```json
{
  "customQueries": [
    {
      "keyword": "ยอดค้างจอง|book out|จองสินค้า",
      "description": "ยอดสินค้าที่ถูกจองแต่ยังไม่ได้จ่าย",
      "sql": "SELECT * FROM get_bookout_qty(NULL)"
    }
  ]
}
```

หรือถ้า user ระบุรหัส:
```sql
SELECT * FROM get_bookout_qty(ARRAY['EC-00002', 'P001'])
```

**ข้อดี:**
- ✅ Clean และง่ายต่อการดูแล
- ✅ Performance ดีกว่า (query plan caching)
- ✅ Reusable (ใช้ได้ทั้ง chatbot และระบบอื่น)
- ✅ Template สั้น กระชับ

**ข้อเสีย:**
- ต้องสร้าง SP ในฐานข้อมูล
- ต้อง deploy ทุกครั้งที่ update logic

---

### Solution 3: Extended Template Format (Future)

ขยาย template format ให้รองรับ conditional queries:

```json
{
  "customQueries": [
    {
      "keyword": "ยอดค้างจอง|book out",
      "description": "ยอดค้างจอง",
      "type": "conditional",
      "conditions": [
        {
          "when": "has_item_codes",
          "sql": "WITH bookout as (...) WHERE item_code IN ({{ITEM_CODES}})"
        },
        {
          "when": "no_item_codes",
          "sql": "WITH bookout as (...) JOIN ic_inventory ..."
        }
      ],
      "parameters": {
        "item_codes": {
          "type": "array",
          "extract_from": "user_message"
        }
      }
    }
  ]
}
```

**ข้อดี:**
- Flexible มาก
- ครอบคลุมทุก use case

**ข้อเสีย:**
- ต้องแก้ code มาก (TemplateService, ChatService)
- ซับซ้อนในการดูแล

---

## 🎯 คำแนะนำ

สำหรับโปรเจค SML Chatbot:

### Phase 1 (ตอนนี้): Solution 1
- ใส่ Special Queries ตัวอย่างใน systemPrompt
- Test ว่า AI สามารถเรียนรู้และใช้ได้หรือไม่
- ดู token usage และ accuracy

### Phase 2 (ถ้า Phase 1 ไม่ดีพอ): Solution 2
- สร้าง Stored Procedures/Functions
- Migrate ทีละ query
- Update template ให้เรียกใช้ SP

### Phase 3 (Long term): Solution 3
- Design extended template format
- Implement ใน TemplateService
- Migrate ทั้งหมดไปใช้ format ใหม่

---

## 📝 ตัวอย่างการใช้งาน

### กรณีที่ 1: ใช้ systemPrompt (Simple)

**User ถาม:** "มียอดค้างจองเท่าไหร่"

**AI Response:**
1. อ่าน systemPrompt
2. เห็นคำสำคัญ "ยอดค้างจอง"
3. หา example SQL ที่เกี่ยวข้อง
4. ใช้ query กรณีที่ 2 (ไม่ระบุรหัส)
5. Execute และ return ผลลัพธ์

### กรณีที่ 2: ใช้ Stored Procedure (Clean)

**User ถาม:** "สินค้า EC-00002 มียอดค้างจองเท่าไหร่"

**AI Response:**
1. ตรวจจับ keyword "ยอดค้างจอง"
2. Extract รหัสสินค้า "EC-00002"
3. Generate SQL: `SELECT * FROM get_bookout_qty(ARRAY['EC-00002'])`
4. Execute และ return ผลลัพธ์

---

## 🔧 SQL Stored Procedures ที่แนะนำ

```sql
-- 1. ยอดค้างจอง
CREATE FUNCTION get_bookout_qty(item_codes TEXT[]) ...

-- 2. ยอดค้างส่ง
CREATE FUNCTION get_accrued_out_qty(item_codes TEXT[]) ...

-- 3. ยอดค้างรับ
CREATE FUNCTION get_accrued_in_qty(item_codes TEXT[]) ...

-- 4. ยอดคงเหลือ
CREATE FUNCTION get_stock_balance(item_codes TEXT[]) ...
```

---

**Recommendation:** เริ่มด้วย Solution 1 ใน Phase 1 แล้วประเมินผล ถ้าไม่ดีพอค่อยไป Solution 2

**Version:** 1.0  
**Created:** December 25, 2025
