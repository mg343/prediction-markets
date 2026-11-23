# remaining_vote

How much of a single market's election-night confidence comes from public information (the structure of the uncounted vote), and how much comes from information beyond it?

*A project with Andrew Gelman's lab: Statistical Modeling, Causal Inference, and Social Science (Columbia).*

## Background

On election night this year, the Mississippi Senate race was as close as a race gets. With 95.6% of the vote counted, Scott Colom had 462,857 votes and Cindy Hyde-Smith had 462,792, a gap of 65 votes out of more than 925,000. Roughly 40,000 votes were still uncounted, hundreds of times the margin. News trackers called it too close to call. Kalshi had Colom at 92%.

At first glance, those two views contradict each other. If you treat the remaining 40,000 votes as a random handful from the state, they should split almost exactly like the votes already counted, and a 65-vote lead is a coin flip. So why would anyone pay 92 cents for a Colom win?

## The key observation

The remaining votes aren't a random handful. Votes are counted in a structured order. Some counties report faster than others. Mail ballots, early votes, and same-day votes are often counted at different times, and they lean differently because different kinds of voters use each method. Within a county, some precincts come in later than others. So by 95.6% counted, the uncounted 4.4% is a specific set of ballots from specific places, and those places have known voting patterns.

If the outstanding ballots came mostly from counties that had been voting heavily for Colom, a 65-vote lead was really a lead of several hundred or a few thousand once those ballots were in. The easily accessible trackers may have been showing the current count, but the market, in line with Gelman's research on smart money, may have been pricing the projected final count.

## The question

Is the market's confidence on election night explained by the structure of the remaining vote, meaning which ballots were still uncounted and how those places had voted? Or was the market using information beyond the public count?

The question matters because the two answers say very different things about markets. If a careful model of the remaining vote also reaches about 92%, the market wasn't doing anything mysterious. It was doing public arithmetic faster than the people reporting the count, and the "information aggregation" was mostly speed. If the model says something like 65% while the market says 92%, then either the market knew something the public count didn't show, or it was overconfident. Telling those two apart is a second step.

It's also a question at the center of Gelman's work: drawing conclusions about a whole population from a sample that isn't random, while being honest about how uncertain the answer still is.

## Methodology

The methodology is built from five basic components. Each one can be done crudely first and refined later, and each answers a different sub-question:

1. **Public benchmark.** What would a careful observer with only public information believe at each moment? We answer this with a set of models of the remaining vote.
2. **Data.** Can we reconstruct the night? We need the count, the public forecasts and the market price, all on one shared clock.
3. **Comparison.** How closely did the market track the public benchmark, and who moved first?
4. **Interpretation.** If the model says X and the market says Y, what does the gap mean, and how much of it is private information?
5. **Market structure.** When *should* a market track public information at all, and does that depend on how big the market is?

Component 6 repeats everything across many races, because one race is an anecdote.

### 0. Framing: nowcasting, not forecasting

It helps to be precise about what's uncertain on election night. A normal election forecast predicts a future event: how people *will* vote. On election night the outcome is already fixed, because the votes have been cast and are sitting in ballot boxes and mail trays. The only uncertainty is about *what's inside the ballots that haven't been counted yet*. That's a nowcasting problem: estimating a quantity that already exists but hasn't been fully observed.

This framing makes "private information" concrete. On election night it can only be one of two things:

- **Knowledge of uncounted ballots.** Some people see parts of the remaining vote before it's published: campaign precinct watchers, county election officials, local reporters at counting centers, and campaigns with their own tabulations.
- **Better processing of public data.** Others see nothing private but do better arithmetic on what's public. They might use precinct-level detail instead of county totals, know which vote types are still out, or simply compute faster than the trackers.

In the price, both look the same: the market is "ahead" of the public models. Separating them is part of component 4. A good benchmark (component 1) shrinks the second category, because the more carefully we process public data, the less of the gap can be explained by better processing.

### 1. The public benchmark: models of the remaining vote

The goal is a *set* of models, each using only information that was public at the time, that output P(win) at every moment of the night. We use several models rather than one for two reasons. First, no single model is "the" public view. Second, the spread between reasonable models measures how much the public information itself was open to interpretation. If careful models disagree by ±10 points, a market 5 points away from one of them tells us nothing.

The models come in three tiers, from simplest to most sophisticated.

**Tier 1: simple baseline (ours).** A transparent, almost mechanical rule built from the last couple of elections. For each county:

