# The Clown Project — V0.1

A causality-first market-microstructure research pipeline for reconstructing the Binance Spot BTCUSDT visible order book, aligning trades to observable book states, constructing auditable event streams, and evaluating whether a Hawkes point process adds information beyond simpler intensity models.

## Research Question

Can short-horizon BUY and SELL market-event arrivals be modeled adequately by a self-exciting Hawkes process after preserving:

* collector-time causality;
* exact-time simultaneity;
* visible-book reconstruction authority;
* locked development and calibration boundaries;
* simpler point-process comparators; and
* strict separation of predictive score from full model adequacy?

The central question is not merely whether a Hawkes model obtains a better likelihood score. It is whether that improvement survives the diagnostics required to treat the model as a credible representation of the observed event-generating process.

## Project Design

V0.1 begins with immutable Binance Spot BTCUSDT trade, depth, snapshot, and collector-metadata records.

The pipeline then:

1. freezes the source and run contracts;
2. reconstructs the visible market-by-price order book;
3. aligns each trade with the latest book state observable before the trade;
4. converts trades into auditable BUY and SELL event representations;
5. constructs causal pre-event market-state features;
6. evaluates simpler point-process baselines;
7. estimates a restricted bivariate Hawkes model;
8. evaluates the frozen model on locked calibration data;
9. applies residual, path, multiplicity, and stationarity diagnostics; and
10. closes the run without opening protected partitions when the model fails adequacy.

The project follows a fail-closed design. A model is not advanced merely because it outperforms a comparator on one metric.

## Data and Event Definition

V0.1 uses one continuous, approximately one-hour Binance Spot BTCUSDT collection.

The authoritative point-process representation is:

> `SAME_MS_SAME_SIDE_BURSTS`

Trades occurring on the same side within the same millisecond are represented as a single event burst while retaining membership, count, quantity, notional, timestamp, and conservation records.

The source stream was divided into four ordered partitions:

* `DEVELOPMENT`
* `CALIBRATION`
* `VALIDATION`
* `ENGINEERING_HOLDOUT`

Model estimation and selection used `DEVELOPMENT`.

The selected model was frozen before `CALIBRATION`. No parameters were updated during calibration.

`VALIDATION` and `ENGINEERING_HOLDOUT` remained unopened because the calibration adequacy gates failed.

## Models Evaluated

### Simpler baselines

The authorized baseline families included:

* side-specific homogeneous Poisson models;
* deterministic elapsed-time Poisson models;
* causal rolling-rate Poisson models; and
* causal exponentially weighted rate models.

The primary simple baseline was:

> `E_SIDE_EWMA_250MS_POISSON`

The formal comparator was:

> `D0_SIDE_CONSTANT_POISSON`

### Hawkes specification

The first authorized Hawkes specification was:

> `H1_DIAGONAL_SHARED_DECAY`

This is a diagonal bivariate exponential Hawkes process with:

* BUY-to-BUY excitation;
* SELL-to-SELL excitation;
* a shared decay parameter;
* zero BUY-to-SELL excitation; and
* zero SELL-to-BUY excitation.

Cross-side excitation and state-dependent Hawkes extensions were not authorized in V0.1.

## Main Result

The frozen H1 Hawkes model ranked first on the locked calibration count score.

On 2,493 observed calibration events:

| Model                       | Total log score |
| --------------------------- | --------------: |
| H1 diagonal Hawkes          |      -14,914.46 |
| EWMA 250 ms Poisson         |      -18,072.44 |
| Rolling 250 ms Poisson      |      -18,095.80 |
| Homogeneous Poisson         |      -18,440.78 |
| Constant Poisson comparator |      -18,440.78 |

H1 improved the total log score by approximately:

* **3,158** relative to the primary EWMA baseline; and
* **3,526** relative to the formal constant-Poisson comparator.

Paired block-bootstrap comparisons supported positive count-score improvement against every authorized baseline.

The fitted model also remained inside the declared stationarity boundary:

* spectral radius: approximately `0.1893`;
* stationarity margin: approximately `0.8107`.

## Why the Model Was Rejected

Better count scoring did not establish full model adequacy.

H1 failed four blocking calibration diagnostics:

1. **Time-rescaling residual distribution**

   The transformed residuals were inconsistent with the required reference distribution.

2. **Residual serial dependence**

   Dependence remained after conditioning on the fitted Hawkes intensity.

