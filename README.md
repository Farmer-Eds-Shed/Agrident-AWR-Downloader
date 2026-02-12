# Agrident Tasks → CSV

A single-file, zero-dependency web app that connects to an Agrident/Allflex reader, downloads task data, lets you **edit/delete rows**, and exports to **CSV**.

Supports:
- **Bluetooth (BLE)** via **Web Bluetooth** (works on Android + desktop Chrome/Edge where supported)
- **USB Serial (COM)** via **Web Serial** (desktop Chrome/Edge)

On **mobile**, the app **enforces BLE** and hides the Serial option.

---

## Features

- Connect / Disconnect
- Sync tasks on connect (`[XGTASK]`)
- Select a task from a dropdown (no separate “Tasks” card)
- Auto-load headers (`[XSH|idx]`) and data (`[CSW|idx]`)
- Preview up to 200 rows
- Row actions:
  - **Edit** (modal form)
  - **Delete**
- Export CSV with:
  - Header row (if available)
  - Date normalization to `dd/mm/yyyy` (accepts common formats)
  - Numeric cleanup for weight-like fields (removes leading zeros)

---

## Requirements

### BLE (Web Bluetooth)
- **Chrome / Edge**
- Android: works well
- Desktop: depends on OS + browser support

### Serial (Web Serial)
- **Chrome / Edge desktop only**
- Requires connecting the reader via USB serial / COM device
- Default baud rate: **9600**

> Safari and Firefox do not currently support Web Bluetooth/Web Serial in the same way, so they’re not recommended for this app.

---

## Usage

1. Save the HTML file locally as `index.html`
2. Open it in Chrome/Edge:
   - Easiest: double-click the file
   - Or serve locally (recommended for predictable behaviour):
     ```bash
     python -m http.server 8000
     ```
     Then open:
     - `http://localhost:8000`

### Connect (BLE)
1. Choose **Bluetooth (BLE)** (mobile will force this)
2. Click **Connect**
3. Select your device when prompted
4. Tasks auto-sync

### Connect (Serial)
1. Choose **USB Serial (COM)**
2. Click **Connect**
3. Pick the COM device when prompted
4. Tasks auto-sync

---

## Workflow

1. **Connect**
2. Select a **Task**
3. App loads:
   - Headers: `[XSH|idx]`
   - Records: `[CSW|idx]`
4. Preview table renders (first 200 rows)
5. Optionally **Edit/Delete** rows
6. Click **Download CSV**

---

## Commands Used

- `XGTASK` – fetch tasks list  
  - expects `XGTASKOK` to finish
- `XSH|<idx>` – fetch headers for selected task  
  - expects `XSHOK` to finish
- `CSW|<idx>` – fetch task rows (pipe-delimited frames)  
  - expects `CSWOK` to finish

Frames are parsed from streamed text by extracting bracket blocks like:
- `[XGTASK]`
- `[0|Weighing (Manu)|...|1]`
- `[XGTASKOK]`

---

## CSV Formatting Rules

### Dates
If a column header includes `"date"` (case-insensitive), values are normalized to `dd/mm/yyyy`.

Accepted examples:
- `08022026`
- `08-02-26`
- `2026-02-08`
- `12/02/2026`
- Unix timestamps (10 or 13 digits)

### Weights / numbers
If a column header includes `"weight"`, `"kg"`, or equals `"wgt"`, leading zeros are trimmed.

---

## Notes on Mobile UI

- On mobile, Serial is hidden and BLE is enforced.
- Connection details are collapsed by default to save screen space.
- Modal editor uses a responsive grid (2 columns desktop, 1 column mobile).

---

## Troubleshooting

### “Web Bluetooth not supported”
- Use Chrome/Edge
- On desktop, check OS/browser support
- On Android, ensure Bluetooth is enabled

### “Web Serial not supported”
- Must use Chrome/Edge **desktop**
- If on mobile, Serial is intentionally disabled/hidden

### Connect succeeds but no tasks appear
- Reader might not support the task commands
- Check that notifications/data are arriving
- Reconnect and try again

### Only part of data loads / timeout
- The app uses a 12s command timeout
- Some readers may respond slowly; increase the timeout in code if needed

---

## Customisation

- Change Serial baud rate:
  ```js
  const SERIAL_BAUD = 9600;
  ```

- Increase preview rows:
 ```js
    const rowsToShow = taskRows.slice(0, 200);
 ```