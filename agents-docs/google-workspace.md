# Google Workspace

Use this when the task involves Google Docs, Sheets, Slides, Drive, Gmail, or Calendar for DeviceNative work.

## DeviceNative-specific wrapper

- Preferred command in this repo: `/bin/zsh ./scripts/gws-devicenative`
- This wrapper pins `GOOGLE_WORKSPACE_CLI_CONFIG_DIR=/Users/alex/.config/gws-devicenative`
- Do not use bare `gws auth login` for DeviceNative work on this machine. The default `~/.config/gws` state is already used by another project/account.
- Current authenticated DeviceNative account on this machine: `alex@devicenative.com`
- Current approved scopes: Drive, Docs, and Sheets
- Normal use should not require another login unless `/bin/zsh ./scripts/gws-devicenative auth status` reports an invalid or missing token

Equivalent direct form:

```bash
env GOOGLE_WORKSPACE_CLI_CONFIG_DIR=/Users/alex/.config/gws-devicenative gws ...
```

## Current state

As of April 20, 2026, this DeviceNative profile is already authenticated and ready for Drive, Docs, and Sheets work.

Typical tasks that should now work directly:

- Read a Google Doc when the user provides a Doc URL
- Read a Google Sheet when the user provides a Sheet URL
- Create a new Google Doc and populate it with structured content
- Create or update a Google Sheet and append values

## First-time setup on this machine

As checked on April 20, 2026:

- `gws` is already installed at `/opt/homebrew/bin/gws`
- `gcloud` was not installed, so manual OAuth setup is the active path unless that changes later

1. Create the config directory if needed:

   ```bash
   mkdir -p /Users/alex/.config/gws-devicenative
   ```

2. In Google Cloud Console for the DeviceNative Google account/project:
   - Configure the OAuth consent screen
   - App type: External
   - If the app is in testing mode, add the DeviceNative Workspace email as a Test user
   - Create an OAuth client with type Desktop app

3. Save the downloaded client JSON to:

   ```text
   /Users/alex/.config/gws-devicenative/client_secret.json
   ```

4. Log in with only the scopes you actually need. Common starting point:

   ```bash
   /bin/zsh ./scripts/gws-devicenative auth login -s drive,docs,sheets,slides,gmail,calendar
   ```

5. Verify:

   ```bash
   /bin/zsh ./scripts/gws-devicenative auth status
   ```

If only Docs, Sheets, and Drive are needed, keep scopes narrower:

```bash
/bin/zsh ./scripts/gws-devicenative auth login -s drive,docs,sheets
```

## Guardrails

- Keep credentials, refresh tokens, and exported auth JSON outside the repo.
- Confirm with the user before any write, share, delete, or email-send action.
- Prefer read operations first.
- Use `--dry-run` when a helper command supports it.
- Avoid broad mailbox or Drive scans unless the task requires them.
- Never paste raw secrets, OAuth codes, or credential JSON into chat, docs, or commits.

## Useful commands

```bash
# Current auth state
/bin/zsh ./scripts/gws-devicenative auth status

# Read a doc by URL-derived file ID
/bin/zsh ./scripts/gws-devicenative docs documents get --params '{"documentId":"DOC_ID"}'

# List recent Drive files
/bin/zsh ./scripts/gws-devicenative drive files list --params '{"pageSize": 10}'

# Read a sheet range
/bin/zsh ./scripts/gws-devicenative sheets +read --spreadsheet SHEET_ID --range 'Sheet1!A1:D20'

# Read raw sheet metadata by URL-derived spreadsheet ID
/bin/zsh ./scripts/gws-devicenative sheets spreadsheets get --params '{"spreadsheetId":"SHEET_ID"}'

# Create a new Google Doc
/bin/zsh ./scripts/gws-devicenative docs documents create --json '{"title":"DeviceNative Working Notes"}'

# Append text to a doc
/bin/zsh ./scripts/gws-devicenative docs +write --document DOC_ID --text 'New note from Codex'

# Create a new Google Sheet
/bin/zsh ./scripts/gws-devicenative sheets spreadsheets create --json '{"properties":{"title":"DeviceNative Tracker"}}'

# Append rows to a sheet
/bin/zsh ./scripts/gws-devicenative sheets spreadsheets values append --params '{"spreadsheetId":"SHEET_ID","range":"Sheet1!A1","valueInputOption":"USER_ENTERED"}' --json '{"values":[["Name","Status"],["Example","Ready"]]}'
```

## Agent usage notes

- When the user pastes a Google Doc or Sheet URL, extract the file ID from the URL and use the corresponding Docs or Sheets command directly.
- For normal repo assistance, it is fine to create docs or sheets on demand after confirming user intent.
- Prefer Docs commands for document content and Sheets commands for tabular data instead of trying to treat every URL as a generic Drive file.

## Placement

- Put durable notes or imported reference material in `docs/` when it belongs to the repo as documentation.
- Put one-off exports, evidence, or debugging artifacts in `ops/` when that directory exists.
- Keep service-local notes near the owning code when they are only relevant to one component.
