# Moniker

Moniker is a desktop app for managing AI agent skills with clear, durable provenance.

It lets skills with the same upstream name coexist under different local aliases. Each skill remains linked to its original repository and path, so Moniker can update it from the correct source without losing its identity.

> Moniker is in the design stage. This repository does not contain a working release yet.

## The problem

Most skill managers identify a skill by its name or installation folder. This fails when two publishers use the same name.

For example, CodeRabbit and Matt Pocock both publish a skill named `code-review`. A name-based manager can confuse them, overwrite one with the other, or update a skill from the wrong repository.

Renaming the folders does not solve the problem. In many tools, the folder name is also the update key. An alias can therefore break the connection to the upstream source.

## The Moniker model

Moniker separates facts that other tools often mix together:

- **Origin identity** identifies the repository and skill subpath. It does not change during an update.
- **Upstream name** is the name declared by the skill publisher.
- **Local alias** is a user-controlled name for a specific agent installation.
- **Installed revision** records the exact Git commit or source revision that Moniker installed.
- **Content digest** records the installed file contents.
- **Verification evidence** records what Moniker checked. A skill cannot declare itself verified.

This allows both skills to exist at the same time:

```text
Origin: github.com/coderabbitai/skills//skills/code-review
Alias:  coderabbit-code-review

Origin: github.com/mattpocock/skills//skills/engineering/code-review
Alias:  pocock-code-review
```

Updating `coderabbit-code-review` follows only the CodeRabbit origin. Updating `pocock-code-review` follows only the Matt Pocock origin. Changing an alias never changes either update path.

## Provenance and trust

Moniker distinguishes between a claim and proof.

- **Self-declared** means the skill states an author, version, or source.
- **Origin-recorded** means Moniker obtained the skill from the recorded location.
- **Content-verified** means the installed files match the recorded digest.
- **Commit-signature verified** means Git reports a valid commit or tag signature.
- **Publisher verified** means a trusted publisher identity controls the verified signing key.
- **Locally modified** means the installed content no longer matches its installation receipt.

A source URL is useful, but it is not proof of authorship. A file hash proves that content matches a manifest, but an unsigned manifest does not prove who created it. Moniker will show these distinctions instead of using one ambiguous **Verified** badge.

## Portable receipts

Author declarations can remain in the standard `metadata` field of `SKILL.md`. Moniker will store observed and verified facts in a separate installation receipt.

The receipt will include:

- Canonical repository URL
- Skill subpath within the repository
- Requested branch, tag, or reference
- Resolved commit
- Bundle digest and file hashes
- Verification method and result
- Verification time

The receipt belongs to Moniker. Upstream skill content remains unchanged, which keeps source comparisons and updates reliable.

## Canonical skills and agent aliases

Moniker keeps one canonical copy of each skill origin. Agent installations are derived copies or links with their own aliases and platform-specific metadata.

This design supports Codex, Claude Code, and other agents without changing the canonical source. It also prevents one agent's naming requirements from corrupting provenance for every other agent.

## Safe updates

An update must preserve the recorded origin. Moniker will:

1. Fetch only from the recorded repository and skill subpath.
2. Resolve and display the proposed revision.
3. Show the affected skill, origin, and local alias.
4. Preview content changes before installation.
5. Preserve the previous version for rollback.
6. Write a new receipt only after a successful update.

Changing a skill's origin is a separate, explicit operation. An ordinary update cannot silently replace one publisher's skill with another publisher's skill.

## Planned application

Moniker is planned as a React and TypeScript interface in a Tauri wrapper for Linux and macOS. Rust will own Git operations, hashing, verification, and safe filesystem changes.

The filesystem will be the database for the first version. Portable files will store origins, aliases, deployment targets, and installation receipts beside the canonical skills they describe. Moniker will build an in-memory index when it starts.

```text
~/.moniker/
├── skills/
│   └── <stable-origin-id>/
│       ├── SKILL.md
│       └── receipt.json
└── installations.json
```

Writes must use file locking and atomic replacement so an interrupted operation cannot leave partial metadata. The file formats will be versioned so Moniker can migrate them safely.

A future database can provide a rebuildable search cache if the library becomes large. It must not become the only place where provenance exists. The portable filesystem records remain the source of truth.

The first complete version will focus on:

- Installing a skill from a Git repository and subpath
- Recording its exact origin and revision
- Assigning independent aliases for each agent
- Supporting same-name skills from different origins
- Deploying to Codex and Claude Code
- Checking, previewing, applying, and rolling back origin-locked updates
- Showing provenance and verification evidence in the library

## Core rules

1. Provenance is identity. A display name is not identity.
2. An alias can change without changing provenance.
3. An update cannot change provenance.
4. Two origins can use the same upstream name.
5. Verification results come from evidence, not skill declarations.
6. Canonical source content is not silently modified.
7. Destructive changes require a preview and a recoverable previous version.
8. Portable filesystem records are the source of truth.
9. Any future database is a disposable, rebuildable cache.

## License

No license has been selected yet.
