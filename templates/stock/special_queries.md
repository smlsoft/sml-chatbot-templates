# Special Queries - แผนกคลังสินค้า

**Database:** sml1  
**Schema:** public  
**Purpose:** Query Templates และตัวอย่างสำหรับการคำนวณยอดค้างต่างๆ

**💡 หมายเหตุสำคัญ:**
- Queries ต่อไปนี้เป็น **Templates และตัวอย่าง** ที่สามารถปรับแต่งได้
- สามารถ **ประยุกต์ใช้ตามสถานการณ์** และ context ของคำถาม
- Keywords ที่ระบุเป็นเพียง **ตัวอย่าง** - คำถามที่มีความหมายใกล้เคียงสามารถใช้ template เดียวกันได้
- **เข้าใจหลักการทำงาน** ของแต่ละ template แล้วนำไปใช้ในกรณีที่เหมาะสม

---

## 1. ยอดค้างจอง (Book Out Quantity)

**ตัวอย่างกรณีใช้งาน:** "ยอดค้างจอง", "book out", "จองสินค้า", "ค้างจอง" และคำถามที่มีความหมายใกล้เคียง

**หลักการ:** ใช้ template นี้เป็นแนวทางสำหรับคำถามเกี่ยวกับยอดสินค้าที่ถูกจองไว้แต่ยังไม่ได้ส่งออก

### กรณีที่ 1: ระบุรหัสสินค้า

**ตัวอย่างคำถาม:** "สินค้า EC-00002 มียอดค้างจองเท่าไร", "ดูยอดค้างจองสินค้า P001"

```sql
WITH bookout as (
 select item_code, sum(bookout_qty_balance) as book_out_qty from (
 select doc_no, item_code, bookout_qty_balance from (
 select doc_no, item_code, qty
 -
 coalesce((
 select sum(qty * (stand_value/divide_value)) from ic_trans_detail
 where (
 (trans_flag = 44 and doc_ref_type = 2)
 or
 (trans_flag = 36 and doc_ref_type = 2)
 or
 ( ic_trans_detail.trans_flag = 39 and (select cancel_type from ic_trans where ic_trans.doc_no = ic_trans_detail.doc_no and ic_trans_detail.trans_flag = ic_trans.trans_flag) = 2)
 ) and last_status = 0 and ic_trans_detail.ref_doc_no = temp1.doc_no and ic_trans_detail.item_code = temp1.item_code
 ), 0) as bookout_qty_balance
 from (
 select doc_no, item_code
 , sum(qty * (stand_value/divide_value)) as qty
 from ic_trans_detail where trans_flag=34 and last_status = 0 and item_code IN ('ITEM_CODE_HERE')
 group by doc_no, item_code
 ) as temp1
 ) as temp2
 ) as temp3 group by item_code
 )
SELECT
  b.item_code,
  i.name_1 as item_name,
  ROUND(b.book_out_qty, 2) as book_out_qty,
  i.unit_standard_name
FROM bookout b
LEFT JOIN ic_inventory i ON b.item_code = i.code
WHERE b.book_out_qty <> 0
ORDER BY b.book_out_qty DESC;
```

**วิธีใช้:** แทนที่ `'ITEM_CODE_HERE'` ด้วยรหัสสินค้า (สามารถใส่หลายรหัสได้)

**ผลลัพธ์:** แสดงรหัสสินค้า, ชื่อสินค้า, ยอดค้างจอง (ทศนิยม 2 ตำแหน่ง), และหน่วยนับ

### กรณีที่ 2: ไม่ระบุรหัสสินค้า

**ตัวอย่างคำถาม:** "สินค้าไหนมียอดค้างจองบ้าง", "แสดงยอดค้างจองทั้งหมด"

```sql
WITH bookout as (
 select item_code, sum(bookout_qty_balance) as book_out_qty from (
 select doc_no, item_code, bookout_qty_balance from (
 select doc_no, item_code, qty
 -
 coalesce((
 select sum(qty * (stand_value/divide_value)) from ic_trans_detail
 where (
 (trans_flag = 44 and doc_ref_type = 2)
 or
 (trans_flag = 36 and doc_ref_type = 2)
 or
 ( ic_trans_detail.trans_flag = 39 and (select cancel_type from ic_trans where ic_trans.doc_no = ic_trans_detail.doc_no and ic_trans_detail.trans_flag = ic_trans.trans_flag) = 2)
 ) and last_status = 0 and ic_trans_detail.ref_doc_no = temp1.doc_no and ic_trans_detail.item_code = temp1.item_code
 ), 0) as bookout_qty_balance
 from (
 select doc_no, item_code
 , sum(qty * (stand_value/divide_value)) as qty
 from ic_trans_detail where trans_flag=34 and last_status = 0
 group by doc_no, item_code
 ) as temp1
 ) as temp2
 ) as temp3 group by item_code
 )
SELECT
  b.item_code,
  i.name_1 as item_name,
  ROUND(b.book_out_qty, 2) as book_out_qty,
  i.unit_standard_name
FROM bookout b
LEFT JOIN ic_inventory i ON b.item_code = i.code
WHERE b.book_out_qty <> 0
ORDER BY b.book_out_qty DESC
LIMIT 30;
```

**ผลลัพธ์:** แสดง 30 รายการที่มียอดค้างจองมากที่สุด

---

## 2. ยอดค้างส่ง (Accrued Out Quantity)

**ตัวอย่างกรณีใช้งาน:** "ยอดค้างส่ง", "accrued out", "ค้างส่ง", "รอส่ง" และคำถามที่มีความหมายใกล้เคียง

**หลักการ:** ใช้ template นี้เป็นแนวทางสำหรับคำถามเกี่ยวกับยอดสินค้าที่รอการส่งออก

### กรณีที่ 1: ระบุรหัสสินค้า

**ตัวอย่างคำถาม:** "สินค้า 02-15061386 มียอดค้างส่งเท่าไร", "ดูยอดค้างส่งสินค้า P001"

