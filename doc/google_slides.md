##### Abusing Google Slides for C2

Threat actors can exploit Google Slides to establish an infrastructure-less C2 channel by embedding commands within table cells of shared presentations using the Slides API.

![google slides](/doc/google_slides.png)

#### How It Works

Google Slides-based C2 uses the **Slides API** (`slides.googleapis.com`). Commands and responses are stored in presentation table cells.

**C2 Flow:** The operator writes commands to specific table cells in a shared presentation. The agent polls the presentation via `GET /v1/presentations/*`, reads commands from designated cells, executes them, and writes results back via `POST /v1/presentations/*:batchUpdate`.

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser processes making API calls to `slides.googleapis.com`
2. **Cell content:** Presentation table cells containing encoded command/response strings updated at regular intervals
