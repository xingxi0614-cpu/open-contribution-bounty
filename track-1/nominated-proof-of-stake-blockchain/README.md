# Nominated Proof-of-Stake Substrate Blockchain Course

This course module explains how to design a local Substrate-based blockchain
that uses Nominated Proof of Stake, or NPoS, for validator selection and stake
weighted security. It is written for learners who already know the basic shape
of a Polkadot SDK runtime and now need to understand how staking, sessions,
validator elections, rewards, and slashing fit together.

The module is intentionally implementation-oriented. It does not claim that a
toy chain has the same economics as Polkadot, but it shows which runtime pieces
must exist before a learner can build a credible NPoS demonstration chain.

## Learning Goals

After finishing this module, a learner should be able to:

- Explain the difference between plain Proof of Stake and NPoS.
- Describe the roles of validator candidate, nominator, inactive staker, and
  session authority.
- Add the staking-related FRAME pallets that a solo chain needs.
- Configure sessions and eras so validator elections happen predictably.
- Seed initial authorities, validators, nominators, balances, and staking
  intentions in genesis.
- Run the staking lifecycle from `bond` to `validate`, `nominate`, `chill`,
  payout, and unbond.
- Explain why election limits, exposure pages, commission, rewards, and
  slashing are part of the security model.
- Write tests and manual checks that prove only elected validators produce
  blocks for the next era.

## Mental Model

NPoS lets token holders back validator candidates without running validator
infrastructure themselves. Validators produce blocks and participate in
finality. Nominators choose trustworthy validator candidates and place stake
behind them. The staking election chooses an active validator set from those
signals.

```text
validator candidate       nominator
        |                    |
        | validate()         | nominate([validators])
        +----------+---------+
                   |
                   v
            staking election
                   |
                   v
       active validator set for next era
                   |
                   v
       sessions, rewards, offences, slashing
```

The key idea is that the validator set is not a static admin list. It is a
runtime result that changes when stake, nominations, chilling, slashing, and
election constraints change.

## Runtime Shape

A practical NPoS solo-chain runtime normally combines these responsibilities:

- `pallet_balances` holds the staking asset.
- `pallet_staking` tracks bonded stake, roles, eras, rewards, and slashes.
- `pallet_session` rotates session keys and applies the elected authority set.
- A block production pallet such as BABE or Aura creates blocks.
- A finality pallet such as GRANDPA finalizes blocks.
- `pallet_authorship` attributes produced blocks to validators for rewards.
- `pallet_offences` records validator misbehavior and routes reports.
- An election provider selects active validators from candidates and nominators.
- A voter list, such as bags list or a bounded map approach, keeps elections
  within runtime weight limits.

The exact crate names and version pins should match the Polkadot SDK release
used by the template. The design is stable even when the dependency paths move.

## Minimal Implementation Path

Start from a solo-chain template rather than a parachain template. A parachain
uses collators and relay-chain security, while this course is about a chain
whose own staking system chooses validator authorities.

### 1. Add Staking Dependencies

Add runtime dependencies for staking, session management, authorship, offences,
and the election provider. Keep the versions aligned with the rest of the SDK:

```toml
pallet-staking = { workspace = true }
pallet-session = { workspace = true }
pallet-authorship = { workspace = true }
pallet-offences = { workspace = true }
pallet-election-provider-multi-phase = { workspace = true }
```

If the template uses explicit versions instead of workspace dependencies, use
the same Polkadot SDK release for every pallet. Mixing SDK releases is one of
the fastest ways to get trait and type errors.

### 2. Define Staking Types

Choose small constants for a local course chain. They must be realistic enough
to show the mechanics, but short enough that a learner can watch era changes
without waiting for hours:

```rust
pub const SessionsPerEra: sp_staking::SessionIndex = 3;
pub const BondingDuration: sp_staking::EraIndex = 2;
pub const MaxNominations: u32 = 16;
pub const MaxValidators: u32 = 32;
```

For a production chain, these values should be changed only after economic
analysis, benchmarking, and governance review. For a course chain, the point is
to make validator rotation and payouts visible in a short demo.

### 3. Configure `pallet_staking`

The staking config ties together balances, session rotation, reward payout,
slash handling, election output, and administrative origins. The important
review question is not "does it compile?" but "who can change each security
parameter, and what happens when a validator misbehaves?"

At minimum, review these areas:

- The currency used for bonded stake.
- The era payout implementation.
- The slash destination and slash cancellation origin.
- The session manager connection.
- The maximum validator count and nominator limits.
- The election provider and voter list strategy.
- The reward destination behavior.
- The weight info generated by benchmarks or a template fallback.

### 4. Connect Sessions and Keys

Staking chooses validator accounts, but the consensus engine needs session
keys. The runtime must map the elected validator account to the session keys
used by block production and finality.

For a BABE plus GRANDPA style demo, the chain spec should give each initial
authority:

- a stash account with bonded stake;
- a validator role declaration;
- session keys for block production;
- GRANDPA authority keys;
- enough free balance for fees and existential deposits.

The lesson should show that a candidate is not active immediately. A learner
should validate, wait for the election boundary, and then observe the account
joining the active validator set.

### 5. Seed Genesis Stakers

The genesis configuration should include at least:

- two or more initial validator candidates;
- one or more nominators with nominations spread across candidates;
- a validator count smaller than the candidate count;
- balances high enough to cover the bonded amount;
- session keys for the initial active authorities.