```sql
WITH accout as (
 select item_code, sum(accout_qty_balance) as acc_out_qty from (
 select doc_no, item_code, accout_qty_balance from (
 select doc_no, item_code, qty
 -
 coalesce((
 select sum(qty * (stand_value/divide_value)) from ic_trans_detail
 where (
 (trans_flag = 44 and doc_ref_type = 3)
 or
 ( ic_trans_detail.trans_flag = 37 and (select cancel_type from ic_trans where ic_trans.doc_no = ic_trans_detail.doc_no and ic_trans_detail.trans_flag = ic_trans.trans_flag) = 2)
 ) and last_status = 0 and ic_trans_detail.ref_doc_no = temp1.doc_no and ic_trans_detail.item_code = temp1.item_code
 ), 0) as accout_qty_balance
 from (
 select doc_no, item_code
 , sum(qty * (stand_value/divide_value)) as qty
 from ic_trans_detail where trans_flag=36 and last_status = 0 and item_code IN ('ITEM_CODE_HERE')
 group by doc_no, item_code
 ) as temp1
 ) as temp2
 ) as temp3 group by item_code
 )
SELECT
  a.item_code,
  i.name_1 as item_name,
  ROUND(a.acc_out_qty, 2) as acc_out_qty,
  i.unit_standard_name
FROM accout a
LEFT JOIN ic_inventory i ON a.item_code = i.code
WHERE a.acc_out_qty <> 0
ORDER BY a.acc_out_qty DESC;
```

**วิธีใช้:** แทนที่ `'ITEM_CODE_HERE'` ด้วยรหัสสินค้า (สามารถใส่หลายรหัสได้)

**ผลลัพธ์:** แสดงรหัสสินค้า, ชื่อสินค้า, ยอดค้างส่ง (ทศนิยม 2 ตำแหน่ง), และหน่วยนับ

### กรณีที่ 2: ไม่ระบุรหัสสินค้า

**ตัวอย่างคำถาม:** "สินค้าไหนมียอดค้างส่งบ้าง", "แสดงยอดค้างส่งทั้งหมด"

```sql
WITH accout as (
 select item_code, sum(accout_qty_balance) as acc_out_qty from (
 select doc_no, item_code, accout_qty_balance from (
 select doc_no, item_code, qty
 -
 coalesce((
 select sum(qty * (stand_value/divide_value)) from ic_trans_detail
 where (
 (trans_flag = 44 and doc_ref_type = 3)
 or
 ( ic_trans_detail.trans_flag = 37 and (select cancel_type from ic_trans where ic_trans.doc_no = ic_trans_detail.doc_no and ic_trans_detail.trans_flag = ic_trans.trans_flag) = 2)
 ) and last_status = 0 and ic_trans_detail.ref_doc_no = temp1.doc_no and ic_trans_detail.item_code = temp1.item_code
 ), 0) as accout_qty_balance
 from (
 select doc_no, item_code
 , sum(qty * (stand_value/divide_value)) as qty
 from ic_trans_detail where trans_flag=36 and last_status = 0
 group by doc_no, item_code
 ) as temp1
 ) as temp2
 ) as temp3 group by item_code
 )
SELECT
  a.item_code,
  i.name_1 as item_name,
  ROUND(a.acc_out_qty, 2) as acc_out_qty,
  i.unit_standard_name
FROM accout a
LEFT JOIN ic_inventory i ON a.item_code = i.code
WHERE a.acc_out_qty <> 0
ORDER BY a.acc_out_qty DESC
LIMIT 30;
```

**ผลลัพธ์:** แสดง 30 รายการที่มียอดค้างส่งมากที่สุด

---

## 3. ยอดค้างรับ (Accrued In Quantity)

**ตัวอย่างกรณีใช้งาน:** "ยอดค้างรับ", "accrued in", "ค้างรับ", "รอรับ" และคำถามที่มีความหมายใกล้เคียง

**หลักการ:** ใช้ template นี้เป็นแนวทางสำหรับคำถามเกี่ยวกับยอดสินค้าที่รอการรับเข้าคลัง

### กรณีที่ 1: ระบุรหัสสินค้า

**ตัวอย่างคำถาม:** "สินค้า AC-00008 มียอดค้างรับเท่าไร", "ดูยอดค้างรับสินค้า P001"

```sql
SELECT 
  acc.item_code,
  i.name_1 as item_name,
  ROUND(acc.acc_in_balance, 2) as acc_in_balance,
  i.unit_standard_name
FROM (
  select item_code, sum(acc_in_balance) as acc_in_balance
  from (
   select doc_no, item_code, acc_in_balance from (
   select doc_no, item_code, qty
   -
   coalesce((
   select sum(qty * (stand_value/divide_value)) from ic_trans_detail
   where (
   trans_flag in (12,310)
   or
   ( ic_trans_detail.trans_flag = 7 and (select cancel_type from ic_trans where ic_trans.doc_no = ic_trans_detail.doc_no and ic_trans_detail.trans_flag = ic_trans.trans_flag) = 2)
   ) and last_status = 0 and ic_trans_detail.ref_doc_no = temp1.doc_no and ic_trans_detail.item_code = temp1.item_code
   ), 0) as acc_in_balance
   from (
   select doc_no, item_code
   , sum(qty * (stand_value/divide_value)) as qty
   from ic_trans_detail where trans_flag=6 and last_status = 0 and item_code IN ('ITEM_CODE_HERE')
   group by doc_no, item_code
   ) as temp1
   ) as temp2 where acc_in_balance <> 0
  ) as temp3
  group by item_code
) acc
LEFT JOIN ic_inventory i ON acc.item_code = i.code
WHERE acc.acc_in_balance <> 0
ORDER BY acc.acc_in_balance DESC;
```

**วิธีใช้:** แทนที่ `'ITEM_CODE_HERE'` ด้วยรหัสสินค้า (สามารถใส่หลายรหัสได้)

**ผลลัพธ์:** แสดงรหัสสินค้า, ชื่อสินค้า, ยอดค้างรับ (ทศนิยม 2 ตำแหน่ง), และหน่วยนับ

### กรณีที่ 1.5: ระบุชื่อสินค้า

**ตัวอย่างคำถาม:** "<ชื่อสินค้า> มียอดค้างรับเท่าไร", "<ชื่อสินค้าอื่น> ค้างรับเท่าไหร่"

**หมายเหตุ:** <ชื่อสินค้า> = ชื่อจริงในฐานข้อมูลของแต่ละร้าน

**⚠️ สำคัญ:** เมื่อ user ให้**ชื่อสินค้า** (ไม่ใช่รหัส) → ต้องค้นหารหัสก่อน แล้วค่อยใช้กับ Special Query

