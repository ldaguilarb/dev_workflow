# Dev Workflow - how to use it

A copyable folder that makes an AI coding agent build software the disciplined way: align first, then plan, implement and verify (the APIV loop), through 12 gated stages. You talk to the agent; the folder tells it what to do next, a script decides whether a stage may pass, and you sign off at a handful of real decision points.

This file is the walkthrough for people. Agents start at [AGENTS.md](AGENTS.md); the pipeline is described once in [CONTEXT.md](CONTEXT.md).

## The idea in one minute

- Work that has not started waits in the **backlog**: one small file per item. A big goal is first cut into items, and the script groups them into waves that can be built in parallel.
- Every piece of work (a new project, a feature, a fix) is one **delivery**: a folder under `deliveries/`, started from a backlog item.
- A delivery walks up to 12 numbered stages. Stages 01-06 are alignment and produce documents only. Product code starts at stage 08.
- Each stage has a contract (`CONTEXT.md`) naming what to read, what to write and how it is checked.
- **A stage passes only when `flow.py pass` says so.** The script checks that every requirement has a case, a design sentence, a plan slice, code that exists, a test that exists and a proof, as each becomes due.
- **Nobody grades their own work.** The brief is attacked by a challenger that never saw your conversation; review and verification run in fresh agents that did not write the code.
- **Signed-off documents cannot drift.** Once a stage passes, its documents are fingerprinted; changing one means reopening its stage.
- It works for one person or for a team. Setup asks which, and team mode adds a branch, a pull request and a progress log per delivery.
- Everything is plain files you can open and correct.

## Requirements

Python 3.8+ on the machine (the gates are a Python script, standard library only) and git for the project. Team mode also uses a GitHub repository and, for the acceptance and merge sign-offs by pull request review, the GitHub CLI (`gh`) signed in on each machine.

## Install in a project (once)

1. Copy the whole `dev-workflow/` folder into the root of your project. Copy from a clean master: `deliveries/` and `backlog/` empty (keep `backlog/CONTEXT.md`), and `_shared/project.md`, `ontology.md`, `architecture.md`, `signals.md`, `lessons.md` and `decisions/` free of another project's content.
2. Open your agent in the project and say: **"Set up the dev workflow."** It runs [the setup questionnaire](setup/questionnaire.md). In an existing project it reads the repo, drafts the answers and the as-built architecture, and asks you to confirm. It then:
   - asks you whether **one person or several** will work on the project, and for a team: the main branch, who may sign off, and how;
   - writes [_shared/project.md](_shared/project.md): stack, exact test / lint / build commands, what must never break, standing quality bars, solo or team mode;
   - runs `python dev-workflow/_tools/flow.py install`, which points the project's `AGENTS.md` and `CLAUDE.md` at the workflow, installs a pre-commit hook and, in team mode, writes the `.gitattributes` and `.gitignore` lines;
   - sets up CI from `_templates/ci/github-actions.yml`;
   - runs `python dev-workflow/_tools/flow.py doctor` to prove the commands work.
3. Commit the folder with the project. The documents are part of the product's history.

## Plan a big goal: the backlog

For anything larger than one delivery, say: **"Map this: I want a customer portal with login, invoices and support tickets."** The agent interviews you about the outcomes, writes a short map, and cuts the goal into backlog items, each small enough to ship in a few days, each naming the modules it touches and the items it depends on. `flow.py backlog` then shows the waves:

| Wave | Meaning |
|---|---|
| `now` | can start today: dependencies done, no module shared with running work |
| `2`, `3`, ... | can start when the wave before is done; items inside one wave can run in parallel |
| `blocked` | waits on something missing or dropped |

Dependencies are a hard rule: an item cannot be started until everything it depends on is shipped (its delivery finished and, in a team, merged into the main branch). The script refuses, and there is no override.

You approve the map once (`backlog approve`). After that, "what's next?" is answered by the backlog, and "start 004" begins that item. Work also flows back: a delivery that is split at stage 01, or defers something at stage 11, puts those pieces on the backlog instead of leaving them in a closed folder. The details are in [backlog/CONTEXT.md](backlog/CONTEXT.md).

## Start a piece of work

Say what you want in plain words: **"Start item 004"**, or for a one-off, **"New delivery: visitors should be able to reach my contact details in one click."** The agent runs `flow.py new`, which stamps `deliveries/004-<slug>/` (a one-off gets a backlog item and number on the way), and starts the stage 01 interview.

| Track | Stages | Use for |
|---|---|---|
| Full | all 12 | new project, a feature with UI or new data, anything crossing several modules |
| Medium | 01, 04, 06, 07-12 | a feature inside the existing architecture: no new UI, no new data model, nothing to research |
| Lite | 01, 07-11 | a small, contained change |
| none | - | trivial or throwaway: just ask the agent directly |

