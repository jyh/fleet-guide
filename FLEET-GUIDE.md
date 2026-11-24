# Running a fleet of agent seats: a field guide

*Version 1, 2026-11-24. Written to be carried to another team: it names no account, machine, path, person or project, and it ships no code. Every mechanism below is described well enough to rebuild. Where a number appears, it is the value that worked for us, not a requirement.*

---

## 1. What a fleet is

A fleet is a set of long-running AI agent sessions ("seats"), each working one campaign in its own directory, coordinated so that one human can steer all of them in about thirty minutes a day.

Four roles:
- **The owner:** the human. Holds the intentions, makes the decisions only a human should make, and is never a bottleneck.
- **The helm:** one coordinating seat. It keeps the fleet aligned to the owner's objectives, routes work, decides everything that can be checked and reversed, prepares the daily council and talks with the owner.
- **The duty officer:** a second, always-on seat that executes routine acts (merges under written conditions, stamps, installs), posts an hourly digest to the helm, and signs other seats' hand-over notes. The helm inspects its every act.
- **Seats:** one per campaign or repository. A seat owns its tree, its queue and its memory.

A minimal fleet is a helm, one seat, a bus, gates and a keeper (sections 3 to 5). Add the duty officer when the helm spends more time on routine acts than on decisions; add the daily council when the owner's questions start to pile up.

## 2. The principles

Everything else in this guide is how these have applied in practice.

**The first principle: do the right thing.** Law cannot and should not grow without end, and law never overrides doing the right thing. When a rule and the right thing part ways, the right thing wins; say so out loud, so the rule can be fixed.

1. **Tell the truth about what you know.** Say what you measured, what you inferred, and what you did not look at. Put a limit beside the claim it limits. "I don't know" is a complete answer.
2. **Don't trust a nothing until you have looked at it.** "None found" is a claim about everything you searched. Check that your instrument could see, check what it actually looked at, and if the nothing would be news, get it a second and different way.
3. **A check earns trust by failing.** Before you rely on a test, a gate or a watch, make it fail once on purpose. A check that cannot fail is decoration.
4. **Done means a stranger could verify it.** If the only way to know it is true is to ask you, it is not done. Leave the receipt where the reader already is: the file, the commit, the number, the command.
5. **Don't sign your own work, and don't simply adopt anyone else's.** What matters gets a reader who did not write it, and that reader measures rather than agrees. When you are corrected, take it whole; then find the mechanism.
6. **Go to the source; don't carry copies.** Read the roster, the amendment, the object itself: not your memory of it, a summary of it, or its title. A copy starts to rot the day it is made. A pointer does not.
7. **Keep the work moving, and hand it on whole.** Nothing waits for ever: every block names what releases it, who owns it, and when to look again. When your time is up, finish the act in hand and leave the next person able to start without asking anyone.
8. **Fix the hole before you post a sign.** When something bites twice, repair the tool, or make the tool refuse the mistake. Write a rule only when nothing can enforce it, and when you add one, look for the one it replaces.
9. **Protect the people who are not in the room.** Other teams, other lanes, anyone named in a record. They cannot consent through you, and the owner cannot consent for them. When in doubt, leave it out and ask.

## 3. The files that hold the fleet together

Keep these in one private repository that every seat can read. The agent's own session cache is disposable; **the repository is the record.**

- **The roster:** one row per seat: name, working directory, terminal window name, account or quota pool, model, effort level, and how it is launched. It is the only authority on who runs where. Never copy a roster value into prose; point at the row.
- **The objectives file:** one row per objective the owner has stated, in the owner's words, with a state (candidate, open, satisfied, retired) and a review-by date. An objective never lapses; only the owner opens, satisfies or retires one. A new objective is written the day it is spoken. A sweep refuses an open row with no review-by date.
- **The desk:** one row per decision or task in flight, with an owner, a default and a date the default fires.
- **Per-seat boot briefs and banks** (section 5).
- **Per-seat memory**, written to the repository and copied into the agent's cache before each launch, never the other way round.
- **Standing queues:** each lead seat keeps a short queue of pre-approved next items. A queue whose every item is blocked counts as empty, and the lead writes the next three from its campaign's own plan.

