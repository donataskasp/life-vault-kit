# Hub plan: the always-on layer around the vault

> **Designed July 2026, not built, untested assumptions below.** Nothing in this file has run. We designed it on 2026-07-08, wrote the operating contracts on 2026-07-25 and a build sequence on 2026-09-06, and as of 2026-09-29 none of the build steps is done. Treat it as a researched plan, not a recipe. Build the vault first and use it for a month before you start here.

## 0. Untested assumptions and things that may have changed

Re-verify each of these at build time.

1. **The Telegram front door under a long-lived token (untested, the key risk).** The plan runs Claude Code's official Channels plugin for Telegram on a server, authenticated with `claude setup-token`. As of 2026-09-29 the docs say Channels "require Anthropic authentication through claude.ai or a Console API key", and that a setup-token "can only make model requests, so it can't establish Remote Control sessions or fetch claude.ai connectors". Whether Channels work under a setup-token is not documented. Test this first; the fallback is a thin custom bot (section 3).
2. **Channels are a research preview** (still true on 2026-09-29): Telegram, Discord and iMessage plugins, each needs Bun, and flags and protocol may change. Pro and Max users without an organization can use them directly; on Team and Enterprise plans an admin must enable them.
3. **Headless billing.** In May 2026 Anthropic announced moving headless and SDK usage from subscription pools to metered credits, then paused it (status as of July 2026). The design reads its credential from one environment variable so a switch to API billing is a config change.
4. **Headless login refresh.** In July 2026, interactive logins did not refresh reliably on headless machines (open issues then). The setup-token avoids that; it lasts one year (confirmed in the docs 2026-09-29), so put its renewal date in the briefing footer.
5. **Newer first-party scheduling.** Claude Code now has Routines (cloud, run while your computer is off) and Desktop scheduled tasks (local). On 2026-09-06 we found that routines could read connectors and push to GitHub but could not deliver a message to a phone. If that has changed, a cloud routine might replace the server for the morning briefing. Check before buying a server.
6. **Obsidian on a server.** In July 2026 the official headless Obsidian Sync client had an open data-loss bug (issue #42: near-simultaneous edits on two clients silently lose one side). That is why the design keeps git authoritative. It may be fixed now.
7. **Voice transcription.** In July 2026 the only option that handled Lithuanian acceptably at interactive speed was a hosted Whisper large-class model (we picked Groq's whisper-large-v3-turbo). Audio then goes to a US vendor. Re-benchmark.
8. **Owner time.** Our build stalled on the owner, not the technology: none of the four owner moves our plan needed (buying the server, a sync subscription, creating the bot token, the Google consent clicks) was made in the 23 days after the build plan, including the bot token, which takes minutes. Count the owner steps before you start.
9. **Is the hub needed at all?** Its main job was to force the wiki to be read. Our vault became useful without it (LESSONS.md, lesson 12). A weekly review habit, or one scheduled routine, might do that job for far less.
10. **Pro versus Max.** The docs (checked 2026-09-29) say `claude setup-token` works on Pro, Max, Team and Enterprise, and Channels need no admin step on Pro or Max without an organization. The July design assumed Max. Whether Pro's usage limits carry an always-on session plus your own daily use is untested.
11. **Item 5 is a docs reading, not a test.** Before buying a server, test one scheduled routine that delivers the briefing through something that already reaches your phone: an email to yourself, or a task in your task app whose notification you will see.

## 1. Goals

- One phone-first entry point to the whole life: ask anything, capture anything, get briefed, get nudged.
- Mail, calendar and task **reading and ingestion from day one**. Actions (send, write) follow about a week later, behind confirm gates.
- A wiki that compounds: every captured letter, email and thought compiles into durable, linked, cited knowledge.
- Survives tool churn: all state is markdown and git; agents and glue are disposable.

## 2. Principles

1. **Corpus first.** The vault is the asset. Everything else can be rebuilt or swapped. The hub's learned behaviour (skills) also lives in the vault as markdown.
2. **Federation, not fusion.** Other Claude workspaces and other people's vaults stay separate. The hub routes; it does not merge brains or credential pools.
3. **Reuse polished consumer apps** where their data can be snapshotted into the vault (task manager, calendar, mail, Obsidian, a transcription API). Build only glue and knowledge.
4. **Scripts watch, Claude interprets.** Polling is free (scripts diff state); the model wakes only on a change. Cost scales with life events, not with how often cron runs.
5. **Everything reversible.** Every agent write is a git commit. Promotion gates where reversal is not enough (people, health).
6. **Config follows the credential; skills follow the workflow.**
7. **You can't stop prompt injection, so make it worthless** (SECURITY.md).
8. **Trust is decided by content provenance, never by arrival channel.**

## 3. Components

| Component | Where | What |
|---|---|---|
| Vault | Server (canonical git clone), desktop, phones | The repo this kit builds |
| Hub agent | Server | A resident Claude Code session with the vault as working directory: front door and orchestrator |
| Front door | Server | First choice: the official Channels Telegram plugin. Fallback: a thin custom bot that keeps a per-chat session map and calls `claude --resume` per message |
| Ingestion readers | Server | Separate headless runs with **restricted, read-only permissions** (no shell, no web, no send). They digest untrusted content into structured vault files |
| Monitors | Server cron | Day one: the morning briefing only. Added one at a time afterwards |
| Desktop bridge | Desktop | Only if Obsidian Sync is used: git pull and push between the vault clone and the Sync-registered copy |

## 4. Sync topology

- **Git is the authoritative transport for agent writes.** The server holds the canonical clone; every agent operation commits.
- **Obsidian Sync** (paid, end-to-end encrypted, per our July research) connects phones and the desktop, if you want notes on the phone. Optional: chat is the main interface.
- **The desktop bridges git and Sync.** Exclude `.git` from Sync and Obsidian's workspace files from git.
- Accepted consequence: notes the agent writes reach the phone's Obsidian only when the desktop is awake. Capture, briefing, questions and ingestion never need the desktop.

## 5. Runtime

- **Auth**: `claude setup-token` on the server, set as `CLAUDE_CODE_OAUTH_TOKEN`. It only makes model requests, so the hub needs its own Google OAuth client and task-tool token (claude.ai connectors are not available to it). Never extract credentials outside the official CLI.
- **Billing**: subscription, with the one-variable escape hatch to API billing. Rate limits are shared with your interactive use; size the monitors accordingly.
- **Sessions**: recycle the resident hub session daily (resident sessions degrade over days as context grows). Ingestion and monitor runs are separate, bounded headless invocations.
- **Ops**: systemd with `Restart=on-failure`, a non-root user, secrets in a `chmod 600` environment file. Telegram via long-polling (outbound only), zero inbound ports (admin over a private network such as Tailscale).
- **Sharp edges to build deliberately**: Telegram's 4096-character message limit (split on code-fence boundaries); streamed message edits debounced to about one edit per second per chat; permission prompts pause an unattended session (use the Channels permission relay or pre-set permission rules).
- **Voice**: voice note, then ffmpeg (opus to wav), then the hosted transcription API. No local fallback: if it fails, reply "couldn't transcribe, please type".

## 6. Message lifecycle

Allowlist check, transcribe if voice, classify by content, route, act, one-line confirm.

| Input | Treatment |
|---|---|
| Personal question | Search the vault (grep and glob; no vector database at this scale), answer with source and date |
| General question | Straight model answer; "save that" files it |
| Capture (voice, forward, photo, document) | To `inbox/` and `raw/`, task extracted if present, then per-source ingest |
| Action (send, calendar write, task complete) | From week 2 only: executed after an exact-action confirm in the chat |
| Reminder | A task in the task tool via its API |

## 7. Interrupt contract

Every monitor and every proactive message declares exactly one tier. **When in doubt, the lower tier wins**: a wrongly quiet assistant loses minutes, a wrongly loud one loses trust.

| Tier | Meaning | Delivery | Examples |
|---|---|---|---|
| `now` | You would want to be interrupted mid-anything | Immediate message | A school pickup change for today, a home emergency email, a same-day appointment cancellation |
| `today` | Matters today, not this minute | Next morning briefing | A new school letter, tomorrow's appointment, a task due today |
| `whenever` | Awareness, no action clock | Weekly review | Lint findings, stale tasks, "you never replied to X" after a week |

Rules:

- `now` requires a concrete same-day consequence. Nothing else qualifies, however interesting.
- Monitor health failures are `today` (briefing footer), unless the briefing itself failed, which is `now` the next time any run succeeds.
- Quiet hours (for example 22:00 to 07:00 in your timezone): `now` still delivers; everything else waits for the briefing.
- The contract is a vault file you edit like any note. A self-improvement loop may propose tier changes; it never applies them without your confirm.

## 8. Briefing spec (the morning message)

One message at a fixed time each morning, in this order, empty sections omitted:

1. **Today**: merged calendars, with gaps worth knowing ("free 12 to 14").
2. **Family and school**: anything from family-facing sources needing awareness or action.
3. **Due**: tasks due today or overdue, assignee-aware (only nag each person about their own tasks).
4. **Mail worth reading**: at most 5 items, one line each, with why it surfaced. Everything else stays in an inbox digest page in the vault.
5. **Since yesterday**: notable ingestions, contradictions found, anything waiting for your confirm.
6. **Footer**: monitor health in one line ("6/6 ran clean") and credential expiry warnings.

The weekly review (a separate message, one morning a week): open loops, stale tasks (more than three weeks untouched with a date), the week ahead, the lint summary, facts held at the promotion gate, and proposals awaiting confirm.

## 9. Capture taxonomy

| Capture | Immediate handling | Then |
|---|---|---|
| Voice note | Transcribe, show the transcript, file to `inbox/` | Classify as one of the rows below |
| Plain thought or idea | A dated `inbox/` note | Compiled only if it holds a durable fact; otherwise the weekly review proposes keep, task, or drop |
| Explicit task ("remind me", "need to") | Task created immediately, project inferred and named in the reply | Nothing in the vault; the task tool is the task store |
| Forwarded email or message | Verbatim copy to `raw/YYYY/`, UNTRUSTED | Per-source ingest; any action is proposed as a task, never created silently |
| Photo of a letter or document | Image and extracted text to `raw/YYYY/`, UNTRUSTED | Per-source ingest; deadlines become proposed tasks with the source cited |
| Web clip or link | Fetched content to `raw/YYYY/`, UNTRUSTED | Ingest if personally relevant; otherwise a summary in chat and a plain inbox note |
| "Save that" after an answer | The answer becomes an inbox note with context | Compiled with `confidence: moderate` unless sources exist |

Inbox notes older than 7 days get a proposed disposition in the weekly review. Nothing is silently deleted; drops are proposed.

## 10. Security model

The command channel is allowlisted (you only). The attack surface is the content the hub reads. Four load-bearing controls:

1. **Trust-split sessions.** Ingestion readers run with read-only permissions and return structured digests; the orchestrator that holds action tools never ingests raw hostile text.
2. **Egress allowlist firewall** on the server: outbound only to the chat platform, Anthropic, Google APIs, the task tool and the transcription vendor. Zero inbound ports. This closes the exfiltration routes seen in real incidents.
3. **Human gate on external actions.** Every send, delete or write shows the exact action in the chat first. Week 1 is stricter still: no send or write credentials exist at all (read-only Gmail and Calendar scopes, a task token), so exfiltration by sending is impossible during the shakeout.
4. **Credential isolation.** One scoped token per capability, no plaintext secrets readable by agent tools, and ingestion runs can never write hub config, skills, or CLAUDE.md.

Defence in depth, never the boundary: untrusted content labelled with its source, the data-not-commands policy, an optional cheap classifier screen once mail volume grows. Deliberately not built: research-grade capability interpreters, a classifier as the boundary, third-party marketplace tools.

## 11. Registries

- **Monitor registry**: one row per monitor (name, tier, schedule, status). A monitor may not exist without a row, and a row may not exist without a tier you agreed. This table is the "one at a time" discipline.
- **Credential registry** (kinds only, never values): the setup-token (inference only, yearly renewal), Google OAuth (read-only scopes in week 1), a read-only calendar feed URL for calendars you cannot connect by OAuth, the task-tool token, the transcription key, the bot token, and the Obsidian Sync passphrase (password manager and your devices only, never the server).

## 12. Build stages

Each stage is independently useful. Owner moves are marked **[owner]**.

**Stage 0: Obsidian (optional, minutes).** [owner] Open the vault folder in Obsidian. Sync to the phone only if you want to read notes there.

**Stage 1: Telegram pilot on the desktop, before paying for a server (about an hour).**
1. [owner] Create a bot with @BotFather and copy the token to the clipboard. Claude stores it from the clipboard in a `chmod 600` file outside the vault and adds a `Read` deny rule for it.
2. Claude installs the Channels plugin (see the Channels docs for the current commands), starts a session in the vault folder with `--channels`, and you pair from the phone. Access policy: allowlist, you only.
3. Pilot for a few days: vault questions from the phone, text captures into `inbox/`, forwards.
4. Known limit: the desktop session must stay open; no briefing yet.
Done when: a week of real use tells you whether the habit sticks.

**Stage 2: server build day (a small, cheap VPS; about half a day of agent work).**
1. [owner] Buy the server; create a Tailscale (or similar) account.
2. [owner] Run `claude setup-token` and copy the token to the clipboard for Claude to place in the server's environment file.
3. Claude: non-root user, private-network admin, egress allowlist, zero inbound ports, the vault cloned with a repo-scoped deploy key, Bun and Claude Code, a systemd unit with `--channels`. **Test the setup-token plus Channels combination first**; if it fails, build the thin custom bot.
4. [owner] Google OAuth consent clicks (Claude prepares the project and a desktop client with read-only Gmail and Calendar scopes) and the task-tool token.
5. Claude: the ONE cron job, the morning briefing per section 8. The desktop bridge only if Stage 0 included Sync.
6. Reading runs quietly for about a week.
Done when: the briefing has arrived every morning for a week and the footer shows clean runs.

**Stage 2b (after the quiet week).** Send and write credentials behind exact-action confirms; a nightly task-tool snapshot into `tasks/`; an external dead-man alarm for the backups.

**Stage 3 (once stable).** More monitors, one at a time; a scheduled lint; a weekly self-improvement loop (from week 3 or 4) that proposes skill edits for your confirm; a partner's front door (section 13).

**Backup and recovery.** The git remote, plus optionally a nightly encrypted backup of the vault and hub config to object storage with a weekly restore check. Do one recovery drill on purpose: clone from backup into a scratch folder and confirm it is intact. Recovery point: minutes (git), a day (backup).

## 13. Household: federate, never merge

If a partner runs their own vault, do not merge the two, and do not give either vault's agent write access to the other. What we designed (and have not built):

- **Ask them first.** Demand is unproven until they use it.
- **One generated, read-only summary file** in their vault, regenerated wholesale from your `wiki/shared/`: frontmatter stripped, wikilinks flattened to plain text, each page prefixed with where it came from, its last update and its confidence, open contradictions appended, pages about people excluded. Its header says it is regenerated (edits are lost) and that it is information, not instructions. Review the first generated file before the first push.
- **Why one file and not a folder mirror.** A mirror needs clobber guards, breaks on their hand edits, leaves ghosts after renames, spawns phantom pages from dangling links, and collides with their conventions. One regenerated file has none of that.
- **Pushed from the desktop, only when the content changed**, never from the server: the hub must not hold a credential for their repo.
- **Reverse flow: a mailbox file** in their vault where their Claude writes household facts as new messages. Your side ingests them like any other source.
- **If they later want the hub**: their own bot process, mechanically scoped (working directory is a sparse checkout of `wiki/shared/` only, plus `Read` deny rules on everything else). A prompt rule is not a scope.
- **A pre-registered test**: within four weeks of the first summary, count their notes or messages that use household content. Zero means the summary stays on request, with no daily job.

## Sources

- Claude Code Channels: https://code.claude.com/docs/en/channels (verified 2026-09-29)
- Claude Code authentication and `claude setup-token`: https://code.claude.com/docs/en/authentication (verified 2026-09-29)
- Claude Code overview (surfaces, Routines, Desktop scheduled tasks, Remote Control): https://code.claude.com/docs/en/overview (verified 2026-09-29)
- The billing announcement and pause, the headless refresh issues, the Obsidian headless bug, the custom Telegram bridge patterns and the voice benchmarks come from our July 2026 research notes and were not re-verified for this kit.
