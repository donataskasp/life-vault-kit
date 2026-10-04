# Lessons from running it

What running our own vault taught us, each lesson with its evidence and a date. Read the numbers with their limits. The vault was set up in July 2026, but it has **about five weeks of real use** (regular since late August: commits on 10 days in August and on 25 of the first 29 days of September), almost all from its own Claude workspace. **Most numbered lessons come from one day, the 2026-09-28 mail import**, which was audited by AI agents and not repeated.

Our vault is one person's, with a shared household area, maintained by Claude Code. On 2026-09-29 it had about 170 commits; 134 raw items (files, plus two folders counted once), about 60% of them agent-written research reports; mail entered as 8 verified extracts; 41 wiki pages including the three system pages; and 6 contradiction-ledger entries. Commit counts overstate life-admin use: about a third of commits belong to one hobby project, and about a sixth are about the AI setup itself.

**Not measured:** how often the vault is read, or how much time it saves. Numbers come from the vault's log and git history; lessons that rest on outside research say so.

---

## 1. Seed it through real decisions, mostly

**Evidence.** Apart from one short brain-dump session early on (basic household facts), the vault grew from things we were actually deciding or handling: a purchase comparison, a repair, a trip, a letter that needed an answer, each ended with "file this". Counting wiki files the same way both times (system pages included): 26 on 2026-09-06, 41 on 2026-09-29. Ten of the 15 new pages came from a two-day document and mail import (27 and 28 September). Most early pages came from a real question; we have not measured how often they are asked again.

**Do this.** End every decision conversation with "file this in the vault". Things the owner says in chat count as sources, cited as "owner, chat, date".

## 2. One source at a time, for documents

**Evidence.** Outside research (July 2026): implementations that batch-compile get shallow pages ("it extracts the table of contents. It looks like knowledge. It isn't"). We held to one-at-a-time for documents. Research runs went in several reports at a time, and a mail extract is itself a summary of many messages.

**Do this.** One document, one full pass, one commit. For a long document, one chapter at a time.

## 3. The mail import method (2026-09-28)

This is the method, including the parts we would change. Preconditions: the Full-core profile, connectors running read-only for a while, and SECURITY.md read in full.

**Containment, first:**

