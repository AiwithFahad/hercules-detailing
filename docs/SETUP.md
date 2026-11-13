# Hercules Receptionist — As-Built Setup Documentation

**ElevenLabs Voice Agent for Hercules Detailing — what was actually configured**
Built: Friday 13 November 2026 · Status: Created and PUBLISHED · Agent ID: agent_8001m7acd9h7fwnvw5k1kmr2jpdv
ElevenLabs account: Ali Ahmed · Source document: Hercules Detailing – ElevenLabs Voice Agent Setup Guide (Nov 13, 2026)

---

## 1. How the system fits together

The customer talks or types to the ElevenLabs agent (website widget or test panel). For every action, the agent calls a webhook tool that POSTs to one n8n Tool Gateway URL. The gateway checks the x-hercules-key header, reads the "tool" field in the request body, and runs the matching sub-workflow (Google Calendar, Google Sheets, Gmail), then returns JSON the agent reads or paraphrases. When a call ends, ElevenLabs sends the transcript and analysis to the EOC Report post-call webhook, which writes to the Conversations, Transcripts and AI Summaries sheets. Business timezone throughout: Asia/Karachi (UTC+05:00). Business hours: Monday to Saturday, 8 AM – 6 PM Pakistan time. Closed Sunday. During testing, all workflow emails go to myroadmap20@gmail.com.

## 2. Agent settings (as built)

| Setting | Value set |
|---|---|
| Agent name | Hercules Receptionist |
| Language | English (en) |
| LLM | Gemini 3.5 Flash (see Deviations) |
| Temperature | 0.3 |
| Timezone | Asia/Karachi |
| Voice | Ms. Walker – Warm, Reassuring, and Round (English) |
| TTS model | Eleven v4 Turbo (low latency) |
| First message | See Section 3 |
| System prompt | See Section 4 |
| Max conversation duration | 600 seconds |
| Text-only mode | Allowed (widget chat + agent text override ON) |
| Tags | Hercules Detailing, Receptionist, Booking |
| Dynamic variable | channel, default value: web-voice |
| Security | Authentication OFF, no domain allowlist (testing phase) |

## 3. First message (exact)

Hi, thanks for contacting Hercules Detailing! I can book, move or cancel a detailing appointment, or help you pick the right package. What can I do for you today?

## 4. System prompt (exact, as pasted)

# Identity
You are the Hercules Receptionist, the friendly, professional front desk of Hercules Detailing, a premium automotive detailing studio. You speak naturally, warmly and concisely (1-3 sentences per turn). You work 24/7, but appointments only happen during business hours.

# Current time
The current UTC time is {{system__time_utc}}. The business timezone is Asia/Karachi (Pakistan Standard Time, UTC+05:00). Always add 5 hours to UTC and reason about dates in Pakistan time. "Today", "tomorrow", "Friday" etc. are relative to the current Pakistan date.

# Business facts
- Hours: Monday to Saturday, 8:00 AM to 6:00 PM Pakistan time. Closed Sunday. Never book outside these hours.
- Packages: Maintenance $89 (exterior hand wash, quick interior vacuum, glass, about 1 hour). Full Detail $249 (exterior wash, Interior Deep Clean, Leather & Fabric Care, Wheels & Engine Bay, about 4 hours). Ceramic Pro $899 (Full Detail plus paint enhancement and ceramic coating, a full day).
- Individual services: Exterior Hand Wash (~1 hr, from $89), Interior Deep Clean (~2 hrs), Paint Correction (4-8 hrs), Ceramic Coating (~1 day), Leather & Fabric Care (~1.5 hrs), Wheels & Engine Bay (~1.5 hrs). Individual services other than the wash are quoted, or included in packages. Never invent prices.
- Vehicles: sedans, hatchbacks, SUVs, trucks, vans and classic cars. Larger vehicles (SUV, truck, van, classic) take about 25% longer; the system sizes the slot automatically.
- Recommend: quick refresh -> Maintenance; dirty interior / family car / selling the car -> Full Detail; swirls, dull paint, long-term protection, new car -> Ceramic Pro or Paint Correction.

