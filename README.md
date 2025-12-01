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

## Solution (High Level)

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


1) InboxAgent
   • fetch_recent_job_emails() → Gmail API
   • LLM parses emails → company/role/status/source/dates
   • save_parsed_applications() → state["parsed_applications"]

2) TrackerAgent
   • load_tracker_snapshot() → Sheets API → state["tracker_rows"]
   • LLM compares parsed_applications vs tracker_rows by thread_id
   • sync_tracker_sheet() → append / update rows idempotently

3) InsightsAgent
   • compute_insights() → stats + stalled applications
   • LLM generates human-readable weekly summary

```


## 🛠 Setup Instructions

Follow these steps to run **JobBuddy** locally with Gmail + Google Sheets + Gemini.

---

## Clone the Repository

git clone https://github.com/<your-username>/Job-Buddy.git
cd Job-Buddy
Your main notebook/script is: jobbuddy.ipynb

## Install Dependencies
pip install -r requirements.txt

## Add Your Gemini API Key (GOOGLE_API_KEY)
JobBuddy uses Gemini via Google AI Studio.

Generate your key:
Go to → https://aistudio.google.com
Create API Key

GOOGLE_API_KEY=your_api_key_here
⚠️ Do NOT commit .env to GitHub.

## Add Google OAuth Credentials (Gmail + Sheets)
JobBuddy requires:

Gmail API → read-only access
Google Sheets API → read/write access

Steps:
Open Google Cloud Console
Create (or select) a project
Enable: Gmail API, Google Sheets API

Go to APIs & Services → Credentials
Click: Create Credentials → OAuth Client ID → Desktop App
Download the OAuth JSON

Rename it to: credentials.json
Place it in your project root:

Copy code
Job-Buddy/
 ├── credentials.json
 ├── jobbuddy.ipynb
 └── ...

## Authenticate Your Google Account (First Run Only)
When you run the notebook/script:
A browser window opens
Sign in with the Google account you want to use
Approve Gmail + Sheets permissions
A new file is auto-generated:
token.json

## Set Up Your Google Sheet
Create a new Google Sheet

Name the tab:
Applications
Add this exact header row:

application_id | company | role | source | status | applied_date | last_activity_date | next_action | priority | thread_id
Copy your Sheet ID from the URL:

https://docs.google.com/spreadsheets/d/<YOUR_SHEET_ID>/edit
Update these fields in your code:

python
Copy code
JOBBUDDY_SHEET_ID = "<YOUR_SHEET_ID>"
JOBBUDDY_SHEET_NAME = "Applications"

## Run JobBuddy

