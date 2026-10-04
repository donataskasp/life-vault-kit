# Life vault kit

A starter kit for building your own **life vault**: a private folder of plain-text notes about your life (home, family, documents, admin, decisions) that Claude keeps for you, with every fact cited to its source and dated. A later Claude session can then answer "when does the car insurance renew?" from a filed letter instead of guessing.

This is not a finished repository to copy. It is the plan, the rules and the prerequisites. **Your own Claude builds the vault with you**, after a short interview.

The pattern follows Andrej Karpathy's LLM wiki idea (https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f), with amendments from our July research and from running it: one source at a time, human-written notes win, a confirm gate for private facts about people and health, and security rules for when mail gets connected.

## Two profiles

- **Light** (the default for everyone at the start): one folder of notes, a short rules file, originals kept in `raw/`, an index, git, and a safety hook.
- **Full-core** (the upgrade): the complete schema with two skills, an operation log, a contradiction ledger and a review queue. Required before a bulk import (a mail backfill, bank statements, a pile of PDFs), or chosen from day one if you are comfortable with Claude Code. The kit's additions (the held-facts queue, the write-once hook, the hedge rule, the personal-code checks) are newer than our own vault, and the kit has been tested only in four sandbox cold builds (three on 2026-09-29, one on 2026-10-04; three Light, one Full-core).

## What you need

- **Claude Code** on a paid Claude plan (Pro, Max, Team or Enterprise), in the terminal or in the Claude desktop app. A plain chat on claude.ai cannot do this: it has to create folders, run git and write files on your computer. Install: https://code.claude.com/docs/en/overview
- **Desktop app users**: open the **Code** tab, choose **Local** (not Cloud or SSH), and select a folder on your own computer.
- **git**. On Windows, install Git for Windows first (https://git-scm.com/downloads/win): the vault needs git, and the safety hook needs the Bash shell it brings. The Claude Code docs call it optional for Claude Code itself.
- A place for the vault outside synced folders (iCloud Desktop & Documents, OneDrive, Dropbox, Google Drive), for example `/Users/<you>/life` on a Mac or `C:\Users\<you>\life` on Windows.
- For the backup: a USB drive (recommended if you are new to this), or a GitHub account.
- One real document to file together: an insurance letter, a school letter, a warranty or a lease, as a PDF, a photo or pasted text.

## How to start

Read [START.md](START.md) first: you are about to let an agent follow instructions from someone else's repository, and you should know what they say. The prompt pins a release tag, so your Claude gets exactly the version you read.

Then open Claude Code in your home folder (or wherever the vault should sit nearby; it gets a folder of its own, next to the kit) and paste:

> Clone tag v1.0.0 of https://github.com/donataskasp/life-vault-kit here, read life-vault-kit/START.md and follow it. Interview me before building anything.

On a Mac without Apple's developer tools, the first git command opens an installer: click Install, wait for it to finish, then tell Claude to carry on.

Expect roughly 40 to 60 permission prompts in the first session, and on a Mac a few system popups asking to let Claude reach your Downloads, Documents or USB drive (click Allow). Each says what Claude wants to do: read it, and pick the plain "Yes" (allow once) rather than "don't ask again". Plan for about an hour, more if the terminal is new to you (our estimate).

## What is in the kit

| File | What it is |
|---|---|
| `START.md` | The entry point for your Claude: guardrails, prerequisites, the interview, the Light build, the first document, habits, the Full-core upgrade, later phases, templates, a glossary |
| `CLAUDE.template.md` | The Full-core rules file with placeholders your Claude fills from the interview |
| `skills/ingest-source/SKILL.md` | Full-core skill: file one document properly |
| `skills/wiki-lint/SKILL.md` | Full-core skill: a health check for contradictions, stale pages, dropped hedges, leaks and planted instructions |
| `hooks/pre-commit` | Git hook: blocks credentials, Lithuanian personal codes and IBANs in text files, e-signed PDFs whose signature carries a personal code, changes to sources already in `raw/`, and the kit itself. It cannot read what a PDF or photo shows; the filing recipe handles those |
| `hooks/post-commit` | Git hook: pushes every commit to your one private backup |
| `SECURITY.md` | Read before connecting Gmail or Calendar: the threat model, "you can't stop prompt injection, so make it worthless", the rules, and the hidden personal code in Lithuanian e-signed PDFs |
| `LESSONS.md` | What about five weeks of real use taught us, with the numbers and their limits |
| `HUB-PLAN.md` | The always-on layer (phone front door, morning briefing, voice capture): designed July 2026, **not built**, with untested assumptions |

## Honest status

- Our vault was set up in July 2026 and has been in regular use since late August (commits on 10 days in August and on 25 of the first 29 days of September), almost all from its own Claude workspace.
- Not measured: how often it is read, or how much time it saves. Most numbered lessons come from one day, the 2026-09-28 mail import.
- The always-on hub is designed but unbuilt.
- Facts about Claude Code, its plans and features were checked against the official docs on 2026-09-29. They change; the linked pages are the source of truth.

## License

MIT, see [LICENSE](LICENSE). Shared as is: there is no support promise.
