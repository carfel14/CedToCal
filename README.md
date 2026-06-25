# 🎂 Birthday Adder → Google Calendar

A single-page web app (no backend) that reads a CSV/Excel of names + Nicaraguan
cédulas, extracts the birthday encoded in each cédula, and adds them to your
Google Calendar as **yearly recurring all-day events**.

## How the cédula is parsed

Format: `XXX-DDMMYY-XXXXY`

- First 3 digits → region/municipality code (ignored)
- **Middle 6 digits → birthday as `DDMMYY`** (e.g. `020602` = 2 June 2002)
- Last block → serial + check letter (ignored)

Century rule: a two-digit year `YY` ≤ the current year's last two digits is read
as `20YY`, otherwise `19YY` (so in 2026, `26`→2026, `27`→1927).

## Setup (one time)

### 1. Get a Google OAuth Client ID (free)

1. Open <https://console.cloud.google.com> and create a project.
2. **APIs & Services → Library** → enable **Google Calendar API**.
3. **APIs & Services → OAuth consent screen** → choose **External**,
   fill the required fields, and under **Test users** add your own Google
   address. (While the app is in "Testing" only listed test users can sign in —
   that's fine for personal use.)
4. **APIs & Services → Credentials → Create Credentials → OAuth client ID →
   Web application**.
5. Under **Authorized JavaScript origins**, add the exact URL you'll open the
   page from. If you use the command below it's `http://localhost:8000`.
6. Copy the **Client ID** (ends in `.apps.googleusercontent.com`).

### 2. Set your Client ID

Copy the example config and paste your ID into it:

```bash
cp config.example.js config.js
```

Then edit `config.js`:

```js
window.GOOGLE_CLIENT_ID = "123456789-abc.apps.googleusercontent.com";
```

`config.js` is git-ignored, so it never gets committed. (The Client ID isn't a
secret — it's visible in the browser regardless — but keeping it out of the repo
avoids leaking it into version control.)

### 3. Serve the page

Google sign-in and the hashing both need a real origin, so serve over `http://`
(a plain `file://` open won't work). Any static server is fine:

```bash
python -m http.server 8000
```

Then open <http://localhost:8000>. (`npx serve`, VS Code Live Server, GitHub
Pages, etc. all work too — just make sure the URL matches an Authorized
JavaScript origin in Google Cloud.)

### 4. Use it

1. **Sign in with Google** (approve the calendar permission).
2. Upload your CSV/Excel. Columns are auto-detected — it looks for a
   `name`/`nombre` column and a `cedula`/`cédula`/`id` column.
3. Review the parsed birthdays, untick any you don't want, and
   **Add selected to Calendar**. Each added row links to the created event.

## File format

First row must be headers. Example (`sample.csv` included):

```csv
nombre,cedula
Juan Pérez,001-020602-0001A
María López,401-151199-0002B
```

## Notes

- Events are created on a dedicated **Cumpleaños** calendar (created
  automatically on first run, reused afterward), marked *private* and *free*
  (won't block your availability), with a popup reminder at 9:00 on the day.
- **Duplicates are skipped** when "Evitar duplicados" is on: each event gets a
  deterministic ID derived from `SHA-256(cédula)`, so re-adding the same person
  (across runs or within one file) is rejected by Google and marked "ya existe".
  The cédula itself is never written to the event — only its hash, via the ID.
- Nothing leaves your browser except the calls to Google. The Client ID lives in
  the git-ignored `config.js`.
