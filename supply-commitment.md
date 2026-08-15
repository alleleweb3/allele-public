# Allele — Supply Commitment

**Published before the genesis mint. This document is a commitment, not a
description, and every claim in it is checkable by you without trusting us.**

---

## The commitment, in one paragraph

There will only ever be **20,000 generation-0 creatures** in Allele. They are
released in two editions of 10,000. Creatures can breed, and breeding is limited
by a published fertility schedule, which means the total number of Allele
creatures that can ever exist is capped at **136,250**. The cap is enforced by a
Solana program whose upgrade authority has been **permanently destroyed**, so no
one — including us — can raise it. Section 5 tells you how to verify that
yourself in about thirty seconds.

---

## 1. Why this document exists

Most NFT projects publish a supply number. Almost none publish one they cannot
change, because keeping the ability to mint more is worth money and giving it up
is worth nothing unless you can prove you have given it up.

The proof is the point. A supply cap you have to take on trust is a marketing
claim. A supply cap enforced by a program that cannot be modified is a fact about
the blockchain, and you can check it whether or not you believe anything else we
say.

CryptoKitties, the closest antecedent to this game, minted 50,000 generation-0
cats over the course of a year, at a rate the operator controlled. We have made
that impossible for ourselves before selling anything.

---

## 2. Generation-0 supply: 20,000, fixed

| Edition | Supply | When |
|---|---|---|
| Edition 1 — First Edition | 10,000 | Genesis |
| Edition 2 | 10,000 | Per the release rule below |
| **Total, permanently** | **20,000** | |

Edition 2 exists so that people who arrive later can still acquire a
generation-0 creature. It is **not additional supply** — both editions are carved
out of the same pre-committed 20,000. Edition 2 opens only when Edition 1 is at
least 95% sold **and** at least 90 days have passed since Edition 1 opened. Both
conditions are checkable from public chain data. We cannot bring it forward
because we would like more revenue this quarter.

Edition 2 introduces **no new trait variants**. It differs from Edition 1 only in
timing and in its on-chain edition field. This matters because new trait
variants would dilute the rarity of every existing creature.

---

## 3. Breeding: the fertility schedule

Two creatures can breed to produce a new one. Every creature has a **limited
number of lifetime breeds**, and that limit falls with each generation:

| Generation | Lifetime breeds |
|---|---|
| 0 (from an egg) | 3 |
| 1 | 2 |
| 2 | 2 |
| 3 | 1 |
| 4 | 1 |
| 5 | 1 |
| 6 and beyond | **0 — infertile** |

Fertility is consumed permanently. A generation-0 creature that has bred three
times can never breed again, and this is enforced on-chain, not by our servers.

---

## 4. The ceiling: 136,250 creatures, and how it is derived

Each breed consumes one fertility point from **each** of two parents and produces
one child. So:

> total births = total fertility consumed ÷ 2

To get the largest possible population you would always pair creatures of the
same, lowest available generation — because fertility shrinks as generation
rises, so pushing children to higher generations destroys future capacity. That
gives the recurrence:

```
n[0]   = 20,000                      (generation-0, pre-committed)
n[g+1] = n[g] × fertility(g) ÷ 2
```

Worked through:

| Generation | Max creatures | Fertility | Children it can produce |
|---|---|---|---|
| 0 | 20,000 | 3 | 20,000 × 3 ÷ 2 = 30,000 |
| 1 | 30,000 | 2 | 30,000 × 2 ÷ 2 = 30,000 |
| 2 | 30,000 | 2 | 30,000 × 2 ÷ 2 = 30,000 |
| 3 | 30,000 | 1 | 30,000 × 1 ÷ 2 = 15,000 |
| 4 | 15,000 | 1 | 15,000 × 1 ÷ 2 = 7,500 |
| 5 | 7,500 | 1 | 7,500 × 1 ÷ 2 = 3,750 |
| 6 | 3,750 | 0 | 0 |

```
20,000 + 30,000 + 30,000 + 30,000 + 15,000 + 7,500 + 3,750 = 136,250
```

**136,250 is an upper bound, and the real number will be lower.** Two reasons,
both working in your favour:

- Real players do not breed in perfect same-generation lockstep. Pairing a
  generation-0 with a generation-5 produces a generation-6 child and wastes the
  generation-0's scarce fertility.
