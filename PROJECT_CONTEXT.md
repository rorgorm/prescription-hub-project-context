# 📄 Veterinary Prescription Hub – Project Context

## 🔄 Last Updated
Date: 2026-04-07

---

## 🚀 Current System State (Working)

### ✅ Authentication & Practice Model
- Supabase Auth implemented
- Login = **practice email + password**
- One login per practice (no individual staff accounts)
- JWT used for all backend API calls
- Backend derives `practice_id` from token (`req.practiceId`)
- No longer passing `practice_id` from frontend (correct design)

---

### ✅ Practice Profile
Each practice record contains:
- `id` (matches Supabase auth user id)
- `name`
- `address`
- `email`
- `default_validity_days` (currently set to 180)

---

### ✅ Prescriber System (Current State)
- Prescribers stored in `prescribers` table:
  - `id`
  - `practice_id`
  - `vet_name`
  - `rcvs_number`
  - `is_active`

- UI loads prescribers and populates dropdown at upload
- Selected `prescriber_id` passed to backend
- `issue_prescription` correctly validates:
  - prescriber belongs to practice
  - prescriber is active

---

### ⚠️ KNOWN ISSUE – PRESCRIBER DUPLICATION (CRITICAL TO FIX NEXT SESSION)

Current situation:
- Two parallel prescriber systems exist:
  1. `prescribers` table (correct, used by backend)
  2. `practice_prescribers` table (legacy UI layer)

- This caused duplication:
  - Required creating **two “Rory Gormley” entries**
  - One in each table

- A **temporary bridge workaround** was implemented:
  - UI loads from `practice_prescribers`
  - Uses `prescriber_id` field to map to `prescribers.id`

🚨 This is NOT acceptable long-term.

### 🔧 REQUIRED NEXT STEP (HIGH PRIORITY)
Remove duplication entirely and simplify:

- Eliminate `practice_prescribers` layer
- UI should load **directly from `prescribers`**
- One prescriber = one row = one source of truth
- No mapping / bridging / duplication

👉 Goal:
> Clean, single-table prescriber model with zero redundancy

---

### ✅ Prescription Flow (Working End-to-End)

#### Issue Flow
1. Upload file → Supabase Storage (`incoming/...`)
2. Call `/api/issue-and-process`
3. Backend:
   - Calls `issue_prescription`
   - Generates `rx_code`
   - Processes file:
     - clean PDF
     - preview PDF (watermarked)
4. Returns result to UI

#### Replace Attachment Flow
- Upload new file
- Reprocess same Rx code
- Regenerates preview + dispense PDFs
- Updates audit fields

#### Void Flow
- Marks prescription as `VOIDED`
- Stores `void_reason`
- Prevents further use

---

### ✅ UI Improvements (Latest Session)

#### Loading UX
- Buttons now show:
  - spinner
  - immediate feedback
  - disabled state

#### Issue Button Flow
- "Uploading..." during file upload
- "Processing..." during backend work
- Spinner stops immediately after success (no UI lock)

#### Replace Attachment Flow
- Same staged loading UX
- Includes 60s timeout protection

#### Timeout Protection
- `AbortController` added to API calls
- Prevents infinite spinner if backend hangs

---

### ✅ Security Model

#### Backend Middleware
- `requirePracticeAuth` validates JWT
- Sets:
  - `req.practiceUser`
  - `req.practiceId`

#### Removed
- ❌ `practice_id` from frontend payloads

#### Result
- Fully secure practice-scoped API

---

### 🧠 Key Architectural Decisions

- Practice = single account (not per-user system)
- Prescriber chosen at upload (not login-based)
- Backend = source of truth for:
  - identity
  - permissions
- Attachments:
  - stored in Supabase Storage
  - processed server-side (Node service)

---

### ⚙️ Backend (Node / Railway)

Handles:
- PDF generation (pdf-lib)
- Image → PDF conversion (sharp)
- Preview watermarking
- Storage management
- API endpoints:
  - issue-and-process
  - replace attachment
  - void
  - fetch prescriptions

---

### 📌 Current Status

✅ Login working  
✅ Logout working  
✅ Prescriber selection working  
✅ Issue flow working  
✅ Replace flow working  
✅ Void flow working  
✅ Prescription list + filters working  
✅ Reference saving working  
✅ Loading UX improved  

---

## 🔜 NEXT SESSION PRIORITIES

### 🔴 1. FIX PRESCRIBER ARCHITECTURE (TOP PRIORITY)
- Remove `practice_prescribers`
- Refactor UI to use `prescribers` directly
- Remove all bridging logic
- Ensure clean 1:1 data model

---

### 🟡 2. UX POLISH
- Add spinner to login button
- Optional: progress stages (Uploading → Processing → Done)

---

### 🟡 3. OPTIONAL IMPROVEMENTS
- Better error surfacing from backend
- File upload progress (nice-to-have)
- Cleaner success state UI

---

## 💡 DESIGN PRINCIPLE (IMPORTANT REMINDER)

Avoid:
> “Temporary fixes” or “bridging layers”

Aim for:
> Clean, single-source-of-truth data model  
> Minimal moving parts  
> Low-friction for real-world practice use  

---

## 🧭 Resume Point

Next session should begin with:

> **Refactoring prescriber system to remove duplication and eliminate practice_prescribers entirely**

---
