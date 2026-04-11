Veterinary Prescription Hub – Unified JWT Auth System (Clean State)

🔄 Last Updated

Date: 2026-04-11

⸻

✅ Current Status (STABLE & WORKING)

Practice UI
	•	Supabase Auth login implemented
	•	JWT used for all backend API calls
	•	Practice identity derived from auth.uid()
	•	Login secured (no information leakage)
	•	No success message before role validation
	•	Logged-in email displayed
	•	Logout button working
	•	Enter/return key submits login
	•	Session persists correctly with isolated storage key
	•	Prescribers loaded directly from prescribers
	•	practice_prescribers no longer used

Pharmacy UI
	•	Supabase Auth login implemented
	•	JWT used for all RPC calls
	•	Pharmacy identity derived from auth.uid()
	•	UI gated behind login (loginCard → mainApp)
	•	Login hardened (no success flash, no role leakage)
	•	Logged-in email displayed
	•	Logout button working
	•	Enter/return key submits login
	•	Session persists correctly with isolated storage key

⸻

🧠 Core Architecture (FINAL MODEL)

System has fully transitioned from:
	•	API key authentication ❌
to:
	•	JWT identity-based authentication via Supabase Auth ✅

Unified rule:
	•	auth.uid() must match:
	•	practices.id for practice flows
	•	pharmacies.id for pharmacy flows

No frontend identity is trusted.

⸻

✅ Active JWT Functions (Pharmacy)
	•	get_prescription_state
	•	get_prescription_preview_attachment
	•	full_dispense_prescription
	•	get_prescription_dispense_attachment
	•	start_partial_dispense
	•	continue_partial_dispense

All:
	•	derive pharmacy from auth.uid()
	•	enforce pharmacies.is_active = true

⸻

✅ Practice Backend Model

Practice backend (Railway API):
	•	Authenticated via Bearer JWT
	•	Practice derived from token server-side
	•	No frontend practice_id or secrets used

Endpoints:
	•	issue-and-process
	•	practice-prescriptions
	•	void-prescription
	•	replace-attachment
	•	update-prescription-reference

⸻

✅ Prescription Flow (END-TO-END)

Practice:
	1.	Login
	2.	Upload prescription
	3.	Issue via backend
	4.	Optional reference added
	5.	Appears in prescription list

Pharmacy:
	1.	Login
	2.	Enter Rx code
	3.	Preview prescription (JWT)
	4.	View watermarked PDF
	5.	Choose:
	•	Full dispense
	•	Start partial dispense
	•	Continue partial dispense
	•	Dispense remaining items
	6.	Access unlocked attachment after dispense

⸻

✅ Partial Dispense System

prescription_items table

Columns:
	•	id
	•	prescription_id
	•	line_number (required)
	•	drug_description
	•	quantity_prescribed
	•	quantity_dispensed
	•	quantity_remaining
	•	status
	•	created_at

Behaviour:
	•	Start partial → creates item rows with line_number
	•	Continue partial → updates quantities
	•	Dispense remaining → completes all
	•	State reflects correctly in UI

⸻

✅ Session Isolation (CRITICAL FIX)

Each UI uses its own Supabase storage key:

Practice:
storageKey: “vet-hub-practice-auth”

Pharmacy:
storageKey: “vet-hub-pharmacy-auth”

Outcome:
	•	Practice and pharmacy sessions do NOT interfere
	•	Eliminates cross-login contamination bug

⸻

✅ Security Hardening

Login behaviour:
	•	No success message before role validation
	•	Generic failure message:
“Invalid email or password.”

Prevents:
	•	account enumeration
	•	role leakage

Role validation:
	•	Practice UI checks practices.id = auth.uid()
	•	Pharmacy UI checks pharmacies.id = auth.uid()

If mismatch:
	•	forced logout
	•	generic error shown

⸻

🧹 Legacy System (REMOVED)

Deleted:
	•	pharmacy_api_keys table
	•	ALL _with_key SQL functions

Result:
	•	Single authentication model
	•	No parallel systems
	•	Reduced bug surface area
	•	Cleaner architecture

⸻

🐛 Issues Encountered (ALL RESOLVED)

ACTIVE_PHARMACY_NOT_FOUND
	•	Cause: wrong user logged in
	•	Fix: role enforcement + session isolation

Cross-session login bug
	•	Cause: shared localStorage
	•	Fix: separate storageKey

Success flash on login
	•	Cause: message before role validation
	•	Fix: removed premature success

Invalid API key errors
	•	Cause: legacy functions
	•	Fix: full JWT migration

line_number null error
	•	Cause: not set during insert
	•	Fix: enforced numbering

Function overload conflict
	•	Cause: duplicate continue_partial_dispense
	•	Fix: removed integer version

Unwatermarked preview
	•	Cause: wrong RPC used
	•	Fix: corrected preview function

⸻

🧭 Current System State

The system is now:
	•	Fully JWT-authenticated
	•	Identity-driven (not key-based)
	•	Cleanly separated (practice vs pharmacy)
	•	End-to-end functional
	•	Free of legacy auth paths
	•	Stable and demo-ready

⸻

🔜 Next Logical Steps (OPTIONAL)
	1.	Role metadata (future improvement)
Store role in Supabase Auth user metadata
	2.	UI polish
	•	Display organisation name (not just email)
	•	Session expiry handling
	•	Mobile optimisation
	3.	Owner-facing flow
	•	Owner receives Rx code
	•	Owner views prescription summary
	•	Potential owner portal

⸻

🧩 Design Principles
	•	One authentication model
	•	One identity source (JWT)
	•	No frontend trust
	•	No parallel systems
	•	No dead code
	•	Clear audit trail

⸻

🏁 Summary

You now have:

A fully functioning, identity-secure, auditable veterinary prescription system
with real-world dispensing workflows and clean architecture.

This is a production-grade foundation ready for further expansion.

⸻

If you want next step, I’d strongly suggest:

👉 building the owner-facing experience — that’s where this becomes genuinely disruptive.