```sql
SELECT 
  acc.item_code,
  i.name_1 as item_name,
  ROUND(acc.acc_in_balance, 2) as acc_in_balance,
  i.unit_standard_name
FROM (
  select item_code, sum(acc_in_balance) as acc_in_balance
  from (
   select doc_no, item_code, acc_in_balance from (
   select doc_no, item_code, qty
   -
   coalesce((
   select sum(qty * (stand_value/divide_value)) from ic_trans_detail
   where (
   trans_flag in (12,310)
   or
   ( ic_trans_detail.trans_flag = 7 and (select cancel_type from ic_trans where ic_trans.doc_no = ic_trans_detail.doc_no and ic_trans_detail.trans_flag = ic_trans.trans_flag) = 2)
   ) and last_status = 0 and ic_trans_detail.ref_doc_no = temp1.doc_no and ic_trans_detail.item_code = temp1.item_code
   ), 0) as acc_in_balance
   from (
   select doc_no, item_code
   , sum(qty * (stand_value/divide_value)) as qty
   from ic_trans_detail 
   where trans_flag=6 and last_status = 0 
     and item_code IN (
       SELECT code FROM ic_inventory WHERE name_1 LIKE '%ITEM_NAME_HERE%'
     )
   group by doc_no, item_code
   ) as temp1
   ) as temp2 where acc_in_balance <> 0
  ) as temp3
  group by item_code
) acc
LEFT JOIN ic_inventory i ON acc.item_code = i.code
WHERE acc.acc_in_balance <> 0
ORDER BY acc.acc_in_balance DESC;
```

**วิธีใช้:** แทนที่ `'%ITEM_NAME_HERE%'` ด้วยชื่อสินค้า (แยกคำสำคัญออกมา)

**ผลลัพธ์:** แสดงรหัสสินค้า, ชื่อสินค้า, ยอดค้างรับ (ทศนิยม 2 ตำแหน่ง), และหน่วยนับ

**⚠️ สำคัญ - การแยกคำสำคัญ:**
- User พิมพ์: "<ชื่อสินค้า> <คุณสมบัติ> <ขนาด>"
- แทนที่: `WHERE name_1 LIKE '%<คำ1>%' OR name_1 LIKE '%<คำ2>%' OR name_1 LIKE '%<คำ3>%'`
- **ใช้ OR** เพื่อให้ค้นหาได้กว้างขวาง ตัดคำถามได้มากที่สุด (มีคำใดคำหนึ่งก็ถือว่าเจอ)
- ห้าม hardcode ชื่อสินค้าเฉพาะ - ต้องค้นหาได้ทุกสินค้า

### กรณีที่ 2: ไม่ระบุรหัสสินค้า

**ตัวอย่างคำถาม:** "สินค้าไหนมียอดค้างรับบ้าง", "แสดงยอดค้างรับทั้งหมด"

```sql
WITH accin as (
 select item_code, sum(acc_in_balance) as acc_in_balance
 from (
  select doc_no, item_code, acc_in_balance from (
  select doc_no, item_code, qty
  -
  coalesce((
  select sum(qty * (stand_value/divide_value)) from ic_trans_detail
  where (
  trans_flag in (12,310)
  or
  ( ic_trans_detail.trans_flag = 7 and (select cancel_type from ic_trans where ic_trans.doc_no = ic_trans_detail.doc_no and ic_trans_detail.trans_flag = ic_trans.trans_flag) = 2)
  ) and last_status = 0 and ic_trans_detail.ref_doc_no = temp1.doc_no and ic_trans_detail.item_code = temp1.item_code
  ), 0) as acc_in_balance
  from (
  select doc_no, item_code
  , sum(qty * (stand_value/divide_value)) as qty
  from ic_trans_detail where trans_flag=6 and last_status = 0
  group by doc_no, item_code
  ) as temp1
  ) as temp2 where acc_in_balance <> 0
 ) as temp3
 group by item_code
)
SELECT
  a.item_code,
  i.name_1 as item_name,
  ROUND(a.acc_in_balance, 2) as acc_in_balance,
  i.unit_standard_name
FROM accin a
LEFT JOIN ic_inventory i ON a.item_code = i.code
WHERE a.acc_in_balance <> 0
ORDER BY a.acc_in_balance DESC
LIMIT 30;
```

**ผลลัพธ์:** แสดง 30 รายการที่มียอดค้างรับมากที่สุด (ทศนิยม 2 ตำแหน่ง)

---

## 4. ยอดคงเหลือสินค้า แยกตามคลังและพื้นที่เก็บ (Stock Balance by Warehouse & Location)

**ตัวอย่างกรณีใช้งาน:** "ยอดคงเหลือ", "stock balance", "สต็อกคงเหลือ", "คงเหลือในคลัง", "สินค้าคงเหลือ", "ยอดสินค้า", "อยู่คลังไหน", "เก็บที่ไหน" และคำถามที่มีความหมายใกล้เคียง

**หลักการ:** ใช้ template นี้เป็นแนวทางสำหรับคำถามเกี่ยวกับสินค้าคงเหลือในคลัง สามารถปรับแต่ง WHERE clause ตาม context

### กรณีที่ 1: ระบุรหัสสินค้า

**ตัวอย่างคำถาม:** "สินค้า 885 มียอดคงเหลือเท่าไร", "ดูยอดคงเหลือสินค้า AC-00001", "สินค้า BT-00002 อยู่คลังไหนบ้าง"

