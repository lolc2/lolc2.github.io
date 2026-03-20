##### Abusing AWS X-Ray for C2

Attackers can exploit AWS X-Ray's tracing and telemetry APIs to establish a covert C2 channel by encoding commands in trace segment annotations and metadata, hiding within legitimate cloud observability infrastructure.

#### How It Works

The [XRayC2](https://github.com/RootUp/XRayC2) project demonstrates how AWS X-Ray can be weaponized as a serverless C2 framework.

**Trace-Based C2:** AWS X-Ray is a distributed tracing service used to monitor application performance. XRayC2 abuses its APIs to pass commands between operator and agent. Commands are encoded in X-Ray **subsegment annotations and metadata** - fields normally used to store application telemetry like response times and error codes.

**C2 Flow:** The operator writes commands by calling `PutTraceSegments` with specially crafted trace data containing the command in annotation fields. The agent polls for new traces using `GetTraceSummaries` and `BatchGetTraces`, extracts commands from the annotations/metadata, executes them, and writes results back as new trace segments via `PutTraceSegments`.

**Serverless Architecture:** No dedicated C2 server is needed. AWS X-Ray's managed infrastructure handles all data storage and retrieval. The operator and agent communicate entirely through AWS's own telemetry service.

**Authentication:** Both operator and agent need valid AWS credentials with X-Ray API permissions (`xray:PutTraceSegments`, `xray:GetTraceSummaries`, `xray:BatchGetTraces`). In compromised AWS environments, these permissions may already be available via instance roles or developer credentials.

#### Why It's Hard to Detect

- X-Ray API calls go to `xray.*.amazonaws.com` - a legitimate AWS service endpoint used by observability teams
- Trace segment data looks like normal application telemetry, with annotations and metadata being standard fields
- Many AWS environments have X-Ray enabled by default for application monitoring, making its API traffic expected
- The channel uses standard AWS SDK calls, indistinguishable from legitimate X-Ray instrumentation
- X-Ray data is ephemeral (traces expire based on retention settings), limiting forensic recovery

#### Key Detection Opportunities

1. **IAM anomalies:** X-Ray API calls (`PutTraceSegments`, `BatchGetTraces`) from IAM principals not associated with application monitoring or observability teams
2. **CloudTrail monitoring:** CloudTrail logs all X-Ray API activity - unexpected principals or unusual call patterns should trigger alerts
3. **Annotation content:** Trace segment annotations containing encoded strings, base64 data, or structured command patterns rather than legitimate telemetry values
4. **Non-instrumented callers:** Direct X-Ray API calls from applications that don't have the X-Ray SDK integrated - indicates raw API abuse
5. **Spike detection:** Anomalous spikes in `PutTraceSegments` or `GetTrace` volume from a single principal, especially at regular intervals

**Source:** [Ghost in the Cloud - Weaponizing AWS X-Ray for C2](https://medium.com/@dhiraj_mishra/ghost-in-the-cloud-weaponizing-aws-x-ray-for-command-control-7539d60f1d77)
