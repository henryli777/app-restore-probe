# Proposed MVP

**Status: Planning — no executable release yet.** This document specifies planned behavior, acceptance gates, and unresolved decisions. No implementation, supported version combination, or successful Docker integration run is claimed.

## Purpose and audience

The proposed users are self-hosted application owners and small teams who already produce backups but rarely verify that a recovered application can perform a useful operation. The first question is: can this export reconstruct the selected application state so an existing document's database-backed metadata and original file can be read together?

[Paperless-ngx](https://github.com/paperless-ngx/paperless-ngx) is the proposed initial target until a release selects and verifies a version combination. The project is independent and has no affiliation with or endorsement by Paperless-ngx.

The MVP would reuse the application's official restore tooling. Generic restoration, isolation, checks, and reports already exist in [restore-drill](https://github.com/RamazanKara/restore-drill); [backupdrill](https://github.com/backupdrill/cli) already covers Supabase. Reuse or integration must be evaluated before implementing shared restore machinery. The proposed additional value is a narrow application probe that joins restored metadata, original-file bytes, and an authenticated read operation.

## Scope and prerequisites

The proposed implementation language is Python 3.11+, with Docker Compose orchestration. The first experiment must select one exact Paperless-ngx exporter/importer version, database engine and version, image digest, Docker Engine version, and Compose version. Those choices are open; no combination is supported today.

Before implementing the restore orchestrator, prepare a reproducible synthetic fixture using that official application version and confirm its documented import behavior. The fixture must already contain the canary before export. Adding a new document after recovery would not prove that the exported document recovered correctly.

The planned initial recovery input is an official Paperless-ngx document-exporter directory. The selected version's `document_importer` would reconstruct application data in a fresh database and media store. This checks logical recovery from that export, not a raw SQL dump, database volume snapshot, production configuration backup, or bit-for-bit recreation of the source database. Export coverage and omissions must be documented for the selected version.

## Inputs

| Input | Planned contract |
| --- | --- |
| Export directory | A local copy of a complete official document export from the selected version, including the files required by that version's importer. It is mounted read-only or copied into a run-owned staging directory; the source is never written. |
| Probe manifest | A local, versioned manifest that names the source application/exporter version, declares fixture provenance, identifies one existing canary unambiguously, and specifies expected original-file hash and selected metadata. |
| Version selection | The exact tested version combination and pinned image identities. Unknown or mismatched combinations must be rejected before restore, without a best-effort fallback. |
| Test environment | An explicitly selected dedicated test host or test Docker daemon with sufficient resources. Production application endpoints, credentials, paths, volumes, networks, and Compose files are outside the input contract. |
| Runtime budget | Explicit deadlines and CPU, memory, disk, and log budgets. Proposed initial values are below; release defaults must be measured and verified. |

The manifest schema is not finalized. Its required semantic fields are a schema version, source application/exporter identity, source export identity, canary selector, expected original-file SHA-256, expected metadata, and the scope of each expectation. Expected tags must use defined name or identity matching rules. The canary selector must return exactly one document after import; source database IDs cannot be assumed stable across versions without fixture evidence.

Expectations should be recorded before the drill. Deriving the expected hash only from the restored download would make the check meaningless. An export hash captured at drill time can identify the input but cannot prove that the backup predates corruption. The first fixture must retain an independently recorded canary expectation.

Input validation must reject malformed manifests, paths escaping the export root, unexpected symlinks, missing required files, oversized inputs, duplicate or ambiguous canary expectations, and incompatible versions. Input documents are potentially private and untrusted; they must not select executable hooks, arbitrary host mounts, or Docker resource names.

## Planned workflow

1. **Preflight:** validate the manifest, export paths and size, chosen versions, test daemon identity, available disk, and runtime budgets. Preflight failure must create no application resources.
2. **Allocate:** generate a unique run ID, run directory, Compose project, resource names, and temporary sandbox credentials. Pin the chosen images; generate the Compose file from a reviewed internal template.
3. **Restore:** start only run-owned services, invoke the chosen version's official importer, and record its outcome. Preserve a bounded, redacted diagnostic log. Do not infer success from a container merely running.
4. **Check readiness:** confirm database and application readiness within deadlines. An HTTP response alone does not demonstrate restoration.
5. **Probe metadata:** authenticate only to the loopback sandbox API and read the existing canary's restored database-backed record. Compare the selected metadata and tags with the manifest. Missing or ambiguous results fail the check.
6. **Probe the operation and file:** use the application's API to download that existing canary's original file, then compare its SHA-256 with the independently recorded expectation. This authenticated document read/download is the proposed business operation. Endpoint and response semantics remain subject to the chosen-version fixture.
7. **Clean up and report:** remove only verified run-owned resources, then write the final report with cleanup outcome. If cleanup is incomplete, retain a resource inventory and instructions scoped to those resources; do not issue broad removal commands.

The API read checks a restored database-backed record; it is not a full database consistency audit. Original-file validation does not cover every generated thumbnail, OCR artifact, archived rendition, or document. These omissions must be explicit in the report.

## Planned modules

| Module | Responsibility and boundary |
| --- | --- |
| Input validator | Validate manifest fields, paths, size, provenance, and version selection; return normalized read-only input descriptions. |
| Version profile | Describe one tested image/database/Compose combination and its official export/import and API contracts. No guessing for unknown versions. |
| Environment manager | Generate run-scoped Compose resources, apply budgets, verify resource ownership, and perform bounded cleanup. It must not accept a user's Compose file. |
| Restore adapter | Invoke the chosen official importer and report readiness and import outcomes without inventing a generic backup engine. |
| Paperless probe | Resolve one existing canary, check selected restored metadata, perform an authenticated original-file read, and return expected/observed results. |
| Report writer | Serialize check evidence, timing, versions, failures, and cleanup status into JSON and Markdown with redaction. It cannot change a failed check into a pass. |

No CLI command names, flags, package distribution, or module paths have been finalized.

## Outputs and result semantics

The planned outputs are a machine-readable JSON report, a readable Markdown report, and a small run inventory for diagnostics and cleanup. Reports must include:

- A report schema version, run ID, UTC timestamps, and monotonic stage durations.
- Input identity and declared provenance, with an explicit distinction between declared and verified facts.
- Exact application, image, database, Docker Engine, and Compose versions used.
- A check list with expected/observed values or evidence references and one of `pass`, `fail`, or `incomplete` per check.
- Restore, readiness, canary metadata, original-file, and business-read outcomes as separate checks.
- Budget settings, available resource observations, failures, and final cleanup status. Unavailable observations must be marked unknown.
- The checks' scope and omissions, including that one canary is not a full-data or full-business validation.

A verification result may be `pass` only when every required restore and probe check ran and passed. An inconclusive or unexecuted check must be `incomplete`, never a pass. The final CLI outcome must be unsuccessful if verification fails, is incomplete, or required cleanup fails; verification and cleanup must remain separately visible. Exact exit codes are a release-contract decision.

Reports and default logs must exclude API tokens, passwords, raw document bytes, and unnecessary document content or metadata. Canary identifiers and hashes can still be sensitive; the reporting policy and redaction defaults must be reviewed before pilot use. Report files require restrictive local permissions.

A canary pass does not establish that all business functions are correct, every file is intact, retention policies work, recovery objectives are met, or the source deployment is disaster-recovery certified. Elapsed time measures this drill on this test environment; it is not a guaranteed production recovery time.

## Errors and recovery from interruption

| Failure | Planned behavior |
| --- | --- |
| Invalid input or unknown version | Fail preflight with an actionable input error; start no application resources. |
| Image retrieval, Docker, startup, or importer failure | Mark the affected stage failed, mark dependent checks incomplete, and attempt only ownership-verified cleanup. |
| Readiness deadline or resource budget exceeded | Stop the run-owned services within a bounded shutdown period; record the deadline/budget and dependent incomplete checks. |
| Missing canary, ambiguous selector, metadata mismatch, or hash mismatch | Mark the corresponding check failed with redacted expected/observed evidence. A successful import must not hide this failure. |
| API authentication or download error | Identify the operation that failed; do not substitute direct file access and report the API read as passed. |
| Cancellation or process interruption | Preserve a minimal run inventory, attempt bounded cleanup, and mark unexecuted checks incomplete. |
| Cleanup failure or lost daemon connection | Report the verified remaining resources and unsuccessful final outcome. Do not escalate to global pruning or removal of unrelated resources. |
| Report-writing failure | Return an unsuccessful outcome; retain a minimal redacted diagnostic record when possible. Missing output is not successful verification. |

The tool should not automatically retry a restore into an existing partially imported database. A retry requires a new run ID and fresh resources. Recovery from a crashed process must inspect recorded and actual ownership before touching any resources.

## Isolation and safety boundaries

These are planned release requirements, not claims about an existing implementation:

- Require a dedicated test host or test daemon explicitly selected by the operator. Validate daemon identity before resource creation, and do not silently select a remote or production context.
- Generate a collision-resistant run ID and unique names for every container, Compose project, network, and volume. Do not reuse a stopped container or existing volume. Reject a name collision.
- Keep ownership labels plus a run inventory. Before cleanup, verify actual resources match the run ID, generated project, and recorded identities; a matching name alone is insufficient. Skip and report ambiguous resources.
- Use only a generated internal Compose template. Never accept, modify, start, stop, or use a user-supplied production Compose file for cleanup. Never perform global container, network, or volume pruning.
- Bind any exposed API/application port explicitly to `127.0.0.1` (or verified `::1` behavior if later selected), using a run-specific available port. Do not bind to all interfaces. Sandbox probes may address only the run's loopback endpoint.
- Do not mount production directories or Docker's control socket inside application containers. Keep the export input read-only, use only run-owned writable paths and volumes, and never use production credentials for the probe.
- Limit application-container network access to what the chosen recovery workflow requires; prevent contact with production application endpoints. Image acquisition is an explicit preflight activity, not permission for recovered services to run production integrations.
- Disable or isolate application integrations, outgoing email, webhooks, and scheduled actions in the selected profile. Confirm behavior with the fixture before accepting real copied exports; if that cannot be verified, do not run the recovery profile.
- Use unprivileged application containers where the chosen official image permits it; require no privileged containers or host networking. Review any necessary exception before the profile is released.
- Use restrictive permissions and redact diagnostics. Deleting local volumes is not a claim of cryptographic erasure.

Docker containers share a kernel and host resources. This workflow is not a security sandbox for deliberately hostile exports, and network/resource isolation must be tested. The dedicated test environment requirement also addresses pressure on production capacity.

## Proposed resource budget

The following are starting targets for a small synthetic fixture, not measured defaults or promises for a real deployment:

| Resource | Proposed initial target |
| --- | --- |
| Export input | At most 1 GiB; reject larger inputs before provisioning in the initial experiment. |
| CPU | At most 2 CPU cores in total across run-owned application services, allocated with verified per-service limits. |
| Memory | At most 4 GiB total across run-owned application services, allocated with verified per-service limits. |
| Disk | Require at least 20 GiB free in the dedicated test environment; target at most 10 GiB for run-owned data, staging, and diagnostics. |
| Logs | At most 100 MiB per service, with bounded rotation and redaction. |
| Time | At most 20 minutes for the drill, with explicit sub-stage deadlines and a separate bounded cleanup deadline. |

Version selection and fixture measurements may change these values. Container CPU/memory limits must be checked on the actual supported Compose combination; declarative settings alone are insufficient evidence of enforcement. Disk enforcement needs an explicit strategy: free-space preflight and periodic measurement cannot guarantee a hard quota. The first release must document the tested enforcement mechanism or state that the disk target is monitored rather than hard-enforced. No run should proceed without adequate dedicated capacity.

## Test plan and acceptance evidence

All tests below are planned. No tests have been executed by this documentation-only project setup.

| Layer | Planned evidence |
| --- | --- |
| Input and manifest tests | Malformed manifests, missing files, path traversal, symlink escape, oversize input, ambiguous selectors, and unknown versions are rejected before provisioning. |
| Report tests | Failed/incomplete checks never become passes; credentials and document content are absent; cleanup failure changes final outcome. |
| Orchestration tests | A generated Compose model uses unique resources and explicit loopback binding; only ownership-matching resources are eligible for cleanup; deadlines and interruption state are preserved. |
| Real Docker integration | Import the synthetic official export using the exact chosen images; resolve the existing canary through the API; compare metadata and downloaded original-file hash; produce the actual report. |
| Failure integration | Exercise corrupt/missing canary data, metadata/hash mismatch, authentication failure, readiness timeout, partial import, resource exhaustion, cancellation, and cleanup failure. |
| Isolation integration | Run concurrent drills; keep unrelated test resources present; verify no name/volume reuse, no non-loopback exposed port, no input mutation, and no modification of unrelated resources. Verify integration suppression and resource limits. |
| Pilot recovery | Use authorized copies of real exports only in dedicated test environments after fixture gates pass; record tested sizes, versions, outcomes, and limits without publishing private files. |

Acceptance requires actual Docker integration evidence for one exact combination, including cleanup and isolation checks. If Docker integration is unavailable, unit/static checks may be reported with their actual results, but restoration, isolation, and end-to-end success must be marked untested. Do not call a version supported or show a successful demonstration report without the run evidence.

## Non-goals

- Creating, scheduling, encrypting, rotating, or transporting production backups.
- Restoring over production, altering a running deployment, or accepting arbitrary Compose environments.
- Supporting raw database dumps, arbitrary filesystem snapshots, multiple applications, or multiple version combinations in the first release.
- Validating every document, user permission, generated artifact, integration, or business operation.
- A universal database recovery engine, a hosted service, a dashboard, or automatic incident remediation.
- Disaster-recovery certification, guaranteed recovery time, guaranteed data completeness, or a security sandbox for malicious input.

## Open decisions

1. Which exact Paperless-ngx/database/image/Engine/Compose combination should the first fixture and release use? What does the official export omit for that version?
2. Which canary selector remains unique after import, and which API endpoints and authentication method can reliably read metadata and original bytes for that combination?
3. Should tags be compared as an exact set or a required subset? Which other metadata adds useful evidence without expanding the first probe?
4. How should expectations be recorded independently before export, and how should fixture/export provenance be represented?
5. Which generic orchestration or reporting components can be reused, and what are the licensing, compatibility, and maintenance costs of that choice?
6. How can the selected profile prevent recovered integrations from contacting production systems, and how will this be proven before using real copied data?
7. Which disk-budget enforcement mechanism is practical on the chosen dedicated test platform? What do real fixture measurements imply for default limits?
8. What report redaction, temporary-data retention, exit codes, and cleanup-recovery contract should the first release expose?
9. Can pilot users repeat the drill, and does this read operation replace a step they otherwise leave undone?
