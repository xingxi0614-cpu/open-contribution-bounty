# Review Checklist

Use this checklist when reviewing the NPoS course submission for issue #5.

## Bounty Fit

- The module is about a Substrate chain with Nominated Proof of Stake.
- The guide describes validators, nominators, sessions, eras, rewards, and
  slashing.
- The guide is implementation-oriented, not only a high level article.
- The content references official Polkadot or FRAME material.
- The content does not claim toy-chain economics are production-ready.

## Technical Correctness

- Validator candidates are not treated as active validators immediately.
- Staking is connected to session rotation.
- Session keys are required for active validators.
- The validator count can be smaller than the candidate count.
- Nominators have bounded nomination targets.
- Reward payout includes validator commission.
- Slash risk is shared by validators and exposed nominators.
- Era and session boundaries are described separately.
- Election limits and runtime weight concerns are discussed.
- Unbonding waits for a bonding duration before withdrawal.

## Demo Completeness

- The suggested genesis has multiple validators and nominators.
- The walkthrough covers bonding, validating, nominating, era advancement,
  payout, chilling, and unbonding.
- The test list covers state transitions, not only happy path calls.
- The failure modes help learners debug realistic staking mistakes.
- The security notes call out production risks and review areas.
- The demo runbook gives reviewers concrete state checks to request.

## Maintainability

- File names and paths follow the Track 1 course convention.
- Markdown is readable in GitHub diffs.
- Links are current and official.
- No public payout details are embedded in the course material.
- The submission can be reviewed without running a custom external service.
- The evidence package avoids private keys, seed phrases, and payment details.
