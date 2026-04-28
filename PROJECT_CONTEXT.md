PROJECT CONTEXT — Veterinary Prescription Hub
Last updated: 2026-04-28

⸻

CURRENT SYSTEM STATE (PRODUCTION-STABLE)

AUTHENTICATION
• Supabase JWT-based auth only
• No API keys
• Identity:
  • auth.uid() = practices.id
  • auth.uid() = pharmacies.id
• Session isolation:
  • Practice: vet-hub-practice-auth
  • Pharmacy: vet-hub-pharmacy-auth

⸻

CORE WORKFLOWS

PRACTICE
• Upload → Issue → Process working
• Reference editable post-issue
• Replace attachment working
• Void working
• Owner messaging generated (link + message)
• Prescription list loads with filters

PHARMACY
• JWT login working
• Preview (watermarked)
• Full dispense
• Partial dispense (start / continue / complete)
• Attachment unlock logic working

OWNER
• No login required
• Access via:
  owner.html?rx=RX-XXXX
• Displays:
  • Status badge
  • Issue + expiry
  • Preview PDF
  • Partial dispense breakdown (if exists)

⸻

NEW FEATURE — PRESCRIPTION QUERY WORKFLOW (V1 COMPLETE)

PURPOSE
• Replace phone/email friction
• Allow pharmacy → practice communication inside system
• Maintain clean audit trail
• Improve pharmacy-side value (key monetisation driver)

⸻

DATABASE MODEL

TABLE: prescription_queries

Fields:
• id (uuid, PK)
• prescription_id (uuid, FK → prescriptions.id)
• raised_by_pharmacy_id (uuid)
• query_type (enum)
• query_message (text)
• practice_response (text)
• status (enum)
• created_at (timestamp)
• resolved_at (timestamp)

ENUM: query_status
• open
• responded
• resolved_by_dispense

ENUM: query_type
• incorrect_attachment
• illegible
• missing_information
• quantity_query
• substitution_query
• controlled_drug_concern
• other

⸻

BACKEND FUNCTIONS

• raise_prescription_query(p_rx_code, p_query_type, p_query_message)
• get_prescription_query_for_pharmacy(p_rx_code)
• get_prescription_query_for_practice(p_rx_code)
• respond_to_prescription_query(p_query_id, p_response)

IMPORTANT
• Only ONE version of respond_to_prescription_query exists:
  (uuid, text)
• Old overloaded function removed (critical fix)

⸻

QUERY STATE LOGIC

Pharmacy raises query:
→ status = open

Practice replies:
→ status = responded

Pharmacy dispenses (full or partial):
→ status automatically set to resolved_by_dispense (SQL trigger logic)

NO manual “resolve” button exists
→ resolution is implicit via dispensing (correct design decision)

⸻

PHARMACY UI

• “Raise Prescription Query” button
• Query panel:
  • type dropdown
  • optional message
• Active query display:
  • status
  • pharmacy message
  • practice response

Behaviour:
• Query persists until dispense
• No duplicate queries allowed per prescription

⸻

PRACTICE UI

Per prescription:
• Query panel appears if query exists

Displays:
• Status
• Query type
• Pharmacy message
• Practice response (if exists)

Response logic:

STATE 1 — no response
• textarea + “Send reply”

STATE 2 — responded
• textarea hidden
• message:
  “Reply sent. Awaiting pharmacy review.”

STATE 3 — resolved_by_dispense
• message:
  “This query has been resolved.”

CRITICAL UX DECISION
• No “resolve” button for practice
• Practice is low-friction party
• They either:
  • reply
  • replace attachment
  • do nothing

⸻

OWNER UI (UPDATED LOGIC)

If active query exists (status = open OR responded):
→ show message:

“Your order is currently on hold while the pharmacy checks a prescription query with your veterinary practice. Please wait for this to be resolved.”

If resolved_by_dispense:
→ normal status messaging resumes

IMPORTANT
• Owner sees NO query detail
• Only status abstraction (correct for UX + liability)

⸻

SECURITY MODEL

• No frontend trust
• JWT required for all actions
• Signed URLs for attachments
• Preview always watermarked
• No raw file exposure
• Query system tied to authenticated roles

⸻

RECENT FIXES

• refreshPrescriptionsSilently missing reference bug
• dropdown duplication / toggle issues
• query panel wiring (practice + pharmacy)
• RPC function overloading conflict (PGRST203)
• submitQueryResponse now correctly exposed via window
• dropdown auto-close UX polish
• post-response textarea removal (UX clarity)

⸻

CURRENT POSITION

You now have:

• Full prescription lifecycle
• Multi-role system (practice / pharmacy / owner)
• Secure document handling
• Partial dispensing ledger
• Integrated communication layer (query workflow)
• Clean resolution model (implicit via dispense)

→ This is now a genuinely differentiated product

⸻

NEXT SESSION OPTIONS

1. Notifications (HIGH VALUE)
   • Email trigger on:
     • query raised
     • practice reply
     • attachment replaced

2. Practice-side visual flags
   • Highlight prescriptions with active queries

3. Pharmacy-side friction reduction
   • Prevent dispense while query = open (optional rule)

4. Audit / billing hooks
   • Track queries per pharmacy (future monetisation)

5. UI polish
   • Inline status badge for queries in list view

⸻

DESIGN PRINCIPLES (UNCHANGED)

• One auth model (JWT)
• No frontend trust
• Minimal friction for practices
• Pharmacy = value capture layer
• Owner = simple, abstracted UX
• Real-world workflow > theoretical purity

⸻

SUMMARY

System is now:
• Functional
• Secure
• Scalable
• Commercially viable

Prescription Query Workflow successfully adds:
→ real operational value
→ clear monetisation pathway
→ reduced friction across all parties

⸻

NEXT STEP WHEN BACK

→ Notifications (this unlocks real-world usability)
