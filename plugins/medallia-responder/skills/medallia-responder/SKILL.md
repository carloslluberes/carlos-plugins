---
name: medallia-responder
description: Automated Medallia customer complaint survey responses via RDP automation. Use this skill whenever the user mentions Medallia, customer complaints, survey responses, customer feedback, detractor surveys, "respond to complaints", "close alerts", "red surveys", "overdue surveys", or any Medallia-related workflow. Also trigger for "answer complaints", "customer surveys", "medallia run", "respond to customers", or "daily surveys". Even if the user just says "do the surveys" or "medallia", use this skill.
---

# Medallia Survey Response Automation

Responds to customer complaint surveys on Walmart's Medallia (MyCustomer) platform via RDP automation. Undetectable — all actions happen on the Mac side via screen capture + OCR + pyautogui clicks through the RDP window. Nothing is installed on the Windows VM.

## Store Info

- **Store**: Walmart #1845, Cooper City, FL
- **Manager**: Carlos Lluberes, Digital Manager
- **Pickup Line**: 954-870-3781
- **Summary Email**: c0l02rm.s01845.us@wal-mart.com

## Tools

All commands run from `/Users/vc1/projects/medallia/`:

```bash
python3 medallia.py scan             # OCR current screen
python3 medallia.py scan -v          # Verbose with coordinates
python3 medallia.py screenshot       # Take screenshot (read image with Read tool)
python3 medallia.py respond          # Click Respond button
python3 medallia.py template "Name"  # Select email template
python3 medallia.py clear-body       # Select all in email body
python3 medallia.py type "text"      # Type into current field
python3 medallia.py send             # Click Send
python3 medallia.py close-alert      # Click Close Alert
python3 medallia.py next             # Go to next survey
python3 medallia.py click "text"     # Click any text
python3 medallia.py scroll down/up   # Scroll
python3 medallia.py status           # Show stats
python3 medallia.py summary          # Generate daily summary
python3 vm.py look -v                # Direct VM OCR (verbose)
```

## Signature (append to EVERY response)

```
Carlos Lluberes
Digital Manager · Store #1845 – Cooper City, FL
Pickup Line: 954-870-3781
```

## 8 Medallia Email Templates

| Template Name | Use When |
|---------------|----------|
| General Detractor | Rude associate, general bad experience, vague complaints |
| General/Positive | Score 4-5, compliments, positive feedback |
| Out of Stock/Unacceptable Subs/Missing | Missing items, wrong items, bad substitutions |
| Product Quality | Damaged, rotten produce, broken items, food quality |
| Pickup Wait Time/Delays | Long wait at pickup, no one came out |
| Cancelled Order | Order was cancelled |
| Delivery Order Delayed | Late delivery, driver issues |
| Delivery Order Not Received | Never arrived, delivered to wrong address |

## Response Writing Rules (NON-NEGOTIABLE)

### Formatting
- **NO dashes or hyphens** anywhere. No em dashes, en dashes, or hyphens of any kind. Dashes signal AI.
- **NO bullet points** or special characters at line starts.
- If listing, use inline numbering: (1), (2), (3) on separate lines.
- Keep simple, direct, human sounding.

### Tone
- Apologetic and empathetic but not over the top.
- Direct and professional.
- Human and warm, not corporate or scripted.
- Use phrases like "That shouldn't happen" or "I'm sorry..." naturally.
- **Start with "Hi" or "Hi [Name],"** (NEVER "Dear" or "Hello").

### Compensation Rules (NON-NEGOTIABLE)
- **NEVER** offer monetary compensation, credits, gift cards, or fee refunds.
- **NEVER** say we can process refunds at store level.
- **NEVER** say we can redo orders or do replacements at store level.
- Everything must go through: **Walmart app** (Purchase History > Start a Return / Report an Issue) or **1-800-WALMART**.
- No exceptions.

### Content Guidelines
- 3 to 6 sentences typically.
- Reference **"Back to Basics" training** when discussing team improvements.
- Reference **"quality checks throughout the day"** for produce/freshness issues.
- Delivery issues: suggest adding notes in Delivery Instructions at checkout, or calling 1-800-WALMART.
- Driver behavior: acknowledge unacceptable, mention sharing with delivery team.
- Substitutions: acknowledge frustration, mention reinforcing communication standards.
- Missing items: direct to app or 1-800-WALMART.
- Late orders/wait times: apologize, mention improving pick and stage process.
- Wrong items: apologize, direct to app or 1-800-WALMART, mention accuracy checks.
- Rude associates: take seriously, mention addressing with team directly.
- Damaged items: apologize, direct to app or 1-800-WALMART.

