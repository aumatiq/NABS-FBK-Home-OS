# BATCH PROMPT — Category-Level Ledger Sync Verification & Audit
### NABS&FBK Home OS — for a separate/new Claude session

Copy-paste this entire document as the first message in a new chat.

---

## 1. CONTEXT (read this first)

Repo: `https://github.com/aumatiq/NABS-FBK-Home-OS.git`
Files: `index.html` (single-file PWA frontend) + `Code.gs` (Google Apps Script
backend → Google Sheets).

**Read skill files first** (per project convention) before touching any code.

A previous session already fixed the *transport* layer of category sync:

- `CONFIG.categories` (Finance expense/income/savings), `CONFIG.projectCategories`,
  `CONFIG.docCategories`, `CONFIG.shoppingLists`, and `CONFIG.billCategories` are
  now serialized into one JSON blob and pushed to the Settings sheet
  (`categories_config_json` key) via `pushDataToBackend()` → `buildCategoriesConfigJson()`.
- On load, `loadSettingsFromBackend()` pulls that blob and applies it via
  `applyCategoriesConfigFromBackend()` **before** `generateMonthlyBills()` runs —
  this fixed the bug where a fresh device login was generating duplicate Bill
  categories (because it used to regenerate its own local category ids).
- Category-manage UI was consolidated: it now only exists in Settings
  (`🗂️ অন্যান্য ক্যাটাগরি ম্যানেজমেন্ট` section) — removed from Bills/Projects/
  Documents/Shopping tab headers.
- `renameCategory` (Finance), `renameGenericCategory` (Project), `renameDocCategory`,
  `renameShoppingList` all already propagate the rename across existing records
  (`APP_DATA.entries` / `.projects` / `.docs` / `.shopping`) and now also call
  `pushDataToBackend()`.

**What is NOT yet done — this batch's job:** verifying and hardening the
*ledger-level* correctness of category totals when a category changes, is
deleted, or an individual entry's category is edited — across every tab, not
just Finance — and confirming the numbers Google Sheets reports match what
each tab displays.

---

## 2. THE PROBLEM (in the client's own words, translated)

> "টাস্ক, প্রজেক্ট, শপিং লিস্ট, ডকুমেন্ট, বিল এবং এন্ট্রি — প্রত্যেকটি ট্যাবের
> ইনফরমেশন এবং ক্যাটাগরির মাঝে যেন সঠিকভাবে সিঙ্ক থাকে। কোনো এন্ট্রির ক্যাটাগরি
> চেঞ্জ করলে বা নতুন করে সেভ করলে তা পূর্বের ক্যাটাগরি থেকে বিয়োগ হয়ে নতুন
> ক্যাটাগরিতে যোগ হবে — এই যোগ-বিয়োগ সঠিকভাবে, প্রফেশনালি হতে হবে, এবং সব
> জায়গায় সাথে সাথে সঠিকভাবে প্রদর্শিত হতে হবে। গুগল শিটের সাথেও ক্যাটাগরি
> অনুযায়ী সঠিকভাবে সিঙ্ক থাকতে হবে — একটি ক্যাটাগরি সিলেক্ট করে সেভ করলে সঠিক
> ক্যাটাগরিতেই সেভ হবে, এবং পরে সার্চ করলে সেই ক্যাটাগরির সব তথ্য পাওয়া যাবে।
> ব্যয়ের ক্যাটাগরি বিশ্লেষণ অংশে সঠিকভাবে যোগ-বিয়োগ হয়ে আপডেট হবে।"

In plain terms: this is **not** asking for a running-balance ledger data
structure per category (the app doesn't have one — every chart/total is
computed live from `APP_DATA.entries` etc. on every render, which is
actually the *correct*, bug-resistant design — there's no separate counter
that can drift out of sync). What actually needs auditing is:

1. When an entry's category is **edited** (via `editEntry()` → Entry tab →
   re-save), does the amount get correctly excluded from the old category's
   totals and included in the new category's totals **everywhere it's
   displayed** — Finance Overview banners, ব্যয়ের ক্যাটাগরি donut chart,
   6-month trend chart, budget gauges, Monthly History tab, dashboard stat
   cards?