## 4. The bus

One append-only text file that every seat reads and writes. It is the fleet's only shared channel.

- **One post, one header line:** `[time, seat=STATE — seat → addressee · addressee — subject; receipt]`, then the body. The arrow names who owes an act; everyone else is mentioned in the body. The header is what watchers and gates match against, so its form is fixed.
- **Byte offsets are the only reliable order.** Cite a post by the byte offset where it starts. Timestamps are typed by writers and drift.
- **Append through one tool, never by hand.** It builds the post as a file, records the file size before appending, appends, then compares the bytes at that offset with what was sent and prints the verdict. Prose never goes through a shell format string.
- **Never edit the bus.** Corrections are new posts that point at the old offset.
- **Read to the end of the file in the same command as any irreversible act** (a merge, a stamp, a post that assigns work), and refuse if a new header has appeared since your last read. An end-of-file read in an earlier command does not count: someone may have withdrawn the instruction meanwhile.
- **Read the post, not the preview.** A watcher that shows the first N characters keeps the praise and cuts the correction.

## 5. A seat's life

**Launch.** A loop script per seat: read the seat's gate (below); if it says go, copy memory from the repository into the cache, launch the agent with a fixed boot prompt, and wait for it to exit; then read the gate again.

**Boot.** The boot brief is the first thing a seat reads, and the most important file the seat has.
1. Prove the brief is current: fetch the repository and refuse to continue if the local copy is behind. A document cannot attest to its own freshness.
2. Read the brief whole, and confirm the reader did not truncate it (check the last line). Keep the brief well under the reader's size limit; every addition owes a subtraction.
3. Read the last hand-over note ("the bank").
4. Arm the seat's watches and confirm each one is running by its process, not by belief.
5. Post a marker line whose first body line is exactly `BOOTED <seat>`. Detect it by position, never by substring.

**Work.** Pull the highest-priority item from your queue that your tier may attempt (appendix A.7). Classify its difficulty before attempting it; give up early and loudly (a written failure record) rather than grind.

**Measure your context.** Read the session's own context gauge before any decision that turns on it (appendix A.6 says how). An estimate is not a measurement, and estimates tend to be wrong by a wide margin in both directions.

**Bank.** At a fixed measured trigger (we use 80 % of the context window), or at a natural seam reached earlier, finish the act in hand and write the bank:
- a five-line status block: STATUS, BLOCKED (with a since-date, or "none"), NEXT, NEEDS, RISK;
- what is owed, item by item, with what releases each;
- every background job still running, by its ID, so the next session takes it over rather than starting a duplicate;
- what this session got wrong, and the lesson;
- the limits: what it did not read.

Post `BANKING <seat>` before writing it. The duty officer reads the bank against its exact file version and signs it (complete, complete with limits, or incomplete, plus findings). The next session starts from the bank.

**Clear or exit.** Stop every watch and timer you own first (a live background task can hold the exit open behind a confirmation dialog with nobody there to answer it). For the helm, use an external relighter process that waits for positive evidence the clear landed before it types the next boot prompt; the session that is about to stop observing cannot check its own restart.

**Auto-compaction is an event.** If the agent's context is compacted underneath it, say so on the bus, rebuild from the files rather than memory, and name what was re-derived.

## 6. Gates and the keeper

Each seat has a gate file. **Line 1 is a verb, executed by a supervisor process (the keeper)**, never a note:
- `relight: now`: launch at the next loop pass;
- `relight-on: <event in words>`, followed by one `probe:` line per event: wait;
- `down`: deliberately closed; only the helm or the owner reopens it.

