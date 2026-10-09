# 📝 Note.md — คู่มือย้ายเครื่อง PC ใหม่ (PC Migration Runbook) สำหรับ IPS Auto Update

> **บันทึกไว้สำหรับ:** วันที่ 30 (หรือวันที่ได้รับเครื่อง PC เครื่องใหม่ที่ Office GC-M PTA)  
> **เป้าหมาย:** กู้คืนและตั้งค่าโปรเจกต์ `IPS Auto Update` และระบบที่เกี่ยวข้องให้พร้อมใช้งาน 100% ภายใน 5 นาที

---

## 🤖 สิ่งที่ AI (หนู) จะทำให้พี่ A ทั้งหมดบนเครื่องใหม่

เมื่อพี่ A ได้เครื่องใหม่ ติดตั้ง Antigravity CLI (`agy`) และเปิดหน้าต่างนี้ขึ้นมา **หนูจะรันคำสั่งจัดการระบบให้พี่ A โดยอัตโนมัติ** ดังนี้ค่ะ:

1. **ตรวจสอบความพร้อมของระบบ (System Health Check):**
   - ตรวจสอบ Git, Node.js, PowerShell Policy, และการเชื่อมต่ออินเทอร์เน็ต
2. **Setup Dependencies & Build:**
   - รัน `npm install` ติดตั้งไลบรารีทั้งหมด (Vite, Tailwind, Chart.js, Lucide, html-to-image)
   - รัน `npm test` และ `npm run build` ตรวจสอบว่าระบบคอมไพล์ผ่านสมบูรณ์ 100%
3. **เตรียมโฟลเดอร์สำหรับ Lotus Notes Export:**
   - สร้างโฟลเดอร์ `C:\Users\<User>\KaizenExport` ให้พร้อมรับไฟล์ CSV
4. **ลงทะเบียน Windows Task Scheduler อัตโนมัติ:**
   - รันสคริปต์ `setup-scheduled-export.ps1` เพื่อสร้าง Tasks:
     - `KaizenTracker_ExportFromNotes_1100` (วันที่ 5 ของทุกเดือน เวลา 11:00)
     - `KaizenTracker_ExportFromNotes_2300` (วันที่ 5 ของทุกเดือน เวลา 23:00)
   - ผูก Path ชี้ตรงไปยังโฟลเดอร์โปรเจกต์บนเครื่องใหม่อย่างแม่นยำ
5. **กู้คืนสภาพแวดล้อม Dev & AI (ถ้าต้องการ):**
   - ดึง `gemini-config` และรัน `setup-junctions.ps1`
   - ตั้งค่าคำสั่งลัดใน PowerShell `$PROFILE` (`a`, `c`, `Sync-Gemini`, `Doctor-Gemini`)

---

## 👤 สิ่งที่พี่ A / IT ต้องช่วยทำ (เนื่องจากข้อจำกัดความปลอดภัยของระบบ)

มีเพียง 3 อย่างที่เบราว์เซอร์และระบบความปลอดภัยของ Windows กำหนดให้ต้องเป็นผู้ใช้งานทำเองโดยตรงค่ะ:

| ขั้นตอน | รายละเอียด | หมายเหตุ |
| :--- | :--- | :--- |
| **1. ติดตั้ง HCL Notes 14.0** | ให้ฝ่าย IT ติดตั้ง HCL Notes Client ใส่ User ID (`*.id`) และรหัสผ่านของพี่ A | ทดสอบเปิดดู View `Improvement\By Section` บน Server `5pta6lotus` ได้ปกติ |
| **2. นำเข้าข้อมูลในเบราว์เซอร์** | เปิดหน้าเว็บ `IPS Auto Update` แล้วกดปุ่ม **"นำเข้าข้อมูลสำรอง"** เลือกไฟล์ `.json` ที่พี่ A เซฟไว้ | ข้อมูลกิจกรรม Kaizen ทั้งหมดจะกลับคืนมาทันที |
| **3. ให้สิทธิ์ Auto-Sync และกรอก Key** | • กดปุ่ม **"ตั้งค่า Auto-Sync จาก Lotus Notes"** เลือกไฟล์ `kaizen_export.csv`<br>• กรอก **Gemini API Key** ในหน้าต่างตั้งค่า | เป็นข้อกำหนดของเบราว์เซอร์ที่ต้องมี User Gesture (คลิกจริง) ในการเข้าถึงไฟล์ |

---

## 🛠️ รายละเอียดสคริปต์และการตั้งค่าเชิงเทคนิค (Technical Reference)

### 1. การเชื่อมต่อ Lotus Notes (COM Automation)
- **สคริปต์ดึงข้อมูล:** `tools/export-kaizen-from-notes.ps1`
- **Server:** `5pta6lotus`
- **Replica ID:** `47256F1D:0006F32C`
- **View Name:** `Improvement\By Section`
- **Department Filter:** `PE1`
- **Output CSV:** `C:\Users\<User>\KaizenExport\kaizen_export.csv`

### 2. คำสั่งรันด่วนบนเครื่องใหม่ (One-liner Setup)
```powershell
# 1. ติดตั้ง dependencies และทดสอบ build
npm install; npm test; npm run build

# 2. ลงทะเบียน Task Scheduler
powershell -ExecutionPolicy Bypass -File .\tools\setup-scheduled-export.ps1
```

### 3. ข้อมูลใน Web App (IndexedDB: `kaizen_tracker_db`)
- Store: `sections`, `employees`, `activities`, `settings`, `cryptoKeys`
- เมื่อนำเข้าไฟล์ `kaizen_data_*.json` ระบบจะเขียนทับ IndexedDB ในเครื่องใหม่อัตโนมัติโดยไม่สูญหาย