This setup makes the election visible. If every candidate is always selected,
the learner cannot observe the effect of nominations or validator limits.

Example scenario:

```text
validator count: 2
candidate A: own stake 1000
candidate B: own stake 1000
candidate C: own stake 1000
nominator 1: nominates A and C with 500
nominator 2: nominates B and C with 800
```

At the next era, the election provider should select an active set based on the
configured election logic and constraints.

## Learner Walkthrough

### Bond Funds

Bonding locks tokens so they can be used for staking. The bonded account is the
economic identity. Modern staking flows increasingly move away from separate
controller accounts, so the course should call out which model the chosen SDK
template exposes.

Checks:

- The account balance decreases or becomes locked according to the pallet.
- The account appears in staking storage as a bonded staker.
- The bonded amount cannot be transferred freely.

### Become a Validator Candidate

Call `validate` from a bonded account. This declares intent to be a validator,
but it does not instantly place the account in the active validator set.

Checks:

- The account has validator preferences, including commission.
- The account is eligible for the next election.
- Session keys are registered before the account can safely validate.

### Nominate Validators

Call `nominate` from a bonded account and choose validator candidates. The
nominator shares rewards and slash risk with the chosen validators.

Checks:

- Nominations are bounded by `MaxNominations`.
- Invalid or duplicate targets are rejected.
- The nominator is exposed to slash risk for elected validators it backs.

### Advance Sessions and Eras

Produce blocks until the chain crosses the configured session and era
boundaries. The active validator set should only change at the correct
boundary, not in the middle of a session.

Checks:

- Session index increments on schedule.
- Era index increments after the configured number of sessions.
- The new validator set appears after the election is applied.

### Pay Rewards

After an era has ended, call `payout_stakers` for a validator and era. Rewards
are split according to validator points, commission, and exposure.

Checks:

- Reward destinations match the selected payee.
- Validator commission is taken before the remainder is shared.
- Large nominator sets are handled through exposure pages when configured.

### Chill and Unbond

Call `chill` to stop seeking validator or nominator status. Then call `unbond`
and wait through the bonding duration before withdrawal.

Checks:

- A chilled validator is not elected in the next era.
- Unbonded funds remain locked until the bonding duration passes.
- `withdraw_unbonded` releases eligible unlocking chunks only.

## Tests to Include

A useful course implementation should include unit or integration tests for:

- initial genesis validators and nominators;
- candidate declaration through `validate`;
- nomination limits and invalid nomination targets;
- era transition and validator set update;
- reward payout to validator and nominators;
- chilling before the next election;
- unbonding and withdrawal after the bonding duration;
- slash reporting or a mocked offence path;
- non-staker accounts failing staking-only flows;
- session keys required before a validator is usable.

Tests should avoid asserting exact production economics unless the runtime uses
the same constants and payout curve as a live network. Instead, they should
prove the state transitions that the course teaches.

## Common Failure Modes

- Treating validator candidates as active validators immediately.
- Forgetting session keys in the chain spec.
- Giving every candidate a validator slot, which hides the election.
- Setting session or era lengths so long that the demo cannot be observed.
- Using balances that are too small for bonds, fees, or deposits.
- Ignoring validator commission in payout expectations.
- Forgetting that nominators share slash risk with validators they back.
- Copying Polkadot production constants into a tiny local test chain.
- Adding staking without benchmarking weights before production use.

## Security Notes

NPoS is an economic security mechanism, not only a runtime wiring exercise.
Before a chain uses this in production, the team should review:

- how many validators are required for liveness and decentralization;
- whether the staking token has enough distribution to support security;
- how slash reports are generated, verified, and appealed;
- whether validator commission creates centralization incentives;
- how emergency origins can pause staking or force eras;
- how runtime upgrades are governed;
- whether off-chain election work is reliable and reproducible;
- how telemetry and monitoring detect offline validators.

For the course, those questions should be visible even if the implementation
uses shortened eras and mocked offence paths.

## Suggested Directory Layout

```text
track-1/nominated-proof-of-stake-blockchain/
  README.md
  implementation-checklist.md
  review-checklist.md
```

The README is the learner guide. The implementation checklist is for builders.
The review checklist is for maintainers who need to verify that the submission
actually teaches a working NPoS runtime design.

## Maintainer Acceptance Evidence

A complete learner submission should provide evidence that the runtime behavior
matches the lesson. The companion `demo-runbook.md` file lists the checks that
should be captured before marking the lesson complete:

- the initial validator and nominator accounts in genesis;
- the active validator set before and after an era change;
- the staking ledger for a validator and a nominator;
- the session key registration path;
- a successful `validate` call;
- a successful `nominate` call;
- a payout call after the era closes;
- a chill and unbond flow;
- at least one rejected staking action from an invalid account.

This makes the bounty easier to review because the maintainer can compare the
course text against observable chain state.

## References

- [Polkadot staking guide][staking-guide]
- [Polkadot proof-of-stake consensus][pos-consensus]
- [FRAME staking pallet docs][staking-pallet]
- [Add an existing pallet to a runtime][add-pallet]

[staking-guide]:
  https://wiki.polkadot.com/learn/learn-staking/
[pos-consensus]:
  https://docs.polkadot.com/reference/polkadot-hub/consensus-and-security/pos-consensus/
[staking-pallet]:
  https://docs.rs/pallet-staking/latest/pallet_staking/
[add-pallet]:
  https://docs.polkadot.com/parachains/customize-runtime/add-existing-pallets/
