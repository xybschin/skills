---
name: implement
description: "Implement a piece of work based on a spec or set of tickets. Takes the next unblocked ticket from the tracker unless you name one."
disable-model-invocation: true
---

Implement one ticket, end to end.

A ticket is **optional** — without one, you take the next from the frontier, not the user. The issue tracker should have been provided to you — run `/setup-skills` if not.

## 1. Choose the ticket

If the user named a ticket (or passed a spec to work from), use it. Otherwise query the tracker's **frontier** — the open, unblocked, unclaimed work — as the tracker doc's "Frontier, claim, resolve" section describes, and take the first in order. Prefer tickets carrying the `ready-for-agent` triage label.

If the frontier is empty, say so and stop. Either everything is done, or what remains is blocked — report which, rather than inventing work.

**Claim it before anything else**, as the tracker doc describes. Other sessions may be working the same frontier in parallel, and the claim is what stops two agents taking the same ticket.

## 2. Implement it

Work only what the ticket describes. Use /tdd where possible, at pre-agreed seams.

Run typechecking regularly, single test files regularly, and the full test suite once at the end.

Work you discover that isn't part of this ticket becomes a **new** ticket, linked to this one as discovered-from-this-work where the tracker can express it. Don't widen the scope of the ticket you claimed.

Once done, use /code-review to review the work.

Commit your work using /commit.

## 3. Close it

Close the ticket with a reason naming what landed, as the tracker doc describes.

**Stop there — one ticket per session.** Closing it may unblock others; they are the next session's work, with fresh context. Tell the user what the frontier looks like now so they can decide whether to continue.