- Extracts were built in a private folder **outside the repo** and outside any synced or backed-up path.
- **Auto-push was paused** for the run until the owner had reviewed the results, then restored.
- A **fail-closed personal-data scanner** ran on every commit (the kit's pre-commit hook does the same job).
- The extract session held **no write or send tools**, and nothing it wrote entered the auto-pushing repo until it was verified and scanned. This resolves the Rule-of-Two tension (SECURITY.md, section 2): the session that reads strangers' mail can neither act nor publish.

**The pipeline, one narrow topic at a time** (a car, one insurance, the home, taxes):

1. **Extract**: one row per fact, each with the message date, sender, subject and the supporting text.
2. **Verify independently**: a separate session that did not write the extract re-reads rows against the original messages. We sampled 27 to 55 rows per extract, and about half the rows in the seven sampled extracts were never re-read (269 of 542). Better: verify every row, or mark each row verified or unverified. If you sample, sample at random.
3. **Send back or fix.** Accepted extracts had 2% to 9% partial errors. One was sent back at 29% because facts about two similar subjects had been attributed to the wrong one; another at 13%. Our "send back above about 10%" line was drawn after the fact and sits within one row of noise at these sample sizes: treat it as a heuristic. Send an extract back for any misattribution between similar subjects, whatever the rate. Fix every flagged row before the extract enters `raw/`, which is write-once.
4. **Compile** one extract at a time, with the gate applied (lesson 7).
5. **Spot-check** a random sample of compiled facts against the mail.
6. **Full compile audit**: every page claim against its extract row (lesson 4).
7. **Lint.**

**Cost.** The run took one evening and used about 36 agents and about 14 million subagent tokens. On a subscription plan that is heavy usage: it can crowd out everything else you use Claude for that day, and may run into your plan's usage limits. Spread topics over several days.

**Do this.** Keep one extract to about a hundred rows or fewer (seven of ours ran 46 to 105).

## 4. Most remaining damage happens while writing pages: audit the hedges

**Evidence (2026-09-28).** After verification, the 40-fact spot-check found no extract errors (before verification, extracts had 2% to 29% partial errors). 35 facts matched, 4 were partial and 1 could not be verified (the source was a screenshot); all 4 partials were page errors. Most remaining damage happened while writing pages. The full compile audit compared **909 page claims** with their extract rows and flagged **79** (about 9%):

| Problem | Count |
|---|---|
| Dropped hedge ("probably" became fact) | 42 |
| Added precision the source did not have | 15 |
| Separate facts merged into one claim | 9 |
| Wrong citation | 8 |
| Wrong value | 5 |

**Limits.** One run, audited by AI agents, not re-run. Pages compiled before the import were never audited. The kit's "keep the hedges" rule was added after this audit, and its effect is not measured.

**Do this.** Keep the full claim-by-claim audit after any bulk import, hedge rule or not. Treat vault answers on money or legal matters as leads to check against the source.

## 5. A check nobody schedules detects nothing

**Evidence.** The lint skill existed 65 days before its first run (added 2026-07-25, first run 2026-09-28, right after the import): **3 errors, 13 warnings, 14 info**. The same-day fixes covered unsourced labels, dropped hedges, amounts sitting on shared pages, and two new contradiction entries.

**Do this.** Put the lint in a calendar or task tool (monthly), and run it after every bulk import.

## 6. Keep amounts and personal detail off shared pages

**Evidence (2026-09-28).** The first lint moved amounts off two shared household pages to the personal money page and dropped a utility connection number from a shared page. Insurance was filed on shared household pages **without amounts**, with the detail on a personal page. During the import, other people's names and a contract number were removed from an extract before filing. Ten of our older shared pages still carry prices; which amounts count is undecided.

**Do this.** Before the first ingest, decide which amounts count (a purchase price? a subscription fee?) and write it into CLAUDE.md. Leave third parties' personal data out unless you will need it.

## 7. The gate holds where facts are labelled, and leaks elsewhere

**Evidence.** It held 128 rows in the import, where the extracts labelled gated rows. It leaked in ordinary ingests: the first lint found at least three more gate issues, one a month old. One gated detail also slipped onto an import page until the compile audit removed it. The queue file (`inbox/held-for-owner.md`, kept out of git until the owner confirms) is new in this kit.

**Do this.** Label gated facts when they are captured, keep them in one queue the owner clears weekly, and have lints and audits check for gate leaks.

## 8. The contradiction ledger fills when sources overlap, and ingest misses conflicts

**Evidence.** The first ledger entry came on 2026-09-05, two months after setup, and there were two by mid-September. The import day added four: two logged during ingest, two the ingests had missed that the first lint caught. The lint and the compile audit also corrected further conflicting claims directly on pages (the audit alone found 5 wrong values).

**Do this.** Create the ledger on day one, and rely on lint and audits, not ingest alone, to find conflicts.

## 9. Claude is the only reader, and mostly in one place

**Evidence.** As of 2026-09-29 the author had never opened the vault in Obsidian. Claude is the only reader, and almost only in the vault's own workspace: in the author's other personal Claude Code projects, 1 of 55 retained sessions opened a vault file.

**Do this.** Obsidian is optional. If you want other Claude projects to use the vault, point them at it explicitly, then check whether they actually do.

## 10. The known killers: "nobody reads it" and trust decay

**Evidence (outside research, July 2026, about 13 sources: write-ups, threads, docs).** The documented abandonment path was "everyone is writing, nobody is reading", even when the technology worked. The other killer is trust decay: "six months in, you have entries that are confidently wrong and the lint pass can't tell which". The four or so long-running successes all had a human review step; the one detailed corruption report did not. In our own vault, `last_checked` is not kept current reliably; lint is what catches it.

**Do this.** Ask real questions every week, and log them (lesson 16). Keep citations, the gate, and the hedge audit.

## 11. Lithuanian e-signed PDFs carry a hidden personal code

**Evidence (2026-09-27).** A registry extract whose visible text showed no personal code still carried one inside its signature certificate. We caught it before filing. SECURITY.md, section 4, has the technique.

## 12. The hub is unbuilt, and the stall is owner time

**Evidence.** The always-on layer (a phone front door, a morning briefing, voice capture, monitors) was designed in July 2026, partly to force the wiki to be read. As of 2026-09-29 none of it is built. The stall is owner time, not technology: none of the four owner moves was made in the 23 days after the build plan (2026-09-06), including one that takes minutes. HUB-PLAN.md lists eleven things to re-check, not one.

**What we can and cannot show.** The payoff is not measured. What can be shown: sessions in the vault's own workspace work from filed, cited facts across multi-week efforts, and the mail import surfaced conflicts between what the owner remembered and what the records say.

**Do this.** Build the vault first and use it for a month before considering the hub, and count the owner steps any plan needs.

## 13. raw/ fills with AI output

**Evidence.** About 60% of our raw items are agent-written research reports, not original documents.

**Do this.** Keep originals apart from AI research (a `raw/YYYY/research/` folder, or a `kind` field), so citations never treat AI output as ground truth.

## 14. Pages go stale when outcomes are never reported

**Evidence.** The first lint (2026-09-28) flagged older stale pages for the owner: a decision page stays "pending" until someone reports what happened.

**Do this.** Close the loop: when something is bought, sent, booked or cancelled, tell the vault. The kit's lint lists pending pages whose date has passed.

## 15. Tinkering with the system can quietly become the hobby

**Evidence.** About a sixth of our commits are about the AI setup itself, not about life.

**Do this.** Watch that share. The vault exists to hold facts, not to be improved.

## 16. Log every question answered from the vault, from day one

**Evidence.** We did not, so we cannot tell use from accumulation.

**Do this.** One line per answered question (`questions.md` in Light, a `query` line in `wiki/log.md` in Full-core).

## 17. Anything that waits on the owner piles up

**Evidence.** None of the hub's four owner moves in 23 days (lesson 12); 128 gated rows held at once in the import (lesson 7); owner items left in the first lint report.

**Do this.** Keep owner steps few, small, and scheduled.

## 18. A rule enforced only by a prompt gets broken

**Evidence.** `raw/` write-once was broken once while only a prompt enforced it.

**Do this.** Enforce hard rules mechanically. The kit's pre-commit hook blocks any change to a committed `raw/` file.
