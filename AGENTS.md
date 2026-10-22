# docs — specification

This file is authoritative for all specification work in this repository. Any agent assigned to a
spec task or working in this folder follows these rules.

Code tasks treat this folder as strictly read-only and return to root `AGENTS.md`.

## Scope

Write only under `docs/<ProductName>/`. Never create, modify, or delete files outside this
directory. Code files, test files, repository configuration, and root files are strictly read-only.

## Approved inputs & raw material

- The user's direct answers, feedback, and decisions. The product is decided in conversation with the
  user, not by the agent.
- `doc-gap` issues raised by implementation or verification tasks.
- External evidence verified before it carries weight: APIs, platform constraints, market or regulatory
  facts. Label unverified claims as unverified.

**Raw material rule:** Whatever the input is — a `doc-gap` issue, build failure log, stack trace,
bug report, sprint note, or code snippet — treat it strictly as raw material to extract missing product
requirements. Never carry implementation plans, code snippets, file paths, commit hashes, or build
output into a specification document. Every document must read as a pure business and product
specification.

## Specification discipline

- **Confirm before writing.** Before adding or changing any document under `docs/<ProductName>/`,
  present the proposed addition, amendment, or outline to the user and obtain explicit confirmation.
  Never write or rewrite documents without checking in.
- **Canonical schema.** Load and follow the `authoring-bprd` skill. It defines the document set,
  order, sections, and validation. Author each document only from the sections its reference states, in
  the reference's order. Never invent ad-hoc sections, copy sections from other documents, or pad with
  filler. If `authoring-bprd` is unavailable, stop and report; never author a specification from memory.
- **Grounding and evidence.** If the user's direction conflicts with verified external constraints,
  state it plainly and let the user decide. Never resolve the conflict silently or present an
  unverified claim as settled fact.
- **Cross-document consistency.** When a requirement changes (e.g. adding an entity in `schema.md`,
  a screen state in `screens.md`, or a capability in `features/`), update all affected documents so the
  specification set remains internally consistent and passes validation.

## Checks

- Run `python3 scripts/validate_bprd.py docs/<ProductName>` when the validator exists.
- Verify that every cross-reference (`FEAT-###`, `NFR-###`, `C-###`, screen IDs) resolves cleanly.
- Confirm every document reads as a specification, without implementation commentary or code traces.

## Definition of Done

- Every change was explicitly confirmed with the user.
- The document set is internally consistent and passes schema validation.
- All doc-gap questions are settled with permanent specification text, not conversational assumptions.
- Commit specification changes separately from code with a conventional commit message (e.g.
  `docs(<product>): update authentication requirements for OAuth provider`).
- When this folder is a git submodule, the commit happens in the docs repository; the pointer bump in
  the parent repo is a separate PR.
