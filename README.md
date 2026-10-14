<!-- solve-problem:4ace497a-c9fd-43c9-8d32-eaf873eb05e7 -->
# Measuring endogenous tie-winning advantage

A [Tangle Solve problem](https://solve.tangle.tools/problems/4ace497a-c9fd-43c9-8d32-eaf873eb05e7).

## What is the problem?

In proof-of-work blockchains, when two miners find blocks nearly simultaneously, the "winner" of the resulting fork is determined by whose block reaches the next miner to find a block first. This creates a **tie-winning advantage**: miners whose blocks propagate faster hear about competing blocks earlier and get their own blocks heard earlier, so their blocks are more likely to survive. Formally, if miner *i*'s blocks propagate to the hashpower-weighted network with CDF *F_i(t)*, miner *i*'s effective revenue exceeds their raw hashpower share *q_i* by a factor depending on the distribution of inter-block propagation delays relative to the block interval.

The problem: **empirically estimate the tie-winning advantage of specific mining organizations** as a function of (a) relay-network access (e.g., FIBRE, Falcon, private relay), (b) geographic colocation of hashing facilities, and (c) block size / bandwidth constraints — while accounting for **unobservable private infrastructure**. The core difficulty is that the advantage is *endogenous*: large miners invest in exactly the propagation advantages we want to measure, so naive OLS of orphan/uncle rate on observables is confounded. A satisfactory treatment must deliver per-organization advantage estimates with quantified uncertainty (confidence intervals or posterior credible sets), not a single scalar "network propagation delay" parameter as prior work does. The most reasonable formalization: a causal/measurement model mapping miner-specific propagation distributions to fork-survival probabilities (via the Bitcoin selfish-mining/stubborn-mining reward framework), with parameters identified from on-chain fork data plus network probes, and private infrastructure treated as latent confounding.

## Why does it matter?

A single "average propagation delay" parameter hides that large pools effectively hold more power than their nominal hashpower share, which corrupts all security analyses (51%-attack thresholds, selfish-mining profitability) that assume propagation is uniform. Regulators and protocol designers estimating decentralization currently cannot distinguish a pool with 20% hashpower from one with effectively 22%. Concrete outputs — per-miner advantage estimates — feed directly into attacker-specific incentive models for selfish mining, undercutting, and consensus-parameter choices (block size, block interval), and into decentralization auditing. It also closes a measurement gap open since Decker–Wattenhofer (2013), which documented propagation-driven forking but never attributed advantage to specific actors.

## What's already known?

- **Decker and Wattenhofer (2013)**: first large-scale measurement of Bitcoin's information propagation; measured block propagation CDFs and showed propagation delay is a principal cause of forks (their Figure-based estimate tied fork rate to the propagation distribution). Established the network layer as a security variable but treated the network as homogeneous.
- **Gervais et al. (2016)**: quantitative model linking propagation (quantified via a Γ-distribution fit of propagation times) to stale rates, and stale rates to the profitability of double-spending and selfish mining; showed two-step attacks are profitable depending on propagation advantage. This is the key theoretical bridge from *measured delays* to *attack incentives* — but it uses a network-wide delay parameter, not per-miner parameters.
- **Selfish-mining literature** (Eyal–Sirer 2014; Sapirshtein et al. 2016, "Optimal Selfish Mining Strategies") (verify): establishes the formal tie-winning parameter λ (probability the attacker wins a fork race) and shows profitability depends critically on it — but treats λ as a free parameter, never measured.
- **Measurement tooling**: DSN/Ivy-style propagation monitoring, FIBRE relay network stats, bitcoind `getblocktemplate`/`inv` timing studies; network tomography methods for inferring unobserved topology.
- **Gap**: no work (to our knowledge) estimates organization-level λ values with uncertainty, or addresses identification when miners use private relays that passive measurement cannot see.

## What would count as solving it?

1. **A measurement + inference pipeline** producing per-major-pool (or per-geographic-cluster) estimates of effective tie-winning probability λ_i or effective-hashpower multiplier, with confidence/credible intervals, validated against ground truth from known events (e.g., forks resolved in observable races).
2. **An identification strategy for private infrastructure**: either (a) a latent-variable model where unobserved relays are inferred from residuals in fork-resolution outcomes, (b) active probing that bounds the unobservable's effect, or (c) explicit sensitivity analysis giving worst-case advantage given the unobserved investment budget.
3. **A validation experiment**: simulation (Simbit/Shadow/ custom PoW simulator with heterogeneous topology) where true λ_i is known, demonstrating the estimator recovers it; plus application to real Bitcoin/Ethereum (uncle-rate) data.
4. Downstream: plugging estimated λ_i into selfish-mining profitability formulas to produce organization-specific attack-profitability bounds.

Partial results that count: tight upper bounds on any pool's advantage under a stated observation model; or a method on public-data miners only with quantified bias from private relays.

## How to start

1. **Read Gervais et al. (2016) §III–IV first** — it gives the propagation→stale-rate→incentive chain you will parameterize per-miner; then Eyal–Sirer / Sapirshtein et al. for the formal λ in fork races.
2. **Build the estimator skeleton**: model each block race as a contest where survival probability depends on hashpower-weighted propagation CDFs F_i; you observe fork outcomes (main chain vs stale, timestamps, coinbase tags identifying pools). Write the likelihood and check what is identified from fork data alone (likely: only relative, not absolute, advantages — a key subtlety).
3. **Simulate**: extend a PoW simulator (or Gervais's model) with heterogeneous per-miner delays; verify the estimator on ground truth before touching real data. This is cheap and will reveal identification failures early.
4. **Attack the private-infrastructure problem last**, via sensitivity analysis: "if an unobserved relay gives miner i delay d, its advantage is bounded by f(d)" — this reframes the unobservable as a bounded nuisance parameter and is defensible even without full identification.

Main obstacle: attribution (pool tags are noisy, hashpower shares drift) and endogeneity (advantage correlates with size, which determines fork frequency — the events you condition on).

## Key references

- C. Decker and R. Wattenhofer. *Information Propagation in the Bitcoin Network*. IEEE P2P 2013. DOI:10.1109/P2P.2013.6688704. — First empirical measurement of Bitcoin block/transaction propagation delays and their causal link to forks.
- A. Gervais, G. O. Karame, V. Capkun, S. Capkun. *On the Security and Performance of Proof of Work Blockchains*. ACM CCS 2016. DOI:10.1145/2976749.2978341. — Quantitative framework converting propagation-delay distributions into stale rates and double-spending/selfish-mining profitability; the propagation–incentive bridge this problem per-minerizes.
- I. Eyal, E. G. Sirer. *Majority is not Enough: Bitcoin Mining is Vulnerable*. Financial Cryptography 2014. (verify DOI:10.1007/978-3-662-45472-5_28) — Introduces the tie-breaking parameter λ governing selfish-mining profitability; the target quantity's formal home.
- A. Sapirshtein, Y. Sompolinsky, A. Zohar. *Optimal Selfish Mining Strategies in Bitcoin*. FC 2016. (verify) — Computes optimal attack profitability as a function of propagation/tie-winning parameters, giving the downstream consumer of your estimates.

## Warnings

- **Endogeneity is the crux, not an afterthought**: fast propagation both causes tie-wins and correlates with miner size; regressions of stale rate on observables will be biased and a referee will notice.
- **Λ is relative**: tie-winning probability in selfish-mining models is defined against a specific adversary/propagation contrast; define carefully whether you estimate absolute propagation CDFs or pairwise advantages.
- Pool attribution via coinbase tags is incomplete and adversarially spoofable; report uncertainty from mislabeling.
- Fork events are rare and censored (you only observe races that became forks); selection bias in the conditioning event.
- Private relays (FIBRE-era successors) are invisible to passive measurement — a naive estimate conflates "no advantage" with "unobserved advantage". Census notes this explicitly; treat as latent.
- Timestamps in block headers are not reliable propagation times; you need monitoring-node data or inference, as Decker–Wattenhofer did.

---
*`BCGT-000262` · flagged for review*

## Acceptance guidance

1. **A measurement + inference pipeline** producing per-major-pool (or per-geographic-cluster) estimates of effective tie-winning probability λ_i or effective-hashpower multiplier, with confidence/credible intervals, validated against ground truth from known events (e.g., forks resolved in observable races).
2. **An identification strategy for private infrastructure**: either (a) a latent-variable model where unobserved relays are inferred from residuals in fork-resolution outcomes, (b) active probing that bounds the unobservable's effect, or (c) explicit sensitivity analysis giving worst-case advantage given the unobserved investment budget.
3. **A validation experiment**: simulation (Simbit/Shadow/ custom PoW simulator with heterogeneous topology) where true λ_i is known, demonstrating the estimator recovers it; plus application to real Bitcoin/Ethereum (uncle-rate) data.
4. Downstream: plugging estimated λ_i into selfish-mining profitability formulas to produce organization-specific attack-profitability bounds.

Partial results that count: tight upper bounds on any pool's advantage under a stated observation model; or a method on public-data miners only with quantified bias from private relays.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/4ace497a-c9fd-43c9-8d32-eaf873eb05e7?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000262.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000262)
