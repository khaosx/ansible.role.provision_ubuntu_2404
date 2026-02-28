# Changelog

## [Unreleased]
- BREAKING: role task layout refactored to import-only `tasks/main.yml` with component task files.
- BREAKING: handler names changed to `provision_ubuntu_2404 | ...` convention; external notify references must match.
- Added: `meta/argument_specs.yml` with validation for all public defaults.
- Added: canonical standalone-role `.gitignore`.
- Added: `verify.yml` for standalone role verification.
- Added: standards-aligned README sections (capabilities, artifacts, idempotency, rollback, platform contract).
- Changed: task naming normalized to `Area | Action` stylebook format.
- Changed: fact access normalized to bracket notation for stylebook compliance.
