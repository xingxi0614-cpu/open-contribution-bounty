# Local Demo Runbook

This runbook turns the NPoS course into a concrete review exercise. It is not a
replacement for production benchmarking or security review. It is a small local
demo that proves the course teaches observable staking behavior.

## Demo Goal

By the end of the demo, a reviewer should see:

- bonded accounts with staking ledgers;
- validator candidates that are not active immediately;
- nominators backing validator candidates;
- an era boundary that applies a new active validator set;
- rewards that can be paid after an era ends;
- chilled or unbonding stakers leaving the election path;
- a rejected call that proves the runtime enforces staking rules.

## Suggested Local Accounts

Use deterministic development accounts so learners can repeat the same demo:

- Alice as an initial validator;
- Bob as an initial validator;
- Charlie as a validator candidate;
- Dave as a nominator;
- Eve as a nominator;
- Ferdie as a negative-test account with insufficient stake.

The chain spec should fund every account except the negative-test case with
enough free balance for fees, staking locks, and existential deposits.

## Suggested Runtime Constants

For a course chain, keep the cycle short:

```text
validator count: 2
candidate count: 3
sessions per era: 3
bonding duration: 2 eras
maximum nominations: 16
minimum validator bond: small but non-zero
minimum nominator bond: small but non-zero
```

These numbers are intentionally small. They make the lesson reviewable on a
laptop while preserving the important NPoS state transitions.

## Step 1: Start the Chain

Start two validator nodes with the development chain spec. The exact command
depends on the template, but the reviewer should record:

- the chain spec file used;
- the binary or node command used;
- the initial authority accounts;
- the first finalized block;
- the initial session index;
- the initial era index.

Acceptance evidence:

```text
session.currentIndex == 0 or 1
staking.activeEra exists
session.validators contains Alice and Bob
Charlie is not an active validator yet
```

## Step 2: Inspect Genesis Staking State

Query staking storage for each seeded account.

The reviewer should confirm:

- Alice and Bob are bonded;
- Alice and Bob have validator preferences;
- Dave and Eve are bonded as nominators if seeded that way;
- Charlie has funds and session keys but is not yet active;
- the validator count is lower than the candidate count.

This proves the course setup can demonstrate an actual election rather than a
static validator list.

## Step 3: Register Session Keys

Before Charlie can safely validate, register session keys for Charlie. If the
template exposes a command to rotate keys, record the generated key material
only locally. Do not publish private keys in course material or PR comments.

Acceptance evidence:

```text
session.nextKeys(Charlie) is set
Charlie is still not active until the election boundary
```

## Step 4: Bond and Validate

From Charlie, bond funds and declare validator intent.

Expected observations:

- Charlie has a staking ledger;
- Charlie has validator preferences;
- Charlie remains outside the active validator set until the era changes;
- a candidate with missing session keys is rejected or documented as unsafe.

Negative check:

```text
Ferdie validate() with insufficient bond fails
```

## Step 5: Nominate Candidates

From Dave and Eve, nominate overlapping validator candidates.

Example nominations:

```text
Dave nominates Alice and Charlie
Eve nominates Bob and Charlie
```

Expected observations:

- nominations are stored for Dave and Eve;
- duplicate targets are rejected;
- more than `MaxNominations` targets are rejected;
- nominators are exposed to reward and slash risk for elected targets.

## Step 6: Advance Sessions and Eras

Produce blocks until the next era is applied.

Record:

- the session index before the era change;
- the active era before the change;
- the active validator set before the change;
- the same values after the change.

Expected observations:

```text
session index increments first
era index increments after SessionsPerEra sessions
validator set changes only at the boundary
```

If Charlie has enough backing and the election selects Charlie, Charlie should
appear in the active validator set for the next era.

## Step 7: Pay Out Rewards

After an era is complete, call `payout_stakers` for one validator and the
completed era.

Expected observations:

- payout succeeds only for a completed era;
- validator commission is applied before nominator reward sharing;
- the selected reward destination receives funds or stake;
- repeated payout for the same validator and era is rejected or has no effect.

Record balances before and after the payout so the learner can see the change.

## Step 8: Chill and Unbond

Call `chill` for Charlie or one nominator. Then unbond part of the stake and
advance eras until withdrawal is allowed.

Expected observations:

- chilled stakers stop participating in later elections;
- unbonded chunks stay locked through the bonding duration;
- `withdraw_unbonded` releases only eligible chunks;
- the account keeps enough free balance for fees.

## Step 9: Review Failure Modes

The demo should include at least one failing path for each category:

- insufficient bond;
- too many nomination targets;
- nomination target is not a validator candidate;
- payout for an era that has not ended;
- unbond withdrawal before the bonding duration has passed;
- validator intent without session-key readiness.

These failures are important because they prove the course covers runtime
constraints, not only successful calls.

## Evidence Package

For a final course submission, include screenshots, logs, or test output showing:

- genesis validator and nominator setup;
- pre-election validator set;
- post-election validator set;
- payout result;
- unbonding and withdrawal timeline;
- at least three negative tests.

Do not include private keys, seed phrases, payment identifiers, or personal
account details in the evidence.
