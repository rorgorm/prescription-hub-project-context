Project Context – Prescription Processing System (MVP)

Last Updated

2026-03-31

⸻

Overview

This project is a minimal end-to-end prescription processing system designed to demonstrate:
	•	Practice-side prescription issuing
	•	Backend processing and validation
	•	Pharmacy-side preview and dispense flow

The system is intentionally simple, with minimal infrastructure and no reliance on local tools or manual database interaction.

⸻

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

⸻

Backend
	•	Node.js (Express) server hosted on Railway

Main endpoints:
	•	POST /api/issue-and-process
	•	POST /process-prescription-attachments
	•	GET /health

/api/issue-and-process responsibilities:
	•	Validate request (including PRACTICE_UI_SECRET)
	•	Upload file to Supabase Storage
	•	Call Supabase RPC to issue prescription
	•	Trigger processing logic
	•	Return Rx code + processed attachment paths

/process-prescription-attachments responsibilities:
	•	Process an already-issued prescription
	•	Generate preview and dispense PDFs
	•	Upload processed files
	•	Update database attachment fields

⸻

Database & Storage (Supabase)
	•	PostgreSQL database with RPC functions
	•	Storage bucket:
	•	prescription-attachments

⸻

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
	•	Manual SQL and manual curl are no longer required

⸻

Practice UI (Minimal Friction MVP)

Inputs
	•	Prescriber
	•	Validity period
	•	Controlled drug checkbox (defaults to No)
	•	Prescription PDF upload

Behaviour
	1.	User selects prescriber
	2.	User selects validity period
	3.	User optionally ticks controlled drug
	4.	User uploads PDF
	5.	User clicks “Upload and issue prescription”
	6.	Frontend sends file + metadata to Railway backend
	7.	Backend handles upload, issuing, and processing
	8.	Response returned with Rx code

⸻

Pharmacy UI

Existing pharmacy HTML remains separate.

Purpose:
	•	Enter or use Rx code
	•	View watermarked preview
	•	Dispense prescription

Design decision:
	•	Practice UI = issuing
	•	Pharmacy UI = checking / dispensing

⸻

Backend Hosting
	•	Hosted on Railway

Public domain:
https://prescription-processor-production.up.railway.app

Health endpoint:
/health

⸻

Environment Variables (Railway)

Required service-level variables:
	•	SUPABASE_URL
	•	SUPABASE_SERVICE_ROLE_KEY
	•	PROCESSOR_SECRET
	•	PRACTICE_UI_SECRET
	•	BUCKET_NAME

Notes:
	•	Missing variables cause full server crash at startup
	•	Previous failure was due to missing PRACTICE_UI_SECRET
	•	Variables must be deployed (not just saved)

⸻

Security Model (Current MVP)

Current state:
	•	Practice UI sends:
	•	Authorization: Bearer PRACTICE_UI_SECRET
	•	Backend validates via requirePracticeUiSecret(...)

Limitations:
	•	Secret is exposed in frontend code
	•	No authentication or user identity
	•	No role separation

Future direction:
	•	Replace with proper authentication (e.g. Supabase Auth)
	•	Introduce practice accounts
	•	Add role-based access control

⸻

CORS

Required because:
	•	practice.html runs in browser
	•	Uses Authorization header
	•	Triggers preflight OPTIONS requests

Symptoms before fix:
	•	“Load failed”
	•	502 / blocked preflight

⸻

Supabase Storage

Bucket:
prescription-attachments

Current state:
	•	Public read policy enabled
	•	Backend performs uploads using service role key

⸻

Watermarking

System is now stable and accepted.

Characteristics:
	•	Horizontal layout
	•	Dense, edge-to-edge coverage
	•	Includes Rx code
	•	Brick/stagger pattern

Settings:
	•	fontSize = 12
	•	opacity = 0.16
	•	rowGap = 22
	•	stagger offset based on text width

⸻

Important Backend Structure (server.js)

Key components:
	•	requireSecret(...)
	•	requirePracticeUiSecret(...)
	•	processPrescriptionAttachment(...)
	•	/process-prescription-attachments
	•	/api/issue-and-process
	•	/health

processPrescriptionAttachment(...) handles:
	•	Download original file
	•	Generate clean dispense PDF
	•	Generate preview watermark PDF
	•	Upload processed files
	•	Update DB attachment fields

⸻

Key Lessons Learned
	•	Missing environment variables = total server crash
	•	Railway changes must be deployed to take effect
	•	CORS/preflight issues can appear as generic browser failures
	•	Backend orchestration layer is essential
	•	Hosting > local server for real-world workflow
	•	Separation of practice vs pharmacy UI is correct

⸻

Current Limitations
	•	Shared secret exposed in frontend
	•	No authentication system
	•	No polished success UI
	•	No Rx code copy button
	•	No direct handoff to pharmacy UI
	•	Basic error handling only
	•	Not production-secure

⸻

Recommended Next Steps

Immediate (UI polish)
	•	Show Rx code clearly
	•	Add copy button
	•	Add link to pharmacy view

Security
	•	Replace shared secret with authentication
	•	Add practice-level identity

Hardening
	•	Improve logging
	•	Improve error handling
	•	Review storage security
	•	Move toward production-grade architecture

⸻

Summary

The system is now a working hosted MVP with:
	•	Practice-side issuing UI
	•	Railway backend
	•	Supabase database + storage
	•	Automated PDF processing
	•	Stable watermarking
	•	Separate pharmacy UI

This is suitable for demonstration, but not production-ready.

⸻

Resume Point

When resuming:

“Improve the practice success screen, then implement authentication.”
