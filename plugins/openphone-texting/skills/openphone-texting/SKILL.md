---
name: openphone-texting
description: OPD recruitment messaging and candidate tracking pipeline using OpenPhone API and Airtable pipeline tracking. Use this skill whenever the user mentions OpenPhone, recruiting candidates, OPD hiring, candidate pipeline, texting applicants, sending recruitment messages, checking candidate replies, tracking interview/orientation status, or any request involving outreach to job candidates. Also trigger for "text a candidate," "check who replied," "update candidate status," "who's coming to orientation," "send interview details," or any hiring-related messaging. Even if the user just says "message that candidate" or "any new replies from applicants," use this skill.
---

# OPD Recruitment Messaging Pipeline

Manages OPD candidate recruitment messaging via **OpenPhone API** with Airtable pipeline tracking.

## OpenPhone API

- **API**: `POST https://api.openphone.com/v1/messages`
- **From number**: +17869987528 / (786) 998-7528
- **API Key**: `fIMLjeSU9MQnumY3tx8QU8eszzaBUqn4`
- **Auth header**: `Authorization: <api_key>` (no Bearer prefix)
- **Body**: `{"from": "+17869987528", "to": ["+1XXXXXXXXXX"], "content": "message"}`
- **Success**: HTTP 200/201/202
- **Phone Number ID**: `PNjX7uIQPd`
- **User ID**: `USNQg9lKvv`
- **No browser needed, no disconnections, instant delivery**

### Send Message
```python
import requests
resp = requests.post(
    'https://api.openphone.com/v1/messages',
    headers={'Authorization': 'fIMLjeSU9MQnumY3tx8QU8eszzaBUqn4', 'Content-Type': 'application/json'},
    json={'from': '+17869987528', 'to': ['+1XXXXXXXXXX'], 'content': 'Your message here'},
    timeout=30
)
# 200/201/202 = success
```

### Check Replies (GET messages)
```python
resp = requests.get(
    'https://api.openphone.com/v1/messages',
    headers={'Authorization': 'fIMLjeSU9MQnumY3tx8QU8eszzaBUqn4'},
    params={'phoneNumberId': 'PNjX7uIQPd', 'participants': ['+1XXXXXXXXXX'], 'maxResults': 10},
    timeout=30
)
incoming = [m for m in resp.json()['data'] if m['direction'] == 'incoming']
```

### Bulk Reply Check (all candidates at once)
```python
import requests, json

API_KEY = 'fIMLjeSU9MQnumY3tx8QU8eszzaBUqn4'
PHONE_ID = 'PNjX7uIQPd'

for name, phone in candidates.items():
    resp = requests.get(
        'https://api.openphone.com/v1/messages',
        headers={'Authorization': API_KEY},
        params={'phoneNumberId': PHONE_ID, 'participants': [phone], 'maxResults': 10},
        timeout=30
    )
    incoming = [m for m in resp.json()['data'] if m['direction'] == 'incoming']
    if incoming:
        print(f'{name}: {len(incoming)} replies')
        for m in incoming:
            print(f'  [{m["createdAt"]}] {m["text"]}')
```

## Prerequisites

- Airtable MCP connected
- Python `requests` library (no browser needed)

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

## Available Positions

Mention ALL open positions in outreach:
- **OPD Pickers** — Online order filling
- **OPD Dispensers** — Order handoff to customers
- **Overnight associates** — Stocking and freight
- **Fresh department** — Produce, deli, bakery, meat

When a candidate asks about full-time: OPD positions are **part-time that can lead to full-time**.

## Core Workflows

### 1. Send Message to a Candidate (Full Auto)

1. **Lookup in Airtable:**
   `search_records(baseId="app6XO4KCFMNXBII4", tableId="tblNUhBbLqcoa9OuS", searchTerm="[name]")`
2. **Send via OpenPhone API:**
   ```python
   requests.post('https://api.openphone.com/v1/messages',
       headers={'Authorization': 'fIMLjeSU9MQnumY3tx8QU8eszzaBUqn4', 'Content-Type': 'application/json'},
       json={'from': '+17869987528', 'to': ['+1{phone}'], 'content': message},
       timeout=30)
   ```
   - HTTP 200/201/202 = success
