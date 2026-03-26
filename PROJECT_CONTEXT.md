Veterinary Prescription Hub – Project Context

📍 Project State

This project is a cloud-based veterinary prescription system designed to prevent fraud, duplication, and unsafe dispensing by using a centralised, auditable dispensing ledger.

The system has moved from a claim/lock model to a multi-pharmacy dispensing model.

Core principle:
	•	no pharmacy ownership / no locking
	•	multiple pharmacies can dispense against the same prescription
	•	all dispensing is tracked as append-only events

⸻

✅ Current Prescription Status Model

Allowed values in prescriptions.status:
	•	ISSUED
	•	PARTIALLY_DISPENSED
	•	FULLY_DISPENSED
	•	EXPIRED
	•	VOIDED

Notes:
	•	CLAIMED has been retired from the live model
	•	DISPENSED replaced with FULLY_DISPENSED

⸻

✅ Current Item Status Model

Allowed values in prescription_items.status:
	•	ISSUED
	•	PARTIALLY_DISPENSED
	•	FULLY_DISPENSED
	•	VOIDED
	•	EXPIRED

⸻

🧾 Key Tables

prescriptions

Important columns now include:
	•	id
	•	rx_code
	•	status
	•	issued_by
	•	issued_at
	•	created_at
	•	expires_at
	•	practice_id
	•	prescriber_id
	•	validity_mode
	•	validity_days
	•	is_controlled_drug

Completion fields:
	•	fully_dispensed_by
	•	fully_dispensed_by_pharmacy_id
	•	fully_dispensed_at

Original attachment fields
	•	attachment_path
	•	attachment_mime_type
	•	attachment_uploaded_at

Preview attachment fields
	•	preview_attachment_path
	•	preview_attachment_mime_type
	•	preview_attachment_uploaded_at

Dispense attachment fields
	•	dispense_attachment_path
	•	dispense_attachment_mime_type
	•	dispense_attachment_uploaded_at

⸻

prescription_items
	•	id
	•	prescription_id
	•	line_number
	•	drug_description
	•	quantity_prescribed
	•	quantity_dispensed
	•	quantity_remaining
	•	status

⸻

prescription_item_dispenses

Each row = one dispense event
	•	prescription_item_id
	•	prescription_id
	•	pharmacy_id
	•	quantity_dispensed
	•	dispensed_at
	•	dispense_session_id
	•	billable

⸻

dispense_sessions

Logical grouping of actions
	•	id
	•	prescription_id
	•	pharmacy_id
	•	created_at
	•	billable
	•	session_type
	•	FULL
	•	PARTIAL_START
	•	PARTIAL_CONTINUE

⸻

dispense_audit

Renamed from claims_audit

Used for:
	•	auth failures
	•	preview readiness / checks
	•	full dispense outcomes
	•	partial dispense events
	•	non-billable document unlock events

Important result values currently allowed:
	•	AUTH_FAILED
	•	CHECK_READY
	•	CHECK_VOIDED
	•	FULL_DISPENSE_ALREADY_COMPLETED
	•	FULL_DISPENSE_NOT_FOUND
	•	FULL_DISPENSE_SUCCESS
	•	FULL_DISPENSE_VOIDED
	•	FULL_DISPENSE_NOT_ALLOWED_ON_ITEMISED
	•	FULL_DISPENSE_EXPIRED
	•	FULL_DISPENSE_RACE_LOST
	•	PARTIAL_DISPENSE_START
	•	PARTIAL_DISPENSE_CONTINUE
	•	DISPENSE_ATTACHMENT_UNLOCKED

⸻

🔐 Authentication Model

Pharmacies authenticate using API keys.
	•	keys stored as SHA256 hashes
	•	raw key only known at creation
	•	API key resolves pharmacy identity
	•	used for:
	•	authorisation
	•	audit attribution
	•	billing attribution

⸻

⚙️ Current Core RPC / DB Functions

get_prescription_state_with_key

Returns:
	•	mode: unitemised / itemised
	•	status
	•	expiry
	•	items if itemised

