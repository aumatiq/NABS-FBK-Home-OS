# NABS&FBK Home OS — পরবর্তী ধাপের প্রম্পট (Batch 4, 5, 6)

নিচের তিনটা ব্লক Batch 2 ও Batch 3-এর (রিপোতে থাকা `NABS-FBK_Next_Steps_Prompts to fix all.md`)
একই কনভেনশনে লেখা — প্রতিটা সম্পূর্ণ স্বয়ংসম্পূর্ণ (self-contained) প্রম্পট, যেকোনো নতুন Claude
চ্যাটে পুরো ব্লকটা কপি-পেস্ট করলেই Claude রিপো ক্লোন করে, প্রাসঙ্গিক স্কিল ফাইল পড়ে, এবং সরাসরি
কাজ শুরু করতে পারবে।

## ⚠️ রান করার আগে গুরুত্বপূর্ণ নোট

- **Batch 2-এর আইটেম ৪ (লগইন কিবোর্ড) স্কিপ করুন** — Batch 6 সেটাকে বদলে দিচ্ছে সংকীর্ণ স্কোপ
  দিয়ে (শুধু App Lock স্ক্রিন + Settings Lock স্ক্রিন, পুরো অ্যাপ না)। Batch 2-এর বাকি ৩টা
  আইটেম (Member settings, Scan/Upload preview+folder+print, Category consolidation) স্বাভাবিকভাবে
  চালান।
- **সুপারিশকৃত রান অর্ডার:** Batch 2 (আইটেম ৪ বাদে) → **Batch 4** → Batch 3 → **Batch 5** →
  **Batch 6**
  কারণ Batch 3 নির্ভর করে Batch 2-এর ক্যাটাগরি কনসোলিডেশনের উপর, আর Batch 5-এর ড্যাশবোর্ড চার্ট
  কাজ Batch 3-এ যোগ হওয়া income চার্টের উপর ভিত্তি করে চলবে।
- প্রতিটা ব্যাচ ডেলিভার ও ডিভাইসে টেস্ট করে GitHub-এ push করার পরই পরের ব্যাচ নতুন চ্যাটে পেস্ট
  করুন।

---

## 🟧 BATCH 4 — এখানে কপি শুরু ↓

