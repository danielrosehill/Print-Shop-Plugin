# print-shop user data

All user-specific data (the printing folder, the printer list, contact details) lives in a
private overlay directory, never in the plugin. Skills read and write only there.

## Where it lives

Resolve the first that applies:

1. `$PRINT_SHOP_DATA` if set.
2. `$CLAUDE_USER_DATA/print-shop` if `CLAUDE_USER_DATA` is set.
3. `~/.claude-user-data/print-shop` if `~/.claude-user-data` exists.
4. `${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins/print-shop` (create it).

Legacy: if none of these holds a `config.json` but `<same root>/digital-printing/config.json`
exists (the plugin's name before 2.0.0), read it, write it to `print-shop/config.json` and
tell the user it was migrated.

Never write under `~/.claude/plugins/` (clobbered on update).

## `config.json`

```json
{
  "printing_folder": {
    "type": "gdrive",
    "drive_id": "<shared drive ID, or null for My Drive>",
    "folder_id": "<root folder ID>",
    "url": "https://drive.google.com/drive/folders/<id>",
    "account": "<Google account the Drive tools act as>"
  },
  "layout": {
    "by_date":    { "name": "By Date",    "folder_id": "<id>", "pattern": "YYYY-MM-DD" },
    "by_subject": { "name": "By Subject", "folder_id": "<id>" },
    "printers":   { "name": "Printers",   "folder_id": "<id>" }
  },
  "default_filing": "by-date",
  "default_printer": null,
  "send_from": "<email address submissions are sent from>",
  "contact": { "name": "", "email": "", "phone": "" }
}
```

- `printing_folder.type` is `gdrive` or `local`. For `local`, use `path` instead of the IDs,
  and each `layout` entry has only `name` (a subfolder of `path`).
- `layout` maps the three filing modes to their top-level folders. Cache folder IDs here so
  later runs skip the lookup; if an ID 404s, look the folder up by name and rewrite the ID.
- `contact` is the customer block used on print orders and in submission emails.

## `printers.json`

One entry per printer or print shop. `slug` is the folder name under the `printers` layout
folder and the key the user refers to it by (match `name`, `slug` or `aliases` loosely).

```json
[
  {
    "slug": "copy-centre-jaffa",
    "name": "Copy Centre, Jaffa St",
    "aliases": ["copy centre", "the printer on jaffa"],
    "submission": "drive-share",
    "email": "orders@example.com",
    "share_email": "copycentre@gmail.com",
    "share_target": "queue-folder",
    "share_role": "reader",
    "email_max_mb": 20,
    "web_upload_url": null,
    "phone": "+000 0 000 0000",
    "address": "",
    "hours": "",
    "accepts": ["pdf"],
    "procedure": "Share the file, then WhatsApp them the filename, copies and colour/B&W. Pay on pickup.",
    "queue_folder_id": "<Drive folder ID of Printers/<slug>>",
    "added": "YYYY-MM-DD",
    "last_used": null,
    "history": []
  }
]
```

`submission` is how a job reaches this printer:

| Value | What `send-to-printer` does |
|---|---|
| `drive-share` | Shares the file (or the printer's queue folder, per `share_target`) with `share_email` |
| `email-attachment` | Emails the file as an attachment to `email` (falls back to `email-link` over `email_max_mb`) |
| `email-link` | Shares the file with `email` on Drive and emails the link |
| `web-upload` | Opens `web_upload_url`; the upload itself is manual or browser-driven |
| `manual` | Files the job in the queue and prints the `procedure` as a checklist (USB stick, walk-in, WhatsApp) |

`share_target` is `file` (share each job on its own) or `queue-folder` (share the printer's
whole queue folder once; later jobs filed there are visible to them automatically).
`procedure` is free text: any step the printer needs besides the file (a follow-up message,
an order form, a payment). Follow it after sending.

`history` gets one line appended per submission:
`{"date": "YYYY-MM-DD", "file": "<name>", "via": "<submission>", "ref": "<Drive URL or message ID>"}`.
