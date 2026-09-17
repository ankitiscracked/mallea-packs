# Pack publishing contract

This repository contains distributable pack source only. Never add stores, user
data, grants, credentials, recordings, runtime profiles, or exported personal apps.

Use the pinned Mallea runtime and `tools/packs/publish.ts` for validation,
submission, and publishing. Humans and agents use the same commands and gates.
Do not mark a prepared ZIP or a submitted PR as published. Published means the
release download exists and the catalog references its SHA-256.

Versions are immutable MAJOR.MINOR.PATCH. Add a new folder and release entry for
changes. Keep earlier versions intact. Publishing must upload every validated ZIP
before replacing catalog.json. Never bypass failed validation.

`release` runs only on trusted main. External contributions require maintainer
review before a branch can access the private runtime checkout key. Never use
pull_request_target to execute contributed workflows or scripts.