- **remaining votes** = expected turnout (past turnout, scaled to this year's registration or early vote) − votes counted so far;
- **remaining lean** = the county's share so far, or its last-election share shifted by the statewide swing observed so far tonight.

Multiply and add across counties to get a projected final margin. Take the uncertainty from how wrong this exact rule was when it was applied to past elections. This tier is roughly what a "naive public" trader with a spreadsheet could have computed. If the market matches even this, the market's confidence needs no explanation at all.

**Tier 2: hierarchical model (ours, Gelman-style).** The careful version, built in three parts.

- *Remaining ballots R<sub>c,t</sub> per county c at time t.* We run this in two modes and keep them separate.
  - **Real-time mode** estimates turnout from past elections, registration, and early-vote/mail-return data available that night. This is what a trader could actually have known.
  - **Oracle mode** uses final certified totals, so R = final − counted. This removes turnout uncertainty and isolates how much the *lean* mattered. It checks the method rather than predicting in real time.

  The gap between the two modes shows how much of the uncertainty is about *how many* ballots remain versus *how they lean*.
- *Remaining lean.* We model the remaining share for one candidate as the share so far plus a late-vote shift:

  logit(*p<sub>c,t</sub><sup>rem</sup>*) = logit(*p<sub>c,t</sub><sup>counted</sup>*) + δ<sub>c</sub> + ε<sub>c,t</sub>

  - δ<sub>c</sub> is the typical early-to-late shift, learned from past elections. Where ballot-type data exists, it's modeled per type (mail, early, election day) and weighted by the mix still outstanding.
  - **Partial pooling.** County parameters are pooled across similar counties (grouped by region, urbanicity, past partisanship and ballot-type mix). A county with only a sliver reported has a noisy share so far, so it borrows strength from its neighbors instead of being trusted on its own numbers. This is the core Gelman technique and where most of the modeling care should go.
  - **Unreported counties** get the previous comparable race's result shifted by tonight's observed swing.
- *Simulation.* Draw turnout and lean from their uncertainty ≥10,000 times. For each draw, add the simulated remaining votes to the current count and record the winner. The share of draws a candidate wins is the model's probability.

  **Errors must be correlated.** If each county's error is drawn independently, the errors cancel out across counties and the model becomes badly overconfident. In reality, late ballots tend to miss in the same direction everywhere (for example, if mail ballots are bluer than expected, they're bluer everywhere). So the simulation needs a shared shock at the statewide, regional or ballot-type level. That choice can move the final probability a lot, so we report results under several correlation assumptions rather than pick one.

**Tier 3: existing state-of-the-art live models.** Professional newsrooms already build election-night models, and they are the strongest public benchmark. We won't train a new model from scratch: it isn't the research question, and it's unlikely to beat models refined over many cycles. Instead we either rerun open-source models on historical count snapshots or use their archived outputs. We'd retrain something only if a race type has no existing model. Candidates (whether code and archives are available is still to be verified):

- **NYT Needle.** Precinct-level, with published methodology. It's the best-known public probability on election night, so it's the most natural "what the public was told" series.
- **Washington Post election-night model.** Open source, I believe (`elex-live-model`). If so, it can be rerun on any snapshot of the count, which means we wouldn't depend on archived outputs at all. That would make it the most useful benchmark for historical races.
- **Decision Desk HQ.** Publishes projections and race calls.
- **AP and the network decision desks.** These give race *calls*, not probabilities. They're still useful as event timestamps: "at time *t* the public was told it's over." The market's behavior just before a call is especially informative.

**Is the county the right unit?** The default plan works county by county, because county totals are what is reported and archived most reliably. But that may be the wrong level of detail, and the choice isn't neutral. Finer units are:

- **county × vote type** (mail / early / election day). Different kinds of voters use each method, and they're counted at different times, so this is often the biggest driver of late shifts (the "red mirage / blue shift" pattern).
- **precinct or reporting unit.** This is what the NYT Needle uses. Within a county, the precincts still out may be very different from those already in.

A coarse model hides exactly the structure that makes the remaining vote predictable, so a county-only model may *understate* what was publicly knowable. In that case we would wrongly credit the market with private information when it was just using finer public data. To test this, run the models at different levels of detail. If the finer models move toward the market, part of the apparent "private" gap was an artifact of the coarse model.

### 2. Data: three series on one clock (likely the bottleneck)

The analysis needs three time series that line up minute by minute. Finding them, especially for past races, is probably the hardest practical part of the project.

