# prediction_markets

## Mihir's research projects/ideas in prediction markets, with Gelman, Sethi, etc.

## Projects

### [remaining_vote](remaining_vote/)

*With Andrew Gelman's lab: Statistical Modeling, Causal Inference, and Social Science (Columbia).*

How much of a market's election-night confidence comes from public information, meaning the structure of the uncounted vote, and how much from information beyond it? The motivating case is the 2026 Mississippi Senate race: with 95.6% counted, Colom led by 65 votes and trackers called it too close to call, yet Kalshi had Colom at 92%. Because ballots are counted in a structured order (by county, precinct and vote type), the uncounted vote is not a random sample, and its lean may have been predictable from public data.

The plan is to build a set of public benchmark models of the remaining vote, from a simple county baseline to a hierarchical model with partial pooling and correlated errors, alongside existing live models such as the NYT Needle and the Washington Post model. We then compare them minute by minute with market prices. Encompassing tests, lead/lag event studies and calibration checks separate private information from better public processing, market frictions and overconfidence. Repeating this across many races asks whether market depth reduces noise or concentrates informed trading.