# Booking flow (follow in order)
1. Find out the service or package, and the vehicle (year, make, model, and type: sedan/hatchback/SUV/truck/van/classic).
2. Ask for their email (spell it back to confirm). Call client_lookup. If found, greet them by name and reuse their details. If not found, collect full name and phone number, then call register_customer with name, email, phone and vehicle details.
3. Ask which day suits them, then call check_availability with the date (YYYY-MM-DD), service and vehicleType. Offer 2-3 of the returned start times. Never offer times not returned by the tool. If no slots are returned, offer to check another day.
4. Read back a summary: service, vehicle, day and time, approximate duration. Ask "Shall I book that?" and wait for a clear yes.
5. Only after a clear yes, call book_appointment with startTime as the exact ISO "start" value from check_availability (it ends in +05:00), plus email, fullName, phoneNumber, service, vehicle, vehicleType.
6. Only if the tool returns success true, confirm and read out the booking ID (spell it clearly, e.g. "H D, 2 6 1 1 2 5, C 0 D 9"). Mention a confirmation email is on its way. If success is false, explain the message and offer alternatives. Never claim a booking succeeded unless the tool said so.

# Reschedule / cancel
- Ask for the booking ID, or their email, and call lookup_appointment. Confirm which appointment.
- Reschedule: check_availability for the new day, offer times, confirm, then call reschedule_appointment with bookingId and newStartTime (exact ISO start from check_availability). Report the tool's result.
- Cancel: confirm "Are you sure you want to cancel your [service] on [day/time]?" Only after a clear yes call cancel_appointment with bookingId and a short reason.

# Other tools
- business_info: use for prices, hours, packages and services if unsure.

# Rules
- Never make up availability, booking IDs, prices or policies.
- Always get explicit confirmation before booking, rescheduling or cancelling.
- If a tool fails, apologise briefly and offer to try again or take their details for a callback.
- Keep personal data private. Don't read back full phone numbers unless asked.
- If asked something unrelated to detailing, politely steer back.
- In text chat, you may write dates and IDs normally; in voice, say them naturally ("Wednesday the 25th at 2 PM").

# Context
Channel: {{channel}}