| Series | Role | Sources |
|---|---|---|
| Raw counts (county, ideally × vote type / precinct) | Input to our models | State Secretary of State feeds, archived tracker snapshots, Wayback Machine captures |
| Public forecasts | Benchmark | Archived Needle/DDHQ outputs, or recomputed by rerunning open-source models on count snapshots |
| Market prices | Object of study | Kalshi export (trades, bid/ask, volume); Polymarket as a second market where it exists |

- **Alignment.** Stamp every update with when it became *publicly visible*, not the time printed in the feed. These can differ by minutes, and minutes are exactly the scale at which we'll measure whether the market "led." Then resample everything to a common grid (e.g. 1 minute) and carry the last value forward. Counts and forecasts update in jumps while prices move continuously, so record when each jump happened. Those moments are the events in the event study in component 3.
- **Resolution rules.** Record exactly how each contract settles: certification, the outcome of a recount, or a named outlet's call. In a 65-vote race, part of the price may reflect recount or litigation risk rather than the count itself, and a contract that settles on certification can't be compared directly with a model of election-night totals.
- **Coverage risk.** Live result pages overwrite themselves, so snapshots from past nights may not exist for many races. If that's the case, part of the project becomes *prospective*: building scrapers that are ready to record counts, forecasts and prices on upcoming election nights. That's slower, but it gives cleaner data than anything reconstructed.

### 3. Comparison at each timestamp

At each time *t*, compare the market price with each model and with the ensemble of models.

**Hypothesis.** If election-night information is essentially public, the market should track the public models closely. A visible "blue wave" across the counties that have reported, for example, is something every trader and every model can see at once. This should hold especially when the market is large and widely traded, so that the price reflects many participants rather than a few people with inside knowledge. A tight match supports the idea that market "information aggregation" on election night is mostly speed at public arithmetic.

Metrics:

- **Correlation, with care.** Price and model both converge to 0 or 1 as the count finishes, so correlation in raw levels is inflated: almost any two sensible series look highly correlated by the end of the night. To avoid fooling ourselves, compute correlation on the logit scale, on *changes* rather than levels, and only in windows before the race is effectively decided.
- **Gap.** The mean and absolute gap, market − model, in logit space. The mean says whether the market is systematically more confident; the absolute gap says how far apart the two typically are.
- **Lead/lag.** Two ways of asking who moved first:
  - an *event study* around each count update: did the price move *before* the update (anticipation, suggesting the market knew what was coming) or *after* it (fast processing of public news)?
  - *cross-correlation* of the two series, finding the time shift at which they agree best.
- **Accuracy.** Once outcomes are known, compute Brier score and log loss for the market and each model as a function of time before the race is called. This shows who was right and when.

Before any of this, clean the price. Fees, the bid-ask spread, the favorite-longshot bias (long shots tend to be overpriced, favorites underpriced) and thin late-night liquidity all push the price away from a pure probability. Use the mid-price and carry the spread as an uncertainty band.

### 4. Interpretation: the model says X, the market says Y

This is the second half of the picture. Suppose the county model says 70% and Kalshi says 92%. A gap does **not** automatically mean private information. There are at least six possible sources:

1. **Private information.** Someone knows what's in the uncounted ballots.
2. **Better public processing.** Traders use finer detail or faster arithmetic than our models (see "Is the county the right unit?").
3. **Model error or disagreement.** Our models are simply wrong, or they disagree among themselves.
4. **Market frictions and biases.** Fees, spread, favorite-longshot bias, risk aversion, limits to arbitrage (traders who see a mispricing may not have the capital or appetite to correct it), and thin liquidity.
5. **Resolution and recount risk.** The contract settles on something other than the election-night count.
6. **Overconfidence or noise.** Traders are simply too sure.

The job is to separate these. Ways to quantify private information:

- **Public envelope.** Shade the range spanned by the reasonable public models. If the market sits inside it, there's no evidence of private information: the market just picked a point within the public range.
- **Encompassing test.** Regress the outcome on logit(model) and logit(market) together, pooled over races and timestamps with errors clustered by race (in the style of Fair–Shiller). If the market gets weight beyond the model, it contains information the public models don't. If all the weight goes to the model, the market's deviations were noise.
- **Information in bits.** Compute the market's log score minus that of the best public model, averaged over races. This turns "how much private information" into a number: bits of extra information per race. It's a natural unit for comparing races and markets.
- **Does the gap predict the next batch?** If the market − model gap at time *t* predicts which way the next count update leans, the market knew something about the ballots before they were published. If it doesn't, the gap is noise.
- **Information vs. overconfidence.** When the market is more confident than the model, does the favorite win at the *market's* implied rate or at the *model's*? Pooled across races, this is a calibration test on Δ = market − model. Winning at the market's rate means the extra confidence was earned; winning at the model's rate means it was overconfidence.

