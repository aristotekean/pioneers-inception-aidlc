# Contract Design Questions

Context carried forward: 11 units; the boundaries that cross a process or protocol are web-shell to identity (sign-in), web-upload to ingestion (uploads, jobs), web-layers to catalog (layers, attributes, styles, visibility, delete), web-layers and web-3d to tile-serving (tiles), web-3d to jobs-terrain3d (tile locations), and the database contract between catalog and the tile server (row policies). No API is consumed outside the system in this piece of work. Integration was confirmed as JSON over HTTP plus a shared schema. These questions pin the details every contract needs.

## Q1. How does the browser prove who it is on every request?

A. A session cookie issued at sign-in (HttpOnly, Secure, SameSite), sent automatically; the backend reads the session on every call
B. A bearer token returned at sign-in, stored by the web app and sent in an Authorization header
C. Not yet defined
X. Other (please specify)

[Answer]: B. A bearer token returned at sign-in, stored by the web app and sent in an Authorization header

## Q2. How does a tile request carry the current user to the database, so the row policies apply?

The tile server reads the database directly with one pooled connection. The mechanism must be safe under connection pooling (no identity leaking between requests).

A. A short-lived signed tile token per user, added to tile URLs as a parameter; a database function validates it and applies the user inside the same query, so nothing is set on the pooled connection
B. One database role per user and a tile server connection per role
C. The backend proxies tile requests after all, and the tile server is not exposed to the browser
D. Not yet defined
X. Other (please specify)

[Answer]: X. Other — use the simplest way for the mvp (interpreted as C: the backend serves tiles itself from the database inside the request's own transaction, where it already knows the user and the row policies apply; no separate tile server is exposed to the browser for the proof of concept)

## Q3. How are contract versions and breaking changes handled?

A. One unversioned API path for the proof of concept; only additive changes; a breaking change is made in the same commit as its consumer (single repository)
B. A version segment in the path from day one, with old versions kept until consumers move
C. Not yet defined
X. Other (please specify)

[Answer]: A. One unversioned API path for the proof of concept; only additive changes; a breaking change is made in the same commit as its consumer

## Q4. What shape do errors take across the HTTP contracts?

A. One uniform JSON error body (code, message, details) with standard status codes; authentication and authorization failures are distinguishable
B. Free-form per endpoint
C. Not yet defined
X. Other (please specify)

[Answer]: A. One uniform JSON error body (code, message, details) with standard status codes; authentication and authorization failures are distinguishable

## Q5. How does the web app learn that a background job changed state?

A. Polling the job status endpoint every few seconds while a job is running
B. A server-sent event stream of job updates
C. Not yet defined
X. Other (please specify)

[Answer]: A. Polling the job status endpoint every few seconds while a job is running

## Q6. Where does the code live?

A. One repository with backend, jobs, web and tile server configuration (contracts change together)
B. Separate repositories per deployable
C. Not yet defined
X. Other (please specify)

[Answer]: B. Separate repositories per deployable

## Q7. Follow-up: with separate repositories, a breaking change cannot ship "in the same commit as its consumer". How are breaking changes coordinated?

Q3 chose unversioned, additive-only contracts with breaking changes made together with the consumer, which assumes one repository. Q6 chose separate repositories.

A. Keep separate repositories and add a version segment to the API path; a breaking change is a new version kept alongside the old one until the web app moves
B. Keep separate repositories and forbid breaking changes for the proof of concept: additive-only, providers deploy first, consumers ignore unknown fields
C. Use one repository after all, so Q3 stands as answered
D. Not yet defined
X. Other (please specify)

[Answer]: B. Keep separate repositories and forbid breaking changes for the proof of concept: additive-only, providers deploy first, consumers ignore unknown fields

## Consolidated Summary Confirmation

- Browser identity: a bearer token returned at sign-in, sent in an Authorization header
- Tiles: the simplest way for the proof of concept, interpreted as the backend serving tiles itself inside the request transaction where the row policies already apply; no tile server exposed to the browser
- Versioning: one unversioned API path, additive-only changes, no breaking changes in the proof of concept; providers deploy first and consumers ignore unknown fields
- Errors: one uniform JSON error body (code, message, details) with standard status codes; 401 and 403 distinguishable
- Job updates: polling the job status endpoint every few seconds while a job runs
- Code: separate repositories per deployable

Does this all look correct before I generate the artifact?

- Looks correct
- Request changes

[Answer]: Looks correct
