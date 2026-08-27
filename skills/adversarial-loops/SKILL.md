---
name: adversarial-loops
description: "Use this skill to verify AI-generated work before you trust it, by running a creator/critic loop where a second, fresh AI session attacks the output until it holds up. Trigger whenever the user asks 'is this actually right?', wants a second opinion on something AI built, is about to ship anything high-stakes (payments, auth, data migrations, pricing, legal copy), notices the AI keeps approving its own work, or asks how to catch the bugs and gaps a single AI pass misses. Also use when setting up a repeatable review loop for specs, code, schemas, or copy."
---

# Adversarial Loops

An LLM will hand you work that reads well. "Reads well" and "is correct" are different things.

A single AI pass glosses over edge cases, makes assumptions without telling you, and produces code that works for the examples it had in mind and breaks on the ones it didn't consider. Then, if you ask it to check its own work, it rubber-stamps it.

The fix is structural: pair a **creator** with a **critic**, and keep them apart. The creator produces. The critic attacks. They alternate rounds until the work holds up or you hit a stopping point. Neither one alone is as good as the pair.

The pattern is borrowed from GANs — a generator making things, a discriminator judging them, and the tension between them driving quality up. The difference: there's no training here, so instead of gradient descent you steer the loop with heuristics — severity thresholds and a hard round cap.