```sql
select ic_code, warehouse, location, ic_name, ic_unit_code,
ROUND(balance_qty/unit_standard_ratio, 2) as balance_qty,
ROUND(average_cost*unit_standard_ratio, 2) as average_cost,
ROUND(average_cost_end, 2) as average_cost_end,
ROUND(balance_amount, 2) as balance_amount,
ROUND(qty_in/unit_standard_ratio, 2) as qty_in,
ROUND(amount_in, 2) as amount_in,
ROUND(average_cost_in*unit_standard_ratio, 2) as average_cost_in,
ROUND(qty_out/unit_standard_ratio, 2) as qty_out,
ROUND(amount_out, 2) as amount_out,
ROUND(average_cost_out*unit_standard_ratio, 2) as average_cost_out
from (select coalesce((select stand_value/divide_value from ic_unit_use where ic_unit_use.ic_code=temp2.ic_code and ic_unit_use.code=temp2.ic_unit_code),1) as unit_standard_ratio
	  ,ic_code,warehouse,location, ic_name, balance_qty, ic_unit_code, case when balance_qty=0 then 0 else balance_amount/balance_qty end as average_cost
	  , coalesce(((select  average_cost from ic_trans_detail where ic_trans_detail.last_status=0  and ic_trans_detail.item_code=temp2.ic_code and doc_date_calc<=CURRENT_DATE
				   and ((trans_flag in (70,54,60,58,310,12) or (trans_flag=66 and (qty>0 or sum_of_cost>0 )) or (trans_flag=14) or (trans_flag=48 and inquiry_type < 2))
						or (trans_flag in (56,68,72,44) or (trans_flag=66 and (qty<0 or sum_of_cost<0 )) or (trans_flag=46)  or (trans_flag=16 ) or (trans_flag=311 ))
						and not (ic_trans_detail.doc_ref <> '' and ic_trans_detail.is_pos = 1)) order by doc_date_calc desc,doc_time desc ,line_number desc  offset 0 limit 1 )*unit_ratio),0) as average_cost_end
	  ,  balance_amount as balance_amount
	  , qty_in, amount_in, case when qty_in=0 then 0 else amount_in/qty_in end as average_cost_in
	  , qty_out, amount_out, case when qty_out=0 then 0 else amount_out/qty_out end as average_cost_out
	  from (select ic_code,warehouse,location, ic_name, balance_qty, ic_unit_code, (select unit_standard_stand_value/unit_standard_divide_value from ic_inventory where ic_inventory.code=temp1.ic_code) as unit_ratio
			, balance_amount, qty_in, amount_in, qty_out, amount_out
			from (select item_code as ic_code,wh_code as warehouse,shelf_code as location, (select name_1 from ic_inventory where ic_inventory.code=item_code) as ic_name
				  , (select unit_standard from ic_inventory where ic_inventory.code=item_code) as ic_unit_code
				  , coalesce(sum(calc_flag*(case when ((trans_flag in (70,54,60,58,310,12) or (trans_flag=66 and qty>0) or (trans_flag=14 and inquiry_type=0) or (trans_flag=48 and inquiry_type < 2))
													   or (trans_flag in (56,68,72,44) or (trans_flag=66 and qty<0) or (trans_flag=46 and inquiry_type in (0,2))
														   or (trans_flag=16 and inquiry_type in (0,2)) or (trans_flag=311 and inquiry_type=0))
													   and not (ic_trans_detail.doc_ref <> '' and ic_trans_detail.is_pos = 1)) then round((qty*stand_value) / divide_value,2) else 0 end)),0) as balance_qty
				  , coalesce(sum( (calc_flag*(case when ((trans_flag in (70,54,60,58,310,12) or (trans_flag=66 and (qty>0 or sum_of_cost>0 )) or (trans_flag=14) or (trans_flag=48 and inquiry_type < 2))
														 or (trans_flag in (56,68,72,44) or (trans_flag=66 and (qty<0 or sum_of_cost<0 )) or (trans_flag=46)  or (trans_flag=16 ) or (trans_flag=311 ))
														 and not (ic_trans_detail.doc_ref <> '' and ic_trans_detail.is_pos = 1))
											  then(case when trans_flag = 66 and qty < 0 then(-1 * sum_of_cost) + coalesce(profit_lost_cost_amount, 0)
												   else (sum_of_cost + coalesce(profit_lost_cost_amount, 0)) end) else 0 end))),0) as balance_amount
				  , sum(case when doc_date_calc>=CURRENT_DATE - INTERVAL '1 year' and (trans_flag in (70,54,60,58,310,12) or (trans_flag=66 and qty>0) or (trans_flag=14 and inquiry_type=0) or (trans_flag=48 and inquiry_type < 2))
						then calc_flag*round((qty*stand_value)/divide_value,2) else 0 end) as qty_in
				  ,sum(case when doc_date_calc>=CURRENT_DATE - INTERVAL '1 year' and (trans_flag in (70,54,60,58,310,12) or (trans_flag=66 and (qty>0 or sum_of_cost>0 )) or (trans_flag=14) or (trans_flag=48 and inquiry_type < 2))
					   then ((calc_flag*sum_of_cost)+coalesce(profit_lost_cost_amount, 0)) else 0 end) as amount_in
				  , -1*sum(case when doc_date_calc>=CURRENT_DATE - INTERVAL '1 year' and (trans_flag in (56,68,72,44) or (trans_flag=66 and qty<0) or (trans_flag=46 and inquiry_type in (0,2))
																	  or (trans_flag=16 and inquiry_type in (0,2)) or (trans_flag=311 and inquiry_type=0))
						   and not (ic_trans_detail.doc_ref <> '' and ic_trans_detail.is_pos = 1) then calc_flag*round((qty*stand_value)/divide_value,2) else 0 end) as qty_out
				  ,-1 * sum(case when doc_date_calc>=CURRENT_DATE - INTERVAL '1 year' and (trans_flag in (56,68,72,44) or (trans_flag=66 and (qty<0 or sum_of_cost<0 )) or (trans_flag=46)  or (trans_flag=16 ) or (trans_flag=311 ))
							and not (ic_trans_detail.doc_ref <> '' and ic_trans_detail.is_pos = 1)
							then(((case when trans_flag=66 and qty < 0 then -1 else calc_flag end)  *(sum_of_cost + coalesce(profit_lost_cost_amount, 0)))) else 0 end) as amount_out
				  from ic_trans_detail
				  where ic_trans_detail.last_status=0  and ic_trans_detail.item_type<>5  and ic_trans_detail.is_doc_copy =0
				  and (select item_type from ic_inventory where ic_inventory.code = ic_trans_detail.item_code) not in (1,3)  and doc_date_calc<=CURRENT_DATE
				  and item_code IN ('ITEM_CODE_HERE')
				  group by item_code, wh_code, shelf_code
				 ) as temp1
		   ) as temp2
	  WHERE (qty_in<>0  or amount_in<>0 or qty_out<>0 or amount_out<>0 or balance_qty<>0 or balance_amount<>0)
	 ) as final
	 order by ic_code,warehouse,location
```

**วิธีใช้:** แทนที่ `'ITEM_CODE_HERE'` ด้วยรหัสสินค้า (สามารถใส่หลายรหัสได้)

**ผลลัพธ์:** แสดงยอดคงเหลือแยกตามคลัง(warehouse) และพื้นที่เก็บ(location) พร้อมข้อมูล (ทศนิยม 2 ตำแหน่ง):

