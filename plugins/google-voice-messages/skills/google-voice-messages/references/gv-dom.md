# Google Voice DOM Reference

Google Voice is an Angular application using custom `gv-*` web components with Material Design (`mat-*`).

## Angular Component Hierarchy

```
gv-app
├── banner (top bar: search, settings, account)
├── gv-side-nav (left sidebar)
│   └── tablist (Calls, Messages, Voicemail, Archive, Spam)
├── main
│   ├── gv-messaging-view
│   │   ├── button "Send new message"
│   │   └── gv-thread-list (inbox conversation list)
│   │       └── gv-thread-list-item (× N conversations)
│   └── gv-thread-details (right panel — conversation view)
│       ├── gv-thread-details-header
│       ├── gv-text-message-list
│       │   └── gv-text-message-item (× N messages)
│       └── gv-message-entry (compose bar at bottom)
├── gv-call-sidebar (right — phone dialer)
│   └── gv-make-call-panel
│       ├── gv-dialpad
│       └── gv-contact-list (suggestions)
└── gv-people-and-options (contact panel)
```

## Key CSS Selectors

| Element | CSS Selector | Notes |
|---------|-------------|-------|
| Message textarea | `textarea.message-input` | The compose text input |
| Send button | `button[aria-label="Send message"]` | Reliable selector |
| Search input | `input[placeholder="Search Google Voice"]` | Top search bar |
| Phone dialer input | `input[placeholder="Enter a name or number"]` | Right-side call panel |
| File upload button | `button[type="file"]` or nearby `"Add image"` | Image attachment |

## Inbox: Conversation List

Component: `gv-thread-list` → contains `gv-thread-list-item` elements

Each `gv-thread-list-item` internal structure:

```
gv-thread-list-item
  div.container[role="button"]     ← CLICK THIS to open conversation
    ├── .container.read            ← has class "read" if read
    ├── .container.unread          ← has class "unread" if unread
    ├── .batch-selection           ← checkbox for bulk select
    ├── gv-avatar                  ← contact avatar
    ├── .thread-info
    │   ├── .title                 ← full title: "Contact Name . Date"
    │   ├── .participants          ← contact name or phone number (clean)
    │   ├── .timestamp             ← relative date: "Thu", "Feb 18", etc.
    │   └── .subtitle              ← full text: "Preview text.Full timestamp."
    └── .preview                   ← last message preview snippet
```

### JavaScript Extraction

```javascript
document.querySelectorAll('gv-thread-list-item').forEach(item => {
  const contact = item.querySelector('.participants')?.textContent?.trim();
  const timestamp = item.querySelector('.timestamp')?.textContent?.trim();
  const preview = item.querySelector('.preview')?.textContent?.trim();
  const subtitle = item.querySelector('.subtitle')?.textContent?.trim();
  const isUnread = item.querySelector('.container')?.classList.contains('unread');
  const isRead = item.querySelector('.container')?.classList.contains('read');
});
```

Via accessibility tree (`read_page`): Each listitem's button contains accessible text with contact name/number, date, message preview, full timestamp, and "Unread" indicator if applicable.

## Thread View: Individual Messages

Component: `gv-thread-details` → `gv-text-message-list` → `gv-text-message-item`

Each `gv-text-message-item` structure:

```
gv-text-message-item
  div.full-container                ← main wrapper
    ├── .full-container.outgoing    ← YOUR sent messages (blue/teal bubbles)
    ├── .full-container.incoming    ← THEIR messages (gray bubbles)
    ├── .end-of-cluster             ← last message in consecutive cluster
    ├── .status                     ← visible timestamp: "7:49 PM"
    ├── .cdk-visually-hidden        ← ACCESSIBLE TEXT (best for extraction):
    │                                 "Message from you, [text], [full timestamp]."
    │                                 "Message from [contact], [text], [full timestamp]."
    └── .message-row
        ├── .message-row.outgoing   ← sent message bubble
        ├── .message-row.incoming   ← received message bubble
        └── gv-avatar               ← sender avatar
```

### Best Extraction Method

The `.cdk-visually-hidden` div inside each `gv-text-message-item` contains the complete accessible text with sender, message content, and full timestamp:

```javascript
document.querySelectorAll('gv-text-message-item').forEach(msg => {
  const accessible = msg.querySelector('.cdk-visually-hidden')?.textContent?.trim();
  // Returns: "Message from you, [text], Sunday, March 1 2026, 7:49 PM."
  // Or: "Message from [contact name], [text], Monday, February 23 2026, 2:45 PM."
  const isOutgoing = msg.querySelector('.full-container')?.classList.contains('outgoing');
  const visibleTime = msg.querySelector('.status')?.textContent?.trim();
});
```

## Compose: New Message Flow

### Step 1 — Click "Send new message"
Located in `main`, use `find` tool with query `"Send new message"`.

### Step 2 — Enter recipient
After clicking, the compose panel appears with a "To" field area.

1. Click directly on the "To" field area (the input next to the "To" label)
2. Type the phone number as **digits only** (e.g., `7866507287`)
3. Wait **2 seconds** for dropdown suggestion: `"Send to (786) 650-7287"`
4. Click the suggestion to confirm the recipient

Recipient chip structure (after adding):
```
treegrid
  row
    gridcell "[contact name]"
    gridcell
      button "Remove [contact name]..."  ← to remove
```

### Step 3 — Type message
Use `form_input` tool on `textarea.message-input` (or via `find` with query `"Type a message"`). `form_input` is the most reliable method — it replaces any existing draft text.

### Step 4 — Send
Click `button[aria-label="Send message"]` (or via `find` with query `"Send message"`).

Send confirmation signals:
- URL changes from `?itemId=draft` to `?itemId=t.%2B1XXXXXXXXXX`
- The conversation thread view loads showing the sent message as a blue bubble

## Navigation URLs

| Page | URL |
|------|-----|
| Messages | `https://voice.google.com/u/0/messages` |
| Voicemail | `https://voice.google.com/u/0/voicemail` |
| Calls | `https://voice.google.com/u/0/calls` |
| Settings | `https://voice.google.com/u/0/settings` |

Alternate accounts: `/u/1/`, `/u/2/`, etc.

## Troubleshooting

### Recipient Field
- Use `type` action (not `form_input`) on the "To" field — it's not a standard input
- Type digits only, no dashes (e.g., `7866507287`)
- Wait 2 seconds for the suggestion dropdown, then click it
- If number matches a Google Contact, GV shows contact name — this is normal

### Message Field
- Use `form_input` on `textarea.message-input` — most reliable
- It replaces any existing draft text automatically
- Do NOT use `type` action for messages — it appends instead of replacing

### Old Drafts
- GV may retain drafts from previous sessions
- Always clear old recipients (click their "Remove" button)
- `form_input` on the textarea handles overwriting old message text

### Send Verification
- URL change from `?itemId=draft` → `?itemId=t.%2B1XXXXXXXXXX` = confirmed sent
- If URL still shows `draft`, send may have failed — screenshot and retry