*Pattern adapted from [LLMs and the Adversarial Loop](https://myers.io/2026/04/15/LLMs-and-the-Adversarial-loop/) (myers.io, April 2026).*

---

## Why "please double-check that" doesn't work

Asking the same chat to review its own output barely helps. The session context anchors the model to its own prior reasoning — it's already committed to the assumptions it made, so it reviews the work it *meant* to write instead of the work on the page.

A fresh session starts cold. It has no stake in the earlier decisions, and because sampling is stochastic, it genuinely reads the thing differently. That's the whole trick, and it's why the critic must live in a **separate conversation**, not a follow-up message.

You give up something real here: the critic doesn't remember the last round, so it may re-raise something you already fixed. That waste is the price of independence, and it's worth paying. A prior-rounds summary (below) softens it.

---

## The loop

```
Round 1
  Creator:  produces the work
  Critic:   fresh session, reads only the work + the requirements
            → structured issues + verdict

Round 2..N
  Creator:  fixes the issues (same session as round 1, keeps context)
  Critic:   fresh session again, re-reviews from scratch
            → structured issues + verdict

Stop when a stop rule fires. Then a human looks at it.
```

Two rules make this work rather than just burn tokens:

**The critic gets the artifact, not the transcript.** Give it the code, the spec, the page — plus the requirements it's supposed to satisfy. Do not give it the creator's reasoning or its own previous review verbatim. Reasoning is persuasive; that's the problem.

**The critic doesn't edit.** Its only output is a list of issues. The moment it starts rewriting, you've lost the adversary and gained a second creator.

---

## The critic's output format

The critic can't be allowed to say "this could be tightened up." Vague feedback produces vague rewrites, and a vague rewrite churns working code. Require this shape:

```
Issues:
1. category:    [correctness | security | edge case | spec gap | data | performance]
   description: [what's wrong, where, and what happens as a result]
   severity:    [critical | major | minor]

2. ...

Verdict: [approved | needs work]
```

Severity definitions worth stating up front, because models drift on them:

| Severity | Means |
|----------|-------|
| **Critical** | Loses data, loses money, exposes something private, or breaks the main flow |
| **Major** | Wrong behavior in a real case a user will hit |
| **Minor** | Style, naming, a nice-to-have, an unlikely edge case |

Concrete issue + severity is what makes the next round a targeted fix instead of a rewrite.

---

## Stop rules

The loop needs an exit or it runs forever. Any of these ends it:

```
- [ ] Critic returns "approved"            → clean exit
- [ ] Only minor issues remain             → good enough, log them and move on
- [ ] No new major issues after round N/2  → stale loop, it's fixated
- [ ] Round cap reached (default 3)        → hard stop
```

The stale-loop rule matters most in practice. If the critic keeps raising the same things round after round, the loop is stuck — continuing just spends money to hear the same opinion again.

**When it doesn't converge, don't throw the work away.** Take the best version you have, write down the unresolved issues, and move on. Some review beats none. If you're looping over 28 work items and 5 never converge, partial progress on those 5 plus 23 clean ones beats holding the whole thing hostage.

---

## Give both sides a prior-rounds summary

Starting the critic cold each round is the point, but starting it *blind* wastes rounds. Include a short summary — not the transcript:

```
Prior rounds:
Round 1: 6 issues (1 critical, 3 major, 2 minor). Critical and majors fixed as follows: [one line each].
Round 2: 3 issues (2 major, 1 minor). Majors fixed as follows: [one line each]. Minor accepted: [reason].
```

It's lossy on purpose. Enough to stop the critic re-raising settled ground, not enough to sell it on the creator's reasoning.

---

## Loop over one item at a time

Adversarial review of "the whole app" returns generic advice. The loop works when the thing under review is small enough to hold in one head.

```
- [ ] Break the work into items (one feature, one endpoint, one page, one table)
- [ ] Run a separate loop per item
- [ ] Commit each item as it converges
- [ ] Keep a list of items that didn't converge, with their open issues
```

This is also what makes non-convergence survivable — a stuck item is one item, not the release.

---

## When it's worth the cost

Every round is at least two AI calls, and a review round costs roughly what the original generation cost. A 3-round loop roughly doubles-to-triples the spend on that piece of work, and because rounds are sequential, it takes about 3x as long. A phase that cost $2 and 20 minutes becomes $4–6 and an hour.

So spend it where a mistake is expensive:

| Loop it | Skip it |
|---------|---------|
| Anything touching money — checkout, subscriptions, refunds | Copy tweaks, button labels, spacing |
| Auth, permissions, row-level security | Styling and layout work |
| Database migrations, bulk updates, anything with `DELETE` | Throwaway prototypes and spikes |
| Data you can't reconstruct if it's wrong | Anything that fails loudly and immediately in front of you |
| Third-party integrations and webhooks | Work you're about to replace anyway |
| Public claims: pricing, guarantees, legal, security pages | Internal notes and scratch work |
| The first version of a core spec, before anyone builds on it | The fifth revision of a paragraph |

Rule of thumb: if you'd notice the mistake within a minute of using the app, skip the loop. If it fails quietly — wrong number in a report, a permission that's too broad, a webhook that silently drops events — loop it.

---

## Running it in your tools

**Claude Code** — you can run both sides. Do the work, then open a *fresh* context for the critic (a subagent, a second session, or `/clear` then paste the artifact) and hand it the diff plus the requirements. Do not let the critic session write files. Feed the issue list back to the original session to fix.

**Lovable / Replit / Cursor / ChatGPT** — start a **new chat** for the critic. Paste the code or content, paste the requirements, and use the prompt below. Bring the issue list back to the original chat to fix. Repeat in another new chat.

**Mixing models** helps if you have access to more than one. A different model is a more independent critic than a fresh session of the same one — it doesn't share the same blind spots.

---

## The critic prompt

**Tell AI (in a fresh chat):**
```
You are reviewing work adversarially. Your job is to find what's wrong with it,
not to be encouraging and not to rewrite it.

REQUIREMENTS this work is supposed to satisfy:
[paste the spec, the acceptance criteria, or a plain-English description]

THE WORK:
[paste the code, schema, page, or plan]

PRIOR ROUNDS (if any):
[one-line summary per round: issue counts by severity, how they were resolved]

Review it for: correctness, unhandled errors, unvalidated input, edge cases
(empty, huge, concurrent, malicious), security and permissions, mismatches
with the requirements, and anything the requirements left ambiguous that the
work silently guessed at.

Output ONLY this format:

Issues:
1. category: [correctness | security | edge case | spec gap | data | performance]
   description: [what's wrong, where, and what breaks as a result]
   severity: [critical | major | minor]

Verdict: [approved | needs work]

Rules:
- Do not rewrite the work. Report issues only.
- Do not invent issues to seem thorough. If it's solid, say "approved".
- Severity: critical = data/money/security loss or main flow broken.
  major = wrong behavior a real user will hit. minor = style or unlikely edge case.
- Flag assumptions the work made that nobody stated.
```

**Then, back in the original session — Tell AI:**
```
An independent reviewer found these issues in what you just built:

[paste the issue list]

Fix every critical and major issue. For each one, tell me in one line what you
changed. If you think an issue is wrong, say so and explain why instead of
changing the code — don't fix something that isn't broken.

Don't refactor anything unrelated.
```

For per-domain critic prompts — specs, database schema, security, pricing pages, launch plans — see [ADVERSARY-PROMPTS.md](ADVERSARY-PROMPTS.md).

---

## What the loop catches — and what it can't

**Catches well:** missing error handling, unvalidated input, race conditions, permission holes, spec ambiguity, unstated assumptions, the "works for the happy path only" class of bug. This is the stuff that survives a single AI pass and shows up in production.

**Can't catch:** anything that requires running the thing. The critic reads and reasons — it doesn't execute code, hit your API, query your database, or click through your app. A loop that converges on code that was never run is a loop that agreed with itself in a vacuum.

**Gets wrong sometimes:** the critic raises false positives, misses real problems, and sometimes fixates on style while correctness sits right there. Treat its output as a lead list, not a verdict — which is exactly why the creator is told to push back on issues it thinks are wrong.

**The honest limit:** an adversarial loop improves quality. It does not prove quality. Convergence is a heuristic, not a proof. The output still needs your eyes on it and real tests against it before it ships. Use **/test** to actually exercise it and **/go-live** as the final gate.

---

## Common mistakes

| Mistake | Fix |
|---------|-----|
| Asking the same chat to review itself | New chat, every round. Anchored context rubber-stamps. |
| Pasting the creator's reasoning to the critic | Give it the artifact and the requirements, nothing else. |
| Letting the critic rewrite the work | Issues only. A critic that edits is just a second creator. |
| Accepting "looks good, though you could improve X" | Require category + severity + verdict, or the feedback isn't actionable. |
| Looping forever chasing "approved" | Cap at 3 rounds. All-minor is a finish line. |
| Discarding work that didn't converge | Ship the best version with the open issues written down. |
| Reviewing the whole app at once | One item per loop, or you get generic advice. |
| Looping everything | Spend it on money, auth, data, and public claims. Skip it on styling. |
| Fixing minor issues while criticals are open | Fix by severity, top down. |
| Treating "approved" as tested | The critic never ran the code. Test it anyway (**/test**). |

---

## Related skills

- **/test** — actually execute the thing. The loop reasons; tests verify.
- **/debug** — when something is already broken, diagnose before you loop.
- **/secure** — the security checklist the critic should be reviewing against.
- **/design-review** — the same adversarial idea, specialized for visual and UX work.
- **/go-live** — final human gate before shipping. The loop feeds it; it doesn't replace it.
- **/plan** — run a loop on the spec before anyone builds on it. Cheapest round you'll ever run.
