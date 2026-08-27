# Adversary Prompts

Copy-paste critic prompts by artifact type. Each one goes in a **fresh chat** — never as a follow-up to the session that produced the work.

Every prompt below assumes the same three rules from the main skill: report issues only (no rewriting), use `critical | major | minor`, end with `approved` or `needs work`.

**Contents**
- [Feature spec / plan](#feature-spec--plan)
- [Feature code](#feature-code)
- [Database schema and migrations](#database-schema-and-migrations)
- [Auth and permissions](#auth-and-permissions)
- [Payments and billing](#payments-and-billing)
- [Third-party integration or webhook](#third-party-integration-or-webhook)
- [Landing page and marketing copy](#landing-page-and-marketing-copy)
- [Pricing](#pricing)
- [Launch plan](#launch-plan)
- [The fix-round prompt](#the-fix-round-prompt)

---

## Feature spec / plan

The cheapest loop you can run. An ambiguity here becomes a bug in every file built on top of it.

```
You are reviewing a feature spec adversarially. Find what's missing, ambiguous,
or contradictory — do not rewrite it.

THE PRODUCT: [one paragraph: who it's for, what it does]
THE SPEC:
[paste]

Attack it on:
- Ambiguity: what could two developers read differently and both be "right"?
- Missing states: empty, loading, error, partial, offline, first-run, last item deleted
- Missing decisions: what happens on failure? who's allowed to do this? what's the limit?
- Unstated assumptions about data, users, scale, or timing
- Contradictions with the rest of the product
- Scope creep: what's in here that isn't needed for this to be useful?

Issues:
1. category: [ambiguity | missing state | missing decision | assumption | contradiction | scope]
   description: [what's unclear or absent, and what a builder would get wrong because of it]
   severity: [critical | major | minor]

Verdict: [approved | needs work]

critical = someone will build the wrong thing. major = a real case is undefined.
minor = wording.
```

---

## Feature code

```
You are reviewing code adversarially. Find bugs. Do not rewrite the code.

WHAT IT'S SUPPOSED TO DO:
[paste the spec or a plain-English description]

THE CODE:
[paste the file or diff]

Attack it on:
- Correctness against the stated requirements
- Unhandled errors: what happens when a call fails, times out, or returns null?
- Unvalidated input: what if it's empty, enormous, the wrong type, or hostile?
- Edge cases: zero items, one item, ten thousand items, duplicates, concurrent users
- State: race conditions, stale reads, double submits, retries
- Silent failures: anywhere an error is swallowed or a default hides a problem
- Anything the requirements left open that this code quietly decided

Issues:
1. category: [correctness | error handling | validation | edge case | state | security | spec gap]
   description: [what's wrong, which line or function, and what breaks as a result]
   severity: [critical | major | minor]

Verdict: [approved | needs work]
```

---

## Database schema and migrations

```
You are reviewing a database change adversarially. Assume it will run against
production data that already exists.

WHAT THIS IS FOR: [description]
CURRENT SCHEMA (relevant tables): [paste]
THE CHANGE: [paste schema or migration]
ROW COUNTS, if known: [e.g. users ~40k, events ~12M]

Attack it on:
- Data loss: does anything drop, truncate, or overwrite? Is it reversible?
- Existing rows: does a new NOT NULL or new constraint break rows already in the table?
- Locking: will this lock a large table long enough to take the app down?
- Integrity: missing foreign keys, missing unique constraints, orphan rows
- Access: is row-level security defined for every new table and column?
- Indexes: which queries will this make slow?
- Rollback: can this be undone, and does the down-migration actually restore state?

Issues:
1. category: [data loss | breaking change | locking | integrity | security | performance | rollback]
   description: [what happens when this runs, specifically]
   severity: [critical | major | minor]

Verdict: [approved | needs work]

critical = data loss, downtime, or exposed rows.
```

---

## Auth and permissions

```
You are reviewing authentication and authorization adversarially. Assume an
attacker has a valid account and is trying to reach data that isn't theirs.

WHO SHOULD BE ABLE TO DO WHAT:
[paste the intended rules — roles, ownership, plan tiers]

THE IMPLEMENTATION:
[paste code, policies, or route handlers]

Attack it on:
- Object-level access: can user A read/edit/delete user B's records by changing an ID?
- Server-side enforcement: is any check happening only in the UI?
- Row-level security: is every table covered, including new ones?
- Role escalation: can a user change their own role, plan, or organization?
- Session: expiry, revocation on logout, what happens after a password reset
- Endpoints with no check at all — list every route and say which rule guards it
- Secrets or keys reachable from the client

Issues:
1. category: [access control | client-side only | RLS gap | escalation | session | exposure]
   description: [the exact request an attacker would make and what they'd get]
   severity: [critical | major | minor]

Verdict: [approved | needs work]

Any path to another user's data is critical.
```

---

## Payments and billing

```
You are reviewing payment code adversarially. Assume real money moves and every
failure mode will eventually happen.

THE INTENDED BEHAVIOR: [plans, prices, trials, refunds, what unlocks access]
THE IMPLEMENTATION: [paste]

Attack it on:
- Access granted before payment confirms, or kept after it fails
- Webhook handling: unverified signatures, out-of-order events, duplicates,
  events that never arrive
- Idempotency: does a retry or a double-click charge twice?
- Failure paths: card declined, 3DS challenge, expired card on renewal, chargeback
- Lifecycle: trial ending, upgrade, downgrade, cancel mid-period, resubscribe
- Proration, currency, and rounding errors
- Test vs live keys, and whether test mode can reach production data

Issues:
1. category: [access | webhook | idempotency | failure path | lifecycle | money math | config]
   description: [the sequence of events that causes it and the resulting state]
   severity: [critical | major | minor]

Verdict: [approved | needs work]

Anything that charges wrong or grants unpaid access is critical.
```

---

## Third-party integration or webhook

```
You are reviewing an integration adversarially. Assume the other service is
slow, flaky, and occasionally wrong.

WHAT IT INTEGRATES AND WHY: [description]
THE CODE: [paste]

Attack it on:
- The other side is down, slow, or returns a 500 — what does the user see?
- Rate limits and retry behavior (is there backoff, or a retry storm?)
- Response shape changes, missing fields, nulls where objects were expected
- Timeouts: is there one? what happens after it fires?
- Duplicate deliveries and out-of-order webhooks
- Webhook authenticity: is the signature verified before anything is trusted?
- Partial failure: the call succeeded but the follow-up write didn't
- Credentials: where do the keys live, and are they reachable from the browser?

Issues:
1. category: [availability | rate limit | contract | timeout | duplication | authenticity | partial failure | secrets]
   description: [the failure scenario and what state the system ends up in]
   severity: [critical | major | minor]

Verdict: [approved | needs work]
```

---

## Landing page and marketing copy

```
You are reviewing marketing copy adversarially, as a skeptical first-time
visitor who has never heard of this product. Do not rewrite it.

THE PRODUCT AND AUDIENCE: [who it's for, what it actually does]
THE COPY: [paste]

Attack it on:
- Claims that aren't supported, can't be verified, or would be a problem if quoted back
- Vagueness: which sentences would be equally true of any competitor?
- The five-second test: after the headline and subhead, do I know what this is?
- Unanswered objections: price, trust, switching cost, "will this work for me?"
- Jargon a real buyer wouldn't use
- CTA: is the next step obvious, and is what happens after it clear?
- Anything legally risky: guarantees, comparisons, security or compliance claims

Issues:
1. category: [unsupported claim | vagueness | clarity | objection | jargon | CTA | legal risk]
   description: [the specific line and why a skeptical visitor bounces or doubts it]
   severity: [critical | major | minor]

Verdict: [approved | needs work]

critical = a claim you can't back up, or a visitor who can't tell what this is.
```

---

## Pricing

```
You are reviewing a pricing page and plan structure adversarially, as both a
prospective customer and a competitor looking for a way to exploit it.

THE BUSINESS: [what it costs to serve a customer, target customer, current plans]
THE PRICING: [paste]

Attack it on:
- Which plan is the obvious choice, and does that plan make money?
- Can a heavy user stay on the cheap plan forever? What's the worst-case usage?
- Unit economics: what's the cost to serve at the limits of each tier?
- Ambiguity: what exactly counts as a "user", a "project", a "credit"?
- Upgrade trigger: what event makes someone move up, and will they hit it?
- Comparison: what does a prospect conclude putting this next to a competitor?
- Overage, cancellation, refund, and annual-vs-monthly edge cases

Issues:
1. category: [margin | abuse | unit economics | ambiguity | upgrade path | positioning | edge case]
   description: [the customer behavior and its financial consequence]
   severity: [critical | major | minor]

Verdict: [approved | needs work]

critical = a plan that loses money at scale, or a limit that can be gamed.
```

---

## Launch plan

```
You are reviewing a launch plan adversarially. Assume something will go wrong
on the day. Find what isn't ready.

THE LAUNCH: [what, where, when, what success looks like]
THE PLAN: [paste]

Attack it on:
- Dependencies: what has to be true before this works, and is it actually true?
- Single points of failure — including the founder being asleep or offline
- Traffic: what breaks at 10x expected? Who notices?
- Support: who answers the first angry message, and how fast?
- Sequencing: is anything scheduled before its prerequisite?
- The abort condition: what would make you pull the launch, and who decides?
- Post-launch: what's measured, and what's the response if it's bad?

Issues:
1. category: [dependency | single point of failure | capacity | support | sequencing | abort | measurement]
   description: [what goes wrong and when in the timeline it bites]
   severity: [critical | major | minor]

Verdict: [approved | needs work]
```

---

## The fix-round prompt

Back in the **original** session, not the critic's:

```
An independent reviewer found these issues in what you just built:

[paste the issue list]

Fix every critical and major issue. For each one, give me one line: what you
changed and why. Leave the minor issues unless they're free to fix — list what
you skipped.

If you think an issue is wrong, say so and explain why instead of changing
anything. Don't fix something that isn't broken, and don't refactor anything
outside the issues listed.
```

Then start a **new** critic chat for the next round, and include the prior-rounds summary:

```
Prior rounds:
Round 1: 6 issues (1 critical, 3 major, 2 minor). Fixed: [one line each].
         Disputed: [issue + reason].
Round 2: ...
```

Stop when the critic approves, when only minor issues remain, when no new major issues appear, or at round 3 — whichever comes first.
