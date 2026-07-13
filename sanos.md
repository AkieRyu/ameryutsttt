# sanos — snapshot 2026-05-19

> Buehler-Horvath-Kratsios-Limmer-Saqur (2026) SANOS reproduction
> (arXiv:2601.11209v3). **E primary mode**. Path:
> `D:/AkieRyuPcloud/UIUC/26Spring/research/RNDwork/sanos/`.

## Role in the paper

**Upstream dearbitrage supplier** for Algorithm 1's `(S-input)` contract.
sanos consumes raw (possibly arb-violating) European call prices on a
non-uniform $(K, T)$ grid; outputs a smooth, strictly arb-free,
forward-aligned discrete grid that satisfies our chord lower bound by
construction. The paper's numerical experiments cite sanos as **the
chosen** upstream tool — readers can substitute any other tool that meets
the same input-layer chord condition.

## Production status (E5b/E6)

- **38 -> 56 tests** pass (Session 4 added per-T r/d + non-uniform 2nd-FD verifier).
- **Production LP**: MDL formulation + boundary sentinels (`K_min/K_max` auto-pad).
- **Production default**: $\eta = 0.36$ scalar (E3 ablation; ADR 0004). $\eta = 0.50$
  has slightly lower mean error but worse worst-quote.
- **Dual-mode export** (ADR 0003): `raw` (same lattice) or `forward_aligned`
  ($k_l^{T_j} = k_l^{T_{\rm ref}} \cdot e^{(d-r)(T_{\rm ref} - T_j)}$).
- **Per-T r/d** (ADR 0006, Session 4 2026-05-18): drops the silent
  "use slices[0].r/d" bug; outputs CSV with per-maturity (r, d) tuples and
  sidecar yaml metadata.
- **Per-T strike usability** (ADR 0007): `first_usable_strike_index_per_T` +
  `spacing_transitions_per_T.indices_per_T` exposed in sidecar.

## Key experiment runs (2026-05-17 / 2026-05-18)

| Run | Setup | Result |
|---|---|---|
| E1 baseline | SSVI synth, η=0.25, no sentinels | mean_rel = 1.04% |
| E2 patched | + A/C'/H/J patches (sentinels auto) | mean_rel = 0.68% (−35%) |
| E2.5 recommended | η=0.36 + sentinels | mean_rel = 0.36% **production default** |
| E3 η-ablation | sweep η ∈ {0.04..0.70} | η=0.50 best mean (0.27%); η=0.36 best worst-quote |
| **E5 SPX 2022-02-22 (7T)** | real OptionMetrics, PCP fwd + implied dividend, η=0.36 | LP 7.6s, butterfly 966/966 + S-chord 840/840 PASS at machine-eps |
| **E5b rd-fix** | + ADR 0006 per-T r/d | 7 distinct (r,d) per T; same arb pass; 56 tests pass |
| **E6 SPX 30T menu** | 30 maturities 1d→661d | "maturity menu" for Alabs to pick from; subset of 6 → 100% PASS at tol=1e-6 |

## Data sources used by paper

- **SSVI synthetic** (paper Section 5.1/5.2): SSVI Power-Law η=2, γ=0.2, ρ=0.5,
  θ_t=0.4t; sanos consumes this synthetic grid and outputs arb-free input for
  Alabs Algorithm 1 (forward-aligned mode).
- **SPX 2022-02-22 real** (paper Section 5.3): IvyDB OptionMetrics call quotes,
  PCP-implied forward + dividend per maturity, 30-T menu → Alabs picks 6 maturities.

## Public API (sanos -> Alabs bridge)

- `src/io/loader.py` — CSV / DataFrame ingestion.
- `src/io/grid_export.py` — dual-mode output (raw / forward_aligned).
- `src/io/surface_io.py` — `SanosSurface` callable evaluator.
- `bridge/sanos_csv_to_alabs_grid.py` — drop-in adapter: sanos CSV → Alabs MarketDataGrid.

## Open items (not blocking paper)

- E5b multi-day vol-regime sweep (5-10 dates).
- E5c real-data η sweep (may favor η=0.50 on SPX-realistic smoother surfaces).
- E4 end-to-end smoke test (sanos → Alabs Algorithm 1 chain in one Python call).

## Why we cite sanos specifically

(1) Active sister project we co-developed during this paper. (2) Honest
reproduction of the most recent state-of-the-art smooth arb-free surface
constructor. (3) Output contract exactly matches our input contract by
design. (4) Readers can swap in any equivalent tool — the paper's chord
condition is the contract, sanos is one supplier.
