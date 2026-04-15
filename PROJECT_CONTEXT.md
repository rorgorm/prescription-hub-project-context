PROJECT CONTEXT — Veterinary Prescription Hub
Last updated: 2026-04-15

⸻

CURRENT SYSTEM STATE (STABLE)

AUTHENTICATION
	•	Supabase JWT-based auth only
	•	No API keys
	•	Identity:
	•	auth.uid() = practices.id
	•	auth.uid() = pharmacies.id
	•	Session isolation:
	•	Practice: vet-hub-practice-auth
	•	Pharmacy: vet-hub-pharmacy-auth

⸻

PRACTICE UI (practice.html)

Working Features
	•	Login/logout stable
	•	Prescribers load correctly
	•	Upload → Issue → Process flow working
	•	Reference editable post-issue
	•	Replace + Void flows working
	•	Drag/drop upload working
	•	Prescription list loads and filters correctly

Owner Messaging (After Issue)
	•	Owner link generated
	•	Owner message generated
	•	Copy buttons:
	•	Copy owner message
	•	Copy owner link

Owner Messaging (Persistent in Log)
	•	Share dropdown now includes:
	•	View owner copy (PDF preview)
	•	Copy owner link
	•	Copy owner message

Known UX Tweak (Next Task)
	•	Move “Copy Rx code” into Share dropdown

⸻

PHARMACY UI (COMPLETE)
	•	JWT login working
	•	Preview (watermarked)
	•	Full dispense
	•	Partial dispense:
	•	Start
	•	Continue
	•	Complete remaining
	•	Final document unlock

⸻

OWNER UI (owner.html)

Access
	•	No login required
	•	Via Rx input OR URL param:
owner.html?rx=RX-XXXX

Displays
	•	Status badge (colour coded)
	•	Issue date
	•	Expiry date
	•	Watermarked preview PDF
	•	Itemised dispensing (only if partial exists)

Enhancements
	•	Auto-load from URL param
	•	URL updates on manual entry
	•	Copy Rx code button

⸻

STATUS SYSTEM

ISSUED → Green → Ready to dispense
PARTIALLY_DISPENSED → Amber → Partially dispensed
FULLY_DISPENSED → Grey → Fully dispensed
EXPIRED → Red → Invalid
VOIDED → Red → Cancelled

⸻

OWNER MESSAGING (FINAL WORDING)

ISSUED
This is a view-only copy.
Pharmacies must use your unique Rx code to retrieve the original prescription before supplying medication.

PARTIALLY DISPENSED
This prescription has been partially dispensed.
A pharmacy must verify the remaining supply using your Rx code.

FULLY DISPENSED
This prescription has been fully dispensed.
No further medication can be supplied.

EXPIRED / VOIDED
This prescription is no longer valid.
Please contact your veterinary practice.

⸻

CORE ARCHITECTURE
	•	Supabase (Postgres + Auth + Storage)
	•	JWT-only identity model
	•	Signed URLs for file access
	•	Node processing service (Railway)
	•	Preview vs dispense document separation
	•	Watermarking applied to preview

⸻

SECURITY MODEL
	•	No frontend trust
	•	No shared secrets
	•	JWT required for all privileged actions
	•	Signed URLs (time-limited)
	•	Preview always watermarked
	•	No download UI exposed
	•	Right-click disabled (soft deterrent)

⸻

REMOVED LEGACY
	•	pharmacy_api_keys table
	•	*_with_key functions
	•	Dual auth system

Result
	•	Cleaner logic
	•	Lower bug surface
	•	Easier scaling

⸻

RECENT FIXES
	•	Login failures due to JS errors
	•	Missing refreshPrescriptionsSilently
	•	Duplicate  bug
	•	Missing function bindings
	•	Dead UI elements
	•	Owner link auto-load fixed

⸻

CURRENT POSITION
	•	Fully working multi-role system
	•	Secure prescription lifecycle
	•	Owner-facing UI live
	•	GDPR-friendly (no client data stored)
	•	Production-ready foundation

⸻

NEXT SESSION START

Immediate task
	•	Move “Copy Rx code” into Share dropdown

Goal
	•	Reduce button clutter
	•	Group all owner-related actions together

⸻

DESIGN PRINCIPLES
	•	One auth model
	•	One identity source (JWT)
	•	No frontend trust
	•	Minimal UI noise
	•	Clear user state
	•	Real-world usability first

⸻

SUMMARY

You now have a fully functioning veterinary prescription system with:
	•	Practice workflow
	•	Pharmacy dispensing control
	•	Owner visibility layer
	•	Clean, scalable architecture

⸻

Next step when back:
Refine Share dropdown (include Rx code)
