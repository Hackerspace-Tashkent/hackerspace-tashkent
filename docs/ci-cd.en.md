# Why our CI/CD looks like this

Date: 2 October 2026
Status: decision taken, part of the rollout blocked on a token permission

## In short

The checks existed from the beginning, but they ran manually, on my machine, after I had already written the material. That arrangement missed defects — not because the checks were weak, but because they ran too late.

We are not building CI to look professional. We are building it so that what already broke cannot get into a repository twice.

## What we found while checking everything by hand

In September we walked all 17 practices as a cold reader: not against my own reference solution, but literally doing what the document says. We found:

- **A fused code fence** in the S0-01 lesson, in all four languages at once. Copy the code out of the lesson and you get a `SyntaxError`. The first person stops at the first step of the first practice.
- **A block in S0-03 used a variable `PASSWORD` that it never assigned.** `NameError`. The check could not pass by any route.
- **In S1-01 the word `sqlite3` appeared zero times**, and there was no command to start the server. The task required something the text never taught.
- **The S0-01 check accepted any process on port 8000**, including someone else's.
- **The S0-02 check printed the invented secret** the student was supposed to find. Copy it from the output and the task is solved without ever opening the history.
- **The harness never ran the fixtures.** It measured a state a student is never in. It also looked for a file named `setup.sh`, while S0-02's is called `setup-repo.sh`, so that fixture never ran at all. For a long time the report said "17 of 17 practices reject the empty state". That was false, and I kept repeating it.
- **A dead domain in the README** surfaced only during a manual link audit. A reader would have hit it and never come back.

Not one automated check caught any of these. All of them surfaced because I read the document by eye.

**Hence the main conclusion:** checking "does the script work" and checking "is enough written to reach the end" are different things. No tool does the second one. So CI is not a substitute for reading. It exists to back up what reading no longer catches.

## What runs in each repository

The repositories differ, and one heavy pipeline across a 16-file repository would be theatre.

| repository | files | what runs | when |
|---|---|---|---|
| `hackerspace-tashkent-learning` | 214 | full harness, content validator, cold pass | every push and PR |
| `Hackerspace-Tashkent-website` | 7 | HTML validation, og-tags and image present, dead links | every push |
| `hackerspace-tashkent` | 16 | link check, secret scan, code of conduct | push and weekly |
| `hackerspace-tashkent-projects` | 5 | link check, hardware file integrity | push and weekly |

The heavy harness lives in `learning` only. The others do not need it.

## Practices we chose

**Branch protection: merge only on green.** The most valuable practice on the list. Without it CI is advice. With it, it is a gate. This one matters especially to me: nearly every defect above was pushed by me and caught after the fact.

**`timeout-minutes` on every job.** The practices start HTTP servers and wait for ports. A hung job without a timeout sits for six hours and eats the free minutes.

**Least privilege.** Read-only by default, widened per job. `board.yml` already works that way: `contents: write` and `issues: write`, nothing else.

**Actions pinned by SHA, not by tag.** `uses: actions/checkout@v4` is a floating reference someone else can move. That is a supply of code into our pipeline.

**Weekly scheduled link check.** Our material is documentation. A dead link does not fail a build — it simply pretends to work.

**No secrets in CI.** The tests need no tokens. That is not thrift, it is a requirement: a check that can run on someone else's fork without handing out secrets is fundamentally stronger than one that needs them issued.

## What we deliberately do not do

- **A matrix across operating systems.** We have no cross-platform code. Fifteen minutes of work for zero benefit.
- **Security scanning.** On a five-file repository it produces noise, and noise trains you to ignore red.
- **Dependabot.** There is nothing to update yet.
- **One shared pipeline for all four repositories.** The organisation is not ready for shared templates: four languages, four structures, different risk.

None of these decisions are permanent. When third-party code, cross-platform libraries, or a repository that is no longer small appears, we revisit the list.

## Current blocker

No workflow has landed, because the fine-grained token lacks the **Workflows** permission. Until that changes, `board.yml` sits locally and never enters the index.

Three routes, in increasing order of effort for you:

1. Grant the token `Workflows: Read and write` — after that we push everything ourselves.
2. Create the first workflow through the web interface: Settings → Actions → New workflow.
3. Wait and keep working locally, as now.

Separately from the token: **protecting `main` can be enabled today.** It requires no workflow at all and immediately closes the hole broken content came through.

## How we know this decision is working

- a red status blocks the merge
- no practice can reach `main` unchecked
- a dead link surfaces within a week instead of on a reader's complaint
- the checks run on someone else's fork without issuing secrets

## What this document does not promise

A green CI means the scripts run, the lessons contain enough to reach the end, and the links are alive.

It does not mean someone understood them. There are zero live completions of the program to date. Until one person has walked a topic end to end, the honest thing to say is "the scripts run", not "the program works".

Any change to this decision — to the reasoning or to the workflows themselves — must update this document. A decision without a record of why it was made looks, six months later, like the author's whim.