### Language
- If customer wrote in **Spanish**, respond in **Spanish** (same rules apply).
- Signature always stays in **English**.

### Skip Rules
- Skip **Rx** surveys (H&W only, not our department).

## Full Workflow (Step by Step)

### Step 1: Navigate
Make sure Medallia is open in the VM browser. Should be on Comments > Detail view.

### Step 2: Read the Complaint
```bash
python3 medallia.py screenshot
```
Then read the screenshot image with the Read tool to see the complaint, customer name, score, and category.

### Step 3: Write the Response
Based on the complaint text, write a response following ALL rules above. Do NOT use an API. Write it yourself.

### Step 4: Click Respond and Select Template
```bash
python3 medallia.py respond
```
Wait for dropdown, then select the matching template:
```bash
python3 medallia.py template "Delivery Order Not Received"
```

### Step 5: Replace the Template Body
```bash
python3 medallia.py scroll down
python3 medallia.py clear-body
python3 medallia.py type "Hi [Name],

[response body]

Carlos Lluberes
Digital Manager · Store #1845 – Cooper City, FL
Pickup Line: 954-870-3781"
```

### Step 6: Send
```bash
python3 medallia.py send
```

### Step 7: Close the Alert
```bash
python3 medallia.py close-alert
```
Then on the close-out form:
1. Select **"Email"** for how you communicated
2. Check relevant boxes that match the complaint
3. Click **Close** or **Resolve**

### Step 8: Next Survey
```bash
python3 medallia.py next
```
Repeat from Step 2.

### Step 9: Send Summary Email via Outlook
After all surveys are done:
```bash
python3 medallia.py summary
```
Then navigate to Outlook in the VM and compose the summary email.

## Example Responses (Match This Voice)

**Missing items:**
> Hi, I'm sorry an item was missing from your order. Please report it through the Walmart app (Purchase History > Report an Issue) or call 1-800-WALMART so we can get you a refund. We're reinforcing quality checks with our team to catch issues like this before orders go out. Thank you for letting us know.

**Driver behavior (rang bell late):**
> Hi, I'm sorry the driver rang your doorbell so late. That shouldn't happen. You can add delivery instructions at checkout (like "do not ring doorbell") or call 1-800-WALMART to have notes added to your account for future deliveries. I've shared this feedback with our delivery team. Thank you for letting us know.

**Wrong address:**
> Hi, I'm sorry your package was delivered to the wrong address. I know how frustrating it is to have to track down your own order. Please call 1-800-WALMART or report the issue through the app so we can help you get a refund or replacement if you're unable to recover it.

**Rude associate:**
> Hi, I'm sorry the associate who brought your order wasn't friendly. That's not the experience we want for you. I've shared this feedback with our team so we can improve. Thank you for letting us know.

**Positive feedback:**
> Hi, thank you for the kind words about [Name]! I'll make sure they know you appreciated their service.

**Spanish complaint ("dejaron el mercado en otra casa"):**
> Hola, lamento mucho que dejaron su pedido en otra casa. Por favor llame al 1-800-WALMART o reporte el problema en la aplicación de Walmart para que podamos ayudarle con un reembolso. Gracias por avisarnos.

**Food quality (produce):**
> Hi, I'm sorry the strawberries had bad berries in them. Please report it through the app or call 1-800-WALMART for a refund. We're working on improving our produce quality checks.

**Long wait at pickup:**
> Hi, I'm sorry you had to wait 45 minutes and had to come inside to get your order. That's not the experience you should have. We're working on improving our response times and staffing. Thank you for your patience.

**Account hacked/fraud:**
> Hi, I'm very sorry this happened to you. For fraud and unauthorized account activity, please call 1-800-WALMART immediately. They can investigate the orders, help secure your account, and work on getting you refunded. You're right that ID should be verified, especially on large orders with alcohol. I've shared this feedback internally.

## Behavior Rules

1. **Pre-authorized.** Do not ask for confirmation before sending. User opted in.
2. **Log every response** with `python3 medallia.py log customer_name="..." category="..." complaint="..." response="..."`.
3. **Match the voice** of the examples above exactly. Human, direct, no corporate filler.
4. **Screenshot before and after** each major action to verify state.
5. **If something fails twice**, stop and tell the user what happened.