```
Repo: https://github.com/aumatiq/NABS-FBK-Home-OS.git

তুমি NABS&FBK Home OS প্রজেক্টে কাজ করছ — একটা personal family management PWA,
single-file frontend (index.html) + Google Apps Script backend (Code.gs) যা Google
Sheets-এ সিঙ্ক হয়। App-এ আছে: Dashboard, Entry, Finance, Monthly History, Home
Projects (DIY), Tasks, Shopping, Documents, News, Settings, Bills।

শুরু করার আগে অবশ্যই:
1. repo clone করে index.html, Code.gs, manifest.json, sw.js পড়ে বর্তমান স্ট্রাকচার বুঝে নাও।
   বিশেষভাবে Bills ক্যাটাগরি ম্যানেজমেন্ট কোড (CONFIG.billCategories, manageCategoriesModal,
   billCategoryModal সংক্রান্ত সব ফাংশন) মনোযোগ দিয়ে পড়ো।
2. /mnt/skills/public/frontend-design/SKILL.md পড়ো (UI কাজ আছে বলে)।
3. প্রতিটা পরিবর্তন "minimal-footprint, targeted edit" নীতি মেনে করবে — যা উল্লেখ নেই
   তা স্পর্শ করবে না। কাজ শেষে: JS syntax (node --check), HTML tag balance (Python),
   Code.gs পরিবর্তন হলে সেটাও আলাদাভাবে extract করে node --check দিয়ে ভ্যালিডেট করো,
   এবং সম্ভব হলে Playwright দিয়ে multiple viewport width-এ ভিজ্যুয়াল ভেরিফাই করবে।
4. ক্লারিফাইং প্রশ্ন থাকলে শুধু ক্লিকেবল অপশন আকারে দেবে, prose প্রশ্ন না।
5. সব ব্যাখ্যা বাংলায়, কোড/ফাইলনেম/কমিট মেসেজ ইংরেজিতে।

এই ব্যাচে ৩টা কাজ আছে — একটার পর একটা, প্রতিটা ডেলিভারির পর ভ্যালিডেট করে পরেরটায় যাবে:

### 1) বাগ ফিক্স: Bills ক্যাটাগরি এডিট করার পর মেইন ডিসপ্লে আপডেট হচ্ছে না (সবচেয়ে জরুরি)
- বর্তমানে Bills ট্যাবের "ক্যাটাগরি ম্যানেজ" মডাল থেকে কোনো ক্যাটাগরি এডিট (নাম পরিবর্তন,
  অর্ডার পরিবর্তন, ইত্যাদি) করার পর Bills ট্যাবের মেইন ডিসপ্লেতে (pending/overdue/paid card
  গুলোতে) পরিবর্তনটা প্রতিফলিত হচ্ছে না। রুট কজ খুঁজে বের করো — এটা render trigger মিসিং
  (edit করার পর renderBills() বা সমতুল্য ফাংশন কল হচ্ছে না), অথবা category id/reference
  ভুলভাবে ম্যাচ হচ্ছে (category rename করার পর পুরনো bill entry-গুলো নতুন নামের সাথে লিংকড
  থাকছে না), অথবা sync timing ইস্যু হতে পারে — আসল কারণ ডিবাগ করে ফিক্স করো।
- নিশ্চিত করো: ক্যাটাগরি অ্যাড, রিমুভ, বা এডিট — যেকোনো একটা করলেই সাথে সাথে (ক) স্থানীয়ভাবে
  Bills ট্যাবের মেইন ডিসপ্লে re-render হয়, (খ) Google Sheets-এর Bills ক্যাটাগরি ডেটাও আপডেট
  হয়, এবং (গ) এই পরিবর্তন অ্যাপের যেখানেই bill category রেফারেন্স করা হয় (Entry prefill,
  Dashboard summary যদি থাকে, Search) সব জায়গায় সামঞ্জস্যপূর্ণভাবে দেখা যায়।
- ফিক্স করার পর একটা টেস্ট সিনারিও বাংলায় লিখে দেখাও: একটা ক্যাটাগরির নাম বদলে, Bills ট্যাব
  রিফ্রেশ ছাড়াই মেইন ডিসপ্লেতে নতুন নামটা দেখাচ্ছে কিনা।

### 2) Settings-এ অ্যাপের মেইন ট্যাব ম্যানেজমেন্ট (নাম এডিট + ড্র্যাগ/ড্রপ রিঅর্ডার)
- এটা বিদ্যমান "ক্যাটাগরি ম্যানেজ" (Bills category, Task category ইত্যাদি) থেকে আলাদা —
  এটা অ্যাপের প্রধান নেভিগেশন ট্যাবগুলোর (Dashboard, Entry, Finance, Monthly History, Home
  Projects, Tasks, Shopping, Documents, News, Bills, Settings) জন্য।
- Settings-এ একটা নতুন সেকশন যোগ করো ("ট্যাব ম্যানেজমেন্ট" বা সমতুল্য নাম) যেখানে:
  - প্রতিটা ট্যাবের ডিসপ্লে-নাম (যা ইউজার দেখে) এডিট করা যাবে — internal routing id/key
    অপরিবর্তিত থাকবে, শুধু লেবেল/টেক্সট বদলাবে, যেন href/onclick/element id রেফারেন্স ভেঙে
    না যায়।
  - ড্র্যাগ ও ড্রপ করে ট্যাবগুলোর অর্ডার পরিবর্তন করা যাবে (bottom nav / side nav যেখানেই
    ট্যাব লিস্ট আছে)।
- এই নাম ও অর্ডার Google Sheets-এ একটা নতুন sheet/tab-এ (যেমন "TabConfig" — কলাম: tabId,
  displayName, order) স্বয়ংক্রিয়ভাবে সিঙ্ক হবে, যেন অন্য ডিভাইস থেকে লগইন করলেও একই
  নাম/অর্ডার দেখা যায়।
- Code.gs-এ এই TabConfig sheet তৈরি/রিড/রাইট করার জন্য সম্পূর্ণ working ফাংশন যোগ করো —
  ম্যানুয়াল স্টেপ না, পুরো কপি-পেস্ট রেডি স্ক্রিপ্ট হিসেবে।
- পরিবর্তনটা শুধু ডিসপ্লে লেয়ারে থাকবে — অন্তর্নিহিত ট্যাব সুইচিং লজিক/state management
  অক্ষত থাকবে।

### 3) Settings — Web App URL এন্ট্রি করলেই ফুল অটো-সিঙ্ক + Folder ID/Reset Email অটো-পপুলেট
- Settings-এ Web App URL ফিল্ডে ইউজার URL পেস্ট/এন্টার করে সেভ করার সাথে সাথেই (পরবর্তী
  নিয়মিত ইন্টারভাল সিঙ্কের অপেক্ষা না করে) একটা তাৎক্ষণিক পূর্ণাঙ্গ bidirectional sync
  ট্রিগার হবে (performSafeSync বা সমতুল্য পুরো ফাংশন কল)।
- এই সিঙ্কের পর, Settings-এর "Folder ID" ও "Reset Email" ফিল্ড — এই দুটো মান connected
  Google Sheet থেকে স্বয়ংক্রিয়ভাবে পড়ে এসে UI-তে পপুলেট হবে (Google Sheet-ই single source
  of truth, লোকাল কনফিগ না)।
- বর্তমানে Google Sheets backend-এ যদি Folder ID / Reset Email স্টোর করার জায়গা না থাকে,
  Code.gs-এ একটা "Config" sheet (বা বিদ্যমান কোনো config sheet থাকলে সেখানেই) key-value
  আকারে এই দুইটা যোগ করো, সাথে read endpoint (Web App URL সেভ হওয়ার পর কল হবে) এবং write
  endpoint (লোকালি এই মান পরিবর্তন করলে Sheet-এও আপডেট হবে) — দুটোই সম্পূর্ণ কাজ করা অবস্থায়
  ডেলিভার করো।
- নিশ্চিত করো ভুল/অকার্যকর Web App URL দিলে স্পষ্ট এরর মেসেজ দেখাবে (সিঙ্ক silently fail
  করবে না)।

কাজ শেষে index.html এবং Code.gs (যা যা পরিবর্তন হয়েছে) সম্পূর্ণ কপি-পেস্ট রেডি ফাইল হিসেবে
ডেলিভার করো, প্রতিটা পরিবর্তনের সংক্ষিপ্ত সারাংশ ও ভেরিফিকেশনের ফলাফল বাংলায় দাও।
```

