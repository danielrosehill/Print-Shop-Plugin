---
name: add-printer
description: Register a print shop or digital printer the user sends work to, with its email or Gmail address, how it takes jobs (Drive share, email attachment, web upload, walk-in) and any extra steps, and give it a queue folder. Also edits or lists printers. Use on "add a printer", "I'm using a new printer", "their Gmail is …", "they take files by email", "update the printer's address", "which printers do I have".
---

# Add printer

Maintain `printers.json`, the user's list of printers and how to submit to each. Schema and
data paths: [references/user-data.md](../../references/user-data.md).

## Add

1. Take what the user gave. Infer what you can and ask only for what's missing to send a job:
   - an email → `submission: email-attachment` unless they said otherwise;
   - a `@gmail.com` (or "share it with them") → `share_email`, `submission: drive-share`;
   - a URL → `web-upload`;
   - none of the above → `manual`.
   Anything said about follow-up steps ("then WhatsApp them", "they need copies and size in
   the email body", "pay in advance") goes into `procedure` verbatim.
2. Make the `slug` from the name: lowercase, hyphenated, short. Put the other names the user
   uses for it in `aliases`.
3. Create the queue folder `<Printers>/<slug>/` in the printing folder (see `file-for-print`
   for the Drive calls) and store its ID in `queue_folder_id`.
4. For `drive-share` with `share_target: queue-folder`: share the queue folder with
   `share_email` now (`manage_drive_access`, `action=grant`, role `share_role`,
   notification off). Ask first only if the user hasn't said to share with them.
5. Append to `printers.json`, set `added` to today, and show the saved entry as JSON.

The user may name a printer in passing while sending a job ("send this to the new place on
Agripas, their Gmail is x@gmail.com"): add it, then go straight on to `send-to-printer`.

## Edit, remove, list

- Edit: change the fields named, keep `history`. If `share_email` changes and the queue
  folder was shared, revoke the old grant and grant the new one.
- Remove: confirm, delete the entry, leave the Drive folder and its files in place, and say so.
- List: a table of name, submission method, address used, last used.