No audit-writing changes needed here.

⸻

full_dispense_prescription_with_key

Current rules:
	•	only works for unitemised prescriptions
	•	if prescription is already itemised, returns:
	•	P3014
	•	Full dispense is only allowed for unitemised prescriptions

Uses:
	•	FULL_DISPENSE_* audit terminology
	•	no CLAIM_* terminology

⸻

start_partial_dispense_with_key
	•	itemises an unitemised prescription
	•	creates item rows
	•	creates dispense records
	•	sets prescription to:
	•	PARTIALLY_DISPENSED
	•	or FULLY_DISPENSED

⸻

continue_partial_dispense_with_key
	•	records further dispensing against remaining quantities
	•	allows different pharmacies to continue dispensing
	•	blocks over-dispensing

⸻

check_prescription_with_key

Cleaned to remove claim terminology.
Uses:
	•	CHECK_READY
	•	FULL_DISPENSE_ALREADY_COMPLETED
	•	etc.

⸻

get_prescription_preview_attachment_with_key

Returns the preview PDF path/mime type.
Used for:
	•	pharmacy preview screen
	•	owner preview in future

Checks:
	•	valid pharmacy API key
	•	prescription exists
	•	not expired
	•	not voided
	•	preview_attachment_path is present

⸻

get_prescription_dispense_attachment_with_key

Returns the dispense PDF path/mime type.
Used when:
	•	Start Partial Dispense
	•	Continue Partial Dispense
	•	Dispense All Remaining
	•	Full Dispense (after successful completion)

Checks:
	•	valid pharmacy API key
	•	prescription exists
	•	not expired
	•	not voided
	•	dispense_attachment_path is present

Also writes a non-billable audit event:
	•	DISPENSE_ATTACHMENT_UNLOCKED
	•	billable = false

This means un-watermarked dispense PDF access is now trackable even if pharmacy later clicks cancel.

⸻

attach_preview_attachment

Sets:
	•	preview_attachment_path
	•	preview_attachment_mime_type
	•	preview_attachment_uploaded_at

⸻

attach_dispense_attachment

Sets:
	•	dispense_attachment_path
	•	dispense_attachment_mime_type
	•	dispense_attachment_uploaded_at

⸻

issue_prescription

Old overloaded versions existed. Legacy one using expires_at directly was dropped.

Current intended upload model:
	•	every prescription gets an RX code at creation
	•	no patient name or drug summary required for v1
	•	practice + prescriber required
	•	validity comes from practice default unless overridden
	•	controlled drug flag stored
	•	upload remains unitemised

Practice defaults:
	•	practices.default_validity_days added
	•	constrained to 1–180

Current behaviour:
	•	if practice default used → validity_mode = 'PRESET'
	•	if uploader overrides → validity_mode = 'CUSTOM'

⸻

🖥️ Current Pharmacy Test Page Behaviour

Preview
	•	shows prescription summary
	•	shows watermarked preview PDF
	•	uses get_prescription_preview_attachment_with_key

Unitemised prescription actions
	•	Dispense Full Prescription
	•	Start Partial Dispense

Itemised prescription actions
	•	Continue Partial Dispense
	•	Dispense All Remaining Items

Dispense workflows
	•	now show dispense PDF
	•	uses get_prescription_dispense_attachment_with_key

Cancel behaviour

This was fixed:
	•	when user cancels after opening a dispense workflow:
	•	partial form disappears
	•	unlocked dispense PDF disappears
	•	preview PDF disappears
This prevents lingering access after cancel.

⸻

✅ Current Document Architecture

This is the key new design:

1. Original upload

Stored unchanged for record:
	•	PDF stays PDF
	•	image stays image

2. Preview copy

Always a watermarked PDF
Used for:
	•	pharmacy preview
	•	owner preview

3. Dispense copy

Always a clean PDF
Used for:
	•	all pharmacy dispense workflows

Reason:
	•	avoids device/file-format compatibility issues
	•	pharmacies always receive a PDF
	•	original remains preserved for record