Probe kinds: a bus probe (a regex over headers after a byte offset, e.g. any post addressed to this seat), a file probe (a file appears), a date probe. **Every event named on line 1 needs its own probe**, and the gate declares how many arms it has, so a sweep can flag a gate that promises two events and watches one. A seat waiting on a post addressed to it should always carry a bus probe; otherwise the post arrives and nothing wakes the seat.

The keeper is a system service (launchd, systemd) that evaluates probes on a cadence and rewrites a fired gate to `relight: now`, keeping the old text below as a comment. The posting tool should warn before a post fires a gate, so a writer relights a seat on purpose, not by accident.

The helm sweeps every gate at boot and at each seam, and asks of each held gate: was the condition met, and should this gate still exist?

## 7. Watchers

Small durable loops (a sentinel, an idle watch, a context-compaction watch, a quota watch), each a singleton with a pidfile, writing to logs the helm tails.

- **An idle seat with live background work is waiting, not stalled.** The watch should read the session's footer for running tasks before raising an alarm, and should never type into a seat's input box on its own.
- **Alive is not current.** A long-running script keeps running the code it started with, so a landed fix is not a running fix. A currency check compares each running watcher's start time with its source's last change.
- **A watch you can see working is not a watch anyone else can see.** Each arm is declared in one file and checked by a tool, not by the arming session's memory.
- **Silence is not success.** A filter must match every terminal state, not just the good one.

## 8. The executor registry

Any work that outlives the turn that started it (a subagent, a detached process, a remote job) gets a registry row at launch: shape, how to test it is alive, where it lands, who harvests it. At its end the row is closed with its exit code. A seat cannot exit cleanly while a finished job's output sits unread.

## 9. The duty officer and the helm's inspection

The duty officer posts a numbered digest every hour or so: routine acts taken (each with what was done, the instrument that justified it, and how to reverse it), what is waiting and on what, and what it did not look at.

The helm answers every routine act with one word: **CONCUR**, **QUERY** (asks for the reading; the act stands), or **REVERSE** (names the single act to undo). A quiet digest still gets an answer, "none this period, read", because an oversight mechanism whose output is "nothing to see" decays fastest during the quiet it exists to cover. Keep a running count of the three words.

The duty officer also signs the helm's banks, and the helm signs the duty officer's, so neither grades its own hand-over.

## 10. The daily council

One sitting a day, about thirty minutes, on the owner's schedule. Its purpose is to spend the owner's attention only where a human is needed.

**What reaches the owner:** a change of intention (open, satisfy, retire or re-rank an objective; start a campaign); anything outward or irreversible (publishing, deleting history, crossing a lane, touching a third party, spending past the budget); a fork that survived a deliberate attempt to refute it. **Everything else the helm decides**, written as a statement with its check: what was decided, what would show it wrong, and when it will be re-read. The owner reverses any of these with one word. If a choice can be checked and reversed, it belongs to the helm.

**Ask the owner for judgement, not for readings.** The owner is needed for taste and direction; a fact an instrument can take (a screenshot, a probe, a log) is the fleet's to take. Before asking the owner to look at something, name the instrument you tried and why it cannot take the reading.

