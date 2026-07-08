# med-logger (เว็บบันทึกยาจากรูป → Google Sheet)

ถ่ายรูปกล่อง/ฉลากยา → อ่านข้อมูลด้วย AI → บันทึกลง Google Sheet
เว็บ (หน้า `index.html`) โฮสต์ฟรีบน **GitHub Pages** ส่วนการอ่านรูปและเขียนชีตทำผ่าน **Google Apps Script** (เก็บ API key ไว้ฝั่งเซิร์ฟเวอร์ ปลอดภัย)

```
ผู้ใช้ (มือถือ/คอม)
   │  เปิดหน้าเว็บบน GitHub Pages
   ▼
index.html ── ส่งรูป(base64)/record ──►  Apps Script (/exec)
                                          │  • เรียก Anthropic API อ่านฉลาก
                                          │  • เขียนแถวลง Google Sheet
   ◄──────── JSON ผลลัพธ์ ───────────────┘
```

---

## ส่วนที่ 1 — ตั้งค่า Google Apps Script (แบ็กเอนด์)

1. เปิด Google Sheet ที่จะใช้เก็บข้อมูล → เมนู **ส่วนขยาย (Extensions)** → **Apps Script**
2. ลบโค้ดเดิมทั้งหมด แล้ววางโค้ดจาก `apps-script/med-logger.gs` → กดบันทึก (💾)
   - แก้บรรทัด `var DEFAULT_SHEET = "ยา";` ให้เป็นชื่อแท็บที่ต้องการ (ถ้ายังไม่มีแท็บ ระบบจะสร้างให้)
3. ตั้งค่า **Script properties** (เก็บความลับ): ไอคอนเฟือง **Project Settings** → เลื่อนลงหา *Script properties* → **Add**
   - `ANTHROPIC_API_KEY` = คีย์ของคุณ (จำเป็น) — ขอที่ https://console.anthropic.com
   - `TOKEN` = สตริงลับใด ๆ (ไม่บังคับ — ใส่เพื่อกันคนอื่นเรียก endpoint; ถ้าใส่ ต้องกรอกให้ตรงในเว็บ)
4. **Deploy** → **New deployment** → เลือกชนิด **Web app**
   - *Execute as*: **Me**
   - *Who has access*: **Anyone**
   - กด Deploy → อนุญาตสิทธิ์ (Authorize) → คัดลอก **Web app URL** (ลงท้าย `/exec`)

> ทุกครั้งที่แก้โค้ด `.gs` ต้อง **Deploy → Manage deployments → แก้ไข → Version: New** เพื่อให้เวอร์ชันใหม่มีผล

---

## ส่วนที่ 2 — วาง URL ลงในเว็บ

เปิดไฟล์ `index.html` หาบรรทัด:
```js
const EMBED_URL = "https://script.google.com/macros/s/..../exec";
```
วาง URL `/exec` ของคุณแทน (ตอนนี้ผมใส่ค่าที่คุณให้ไว้แล้ว หากเปลี่ยน deployment ต้องอัปเดตตรงนี้)
*(หรือจะเว้นไว้ แล้วไปกรอกในหน้าเว็บช่อง “Apps Script Web App URL” ก็ได้)*

---

## ส่วนที่ 3 — โฮสต์บน GitHub Pages

**แบบเว็บ (ง่ายสุด ไม่ต้องใช้ command line):**
1. สร้าง repo ใหม่ที่ https://github.com/new (เช่นชื่อ `med-logger`) → Create
2. หน้า repo → **Add file → Upload files** → ลากไฟล์ทั้งหมดในโฟลเดอร์นี้ (`index.html`, `.nojekyll`, `README.md`, โฟลเดอร์ `apps-script/`) → **Commit changes**
3. **Settings** → เมนูซ้าย **Pages** → หัวข้อ *Build and deployment* → Source: **Deploy from a branch** → Branch: **main** / **/ (root)** → **Save**
4. รอสักครู่ แล้วเปิดลิงก์ที่ปรากฏ: `https://<username>.github.io/med-logger/`

**แบบ command line (ถ้าถนัด git):**
```bash
git init
git add .
git commit -m "med-logger web"
git branch -M main
git remote add origin https://github.com/<username>/med-logger.git
git push -u origin main
# แล้วไปเปิด GitHub Pages ที่ Settings → Pages เหมือนด้านบน
```

---

## ส่วนที่ 4 — ทดสอบ

1. เปิดหน้าเว็บ (ลิงก์ GitHub Pages) — สถานะการเชื่อมต่อควรเป็น **“พร้อม”**
   - ถ้าตั้ง `TOKEN` ไว้ ให้กรอก Token ในหน้าเว็บให้ตรง แล้วกดบันทึกการตั้งค่า
2. กด **ทดสอบเชื่อมต่อ** → ควรขึ้นว่าเพิ่มแถว `__TEST__` (ไปลบออกในชีตได้)
3. ถ่าย/เลือกรูปยา → ตรวจแก้ข้อมูล → **บันทึกลง Google Sheet** → แถวขึ้นในชีตทันที

---

## แก้ปัญหาที่พบบ่อย
- **กด “ทดสอบเชื่อมต่อ” แล้ว error / อ่านผลไม่ได้** → มักเป็นเพราะ *Who has access* ไม่ได้ตั้งเป็น **Anyone** หรือยังไม่ได้ Deploy เวอร์ชันใหม่หลังแก้โค้ด
- **แจ้ง “ยังไม่ได้ตั้งค่า ANTHROPIC_API_KEY”** → เพิ่มใน Script properties ให้ครบ แล้ว Deploy ใหม่
- **หน้าเว็บเปิดแต่ไม่เชื่อมต่อ** → ตรวจว่า `EMBED_URL` ลงท้าย `/exec` และตรงกับ deployment ล่าสุด
- **รูป iPhone (HEIC) เปิดไม่ได้** → เว็บย่อ/ถอดรหัสให้อัตโนมัติ ถ้ายังพลาด ตั้งกล้อง iPhone เป็น *ประสิทธิภาพสูงสุด* (ถ่ายเป็น JPG)

## ข้อจำกัด/ความปลอดภัย
- Web app แบบ *Anyone* = ใครมี URL ก็เรียกได้ (ใช้ควอตา API ของคุณ) — แนะนำตั้ง `TOKEN` เพื่อจำกัด
- โควตา Apps Script `UrlFetchApp` ~20,000 ครั้ง/วัน (บัญชีทั่วไป) เพียงพอสำหรับงานทั่วไป
- เครื่องมือช่วย “จดข้อมูลบนฉลาก” เท่านั้น ไม่ใช่คำแนะนำการรักษา — ตัวเลขสำคัญควรตรวจกับกล่องจริง
