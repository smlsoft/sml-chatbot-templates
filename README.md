# SML Chatbot - Template Repository

Template repository สำหรับ SML Chatbot ระบบ AI Assistant สำหรับ ERP

## 📋 เกี่ยวกับ Repository นี้

Repository นี้เก็บ templates ที่กำหนดพฤติกรรมของ AI สำหรับแต่ละแผนก/Channel ใน SML Chatbot

### Templates ที่มี:
- **stock** - แผนกคลังสินค้า (Inventory)
- **ap** - แผนกเจ้าหนี้ (Accounts Payable)
- **ar** - แผนกลูกหนี้ (Accounts Receivable)

---

## 🚀 Quick Start - Deploy to GitHub

**ดูคู่มือฉบับเต็ม:** [DEPLOYMENT.md](./DEPLOYMENT.md)

### ขั้นตอนสั้น:

1. **สร้าง GitHub repository ใหม่**
2. **Push code ขึ้น GitHub:**
   ```bash
   cd /Users/nontawatwongnuk/dev/sml_chatbot/template-repo
   git init
   git add .
   git commit -m "Initial commit: Stock, AP, AR templates"
   git remote add origin https://github.com/YOUR_USERNAME/sml-chatbot-templates.git
   git branch -M main
   git push -u origin main
   ```

3. **อัพเดต .env ใน chatbot:**
   ```env
   TEMPLATE_REPO_URL=https://raw.githubusercontent.com/YOUR_USERNAME/sml-chatbot-templates/main
   ```

4. **Restart chatbot server**

---

## 📁 โครงสร้าง

```
template-repo/
├── README.md              # เอกสารนี้
├── TEMPLATE_GUIDE.md      # คู่มือการสร้าง template
├── SPECIAL_QUERIES_GUIDE.md  # การจัดการ special queries
├── DEPLOYMENT.md          # คู่มือการ deploy ไป GitHub
├── .gitignore            # Git ignore rules
├── version.json          # รายการ templates ทั้งหมด
└── templates/
    ├── stock.json        # Stock/Inventory template
    ├── ap.json          # Accounts Payable template
    └── ar.json          # Accounts Receivable template
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

---

## 📝 Template Format

แต่ละ template เป็นไฟล์ JSON ที่มีโครงสร้าง:

```json
{
  "templateId": "stock",
  "templateName": "Stock/Inventory Template",
  "department": "inventory",
  "version": "1.0.0",
  "systemPrompt": "คำสั่งให้ AI...",
  "relatedTables": ["table1", "table2"],
  "customQueries": [...],
  "sampleQuestions": [...],
  "settings": {
    "temperature": 0.3,
    "maxTokens": 2048,
    "enableCardFormat": true
  }
}
```

**ดูรายละเอียดเพิ่มเติม:** [TEMPLATE_GUIDE.md](./TEMPLATE_GUIDE.md)

---

## ✏️ การเพิ่ม Template ใหม่

1. สร้างไฟล์ `templates/your_template.json`
2. แก้ไข `version.json` เพิ่ม template ใหม่
3. Commit และ Push
4. Restart chatbot server

---

## �� เอกสาร

- [TEMPLATE_GUIDE.md](./TEMPLATE_GUIDE.md) - คู่มือการสร้าง template
- [SPECIAL_QUERIES_GUIDE.md](./SPECIAL_QUERIES_GUIDE.md) - การจัดการ queries ที่ซับซ้อน
- [DEPLOYMENT.md](./DEPLOYMENT.md) - คู่มือการ deploy ไป GitHub

---

**Version:** 1.0.0  
**Last Updated:** December 25, 2025  
**Maintained by:** SML Development Team