- ic_code, warehouse, location, ic_name, ic_unit_code
- balance_qty (ยอดคงเหลือ), average_cost (ต้นทุนเฉลี่ย)
- balance_amount (มูลค่าคงเหลือ)
- qty_in, amount_in (รับเข้าปีล่าสุด)
- qty_out, amount_out (จ่ายออกปีล่าสุด)

### กรณีที่ 2: ไม่ระบุรหัสสินค้า (ดูยอดคงเหลือทั้งหมด)

**ตัวอย่างคำถาม:** "แสดงยอดคงเหลือทั้งหมด", "สินค้าไหนมียอดคงเหลือบ้าง", "ดูสต็อกคงเหลือ"

```sql
select ic_code, warehouse, location, ic_name, ic_unit_code,
ROUND(balance_qty/unit_standard_ratio, 2) as balance_qty,
ROUND(average_cost*unit_standard_ratio, 2) as average_cost,
ROUND(average_cost_end, 2) as average_cost_end,
ROUND(balance_amount, 2) as balance_amount,
ROUND(qty_in/unit_standard_ratio, 2) as qty_in,
ROUND(amount_in, 2) as amount_in,
ROUND(average_cost_in*unit_standard_ratio, 2) as average_cost_in,
ROUND(qty_out/unit_standard_ratio, 2) as qty_out,
ROUND(amount_out, 2) as amount_out,
ROUND(average_cost_out*unit_standard_ratio, 2) as average_cost_out
from (select coalesce((select stand_value/divide_value from ic_unit_use where ic_unit_use.ic_code=temp2.ic_code and ic_unit_use.code=temp2.ic_unit_code),1) as unit_standard_ratio
	  ,ic_code,warehouse,location, ic_name, balance_qty, ic_unit_code, case when balance_qty=0 then 0 else balance_amount/balance_qty end as average_cost
	  , coalesce(((select  average_cost from ic_trans_detail where ic_trans_detail.last_status=0  and ic_trans_detail.item_code=temp2.ic_code and doc_date_calc<=CURRENT_DATE
				   and ((trans_flag in (70,54,60,58,310,12) or (trans_flag=66 and (qty>0 or sum_of_cost>0 )) or (trans_flag=14) or (trans_flag=48 and inquiry_type < 2))
						or (trans_flag in (56,68,72,44) or (trans_flag=66 and (qty<0 or sum_of_cost<0 )) or (trans_flag=46)  or (trans_flag=16 ) or (trans_flag=311 ))
						and not (ic_trans_detail.doc_ref <> '' and ic_trans_detail.is_pos = 1)) order by doc_date_calc desc,doc_time desc ,line_number desc  offset 0 limit 1 )*unit_ratio),0) as average_cost_end
	  ,  balance_amount as balance_amount
	  , qty_in, amount_in, case when qty_in=0 then 0 else amount_in/qty_in end as average_cost_in
	  , qty_out, amount_out, case when qty_out=0 then 0 else amount_out/qty_out end as average_cost_out
	  from (select ic_code,warehouse,location, ic_name, balance_qty, ic_unit_code, (select unit_standard_stand_value/unit_standard_divide_value from ic_inventory where ic_inventory.code=temp1.ic_code) as unit_ratio
			, balance_amount, qty_in, amount_in, qty_out, amount_out
			from (select item_code as ic_code,wh_code as warehouse,shelf_code as location, (select name_1 from ic_inventory where ic_inventory.code=item_code) as ic_name
				  , (select unit_standard from ic_inventory where ic_inventory.code=item_code) as ic_unit_code
				  , coalesce(sum(calc_flag*(case when ((trans_flag in (70,54,60,58,310,12) or (trans_flag=66 and qty>0) or (trans_flag=14 and inquiry_type=0) or (trans_flag=48 and inquiry_type < 2))
													   or (trans_flag in (56,68,72,44) or (trans_flag=66 and qty<0) or (trans_flag=46 and inquiry_type in (0,2))
														   or (trans_flag=16 and inquiry_type in (0,2)) or (trans_flag=311 and inquiry_type=0))
													   and not (ic_trans_detail.doc_ref <> '' and ic_trans_detail.is_pos = 1)) then round((qty*stand_value) / divide_value,2) else 0 end)),0) as balance_qty
				  , coalesce(sum( (calc_flag*(case when ((trans_flag in (70,54,60,58,310,12) or (trans_flag=66 and (qty>0 or sum_of_cost>0 )) or (trans_flag=14) or (trans_flag=48 and inquiry_type < 2))
														 or (trans_flag in (56,68,72,44) or (trans_flag=66 and (qty<0 or sum_of_cost<0 )) or (trans_flag=46)  or (trans_flag=16 ) or (trans_flag=311 ))
														 and not (ic_trans_detail.doc_ref <> '' and ic_trans_detail.is_pos = 1))
											  then(case when trans_flag = 66 and qty < 0 then(-1 * sum_of_cost) + coalesce(profit_lost_cost_amount, 0)
												   else (sum_of_cost + coalesce(profit_lost_cost_amount, 0)) end) else 0 end))),0) as balance_amount
				  , sum(case when doc_date_calc>=CURRENT_DATE - INTERVAL '1 year' and (trans_flag in (70,54,60,58,310,12) or (trans_flag=66 and qty>0) or (trans_flag=14 and inquiry_type=0) or (trans_flag=48 and inquiry_type < 2))
						then calc_flag*round((qty*stand_value)/divide_value,2) else 0 end) as qty_in
				  ,sum(case when doc_date_calc>=CURRENT_DATE - INTERVAL '1 year' and (trans_flag in (70,54,60,58,310,12) or (trans_flag=66 and (qty>0 or sum_of_cost>0 )) or (trans_flag=14) or (trans_flag=48 and inquiry_type < 2))
					   then ((calc_flag*sum_of_cost)+coalesce(profit_lost_cost_amount, 0)) else 0 end) as amount_in
				  , -1*sum(case when doc_date_calc>=CURRENT_DATE - INTERVAL '1 year' and (trans_flag in (56,68,72,44) or (trans_flag=66 and qty<0) or (trans_flag=46 and inquiry_type in (0,2))
																	  or (trans_flag=16 and inquiry_type in (0,2)) or (trans_flag=311 and inquiry_type=0))
						   and not (ic_trans_detail.doc_ref <> '' and ic_trans_detail.is_pos = 1) then calc_flag*round((qty*stand_value)/divide_value,2) else 0 end) as qty_out
				  ,-1 * sum(case when doc_date_calc>=CURRENT_DATE - INTERVAL '1 year' and (trans_flag in (56,68,72,44) or (trans_flag=66 and (qty<0 or sum_of_cost<0 )) or (trans_flag=46)  or (trans_flag=16 ) or (trans_flag=311 ))
							and not (ic_trans_detail.doc_ref <> '' and ic_trans_detail.is_pos = 1)
							then(((case when trans_flag=66 and qty < 0 then -1 else calc_flag end)  *(sum_of_cost + coalesce(profit_lost_cost_amount, 0)))) else 0 end) as amount_out
				  from ic_trans_detail
				  where ic_trans_detail.last_status=0  and ic_trans_detail.item_type<>5  and ic_trans_detail.is_doc_copy =0
				  and (select item_type from ic_inventory where ic_inventory.code = ic_trans_detail.item_code) not in (1,3)  and doc_date_calc<=CURRENT_DATE
				  group by item_code, wh_code, shelf_code
				 ) as temp1
		   ) as temp2
	  WHERE (qty_in<>0  or amount_in<>0 or qty_out<>0 or amount_out<>0 or balance_qty<>0 or balance_amount<>0)
	 ) as final
	 where balance_qty <> 0
	 order by balance_qty desc, ic_code, warehouse, location
	 LIMIT 100
```

