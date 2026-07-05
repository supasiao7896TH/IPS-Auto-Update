# Agents.md — คำแนะนำสำหรับ AI agent

คำแนะนำนี้สำหรับ AI agent (Claude Code หรือเครื่องมืออื่น) ที่จะพัฒนาต่อบน repo นี้
ดูภาพรวมโปรเจกต์และประวัติที่ `context.md`

## โครงสร้าง repo

- ไม่มี build step, ไม่มี test suite, ไม่มี `package.json`, ไม่มี CI
- โค้ดทั้งหมดอยู่ในไฟล์เดียว: **`Kaizen_Tracker07_1.html`** (single-file web app)
- เปิดไฟล์นี้ตรงในเบราว์เซอร์เพื่อรัน ไม่ต้องติดตั้งอะไรเพิ่ม

## โครงสร้างโค้ดใน `Kaizen_Tracker07_1.html`

โค้ด JS ทั้งหมดอยู่ใน IIFE เดียว รวมศูนย์ที่ตัวแปร `App`:

```
App.config    — ค่าคงที่ (ชื่อ localStorage key, ชื่อเดือน)
App.state     — state ปัจจุบัน (sections, employees, activities, filter ต่างๆ)
App.dom       — cache reference ของ DOM element
App.methods.ui     — render, modal, notification
App.methods.data   — โหลด/บันทึกข้อมูล, สร้างรายงาน/อีเมล/PDF/banner, OCR
App.methods.events — event binding และ handler
```

## Data model

```js
sections   = { id, name }
employees  = { id, firstName, lastName, sectionId, annualTarget }
activities = { employeeId, year, month, count }
```

localStorage keys: `kaizen_v2_sections`, `kaizen_v2_employees`, `kaizen_v2_activities`,
`kaizen_global_target`, `kaizen_gemini_key`, `kaizen_report_author`

**สำคัญ**: เป้าหมายที่ใช้คำนวณจริงคือ `App.state.globalTarget` (ค่าเดียว กรอกจากหน้าเว็บ ปกติ 12) —
ฟิลด์ `annualTarget` ต่อพนักงานมีอยู่ในข้อมูลแต่ **ไม่ถูกใช้คำนวณ** อย่าเผลอไปอ้างอิงฟิลด์นี้
เว้นแต่ผู้ใช้ขอให้เปลี่ยน logic

## Conventions ที่ต้องรักษา

1. **XSS**: ข้อมูลที่มาจากผู้ใช้ (ชื่อพนักงาน, ชื่อแผนก, ข้อความ input ฯลฯ) ต้องผ่าน `escHtml()` เสมอ
   ก่อนแทรกลงใน template literal ที่จะกลายเป็น HTML — ดูรูปแบบได้จากเกือบทุกฟังก์ชันใน `methods.ui`/`methods.data`
2. **Helper ที่ใช้ร่วมกัน ห้ามเขียนซ้ำ**:
   - `App.methods.data.buildStats(year, month)` — คำนวณสถิติต่อพนักงาน (ใช้ใน email summary, email HTML, PDF report, email banner)
   - `App.methods.data.makePodiumSvg(stats)` — สร้าง podium SVG (ใช้ใน PDF report และ email banner)
   ถ้าต้องการฟีเจอร์ใหม่ที่ใช้สถิติพนักงานหรือ podium ให้เรียกใช้ helper เหล่านี้แทนการคำนวณเอง
3. **Modal system**: ใช้ `App.methods.ui.showModal({title, body, actions}, trigger)` และ
   `App.methods.ui.closeModal(trigger)` — มี focus trap และ ARIA attributes ในตัวอยู่แล้ว
   ถ้า modal ผูก event listener ระดับ `document` (เช่น paste event) ต้อง cleanup ผ่าน `el._onClose`
   callback เพื่อไม่ให้ listener ค้างเมื่อปิด modal ด้วยปุ่ม Escape
4. **CDN dependencies pin เวอร์ชันแล้ว**: Tailwind `3.4.16`, Chart.js `4.4.7`, Font Awesome `6.5.1`
   — อย่าเปลี่ยนกลับไปใช้ URL แบบไม่ระบุเวอร์ชัน (`@latest` หรือไม่มี version เลย)
5. **UI ภาษาไทยทั้งหมด** — ข้อความใหม่ที่เพิ่มต้องเป็นภาษาไทย ให้โทนเดียวกับข้อความที่มีอยู่
6. **Chart.js เป็น optional**: `renderDashboardChart()` เช็ค `typeof Chart === 'undefined'` แล้ว skip
   กราฟถ้าโหลด CDN ไม่สำเร็จ เพื่อไม่ให้ทั้งแอปพังจาก dependency เดียว — รักษา pattern นี้ไว้ถ้าเพิ่ม CDN ใหม่

## วิธีทดสอบการเปลี่ยนแปลง

ไม่มี automated test — ต้องทดสอบด้วยมือทุกครั้ง:

1. เปิดไฟล์ตรงในเบราว์เซอร์ (`file://.../Kaizen_Tracker07_1.html`) หรือใช้ Chromium ที่ path
   `/opt/pw-browsers/chromium` ผ่าน Playwright สำหรับทดสอบแบบ headless
2. เช็ค browser console ว่าไม่มี error (โดยเฉพาะหลังแก้ JS ในไฟล์)
3. ทดสอบ flow หลักที่เกี่ยวข้องกับจุดที่แก้ เช่น: บันทึกกิจกรรม → ตารางอัพเดท, import JSON ที่มี record ซ้ำ →
   ยอดรวมต้องตรงกับผลรวมรายเดือน, export PDF/email banner → เปิด preview ใน iframe แล้วตรวจ podium/สี/เลข
4. ถ้าแก้ syntax ของ `<script>` block ให้ตรวจ syntax ก่อนด้วย `node --check` (ดึงเนื้อหาใน `<script>` ออกมาก่อน)
   เพราะไฟล์เป็น HTML ทั้งไฟล์ ไม่ใช่ `.js` โดยตรง

## Git

- Branch หลัก: `main`
- แนวทางที่ใช้อยู่: สร้าง branch ชื่อ `claude/<หัวข้องาน>`, commit, แล้วเปิด PR เข้า `main`
  (เว้นแต่เป็นงานเอกสารความเสี่ยงต่ำที่ผู้ใช้ขอให้ push ตรงเข้า `main`)