3. **Log** to Airtable Message Log + update Candidate record
4. **Report:** "Sent to [Name]: [preview]"

### 2. Bulk Outreach

1. **Query Airtable** for candidates without "Contacted" status (not yet texted)
2. **Show list** to user (informational)
3. **Per candidate:** Send via OpenPhone API → wait 2-3 seconds → log to Airtable
4. **Update each candidate:** Status → "Contacted", Last Message Sent, Last Contact Date
5. **Report summary** with count sent

**Bulk send script pattern:**
```python
import requests, time
API_KEY = 'fIMLjeSU9MQnumY3tx8QU8eszzaBUqn4'
FROM = '+17869987528'

for name, phone, rec_id in candidates:
    msg = f'Hi {name}, this is the Digital/OPD department at Walmart in Cooper City...'
    resp = requests.post(
        'https://api.openphone.com/v1/messages',
        headers={'Authorization': API_KEY, 'Content-Type': 'application/json'},
        json={'from': FROM, 'to': [f'+1{phone}'], 'content': msg},
        timeout=30
    )
    print(f'{name}: {resp.status_code}')
    time.sleep(2)  # Rate limit between sends
```

### 3. Check Incoming Replies

1. **Via OpenPhone API** (no browser needed):
   - Loop through all "Contacted" candidates from Airtable
   - GET messages for each candidate's phone number
   - Filter for `direction == "incoming"`
2. **Cross-reference** with Airtable candidates
3. **Update Airtable:** Last Message Received, Last Contact Date, Status → "Replied"
4. **Log** inbound in Message Log
5. **Report** who replied and what they said

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

**Initial outreach (with all positions):**
> Hi [Name], this is the Digital/OPD department at Walmart in Cooper City. We saw your application and wanted to reach out about the OPD position. We're also currently hiring for overnight and fresh department roles. Are you interested in any of these? If so, we'd love to set up a quick interview.

**Initial outreach (OPD only):**
> Hi [Name], this is the Digital/OPD department at Walmart in Cooper City. We saw your application and wanted to reach out about the OPD position. Are you still interested? If so, we'd love to set up a quick interview.

**Interview invite:**
> Hi [Name], we'd like to invite you for an interview on [date] at [time] at Walmart Store #1845, Cooper City. Let me know if that works!

**Interview scheduling (let candidate pick):**
> Hi [Name], great to hear! We'd love to set up a quick interview at Walmart Store #1845, Cooper City. What day works best for you this week?

**Interview reminder:**
> Hi [Name], just a reminder about your interview tomorrow at [time]. See you then!

**Orientation details:**
> Hi [Name], congrats! Your orientation is scheduled for [date] at [time]. Please bring two forms of ID. Any questions, let me know.

**No-show follow-up:**
> Hi [Name], we missed you at your [interview/orientation] today. Still interested? Let us know and we can reschedule.

**Decline:**
> Hi [Name], thank you for your interest in the OPD position. We've decided to go in a different direction. We appreciate your time and wish you the best.

**Part-time question response:**
> It's a part-time position that can lead to full-time. Are you interested? We'd love to set up a quick interview.

**Hold/delay message:**
> Sounds good, [Name]! Let me finish up a couple of things and I will let you know about a time.

## Behavior Rules

1. **Full auto-send.** No confirmation prompts. User opted in.
2. **Show before sending** for important/custom messages (interview invites, replies to questions).
3. **Airtable = source of truth.** Always lookup before messaging.
4. **Bilingual.** English default, switch to Spanish if candidate messages in Spanish.
5. **Rate limit.** 2-3 seconds between bulk sends.
6. **Always log.** Every message (in or out) gets logged to Message Log AND Candidate record updated.
7. **Mention all positions.** Initial outreach should mention OPD pickers/dispensing + overnight + fresh department roles.
8. **Store info:** Walmart #1845, Cooper City, FL — Digital/OPD department.