**ผลลัพธ์:** แสดง 100 รายการที่มียอดคงเหลือมากที่สุด (ทศนิยม 2 ตำแหน่ง)

---

## หมายเหตุสำคัญ

### การใช้งาน Special Queries

- Query เหล่านี้เป็น **Business Logic ที่ซับซ้อน** ห้าม AI สร้างเอง
- **ต้องใช้ Query ตามที่ระบุเท่านั้น** ห้ามแก้ไขโครงสร้าง
- เมื่อระบุรหัสสินค้า: แทนที่ `'ITEM_CODE_HERE'`
- เมื่อไม่ระบุรหัส: ใช้ Query กรณีที่ 2 (มี JOIN กับ ic_inventory)

### ผลลัพธ์

- **กรณีที่ 1:** ได้เฉพาะ item_code และจำนวน
- **กรณีที่ 2:** ได้ item_code, item_name, จำนวน, unit_standard_name

### การขยายในอนาคต

- สามารถเพิ่ม Special Query อื่นๆ ได้ในไฟล์นี้
- แต่ละ Query ควรมี 2 กรณี (ระบุรหัส / ไม่ระบุรหัส)

---

## 5. สินค้าขายดี (Best Selling Items)

**ตัวอย่างกรณีใช้งาน:** "สินค้าขายดี", "ขายดีที่สุด", "best seller", "ขายเยอะ", "นิยม" และคำถามที่มีความหมายใกล้เคียง

**หลักการ:** ใช้ template นี้เป็นแนวทางสำหรับคำถามเกี่ยวกับสินค้าที่ขายดี สามารถปรับแต่งช่วงเวลาหรือสถิติได้

**ตัวอย่างคำถาม:** "สินค้าไหนขายดีที่สุด", "แสดงสินค้าขายดี Top 10", "สินค้าที่ขายเยอะที่สุด"

```sql
SELECT 
  itd.item_code,
  i.name_1 as item_name,
  ROUND(SUM(ABS(itd.qty * itd.stand_value / itd.divide_value)), 2) as total_qty_sold,
  i.unit_standard_name,
  COUNT(DISTINCT itd.doc_no) as transaction_count
FROM ic_trans_detail itd
LEFT JOIN ic_inventory i ON itd.item_code = i.code
WHERE itd.last_status = 0
  AND itd.trans_flag IN (44, 56, 68, 72)
  AND itd.doc_date_calc >= CURRENT_DATE - INTERVAL '1 year'
  AND i.item_type NOT IN (1, 3)
GROUP BY itd.item_code, i.name_1, i.unit_standard_name
HAVING SUM(ABS(itd.qty * itd.stand_value / itd.divide_value)) > 0
ORDER BY total_qty_sold DESC
LIMIT 30;
```

**ผลลัพธ์:** แสดง 30 รายการสินค้าที่ขายดีที่สุดในรอบ 1 ปี พร้อมปริมาณที่ขายและจำนวนครั้งที่ขาย

**คำอธิบาย:**
- `trans_flag IN (44, 56, 68, 72)` = เอกสารจ่ายออก/ขาย
- ใช้ข้อมูลย้อนหลัง 1 ปี
- เรียงตามปริมาณที่ขายมากที่สุด

---

## 6. สินค้าในคลัง (Items in Warehouse)

**ตัวอย่างกรณีใช้งาน:** "สินค้าในคลัง", "ที่คลัง", "อยู่คลัง", "warehouse" และคำถามที่มีความหมายใกล้เคียง

**หลักการ:** ใช้ template นี้เป็นแนวทางสำหรับคำถามเกี่ยวกับสินค้าที่เก็บในคลัง สามารถปรับแต่ง WHERE clause ตาม context

### กรณีที่ 1: ระบุชื่อคลัง หรือ ค้นหาสินค้า

**ตัวอย่างคำถาม:** 
- "สินค้าที่อยู่ที่คลัง <ชื่อคลัง> มีอะไรบ้าง"
- "แสดงสินค้าในคลัง <ชื่อคลัง>"
- "หาสินค้า <รหัสสินค้า> อยู่คลังไหนบ้าง"
- "สินค้าที่มีชื่อว่า <ชื่อสินค้า> อยู่คลังไหน"

**หมายเหตุ:** <ชื่อคลัง>, <รหัสสินค้า>, <ชื่อสินค้า> = ข้อมูลจริงของแต่ละร้าน

**กรณีที่ 1.1: ค้นหาตามชื่อคลัง**
```sql
SELECT DISTINCT
  itd.item_code,
  i.name_1 as item_name,
  itd.wh_code as warehouse,
  i.unit_standard_name,
  ROUND(SUM(itd.calc_flag * ROUND((itd.qty * itd.stand_value) / itd.divide_value, 2)), 2) as balance_qty
FROM ic_trans_detail itd
LEFT JOIN ic_inventory i ON itd.item_code = i.code
WHERE itd.last_status = 0
  AND itd.item_type <> 5
  AND itd.is_doc_copy = 0
  AND i.item_type NOT IN (1, 3)
  AND itd.doc_date_calc <= CURRENT_DATE
  AND UPPER(itd.wh_code) = UPPER('WAREHOUSE_NAME_HERE')
GROUP BY itd.item_code, i.name_1, itd.wh_code, i.unit_standard_name
HAVING SUM(itd.calc_flag * ROUND((itd.qty * itd.stand_value) / itd.divide_value, 2)) <> 0
ORDER BY balance_qty DESC
LIMIT 100;
```
**วิธีใช้:** แทนที่ `'WAREHOUSE_NAME_HERE'` ด้วยชื่อคลัง

