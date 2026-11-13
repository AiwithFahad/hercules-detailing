# Hercules Detailing — AI Voice Receptionist

![Hercules Detailing studio](assets/banner.jpg)

**Premium car detailing, booked by voice.** A 24/7 AI receptionist that answers on the website by voice or text, checks the live calendar, sizes the slot to the vehicle, and books, moves or cancels appointments — confirming everything by email.

🔗 **Live:** https://hercules-car.netlify.app/

![Status](https://img.shields.io/badge/status-in%20testing-yellow) ![Voice](https://img.shields.io/badge/voice-ElevenLabs-blue) ![Automation](https://img.shields.io/badge/automation-n8n-orange)

---

## What it does

- **Books appointments** — collects the service, vehicle and email, checks real availability, offers actual free start times, and books only after the customer clearly confirms
- **Reschedules and cancels** — finds the booking by email, phone or booking ID, confirms first, then moves or cancels it
- **Answers questions** — packages, prices, individual services and opening hours, straight from the business data
- **Registers customers** — new callers are added to the CRM with their vehicle details; returning customers are recognised by email and greeted by name
- **Reports every conversation** — after each call, the transcript, a summary and structured data (customer, intent, booking ID, outcome) are logged automatically to Google Sheets

## Try the receptionist

Open the [live site](https://hercules-car.netlify.app/), press **Start talking** (or type in the chat), and say something like:

- “I'd like a Full Detail for my SUV this Saturday.”
- “What are your prices?”
- “I need to move my booking to next week.”
- “Cancel my appointment, please.”

Have ready: your email, your car's year, make and model, and the day you'd prefer. From hello to booked takes about two minutes.

## Packages

| Package | Price | Includes | Duration |
|---|---|---|---|
| Maintenance | $89 | Exterior hand wash, quick interior vacuum, glass | ~1 hour |
| Full Detail | $249 | Exterior wash, Interior Deep Clean, Leather & Fabric Care, Wheels & Engine Bay | ~4 hours |
| Ceramic Pro | $899 | Full Detail plus paint enhancement and ceramic coating | Full day |

Appointments: Monday to Saturday, 8:00 AM – 6:00 PM (Asia/Karachi). The receptionist takes bookings 24/7. Larger vehicles (SUV, truck, van, classic) automatically get about 25% more time.

## How it works

```
Customer (voice / text on the website)
        │
        ▼
ElevenLabs agent — "Hercules Receptionist"
        │  webhook tools (POST, key-authenticated)
        ▼
n8n Tool Gateway ── routes on the "tool" field ──► 8 sub-workflows
        │                                            Google Calendar · Sheets CRM · Gmail
        ▼
Post-call webhook (EOC Report) ──► Conversations / Transcripts / AI Summaries sheets
```

The agent never guesses: availability, prices and booking results all come from the tools, and it only reports a booking as successful when the booking workflow confirms it.

## The voice agent

- **Platform:** ElevenLabs Conversational AI
- **Model / voice:** Gemini Flash-class LLM · warm, professional English receptionist voice · low-latency Turbo TTS
- **Prompting:** Pakistan-time reasoning (UTC+5), strict confirm-before-book flow, identity by email, booking IDs spelled out on voice calls
- **Modes:** voice and text chat use the same agent

## Tools (8 webhook tools, one gateway)

| Tool | Purpose |
|---|---|
| `client_lookup` | Find a customer in the CRM by email or phone |
| `register_customer` | Register a new customer with vehicle details |
| `check_availability` | Free start times for a service and date, sized to the vehicle |
| `book_appointment` | Create the calendar event + CRM booking, send confirmation email |
| `lookup_appointment` | Find upcoming appointments by email, phone or booking ID |
| `reschedule_appointment` | Move a booking to a new start time |
| `cancel_appointment` | Cancel a booking and log the cancellation |
| `business_info` | Opening hours, packages, prices and services |

After each conversation, the agent fills 7 analysis fields — customer name, email, phone, intent, booking ID, call outcome and booking status — which the post-call webhook stores alongside the transcript and summary.

## Tech stack

- **Voice AI:** ElevenLabs Conversational AI (agent, webhook tools, post-call webhooks)
- **Automation:** n8n (Tool Gateway + 8 published sub-workflows)
- **Data & scheduling:** Google Calendar, Google Sheets (CRM, bookings, logs)
- **Email:** Gmail (confirmations for bookings, changes and cancellations)
- **Website:** static site on Netlify with the embedded voice/text receptionist

## Project status

The ElevenLabs agent was configured and published on 13 November 2026, following a full setup guide; the complete as-built configuration is documented in [docs/SETUP.md](docs/SETUP.md). The system is currently in its testing phase: workflow emails are routed to a test inbox, and the end-to-end test script (book → reschedule → cancel → verify sheets) is the next step before real-customer launch.

## Documentation

- [docs/SETUP.md](docs/SETUP.md) — the complete as-built setup: agent settings, the full system prompt, all 8 tool definitions, analysis fields, the post-call webhook, and the go-live checklist.
