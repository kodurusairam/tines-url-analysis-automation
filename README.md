# Tines URL Analysis Automation

A SOAR workflow built in **Tines** that automates URL investigation using the **URLScan.io** API. Designed to replicate a real SOC analyst's URL triage process — extract, enrich, and verdict — without manual steps.

---

## Workflow Overview

```
Webhook (Submit URL)
        ↓
Event Transform — Extract domain from URL
        ↓
Condition — Check if domain is known good
        ↓                        ↓
  [Known Good]              [Unknown Domain]
  Build result                    ↓
  (skip scan)         HTTP Request — Submit URL privately to URLScan.io
                                ↓
                      Condition — Successful submit?
                          ↓                  ↓
                     [Yes]               [No — Build error result]
                       ↓
              Event Transform — Delay for scan to complete
                       ↓
              HTTP Request — Retrieve result from URLScan.io
                       ↓
              HTTP Request — Download screenshot from URLScan.io
                       ↓
              Event Transform — Build enriched investigation output
```

---

## How It Works

1. **Webhook trigger** — An analyst or upstream alert sends a URL via HTTP POST to the Tines webhook
2. **Domain extraction** — An Event Transform parses the domain from the submitted URL
3. **Known-good check** — A Condition action checks if the domain matches a known-safe list; if yes, the scan is skipped and a clean verdict is returned immediately
4. **URLScan submission** — If unknown, the URL is submitted **privately** to URLScan.io via API (private flag ensures it doesn't appear in public URLScan results)
5. **Submit validation** — A Condition checks the API response; if submission failed, an error result is built and the story exits gracefully
6. **Delay** — The workflow waits for URLScan to complete its analysis before polling for results
7. **Result retrieval** — The scan result is fetched using the unique scan ID returned at submission
8. **Screenshot download** — The visual screenshot captured by URLScan is pulled and stored with the event
9. **Output** — A final Event Transform assembles the enriched data including verdict, threat indicators, and screenshot for analyst review

---

## Sample Webhook Payload

```json
{
  "url": "http://suspicious-domain.xyz/login"
}
```

---

## Tech Stack

| Component | Purpose |
|---|---|
| Tines | SOAR platform — workflow orchestration |
| URLScan.io API | URL scanning, result retrieval, screenshot |
| Webhook | Trigger point for incoming URL submissions |
| HTTP Request actions | API calls to URLScan |
| Event Transform | Data parsing and output formatting |
| Condition actions | Branching logic (known-good bypass, error handling) |

---

## Key SOC Concepts Demonstrated

- **IOC enrichment** — automated URL/domain investigation without analyst manually visiting URLScan
- **Conditional branching** — known-good bypass reduces noise and unnecessary API calls
- **Private scanning** — URLScan private flag prevents tipping off threat actors
- **Async workflow handling** — delay action accounts for real-world API processing time
- **Error handling** — failed submissions are caught and handled gracefully, not silently dropped
- **Evidence collection** — screenshot captured automatically as part of investigation artifact

---

## Screenshots

### Full Workflow
![Workflow](screenshots/01_Workflow.png)

### URLScan Execution — Live Event Data
![URLScan Execution](screenshots/02_URLScan_Execution.png)

---

## Credentials Required

| Credential Name | Service |
|---|---|
| `CREDENTIAL.urlscan_id` | URLScan.io API Key |

---

## Author

**Koduru Sairam** — SOC Analyst  
[LinkedIn](https://linkedin.com/in/kodurusairam) | [GitHub](https://github.com/kodurusairam)