**กรณีที่ 1.2: ค้นหาตามรหัสสินค้าหรือชื่อสินค้า (ดูว่าอยู่คลังไหน)**
```sql
SELECT DISTINCT
  itd.item_code,
  i.name_1 as item_name,
  itd.wh_code as warehouse,
  itd.shelf_code as location,
  i.unit_standard_name,
  ROUND(SUM(itd.calc_flag * ROUND((itd.qty * itd.stand_value) / itd.divide_value, 2)), 2) as balance_qty
FROM ic_trans_detail itd
LEFT JOIN ic_inventory i ON itd.item_code = i.code
WHERE itd.last_status = 0
  AND itd.item_type <> 5
  AND itd.is_doc_copy = 0
  AND i.item_type NOT IN (1, 3)
  AND itd.doc_date_calc <= CURRENT_DATE
  AND (
    itd.item_code = 'ITEM_CODE_HERE'
    OR UPPER(i.name_1) LIKE UPPER('%ITEM_NAME_HERE%')
  )
GROUP BY itd.item_code, i.name_1, itd.wh_code, itd.shelf_code, i.unit_standard_name
HAVING SUM(itd.calc_flag * ROUND((itd.qty * itd.stand_value) / itd.divide_value, 2)) <> 0
ORDER BY itd.wh_code, balance_qty DESC
LIMIT 100;
```
**วิธีใช้:** 
- แทนที่ `'ITEM_CODE_HERE'` ด้วยรหัสสินค้า หรือ
- แทนที่ `'%ITEM_NAME_HERE%'` ด้วยชื่อสินค้า (ใช้ % ครอบ)
- หรือใช้ทั้ง 2 조건 โดย AI จะเลือกตาม context

**ผลลัพธ์:** แสดงว่าสินค้านั้นอยู่คลังไหนบ้าง พร้อมยอดคงเหลือ (ทศนิยม 2 ตำแหน่ง)

### กรณีที่ 2: แสดงสินค้าทุกคลัง (สรุปตามคลัง)

**ตัวอย่างคำถาม:** "สินค้าอยู่คลังไหนบ้าง", "แสดงสินค้าแยกตามคลัง"

```sql
SELECT 
  itd.wh_code as warehouse,
  COUNT(DISTINCT itd.item_code) as item_count,
  ROUND(SUM(ABS(itd.calc_flag * ROUND((itd.qty * itd.stand_value) / itd.divide_value, 2))), 2) as total_qty
FROM ic_trans_detail itd
LEFT JOIN ic_inventory i ON itd.item_code = i.code
WHERE itd.last_status = 0
  AND itd.item_type <> 5
  AND itd.is_doc_copy = 0
  AND i.item_type NOT IN (1, 3)
  AND itd.doc_date_calc <= CURRENT_DATE
GROUP BY itd.wh_code
HAVING SUM(ABS(itd.calc_flag * ROUND((itd.qty * itd.stand_value) / itd.divide_value, 2))) > 0
ORDER BY item_count DESC;
```

**ผลลัพธ์:** แสดงสรุปจำนวนสินค้าและปริมาณรวมในแต่ละคลัง

---

## 7. ยอดคงเหลือสินค้าตามลูกหนี้ (Stock Balance by Customer/AR)

**ตัวอย่างกรณีใช้งาน:** "ยอดคงเหลือตามลูกหนี้", "สินค้าของลูกค้า", "ขายให้ลูกค้า", "ลูกหนี้มีสินค้าอะไรบ้าง" และคำถามที่มีความหมายใกล้เคียง

**หลักการ:** ใช้ template นี้เป็นแนวทางสำหรับคำถามเกี่ยวกับยอดสินค้าที่เกี่ยวข้องกับลูกหนี้/ลูกค้า (ฝั่งขาย/AR)

**💡 สำคัญ:** Query นี้ใช้ `trans_type = 1` เพื่อกรองเฉพาะธุรกรรมฝั่งขาย/ลูกหนี้

### กรณีที่ 1: ระบุรหัสหรือชื่อลูกหนี้

**ตัวอย่างคำถาม:** "ลูกหนี้ XXX มีสินค้าอะไรบ้าง", "ยอดคงเหลือของลูกค้า ABC", "ขายให้ลูกหนี้ YYY อะไรไปบ้าง"

```sql
SELECT 
  itd.item_code,
  i.name_1 as item_name,
  ar.code as customer_code,
  ar.name_1 as customer_name,
  ROUND(SUM(itd.calc_flag * ROUND((itd.qty * itd.stand_value) / itd.divide_value, 2)), 2) as balance_qty,
  i.unit_standard_name,
  COUNT(DISTINCT itd.doc_no) as transaction_count
FROM ic_trans_detail itd
LEFT JOIN ic_inventory i ON itd.item_code = i.code
LEFT JOIN ar_customer ar ON itd.cust_code = ar.code
WHERE itd.last_status = 0
  AND itd.trans_type = 1
  AND ar.status = 0
  AND (ar.code LIKE '%CUSTOMER_CODE_OR_NAME%' OR ar.name_1 LIKE '%CUSTOMER_CODE_OR_NAME%')
GROUP BY itd.item_code, i.name_1, ar.code, ar.name_1, i.unit_standard_name
HAVING SUM(itd.calc_flag * ROUND((itd.qty * itd.stand_value) / itd.divide_value, 2)) <> 0
ORDER BY balance_qty DESC
LIMIT 30;
```

**วิธีใช้:** แทนที่ `'%CUSTOMER_CODE_OR_NAME%'` ด้วยรหัสหรือชื่อลูกหนี้

**ผลลัพธ์:** แสดงรายการสินค้า, ชื่อลูกหนี้, ยอดคงเหลือ, และจำนวนครั้งที่ทำธุรกรรม

### กรณีที่ 2: แสดงสินค้าทั้งหมดที่มีธุรกรรมกับลูกหนี้

**ตัวอย่างคำถาม:** "สินค้าไหนขายให้ลูกหนี้บ้าง", "แสดงสินค้าที่มีลูกหนี้"

