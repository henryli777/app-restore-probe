# Roadmap

**Planning — no executable release yet.** All milestones below are planned and uncompleted. Their order expresses dependencies, not delivery dates. Paperless-ngx is the proposed initial target; no version combination is supported today.

## 1. Select one recovery scenario

- [ ] Confirm the workflow with maintainers who have an existing backup but have not regularly verified application recovery.
- [ ] Evaluate reuse of [restore-drill](https://github.com/RamazanKara/restore-drill), [backupdrill](https://github.com/backupdrill/cli), and the selected application's official tools; document what must remain application-specific.
- [ ] Select and record one exact Paperless-ngx exporter/importer version, database engine and version, image digest, Docker Engine version, and Compose version.
- [ ] Verify official export/import and API behavior for that combination, including how to identify a canary after import and download the original file.
- [ ] Decide whether the proposed read operation is useful enough for the first users before expanding scope.

Gate: a written version choice and a reviewed recovery procedure. Do not label the combination supported until integration evidence exists.

## 2. Establish a reproducible fixture and input contract

- [ ] Create a synthetic Paperless-ngx fixture with an existing canary document, original file, and tags; export it using the chosen official exporter.
- [ ] Define the manifest schema, unique canary selector, expected original-file hash, metadata comparison rules, and provenance fields.
- [ ] Validate a clean official import of that fixture and record the actual results.
- [ ] Define input path restrictions and cases for missing files, symlinks, malformed manifests, ambiguous selectors, corrupt content, and mismatched versions.
- [ ] Decide initial resource defaults from measured fixture behavior.

Gate: repeatable fixture production and documented expectations. A fixture is test data, not evidence of a successful probe.

## 3. Build the isolated restore workflow

- [ ] Implement the proposed Python 3.11+ orchestration with a tool-generated Compose template.
- [ ] Require an explicitly selected dedicated test daemon or host, and verify its identity before any resource creation.
- [ ] Generate a unique run ID and unique resource names; verify ownership before cleanup.
- [ ] Keep exports read-only, bind ports only to loopback, set supported CPU/memory/log limits, and enforce stage deadlines.
- [ ] Restore using the selected official importer and wait for bounded readiness checks.
- [ ] Exercise cancellation, startup failure, import failure, and partial cleanup in the test environment.

Gate: Docker integration evidence shows that unrelated resources are preserved and only verified run-owned resources are removed. A unit-test pass alone cannot satisfy this gate.

## 4. Add the application probe and report

- [ ] Check restored database-backed canary metadata through the application API, including the chosen tag comparison rule.
- [ ] Perform an authenticated read of the existing canary and compare downloaded original-file bytes with the expected hash.
- [ ] Generate JSON and Markdown reports that distinguish pass, fail, and incomplete outcomes, and include cleanup status.
- [ ] Redact credentials and avoid including document contents in reports or logs.
- [ ] Record source/input identity, exact tested versions, timing, resource observations, and evidence limitations.

Gate: a report is produced from a real fixture run, with check-level evidence; failed and unexecuted checks cannot become passes.

## 5. Prepare the first experimental release

- [ ] Complete the test plan in [docs/MVP.md](docs/MVP.md), recording which checks were actually run and any limitations.
- [ ] Publish a compatibility matrix containing only tested combinations, plus a synthetic demonstration report.
- [ ] Add installation and usage instructions only after the CLI exists and the documented workflow has been verified.
- [ ] Review release instructions for production-contact paths, resource ownership, secret handling, and report claims.
- [ ] Obtain pilot feedback from real deployments using copies in dedicated test environments; record the tested cases without publishing private data.

Gate: reproducible installation, recovery, probe, reporting, and cleanup for one tested combination. A first release remains limited to its declared checks and does not certify disaster recovery.

## Later candidates

Scheduling, a second application adapter, other backup formats, historical comparisons, and team reporting are future possibilities. None is committed to the first release. Expansion should follow evidence that the initial probe is useful and repeatable.
