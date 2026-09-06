<!-- altitude:generated from your journey — local edits don't sync; park ideas in a lesson or edit on the web -->

# Klipper Triage Service: A Log-In, Fix-Out Pipeline

A service that accepts a Klipper log — pasted, uploaded, or posted by a bot-style endpoint — and returns a single concrete fix with the reasoning behind it. A cheap model scans the log for the failing signature, a retrieval layer built from Klipper docs, Voron Discord threads, and your own saved logs supplies the domain knowledge, and the service answers with 'check the crimps on your MCU cable' plus why 'Timer too close' usually means exactly that. A thin web page exists only to demonstrate it: drop a log, watch the pipeline stages, read the verdict.

## Locked decisions

## Build brief
### Stack
- Cheap/low-powered LLM
- RAG over Klipper/Voron domain knowledge (guides, Discord threads, logs)
- Log upload flow
- Background job pipeline
- Vector store for retrieval
- Self-hostable single-container deploy
### Features
- Drop in a log and get one concrete fix back
- Shows which known failure signature it matched
- Cites the guide or thread the answer came from

## 00 · A repo, a runtime, and a pile of real logs
The project exists on disk and in Git, runs on a reproducible Python environment, and holds your saved Klipper logs as the corpus everything else will be tested against.
- [ ] Set up the project directory and first commit ← you are here
  You can navigate the repo from the terminal, and `git log` shows an initial commit containing your folder layout and a README stating what the service does.
- [ ] Establish a branch-and-push workflow
  Work happens on branches, diffs are read before staging, and the repo is pushed to a remote you can clone onto another machine.
- [ ] Stand up a reproducible Python environment
  A fresh clone plus one install command gives a working interpreter and pinned dependencies; you can open a REPL and load a saved log file.
- [ ] Build a small library module for log handling
  A `logs` module with importable functions reads a log from disk and reports line count and time span, raising a clear error on an unreadable file.
- [ ] Record the two decisions you already made
  Two short decision records in the repo explain why a small model plus retrieval beats a large model reasoning cold here, and why the corpus comes from Voron sources and your own logs — each with the cost you accept.

## 01 · Getting the failure out of the log
Given any log in your collection, the code finds the actual failing line and names the signature it matched, instead of dumping the whole file at a model.
- [ ] Extract events from line-oriented Klipper output
  A parser walks a real log line by line and returns timestamped events, shutdown blocks, and the final error line, tested against three different logs.
- [ ] Give parsed logs a typed shape
  Parsed output becomes typed objects with named fields rather than loose dicts, so downstream code fails loudly when a field is missing.
- [ ] Write the known-failure signature catalog
  A validated config file holds signatures like 'Timer too close' with matching rules and a canonical failure name; loading a malformed entry is rejected with a readable message.
- [ ] Lock the parser down with tests over your log corpus
  A pytest suite streams each saved log through the parser and asserts the expected signature, so future changes can't silently break matching.

## 02 · A place to keep logs, chunks, and answers
The service has a real database with a schema, migrations, and seed data, so runs survive restarts and can be re-queried.
- [ ] Model logs and findings as data
  You can describe a stored log, its parsed findings, and a produced verdict as JSON structures and explain why each field exists.
- [ ] Query your first tables
  A local database holds uploaded logs; you can SELECT the last ten and read their signatures from the command line.
- [ ] Design the schema and make writes safe to repeat
  Re-ingesting the same log twice leaves one row, not two, and lookups by signature use an index instead of a full scan.
- [ ] Add keyword search over the corpus
  A file-backed database supports full-text search across log excerpts and guide text, driven only by parameterized queries.
- [ ] Make the database reproducible for anyone who clones it
  Migrations build the schema from scratch and a seed command loads a handful of sample logs, with results in a stable, tie-broken order.

## 03 · Talking to a cheap model without trusting it
The project can call a small hosted model through a typed client with keys, quotas, and a strict boundary you control.
- [ ] Wire the toolchain and configuration
  A dev server runs, secrets live in environment variables rather than source, and modules import cleanly across the project.
- [ ] Type the edges of the system
  TypeScript is configured for the demo surface and shared request shapes are described with unions instead of loose strings.
- [ ] Narrow untrusted values at the boundary
  A guard function turns an `unknown` model response into a checked object, or rejects it, with tests for both paths.
- [ ] Build a client for the model provider
  An async client sends a request to the model endpoint and returns a parsed, validated result; a fake response exercises the failure path.
- [ ] Handle keys, headers, and quotas honestly
  Calls carry the right auth header, key rotation needs no code change, and hitting a quota produces a clear message instead of a stack trace.