```sql
SELECT 
  itd.item_code,
  i.name_1 as item_name,
  COUNT(DISTINCT ar.code) as customer_count,
  ROUND(SUM(itd.calc_flag * ROUND((itd.qty * itd.stand_value) / itd.divide_value, 2)), 2) as total_qty,
  i.unit_standard_name
FROM ic_trans_detail itd
LEFT JOIN ic_inventory i ON itd.item_code = i.code
LEFT JOIN ar_customer ar ON itd.cust_code = ar.code
WHERE itd.last_status = 0
  AND itd.trans_type = 1
  AND ar.status = 0
GROUP BY itd.item_code, i.name_1, i.unit_standard_name
HAVING SUM(itd.calc_flag * ROUND((itd.qty * itd.stand_value) / itd.divide_value, 2)) <> 0
ORDER BY customer_count DESC, total_qty DESC
LIMIT 30;
```

**ผลลัพธ์:** แสดงสินค้าที่มีธุรกรรมกับลูกหนี้ พร้อมจำนวนลูกหนี้และยอดรวม

---

## 8. ยอดคงเหลือสินค้าตามเจ้าหนี้ (Stock Balance by Supplier/AP)

**ตัวอย่างกรณีใช้งาน:** "ยอดคงเหลือตามเจ้าหนี้", "สินค้าจากซัพพลายเออร์", "ซื้อจากเจ้าหนี้", "เจ้าหนี้ขายสินค้าอะไรให้เราบ้าง" และคำถามที่มีความหมายใกล้เคียง

**หลักการ:** ใช้ template นี้เป็นแนวทางสำหรับคำถามเกี่ยวกับยอดสินค้าที่เกี่ยวข้องกับเจ้าหนี้/ซัพพลายเออร์ (ฝั่งซื้อ/AP)

**💡 สำคัญ:** Query นี้ใช้ `trans_type = 2` เพื่อกรองเฉพาะธุรกรรมฝั่งซื้อ/เจ้าหนี้

### กรณีที่ 1: ระบุรหัสหรือชื่อเจ้าหนี้

**ตัวอย่างคำถาม:** "เจ้าหนี้ XXX ขายสินค้าอะไรให้เราบ้าง", "ยอดคงเหลือจากซัพพลายเออร์ ABC", "ซื้อจากเจ้าหนี้ YYY อะไรบ้าง"

```sql
SELECT 
  itd.item_code,
  i.name_1 as item_name,
  ap.code as supplier_code,
  ap.name_1 as supplier_name,
  ROUND(SUM(itd.calc_flag * ROUND((itd.qty * itd.stand_value) / itd.divide_value, 2)), 2) as balance_qty,
  i.unit_standard_name,
  COUNT(DISTINCT itd.doc_no) as transaction_count
FROM ic_trans_detail itd
LEFT JOIN ic_inventory i ON itd.item_code = i.code
LEFT JOIN ap_supplier ap ON itd.cust_code = ap.code
WHERE itd.last_status = 0
  AND itd.trans_type = 2
  AND ap.status = 0
  AND (ap.code LIKE '%SUPPLIER_CODE_OR_NAME%' OR ap.name_1 LIKE '%SUPPLIER_CODE_OR_NAME%')
GROUP BY itd.item_code, i.name_1, ap.code, ap.name_1, i.unit_standard_name
HAVING SUM(itd.calc_flag * ROUND((itd.qty * itd.stand_value) / itd.divide_value, 2)) <> 0
ORDER BY balance_qty DESC
LIMIT 30;
```

**วิธีใช้:** แทนที่ `'%SUPPLIER_CODE_OR_NAME%'` ด้วยรหัสหรือชื่อเจ้าหนี้

**ผลลัพธ์:** แสดงรายการสินค้า, ชื่อเจ้าหนี้, ยอดคงเหลือ, และจำนวนครั้งที่ทำธุรกรรม

### กรณีที่ 2: แสดงสินค้าทั้งหมดที่มีธุรกรรมกับเจ้าหนี้

**ตัวอย่างคำถาม:** "สินค้าไหนซื้อจากเจ้าหนี้บ้าง", "แสดงสินค้าที่มีเจ้าหนี้"

```sql
SELECT 
  itd.item_code,
  i.name_1 as item_name,
  COUNT(DISTINCT ap.code) as supplier_count,
  ROUND(SUM(itd.calc_flag * ROUND((itd.qty * itd.stand_value) / itd.divide_value, 2)), 2) as total_qty,
  i.unit_standard_name
FROM ic_trans_detail itd
LEFT JOIN ic_inventory i ON itd.item_code = i.code
LEFT JOIN ap_supplier ap ON itd.cust_code = ap.code
WHERE itd.last_status = 0
  AND itd.trans_type = 2
  AND ap.status = 0
GROUP BY itd.item_code, i.name_1, i.unit_standard_name
HAVING SUM(itd.calc_flag * ROUND((itd.qty * itd.stand_value) / itd.divide_value, 2)) <> 0
ORDER BY supplier_count DESC, total_qty DESC
LIMIT 30;
```

**ผลลัพธ์:** แสดงสินค้าที่มีธุรกรรมกับเจ้าหนี้ พร้อมจำนวนเจ้าหนี้และยอดรวม

---

## 📌 หมายเหตุสำคัญสำหรับ Query #7 และ #8

**การวิเคราะห์บริบท:**
- AI ต้อง**วิเคราะห์บริบทคำถาม**เพื่อเลือกใช้ Query ที่เหมาะสม
- **ห้าม hardcode keywords** - ใช้ความเข้าใจบริบทในการตัดสินใจ

**แนวทางการเลือกใช้:**
- คำถามเกี่ยวกับ **ลูกหนี้/ลูกค้า/ขาย/AR** → ใช้ Query #7 (trans_type=1)
- คำถามเกี่ยวกับ **เจ้าหนี้/ซัพพลายเออร์/ซื้อ/AP** → ใช้ Query #8 (trans_type=2)

**การปรับแต่ง:**
- สามารถเพิ่ม WHERE clause ตามความต้องการ (เช่น กรองตามคลัง, วันที่)
- สามารถเพิ่ม JOIN กับตารางอื่นเพื่อแสดงข้อมูลเพิ่มเติม
- สามารถปรับ GROUP BY และ ORDER BY ตามบริบท

---

**Version:** 1.2  
**Last Updated:** December 29, 2025
