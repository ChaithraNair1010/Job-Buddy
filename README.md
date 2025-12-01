# JobBuddy – Gmail → Job Tracker Agent

JobBuddy is an AI-powered assistant that reads your Gmail, finds job application emails, logs them into a Google Sheets tracker, and gives you a weekly summary of your job search.

> One prompt like:  
> **"Sync my job search from Gmail and tell me how my week went."**  
> runs the whole multi-agent pipeline end-to-end.

---

## 🧩 Problem

When you apply to many jobs across platforms (LinkedIn, company portals, ATS emails, etc.), your Gmail fills up with:

- application confirmations  
- recruiter follow-ups  
- interview invites  
- rejections  
- random job alerts and newsletters  

Manually tracking all of this in a spreadsheet is:

- boring  
- error-prone  
- easy to forget  

I wanted an agent that **reads my inbox for me**, **updates my job tracker**, and **summarizes my progress**, without me having to copy-paste anything.

---

## ✅ Solution (High Level)

JobBuddy connects three things:

1. **Gmail** – as the raw source of truth  
2. **Google Sheets** – as the structured job application tracker  
3. **Gemini-powered agents (via ADK)** – to interpret emails and keep the sheet updated

The pipeline:

1. **InboxAgent**  
   - Calls a tool to fetch job-related emails from Gmail  
   - Uses Gemini to decide company, role, status, etc.  
   - Saves a clean list of `parsed_applications` into shared state  

2. **TrackerAgent**  
   - Loads the current tracker from Google Sheets  
   - Merges new applications (by `thread_id`)  
   - Calls a tool to append/update rows in the sheet  

3. **InsightsAgent**  
   - Reads the updated tracker  
   - Computes stats (applied, interviews, rejections, stalled)  
   - Returns a friendly weekly summary back to the user  

---

## 🏗 Architecture Overview

### Big Picture

```text
USER
 │
 ▼
JobBuddyPipeline (SequentialAgent)
 │
 ├── InboxAgent    → Gmail Tool   → parsed_applications in state
 ├── TrackerAgent  → Sheets Tools → tracker_rows in state
 └── InsightsAgent → Insights Tool→ summary for user

