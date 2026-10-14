<!-- solve-problem:db3f8c1e-0261-437f-a98b-a14e35620fae -->
# Oracle-aware adaptive borrowing limits

A [Tangle Solve problem](https://solve.tangle.tools/problems/db3f8c1e-0261-437f-a98b-a14e35620fae).

## What is the problem?

Setting: a DeFi lending protocol (Protocol for Loanable Funds, PLF) where users borrow against collateral whose price is reported by an oracle (e.g., a decentralized exchange spot price or an aggregated feed). Positions are overcollateralized by a loan-to-value (LTV) ratio / collateral factor θ chosen per asset.

The problem: design **adaptive collateral limits** θ(t) that depend on the *economic cost of manipulating the price oracle* for that asset, rather than a static governance-voted number. Formally, given:

- an oracle mechanism O reporting price p̂ of asset X,
- an attacker with capital C who can move p̂ to p̂′ at cost κ(p̂′; C, venue liquidity, time), profiting V(p̂′, θ) from over-borrowing,
- a manipulation-cost function that accounts for **cross-venue effects**: the attacker can recycle capital across trading venues and lending venues, and liquidity migrates between venues during/after the attack,

find the rule θ* = f(κ, liquidity state) such that no attack with capital C ≤ C_max is profitable, while keeping θ as large as possible (credit efficiency). The open part is a principled, ideally provably safe, mapping from a time-varying, cross-venue manipulation-cost estimate to borrowing limits. A game-theoretic/dynamic-systems formulation (attacker–protocol–LP game, possibly repeated) is the most reasonable precise interpretation; the census statement is informal about the exact threat and utility model, so the solver must fix these.

## Why does it matter?

Oracle manipulation is the dominant loss vector for on-chain lending (numerous multi-million-dollar incidents via low-liquidity spot oracles). Current collateral factors are static and set by governance intuition, forcing an awkward tradeoff between capital efficiency and safety. A sound adaptive rule would let protocols raise LTV on well-secured assets and automatically de-risk assets whose manipulation cost drops as liquidity migrates, and would inform how multiple lending venues can be composed without creating circular amplification of credit.

## What's already known?

- The attack pattern (flash-loan or capital-funded spot-price manipulation to over-borrow) is well documented empirically in incident post-mortems and measurement papers on DeFi attacks.
- SecPLF (Arora et al., 2024) gives formal protocol designs and security definitions for PLFs against oracle-manipulation attacks; it is the reference point for the 2024 state of the art (verify exact results — treat as the baseline formalism).
- Related theory exists on manipulation cost curves for single venues (cost ≈ quadratic in trade size times liquidity depth, à la constant-function AMMs), on flash-loan amplification bounds, and on manipulation-resistant benchmarks / robust aggregation (the sensor-fusion analogy in the record).
- Known gap: analyses typically model a *single venue with static liquidity*. They do not model (a) capital recycling across venues mid-attack, (b) liquidity migration changing κ over the attack horizon, or (c) multiple lending venues sharing the same oracle. This is exactly what the census flags as open.

## What would count as solving it?

1. **Model:** a formal game specifying attacker capital, venues, liquidity dynamics, oracle aggregation, and protocol payoff.
2. **Main theorem:** a bound θ*(κ_t, L_t) such that manipulation is unprofitable (or expected loss ≤ ε) for all attacker strategies including cross-venue capital recycling — plus a matching or near-matching lower bound showing larger θ is exploitable.
3. **Dynamics:** stability/convergence of the adaptive rule under liquidity migration (no oscillation or gaming of the adaptation itself).
4. Optionally: a mechanism-design version where the rule is incentive-compatible and cannot be gamed by LPs moving liquidity to inflate θ.

A referee wants: precise threat model, theorem with proof, and either a counterexample/optimality gap or a construction (simulation on realistic liquidity data is supporting, not sufficient).

## How to start

1. Read SecPLF (arXiv:2401.08520) for its security definitions and see where the single-venue/static-liquidity assumption enters; that's the extension point.
2. Write down the simplest two-venue model: two AMMs, one shared price oracle or two correlated oracles, attacker moves liquidity/trades between them; derive κ as a function of both depths. Check whether recycling ever *lowers* effective cost (it plausibly does — that's the interesting lemma).
3. Treat θ selection as a robust optimization: θ = max θ s.t. sup over attacks of (profit) ≤ 0, with κ parameterized; then add dynamics (liquidity follows a flow/replicator dynamic) and analyze stability.
4. Main obstacle: κ is not observable directly — it must be estimated from on-chain liquidity, and the estimator itself becomes an attack surface. A mechanism where the estimate is strategically robust (truthful liquidity reporting) is likely needed.

## Key references

- Arora, Li, Feng, Xu (2024), "SecPLF: Secure Protocols for Loanable Funds against Oracle Manipulation Attacks," arXiv:2401.08520. Formalizes oracle-manipulation attacks on lending protocols and proposes secure protocol designs; the baseline this problem extends beyond. https://arxiv.org/abs/2401.08520
- (Fill in from the empirical literature on oracle-manipulation incidents and on AMM price-impact/manipulation-cost bounds — the census record does not name specific papers, so verify and select primary sources yourself.)

## Warnings

- The census record is informal; the terms "capital recycling" and "liquidity migration" are not standard with fixed definitions — you must define them and defend the choice.
- Do not assume manipulation cost is just AMM slippage on one pool; cross-venue and temporal effects are the whole point, and single-venue formulas will silently underestimate attacker power.
- Beware the adaptive rule becoming a new attack surface (manipulate the inputs to θ to pump borrowable amounts). Any solution must address strategic manipulation of the adaptation itself.
- Attribution/details of SecPLF beyond arXiv ID should be verified before citing results from it.

---
*`BCGT-000347` · flagged for review*

## Acceptance guidance

1. **Model:** a formal game specifying attacker capital, venues, liquidity dynamics, oracle aggregation, and protocol payoff.
2. **Main theorem:** a bound θ*(κ_t, L_t) such that manipulation is unprofitable (or expected loss ≤ ε) for all attacker strategies including cross-venue capital recycling — plus a matching or near-matching lower bound showing larger θ is exploitable.
3. **Dynamics:** stability/convergence of the adaptive rule under liquidity migration (no oscillation or gaming of the adaptation itself).
4. Optionally: a mechanism-design version where the rule is incentive-compatible and cannot be gamed by LPs moving liquidity to inflate θ.

A referee wants: precise threat model, theorem with proof, and either a counterexample/optimality gap or a construction (simulation on realistic liquidity data is supporting, not sufficient).

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/db3f8c1e-0261-437f-a98b-a14e35620fae?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000347.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000347)
