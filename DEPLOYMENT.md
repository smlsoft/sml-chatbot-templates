# Template Repository - Deployment Guide

คู่มือการ deploy template repository ไปยัง GitHub

---

## 📋 Pre-requisites

- Git installed
- GitHub account
- Access to SML Chatbot server

---

## 🚀 ขั้นตอนการ Deploy

### Step 1: สร้าง GitHub Repository

1. **ไปที่ GitHub** → New Repository
2. **ตั้งค่า:**
   - Repository name: `sml-chatbot-templates` (หรือชื่อที่คุณต้องการ)
   - Description: "Template repository for SML Chatbot AI Assistant"
   - Visibility: 
     - **Private** (แนะนำ) - สำหรับข้อมูลภายในบริษัท
     - **Public** - ถ้าต้องการแชร์
   - ไม่ต้องเลือก "Initialize with README" (เพราะเรามีอยู่แล้ว)
3. **Create repository**

---

### Step 2: Initialize Git และ Push

```bash
# ไปที่ template-repo directory
cd /Users/nontawatwongnuk/dev/sml_chatbot/template-repo

# Initialize Git (ถ้ายังไม่ได้ทำ)
git init

# Add remote (แทนที่ YOUR_USERNAME ด้วย GitHub username ของคุณ)
git remote add origin https://github.com/YOUR_USERNAME/sml-chatbot-templates.git

# Add all files
git add .

# Commit
git commit -m "Initial commit: Stock, AP, AR templates with guides"

# Push to GitHub
git branch -M main
git push -u origin main
```

---

### Step 3: Verify Deployment

1. **ไปที่ GitHub repository ของคุณ**
2. **ตรวจสอบไฟล์:**
   - ✅ README.md
   - ✅ version.json
   - ✅ templates/stock.json
   - ✅ templates/ap.json
   - ✅ templates/ar.json
   - ✅ TEMPLATE_GUIDE.md
   - ✅ SPECIAL_QUERIES_GUIDE.md

3. **ทดสอบ Raw URL:**

คลิก `version.json` → คลิก **Raw** button → Copy URL

URL จะเป็น:
```
https://raw.githubusercontent.com/YOUR_USERNAME/sml-chatbot-templates/main/version.json
```

---

### Step 4: Update SML Chatbot Configuration

1. **แก้ไข `.env`:**

```bash
# ไปที่ chatbot directory
cd /Users/nontawatwongnuk/dev/sml_chatbot

# แก้ไข .env
nano .env  # หรือ code .env
```

2. **อัพเดต TEMPLATE_REPO_URL:**

```env
# แทนที่ YOUR_USERNAME ด้วย GitHub username ของคุณ
TEMPLATE_REPO_URL=https://raw.githubusercontent.com/YOUR_USERNAME/sml-chatbot-templates/main
```

3. **Save และ Restart server:**

```bash
# หยุด server ปัจจุบัน (Ctrl+C หรือ)
pkill -f "tsx watch src/server.ts"

# Start ใหม่
npm run dev
```

---

### Step 5: Verify Template Sync

ตรวจสอบ logs ว่า templates sync สำเร็จ:

```
🔄 Syncing templates from repository...
Repository URL: https://raw.githubusercontent.com/YOUR_USERNAME/sml-chatbot-templates/main
Repository version: 1.0.0
Fetching template: stock
Fetching template: ap
Fetching template: ar
Synced template: stock
Synced template: ap
Synced template: ar
✅ Template sync completed: 3 templates synced
```

---

## 🔄 Workflow: การอัพเดต Templates

### Development → Production Flow

#### 1. Development (Local)

```bash
# ใช้ local server
cd template-repo
python3 -m http.server 8888
```

**.env:**
```env
TEMPLATE_REPO_URL=http://localhost:8888
```

#### 2. Testing

- แก้ไข templates
- Restart chatbot server
- ทดสอบในแชท
- Verify ผลลัพธ์