2. Same question for Bills (`billCategory`), Projects, Shopping, Documents —
   whichever of these have a "total/count by category" display anywhere.
3. When a category itself is **renamed**, are ALL of the above re-rendered
   immediately (not just on next page load)?
4. When a category is **deleted**, what happens to existing records that
   still reference it? (Currently: nothing — they keep the old category
   string, which then shows as an "orphan" category in filters/dropdowns.
   Decide and implement a deliberate policy — see Section 4.)
5. Does **search** (Finance search box, and any other tab's search) correctly
   find records by their *current* category after a rename?
6. Confirm the Google Sheets rows themselves get updated (not just local
   state) when a category is renamed — i.e. `pushDataToBackend()` actually
   pushes the *entries* with their updated category strings, not just the
   category list.

---

## 3. SCOPE — TABS TO AUDIT

For **each** of these, trace the full lifecycle (add entry → edit its
category → rename the category → delete the category) and confirm correct
behavior at every display surface:

- **Entry / Finance** (`APP_DATA.entries`, type=expense/income/savings)
- **Bills** (`APP_DATA.bills`, `CONFIG.billCategories`)
- **Home Projects** (`APP_DATA.projects`, `CONFIG.projectCategories`)
- **Documents** (`APP_DATA.docs`, `CONFIG.docCategories`)
- **Shopping** (`APP_DATA.shopping`, `CONFIG.shoppingLists`) — note: this is
  "list" not "category" but functions the same way for this audit
- **Tasks** — currently `tmCategory`/task category is a **hardcoded static
  dropdown**, not a `CONFIG`-driven manageable list at all. Decide: should
  Tasks get a real manageable category system like the others (consistent
  with the rest of the app), or is it intentionally out of scope? If in
  scope, build it the same way Project/Document categories work (Settings-
  only management, `CONFIG.taskCategories` array, synced via
  `buildCategoriesConfigJson()`/`applyCategoriesConfigFromBackend()`).

---

## 4. SPECIFIC DECISIONS NEEDED (ask the client via `ask_user_input_v0`,
   clickable options only — do not ask open-ended questions)

1. **Category delete policy** — when a category with existing records is
   deleted, should the app:
   (a) Block deletion and show which records use it, or
   (b) Reassign those records to an "অন্যান্য" (Other) fallback category, or
   (c) Allow deletion and leave records with the orphaned string (current
       behavior — just make sure it doesn't crash any dropdown/filter)?
2. **Tasks category system** — build it out fully (Settings-managed, synced)
   to match Projects/Documents, or leave Tasks' category as-is (static list)?
3. **Search scope** — should Finance search also match by amount or date,
   or stay category/note/member only (current behavior)?

---

## 5. VALIDATION REQUIREMENTS (must pass before delivery)

Follow the project's established validation pattern:
- `node --check` on every extracted `<script>` block and on `Code.gs`
  (copy to `.js` first since `node --check` doesn't recognize `.gs`).
- Python brace/`<div>` balance check on the full `index.html`.
- **Playwright, multi-viewport (375px/768px/1280px)**, scripted end-to-end:
  1. Add an expense entry under category A.
  2. Confirm category A's total increases in: Finance banner, donut chart
     legend value, budget gauge %, dashboard stat card.
  3. Edit that entry, change its category to B.
  4. Confirm category A's total decreases back down and category B's total
     increases, **in the same four places**, without a page reload.
  5. Rename category B to B2 in Settings.
  6. Confirm the entry now shows category B2 everywhere (Finance table,
     donut legend, search results for "B2"), and searching for the old
     name "B" returns nothing.
  7. Repeat steps 1–4 (add + edit-category) for at least one entry each in
     Bills, Projects, Documents, Shopping (and Tasks if in scope per
     Section 4.2).
  8. Take screenshots at each of the 3 viewport widths for the Finance tab
     and one other affected tab, attach them to your delivery summary.

---

## 6. DELIVERY

- Complete corrected `index.html` and `Code.gs` (full files, not diffs),
  downloadable.
- One-paragraph Bangla summary of what changed + which of the Section 4
  decisions were assumed (state assumptions explicitly if the client
  doesn't answer the `ask_user_input_v0` prompt) + GitHub Desktop commit
  message suggestion.
