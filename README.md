# Mallea packs

Versioned pack sources, validation gates, and a static distribution catalog.
GitHub Releases is the first storage backend; clients consume ordinary HTTPS.

Each release lives in `packs/<id>/<version>/`. `releases.json` declares its ID,
version, description, and minimum Mallea version. Pack definitions stay in their
existing `theme.jsonc`, `keymap.jsonc`, `chrome.jsonc`, or `app.jsonc` format. App
TypeScript sources travel together with the definition in one ZIP.

## Validate and publish

Use Bun 1.4.0 and a sibling Mallea checkout at the commit pinned in
`.github/workflows/packs.yml`:

```sh
bun run ../mallea/tools/packs/publish.ts validate packs/user.slate/1.0.0
bun run ../mallea/tools/packs/publish.ts build . dist
```

Validation uses a disposable daemon profile, the public `pack.validate`,
`pack.install`, and `pack.export` commands. Apps prepare their declared stores with
`app.create` in that disposable profile before `app.verify`; additional capability
requests stay unapproved. Published file/directory stores must use relative paths
without parent traversal so validation cannot touch host files. The exact
exported ZIP is validated again and hashed. Nothing touches your installed packs.
App static verification does not replace Native SDK interaction verification;
official app releases must also pass their declared live flows before merging.

An agent or person can submit a complete pack through the same CLI:

```sh
bun run ../mallea/tools/packs/publish.ts submit . /absolute/path/to/pack 1.0.0
```

This validates source, exports accepted bytes, creates an isolated Git worktree,
commits only canonical source plus release metadata, pushes a branch, and opens a
PR. Output is JSON with `status: "submitted"` and the actual PR URL. It does not
claim publication. Review generated description and minimum version before merge.

On main, CI validates again, publishes immutable versioned ZIPs, then atomically
commits `catalog.json` on the dedicated `catalog` branch. The publishing CLI emits
`status: "published"` only after all
uploads succeed. Existing version bytes must match exactly; revisions require a
new version. Failed uploads leave the preceding catalog available. Older catalog
snapshots cannot remove published packs or downgrade their versions.

Catalog URL:
`https://raw.githubusercontent.com/ankitiscracked/mallea-packs/catalog/catalog.json`

Install using `pack.install` with
`{"source":{"kind":"catalog","url":"<catalog URL>","id":"user.slate"}}`.
Agents use the same command, subject to normal capabilities and approval rules.

## Repository setup

Mallea source is private. Actions needs `MALLEA_READ_SSH_KEY`, a dedicated read-only
deploy key for `ankitiscracked/mallea`. Do not copy a personal GitHub token into
this repository. The workflow pins the validator to an exact commit; update both
checkout references deliberately when adopting new validation rules. Validator
source never belongs in release downloads, build logs, or caches.

Fork PRs cannot access this secret and fail closed. A maintainer must review the
source and move approved changes onto a trusted branch for validation. Never run
contributed scripts with private checkout credentials. Require the `validate` check
on main, and restrict direct pushes to release maintainers.

## Change storage backend

The catalog schema and client commands have no GitHub dependency. Replace the
upload step in the workflow; validated `dist/` files are ready for any HTTPS
storage service. Set `downloadBaseUrl` to its public prefix and preserve
`<id>-v<version>/<id>-v<version>.zip` paths, or map those paths during upload.
Upload ZIPs first and catalog last. Keep the catalog at a stable URL you control
before a production migration; moving its URL requires changing configured catalog
sources. GitHub-specific operations live only in Mallea's `tools/packs/github.ts`.
