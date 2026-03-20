# 🧠 Project: Veterinary Prescription Hub (Updated State – End of Session)

## 🔹 Overview
Cloud-based veterinary prescription system designed to:
- Issue prescriptions
- Allow pharmacies to verify and redeem prescriptions
- Prevent duplication and fraud
- Maintain a full audit trail
- Support pharmacy-triggered partial dispensing
- Preserve low-friction workflows for both practices and pharmacies

Core flow:
Issue → Check → Claim (simple full-redemption path) → Audit → Flag → Void/Supersede

or

Issue → Start Partial Dispense → Continue Partial Dispense → Get State → Audit

---

## 🧠 Product Philosophy (CRITICAL)

> Frictionless by default, structured only when needed.

- No unnecessary data entry for practices
- No unnecessary friction for pharmacies
- Prescriptions remain unstructured unless partial dispensing begins
- Once partial dispensing begins, the entire prescription must be itemised
- Controlled drugs may be flagged, but should not have mandatory extra workflow enforced by the hub at this stage

---

## 🗄️ Core Tables

- prescriptions
- prescription_items
- prescription_item_dispenses
- dispense_sessions ✅ NEW
- prescription_claims (currently underused / likely redundant)
- claims_audit
- pharmacies
- pharmacy_api_keys
- practices
- prescribers
- prescription_flags

---

## 🧩 Key Table Notes

### prescriptions
- id (uuid, PK)
- rx_code (text, UNIQUE)
- status (ISSUED / CLAIMED / DISPENSED / EXPIRED)
- claimed_by
- claimed_by_pharmacy_id (uuid FK → pharmacies.id) ✅
- claimed_at
- issued_by
- patient_name
- drug_summary (unstructured)
- issued_at ✅
- validity_mode ✅
- validity_days ✅
- expires_at (trigger-derived)
- supersedes_id
- voided_at / void_reason
- is_controlled_drug (planned / should exist if added)
- attachment fields

### prescription_items
Used only once partial dispensing begins.

- id
- prescription_id
- line_number
- drug_description
- quantity_prescribed
- quantity_dispensed
- quantity_remaining
- status (ISSUED / PARTIALLY_DISPENSED / DISPENSED)

### prescription_item_dispenses
Ledger rows for each item dispense event.

- id
- prescription_item_id
- prescription_id
- pharmacy_id
- quantity_dispensed
- dispensed_at
- created_at
- billable
- dispense_session_id ✅ NEW FK to dispense_sessions.id

### dispense_sessions ✅ NEW
Groups multiple dispense rows that belong to the same pharmacy interaction.

- id
- prescription_id
- pharmacy_id
- created_at

Purpose:
- allows multiple item dispense rows from one pharmacy interaction to be treated as one future billing unit if desired
- preserves flexibility for later billing design

### claims_audit
Append-only audit log.

Includes:
- prescription_id
- pharmacy_id
- rx_code
- result
- errcode
- message
- billable
- created_at (should exist / recommended)

Audit result constraint was broadened to allow:
- CHECK_*
- CLAIM_*
- DISPENSE_*
- PARTIAL_DISPENSE_*

---

## ⚙️ Functions (Current State)

### issue_prescription ✅ FIXED
- sets issued_at
- sets validity_mode
- sets validity_days
- no longer accepts expires_at directly
- expiry derived by trigger

### check_prescription_with_key ✅
- validates pharmacy API key
- checks prescription existence / expiry / claimability
- logs CHECK_* audit events

### claim_prescription_with_key ✅
Current simple full-redemption path.
- validates API key
- atomically claims prescription
- sets claimed_by, claimed_by_pharmacy_id, claimed_at
- logs CLAIM_SUCCESS
- currently serves as practical “full dispense” path, though naming may later be cleaned up

### create_pharmacy_api_key ✅
- standardised SHA256 hashing:
  `extensions.digest(convert_to(v_key, 'utf8'), 'sha256')`
- returns raw key once
- stores only hashed key

### flag_prescription_issue_with_key ✅
- hashing aligned
- writes flags correctly

### void_and_supersede_prescription ✅
- corrected audit codes:
  - CLAIM_SUCCESS
  - CLAIM_VOIDED
- old prescription voided
- new prescription links via supersedes_id
- prior CLAIM_SUCCESS rows made non-billable

### start_partial_dispense_with_key ✅ NEW
Pharmacy-triggered start of ledger mode.

- validates API key
- validates prescription
- accepts full item list as JSON
- creates all `prescription_items`
- creates first `prescription_item_dispenses` rows
- updates parent prescription status
- writes `PARTIAL_DISPENSE_START` audit row

Rule enforced:
> If partial dispensing begins, the entire prescription must be itemised.

Current note:
- function works
- not yet updated to populate `dispense_session_id`
- next session should update it to create one `dispense_sessions` row and apply that session id to all dispense rows created in the function

### continue_partial_dispense_with_key ✅ NEW
Used after ledger mode already exists.

- validates API key
- validates prescription/item
- enforces remaining quantity
- inserts additional dispense ledger row
- updates item totals
- updates parent prescription status
- writes `PARTIAL_DISPENSE_CONTINUE` audit row

Current note:
- function works
- not yet updated to populate `dispense_session_id`
- next session should update it to create a new `dispense_sessions` row for each continuation interaction and attach that id to the new dispense row

### get_prescription_state_with_key ✅ NEW
Critical pharmacy-facing state retrieval function.

Returns:

#### If not itemised yet:
- mode = unstructured
- status
- expires_at
- drug_summary

#### If itemised:
- mode = structured
- status
- expires_at
- full list of items with:
  - drug_description
  - quantity_prescribed
  - quantity_dispensed
  - quantity_remaining
  - status

This is the key function that allows later pharmacies to safely see what remains.

---

## ❌ Deprecated / Remove

### dispense_prescription_partial_with_key
- deprecated
- not aligned with final design
- assumed pre-existing items
- should be dropped if still present

Recommended drop:
```sql
drop function if exists public.dispense_prescription_partial_with_key(text, text, uuid, numeric);
Use this as the source of truth for the current Prescription Hub state. Do not assume anything else unless I provide code.