**Is "they should be close" even the right assumption?** The comparison in component 3 assumes the price equals the consensus belief built from public data. That assumption can fail in both directions:

- The two can be *far apart with no private information*, because of frictions (source 4) or because traders use better public models than ours (source 2).
- They can be *close despite private information*, if that information is small relative to what's already public, or if public models and insiders happen to agree.

So closeness is neither necessary nor sufficient to rule private information in or out. Correlation alone can't answer the question, which is why the encompassing and predictive tests above carry the real weight.

### 5. Market structure: when should a market track public information?

Whether a market *should* match public models may depend on what kind of market it is.

**Hunch.** Some markets are thin, or their outcome is decided by a few people, for example "what word will a CEO say on stage." There, public forecasts are a poor guide: a handful of people know or control the answer, and a few informed traders can set the price. Deep markets are different. With tens of millions of dollars and many participants, any one trader's quirks or outlier bets should average out over the larger population, and the price should sit near the public consensus. An election-night market is the second kind: millions of voters decide the outcome and the market is large.

**Counterpoint.** Averaging cancels *noise*, not *information*. Idiosyncratic errors wash out in a big market, but a trader who actually knows something has the most reason to bet heavily precisely where liquidity is deep. A deep market may therefore still contain private information; it just gets priced in faster. So depth plausibly shrinks noise-driven gaps while leaving information-driven gaps in place, or even making them sharper.

**Test.** Across races, plot the gap size, the encompassing weight and the bits of extra information against market volume, open interest and number of traders. If gaps shrink with depth, depth mostly removes noise. If the information measures *grow* with depth, informed traders are concentrating where the money is. As an extension, contrast elections (millions of voters decide, and private information can only be about the *count*) with few-decider markets (private information is about the *decision itself*).

### 6. Repeat across many races

One race can't settle any of this. With dozens of close races across several election nights (House, Senate, governor, and ballot measures where markets exist), every metric above becomes an average with honest uncertainty around it.

- **Race selection.** Choose races using a rule that was knowable in real time (e.g. margin < X% with ≥ Y% counted), not "races that turned out close." Selecting on the final result builds the answer into the sample.
- **Results by market size.** Break every result down by depth bucket, so the component 5 question gets answered along the way.

## What results look like

1. **Race timeline.** For each race: Kalshi price with a band for the bid-ask spread, each model's probability as its own line, count updates as ticks along the time axis, and a vertical line at the AP call. This is the headline figure; you should be able to see at a glance whether the market ran ahead of the models.
2. **Gap timeline.** Market − ensemble in logit space over the night, with the public envelope (the spread of the models) shaded. Periods where the market leaves the envelope are the interesting ones.
3. **Model comparison table.** One row per model (Tier 1, Tier 2 at each level of detail, each Tier 3 model) plus the market. Columns: Brier score, log loss, calibration, mean |gap| to Kalshi, correlation with Kalshi (on logit changes) and lead/lag.
4. **Correlation matrix.** All models plus Kalshi, computed on logit changes. It shows whether the market clusters with the sophisticated models, the simple ones, or neither.
5. **Event-study plot.** Average price move in windows before and after count updates. A rise *before* updates is the signature of private information.
6. **Calibration of Δ.** Realized win rate vs. the market's and the model's probabilities, binned by how far the market and model disagree. This answers "information or overconfidence."
7. **Cross-race scatter.** Gap size or bits of extra information vs. market volume or trader count. This answers the market-structure question.

## Open questions / to do

- [ ] Survey live election-night models and confirm which publish code or archived outputs (Needle, WaPo, DDHQ).
- [ ] Find timestamped county × vote-type results for Mississippi 2026, and check whether any archive has precinct-level data.
- [ ] Pull Kalshi trade and order book history for the MS Senate market; check whether Polymarket had a market on it.
- [ ] Build the Tier 1 baseline on MS as a first end-to-end pass, even if crude.
- [ ] Assemble the list of candidate races (2026 and earlier cycles that had markets), with volume data.
- [ ] Decide how to handle recount and resolution risk in the price.
- [ ] If archives are thin: set up scrapers for upcoming election nights.
