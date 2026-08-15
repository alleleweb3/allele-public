# allele-public

# Allele — Supply Commitment

A creature breeding game on Solana with real recessive genetics.

This repository contains the **supply program** and the **supply
commitment**: the code and the derivation behind Allele's maximum
population, and the steps to verify it yourself.

---

## The claim

**136,250 creatures will ever exist.**

Not a policy. Not a promise. A cap enforced by an on-chain program whose
upgrade authority has been burned — meaning no one, including us, can raise
it.

## The derivation

Genesis supply is pre-committed at **20,000** creatures, fixed across all
editions. Every creature has a finite lifetime breed allowance that declines
by generation:

| Generation | Breeds allowed |
|---|---|
| 0 | 3 |
| 1 | 2 |
| 2 | 2 |
| 3 | 1 |
| 4 | 1 |
| 5 | 1 |
| 6+ | 0 |

Each breed consumes one allowance from **both** parents, so each generation
is bounded by:
n[g+1] = n[g] × fertility(g) ÷ 2

Because fertility falls below 2, the series converges. Summed across all
generations: **136,250 creatures, maximum, forever.**

Full derivation: [`supply-commitment.md`](./supply-commitment.md)

## Verify it yourself

Don't take our word for any of the above.

**1. Confirm the upgrade authority is burned**

```bash
solana program show <PROGRAM_ID> --url devnet
```

Look for `Authority: none`. A program with no upgrade authority cannot be
redeployed or modified by anyone.

**2. Confirm the deployed bytecode matches this source**

Build from the pinned toolchain (see [`BUILD.md`](./BUILD.md)) and compare
the resulting hash against the deployed program. Instructions and the
expected `sha256` are in the build doc.

**3. Confirm the cap in the source**

`GENESIS_CAP` is a compile-time constant. There is no instruction, admin
path, or editable account that can change it. Read the program — it's ~200
lines.

---

## Status: devnet

The supply program is currently **deployed and burned on devnet**. This is
the rehearsal, not the launch.

- Devnet program ID: `<PROGRAM_ID>`
- Audited at commit: `838462a`
- Mainnet deployment and burn: **not yet performed**

The mainnet deploy-and-burn happens once, immediately before genesis, and
will be announced with its own program ID and verification steps. Until
then, every claim on this page refers to devnet.

## What this repo does *not* contain

The genome engine, breeding algorithm, combat model, and trait pipeline are
not published yet. They'll open after launch.

What's here is what's needed to verify the supply claim — the program that
enforces the cap and the derivation behind the number. That's the claim
we're making publicly, so that's the code we're publishing.

## What is *not* guaranteed

A commitment that overstates itself is worth less than one that states its
limits.

**Frozen and unchangeable:** the genesis cap, the per-generation population
caps, and therefore the 136,250 lifetime ceiling.

**Not frozen:** everything else. Breeding mechanics, combat, marketplace,
fees, and rendering live in a separate upgradeable program and will change
as the game develops.

We are not claiming the game is immutable. We're claiming the supply is, and
that specific claim is the one you can check above.

---

## Links

- Supply commitment: [`supply-commitment.md`](./supply-commitment.md)
- Build & verification: [`BUILD.md`](./BUILD.md)
- Site: [allele.to](https://allele.to)

## License

[MIT](https://github.com/stripe/ai/blob/main/LICENSE)