⸻

✅ Watermark Design Choice

Chosen watermark design:
	•	red
	•	repeated
	•	diagonal
	•	obvious
	•	includes RX code

Current implementation works, but watermark needs refinement:
	•	current version is too dense
	•	words overlap and become illegible
	•	watermark should cover more of the page without overlapping

Next planned code change

Replace the current applyPreviewWatermark function in server.js with a version that:
	•	uses shorter repeated stamps:
	•	PREVIEW ONLY
	•	NOT VALID FOR DISPENSING
	•	RX-<code>
	•	spaces them more cleanly
	•	keeps strong red coverage without overlap

The replacement function was already drafted in chat, but not yet applied/tested.

⸻

✅ Processing Service Prototype

A local Node-based processing service has been built in folder:
	•	prescription-processor

Files created
	•	package.json
	•	.env
	•	server.js

Installed dependencies
	•	@supabase/supabase-js
	•	dotenv
	•	express
	•	pdf-lib
	•	sharp

Current service behaviour

Endpoint:
	•	POST /process-prescription-attachments

Input:
	•	rx_code
	•	original_path
	•	original_mime_type

It:
	1.	loads prescription by RX code
	2.	downloads original file from storage
	3.	if original is PDF:
	•	keeps/re-saves as clean PDF
	4.	if original is image:
	•	converts image into PDF
	5.	creates:
	•	dispense.pdf
	•	preview.pdf
	6.	uploads both to:
	•	<prescription-id>/dispense.pdf
	•	<prescription-id>/preview.pdf
	7.	copies original to canonical:
	•	<prescription-id>/original.<ext>
	8.	updates prescription DB row fields automatically

Canonical storage layout now in use

Within prescription-attachments bucket:
	•	<prescription-id>/original.<ext>
	•	<prescription-id>/dispense.pdf
	•	<prescription-id>/preview.pdf

⸻

✅ Verified Working Example

For test prescription RX-613fc162, processor successfully produced:
	•	cb877bb6-4625-4f3c-8113-6ea47caa2aa2/original.pdf
	•	cb877bb6-4625-4f3c-8113-6ea47caa2aa2/dispense.pdf
	•	cb877bb6-4625-4f3c-8113-6ea47caa2aa2/preview.pdf

And pharmacy page correctly showed:
	•	Preview → generated preview.pdf
	•	Start Partial → generated dispense.pdf

This is a major milestone.

⸻

✅ Storage / Format Decisions

Allowed original uploads should ultimately include:
	•	PDF
	•	JPG / JPEG
	•	PNG
	•	HEIC / HEIF if feasible

System policy:
	•	store original unchanged
	•	always generate preview as PDF
	•	always generate dispense copy as PDF

This avoids pharmacy device compatibility problems.

⸻

🚧 Next Planned Steps

Immediate next step

Apply the improved non-overlapping watermark function in server.js, restart processor, and re-process a test prescription.

Then

Wire the processing service into the practice upload flow so that after upload:
	1.	original file uploaded
	2.	processor called automatically
	3.	preview and dispense PDFs created automatically
	4.	prescription row updated automatically

After that

Potential future enhancements:
	•	owner-facing preview page
	•	stronger anti-forgery watermark pattern
	•	OCR / AI extraction from uploads
	•	queue/retry system for document processing
	•	production deployment of processor service

⸻

🧩 Notes for Future Sessions

Important reminders:
	•	do not reintroduce claim terminology
	•	dispense_audit is the live audit table name
	•	preview and dispense are now deliberately split
	•	original file should not be shown directly to pharmacies in normal workflow
	•	if a bug appears showing wrong document, check browser cache / stale HTML first
	•	full HTML rewrites are often preferable to piecemeal frontend edits for this project

⸻

📌 Summary

This system now functions as:

A multi-pharmacy, centralised, auditable prescription dispensing ledger with a three-layer document model:
original record copy, watermarked preview PDF, and clean dispense PDF.

The core dispensing/document architecture is now working end-to-end.
:::
