# Implementation Checklist

Use this checklist to build the course module for a local NPoS chain. Each item
should be either implemented, documented as intentionally simplified, or left
with a clear learner exercise.

## Repository Structure

- Add the module under `track-1/nominated-proof-of-stake-blockchain/`.
- Keep the learner guide in `README.md`.
- Keep builder checks in `implementation-checklist.md`.
- Keep maintainer checks in `review-checklist.md`.
- Keep local verification steps in `demo-runbook.md`.
- Use short Markdown lines so the content is easy to review in diffs.

## Runtime Pallets

- Include `pallet_balances` for the staking asset.
- Include `pallet_staking` for bonded stake and NPoS state.
- Include `pallet_session` for validator set rotation.
- Include a block production pallet suitable for a solo chain.
- Include a finality pallet suitable for a solo chain.
- Include `pallet_authorship` for validator reward points.
- Include `pallet_offences` or a documented mocked offence path.
- Include an election provider for validator selection.
- Include a bounded voter list strategy for election weight safety.

## Runtime Configuration

- Define local demo constants for sessions per era.
- Define a short bonding duration for learner visibility.
- Bound the number of validators.
- Bound the number of nominations per nominator.
- Configure reward payout behavior.
- Configure slash destination and slash cancellation authority.
- Configure validator commission behavior.
- Configure session key conversion.
- Connect staking as the session manager.
- Use generated weights or explicitly call out template weights.

## Genesis Configuration

- Seed enough balances for all validator and nominator accounts.
- Seed at least three validator candidates.
- Set the validator count lower than the candidate count.
- Seed at least two nominators with overlapping nominations.
- Register session keys for initial validator authorities.
- Set reward destinations for demo accounts.
- Document which accounts are learners, validators, and nominators.

## Learner Flow

- Show how to bond funds.
- Show how to declare validator intent with `validate`.
- Show how to nominate validator candidates with `nominate`.
- Show how to inspect staking ledger and nomination state.
- Show how to advance sessions and eras.
- Show how to inspect the active validator set.
- Show how to call `payout_stakers`.
- Show how validator commission affects rewards.
- Show how to chill a staker.
- Show how to unbond and withdraw after the bonding duration.

## Tests

- Test genesis staking state.
- Test validator candidate registration.
- Test nomination limits.
- Test invalid nomination rejection.
- Test era transition.
- Test active validator set change.
- Test reward payout effects.
- Test chilling before a later era.
- Test unbonding and withdrawal timing.
- Test non-staker rejection paths.
- Test missing session key failure or safety handling.
- Test offence or slash routing when included.

## Documentation Quality

- Explain the difference between validator candidates and active validators.
- Explain that nominators share both rewards and slash risk.
- Explain why election limits matter for runtime weight.
- Explain why production constants should not be copied from a toy chain.
- Explain the role of sessions and eras separately.
- Explain where production benchmarking is required.
- Link to official Polkadot and FRAME references.

## Acceptance Evidence

- Provide the local chain spec or document the template used.
- Record the initial validator set.
- Record the validator set after an era transition.
- Record at least one nominator backing an elected validator.
- Record one successful payout call.
- Record one chill and unbond flow.
- Record at least three rejected calls for invalid staking actions.
- Do not publish private keys, seed phrases, or payout identifiers.
