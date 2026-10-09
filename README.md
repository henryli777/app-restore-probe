# app-restore-probe

[简体中文](README.zh-CN.md)

**Planning — no executable release yet.** This repository contains the proposed scope and roadmap. Every feature below is planned; there is no CLI, installable package, verified compatibility matrix, or integration test result yet.

app-restore-probe aims to check whether a backup of a specific self-hosted application can restore its database-backed records, files, and one meaningful application operation together. It would reuse existing backup and restore tools, then add a small application-specific probe.

## Proposed first target

[Paperless-ngx](https://github.com/paperless-ngx/paperless-ngx) is the **proposed initial target until a release selects and verifies a version combination**. This is an independent project, with no endorsement by or affiliation with Paperless-ngx. No application, database, or Docker Compose version is currently supported.

The proposed MVP would:

1. Accept a Paperless-ngx official document-exporter directory and a manifest describing an existing canary document.
2. Validate the input and selected versions, then use Python 3.11+ to orchestrate a fresh Docker Compose environment on a dedicated test host or test daemon.
3. Use the selected version's official `document_importer` workflow to rebuild application state, then read the existing canary through the application API and download its original file to check its hash and expected metadata, such as tags.
4. Produce JSON and Markdown reports with check outcomes, evidence references, timings, failures, and cleanup status.

Version selection and a reproducible synthetic export fixture must come before restore implementation or a demonstration report. An application export is a logical recovery input; it does not verify a raw database backup or recreate every production setting.

## Isolation requirements

The planned implementation must generate a unique run ID and unique names for every container, network, and volume; expose application ports only on loopback; keep input exports read-only; and never modify production services, paths, or credentials. Cleanup must use only the tool's generated Compose file and resources whose ownership is verified for that run. A user-supplied production Compose file must never be used for cleanup.

The MVP would require a dedicated test environment and explicit resource limits. Containers share host capacity, so isolation alone does not protect a production host from resource pressure. This planning repository provides no production-operation instructions.

## What a result would mean

A canary pass would show that the listed checks succeeded for that input and version combination. It would not prove all business functions are correct, all data is intact, recovery objectives are met, or a deployment is disaster-recovery certified. Failed and incomplete checks must remain visible.

## Existing work and proposed focus

[restore-drill](https://github.com/RamazanKara/restore-drill) already covers isolated database restoration, checks, and reports. [backupdrill](https://github.com/backupdrill/cli) covers Supabase recovery drills. The proposed focus here is a narrow application-level check that connects restored records, files, and an actual read operation. Reuse or integration should be evaluated before building generic restore infrastructure.

## Project documents

- [MVP scope and safety boundaries](docs/MVP.md)
- [Roadmap and release gates](ROADMAP.md)
- [Contributing](CONTRIBUTING.md)
- [MIT license](LICENSE)
