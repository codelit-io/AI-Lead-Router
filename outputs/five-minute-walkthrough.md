# Five-minute walkthrough

Use the current source and existing receiver evidence. Do not make a new live call: the approved exercise budget is inferred exhausted. Keep keys and the private webhook URL out of the recording. The three rehearsal questions follow the script.

## 0:00–1:00 — Problem and scope

“The assignment was to accept a visitor's message, let an LLM choose Sales, Support, Enterprise or Other, and show that category arriving at webhook.site. I kept the app in one HTML file with inline JavaScript and CSS. There is no framework or backend. The form has labeled inputs, a status area, the validated category and a receipt. This is a local development demo using a privately entered non-production key. I don't describe that browser-held key as production-safe. I also distinguish a model result, a webhook acknowledgment and an actual receiver receipt; each proves a different part of the flow.”

Show the form and the file structure, without entering credentials or submitting.

## 1:00–2:00 — Letting the model decide

“The prompt defines the four categories and includes the exercise's two examples. A 200-person pricing inquiry belongs to Enterprise; a missing password-reset email belongs to Support. Visitor text is sent as user data, separate from those instructions. The request uses GPT-4.1 Mini through the Responses API and asks for a strict JSON schema containing only category. JavaScript never examines the message to choose a bucket. Its job is to validate the returned response and enum. A valid Other is a model decision; an error is never silently converted into Other. The schema controls shape, but it cannot guarantee that every classification is correct.”

Show `modelRequest`, the prompt and the category validator.

## 2:00–3:00 — Reliability before side effects

“Before making a request, I validate input length, runtime destination and the approved attempt counts. I snapshot the submission, generate a request ID, clear the private fields and disable inputs while it is running. The model has a 20-second phase deadline; the webhook has ten seconds, inside a 30-second total deadline including body reading. Bodies are capped at 64 KiB. Failed, incomplete, refused, malformed or timed-out classifications send no webhook request. Late results cannot update an ended submission or trigger delivery. Errors are displayed safely as text. I also found and fixed a validator defect: valid JSON escapes were rejected. The regression accepts decoded values without allowing duplicate keys or extra fields.”

Show the escaped-JSON fix and the simulated pass/fail evidence, not a new live request.

## 3:00–4:00 — Proving the receiver boundary

“Only validated category JSON goes to the receiver. The key stays out of that request, and the visitor text is not forwarded. I put the correlation ID in X-Request-ID. Initially the browser's OPTIONS preflight arrived without a POST because CORS was disabled on the endpoint. Enabling the receiver's CORS setting led to a real matching Enterprise POST. The later Enterprise and Support runs were manually observed with their category and matching IDs. Those latest results are user-observed evidence, not automation I can claim independently. A webhook HTTP success alone is not proof of the matching receipt. After an uncertain failure, I inspect the inbox rather than automatically retry.”

Show an existing receiver POST and its matching saved client ID. Hide the private endpoint URL.

## 4:00–5:00 — Evidence, boundaries and next steps

“The offline suite covers 150 synthetic cases, the 14 baseline groups and six follow-up groups. Those passed, including junk output, slow bodies, authentication errors, double submissions, late replies and webhook uncertainty. They are simulations, not broad model-accuracy evidence. Manual Chrome verification also passed, but detailed UI observations and exact latest headers were not archived. The back-forward-cache concern is still an unverified usability risk; I won't fix it without reproducing it in Chrome. Production would need a trusted credential boundary and durable operational controls. The UUID is correlation, not exactly-once delivery. I have kept the pending budget adjustment small, reviewed Git history for obvious secret literals, and prepared the private repository destination for approval. Nothing has been created or pushed.”

## Three rehearsal questions about your decisions

1. Why did you put category definitions in the model prompt and keep JavaScript limited to validation? What does a valid schema still fail to prove?
2. Why is automatic webhook retry dangerous after a timeout, and what evidence lets you call this particular submission received?
3. Why is in-memory, masked BYOK acceptable for this approved local demo but insufficient for production, and what is the first boundary you would change?
