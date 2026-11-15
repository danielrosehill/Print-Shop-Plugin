---
name: file-for-print
description: File a print-ready file (a PDF, a diagram, an image) into the user's printing folder on Google Drive or disk, by date, by subject, or into a specific printer's queue. Use on "put this in the printing folder", "file this for printing", "park it for printing", "put that in the by-date folder", "queue this for <printer>", "add it to the <printer> queue".
---

# File for print

Put a finished file where the user parks things for printing. Three filing modes, each with
its own top-level folder in the printing folder:

| Mode | Destination | Use when the user says |
|---|---|---|
| `by-date` | `<By Date>/<YYYY-MM-DD>/` | "by date", "today's folder", or nothing more specific |
| `by-subject` | `<By Subject>/<Subject>/` | a subject or project ("under taxes", "in the Ezra folder") |
| `printer` | `<Printers>/<printer slug>/` | a printer's name or "queue it for …" |

Data paths, `config.json` and `printers.json` are defined in
[references/user-data.md](../../references/user-data.md). Read it first.

## First run

No `config.json`: ask where the printing folder is (a Drive folder URL or ID, or a local
path). For Drive, list the folder. If it already holds folders that look like the three
modes ("By Date", "By Subject", "Printers" or close), map them and record their IDs; create
any that are missing only after saying so. Ask the default mode only if the user has not
implied one. Write `config.json`, then carry on with the filing.

## Steps

1. **Resolve the file.** "The PDF we just did" means the most recent output in this session;
   name it back to the user in the result. Several candidates and no clear winner: ask.
2. **Pick the mode** from the request, else `default_filing`.
   - `printer`: resolve against `printers.json` (`slug`, `name`, `aliases`). An unknown
     printer: offer `add-printer`, or file under a new `Printers/<slug>/` folder now and
     register the rest later.
   - `by-subject`: reuse an existing subject folder when one matches loosely (case,
     plural, synonym); create a new one only when none does, named in Title Case.
3. **Find or create the subfolder.** Drive: `list_drive_items` on the layout folder (pass
   `drive_id` for a shared drive), `create_drive_folder` if missing. Local: `mkdir -p`.
4. **Upload.** Drive: `create_drive_file` with `folder_id` = the subfolder. It does not read
   local paths: pass `base64_content` for files under ~5 MB, otherwise stage with
   `cloud-storage:stage-file` and pass the presigned URL as `fileUrl` (delete the staged
   object afterwards). Keep the original filename; on a name clash in the same folder,
   append ` (2)` rather than overwrite. Local: `cp`.
5. **Verify** by listing the subfolder and finding the file with a non-zero size. Do not
   report success from the upload call alone.
6. **Report** one line: the file, the folder path, the Drive link.

Do not delete the source file. Do not share anything: filing is private. If the user also
asked to send it, hand over to `send-to-printer` with the uploaded file's ID.

## Tools

Drive tools are the Google Workspace MCP ones (`google-workspace-personal__*` on a gateway, or
whatever Google Drive MCP the user has), acting as `printing_folder.account`.
