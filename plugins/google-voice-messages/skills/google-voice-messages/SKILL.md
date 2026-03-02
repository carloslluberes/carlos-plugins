---
name: google-voice-messages
description: OPD recruitment messaging and candidate tracking pipeline using Google Voice (browser automation) and Airtable. Use this skill whenever the user mentions Google Voice, GV messages, GV texts, recruiting candidates, OPD hiring, candidate pipeline, texting applicants, sending recruitment messages, checking candidate replies, tracking interview/orientation status, or any request involving outreach to job candidates. Also trigger for "text a candidate," "check who replied," "update candidate status," "who's coming to orientation," "send interview details," or any hiring-related messaging. Even if the user just says "message that candidate" or "any new replies from applicants," use this skill.
---

# Google Voice OPD Recruitment Pipeline

Manages OPD candidate recruitment messaging via Google Voice SMS and Airtable pipeline tracking.

## Prerequisites

- Claude in Chrome browser extension active with MCP tab group
- User logged into Google Voice at voice.google.com (carloslluberes@gmail.com)
- Airtable MCP connected
- GV number: (786) 262-8638
- **Read `references/gv-dom.md` for DOM selectors before any browser automation**

## Airtable Reference

**Base**: OPD Recruitment Pipeline — `app6XO4KCFMNXBII4`

| Table | ID |
|-------|-----|
| Candidates | `tblNUhBbLqcoa9OuS` |
| Message Log | `tbl2G0LtotwayE15f` |

### Candidates Fields

| Field | Type | Details |
|-------|------|---------|
| Name | text | primary field |
| Phone | phone | |
| Status | select | Contacted, Replied, Interview Scheduled, Interview Complete, Orientation Scheduled, Orientation Complete, Hired, No-Show, Declined, No Response |
| Position | select | OPD Picker, OPD Dispenser, OPD Digital TL, Other |
| Source | text | |
| Interview Date | date | |
| Orientation Date | date | |
| Last Message Sent | long text | |
| Last Message Received | long text | |
| Last Contact Date | date | |
| Notes | long text | |

### Message Log Fields

| Field | Type | Details |
|-------|------|---------|
| Summary | text | primary field |
| Candidate | linked | → Candidates |
| Direction | select | Outbound, Inbound |
| Message | long text | |
| Timestamp | datetime | ET timezone |
| Status | select | Sent, Delivered, Failed, Received |

## Core Workflows

### 1. Send Message to a Candidate (Full Auto)

1. **Lookup in Airtable:**
   `search_records(baseId="app6XO4KCFMNXBII4", tableId="tblNUhBbLqcoa9OuS", searchTerm="[name]")`
2. **Navigate** to `https://voice.google.com/u/0/messages`
3. **Compose** (see `references/gv-dom.md` → Compose section):
   a. `find "Send new message"` → click
   b. Click "To" field, type digits only, wait 2s, click suggestion
   c. `find "Type a message"` → `form_input` with message text
   d. `find "Send message"` → click
4. **Verify:** URL changes to `?itemId=t.%2B1...` = success
5. **Log** to Airtable Message Log + update Candidate record
6. **Report:** "Sent to [Name]: [preview]"

### 2. Bulk Outreach

1. **Query:** `list_records(baseId="app6XO4KCFMNXBII4", tableId="tblNUhBbLqcoa9OuS", filterByFormula="Status='Contacted'")`
2. **Show list** (informational, no gate)
3. **Per candidate:** compose → send → wait 2-3 seconds → log
4. **Report summary**

### 3. Check Incoming Replies

1. **Navigate** to messages page
2. **JavaScript extraction** (most reliable — see `references/gv-dom.md` → Inbox section):
   ```javascript
   document.querySelectorAll('gv-thread-list-item').forEach(item => {
     const isUnread = item.querySelector('.container')?.classList.contains('unread');
     const contact = item.querySelector('.participants')?.textContent?.trim();
     const preview = item.querySelector('.preview')?.textContent?.trim();
   });
   ```
   Or use `read_page` and look for "Unread" markers.
3. **Cross-reference** with Airtable candidates
4. **Click into** each matching conversation, extract new messages using `.cdk-visually-hidden` text
5. **Update Airtable:** Last Message Received, Last Contact Date, Status → "Replied"
6. **Log** inbound in Message Log

### 4. Pipeline Status

1. `list_records(baseId="app6XO4KCFMNXBII4", tableId="tblNUhBbLqcoa9OuS")`
2. Group by Status, present counts
3. Highlight: replied but no follow-up, interviews/orientations today, no response after X days

### 5. Update Candidate Status

1. Search Airtable → update Status + dates + notes
2. Confirm: "[Name] → [New Status]"

### 6. Add New Candidate

1. Minimum: Name + Phone
2. Create in Airtable (Status = "Contacted")
3. Optionally send initial outreach immediately

## Message Templates

Short, professional, friendly — 1-3 sentences. Default English, auto-switch to Spanish if candidate messages in Spanish.

**Initial outreach:**
> Hi [Name], this is the Digital/OPD department at Walmart in Cooper City. We saw your application and wanted to reach out about the [position] opening. Are you still interested? If so, we'd love to set up a quick interview.

**Interview invite:**
> Hi [Name], we'd like to invite you for an interview on [date] at [time] at Walmart Store #1845, Cooper City. Let me know if that works!

**Interview reminder:**
> Hi [Name], just a reminder about your interview tomorrow at [time]. See you then!

**Orientation details:**
> Hi [Name], congrats! Your orientation is scheduled for [date] at [time]. Please bring two forms of ID. Any questions, let me know.

**No-show follow-up:**
> Hi [Name], we missed you at your [interview/orientation] today. Still interested? Let us know and we can reschedule.

**Decline:**
> Hi [Name], thank you for your interest in the OPD position. We've decided to go in a different direction. We appreciate your time and wish you the best.

## Behavior Rules

1. **Full auto-send.** No confirmation prompts. User opted in.
2. **Airtable = source of truth.** Always lookup before messaging.
3. **Bilingual.** English default, switch to Spanish if candidate messages in Spanish.
4. **Rate limit.** 2-3 seconds between bulk sends.
5. **Store info:** Walmart #1845, Cooper City, FL — Digital/OPD department.