## 🟧 BATCH 4 — কপি শেষ ↑

---

## 🟨 BATCH 5 — এখানে কপি শুরু ↓

```
Repo: https://github.com/aumatiq/NABS-FBK-Home-OS.git

তুমি NABS&FBK Home OS প্রজেক্টে কাজ করছ — একটা personal family management PWA,
single-file frontend (index.html) + Google Apps Script backend (Code.gs) যা Google
Sheets-এ সিঙ্ক হয়।

এই ব্যাচটা মূলত UI/লেআউট-কেন্দ্রিক, কিন্তু আগের কাজে (mobile portrait header rebuild)
device-specific রেন্ডারিং সমস্যা হয়েছিল বলে এখানেও সতর্কভাবে multi-viewport এবং
multi-orientation টেস্টিং লাগবে।

শুরু করার আগে অবশ্যই:
1. repo clone করে index.html পড়ো — বিশেষভাবে header-এর CSS Grid implementation
   (display:contents ব্যবহার করা mobile portrait layout), এবং Dashboard-এর বাজেট
   অগ্রগতি সেকশন (id="budgetProgress", gauge-item/gauge-canvas-wrap ক্লাসগুলো,
   এবং Batch 3-এ যোগ হওয়া income চার্ট কোড যদি ইতিমধ্যে থাকে) ভালোভাবে বুঝে নাও।
2. /mnt/skills/public/frontend-design/SKILL.md পড়ো।
3. আগের memory অনুযায়ী স্কোপ ডিসিপ্লিন মেনে চলবে: "সেন্টার অ্যালাইন" মানে বিদ্যমান গঠনের
   মধ্যে কন্টেন্ট অ্যালাইনমেন্ট ঠিক করা, পুরো লেআউট রিস্ট্রাকচার করা না। Desktop layout,
   যা flexbox দিয়ে করা এবং ঠিক আছে, একদম স্পর্শ করবে না।
4. কাজ শেষে: JS syntax (node --check), HTML tag balance (Python), এবং Playwright দিয়ে
   অন্তত এই ভিউপোর্টগুলোতে স্ক্রিনশট নিয়ে ভিজ্যুয়াল ভেরিফাই করবে: 375×667 (portrait),
   667×375 (landscape — এই ব্যাচের মূল ফোকাস), 414×896 (portrait), 896×414 (landscape),
   এবং একটা ডেস্কটপ width (1280px) যেন ডেস্কটপ ভাঙেনি তা নিশ্চিত হয়।

এই ব্যাচে ৩টা কাজ আছে:

### 1) মোবাইল রোটেট/ল্যান্ডস্কেপ বাগ ফিক্স (সবচেয়ে জরুরি এই ব্যাচে)
- বর্তমান header মোবাইল পোর্ট্রেটে CSS Grid (৩ কলাম × ৩ রো, display:contents কৌশল) দিয়ে
  ঠিকভাবে কাজ করে, কিন্তু ডিভাইস রোটেট করে ল্যান্ডস্কেপে গেলে header ভেঙে যাচ্ছে/সঠিকভাবে
  দেখাচ্ছে না।
- প্রথমে ডিবাগ করো কোন media query landscape orientation-কে কভার করছে (বা করছে না) —
  সম্ভবত বর্তমান portrait-only grid rule কোনো landscape-specific override ছাড়াই
  landscape-এও প্রযোজ্য হয়ে যাচ্ছে, যার ফলে narrow-height landscape-এ কলাম/রো
  squeeze হয়ে যাচ্ছে।
- landscape orientation-এর জন্য আলাদা media query (orientation:landscape এবং/অথবা
  max-height ভিত্তিক) যোগ করো যা header-কে landscape-এ readable ও properly aligned
  রাখবে — grid কলাম/রো কাউন্ট দরকার হলে landscape-এর জন্য আলাদা adapt করবে, কিন্তু
  portrait grid rule (যা এখন ঠিক আছে) অপরিবর্তিত রাখবে।
- desktop flexbox layout সম্পূর্ণ অক্ষত রাখবে।

### 2) Client header name + ১ম ও ২য় কলাম সেন্টার অ্যালাইনমেন্ট
- header grid-এর ১ম কলাম (logo/sync/lock আইকন) এবং ২য় কলাম (client name + clock) —
  এই দুই কলামের ভেতরের কন্টেন্ট সেন্টার-অ্যালাইন (vertical + horizontal, যেভাবে প্রযোজ্য)
  করো — শুধু align-items/justify-content/text-align জাতীয় CSS প্রোপার্টি অ্যাডজাস্ট
  করবে, গ্রিড স্ট্রাকচার (কলাম/রো সংখ্যা, কোন এলিমেন্ট কোন কলামে) অপরিবর্তিত রাখবে।
- ৩য় কলাম (তারিখ ফরম্যাটগুলো — right-aligned) স্পর্শ করবে না।
- Portrait ও Landscape দুই orientation-এই এই অ্যালাইনমেন্ট সঠিকভাবে বজায় থাকবে।

### 3) Dashboard বাজেট অগ্রগতি — চার্ট অর্ডার: আয় → সঞ্চয় → ব্যয়
- Dashboard-এর "বাজেট অগ্রগতি" সেকশনে (id="budgetProgress") এখন যদি Batch 3-এ যোগ হওয়া
  income চার্ট এবং বিদ্যমান expense চার্ট থাকে (savings চার্ট না থাকলে এখানেই যোগ করো —
  বিদ্যমান savings ক্যাটাগরি ডেটা থেকে, একই gauge স্টাইলে), তাহলে এই তিনটা সেকশনের
  প্রদর্শনের ক্রম নিশ্চিত করো এই অর্ডারে হবে: **১) আয় (Income) → ২) সঞ্চয় (Savings) →
  ৩) ব্যয় (Expense)**।
- প্রতিটা সেকশনের ভেতরের ক্যাটাগরি গেজগুলো (যেমন expense-এর ভেতরে বিভিন্ন খরচের ক্যাটাগরি)
  আগের মতোই থাকবে — শুধু তিনটা বড় সেকশনের (income/savings/expense) ক্রম ঠিক করো এবং savings
  সেকশন না থাকলে সেটা expense-এর same visual language (gauge style, রং প্যাটার্ন, লেআউট)
  অনুসরণ করে যোগ করো।
- HTML markup-এ সরাসরি সঠিক অর্ডারে বসাও (CSS order প্রোপার্টি দিয়ে visual hack করার বদলে),
  যেন DOM order ও visual order দুটোই মেলে (accessibility ভালো থাকবে)।

কাজ শেষে index.html সম্পূর্ণ কপি-পেস্ট রেডি ফাইল হিসেবে ডেলিভার করো, প্রতিটা viewport/
orientation-এর স্ক্রিনশট-ভিত্তিক ভেরিফিকেশন রেজাল্ট এবং পরিবর্তনের সংক্ষিপ্ত সারাংশ বাংলায় দাও।
```