- Allele blocks pairings that share an ancestor within three generations, so
  some theoretically available breeds cannot happen at all.

We publish the upper bound rather than an estimate because it is the number we
can promise never to exceed.

The reference implementation is `allele/crates/allele-genome/src/fertility.rs`,
and a test asserts the published 136,250 recomputes from the schedule so the two
can never drift apart. `sim/supply.py` computes the same number independently.

---

## 5. How to verify the cap yourself

The generation-0 cap lives in a small, single-purpose Solana program that does
nothing except count mints and refuse the 20,001st. Its upgrade authority has
been destroyed.

### 5.1 Check that nobody can change the program

```
solana program show <ALLELE_SUPPLY_PROGRAM_ID>
```

Look at the `Authority` line. It must read:

```
Authority: none
```

If it names any address, that address can replace the program and the cap means
nothing. `none` means the bytecode currently deployed is the bytecode that will
be there forever. There is no governance process, no multisig, and no timelock
that can override this — the capability has been destroyed, not delegated.

### 5.2 Check that the deployed program is the source we published

```
solana-verify verify-from-repo \
    --program-id <ALLELE_SUPPLY_PROGRAM_ID> \
    https://github.com/<org>/allele
```

This rebuilds our published source and confirms the resulting bytecode matches
what is deployed. Step 5.1 proves the program cannot change; this step proves the
unchangeable program is the one whose source you can read.

### 5.3 Check the constant and the live count

The cap is a compiled-in constant, not a configuration value:

```rust
// programs/supply/src/lib.rs
pub const GENESIS_CAP: u64 = 20_000;   // sourced from allele-genome
```

Configuration values can be rewritten by whoever holds the config authority.
A constant can only be changed by deploying different code, which 5.1 rules out.

To read how many have actually been minted:

```
solana account <SUPPLY_STATE_PDA>     # PDA seed: "supply"
```

The `genesis_minted` field only ever increases, and the program rejects any mint
that would take it past 20,000.

*(The program ID, registry address, and repository URL are published at
allele.to/verify and are fixed at deployment.)*

---

## 6. What we can still change, stated plainly

The supply cap is not the only thing that matters, so here is an honest boundary.

**Cannot be changed by anyone, ever:**

- The 20,000 generation-0 cap
- The per-generation population caps that produce the 136,250 ceiling
- Any creature's genome, once minted

On that second point, precisely: the supply program does not track individual
creatures' breeding. It enforces a **cap on the population of each generation**,
derived from the fertility schedule by `n[g+1] = n[g] x fertility(g) / 2`:

| Generation | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---|---|---|---|---|---|---|
| Maximum ever | 20,000 | 30,000 | 30,000 | 30,000 | 15,000 | 7,500 | 3,750 |

Those bound the total identically to per-creature fertility, using only global
counters — which is what lets the whole guarantee live inside a program small
enough to audit exhaustively and then freeze. The per-creature breeding limits
that shape gameplay live in the upgradeable game program; the aggregate ceiling
this document commits to does not.

**Can still be changed by us:**

- Game logic: breeding fees, cooldown timing, combat, marketplace behaviour,
  and per-creature breeding limits. These live in a separate, upgradeable
  program so that bugs remain fixable. Changing them cannot raise any number in
  this document, because the caps above are enforced independently.
- Trait artwork for *new* variants. Published art is immutable — a released
  asset is never edited, only superseded by a new variant.
- The website, the API, and anything off-chain.

We have deliberately split the two. Making the whole game immutable would mean
never fixing a bug; making the supply cap upgradeable would mean the cap is a
promise rather than a fact. The split gives you a hard guarantee on the thing
that cannot be undone, and gives us the ability to repair the things that can.

---

## 7. Summary

| Claim | Value | How you check it |
|---|---|---|
| Generation-0 supply | 20,000 | §5.3, compiled constant |
| Editions | 2 × 10,000 | §2, on-chain edition field |
| Lifetime ceiling | 136,250 | §4 derivation, `sim/supply.py` |
| Cap cannot be raised | Authority destroyed | §5.1, `Authority: none` |
| Deployed code matches source | Verifiable build | §5.2 |
| Minted so far | Live | §5.3, registry account |

If any check in this document fails, the commitment is broken and you should say
so publicly.
