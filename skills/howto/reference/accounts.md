# Capture accounts

One account per entitlement tier. `/howto` settles which **tier** a document is
written at (§1 of `SKILL.md`) and resolves the login from this table. **The user
is never asked to pick an account by name** — they are asked for the tier, and
usually not even that, because the `/validation` matrix the how-to is built from
already records it.

| Tier | Plan name | Account | Org |
|---|---|---|---|
| 1 | Foundation | `<fill in>` | `<fill in>` |
| 2 | Control | `<fill in>` | `<fill in>` |
| 3 | Vigilance | `<fill in>` | `<fill in>` |

## No credentials in this file

It ships with a published plugin. Record only the identifier needed to pick the
right session — the passwords live wherever the rest of the test logins are
kept, and never here.

## Why the tier and not the account

The tier is what the document states in *Before you start* and what a reader
checks their own plan against. The account is only how you reach a screen to
photograph, and accounts are recreated far more often than tiers are
renumbered — so a rotated login is a one-line edit here and no change to
`SKILL.md`. Add a row rather than repurposing one if a fourth tier appears.

## The tier names

**Tier 1 = Foundation, Tier 2 = Control, Tier 3 = Vigilance.** Write the plan
name in the document, not the number — a reader recognises *Control* on their
invoice and has never seen "Tier 2".

## If a tier has no working account

Say so and hold the document. Do not capture on a neighbouring tier: the
screenshots are the proof of the tier the document claims, so a Foundation
how-to shot on a Vigilance login shows menu entries the reader's plan will never
render — and the tester who cannot find them files a defect that does not exist.
