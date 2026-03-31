Project Context – Prescription Processing System (MVP)

Last Updated

2026-03-31

Overview

This project is a minimal end-to-end prescription processing system designed to demonstrate:
	•	Practice-side prescription issuing
	•	Backend processing and validation
	•	Pharmacy-side preview and dispense flow

The system is intentionally simple, with minimal infrastructure and no reliance on local tools or manual database interaction.

Architecture

Frontend (Practice)
	•	File: practice.html
	•	Runs locally in browser
	•	Allows user to:
	•	Select prescriber
	•	Select validity period
	•	Mark as controlled drug
	•	Upload prescription PDF
	•	Submit in a single action

Backend
	•	Node.js (Express) server hosted on Railway
	•	Main endpoints:
	•	POST /api/issue-and-process
	•	POST /process-prescription-attachments
	•	GET /health

Responsibilities of /api/issue-and-process:
	•	Validate request
	•	Call Supabase RPC to issue prescription
	•	Trigger processing logic
	•	Return Rx code and processed attachment paths

Responsibilities of /process-prescription-attachments:
	•	Process an already-issued prescription
	•	Generate preview and dispense PDFs
	•	Update database attachment fields

Database & Storage (Supabase)
	•	PostgreSQL database with RPC functions
	•	Storage bucket:
	•	prescription-attachments

Current Working State

The following are now working:
	•	Practice-side upload from practice.html
	•	Practice-side issue-and-process via hosted Railway backend
	•	Railway deployment is live and no longer dependent on local laptop server
	•	Prescription processing generates:
	•	canonical original attachment path
	•	preview PDF
	•	dispense PDF
	•	Existing pharmacy-side HTML remains separate and continues to handle preview/dispense flow
	•	Manual SQL and manual curl are no longer required for the practice-side issuing workflow

Practice UI (Minimal Friction MVP)

Current inputs on practice.html:
	•	Prescriber
	•	Validity period
	•	Controlled drug checkbox
	•	defaults to No
	•	Prescription PDF upload

Current behaviour:
	1.	User selects prescriber
	2.	User selects validity period
	3.	User optionally ticks controlled drug
	4.	User uploads PDF
	5.	User clicks “Upload and issue prescription”
	6.	Frontend uploads file to Supabase Storage
	7.	Frontend calls Railway backend /api/issue-and-process
	8.	Backend issues prescription and processes files
	9.	UI returns Rx code

Pharmacy UI

Existing pharmacy HTML is still used and remains separate from the practice UI.

Purpose:
	•	Enter or use Rx code
	•	View watermarked preview
	•	Dispense prescription

This separation is intentional:
	•	Practice UI = issue
	•	Pharmacy UI = preview/dispense

Backend Hosting

Backend is now hosted on Railway.

Public domain:
	•	https://prescription-processor-production.up.railway.app

Health endpoint:
	•	/health

The processor is no longer dependent on local localhost.

Environment Variables (Railway)

Required service-level variables:
	•	SUPABASE_URL
	•	SUPABASE_SERVICE_ROLE_KEY
	•	PROCESSOR_SECRET
	•	PRACTICE_UI_SECRET
	•	BUCKET_NAME

Notes:
	•	Missing variables cause startup failure
	•	A previous Railway crash was caused by missing PRACTICE_UI_SECRET
	•	Variables must be correctly deployed at service level

Security Model (Current MVP)

Current state:
	•	Practice UI uses a shared secret via Authorization: Bearer ...
	•	Backend validates PRACTICE_UI_SECRET

Important limitation:
	•	This secret is currently present in frontend code
	•	This is acceptable only for controlled MVP/demo use
	•	This is not the final production security model

Future direction:
	•	Replace shared secret with proper authentication
	•	Add practice accounts and role-based access

CORS

CORS needed to be enabled because practice.html is browser-based and calls Railway directly.

This was required due to:
	•	Browser preflight OPTIONS requests
	•	Authorization header in fetch requests

Without proper CORS handling, browser showed:
	•	“Load failed”
	•	preflight 502 / blocked fetch errors

Supabase Storage

Bucket:
	•	prescription-attachments

Current relevant policy state:
	•	Public read policy exists
	•	Upload path is now working in the MVP flow

Watermarking

Watermark system is now in a good state and considered acceptable.

Current watermark characteristics:
	•	Horizontal
	•	Dense
	•	Edge-to-edge feel
	•	Repeating phrase includes Rx code
	•	Brick/stagger pattern
	•	Final tuning accepted by user

Current watermark settings:
	•	fontSize = 12
	•	opacity = 0.16
	•	rowGap = 22
	•	stagger offset based on unit text width

Important Helper Structure in server.js

Current backend structure includes:
	•	requireSecret(...)
	•	requirePracticeUiSecret(...)
	•	processPrescriptionAttachment(...)
	•	/process-prescription-attachments
	•	/api/issue-and-process
	•	/health

The processPrescriptionAttachment(...) helper centralises:
	•	download from storage
	•	clean dispense PDF creation
	•	preview watermark generation
	•	upload of processed files
	•	DB update of attachment fields

Key Lessons Learned
	•	Missing environment variables on Railway cause full startup crash
	•	Railway changes must actually be deployed, not just edited
	•	Browser errors like “Load failed” can hide CORS/preflight issues
	•	The real product architecture needs a backend orchestration layer
	•	It was correct to separate practice and pharmacy UIs
	•	The processor should be hosted, not run from a laptop, for real demos

Current Limitations
	•	Practice UI still uses shared secret in browser code
	•	No real auth yet
	•	No polished success state yet
	•	No copy button for Rx code yet
	•	No direct handoff from practice UI to pharmacy UI yet
	•	Error handling is basic
	•	Security model is MVP-only

Recommended Next Steps

Immediate UI polish
	•	Improve practice success screen
	•	Display Rx code more clearly
	•	Add copy button
	•	Add direct link or handoff to pharmacy page

Security improvements
	•	Replace shared secret with real authentication
	•	Move toward practice-level auth and role separation

Product hardening
	•	Better logging
	•	Better error messages
	•	More robust storage policy review
	•	Cleaner production security model

Summary

The system is now a real hosted MVP with:
	•	Practice-side issuing UI
	•	Hosted Railway backend
	•	Supabase database and storage
	•	Working PDF processing
	•	Working watermarking
	•	Separate pharmacy-side preview/dispense UI

This is now suitable for demonstration, but not yet production-grade from a security perspective.

Resume Point

When resuming, start with:

“Improve the practice success screen and then tighten security/auth.”
