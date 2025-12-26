# SML Chatbot - Template Repository (v2.0.0)

Template repository สำหรับ SML Chatbot ระบบ AI Assistant สำหรับ ERP

## 🎉 Version 2.0.0 - Markdown-Based Templates

เวอร์ชันนี้เปลี่ยนจาก JSON เป็น **Markdown format** เพื่อ:
- ✅ แก้ไขง่ายกว่า (ไม่ต้องเขียน JSON)
- ✅ AI-friendly (AI อ่านและเข้าใจได้ดีกว่า)
- ✅ Modular (แยกไฟล์ชัดเจน)
- ✅ Git-friendly (diff ชัดเจน)

---

## 📋 Templates ที่มี

- **stock** - แผนกคลังสินค้า (Inventory) - ✅ Full Markdown
- **ap** - แผนกเจ้าหนี้ (Accounts Payable) - 🔧 Basic structure
- **ar** - แผนกลูกหนี้ (Accounts Receivable) - 🔧 Basic structure

---

## 📁 โครงสร้าง v2.0

```
template-repo/
├── README.md                   # เอกสารนี้
├── TEMPLATE_GUIDE.md           # คู่มือการสร้าง template (Legacy)
├── SPECIAL_QUERIES_GUIDE.md    # การจัดการ special queries (Legacy)
├── DEPLOYMENT.md               # คู่มือการ deploy ไป GitHub
├── .gitignore                 # Git ignore rules
├── version.json               # Version metadata (format: "markdown")
├── templates.json             # 🆕 Template catalog with AI-friendly keywords
└── templates/
    ├── stock/
    │   ├── system_prompt.md       # บทบาทและหน้าที่ของ AI
    │   ├── schema.md              # Database schema + FAQ queries
    │   └── special_queries.md     # Complex business logic queries
    ├── ar/
    │   ├── system_prompt.md
    │   ├── schema.md
    │   └── special_queries.md
    └── ap/
        ├── system_prompt.md
        ├── schema.md
        └── special_queries.md
```

---

## 🚀 Quick Start - Deploy to GitHub

**Repository URL:** https://github.com/smlsoft/sml-chatbot-templates

### ขั้นตอนสั้น:

1. **Clone repository:**
   ```bash
   git clone https://github.com/smlsoft/sml-chatbot-templates.git
   cd sml-chatbot-templates
   ```

2. **อัพเดต .env ใน chatbot:**
   ```env
   TEMPLATE_REPO_URL=https://raw.githubusercontent.com/smlsoft/sml-chatbot-templates/main
   ```

3. **Sync templates:**
   ```bash
   POST http://localhost:3001/api/templates/sync
   ```

---

## 📝 Template Format (v2.0)

แต่ละ template ประกอบด้วย 3 ไฟล์:

### 1. system_prompt.md
กำหนดบทบาทและหน้าที่ของ AI

```markdown
# System Prompt - แผนกคลังสินค้า

คุณเป็น **ผู้ช่วยเลขานุการมืออาชีพ** ของแผนกคลังสินค้า

## บทบาทและหน้าที่
- ตอบคำถามเกี่ยวกับข้อมูลสินค้าคงคลัง
- ค้นหาและแสดงข้อมูลสินค้า
...
```

### 2. schema.md
กำหนด Database schema และตัวอย่าง SQL

```markdown
# Database Schema - แผนกคลังสินค้า

## โครงสร้างตาราง
...

## ตัวอย่าง SQL Queries
...
```

### 3. special_queries.md
กำหนด Business logic queries ที่ซับซ้อน

```markdown
# Special Queries - แผนกคลังสินค้า

## 1. ยอดค้างจอง (Book Out Quantity)

**Keywords:** ค้างจอง, book out, จองสินค้า

### Query Template:
```sql
...
```
```

---

## ✏️ การเพิ่ม Template ใหม่

### ขั้นตอน:

1. **สร้างโฟลเดอร์ใหม่:**
   ```bash
   mkdir templates/your_template
   ```

2. **สร้าง 3 ไฟล์:**
   ```bash
   touch templates/your_template/system_prompt.md
   touch templates/your_template/schema.md
   touch templates/your_template/special_queries.md
   ```

3. **แก้ไข `templates.json`:**
   ```json
   {
     "templates": [
       {
         "templateId": "your_template",
         "templateName": "Your Template Name",
         "department": "your_department",
         "description": "...",
         "keywords": ["keyword1", "keyword2"],
         "relatedTables": ["table1", "table2"],
         "enabled": true,
         "files": {
           "systemPrompt": "templates/your_template/system_prompt.md",
           "schema": "templates/your_template/schema.md",
           "specialQueries": "templates/your_template/special_queries.md"
         }
       }
     ]
   }
   ```

4. **Commit และ Push:**
   ```bash
   git add .
   git commit -m "Add new template: your_template"
   git push
   ```

5. **Sync ใน chatbot:**
   ```bash
   POST http://localhost:3001/api/templates/sync
   ```

---

## 🔧 Local Development

### Start Local Template Server:

```bash
cd template-repo
python3 -m http.server 8888
```

### Set .env (Development):

```env
TEMPLATE_REPO_URL=http://localhost:8888
```

### Test Template Loading:

```bash
curl http://localhost:8888/version.json
curl http://localhost:8888/templates.json
curl http://localhost:8888/templates/stock/system_prompt.md
```

---

## 🎯 Custom System Prompt

แต่ละ Channel สามารถมี `customSystemPrompt` เพื่อปรับแต่งพฤติกรรม AI:

### ตัวอย่าง:

**Channel Config:**
```json
{
  "channelName": "Inventory - DEDE Branch",
  "templateId": "stock",
  "customSystemPrompt": "แสดงเฉพาะคลังรหัส \"DEDE\""
}
```

**ผลลัพธ์:** AI จะเพิ่ม `AND wh_code = 'DEDE'` ในทุก SQL query

---

## 📚 เอกสาร

- **TEMPLATE_GUIDE.md** - คู่มือการสร้าง template (Legacy JSON format)
- **SPECIAL_QUERIES_GUIDE.md** - การจัดการ queries ที่ซับซ้อน
- **DEPLOYMENT.md** - คู่มือการ deploy ไป GitHub

---

## 🔄 Migration from v1.x

ระบบรองรับทั้ง **JSON v1.x** และ **Markdown v2.0**:

- ✅ Auto-detect format จาก `version.json`
- ✅ Backwards compatible
- ✅ ไม่กระทบระบบเดิม

---

## 📖 Version History

### v2.0.0 (December 26, 2025)
- 🎉 Migrated to Markdown format
- ✅ Split templates into 3 files (system_prompt, schema, special_queries)
- ✅ Added AI-friendly keywords
- ✅ Improved custom system prompt integration

### v1.0.3 (December 25, 2025)
- Legacy JSON-based template system

---

**Version:** 2.0.0
**Format:** Markdown
**Last Updated:** December 26, 2025
**Maintained by:** SML Development Team
**Repository:** https://github.com/smlsoft/sml-chatbot-templates