## 🟨 BATCH 5 — কপি শেষ ↑

---

## 🟥 BATCH 6 — এখানে কপি শুরু ↓

```
Repo: https://github.com/aumatiq/NABS-FBK-Home-OS.git

তুমি NABS&FBK Home OS প্রজেক্টে কাজ করছ — একটা personal family management PWA,
single-file frontend (index.html) + Google Apps Script backend (Code.gs)।

শুরু করার আগে অবশ্যই:
1. repo clone করে index.html পড়ো — বিশেষভাবে App Lock স্ক্রিন এবং Settings-এর Lock/
   লগইন স্ক্রিনের বিদ্যমান HTML/CSS/JS (two-tier password lock system) ভালোভাবে বুঝে নাও।
2. /mnt/skills/public/frontend-design/SKILL.md পড়ো।
3. কাজ শেষে: JS syntax (node --check), HTML tag balance (Python), এবং Playwright দিয়ে
   মোবাইল viewport (375×667 এবং 414×896) স্ক্রিনশট নিয়ে ভিজ্যুয়াল ভেরিফাই করবে — বিশেষভাবে
   চেক করো কীবোর্ড শো হওয়ার পর নিচের ইনপুট/বাটন কোনো overlap বা cut-off হচ্ছে কিনা।
4. স্কোপ: এই কাস্টম কিবোর্ড **শুধুমাত্র দুইটা জায়গায়** বসবে — (ক) App Lock স্ক্রিন
   (মূল অ্যাপ আনলক করার পাসওয়ার্ড উইন্ডো), এবং (খ) Settings-এর Lock/লগইন স্ক্রিন। অ্যাপের
   অন্য কোনো ইনপুট ফিল্ডে (Entry ফর্ম, Search বক্স, ইত্যাদি) এই কাস্টম কিবোর্ড বসাবে না —
   সেগুলো ডিভাইসের native কীবোর্ডই ব্যবহার করবে যেমন এখন করছে।

এই ব্যাচে ১টা ফিচার আছে:

### App Lock ও Settings Lock স্ক্রিনে কাস্টম অন-স্ক্রিন কিবোর্ড
- App Lock স্ক্রিন এবং Settings Lock স্ক্রিন — এই দুইটাতে পাসওয়ার্ড/PIN ইনপুট বক্সে ফোকাস
  হলে (বা স্ক্রিন ওপেন হওয়ার সাথে সাথেই) একটা on-screen কাস্টম কিবোর্ড কম্পোনেন্ট শো হবে
  (device-এর native OS কিবোর্ড না — অ্যাপের নিজস্ব ডিজাইন করা on-screen কিবোর্ড UI)।
- স্ক্রিন ওপেন হওয়ার সাথে সাথেই ইনপুট বক্স ডিফল্টভাবে ফোকাসড থাকবে (আলাদা ট্যাপ ছাড়াই টাইপ
  করা শুরু করা যাবে)।
- Enter/Go কি (key) চাপলে বা ফিজিক্যাল কিবোর্ডে Enter চাপলে ফর্ম সাবমিট/লগইন ট্রিগার হবে।
- কিবোর্ড লেআউট ফিজিক্যাল কিবোর্ডের মতো organized তিনটা সেকশনে ভাগ করা থাকবে:
  - **আলফাবেটিক সেকশন** (মূল, সবসময় ভিজিবল বা ডিফল্ট ট্যাব)
  - **নিউমেরিক সেকশন** (আলাদা, একটা টগল/ট্যাব দিয়ে সুইচ করা যাবে অথবা যদি ইনপুট শুধু
    PIN/নাম্বার হয় তাহলে numeric-first লেআউট দেখাবে)
  - **সিম্বল সেকশন** (আলাদা টগল/ট্যাব দিয়ে)
  - তিনটা সেকশন একসাথে মিশে থাকবে না — clear ভিজ্যুয়াল সেপারেশন এবং সহজ টগল (যেমন "123"
    এবং "#+=" জাতীয় toggle key, iOS/Android native কিবোর্ডের প্যাটার্নের মতো) থাকবে।
- প্রতিটা কি (key) কমপক্ষে ~44×44px টাচ টার্গেট সাইজে হবে (Apple/Google accessibility
  গাইডলাইন), যেন মোবাইল স্ক্রিনে miss-touch না হয়। কি-গুলোর মধ্যে যথেষ্ট gap/spacing থাকবে।
- Backspace, Enter/Go, এবং কিবোর্ড বন্ধ/hide করার একটা কি — এই স্ট্যান্ডার্ড কিগুলোও থাকবে।
- AUMATIQ dark theme brand token ব্যবহার করবে: bg #0A0A0F, card/key bg #0D1117, active/
  pressed key indigo #4F46E5, gold #F5A623 (Enter/action key accent-এর জন্য ব্যবহার করা
  যেতে পারে), টেক্সট #F8F9FF, ফন্ট Inter (কি লেবেলের জন্য)।
- কিবোর্ড শো হওয়ার সময় নিচের কোনো UI এলিমেন্ট (যদি থাকে) cover/overlap না করে সেটা
  নিশ্চিত করো — প্রয়োজনে ইনপুট এলাকা কিবোর্ডের উপরে scroll/reposition হবে।
- এই কাস্টম কিবোর্ড ইমপ্লিমেন্ট করার সময় ডিভাইসের native কিবোর্ড pop-up হওয়া বন্ধ করতে
  হবে (readonly ইনপুট + JS দিয়ে ভ্যালু ম্যানেজ করা, অথবা inputmode="none" ব্যবহার — যেটা
  সব মোবাইল ব্রাউজারে বেশি রিলায়েবল সেটা ব্যবহার করো)।

কাজ শেষে index.html সম্পূর্ণ কপি-পেস্ট রেডি ফাইল হিসেবে ডেলিভার করো, মোবাইল viewport
স্ক্রিনশট-ভিত্তিক ভেরিফিকেশন রেজাল্ট এবং পরিবর্তনের সংক্ষিপ্ত সারাংশ বাংলায় দাও।
```

