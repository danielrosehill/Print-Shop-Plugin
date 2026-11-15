---
name: send-to-printer
description: Submit a file to a registered print shop or digital printer the way that printer takes jobs, by sharing it on Google Drive with their Gmail, emailing it as an attachment or a link, or giving a web-upload or walk-in checklist, and file a copy in that printer's queue folder. Use on "send this to the printer", "send it to <printer>", "share it with the print shop", "email this to <printer> for printing", "get this printed at <printer>".
---

# Send to printer

Get one or more files to a printer by its saved submission procedure. Printer records and
data paths: [references/user-data.md](../../references/user-data.md). For a full job folder
with an order document, use `share-job-folder` instead.

## Steps

1. **Resolve the printer** in `printers.json` (`slug`, `name`, `aliases`; else
   `default_printer`). Unknown printer with an address given: run `add-printer` inline, then
   continue. Unknown and no address: ask for one.
2. **Resolve the file(s)**: most recent output in the session, a path, or a Drive file.
3. **File it** into the printer's queue with `file-for-print` (mode `printer`), unless it's
   already there. This also gives the Drive file ID the share routes need.
4. **Take the job details** the user gave: copies, colour or B&W, single- or double-sided,
   paper size, binding, pickup time. Don't ask for any the printer's `procedure` doesn't
   need. They go into the message and the `history` line.
5. **Submit** by `submission`:
   - `drive-share`:
     - `share_target: queue-folder`: check the folder is shared with `share_email`
       (`get_drive_file_permissions`) and grant it if not.
     - `share_target: file`: grant the file with `manage_drive_access` (`action=grant`,
       `share_with=share_email`, role `share_role`, with a notification message carrying
       the job details).
   - `email-attachment`: send from `send_from` via the Gmail tool with the file attached. Over
     `email_max_mb`: switch to `email-link` and say so.
   - `email-link`: grant `email` reader access on the Drive file, then email the link.
   - `web-upload`: give the URL and the local path. Use a browser tool only if the user asks
     you to do the upload.
   - `manual`: give the queue link and the `procedure` as a checklist.
6. **Follow the `procedure`** for any extra step you can do (a set subject format, a CC). For
   any you can't (phone, WhatsApp, payment), list them for the user.
7. **Verify** the submission itself: read back the permission list, or find the sent message
   in Sent. A tool's success return alone doesn't count.
8. **Record it**: append to the printer's `history`, set `last_used`, and report one line:
   what went where, by which route, and any steps left for the user.

## Email body

Short and printer-facing:

```
Hi,

Please print the attached / linked file:

  <filename> — <pages> pages
  <copies> copies, <colour|B&W>, <single|double>-sided, <paper size>[, <binding>]
  [Needed by: <when>]

<link, if email-link>

<contact.name>
<contact.phone>
```

Subject: `Print job: <filename>`, unless `procedure` sets another format. Write it in the
printer's language if `procedure` says so.

## Notes

- If the printing folder is on a Google shared drive, the drive's settings may block
  sharing outside the organisation. A failed grant usually means this: tell the user, and
  offer `email-attachment` or `email-link` from My Drive instead.
- "Send" means send. Ask first only when the printer's address was guessed rather than
  saved or given.