## 04 · The knowledge that lives in a few people's heads
Klipper docs, Voron threads, and your own logs become a searchable retrieval corpus that answers 'what does this error usually mean'.
- [ ] Embed and index domain chunks
  Guide text is split into retrievable chunks, embedded, and searchable by similarity; a query about 'Timer too close' returns wiring-related chunks near the top.
- [ ] Ingest messy sources into one corpus
  An ingest command pulls text from saved HTML guides and exported threads, normalizes it, and can be re-run without duplicating entries.
- [ ] Combine keyword and vector retrieval
  One retrieval call fuses full-text hits with similarity hits into a single ranked list, and a first prompt is grounded in those retrieved passages.
- [ ] Force the model into a fixed answer shape
  The model returns a JSON verdict with fix, reason, and sources; malformed output is caught rather than crashing the request.
- [ ] Set up a debugging loop you can trust
  When a verdict is wrong you can reproduce it from the saved log, inspect each pipeline stage, and read the error that led there.

## 05 · One concrete fix, or an honest 'I don't know'
The pipeline produces a single cited fix when the evidence supports it and visibly refuses when it doesn't.
- [ ] Score model answers against a graded set
  A dozen logs with known correct fixes form an evaluation set; a test run reports how many verdicts the pipeline gets right today.
- [ ] Measure and tune retrieval itself
  Retrieval quality is measured separately from answer quality, and you can state which model size the extraction step actually needs.
- [ ] Separate hard signature matches from soft evidence
  A matched catalog signature filters candidates deterministically while similarity contributes a score, and you can show the score breakdown for any log.
- [ ] Cite the guide or thread the answer came from
  Every verdict names its supporting sources, and a fix with no supporting passage cannot be returned as confident.
- [ ] Teach it to refuse
  A tuned confidence threshold makes the service answer 'this failure isn't one I recognize' on unseen logs, and your evaluation set proves it refuses more often than it guesses wrong.

## 06 · A drop-a-log page and a bot-callable endpoint
Someone other than you can use the service: paste or upload a log in a browser, or POST one from a Discord bot.
- [ ] Build the thin demo surface
  A single page renders pipeline stages and the final verdict from component state, updating as the result arrives.
- [ ] Define the service's URLs
  The page has a route per submitted log, the API exposes a triage endpoint, and status codes distinguish 'bad log', 'not recognized', and 'server broke'.
- [ ] Accept a pasted or uploaded log file
  A multi-megabyte klippy.log uploads successfully and reaches the parser; oversized files are rejected before anything is read into memory.
- [ ] Strip anything private before the model sees it
  Request and response bodies are declared as validated models, and API keys, tokens, and Wi-Fi details in a pasted log are redacted before any third-party call.
- [ ] Defend the prompt against the log's own contents
  A log containing text that reads like instructions cannot steer the verdict, and oversized logs are budgeted down to the relevant window instead of being truncated blindly.

## 07 · Replaying a whole folder of logs
Triage runs as a background pipeline you can queue, resume, trace, and compare between runs — the infrastructure part you'd actually defend.
- [ ] Run the service as a real process with a connection pool
  The app starts as a managed process, holds database connections properly, and shuts down without leaving work half-written.
- [ ] Make failures visible and survivable
  Errors are logged with the run they belong to, provider timeouts retry with backoff, and a nightly re-index job runs on a schedule.
- [ ] Queue a folder of logs for batch triage
  Pointing the service at a directory of saved logs enqueues one job per file, and each job's stages are traced so you can see where a verdict came from.
- [ ] Make long runs resumable and comparable
  Killing the worker mid-batch and restarting resumes where it stopped, and each run writes a manifest so you can diff yesterday's verdicts against today's.
- [ ] Report how long triage actually takes
  Stored timestamps yield per-stage latency and throughput for a batch, so you can say what the cheap model costs you in seconds.

## 08 · Live, self-hostable, and demonstrable
The service runs at a public URL, ships as a container anyone in the Voron community can self-host, and you can demonstrate both its correct answers and its refusals.
- [ ] Put the whole pipeline in a container
  One container image runs the app, and a compose setup brings up the database and a local inference option so the service can run without a hosted provider.
- [ ] Test the critical path end to end
  An integration test posts a real log to the running app against an isolated test database and asserts the exact verdict, plus a regression test for a bug you already fixed.
- [ ] Deploy to a URL you can share
  The service is live over HTTPS on a domain you chose, and pushing to main runs the test suite before anything ships.
- [ ] Issue keys for the bot endpoint and publish the release
  A Discord bot can call the triage endpoint with an issued key that you can revoke, and a tagged release with a pull-and-run command lets someone self-host it.
- [ ] Run the graduation demo
  You walk someone through a curl call from the terminal, a browser upload that produces a cited fix, and a log the service correctly refuses — then reproduce and explain one wrong verdict end to end.
