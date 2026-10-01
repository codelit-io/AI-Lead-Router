# AI lead-routing exercise: final review

Reviewed October 1, 2026. The core exercise is demonstrated: one browser HTML file, LLM-selected category, validated category JSON, and real webhook receipts. Evidence strength varies: simulated failures, an independently recorded earlier Enterprise receipt, and user-observed recent Enterprise/Support receipts. No full-security or exactly-once-delivery claim is made.

The original one-page exercise PDF was read and visually checked. Its requirements are source material for this audit, not permission to publish the PDF or perform external actions. It requires exactly one category selected by the LLM, not an optional dropdown or manual category override. The PDF stays excluded from Git.

## Requirement audit

| Requirement | Result | Evidence and qualification |
|---|---|---|
| One HTML file with JavaScript, opened in a browser; no framework scaffolding | PASS | `lead-router.html` contains inline CSS and JavaScript. No backend, framework, package installation or external script is needed for the app. Repository evidence documents do not add application dependencies. |
| Send visitor free text to an LLM | PASS | `modelRequest` sends the trimmed message as user data, separate from instructions, to OpenAI's Responses API using `gpt-4.1-mini`. Real model calls passed. |
| LLM chooses exactly Sales, Support, Enterprise or Other | PASS within tested scope | Prompt defines all four, including the two supplied examples. JavaScript does not semantically classify visitor text. Strict JSON schema requests the exact capitalized enum. All four validator values pass simulation; Enterprise and Support were evaluated live. |
| Structured output and local validation | PASS | One JSON string pair is required before decoding, preserving duplicate-key rejection. Decoded own key must be `category`, value must be in the enum. Escaped JSON now passes. Extra fields, duplicates, arrays, invalid values, prose and fences are rejected. |
| POST validated JSON to the supplied unique webhook.site endpoint | PASS | Only category JSON is sent. UUID correlation is in `X-Request-ID`; key and visitor message are not forwarded. Destination is supplied privately at runtime and restricted to the supported HTTPS UUID endpoint. No invented/replacement webhook endpoint was used. |
| Show the request landed with its category | PASS, mixed evidence levels | Earlier Enterprise POST was independently matched in the receiver. Recent Enterprise and Support were manually observed by the user, including matching client/receiver request IDs. A page acknowledgment alone is not considered delivery proof. |
| Handle junk, failure and slowness reliably | PASS in simulation | Invalid/refused/failed/incomplete/timed-out classification produces zero webhook posts and never becomes Other. HTTP/network/body errors stop safely. Limits include 5,000 input units as measured by JavaScript/HTML, 64 KiB response bodies, 20 seconds for model and 10 seconds for webhook, within a 30-second total deadline. Timers depend on browser execution; late results are rejected when execution resumes. |
| Freeze submissions, prevent duplicate clicks and render safely | PASS in simulation/static review | In-flight fields/button are disabled, message/credentials/URL are snapshotted, each submission has a UUID, and late results are ignored. Output uses `textContent`; raw response bodies/errors and unsafe model metadata are not rendered. |
| No automatic webhook retries; honest uncertainty | PASS | Exactly one webhook fetch attempt per accepted submission. Webhook errors occur after an attempted POST and may mean delivery happened; the page reports uncertainty and requires inbox inspection before manual resend. |
| Browser usability checks | User-reported overall PASS; details limited | User confirmed manual Chrome verification passed. Detailed keyboard, narrow-viewport, native-validation and console outcomes were not separately supplied or independently captured. Static linked-label, focus-style, status-region and responsive-rule checks pass. |

The project also meets the user's stronger constraints: plain small functions, masked runtime credential fields, no persisted runtime settings in application storage, bounded waiting/body reading, no failure fallback category, synthetic inputs and an explicit development-only warning.

## Actual evidence

**Earlier independently recorded Enterprise receiver match:** see [webhook-cors-evidence.txt](webhook-cors-evidence.txt).

- Browser/OpenAI receipt: requested `gpt-4.1-mini`, HTTP 200, completed, locally validated, 181 input/6 output tokens, 2.66 seconds.
- Category: `{"category":"Enterprise"}`.
- Client and received `X-Request-ID`: `b173799d-c7bc-466c-a03e-68fd41205998`.
- Receiver POST ID: `593849ba-a79a-4622-aebc-7e6aff81b1e2`.
- Receiver observed OPTIONS followed by POST, JSON content type, origin `null`, category JSON and the matching ID.
- Receiver CORS was originally disabled. Enabling Add CORS headers on the same endpoint was followed by this successful match.

**Most recent manual evidence:** user confirmed fresh Enterprise and Support results, real receiver POSTs and matching request IDs, plus an overall Chrome verification pass. Accepted as user-observed evidence. Exact new IDs, response IDs, usage totals and header values were not supplied; none are invented. They were not independently verified by browser automation, which last failed with “Debugger unattached.”

**Offline evidence:** 150 synthetic cases across 21 groups, original 14 baseline groups, and six follow-up groups passed. These use the actual inline application script with a fake DOM, mocked fetch and controlled time. They are not live model evaluations or proof of browser rendering/CORS. Temporary harnesses remain under ignored `work/`, outside the app and proposed publication.

