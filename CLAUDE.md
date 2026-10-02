# CRM Space · CRM Back-office

ระบบดูงบคะแนน (point budget) ของพนักงาน จากข้อมูล CRM ของ Wongnai POS / Foodstory
A small web app where employees see the loyalty points they have given out against their monthly budget, and admins see everyone's usage and assign teams.

## Owner / ผู้ดูแล

- Bie (GitHub: `waranchit-source`) · คุยภาษาไทย · มักอ่านจากมือถือ ตอบสั้น อ่านง่าย

## กติกาการทำงาน (สำคัญมาก) · Working rules

1. แก้ไข **เฉพาะ** สิ่งที่ถูกขอเท่านั้น ส่วนอื่นห้ามแตะ
   Change only what was asked. Everything else is considered working and must stay as is.
2. ถ้าสิ่งที่ขอทำให้ต้องแก้ส่วนอื่น ทำได้ แต่ต้องบอกให้ชัดว่าเปลี่ยนอะไรไปบ้าง
   If a requested change forces changes elsewhere, list exactly what else changed and why.
3. ทดสอบและตรวจหา bug ก่อนส่งงานทุกครั้ง (ถ้าแก้ logic ให้เทียบผลลัพธ์กับโค้ดเดิม)
   Test before delivering. When refactoring logic, prove the output matches the old code.
4. ส่งโค้ดฉบับเต็ม ห้ามย่อ ห้ามตัด และ **ห้ามใส่ comment ในโค้ด**
   Deliver full, unabridged code with no comments in the code.
5. แก้แค่บางไฟล์ ส่งเฉพาะไฟล์ที่แก้ ถ้าแก้ทั้งหมด ส่งทั้งหมด
   Send only the changed files (full file each), or everything if everything changed.
6. ถ้ามีขั้นตอนที่ Bie ต้องทำเอง (deploy, ตั้งค่า) ต้องอธิบายและสอนทีละขั้น
   Explain and teach any manual step (deploy, settings) step by step.
7. ต้องขออนุญาตก่อนเข้าถึงข้อมูลสำคัญ (ข้อมูลลูกค้า, รหัสผ่าน)
   Ask permission before touching sensitive data.

## Architecture / โครงสร้างระบบ

```
Browser (index.html on GitHub Pages)
   │  POST text/plain  { action, data }
   ▼
Google Apps Script web app (Code.gs, bound to the Google Sheet)
   │  reads / writes
   ▼
Google Sheet "CRM-Back Office 2026"
```

- **Frontend**: `index.html` in this repo, a single file (HTML + CSS + vanilla JS, Chart.js from CDN). Hosted on GitHub Pages: https://waranchit-source.github.io/crm-budget-system/
  - The Apps Script URL is the `WEB_APP_URL` constant near the top of the script.
  - Bilingual UI (EN/TH) through the `langData` object. Every new visible string needs both languages.
  - Remembers the last dashboard in `localStorage` (`crm_cache_<email>`) so repeat visits render instantly, then refreshes in the background.
  - Mobile (≤600px): tables become cards using `data-label` on each `<td>`.
- **Backend**: `Code.gs` lives inside the Google Sheet (Extensions → Apps Script). It is **not** in this repo.
  - Actions handled by `doPost`: `login`, `register`, `getDashboard`, `updateTeam`.
  - `getDashboard` serves from `CacheService` (chunked, prefix `dash_v2`). Cache is cleared by the installable `onSheetChange` trigger and rebuilt by the `warmCache` time trigger every 10 minutes. `registerUser` / `updateTeam` patch the cache after writing.
  - `testSpeed()` logs build time vs cache read time.
- **Sheet tabs used**
  - `BOF & IN`: transactions (A date, B member name, C phone, D email, F points, K branch name, L month tag like `Jan-26`).
  - `Remaining Budget`: per-employee budget (A name, B phone, C budget, F email).
  - `Web_Users`: web accounts (email, role `Admin`/`User`, team).
  - `Dropdown`: column B is the team list.
  - `Raw Data`: full transaction export from Foodstory (https://crm-owner.foodstory.co/report/transaction), pasted in by hand.

## Deploy / วิธี deploy

### หน้าเว็บ (index.html)
- แก้ไฟล์แล้ว commit ขึ้น branch `main` GitHub Pages จะอัปเดตเองใน 1–5 นาที
- ผู้ใช้กด Ctrl+Shift+R (คอม) หรือปิดแท็บเปิดใหม่ (มือถือ) เพื่อเห็นเวอร์ชันใหม่

### Apps Script (Code.gs)
1. เปิดชีท → Extensions → Apps Script → วางโค้ด → Save
2. Deploy → Manage deployments → ✏️ Edit → Version: **New version** → Deploy
   (ห้ามกด New deployment เพราะลิงก์ `/exec` จะเปลี่ยน ต้องไปแก้ `WEB_APP_URL` ใน index.html)
3. Execute as = Me, Who has access = Anyone
4. ถ้าเพิ่ม/เปลี่ยน trigger ให้ Run `installTriggers` 1 ครั้ง

## Testing / การทดสอบ

- Frontend: serve the folder locally and drive it with Playwright, mocking the Apps Script URL with `page.route`. Check desktop (1440, 1280), tablet (820) and phones (390, 360) for horizontal overflow.
- Backend: run old and new `Code.gs` side by side in Node `vm` with mocked `SpreadsheetApp`, `CacheService`, `LockService`, and compare `doPost` output byte for byte.

## Things to keep out of this repo

This repository is **public**. Never commit the sheet ID, `Code.gs`, user data, exports, or anything from `Web_Users`.