NOTE — the final "# Context / Channel: {{channel}}" lines were appended during setup (the only addition to the guide's text). The ElevenLabs UI only registers a dynamic variable when it is referenced in the configuration; this line registers "channel" so the website can pass web-voice or web-text and the EOC Report can store it.

## 5. Webhook tools (8) — shared configuration

Every tool uses the same connection settings; only the name, description and body parameters differ.

| Field | Value |
|---|---|
| Type | Webhook |
| Method | POST |
| URL (Tool Gateway) | https://balancedorangutanmgrq1.app.n8n.cloud/webhook/16e081d9-f84a-4867-8345-0d18985d7416 |
| Header 1 | x-hercules-key = (the gateway key from the original setup guide — value intentionally not reprinted in this document; it is set identically in all 8 tools) |
| Header 2 | Content-Type = application/json |
| Response timeout | 25 seconds |
| Execution mode | Immediate |
| Pre-tool speech | Auto |
| Body schema description | Hercules tool request |
| First body parameter (every tool) | tool — String — value type CONSTANT — value = the exact tool name |

All other parameters are String type, value type "LLM prompt", with the descriptions listed per tool below.

### Tool 1 — client_lookup
Description: Look up a customer in the CRM by email or phone. Call after the customer gives their email.
Parameters: email (optional — "Customer email address"); phone (optional — "Customer phone number").

### Tool 2 — register_customer
Description: Register a new customer with their vehicle details. Call only when client_lookup returned found false.
Parameters: email (REQUIRED — "Customer email"); fullName (REQUIRED — "Customer full name"); phoneNumber (REQUIRED — "Customer phone number"); vehicleMake (optional — "e.g. Toyota"); vehicleModel (optional — "e.g. Corolla"); vehicleYear (optional — "e.g. 2020"); vehicleType (optional — "sedan, hatchback, SUV, truck, van or classic").

### Tool 3 — check_availability
Description: Get free start times for a service on a date, sized to the vehicle. Always call before offering times.
Parameters: date (REQUIRED — "Date in YYYY-MM-DD (Pakistan time)"); service (REQUIRED — "Service or package name"); vehicleType (optional — "sedan, hatchback, SUV, truck, van or classic").

### Tool 4 — book_appointment
Description: Book the appointment. Only call after the customer clearly confirmed the summary.
Parameters: startTime (REQUIRED — "Exact ISO start value returned by check_availability, e.g. 2026-11-25T10:00:00+05:00"); email (REQUIRED — "Customer email"); service (REQUIRED — "Service or package name"); fullName (optional — "Customer full name"); phoneNumber (optional — "Customer phone number"); vehicle (optional — "e.g. 2020 Toyota Corolla"); vehicleType (optional — "sedan, hatchback, SUV, truck, van or classic"); requests (optional — "Any special notes").

### Tool 5 — lookup_appointment
Description: Find upcoming appointments by email, phone or booking ID.
Parameters: email (optional — "Customer email"); phone (optional — "Customer phone number"); bookingId (optional — "Booking ID starting with HD-"). None required.

### Tool 6 — reschedule_appointment
Description: Move a booking to a new start time. Only call after the customer confirmed the new time.
Parameters: bookingId (REQUIRED — "Booking ID starting with HD-"); newStartTime (REQUIRED — "Exact ISO start value from check_availability").

### Tool 7 — cancel_appointment
Description: Cancel a booking. Only call after the customer clearly confirmed the cancellation.
Parameters: bookingId (REQUIRED — "Booking ID starting with HD-"); reason (optional — "Short reason for cancelling").

### Tool 8 — business_info
Description: Get opening hours, packages, prices and services.
Parameters: none — only the constant "tool" parameter.

Example of what n8n receives (book_appointment): { "tool": "book_appointment", "startTime": "2026-11-25T10:00:00+05:00", "email": "customer@example.com", "fullName": "Ali Khan", "phoneNumber": "+923001234567", "service": "Full Detail", "vehicle": "2020 Toyota Corolla", "vehicleType": "sedan" }

## 6. Analysis and data collection (as built)

Transcript summary / Analysis language: English.

Data collection items (all Type = String) — the EOC Report workflow reads these exact identifiers:

| Identifier | Description | Sheet column |
|---|---|---|
| customer_name | The customer's full name if given, otherwise empty. | Customer Name |
| customer_email | The customer's email address if given, otherwise empty. | Customer Email |
| customer_phone | The customer's phone number if given, otherwise empty. | Customer Phone |
| intent | The main intent: New booking, Reschedule, Cancellation, Availability check, Pricing enquiry, or General enquiry. | Intent |
| booking_id | The booking ID starting with HD- mentioned or created in the conversation, otherwise empty. | Booking ID |
| call_outcome | One of: Booked, Rescheduled, Cancelled, Enquiry answered, Unresolved. | Outcome |
| booking_status | Final booking status: Confirmed, Rescheduled, Cancelled, or None. | Booking Status |

Evaluation criterion added: booking_handled_correctly (Binary) — "The agent confirmed details before booking, rescheduling or cancelling, and only reported success when a tool returned success true."

## 7. Post-call webhook (EOC Report)

| Field | Value |
|---|---|
| Name | Hercules EOC Report |
| Level | Workspace webhook, attached to the Hercules Receptionist agent as an explicit post-call override |
| URL | https://balancedorangutanmgrq1.app.n8n.cloud/webhook/e72f9562-b2f2-409d-8d22-77bd38ba9841 |
| Event | post_call_transcription (Transcript) |
| Audio | OFF |
| Authentication | HMAC signing (see Deviations) |

No earlier webhook pointing at the old middlechickadeeuejc1 instance existed, so a new webhook was created. The webhook briefly became the workspace default during creation; that default was removed, so the Hotel Reservations Assistant agent is unaffected — the EOC Report fires only for Hercules Receptionist.

## 8. Deviations from the original guide

1. LLM: the guide suggests Gemini 2.5 Flash or GPT-4o mini. Neither appears in this account's model list, so Gemini 3.5 Flash — the available Gemini Flash-class model — was selected.
2. TTS: no separate Flash TTS family is offered in this UI; Eleven v4 Turbo (the low-latency option) was selected.
3. System prompt: one short "# Context / Channel: {{channel}}" block appended to register the channel dynamic variable (explained in Section 4). Everything else is verbatim.
4. Post-call webhook authentication: the guide specifies "none (HMAC optional)". The ElevenLabs webhook creator offers HMAC as its only auth method, so the webhook was created with HMAC signing. The guide states the n8n workflow does not check HMAC, so deliveries should still be accepted — verify this first when running the Step 7 tests (check that a finished test call appears in the Conversations sheet).

## 9. Not yet done

- Step 7 tests have NOT been run: prices question, new-customer registration, availability in Pakistan time, booking, reschedule, cancel, and the end-of-call sheet check. The shared Google Calendar is busy until 24 Nov 2026, so pick a later date with free slots for the booking test.
- Go-live checklist from the guide: restore customer emails in the n8n Gmail nodes, give Hercules its own Google Calendar, rotate the gateway key in n8n and in all 8 tools (and change the admin password), add the website domain to the agent allowlist, and update the website's agent ID to agent_8001m7acd9h7fwnvw5k1kmr2jpdv.