You do not pick the track and neither does the agent. At stage 01 the brief answers five yes/no questions, and the script derives it:

| If the answer is yes to | Track |
|---|---|
| new or changed UI, new or changed data, or unproven assumptions | Full |
| otherwise: more than one module, or touches the must-never-break list | Medium |
| none of them | Lite |

The agent tells you the result before stage 01 passes, and the challenger checks the answers against the requirements. To override, say so: the agent writes the track and your reason into the delivery's `CONTEXT.md`, and the gate refuses an override without a reason. Stages outside the track are skipped automatically, as are boards on a full delivery with no UI. Keep deliveries small: if it cannot ship in a few days, stage 01 splits it and puts the rest on the backlog.

## The stages, and what you do

| Stage | The agent | You |
|---|---|---|
| 01 outcome | interviews you; writes the brief: business requirements (BR-n) each with a concrete example, quality requirements (NFR-n), a pre-mortem, a success signal; fixes one word per thing in the ontology. Then a **brief challenger** that sees only the brief lists everything vague, unmeasurable or unanswered, until nothing is left | describe outcomes, not solutions; answer the challenger's questions |
| 02 patterns | finds code, libraries and open source to reuse; proves every assumption with a throwaway spike; records decisions | **Gate A**: approve brief, terms and reuse decisions |
| 03 boards | builds throwaway HTML mockups | iterate freely, then declare each board final |
| 04 enumerate | lists every case and combination, starting from your examples | - |
| 05 data model | designs the change to the architecture; a fresh agent replays every case against it | - |
| 06 ledger | links each requirement to design sentences (FR-n.m) | **Gate B**: approve cases, design and ledger |
| 07 plan | writes the sliced plan, with lanes if work can run in parallel | **Gate C**: approve the plan |
| 08 implement | per slice: a failing test per case, then code, doctor, commit | nothing, unless it stops on a gap |
| 09 review | a fresh agent reviews the diff: plan conformance, slop, design, security, test quality | nothing; blockers and majors must be closed |
| 10 verify | a fresh agent proves every requirement, measures every quality bar, runs the doctor (can run alongside 09) | **Acceptance**: try the running thing |
| 11 close | merges, folds the design into the architecture, updates docs, logs the success signal to check later | approve the merge |
| 12 lessons | turns every failure into a gate rule, test or template fix | agree the lessons |

On a full delivery you sign off six times (A, boards, B, C, acceptance, merge) plus the lessons chat; on medium four; on lite three. The first gate you reach also covers the brief. Everything else is checked by script or by an independent agent.

A stage that does not apply (boards with no UI) is skipped with a one-line reason in `output/skipped.md`.

## Commands

Run from the project root. `DELIVERY` is optional when only one delivery is active.

| Command | What it does |
|---|---|
| `python dev-workflow/_tools/flow.py backlog` | list the backlog with its waves; rebuilds `backlog/_index.md` |
| `python dev-workflow/_tools/flow.py backlog add "<title>" [--outcome ..] [--modules a,b] [--depends 1,2] [--group name] [--size S\|M\|L]` | add a backlog item with the next free number |
| `python dev-workflow/_tools/flow.py backlog approve <group> --by "Name"` | check a map and record your approval |
| `python dev-workflow/_tools/flow.py new <number>` or `new "<title>"` | start a delivery from a backlog item, or a one-off |
| `python dev-workflow/_tools/flow.py status [--all]` | where every delivery stands; rebuilds `deliveries/_index.md`. `--all` reads every branch (team mode) |
| `python dev-workflow/_tools/flow.py gate [DELIVERY]` | dry run: would the current stage pass, and if not, why |
| `python dev-workflow/_tools/flow.py pass [DELIVERY] [--by "Name"]` | run the gate and, if green, record the pass; `--by` is required at your sign-off stages |
| `python dev-workflow/_tools/flow.py reopen <stage> [DELIVERY]` | reopen a stage and everything after it; old sign-offs are kept as `check-superseded-N.md` |
| `python dev-workflow/_tools/flow.py doctor [--fast]` | run the project's test, lint and build commands plus the workspace checks (links, duplicate numbers) |
| `python dev-workflow/_tools/flow.py install` | pointer lines in `AGENTS.md` / `CLAUDE.md`, pre-commit hook, team-mode git settings |

In practice you mostly talk: "continue the current delivery", "approved" (the agent then runs `pass --by` with your name), "reopen stage 03".

## Working as a team

Setup asks whether one person or several will work on the project; the backlog map asks again when a goal has work that could run in parallel. To switch later, change the `Mode` line in `_shared/project.md` and run `flow.py install` again.

In team mode each person, with their own agent, owns one delivery from brief to merge:

- **Claim.** `flow.py new 004` cuts the branch `delivery/004-<slug>`, pushes it and opens a draft pull request. The branch is the claim; nobody else can start 004.
- **Progress.** Every pass, refusal and reopen lands in the delivery's `log.md` and is pushed, so the pull request shows where the work stands. `flow.py status --all` prints every delivery on every branch with its owner, stage and last log line. Nobody writes a status update by hand.
- **Sign-offs.** Gates A, boards, B, C and the lessons are given in chat, as when working alone. Acceptance (10) and the merge (11) are approving reviews on the pull request: the script checks for one at the exact commit, from an allowed login that is not the delivery's owner, so an agent cannot sign for you and nobody accepts their own work.
- **No collisions.** Numbers are taken after looking at every branch. The plan gate refuses a plan that touches the modules of another delivery whose plan is signed. Stages 10 and 11 refuse a branch that does not contain the main branch.
- **Landing.** The pull request is merged after the last stage passes. Protect the main branch with a merge queue or "require branches to be up to date".

Bigger deliveries can also be split into lanes at stage 07, each lane with its own owner and branch. What changes in team mode is listed once in [CONTEXT.md](CONTEXT.md).

## When something is wrong

Go back, do not patch forward. A failed proof or a result you do not like is traced to the earliest stage whose document is wrong: a missing requirement reopens 01, a disproven assumption reopens 02, a wrong look reopens 03, a missed case reopens 04. Reopen the stage (`flow.py reopen 01`) and fix the document; later stages then update only what the change touches, their gates rerun, and you re-approve just the difference. The gates enforce this: a signed-off document that changed without a reopen is refused. Changing code to match a new idea without changing the document is how slop gets in.

## What keeps quality high

- **The brief is attacked before anything is built.** The challenger must be able to turn every requirement into a pass/fail test and have no questions left.
- **Quality is written down.** NFRs carry a measurable bar at stage 01 and are measured at stage 10; project-wide bars live in `_shared/project.md`.
- **Assumptions are proven early.** "The library can do X" gets a spike at stage 02, before any design depends on it.
- **Cases are tests.** Every case in the matrix becomes an automated test at stage 08, named for its case, and the gate looks for that test in your test files, so the regression suite grows with each delivery. Small (lite) deliveries carry one test per requirement instead.
- **One living architecture.** [_shared/architecture.md](_shared/architecture.md) describes the system as built; each delivery designs a change to it and folds it back at close. The reasons live in `_shared/decisions/`.
- **The same doctor everywhere.** One command runs on each slice, in the pre-commit hook (fast mode), in CI, and inside the stage 10 gate. It fails if the test command was never configured.
- **Outcomes are checked later.** Each delivery logs its success signal in `_shared/signals.md`; the next delivery's interview starts by asking what happened.
- **Mistakes are paid for once.** Stage 12 prefers the most automatic fix: a new gate rule beats a new sentence.

## Tips

- Spend your time in stages 01-06. If alignment is right, the rest is close to automatic.
- Talk about problems by number and ontology term: "BR-3 fails in case C-14."
- Iterate on boards as much as you like, and never in production code.
- Start each stage in a fresh agent session when you can; the files carry everything it needs.
- The gate reads table columns, so keep the exact headers the stage contracts give, and never put a `|` inside a cell.
- Match the model to the stage: the Effort column in [CONTEXT.md](CONTEXT.md) marks which stages deserve your strongest model and which a small fast one handles.
- One delivery at a time per area of the code. The backlog's waves show what can run side by side, and the plan gate refuses two deliveries on the same modules.
- Glance at `git log` for `check.md` files now and then: every sign-off recorded in your name should be one you gave.

## Not covered yet

Release and operations (rollout, data migration, rollback, monitoring) have no stage. The workflow ends at a proven merge, with the success signal logged for a later look.

Known limits: the script proves that tests and files exist and are named for their cases, not that they are meaningful (that is the reviewer's and your job). The `checker:` line on each independent verdict is a statement, not proof that the checker was fresh. Solo, or with sign-offs in chat, the script cannot stop an agent from recording a sign-off you did not give, only make it visible; with sign-offs on GitHub it can.

## What is in the folder

| Path | What it is |
|---|---|
| `AGENTS.md` | the agent's entry point and routing table |
| `CONTEXT.md` | the pipeline, gates, tracks and status rules |
| `_shared/` | golden rules, project facts, ontology, architecture, decisions, success signals, lessons |
| `_templates/` | the blank delivery pipeline, the decision template, CI and hook starters |
| `backlog/` | work not yet started, the maps of large goals, the generated tracker `_index.md` |
| `deliveries/` | your deliveries, each with its `log.md`, plus the generated tracker `_index.md` |
| `_tools/` | `flow.py` and `links.py` |
| `setup/` | the one-time questionnaire |
"# dev_workflow" 
