# NABS&FBK Home OS — Deployment Guide
## Version 2.0 | সম্পূর্ণ ডেপ্লয়মেন্ট গাইড

---

## ফাইল লিস্ট (৫টি ফাইল)

```
nabs-home-os/
├── index.html         ← পুরো PWA frontend (lock screen + সব tab)
├── Code.gs            ← Apps Script backend (email, OTP, sync, drive)
├── manifest.json      ← PWA manifest
├── sw.js              ← Service Worker (offline support)
└── DEPLOYMENT.md      ← এই গাইড
```

---

## STEP 1 — Google Sheets সেটআপ

### ১.১ নতুন Spreadsheet তৈরি করো

1. [sheets.new](https://sheets.new) খোলো
2. নাম দাও: **NABS&FBK Home OS Data**
3. URL থেকে Spreadsheet ID copy করো:
   ```
   https://docs.google.com/spreadsheets/d/[THIS_IS_THE_ID]/edit
   ```
4. এই ID টা রেখে দাও — পরে Code.gs-এ লাগবে

---

## STEP 2 — Apps Script প্রজেক্ট তৈরি করো

### ২.১ Apps Script খোলো

1. Google Sheets-এ যাও → **Extensions → Apps Script**
2. বাম দিকে `Code.gs` দেখবে — সব existing code মুছে ফেলো
3. `Code.gs` ফাইলের সম্পূর্ণ কোড paste করো

### ২.২ Code.gs-এ Config আপডেট করো

ফাইলের একদম উপরে এই অংশ খোলো:

```javascript
const CONFIG = {
  SHEET_ID:         '',   // ← Step 1.3 থেকে পাওয়া Spreadsheet ID
  APP_PASS_HASH:    '',   // ← নিচে বলছি কীভাবে পাবে
  ADMIN_PASS_HASH:  '',   // ← নিচে বলছি কীভাবে পাবে
  RESET_EMAIL:      '',   // ← তোমার Gmail address
  REPORT_EMAIL:     '',   // ← Weekly/monthly report কোথায় যাবে
  DRIVE_FOLDER_ID:  '',   // ← Google Drive folder ID (STEP 3-এ)
  TIMEZONE:         'Asia/Dhaka'
};
```

### ২.৩ Password Hash বের করো

Apps Script Editor-এ নিচে `setup()` function-এ যাও এবং:

1. **Run → setup()** চাপো
2. নিচে Execution Log-এ দেখবে:
   ```
   Hash for NABSFBK2025: [একটা কোড]
   Hash for ADMIN2025: [আরেকটা কোড]
   ```
3. এই দুটো hash কপি করে CONFIG-এ বসাও:
   ```javascript
   APP_PASS_HASH:   '[প্রথম hash]',
   ADMIN_PASS_HASH: '[দ্বিতীয় hash]',
   ```

> **⚠️ গুরুত্বপূর্ণ:** Default password এই দুটো:
> - App Lock: `NABSFBK2025`
> - Settings/Admin: `ADMIN2025`
> পরে Settings ট্যাব থেকে যেকোনো সময় পরিবর্তন করতে পারবে।

---

## STEP 3 — Google Drive Folder তৈরি করো

1. [drive.google.com](https://drive.google.com) খোলো
2. **New → Folder** → নাম দাও: **NABS&FBK Documents**
3. Folder-এ ঢোকো → URL থেকে Folder ID copy করো:
   ```
   https://drive.google.com/drive/folders/[THIS_IS_THE_FOLDER_ID]
   ```
4. Code.gs-এ `DRIVE_FOLDER_ID`-এ বসাও

---

## STEP 4 — Apps Script Deploy করো

### ৪.১ Web App হিসেবে Deploy

1. Apps Script Editor-এ উপরে **Deploy → New deployment**
2. Type: **Web app**
3. Settings:
   ```
   Execute as: Me (your account)
   Who has access: Anyone
   ```
4. **Deploy** বাটন চাপো
5. Permission চাইলে **Allow** দাও
6. Web App URL copy করো — এরকম দেখাবে:
   ```
   https://script.google.com/macros/s/AKfycb.../exec
   ```
7. এই URL রেখে দাও

---

## STEP 5 — GitHub-এ Upload করো

### ৫.১ Repository তৈরি করো

1. [github.com](https://github.com) → **New repository**
2. নাম: `nabsfbk-home-os`
3. **Private** রাখো (family data)
4. **Create repository**

### ৫.২ GitHub Desktop দিয়ে Upload

1. GitHub Desktop খোলো → **File → Clone repository**
2. সদ্য তৈরি `nabsfbk-home-os` clone করো
3. Clone করা folder-এ এই ৪টি ফাইল রাখো:
   - `index.html`
   - `manifest.json`
   - `sw.js`
4. ⚠️ **`Code.gs` GitHub-এ upload করো না** — এটা শুধু Apps Script-এ থাকবে
5. GitHub Desktop-এ commit message লিখো:
   ```
   feat: initial deploy — NABS&FBK Home OS v2.0
   ```
6. **Commit to main → Push origin**

---

## STEP 6 — Vercel-এ Deploy করো

### ৬.১ Vercel Setup

1. [vercel.com](https://vercel.com) → **Add New Project**
2. GitHub থেকে `nabsfbk-home-os` import করো
3. Settings (সব default রাখো):
   ```
   Framework Preset: Other
   Root Directory: ./
   Build Command: (খালি রাখো)
   Output Directory: ./
   ```
4. **Deploy** চাপো
5. কিছুক্ষণ পরে live URL পাবে:
   ```
   https://nabsfbk-home-os.vercel.app
   ```

### ৬.২ Custom Domain (ঐচ্ছিক)

Vercel Dashboard → Settings → Domains → তোমার domain যোগ করো

---

## STEP 7 — PWA প্রথমবার Configure করো

### ৭.১ অ্যাপ খোলার পর

1. Vercel URL খোলো
2. **App Lock Screen** দেখবে
3. Default password: `NABSFBK2025` — দিয়ে ঢোকো

### ৭.২ Settings Configure করো

Settings ট্যাবে ঢুকতে Admin password লাগবে: `ADMIN2025`

Settings-এ গিয়ে এগুলো করো:

**1. Apps Script URL সেট করো:**
```
Settings → Google Sheets Connection → URL field-এ STEP 4-এর Web App URL বসাও → Save
```

**2. পরিবারের সদস্য যোগ করো:**
```
Settings → পরিবারের সদস্য → নাম যোগ করো (NABS, FBK, এবং অন্যরা)
```

**3. Budget Goal সেট করো:**
```
Settings → মাসিক বাজেট লক্ষ্য → আয়/ব্যয়/সঞ্চয় লক্ষ্য দাও
```

**4. Email Setup করো:**
```
Settings → Email Notification → তোমার email দাও → Weekly/Monthly schedule সেট করো
```

**5. Password পরিবর্তন করো (সুপারিশকৃত):**
```
Settings → Password & Security → নতুন App Password + Admin Password দাও
```

### ৭.৩ Mobile-এ Install করো (PWA)

**Android (Chrome):**
```
Browser-এ URL খোলো → ⋮ মেনু → "Add to Home screen" → Install
```

**iPhone (Safari):**
```
Safari-এ URL খোলো → Share বাটন → "Add to Home Screen" → Add
```

---

## STEP 8 — Apps Script Triggers চালু করো

এটা না করলে weekly/monthly email আসবে না।

### ৮.১ Trigger Install

1. Apps Script Editor খোলো
2. **Run → installTriggers()** চাপো
3. Execution Log-এ দেখবে: `Triggers installed successfully`

### ৮.২ Verify Triggers

1. বাম দিকে **⏰ Triggers** আইকনে ক্লিক করো
2. দেখবে:
   - `sendWeeklyReport` — সাপ্তাহিক (তোমার দেওয়া বার + সময়ে)
   - `sendMonthlyReport` — মাসিক (তোমার দেওয়া তারিখ + সময়ে)

---

## STEP 9 — YouTube API Key (News Tab-এর জন্য)

YouTube suggested videos দেখাতে হলে:

### ৯.১ API Key তৈরি করো (ফ্রি)

1. [console.cloud.google.com](https://console.cloud.google.com) → New Project
2. **APIs & Services → Enable APIs → YouTube Data API v3 → Enable**
3. **APIs & Services → Credentials → Create Credentials → API Key**
4. Key copy করো

### ৯.২ App-এ বসাও

```
Settings (Admin password দিয়ে) → News & YouTube Configuration → YouTube API Key field-এ বসাও → Save
```

> ⚠️ **Free Limit:** দিনে ১০,০০০ quota unit (~১০০টি search)। পারিবারিক ব্যবহারে যথেষ্ট।

---

## Password Summary

| পাসওয়ার্ড | Default | পরিবর্তন কোথায় |
|---|---|---|
| App Lock (প্রতিবার ঢুকতে) | `NABSFBK2025` | Settings → Password & Security |
| Admin/Settings Lock | `ADMIN2025` | Settings → Password & Security |
| Password Reset | Email OTP | Settings → Reset Email সেট করো |

---

## Quick Reference — কী কোথায় আছে

| ফিচার | কোথায় |
|---|---|
| নতুন আয়/ব্যয় এন্ট্রি | Entry ট্যাব |
| এই মাসের হিসাব | Finance ট্যাব |
| আগের মাসের হিসাব | মাসিক হিসাব ট্যাব (◀ ▶ দিয়ে মাস পরিবর্তন) |
| নতুন DIY প্রজেক্ট | Home Projects ট্যাব |
| প্রজেক্টে খরচ যোগ | Project card-এ ➕ বাটন |
| Shopping list | Shopping ট্যাব |
| Document/Note সেভ | Documents ট্যাব |
| File Google Drive-এ upload | Documents → নতুন Document → ফাইল drag করো |
| News দেখা | News ট্যাব → Refresh চাপো |
| নতুন Tab যোগ | Settings বা Tab bar-এর শেষে ＋ |
| Google Sheets sync | Header-এর ⟳ বাটন |

---

## Troubleshooting — সমস্যা হলে

### Connection Error দেখাচ্ছে
→ Settings → Apps Script URL ঠিকমতো বসানো আছে কিনা দেখো
→ Apps Script-এ "Anyone" access দেওয়া আছে কিনা দেখো

### Email আসছে না
→ Apps Script → installTriggers() আবার run করো
→ CONFIG-এ REPORT_EMAIL ঠিকমতো আছে কিনা দেখো

### Drive Upload হচ্ছে না
→ Settings-এ DRIVE_FOLDER_ID সেট আছে কিনা দেখো
→ Apps Script-এর Google Account-এর সেই Folder-এ access আছে কিনা দেখো

### Password ভুলে গেছি
→ Lock screen-এ "পাসওয়ার্ড ভুলে গেছেন?" → OTP flow follow করো
→ OTP-র জন্য Settings-এ Reset Email আগে থেকে সেট করা থাকতে হবে

### PWA Install হচ্ছে না
→ HTTPS URL থেকে খুলতে হবে (Vercel URL-এ HTTPS আছে ✅)
→ `manifest.json` সঠিকভাবে serve হচ্ছে কিনা দেখো

---

## Update করার নিয়ম

### Frontend Update (index.html পরিবর্তন)
```
1. index.html edit করো
2. GitHub Desktop → Commit → Push
3. Vercel স্বয়ংক্রিয়ভাবে redeploy করবে (~১ মিনিটে)
```
Commit format:
```
fix: entry form category dropdown refresh issue
feat: add new document type — Certificate
style: improve mobile shopping list layout
```

### Backend Update (Code.gs পরিবর্তন)
```
1. Apps Script Editor-এ Code.gs edit করো
2. Deploy → Manage deployments → Edit → New version → Deploy
3. নতুন URL পেলে Settings-এ আপডেট করো
   (⚠️ সাধারণত URL একই থাকে — "New version" করলেই হয়)
```

---

## Cost Summary — মাসিক খরচ

| Tool | Cost |
|---|---|
| Google Sheets | ফ্রি |
| Google Apps Script | ফ্রি (6 min/day execution limit, যথেষ্ট) |
| Google Drive | ফ্রি (15 GB) |
| GitHub | ফ্রি (private repo) |
| Vercel | ফ্রি (100 GB bandwidth/month) |
| YouTube Data API | ফ্রি (~100 searches/day) |
| RSS News Feed (rss2json.com) | ফ্রি (10,000 requests/month) |
| **মোট মাসিক খরচ** | **৳০** |

---

*Built with ❤️ for NABS & FBK Family — AUMATIQ Internal Tool*
*Version 2.0 | Two-Tier Password | Multi-Calendar | News+YouTube | Google Drive*