3. **Path-count calibration**

   Observed event-count paths were not adequately reproduced by the model-generated reference envelopes.

4. **Exact-time multiplicity**

   The model did not reproduce the observed concentration of simultaneous or same-time event multiplicities within the declared tolerance.

No repair, parameter update, replacement model, or post-hoc specification change was performed after these failures.

## Final V0.1 Decision

> **V0.1 CLOSED — ENGINEERING PASS — H1 COUNT-SCORE ADVANTAGE RETAINED — H1 GENERATIVE ADEQUACY REJECTED — NOTEBOOK 09 SKIPPED — PREDICTIVE VALUE UNTESTED — PROTECTED PARTITIONS UNOPENED — V0.2 HANDOFF COMPLETE**

The permitted interpretation is narrow:

* H1 contains statistically supported count-score information relative to the authorized simpler baselines.
* H1 is not an adequate generative model of the calibration event stream.
* H1 is not authorized as a trading, quoting, market-making, or predictive signal.
* No out-of-sample predictive claim is made.
* The failed model is retained only as diagnostic evidence and as a reference for successor research.

## Notebook Sequence

| Notebook                                             | Purpose                                                                                                                     |
| ---------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `00_V01_RUN_CONTRACT.ipynb`                          | Freezes the source authority, run identity, operating contract, and raw-data audit boundary.                                |
| `02_VISIBLE_BOOK_RECONSTRUCTION.ipynb`               | Independently reconstructs the visible Binance BTCUSDT order book from the REST snapshot and differential-depth stream.     |
| `03_CAUSAL_TRADE_BOOK_ALIGNMENT.ipynb`               | Aligns each trade to the latest visible book state already observable to the collector.                                     |
| `04_EVENT_STREAM_CONSTRUCTION.ipynb`                 | Constructs individual and grouped BUY/SELL event streams while preserving causal and conservation invariants.               |
| `05_MARKET_STATE_FEATURES.ipynb`                     | Builds causal pre-batch market-state features without using information from the current or future event batch.             |
| `06_POINT_PROCESS_BASELINES.ipynb`                   | Evaluates simpler Poisson and adaptive-rate models and determines whether Hawkes estimation is justified.                   |
| `07_HAWKES_ESTIMATION.ipynb`                         | Estimates, selects, freezes, and replays the restricted-first Hawkes specification.                                         |
| `08_HAWKES_DIAGNOSTICS.ipynb`                        | Compares H1 with the authorized baselines and performs the locked calibration adequacy tests.                               |
| `10_V01_FINAL_AUDIT_SYNTHESIS_AND_V02_HANDOFF.ipynb` | Independently audits the full run, preserves the accepted and rejected findings, closes V0.1, and defines the V0.2 handoff. |

Notebook 09 was intentionally not executed. Intensity-signal validation was not authorized after H1 failed the blocking model-adequacy gates.

## Engineering Principles

The pipeline was designed around the following controls:

* immutable source identities and SHA-256 verification;
* explicit source-authority registration;
* clean-kernel execution;
* persisted-input-only handoffs;
* causal as-of alignment;
* no same-event feature leakage;
* exact-time batch preservation;
* trade, event, quantity, and notional conservation;
* predeclared comparator and model families;
* frozen calibration evaluation;
* zero calibration parameter updates;
* protected validation and holdout partitions;
* machine-readable decision ledgers;
* artifact manifests and read-back verification; and
* failure preservation without retrospective repair.

## Scope and Limitations

V0.1 is an engineering and diagnostic research run based on one continuous collection.

It does not establish:

* generality across trading sessions;
* stability across market regimes;
* cross-venue portability;
* economic profitability;
* execution feasibility;
* market-making value; or
* out-of-sample predictive performance.

The short single-session source is sufficient for testing the causal research architecture and identifying model failure modes, but not for making broad empirical claims.

V0.2 is intended to extend the design to a larger, multi-session dataset with stronger temporal and regime coverage.

## Reproduction Note

The notebooks preserve the original executed project contracts, artifact identities, outputs, and local filesystem paths.

Full re-execution requires:

* the registered raw Binance source files;
* the expected V0.0 and V0.1 artifact hierarchy;
* compatible Python dependencies; and
* either the original local directory structure or equivalent path remapping.

The executed notebooks can also be read directly as the complete V0.1 research record.

## Author

**Donn Bryan Julian**

Independent AI & Data Consultant
Quantitative Research
Economic Forecasting
Systematic Strategy Development
