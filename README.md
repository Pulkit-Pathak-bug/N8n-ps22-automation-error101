# Shortlist — an eligibility and readiness agent for students

**Team Error 101 · Track 2: AI Automation with n8n · HackSprint, MAHE, 17–18 October 2026**

Shortlist reads the opportunity emails a student already receives, works out which ones they are
actually eligible for, explains every verdict, and then builds a day-by-day preparation plan for the
ones worth chasing. It asks for human approval before anything leaves the system.

---

## The problem

A student hunting internships watches Internshala, Unstop, LinkedIn and Naukri Campus, plus whatever
the placement cell emails out. Three things make this expensive:

- **Repetition.** Every posting has to be checked by hand against the same profile: CGPA cutoff,
  allowed branches, batch year, backlog limit.
- **Fragmentation.** The same opening arrives three times from three sources, and campus notices
  arrive as PDFs that nobody re-reads.
- **Decisions.** Eligible or not, worth the time or not, prepare for which round — several judgement
  calls per posting, repeated weekly.

The result is missed deadlines, applications sent to roles the student was never eligible for, and no
record of why anything was skipped.

## What Shortlist does

1. **Ingests** job-alert email, forwarded placement mail and notice PDFs into one normalised shape.
2. **Deduplicates** deterministically, by hashing subject and date — the same opening from
   three sources collapses into one item.
3. **Classifies** each item and extracts structured fields: company, role, CGPA cutoff, branches,
   batch, backlog limit, deadline, apply link.
4. **Decides eligibility in code**, not in the prompt, and records a reason: *"not eligible — needs
   8.0 CGPA, profile has 7.6."*
5. **Plans readiness** for eligible openings: compares role requirements against the student's skill
   profile, then back-plans a preparation schedule from the deadline into the free slots in their
   timetable.
6. **Tracks academic dates** from the same inbox: exams, quizzes, submissions, registrations and venue
   changes. When notices disagree, the higher-authority source wins; when two equally official sources
   disagree, it asks a human instead of guessing. Confirmed dates become calendar events with reminders.
7. **Asks for approval** on Telegram before sending, applying or writing anything.
8. **Logs every decision** with its reason, and scores itself against a labelled set.

## Architecture

![Data flow diagram](docs/dfd.png)

Full description: [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)

The design rule running through it: **the LLM extracts and classifies; plain code decides.** A model
is unreliable at comparing 7.6 against a 8.0 cutoff and cannot be audited; a Code node is reliable
and prints its reasoning. Everything a judge or a user needs to trust lives in the deterministic half.

## Tech stack

| Layer | Choice | Why |
|---|---|---|
| Orchestration | n8n Community Edition, Docker, Postgres | free, self-hosted, visual canvas for the demo |
| Reasoning | Google Gemini API (free tier) | no card required, fast enough for classification |
| Extraction | n8n Information Extractor + Structured Output Parser | fixed schema instead of free text |
| Safety | n8n Guardrails node | email is untrusted input; jailbreak and PII screening |
| Research | Tavily search API | cited sources for interview-round information |
| Human loop | Telegram bot, send-and-wait | approval before any outgoing action |
| State | Google Sheets (profile, decision log, prep plan) | inspectable during a live demo |
| Scheduling | Google Calendar | deadlines and prep blocks as real events |
| Measurement | n8n Evaluations | accuracy over a hand-labelled set |

Running cost: zero.

## Repository layout

```
workflows/     n8n workflow JSON, importable
docs/          architecture and data flow diagram
screenshots/   canvas and output screenshots
```

## Status

| Component | State |
|---|---|
| Ingestion and dedup skeleton | importable, see `workflows/01-skeleton.json` |
| Eligibility rule engine | implemented in the skeleton as a Code node |
| Classification and extraction | keyword stubs in the skeleton; Gemini versions planned |
| Deadline branch (conflicts, reminders) | rule-based skeleton, see `workflows/02-two-branches.json` |
| Readiness planner | planned for the build |
| Approval, digest, audit log | planned for the build |
| Evaluation run | planned for the build |

Built during the 24-hour event on 17–18 October; this repository is the Round 2 submission state.

## Team

**Error 101** — Pulkit Pathak, M.D. Adityapranav, Shreyas Sonaje, Dilraj P.D. || Computer Science & Engineering, Manipal Institute of Technology
Bengaluru. 

## Licence

MIT — see [LICENSE](LICENSE).