## 🟥 BATCH 6 — কপি শেষ ↑

---

### ব্যবহারের নিয়ম

- একবারে একটা ব্যাচ পেস্ট করুন। প্রতিটা ব্যাচ ডেলিভার ও ডিভাইসে টেস্ট করে GitHub-এ push
  করার পর, তারপর নতুন চ্যাটে পরের ব্যাচ পেস্ট করুন।
- সুপারিশকৃত অর্ডার: **Batch 2 (আইটেম ৪ বাদে) → Batch 4 → Batch 3 → Batch 5 → Batch 6**
- নতুন চ্যাটে Claude-এর কাছে আগের কোনো মেমরি/কনটেক্সট না থাকলেও এই প্রম্পট নিজে থেকেই
  repo link, প্রজেক্ট প্রেক্ষাপট, এবং কাজের ডিটেইলস দিয়ে দেয় — তাই আলাদা করে কিছু ব্যাখ্যা
  করার দরকার নেই।
- প্রতিটা ব্যাচেই "find and fix any other bug you notice" নীতি প্রযোজ্য — যদি সংশ্লিষ্ট
  ফাইল পড়ার সময় স্কোপের বাইরের কোনো স্পষ্ট বাগ চোখে পড়ে, সেটা fix না করে শুধু flag করে
  বাংলায় জানানো উচিত (মেমরির "change discipline" নীতি অনুযায়ী)।
