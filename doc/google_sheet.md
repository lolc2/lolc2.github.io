##### Abusing Google Sheets for C2

Attackers can exploit Google Sheets' API to embed commands in spreadsheet cells and retrieve results, making C2 traffic indistinguishable from legitimate spreadsheet collaboration.

![google sheet](/doc/google_sheet.png)

#### How It Works

Google Sheets-based C2 uses the **Sheets API** (`sheets.googleapis.com`). Specific cells serve as the command and response channels.

**C2 Flow:** The operator writes commands to designated cells via `batchUpdate` or `values` endpoints. The agent polls the spreadsheet via `values:batchGet`, reads commands from designated cells, executes them, and writes results back to response cells. The entire C2 console is a shared Google Spreadsheet.

**GC2-sheet** combines Google Sheets with Google Drive for a complete C2 framework, using sheets for tasking and Drive for file staging. The [looCiprian/GC2-sheet](https://github.com/looCiprian/GC2-sheet) project demonstrates both channels working together.

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser processes making API calls to `sheets.googleapis.com`
2. **Polling cadence:** Periodic `batchGet` or `values` requests to the same spreadsheet at fixed intervals
3. **Cell content:** Spreadsheet cells containing encoded or structured command/response data