**Manual parser regression:** user reported `PASS 26/26 browser parser checks` from the separate offline Chrome fixture. It does not prove networking or all UI interactions.

[test-evidence.txt](test-evidence.txt) and [live-evidence.txt](live-evidence.txt) are explicitly historical snapshots. Their older pending/failed states should not be confused with the later successful evidence above.

## Defects, assumptions and remaining gaps

The confirmed escaped-JSON rejection defect is fixed and committed. For example, `{"category":"Supp\u006frt"}` is now decoded and accepted as Support. Duplicate and escaped-duplicate keys remain rejected. No new confirmed classification/delivery safety defect was found in the tested paths.

**Back-forward-cache is an unverified usability risk, not a confirmed Chrome defect.** Simulation shows that page exit while loading stops the submission but leaves its loading text and disabled controls. If Chrome restores that state from cache, it could appear stuck. Chrome eligibility/restoration has not been reproduced. Keep this deferred; fix only after Chrome reproduction and add a regression check then.

Other limits:

- The prompt's Enterprise and mixed-message priorities are assumptions expressed to the model, not local business rules. Larger evaluation sets, ambiguous cases, language coverage and prompt-injection resistance have not been established. Schema-valid does not guarantee semantically correct.
- Sales and Other have simulated validator coverage, not fresh real model evaluations. The two exercise examples have manually observed live results.
- Detailed native keyboard, mobile rendering, console and navigation-recovery evidence is not archived. Broad manual Chrome confirmation is useful but less specific than itemized observations.
- Post-dispatch errors intentionally stop the page. Reload recovery requires correctly carried-forward attempt counts. Limits are memory-only and can be mis-entered after reload.
- Actual response headers for the most recent manual runs are not recorded. Manually observed OPTIONS→POST delivery passed, but no exact header values are claimed.
- Temporary automated harnesses are excluded from publication as requested. A new clone can run the app and read the evidence, but cannot rerun those local harnesses from the proposed repository alone.

## Run steps and current budget

1. Manually open `outputs/lead-router.html` in Chrome. The title should be **Lead router - local development demo**. Do not use the separate offline fixture for a live demo.
2. Use only synthetic visitor text. Keep the exact supplied webhook endpoint private and ensure its Add CORS headers setting remains enabled.
3. For any newly authorized future live session, carry forward actual prior-use totals before the first request. Pause recording for credential/destination entry; use a non-production key, decline password saving, avoid recording DevTools/network data and revoke the key after the demo.
4. Submit once. Observe fields/button disable and private fields clear. A successful model result must pass local validation before a category-only POST is attempted.
5. Inspect the actual receiver POST: method POST, expected category JSON and received `X-Request-ID` matching the browser receipt. OPTIONS alone is preflight, and HTTP acknowledgment alone is not matching receipt proof.
6. On failure, stop. Never treat it as Other or automatically resend. Inspect the inbox before any separately authorized manual resend.

**No more live calls under this exercise budget.** Based on exactly one new Enterprise and one new Support attempt after the earlier 3/3 totals, the approved 5/5 model and 5/5 webhook budgets are inferred to be exhausted. Exact latest usage receipts were not supplied. Do not reset totals or enlarge caps to continue. A future live demo needs separate authorization.

For the walkthrough, reuse existing receipts and offline evidence; do not click Submit again. See [five-minute-walkthrough.md](five-minute-walkthrough.md).

## Production limitations

This is the explicitly approved local development BYOK demo. Masking and in-memory use do not make a browser API key secret from the browser owner, extensions, DevTools or recording software. OpenAI's [authentication documentation](https://platform.openai.com/docs/api-reference/introduction) warns against exposing ordinary API keys in client-side code. A production design would need an appropriate trusted credential boundary; that has not been added because the exercise is deliberately one file with no backend.

Input framing, strict schema, destination checks and safe rendering reduce specific risks but are not a full security audit or a guarantee against model manipulation. `store:false` is not a guarantee of zero retention across providers, infrastructure or the webhook service. Synthetic data and an ephemeral test key remain the intended scope.

This app also lacks durable global rate/budget enforcement, authenticated receiver access, a durable outbox, receiver-side idempotency enforcement, durable reconciliation and an operational delivery ledger. `X-Request-ID` is a correlation identifier, not an enforced deduplication guarantee. One attempt per submission is not exactly-once delivery; manual resubmission/reloads and ambiguous network outcomes can still create duplicates. Nothing here establishes full security, unlimited reliability or exactly-once delivery.

## Git checkpoint

- `18fb82d` — `feat: add single-file lead router with bounded delivery`.
- `fe5377bc5bcc24732f581a0688ccfee9333b0c12` — `fix: accept escaped JSON categories without weakening validation` (current HEAD).
- Current branch: `main`.
- Only pending application diff: the approved webhook cap 4→5 in the numeric input maximum, visible budget copy and runtime limit. Three inserted/three removed lines; no classification changes.
- No final commit or push has been made. The separate publication preview identifies the exact proposed destination and file scope for approval.
