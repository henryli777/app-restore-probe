# Contributing

**Planning — no executable release yet.** Contributions currently focus on the proposed recovery scenario, reproducible synthetic fixtures, safety requirements, and review of [the MVP scope](docs/MVP.md). There is no verified installation or test command to run yet.

## Useful early contributions

- Describe an existing backup workflow and the application operation you would want a recovery drill to verify. Do not post real document contents, credentials, internal addresses, or production exports.
- Help select one exact Paperless-ngx exporter/importer, database, image, Docker Engine, and Compose combination. Link to version-specific official documentation and distinguish documented behavior from behavior you actually tested.
- Propose a reproducible synthetic fixture whose canary document exists before export, with independently recorded file hash and metadata expectations.
- Review possible reuse of [restore-drill](https://github.com/RamazanKara/restore-drill), [backupdrill](https://github.com/backupdrill/cli), and official application tools before proposing generic restore infrastructure.
- Review resource ownership, loopback binding, integration suppression, secret redaction, and bounded cleanup requirements.

Paperless-ngx remains a proposed initial target. Do not describe any version as supported until the required integration evidence exists. This project has no affiliation with or endorsement by Paperless-ngx.

## Proposed changes

Keep changes small and explain the specific problem, intended behavior, evidence, and remaining uncertainty. Align them with [the roadmap](ROADMAP.md); discuss additional applications or backup formats before broadening the MVP.

Documentation changes should label planned behavior clearly. Do not add installation commands, screenshots, badges, sample success reports, or passing-test claims unless the corresponding artifact exists and has been verified. A report example must identify whether it is a schema illustration or the output of a real run.

For future code changes, include the exact tests executed and their results. Real Docker integration must use synthetic data in a dedicated test environment. If it was not run, state that explicitly; unit or static checks cannot establish successful restore, isolation, or cleanup.

## Restore experiment boundaries

Never target production endpoints, services, credentials, Compose files, paths, volumes, or networks. Use only a dedicated test daemon or host, copied read-only inputs, unique run-owned resources, and loopback application ports. Cleanup must verify resource ownership and use only a tool-generated Compose file. It must never use a user-provided production Compose file or global pruning.

Record the fixture provenance, exact version combination, resource budget, checks performed, failures, and cleanup outcome. One successful canary read does not prove every document or business function is correct, and it is not disaster-recovery certification.

## License

Contributions are intended to be distributed under the repository's [MIT license](LICENSE). Keep attribution and comply with the licenses of any reused code or fixtures.
