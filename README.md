# Life vault kit

A starter kit for building your own **life vault**: a private folder of plain-text notes about your life (home, family, documents, admin, decisions) that Claude keeps for you, with every fact cited to its source and dated. A later Claude session can then answer "when does the car insurance renew?" from a filed letter instead of guessing.

This is not a finished vault. It is the plan, the rules and the prerequisites. **Your own Claude builds the vault with you**, after a short interview.

I built it for my own home admin and shared it after friends asked how to set one up. ("We" in these files means me and the Claude that keeps my vault.) The pattern follows Andrej Karpathy's LLM wiki idea (https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f), with a few additions from running it: one document at a time, private facts about people and health wait for your OK, and security rules for when mail gets connected.

## How to start

1. Get **Claude Code**. It comes with the paid Claude plans (Pro and up, not the free one; prices: https://claude.com/pricing) and runs in the Claude desktop app (https://claude.com/download): open the **Code** tab, choose **Local**, and pick your home folder. The kit itself is free.
2. Paste this:

   > Copy the repo https://github.com/donataskasp/life-vault-kit here, read START.md and follow it. Interview me before building anything.

3. Answer its questions. Nothing is built before the interview. Then file one real document together, such as an insurance letter.

Plan for about an hour. Claude asks permission before most actions (roughly 40 to 60 times in the first session): read what it wants to do and pick the plain "Yes", or "No" if it is not what you expected.

[START.md](START.md) is what your Claude will follow. You are letting an agent act on instructions from someone else's repository, so read it first if you want to know what they say. To pin the exact release you read, paste "Clone tag v1.0.2 of https://github.com/donataskasp/life-vault-kit here, read life-vault-kit/START.md and follow it. Interview me before building anything." instead.

## What you need

- **Claude Code** on a paid Claude plan (Pro, Max, Team or Enterprise), in the Claude desktop app or in the terminal. A plain chat on claude.ai cannot do this: it has to create folders, run git and write files on your computer. Install: https://code.claude.com/docs/en/overview
- **Desktop app users**: open the **Code** tab, choose **Local** (not Cloud or SSH), and select a folder on your own computer.
- **git**. On Windows, install Git for Windows first (https://git-scm.com/downloads/win): the vault needs git, and the safety hook needs the Bash shell it brings. The Claude Code docs call it optional for Claude Code itself. On a Mac without Apple's developer tools, the first git command opens an installer: click Install, wait for it to finish, then tell Claude to carry on.
- A place for the vault outside synced folders (iCloud Desktop & Documents, OneDrive, Dropbox, Google Drive), for example `/Users/<you>/life` on a Mac or `C:\Users\<you>\life` on Windows. Claude sets this up with you.
- For the backup: a USB drive (recommended if you are new to this), or a GitHub account.
- One real document to file together: an insurance letter, a school letter, a warranty or a lease, as a PDF, a photo or pasted text.

On a Mac, expect a few system popups asking to let Claude reach your Downloads, Documents or USB drive (click Allow). At Claude's own prompts, the plain "Yes" allows one action; avoid "don't ask again". The first session may take longer than an hour if all of this is new to you (our estimate).

## Where your data goes

The notes and the original documents are stored only on your computer and in your one backup (if you choose GitHub for the backup, GitHub can read it). To work with a document or a note, Claude has to read it, so its content is sent to Anthropic like anything else you give Claude. Your Claude account's privacy settings decide whether that content may also be used to improve the models: check them before you file sensitive documents.

## Two profiles

- **Light** (the default for everyone at the start): one folder of notes, a short rules file, your original documents kept unchanged, an index, version history, a backup if you choose one, and a safety check.
- **Full-core** (the upgrade): the complete schema with two skills, an operation log, a contradiction ledger and a review queue. Required before a bulk import (a mail backfill, bank statements, a pile of PDFs), or chosen from day one if you are comfortable with Claude Code.

The kit has been tested in six sandbox builds by a fresh Claude playing a newcomer (three on 2026-09-29, one on 2026-10-04, two on 2026-11-10; five Light, one Full-core). It has had little real-world feedback so far.

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
| `LESSONS.md` | What running our own vault taught us, with the numbers and their limits |
| `HUB-PLAN.md` | The always-on layer (phone front door, morning briefing, voice capture): designed July 2026, **not built**, with untested assumptions |

## Honest status

- Our vault was set up in July 2026 and has been in regular use since late August, almost all from its own Claude workspace.
- Not measured: how often it is read, or how much time it saves. Most numbered lessons come from one day, the 2026-09-28 mail import.
- The always-on hub is designed but unbuilt.
- Facts about Claude Code, its plans and features were checked against the official docs on 2026-09-29, and the plan list and the desktop app steps again on 2026-11-10. They change; the linked pages are the source of truth.

## License

MIT, see [LICENSE](LICENSE). Shared as is: there is no support promise.