#### 3. Staging (GitHub Branch)

```bash
# สร้าง branch ใหม่
git checkout -b feature/update-stock-template

# Commit changes
git add templates/stock.json
git commit -m "Update stock template: add special queries"

# Push to GitHub
git push origin feature/update-stock-template
```

**Test staging:**
```env
TEMPLATE_REPO_URL=https://raw.githubusercontent.com/YOUR_USERNAME/sml-chatbot-templates/feature/update-stock-template
```

#### 4. Production (Main Branch)

```bash
# Merge to main
git checkout main
git merge feature/update-stock-template

# Update version
# แก้ไข version.json: "1.0.0" → "1.0.1"

# Commit version bump
git add version.json
git commit -m "Bump version to 1.0.1"

# Push to production
git push origin main
```

**Production:**
```env
TEMPLATE_REPO_URL=https://raw.githubusercontent.com/YOUR_USERNAME/sml-chatbot-templates/main
```

---

## 🔐 Security Considerations

### Private Repository (แนะนำ)

ถ้าใช้ **Private repository**:

1. **สร้าง Personal Access Token (PAT):**
   - GitHub → Settings → Developer settings → Personal access tokens
   - Generate new token (classic)
   - Select scopes: `repo` (full control)
   - Copy token

2. **อัพเดต URL ใน .env:**

```env
TEMPLATE_REPO_URL=https://YOUR_GITHUB_TOKEN@raw.githubusercontent.com/YOUR_USERNAME/sml-chatbot-templates/main
```

⚠️ **หรือดีกว่า:**

```env
TEMPLATE_REPO_URL=https://raw.githubusercontent.com/YOUR_USERNAME/sml-chatbot-templates/main
GITHUB_TOKEN=ghp_your_personal_access_token_here
```

แล้วแก้ code ให้ใช้ token ใน headers

### Public Repository

ถ้าใช้ **Public repository**:
- ✅ ไม่ต้องใช้ token
- ⚠️ ระวังอย่า commit ข้อมูลลับ (credentials, passwords)
- ✅ เหมาะสำหรับ templates ที่ไม่มี sensitive data

---

## 🆘 Troubleshooting

### ปัญหา: Template sync failed

**Symptoms:**
```
❌ Failed to sync templates: Error fetching version.json
```

**Solutions:**

1. **ตรวจสอบ URL:**
   ```bash
   # ทดสอบ URL ด้วย curl
   curl https://raw.githubusercontent.com/YOUR_USERNAME/sml-chatbot-templates/main/version.json
   ```

2. **ตรวจสอบ network:**
   ```bash
   ping raw.githubusercontent.com
   ```

3. **ตรวจสอบ GitHub status:**
   - ไปที่ https://www.githubstatus.com/

### ปัญหา: 404 Not Found

**Solutions:**

1. ตรวจสอบ repository เป็น **Public** หรือใช้ token ถูกต้อง
2. ตรวจสอบ branch name (`main` vs `master`)
3. ตรวจสอบไฟล์มีอยู่จริง

### ปัญหา: Old templates cached

**Solutions:**

```bash
# Clear MongoDB cache (ถ้ามี)
npm run seed  # Re-seed database

# หรือ restart server
pkill -f "tsx watch src/server.ts"
npm run dev
```

---

## 📚 Additional Resources

- [GitHub - Creating a Repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository)
- [GitHub - Personal Access Tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
- [Git Basics](https://git-scm.com/book/en/v2/Getting-Started-Git-Basics)

---

**Next Steps After Deployment:**

1. ✅ Test template sync from GitHub
2. ✅ Update documentation with actual GitHub URL
3. ✅ Share repository with team members
4. ✅ Set up branch protection rules (optional)
5. ✅ Configure webhooks for auto-sync (future enhancement)

---

**Version:** 1.0.0  
**Last Updated:** December 25, 2025
