##### Abusing Strava for C2

Attackers can exploit Strava's activity API to hide C2 commands within athletic activity data, blending malicious traffic with legitimate fitness tracking.

![strava](doc/strava_c2.png)

#### How It Works

The [ConceptC2](https://github.com/BongoKnight/ConceptC2) project demonstrates C2 over the **Strava API** (`strava.com/api/v3/`). Commands and responses are embedded in activity descriptions or metadata - fields normally used for workout notes.

**C2 Flow:** The operator authenticates via OAuth, creates or updates Strava activities with command strings in the description field. The agent polls activities via `GET /api/v3/activities`, retrieves commands from activity metadata, executes them, and writes results back via `PUT /api/v3/activities/*`.

#### Why It's Hard to Detect

- Strava API traffic is HTTPS to a legitimate fitness platform
- Activity updates with text descriptions are normal Strava behavior
- The API is used by countless fitness integrations and wearable devices

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser/non-fitness processes making OAuth and API calls to `strava.com`
2. **Activity content:** Activity descriptions containing encoded strings rather than normal workout notes
