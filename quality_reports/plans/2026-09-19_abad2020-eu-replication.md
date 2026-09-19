# Plan: Replicating Abad, Ferreras & Robles (2020) on European corporate bonds

**Date:** 2026-09-19
**Status:** DRAFT — awaiting owner approval
**Paper:** Abad, P., Ferreras, R., Robles, M.-D. (2020). *Information opacity and corporate bond returns:
The dynamics of split ratings.* J. Int. Financ. Markets Inst. Money 68, 101239.
**Scope:** empirical replication only. From the current unmerged files to every table in the paper.

**Basis.** The full paper was read (13 pages, all tables). Three of the four data files were profiled
directly (`FR Bond List.xlsx`, `Historical_Ratings_Datset.csv`, `Treasury_Match_Final.xlsx`).
`Bond_Time_Series_Dataset copy.csv` was **not** in the push and has not been seen; everything that
depends on it is marked **[TS]** and rests on the description in the treasury workbook's notes.

---

## 1. What the paper actually does (verified against the PDF)

| Element | Paper (Section, Table) |
|---|---|
| **Events** | *Negative* rating events only: downgrades **and** additions to negative watch, by S&P, Moody's, Fitch. Same-day announcements by several CRAs = one "multi-rating" event (§2). 16,625 events, US, USD, plain-vanilla fixed-rate, 1 Oct 2004 – 31 Dec 2014. Excludes callable/redeemable, convertible, ABS, linked and perpetual bonds (fn. 8). |
| **Rating scale** | 58-point Sy (2004) scale: AAA = 58 down to CC/SD/D = 1; one notch = 3 points, a watch change = 1 point (§2). |
| **Liquidity filter** | At least 4 traded prices between consecutive rating events on the same bond (§2). |
| **Split** | Eq. (1): `Split_b,t = max(CR_S&P, CR_Moody's, CR_Fitch) − min(...)` over the CRAs rating the bond that day. |
| **Dynamics** | `Widen = 1{Split_t > Split_t-1}`, `Narrow = 1{Split_t < Split_t-1}`; 29% narrow, 45% widen, 27% unchanged (Table 3). |
| **Excess return** | Eq. (2): `ER_i = [ (P_t2 − P_t1)/P_t1 − (T_t2 − T_t1)/T_t1 ] × 365/(t2 − t1)`. **Clean** prices; `t1` = last price before the event, `t2` = first price after; `T` = price of a maturity-matched Treasury computed from **zero-coupon yields** (Bloomberg); annualised. `Window W` = trading days between t1 and t2, used as a control (§3). |
| **Matched non-event sample** | For every event, one non-event day drawn at random (with replacement) from the same issuer and the same NBER period; models run on events + matches, so N = 2 × events (§3, fn. 17). |
| **Model 1** | `ER = β0 + β1 CR_-1 + β2 ΔCR + λ W + issuer FE + period FE` |
| **Model 2** | + `β3 Widen + β4 Narrow` |
| **Model 3** | + `β5 Widen×Split_t + β6 Narrow×Split_t` (Split at the **event** date) |
| **Panel B controls** | Senior, Volume (= **amount issued**, not trade volume), SupSP (S&P gives the highest rating), Covenants, Risk (Δ Moody's AAA corporate index yield), Multi-rating, #CRA following, Maturity (Table 4 note). |
| **Estimation** | OLS, Huber-White robust SE, issuer and period dummies. |
| **Headline numbers** | Mean ER: events −0.321 vs matched +0.082 (t = −8.07). Narrow = −0.439*** (M2), −0.739*** with Narrow×Split = +0.081*** (M3). |
| **IG vs HY** | IG = still ≥ BBB− after the event; HY = ≤ BB+ before it; fallen angels excluded; models without CR_-1 (Table 5, 6). HY: Narrow −2.758***, Narrow×Split +0.447***; IG: nothing. |
| **Alternatives** | ED = Euclidean distance between the three ratings; EDW = distance of each rating to the worst; Widen/Narrow redefined on each (Eq. 6–7, Table 7). |
| **Descriptives** | Table 1 (events by CRA, ΔCR size, #CRA following, multi-rating), Table 2 (issues, issuers, maturity, senior, covenants), Table 3 (split distribution), Fig. 1 (events by year), Table 5 (same by IG/HY). |

Two consequences that change the earlier assumptions: the paper needs **no trade volume** anywhere,
and **watch status enters only through the 58-point scale and the event definition**. Neither of the
two variables you said you lack is a blocker.

---

## 2. What your files actually contain

### 2.1 `FR Bond List.xlsx` — 4,546 bonds, 1,205 issuers (sheet `Search Results`)
- Key: `ISIN` (4,545 distinct; one Vonovia SEK bond has no ISIN), `Preferred RIC`, `Ticker`.
- Static fields the paper needs: `Coupon`, `Maturity`, `Issue Date`, `Amount Issued (EUR)` (23 rows
  non-numeric), `Seniority Type` (SR 3,793 / UN 355 / SRSEC 301 / secured & subordinated tail),
  `Callable` (**2,387 yes, 52%**), `Putable` (7), `Perpetual` (32), `Guaranteed`, `Bond Grade`
  (IG 1,914 / HY 437 / **missing 2,195**), `Coupon Type` (plain vanilla 4,533).
- Currency: EUR 3,000, CHF 490, GBP 423, NOK 303, SEK 230, HUF 65, CZK 17, ISK 12, PLN 6.
- Issue years: heavily 2016–2026 (3,996 of 4,546); a handful of pre-1990 dates look like
  placeholders (`1900-01-02` ×5) and need checking.
- No covenant field. No issue-level rating.

### 2.2 `Historical_Ratings_Datset.csv` — 23,789 rows, 649 companies, 1975–2026
- **Issuer-level, not issue-level.** Key is `Company` (name), no ISIN. 637 of 647 company names match
  a bond-list issuer after normalisation; 3,405 of 4,546 bonds belong to a rated issuer.
- Each row is a rating *action*: `Agency`, `Rating Code` + `Seniority` (the rating type), `Rating`,
  `Rating Date`, plus `Watch Code`, `Outlook`, effective dates. Rows are duplicated across
  `Market` = Foreign / Domestic: 23,789 rows → 17,674 unique ignoring `Market`.
- Agencies: Moody's 9,667, S&P 7,653, Fitch 4,448, Egan-Jones 1,733, minor others.
- Rating types to use as the bond's rating (one series per agency): S&P `SSU` (senior unsecured) or
  `SPI` (LT issuer); Moody's `MSU` or `MIS`; Fitch `FSU` or `FDL`. Ignore `-PD`, `LGD…`, `MCF`,
  short-term codes, national scale. Strings carry `(P)` (provisional), `*` / `**` markers (915 / 646
  rows; confirm the vendor legend, `**` may be "under review"), `(EXP)` (expected), `WR`/`NR`/`WD`.
- **Watch history is effectively absent**: `Watch Code` has 53 real values, `Watch Eff Date` 53.
  `Outlook` has 1,812 dated values. So negative-watch events cannot be built.
- Issuer-level downgrade census (Big Three, LT senior/issuer series, one series per issuer-agency):

| | Count |
|---|---|
| Downgrades / upgrades | 1,568 / 1,006 |
| Downgrades by agency | S&P 676, Moody's 476, Fitch 416 |
| One-notch downgrades | 1,315 (84%) |
| Issuer rated by 1 / 2 / 3 CRAs at the event | 352 / 587 / 629 |
| Downgrades with ≥ 2 CRAs: narrow / widen / unchanged | 446 / 520 / 247 (of 1,216) |
| Split > 0 before the event (≥ 2 CRAs) | 73%; mean pre-split 1.2 notches |
| Same-day multi-CRA downgrades | 54 |
| IG→IG / HY→HY / fallen angels | 985 / 470 / 113 |
| HY downgrades with ≥ 2 CRAs: narrow / widen | 122 / 203 (of 385) |
| Downgrades from 2016 on | 721 (2020 alone: 153) |

  These are issuer-events. Bond-events will be several times larger (each issuer has ~3.8 bonds),
  but only for bonds that were outstanding and priced on the event date.

### 2.3 `Treasury_Match_Final.xlsx` — benchmark workbook (six sheets)
- `map_tenor` / `static_match`: per-ISIN remaining maturity and a **US** GSW zero yield and zero price
  at issue and at 2026-09-04. `treasury_curve_daily` (1961–2026, 1y–30y) and `gsw_parameters`
  (daily Svensson β0–β3, τ1, τ2) let you evaluate the curve at any maturity on any day.
- `treasury_at_event`: **1,265 events already exist** (748 downgrades, 517 upgrades; keyed by RIC;
  557 RICs, 107 issuers; 2007–2025 with 83% from 2019 on; `dcr_notches` 1–5). 1,180 RICs match the
  bond list and 1,181 (issuer, date) pairs match a `Rating Date` in the ratings file. Someone has
  therefore built a bond-level event file before; its rules are not in the workbook.
- The notes say: "Currency is irrelevant to the match: one US Treasury for all currencies (Maylis)",
  and mention a separate **price file**, an **excess-return file** and an **FX file** that are not in
  the repo.

### 2.4 `Bond_Time_Series_Dataset copy.csv` — **not received** [TS]
From the notes it is a bond × day panel of about 11M rows. Needed on day one. If it is over
GitHub's 100 MB limit, gzip it (`gzip -k`), or push a `head -n 200000` sample plus the column list
so the schema can be fixed while the full pull is arranged.

---

## 3. Gap analysis and the decisions it forces

| # | Paper | Your data | Decision (recommended in bold) |
|---|---|---|---|
| D1 | Events = downgrades + negative watch, 58-pt scale | No watch history | **Events = downgrades only, on a 22-notch scale (AAA = 22 … D/SD = 1).** ΔCR in notches; the paper's 3 points = 1 notch, so coefficients on ΔCR/Split rescale by ×3 when compared. Record in the deviation ledger. Outlook changes are *not* events (the paper does not use outlook either). |
| D2 | Issue-level ratings (Mergent) | Issuer-level ratings | **Attach each issuer's senior-unsecured (fallback: LT issuer) rating per agency to every bond of that issuer.** Bonds with `Seniority Type` ≠ SR/UN/SRSEC-with-no-secured-rating inherit only if the agency has no matching seniority series. One issuer downgrade → one bond-event per priced bond. Cluster SE by issuer-event (the paper uses issuer dummies plus robust SE; add clustering as robustness). |
| D3 | US Treasury zero curve, USD bonds | 66% EUR, plus CHF/GBP/NOK/SEK; benchmark built on the **US** curve | **Benchmark in the bond's own currency: ECB euro-area AAA Svensson curve for EUR (daily parameters published since 2004-09-06), BoE nominal zero curve for GBP, SNB for CHF; drop or robustness-only for NOK/SEK/HUF/CZK/ISK/PLN.** The paper subtracts the Treasury return to remove *risk-free rate moves in the bond's own market*; a USD curve leaves euro rate moves inside the "excess" return and adds USD rate noise. Keep the existing US-curve version as a robustness column since it is already built. **This overturns a choice attributed to Maylis; raise it with them before building.** |
| D4 | Excludes callable bonds (fn. 8) | 52% callable | **Primary sample keeps callable bonds and adds a `Callable` control; robustness re-runs on non-callable only.** Most post-2010 European IG bonds carry make-whole or 3-month par calls that are not economically meaningful optionality; excluding them halves the sample and skews it to older IG issues. If Refinitiv gives call type, exclude only genuine (non-make-whole) callables. |
| D5 | 4 traded prices between events | Quoted / evaluated prices [TS] | **Replace with a staleness filter:** require price *changes* on ≥ 4 distinct days between consecutive events, and drop bonds whose share of zero-return days over the prior 60 days exceeds a cut set after inspecting the distribution. |
| D6 | t1 = last trade before, t2 = first trade after | Daily quotes [TS] | With quotes, t1 = day −1 and t2 = day 0 or +1 for nearly all events, so `W` will be nearly constant. **Keep Eq. (2) exactly as the primary ER; add a fixed (−1, +1) and (−1, +5) window as robustness.** Confirm from the vendor whether the quote is end-of-day; an announcement after the close belongs to day +1. |
| D7 | Covenants, Risk (Moody's AAA index) | No covenant field; no index series | Covenants: **drop** (state in ledger) unless Refinitiv exports it. Risk: **use the daily change in the ICE BofA / iBoxx Euro AAA corporate yield** (or the ECB AAA sovereign 10y as a fallback). |
| D8 | Volume issued | `Amount Issued (EUR)` | Same variable; use log amount. |
| D9 | 2004–2014, 16,625 events | ~2007–2025, ~750–1,500 downgrades (issuer level), bond-level larger | **Sample = all priced years; report 2008–2025.** Period dummies: replace NBER phases with **CEPR euro-area cycle dates** plus a COVID dummy (2020 holds 10% of downgrades). Power: the HY × narrow cell has ~120 issuer-events; multiply by bonds per issuer, but expect Table 6's HY coefficients to have wide intervals. |
| D10 | Fitch as third rater | Fitch present; Egan-Jones present | Big Three only in the main tables. Egan-Jones as an added robustness for ED/EDW if coverage is material. |
| D11 | — | 1,265 prebuilt events in `treasury_at_event` | **Rebuild events from the ratings file with written rules, then reconcile against the 1,265**: every difference must be explained (rating type used, dedup across `Market`, treatment of `(P)`/`*`/`WR`). Do not inherit a file whose rules are unknown. |

---

## 4. Table map

| Paper | Content | Module | Notes for the European version |
|---|---|---|---|
| Table 1 | Events by CRA (share, avg CR, avg ΔCR, best/worst rater); ΔCR size; #CRA following; multi-rating share | M5 | ΔCR in notches |
| Table 2 | # issues, # issuers, avg years to maturity, senior share, covenant share | M5 | covenant row dropped; add callable share, currency split |
| Table 3 | Split distribution: ΔCR, Split, Split_-1, ΔSplit, narrow/widen/unchanged | M5 | in notches |
| Fig. 1 | Events by year | M5 | |
| §4 text | Mean ER events vs matched, t-test and sign test | M6 | |
| Table 4 A/B | Models 1–3, without and with controls | M7 | + Callable, − Covenants |
| Table 5 | Table 3 by IG / HY | M5 | |
| Table 6 A/B | Models 1–3 by IG / HY, no CR_-1 | M7 | |
| Table 7 A/B | Models 2–3 with ED and EDW | M8 | |
| new | Benchmark-currency robustness (own-currency vs US curve), callable-excluded, fixed windows, issuer-clustered SE | M8 | European additions |

---

## 5. Pipeline

All code under `explorations/abad2020_eu/` (repo sandbox protocol), R with `data.table` + `fixest`
(Python is equally fine; keep the module boundaries). Each module writes one `.rds` the next reads.
Raw files are never edited. Move the four raw files out of the repo root into
`explorations/abad2020_eu/data/raw/` — the hygiene gate already flags all four at root.

### M0 `00_inventory.R` — freeze the facts in §2 as a script
Re-runs the profiling above from the raw files and writes `output/inventory.md`. When the time
series arrives, extend it with: long vs wide shape, price field (clean / dirty / bid / ask / mid),
frequency, date range, share of zero-return days per bond, and the ISIN overlap with the bond list.

### M1 `01_bonds.R` — bond universe
- Parse dates, coupon, `amount_eur` (numeric; 23 failures to inspect), `callable`, `perpetual`,
  `seniority` collapsed to {senior, senior-secured, subordinated, other}, `currency`, `issuer_name`
  and a normalised `issuer_key` (the normaliser used in §2.2: lower-case, strip legal suffixes).
- Sample filters, logged one line each (this is Table 2's provenance): plain-vanilla fixed coupon;
  not perpetual; not putable; not convertible/with warrants; currency in {EUR, GBP, CHF} for the main
  sample; issue date sane (drop the 1900 placeholders or fix them from the vendor).
- Output: `bonds.rds`, one row per ISIN.

### M2 `02_ratings.R` — issuer × agency rating history
- Drop `Market` duplicates; keep Big Three; keep LT senior-unsecured series (`SSU`/`MSU`/`FSU`),
  fall back to LT issuer (`SPI`/`MIS`/`FDL`) when an issuer-agency has no senior-unsecured series.
  Write the choice per issuer-agency to `output/rating_series_choice.csv`.
- Clean strings: strip `(P)`, `*`, `**`, `(EXP)`, `u`, `e` into flags; map to the 22-notch scale;
  `WR`/`NR`/`WD` end coverage (not a downgrade). Write the mapping table to disk.
- Build the **daily state** table: issuer × agency × date, rating carried forward until the next
  action or withdrawal. From it: `split_t`, `n_cra_t`, `ED_t`, `EDW_t`, `sup_sp_t` (S&P holds the
  highest rating), for every issuer-day.
- Output: `rating_state.rds`, `rating_actions.rds`.

### M3 `03_events.R` — issuer downgrade events → bond events
- Issuer-event: a fall in the numeric rating of an agency's series. Fields: `agency`, `date`,
  `cr_pre`, `cr_post`, `dcr` (notches, positive), `split_pre`, `split_post`, `widen`, `narrow`,
  `n_cra`, `multi` (another Big-Three series falls the same day; for multi events take the max ΔCR
  as the paper does, fn. 16), `sup_sp`, `ed_pre/post`, `edw_pre/post`, `ig_pre`, `ig_post`,
  `fallen_angel`.
- Bond-event: join to every bond of that issuer outstanding on the date (issue < date < maturity).
  Keep `issuer_event_id`.
- **Reconcile against `treasury_at_event`** (D11): match on (RIC, date), report matched /
  only-here / only-there with the reason for each unmatched row.
- Output: `events_issuer.rds`, `events_bond.rds`, `output/event_reconciliation.md`.

### M4 `04_prices_and_er.R` — returns and excess returns **[TS]**
- Reshape to long if wide; one row per ISIN-date; keep clean price (paper uses clean; if only dirty
  is available, strip accrued from coupon, frequency and day count and validate on ten bonds).
- Staleness diagnostics and the D5 filter.
- Benchmark price: for each bond-day, remaining maturity `n` in years and the own-currency zero
  yield `y(n)` from the currency's Svensson parameters (ECB for EUR; the workbook's GSW for the US
  robustness); `T = exp(−y/100 × n)`. Note the paper's `T` is a *price*, so `(T_t2 − T_t1)/T_t1`
  is the return on a zero maturing with the bond.
- For each bond-event: `t1` = last priced day before the event date, `t2` = first priced day on or
  after it (D6), `ER` per Eq. (2) annualised with 365/(t2 − t1), `W` = trading days between.
- Output: `panel.rds` (ISIN-date with price, benchmark, staleness flags), `er_events.rds`.

### M4b `04b_matched_sample.R` — non-event control sample
- For each bond-event draw one non-event day (with replacement) for the **same bond**, same period
  (D9), at least 30 days from any rating action of that issuer; compute ER over (t1, t2) around the
  drawn day exactly as for events; set `dcr = 0`, `widen = narrow = 0`, `cr_-1` and `split` at their
  current values. Seed fixed (`set.seed(20260919)`). The paper draws from the same issuer; the same
  bond is the closest equivalent when ratings are issuer-level.
- Output: `sample_full.rds` = events ⊕ matches, `event = 1/0`.

### M5 `05_descriptives.R` → Tables 1, 2, 3, 5, Fig. 1
Row and column layout identical to the paper's so the side-by-side comparison is visual.

### M6 `06_univariate.R` → the §4 preliminary test
Mean and median ER for events vs matches; t-test, sign test, Wilcoxon; by direction of split
change and by IG/HY.

### M7 `07_models.R` → Tables 4 and 6
`fixest::feols(ER ~ cr_pre + dcr + widen + narrow + widen:split + narrow:split + W + controls |
issuer + period, vcov = "hetero")` for Panel A/B, all / HY / IG. Controls: senior, log amount,
sup_sp, risk, multi, n_cra, maturity, **callable**. Report also `vcov = ~issuer_event`.
Acceptance test (the abstract's three claims), each reported as sign / magnitude / p-value next to
the paper's: (1) `narrow < 0` in M2; (2) `narrow:split > 0` in M3; (3) both hold in HY, not in IG.

### M8 `08_robustness.R` → Table 7 and the European additions
ED / EDW versions; own-currency vs US benchmark; non-callable only; fixed (−1, +1) and (−1, +5)
windows; issuer-clustered SE; EUR-only; financials vs non-financials (European samples are
bank-heavy and bank opacity is the Morgan 2002 origin of the split-rating literature).

### M9 `09_reconcile.R` — paper vs replication
One table: every coefficient and descriptive in the paper you can map, your estimate, the ratio
after the ×3 scale conversion, and agreement in sign/significance. Plus the **deviation ledger**
(seeded in §7). This is what the capstone's results and discussion are written from.

### Checks that gate each module (`*_checks.R`, stop on failure)
No duplicate keys; every bond-event date inside the bond's priced range; narrow + widen + unchanged
= events with ≥ 2 CRAs; `split_pre` equals the previous day's state-table split for 100% of events;
20 hand-checked events (10 IG, 10 HY) traced back to the raw ratings rows; ER recomputed by hand for
5 events from raw prices and the curve parameters.

---

## 6. Feasibility and power

- Issuer-level: ~1,216 downgrades with a split defined, 446 narrow. HY with split: 385, of which
  122 narrow. Bond-level multiplies by outstanding priced bonds per issuer (median unknown until
  [TS]; the bond list averages 3.8 bonds per issuer, 5–15 for large issuers).
- The paper's Table 4 effects are detectable at these sizes; Table 6 HY effects will be estimated
  with wide intervals. Say so in the write-up rather than pooling until something is significant.
- **Preregister the three acceptance-test claims before running M7** (`/preregister`); with a
  sample this size the result is credible either way only if the specification was fixed first.

---

## 7. Deviation ledger (seed — every row must survive into the paper's appendix)

| Item | Paper | Replication | Why |
|---|---|---|---|
| Events | downgrades + negative watch | downgrades only | no watch history in the ratings export |
| Scale | 58-point | 22-notch | no watch; ×3 conversion stated |
| Rating level | issue | issuer senior-unsecured / LT issuer | export is issuer-level |
| Market | US, USD, TRACE trades | Europe, EUR/GBP/CHF, quoted prices [TS] | design of the study |
| Benchmark | US Treasury zero curve | own-currency sovereign zero curve; US as robustness | D3 |
| Callable | excluded | included with control; excluded in robustness | D4 |
| Liquidity filter | 4 trades between events | staleness filter | quotes, not trades |
| Covenants | control | dropped | not in export |
| Risk | Δ Moody's AAA US index | Δ euro AAA corporate index | market |
| Period dummies | NBER phases | CEPR phases + COVID | market |
| Matched sample | same issuer | same bond | ratings are issuer-level |

---

## 8. Order of work

| Step | Effort | Blocked by |
|---|---|---|
| Push the time series; move raw files under `explorations/abad2020_eu/data/raw/` | 0.5 h | — |
| Decide D3 (benchmark currency) and D4 (callable) with your supervisor | a conversation | — |
| M0, M1, M2 | 1.5 days | — |
| M3 + reconciliation with the 1,265 prebuilt events | 1 day | M2 |
| Pull ECB AAA Svensson parameters (and BoE/SNB curves if GBP/CHF stay in) | 0.5 day | D3 |
| M4, M4b | 2 days | time series [TS], M3 |
| M5, M6 | 1 day | M4 |
| M7, M8 | 1.5 days | M4b |
| M9 + write-up | 1–2 days | M7, M8 |

**Longest-lead item:** the time-series file. **Highest-stakes decision:** D3, because every ER in
every table depends on it and the version already built uses the other choice.

---

## Suggestions outside this plan's scope (owner decides)
- Ask Refinitiv for the *issue-level* rating history by ISIN (`TR.IssueRating…` fields exist); it
  would remove deviation D2 entirely.
- Ask for the call-type field to make D4 a clean exclusion rather than a control.
