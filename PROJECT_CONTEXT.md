# 🧾 Veterinary Prescription Hub – PROJECT_CONTEXT.md

## 🔄 Last Updated
Date: 2026-03-30

---

## ✅ Current System State

### 🧱 Core Architecture
- Supabase (Postgres + Storage) is the source of truth
- Local Node.js processing service (server.js) runs on:
  - http://localhost:3000
- No Edge Functions in use
- Processing is triggered manually via HTTP POST (curl)

---

## 📦 Attachment Processing Pipeline (WORKING ✅)

### Flow
1. Upload original file to Supabase Storage (e.g. test-prescription-X.pdf)
2. Create prescription via:
   issue_prescription(...)
3. Manually trigger processor:

   POST /process-prescription-attachments
   Authorization: Bearer PROCESSOR_SECRET

4. Processor performs:
   - Download original file
   - Convert to clean PDF (if needed)
   - Generate:
     - dispense.pdf (clean)
     - preview.pdf (watermarked)
   - Upload both to:
     {prescription_id}/dispense.pdf
     {prescription_id}/preview.pdf
   - Update DB fields:
     - attachment_path → canonical original
     - preview_attachment_path
     - dispense_attachment_path

---

## 🖨️ Watermark System (FINAL VERSION ✅)

### Style
- Horizontal (not diagonal)
- Dense, edge-to-edge coverage
- Repeating phrase:
  RX_CODE - PREVIEW ONLY - NOT VALID FOR DISPENSING
- Brick-layer (staggered) pattern applied

### Parameters
- fontSize = 12
- opacity = 0.16
- rowGap = 22
- offsetAmount = unitWidth / 2

### Key Implementation Detail
Offset is based on unit text width, not full repeated line:
const unitWidth = font.widthOfTextAtSize(unitText + separator, fontSize);
const offsetAmount = unitWidth / 2;

### Result
- Strong anti-copy visual protection
- Even page coverage
- Professional “security paper” appearance

---

## 🧠 Key Lessons Learned

### ❌ What was wrong before
- Watermark logic was fine
- But processor was NOT being triggered
- Led to:
  - no preview generation
  - no terminal activity
  - null DB fields

### ✅ Resolution
- Manual curl call confirmed pipeline works
- System architecture validated
- Issue identified as missing trigger, not processing logic

---

## ⚠️ Current Limitation

### 🚧 Processing is NOT automatic
- Requires manual POST call to processor
- No trigger on:
  - prescription creation
  - file upload

---

## 🎯 Next Step (HIGH PRIORITY)

### Automate processing trigger

Goal:
When a prescription is issued + file uploaded → processor runs automatically

### Options (to decide next session)
1. Add call from issuing workflow (preferred)
2. Add Supabase Edge Function trigger
3. Add DB webhook / function trigger
4. Keep manual trigger (not viable long-term)

---

## 🔐 Environment Variables

Local .env:

SUPABASE_URL=...
SUPABASE_SERVICE_ROLE_KEY=...
PROCESSOR_SECRET=my-super-long-random-processor-secret-2026
BUCKET_NAME=prescription-attachments
PORT=3000

---

## 🧪 Testing Workflow (CURRENT)

1. Upload file (e.g. test-prescription-3.pdf)
2. Run issue_prescription(...)
3. Copy rx_code
4. Run curl:

curl -X POST http://localhost:3000/process-prescription-attachments \
  -H "Authorization: Bearer PROCESSOR_SECRET" \
  -H "Content-Type: application/json" \
  -d '{ ... }'

5. Confirm:
   - preview + dispense paths populated
6. Test via HTML Preview button

---

## 📌 System Status Summary

| Component                  | Status |
|--------------------------|--------|
| Prescription creation     | ✅ Working |
| File upload               | ✅ Working |
| Processing server         | ✅ Working |
| Watermark generation      | ✅ Finalised |
| Preview rendering         | ✅ Working |
| Automatic processing      | ❌ Not implemented |

---

## 🚀 Position

You now have:
- A fully functioning document pipeline
- A production-grade watermark system
- A clean separation of concerns

Next session will focus on:
making it automatic and production-ready

---

## 🧭 Resume Point for Next Session

“Let’s wire the processor into the prescription issue/upload flow so previews are generated automatically.”