**The pack:** at most six asks, each at most three lines: the objective it serves and why, the ask with a recommendation, and the default that fires at the close. An ask with no default cannot be "not reached"; an unanswered ask is its default. Beside it: the ledger (the helm's decisions since the last sitting, each one word from reversal), the owner's hands-on items, objectives due for review, and news. A lint tool checks the bounds before the pack is posted.

**During the sitting,** outgoing fleet actions that would change a line of the pack are held; seats keep working. The sitting closes only on the owner's explicit closure word, never on an inferred one. The minute records every item's disposition in the owner's words, including the ones not reached. After the close, the duty officer cuts a short "what is binding now" page from the minute, with every reference pinned by commit, and the helm ratifies it by file version.

**Uncertain about an internal choice?** Make a branch and go; the branch can be abandoned. Report a branch that changes what the project will publicly claim at council, as already taken.

## 11. Blocks, capacity and paging

- **No indefinite blocks.** A block carries a release condition, an owner and a re-measure date, and goes on a surface a sweep reads (a registry or a desk row, not a chat post). A blocked item never blocks a seat; it takes the next item.
- **Budget capacity per pool per day**, with a reserve kept unspent. A seat over its pool's line holds on the pool's clock, not on its work. Before moving a seat to another pool, read every pool and name why no other could carry it.
- **Paging:** a problem reaches the owner by an instant channel at any hour, once per episode, with a one-tap silence; routine updates go by email in the day.

## 12. Verification habits that paid for themselves

- **Non-author reads** report findings with their coverage, never a bare verdict, and drive the change on a scratch copy with the reader's own probes, including neighbours of the stated case.
- **Red first:** a new check is shown failing on the old code before it is trusted on the new. When the fix has already landed, restore the old behaviour as a deliberate mutant and watch the check fire.
- **Mutants run against stubs:** before running a mutant, replace every side effect the guarded path can reach (mail, pager, live state, network) with a refusing, logging stub, or the mutant will page someone.
- **Derived expectations:** an expected value is computed from the artifact, never typed.
- **Carry a read across a rebase** by showing the change's content is byte-identical at the new base, with a control that differs.
- **Read a merge back** from the forge: parents, tree equal to the reviewed head, no stray trailers.
- **A red check's name is a hypothesis:** read which step failed and its error text before acting.

## 13. False zeros: instruments that fail toward "fine"

Design against these from day one:
- an empty search that searched nothing (an ignore file, a wrong root, a pattern the tool silently rejected, an invalid byte that made it refuse the whole file);
- the exit code of a pipeline, which is the last stage's;
- a flag that does not bind and runs the default, exiting green (prove it binds against the tool's own fake first);
- a stale address (a window, a pane, a pid) that silently points at something else;
- a count compared with a population nobody measured.

When a nothing would be interesting, get it a second way along a different axis: a different tool, a different population, or ask the object to describe itself.

## 14. Lanes

If the fleet does work that must not mix (two clients, a confidential project, personal matters), give each its own lane, written down: which repositories belong to it, what may cross and in which direction. Ideas can flow where the owner says; **code crosses only with a licence and provenance; private records never cross.** A coordinator watching a fenced lane sees its behaviour only (whether it is running, its context gauge), never its content.

## 15. Where rules live

A rule is homed where the party bound by it will meet it in the ordinary course of work: the seat's boot brief, the repository's instructions file, or best of all a tool that refuses the wrong act and explains itself. A chat post, a meeting minute or a status file is a record, not a home. When you close a question, ask who is bound by the answer and what they will be reading when it applies.

## 16. Model levels

Most model vendors ship a family in three sizes: a fast, cheap model; a strong middle model; and a frontier model that is slower, dearer and scarcer. A fleet should use all three on purpose, by the work rather than by habit.

- **Tier the task, then pick the model.** Classify every task before it starts (appendix A.7: A one step, B routine, C real design, D open research). The fast model takes A and B, the middle model up to C, the frontier model design work and anything at D, which runs only with a human watching. A helper task inherits its parent's tier. A model that meets a task above its tier stops and writes it down for a higher tier; a lower tier never gates a higher one.
- **Give roles a steady model and a stepping-up rule.** Routine executors run on the fast or middle model. The duty officer runs on the middle model: its work is checks and routine acts, and it must be always on. The helm holds on the middle model day to day and steps up to the frontier model for the work that sets direction: the daily council, a design freeze, an ask to the owner. Statements, specifications and anything a later build will trust get the frontier model, because the statement is the expensive artifact and a proof or a build of the wrong statement is waste.
- **Put the model and the effort level in the roster**, one cell each, read by the seat loop at every launch. A setting typed by hand into one session is lost at its next restart, and a seat born tomorrow inherits whatever the default is.
- **Read a strong model's work with a different reader.** A non-author read is best done by a fresh session, and better still by a different model, so the reader does not share the author's blind spots. A design freeze gets a refuter pass (several readers, each with a named way to try to break it) before cheaper models build on it.
- **Check which model actually ran.** A runtime can quietly substitute a default for a model name it does not recognise. Read the model the session itself reports (its transcript or log), not the flag you passed.
- **Mind the window when switching.** Models in one family can have very different context windows. Switching a long session to a model with a smaller window can force an immediate compaction, so read the context gauge first and switch only when the session fits; otherwise start a fresh session on the new model.
- **Know what is fixed at launch.** In many runtimes the account (and so the quota pool) is fixed when the session starts, while the model can be changed inside it. A move between accounts therefore needs a restart, and a model switch may not.
- **Budget the frontier tier separately.** Quotas are often per pool and sometimes per model, and the frontier model's allowance runs out first. Plan its use per day with a reserve, and route work that does not need it elsewhere.
- **Don't answer pricing, limits or model names from memory.** They change; read the vendor's current documentation or the runtime's own report.

---

*What this guide does not include: any tool's code (each is described above so a team can build its own); any account, machine or quota detail; any campaign's results.*

---

## Appendix A: starter formats

Minimal forms that work. Change any of them; keep each one in a single place and make a tool read it.

**A.1 Roster** (tab-separated, one header line): `seat  dir  window  pool  model  effort  launch  notes`. *pool* is the account or quota bucket the seat's usage is charged to; *effort* is the agent's reasoning-effort setting, if the runtime has one; *launch* is `loop` (a seat loop starts it) or `hand` (the owner opens it).

**A.2 Objectives** (tab-separated): `id  opened  objective  priority  where-the-work-lives  owner  state  last_reviewed  review-by  closed  closing-words`. *state* is one of `CANDIDATE · OPEN · SATISFIED · RETIRED`; *closing-words* holds the owner's words for every change, appended, never rewritten.

**A.3 Gate file** (one per seat, outside every repository the seat edits):
```
relight-on: a post addressed to me, or the nightly result file
probe: bus 1048576 (→ ([^—]*( |·))?myseat([^A-Za-z0-9_-]|$))
probe: file /path/to/nightly-result
arms: 2
re-measure: 2026-12-01 (why that date)
```
`bus N (regex)` fires on the first header past byte N that matches; `file P` fires when P exists; `date D` fires on that day. The keeper rewrites a fired gate to `relight: now` with a comment line naming the probe and the matched header, and keeps the old gate below as comments.

**A.4 Bus post:**
```
[11/24 10:46:21, myseat=LIT — myseat → helm · otherseat — one-line subject; body receipt bytes=1489 sha256/16=0123456789abcdef offset-pre-append=1048576]
body text…
```
*STATE* is what the poster is doing: `LIT` (working), `BANKING`, `DARK`, `HOLD`. The receipt is the body's byte count, the first 16 hex of its SHA-256, and the file size before the append; the append tool computes all three and checks them after landing.

**A.5 Boot prompt** (typed by the loop into a fresh session): `BOOT: read <path to this seat's boot brief> whole, then follow it.` The brief's first lines do the freshness check, name the current bank, and list the watches to arm.

**A.6 Context gauge:** use whatever the runtime shows (a status-line percentage, a token count against the window). If it shows nothing, count the transcript's tokens yourself and divide by the window size. Whatever the method, the seat reads a number, never a feeling, and the bank trigger is a number.

**A.7 Tiers:** classify every task before attempting it. A: one step or computational. B: routine multi-step. C: real design (new definitions, long proofs, porting). D: open research, never attempted unattended. Map tiers to models: a cheap model takes A and B, a strong one A to C, the strongest also design work and D under a human's eye. A helper inherits its task's tier; if it feels harder than your tier, stop and write it down.

**A.8 Bank** (one file per hand-over; the boot brief's `BANK:` line points at the newest):
```
# BANK — <seat>, <date> <start> → <end>
## STATUS LINE
STATUS: … · BLOCKED: none | <what> since <date> · NEXT: … · NEEDS: … · RISK: …
## §0 WHAT MOVED (numbered, each with its receipt: commit, file version, bus offset)
## §2 OWED (each with its release condition and owner)
## §3 LIVE EXECUTORS (by ID; the next session takes them over)
## §4 WHAT THIS SESSION GOT WRONG
## §6 LIMITS (what it did not read)
```
The signer names the file and its exact version (a content hash), reads it whole, checks each claim it can at the object, and answers `COMPLETE`, `COMPLETE WITH LIMITS` or `INCOMPLETE`, with findings graded HIGH / MEDIUM / LOW.

**A.9 Executor registry row** (one small file per job): `seat  id  shape  liveness  marker  landing  harvester  fired-at  ended  rc`. *shape* is `subagent · process · tmux · remote`; *liveness* is how to test it (a pid, a file's age, the runtime's own notification); *landing* is where its output appears; the row closes with `ended` and `rc`, or is retired with a reason after its landing is read.

**A.10 Digest** (duty officer → helm): `HEADLINE · WORD NEEDED · ROUTINE ACTS (each: WHAT · INSTRUMENT · REVERSAL) · WAITS · RANGE (headers read) · NOT LOOKED AT · next digest floor and time`. The helm's reply carries one word per routine act and the running count `concur · query · reverse`.

**A.11 Watches a seat arms:** its bus watch (posts addressed to it, from its last read offset), and any timer or job monitor its work needs. Each is a background process the session owns, listed by name in a declared-arms file, and checked present by a tool after arming. Every one is stopped before the session clears or exits.

**A.12 Memory:** one directory per seat in the repository: an index file (one line per memory, each a pointer with a short hook) and one small file per memory (one fact, with why it matters and how to apply it). Keep the index under the runtime's load limit, and put a canary token on its last line so a truncated load is detectable. Before every launch, the loop copies the directory into wherever the runtime loads memory from. A session that learns something writes it to the repository copy, never only to the runtime's.

**A.13 Relighter** (for the helm, which cannot watch its own restart): a small detached process started just before the helm clears its session. It waits for positive evidence the clear happened (the context gauge falling sharply from its pre-clear reading, an empty input box, no pending text), then types the boot prompt once. If it cannot tell, it does nothing and writes a loud marker; a second, one-shot watch fires the boot prompt only if no `BOOTED` post has appeared within ten minutes.

---

## Appendix B: practical operation on one machine (tmux, loops, keystrokes, hazards)

**B.1 Layout.** One tmux server. A session (say `fleet`) with one window per seat, named as the roster's `window` column, the helm in its own window. Optionally a second session that links the windows of the seats currently working (`tmux link-window`), so the owner can watch everything in one place. **Address a pane by its id** (`%23`), found by listing: `tmux list-panes -a -F '#{pane_id} #{session_name}:#{window_name} #{pane_current_command}'`. Prove a pane exists by that list, never by a successful capture: a stale address can silently resolve to a different pane.

**B.2 The seat loop.** A shell script running in the seat's window, and the process tmux actually keeps: loop { read the gate's line 1; if `relight: now` (or no gate, logged loudly), read the seat's roster row (model, effort, account) and launch the agent CLI in its directory with the boot prompt; when the agent exits, record the exit code and re-read the gate; while the gate holds, sleep about 30 s and re-read }. Because the loop re-reads the roster at each launch, a roster change takes effect at the next launch, not when a session merely clears its context. Copy memory from the repository into the runtime before each launch.

**B.3 Sending text to a seat.** Two steps: `tmux send-keys -t %ID -l '<text>'`, then, after a short pause, `tmux send-keys -t %ID Enter`. A long text arrives as a pasted block, and some interfaces then need one more Enter. **Verify delivery by the agent's own transcript or log, never by the screen:** unsubmitted text in an input box looks exactly like delivered text. Never send keys into your own pane (they append to whatever is half-typed there). Start every dispatch with its source (`HELM:`, `OWNER-RELAY:`), with "unknown" as the default when none is given. Keep a pasted message under about 1 KB; a longer one goes in a file that the message points at.

**B.4 Reading a seat.** `tmux capture-pane -p -t %ID` and read the last few lines (the footer): the context gauge, the model, and any running background tasks. Dim text in the input box (escape code SGR 2) is the interface's suggestion, not something anyone typed; capture with `-e` to see it. For a seat in a fenced lane, read the footer line only, never the transcript area.

**B.5 Modals and exits.** A confirmation dialog blocks a seat until someone answers it. Stop every background task a session owns before it exits, or the exit itself raises a dialog nobody is there to answer. If a dialog must be dismissed from outside, `Escape` sent by key name may not reach it; a literal double escape (`tmux send-keys -t %ID -l $'\e\e'`) has worked where a single one did not. Let another seat press a key in your pane only when you announced the act first, the dialog is one you cannot clear yourself, the pre-selected option is exactly the announced act, and the seat posts what it saw before pressing.

**B.6 Locale.** Run tmux with UTF-8 (`tmux -u`, or a UTF-8 locale in the environment). Without it, some tmux versions print a tab as `_` in `-F` output, and every tab-split parse then fails quietly.

**B.7 Services versus loops.** The keeper can be a system service (launchd, systemd) if it touches only ordinary paths. On macOS, a launchd agent does not inherit the file-access grants that a terminal holds for protected folders, so watchers that read protected locations run as `nohup` loops started from a granted terminal. Each one writes a pidfile and refuses to start twice. Keep the list of loops in one file, because a reboot kills them all and they are re-armed from that list.

**B.8 Stopping things.** Kill by pid: read the pidfile, confirm with `ps -p <pid> -o command` that it is the process you mean, then kill. Never `pkill -f <pattern>` or `killall`: a pattern also matches your own supervisor. If a process you killed comes back, you killed a child, and its spawner restarted it; walk the parent pids up to find the right layer. Guard every `rm` that uses a variable (`"${DIR:?}"`), and prefer `find … -delete` for generated paths.

**B.9 Heavy jobs.** Allow one heavy job at a time across the fleet (builds, large solves): a lock taken by an atomic `mkdir` with a pid file inside, a free-memory floor checked inside the lock, and abort rather than proceed when the floor is not met. Builds go through a wrapper that takes the lock and caps threads, and are judged only by the wrapper's own printed exit line, never by a pipeline's status.

**B.10 Other machines.** A seat on another machine runs in a tmux window there and is reached by ssh. Check the network (the mesh VPN's status, a ping) before concluding the seat is at fault. A laptop on battery can sleep through a night job: the job should refuse to start off mains power, and hold the machine awake for its duration (`caffeinate -i -s -w <pid>` on macOS, `systemd-inhibit` on Linux).

**B.11 Small traps.** macOS has no `timeout`: run the command in the background, sleep, then kill it by pid if it is still alive. A pipeline's exit code is its last stage's (`${PIPESTATUS[0]}` for the first). A command that might print nothing writes to a file, and you read the file. An architecture shim can print success without running anything, so require the subject's own output (for example `uname -m`).

**B.12 One checkout, many writers.** Seats sharing a repository checkout commit through one tool that holds a lock, commits by path, pushes, and verifies, never a bare `git commit` in the shared tree. Pull with peers' work in mind: never autostash someone else's uncommitted file.

---

© 2026 Jason Hickey. Licensed under the Creative Commons Attribution 4.0 International License (CC BY 4.0); see [LICENSE](LICENSE).
