# Risk Scoring Methodology

This document describes how Raport360 computes its Risk Score and A-E rating for Polish companies. The methodology is rule-based and heuristic. Empirical validation against historical bankruptcy filings is in progress; see "Validation" below.

## Overview

Raport360 produces two headline numbers for every company in scope:

- Risk Score: integer in the range 0-100, where 0 is the lowest observed risk and 100 is the highest.
- Rating: a letter grade from A (very safe) to E (high risk), derived directly from the Risk Score.

Both are computed from a weighted combination of 12 factors extracted from public Polish registries and financial filings.

## Data Sources

| Source | Coverage | Refresh |
|--------|----------|---------|
| KRS (Krajowy Rejestr Sadowy) | ~700,000 registered companies | Daily via official API |
| CEIDG (Centralna Ewidencja i Informacja o Dzialalnosci Gospodarczej) | 683,000+ sole proprietorships | Daily via official API |
| MF e-Reports (Krajowy Rejestr Zadluzonych / e-Sprawozdania) | Financial statements of companies obligated to file | On publication, pulled within 24h |
| VAT Whitelist (Biala Lista podatnikow VAT) | All active VAT payers | Daily |

Freshness guarantees:

- KRS and CEIDG snapshots have a maximum staleness of 24 hours under normal operation.
- Financial statements are reflected within 24 hours of becoming available in the MF repository.
- VAT Whitelist status is rechecked daily; an on-demand recheck is available on the company page.

## Risk Score Formula

The score is a normalized weighted sum:

```
RawScore   = sum( w_i * normalized_factor_i )   for i = 1..12
RiskScore  = round( 100 * RawScore / sum(w_i) )
```

Each factor is independently normalized to the range [0, 1] where 0 indicates the lowest risk contribution and 1 indicates the highest. The weights `w_i` are listed per factor below. The mapping from raw value to normalized value is also documented per factor.

Missing data is handled per factor: when a factor cannot be computed (for example, no financial statement exists), it is excluded from both numerator and denominator so that companies with partial data are not unduly penalized or rewarded.

## Factors

### 1. Company Age

- Data source: KRS / CEIDG registration date.
- Normalization: piecewise linear. Age < 1 year => 1.0; 1-3 years => 0.7; 3-7 years => 0.4; 7-15 years => 0.2; > 15 years => 0.0.
- Weight: 10.
- Rationale: First-year failure rates are substantially higher than for established firms. Survivorship at 7+ years is a strong negative risk signal.

### 2. Share Capital

- Data source: KRS field "kapital zakladowy".
- Normalization: log-scaled against form-specific medians. Capital at or below the legal minimum => 0.8; capital at the median for the legal form => 0.3; capital at the 90th percentile => 0.0.
- Weight: 6.
- Rationale: A capitalization at the legal floor is a weak signal of shell or low-substance setup. Higher capital provides a buffer against insolvency.

### 3. Management Board Changes

- Data source: KRS history of entries in section "Organ uprawniony do reprezentacji".
- Normalization: change count within the trailing 24 months. 0 changes => 0.0; 1-2 => 0.2; 3-5 => 0.6; 6+ => 1.0.
- Weight: 7.
- Rationale: High churn in board composition correlates with operational instability and, at the extreme, with control-evasion patterns.

### 4. VAT Whitelist Status

- Data source: Biala Lista API.
- Normalization: active VAT payer => 0.0; exempt (zwolniony) => 0.2; removed (wykreslony) => 1.0; never registered (where VAT registration is expected) => 0.7.
- Weight: 9.
- Rationale: Removal from the VAT whitelist is one of the strongest publicly available signals of fiscal distress or fraud risk and has direct legal consequences for counterparties.

### 5. Altman Z'-Score

- Data source: MF e-Reports (latest annual statement).
- Normalization: Z' > 2.9 => 0.0 (safe); 1.23 <= Z' <= 2.9 => linear interpolation to 0.5 (grey zone); Z' < 1.23 => linear ramp to 1.0 (distress).
- Weight: 12 (when available; factor is excluded if no statement is available).
- Rationale: Altman Z'-Score is the most widely cited bankruptcy-prediction metric for private firms. See "Altman Z'-Score Implementation" below for the variant used.

### 6. Legal Form

- Data source: KRS / CEIDG legal form field.
- Normalization: spolka akcyjna (SA) => 0.1; spolka z ograniczona odpowiedzialnoscia (sp. z o.o.) => 0.2; spolka komandytowa => 0.3; jednoosobowa dzialalnosc gospodarcza => 0.5; other / less common forms => 0.4.
- Weight: 3.
- Rationale: Capital-company forms (SA, sp. z o.o.) impose stronger disclosure and governance requirements, which correlates with lower opacity risk. This is a weak signal and is weighted accordingly.

### 7. Time Since Last Financial Statement

- Data source: MF e-Reports.
- Normalization: <12 months => 0.0; 12-24 months => 0.4; 24-36 months => 0.7; >36 months or never filed (when required) => 1.0.
- Weight: 8.
- Rationale: Polish law mandates annual filing for most legal forms. A missing or stale statement signals either administrative neglect or active concealment, both of which correlate with elevated risk.

### 8. Connected Entities

- Data source: KRS, cross-referenced by shared PESEL / NIP of board members or shareholders.
- Normalization: number of currently active entities sharing at least one key person. 0-1 => 0.0; 2-4 => 0.2; 5-9 => 0.5; 10+ => 1.0.
- Weight: 5.
- Rationale: High counts of co-controlled entities are characteristic of group structures (legitimate) and of carousel or shell patterns (illegitimate). The factor is interpreted with care and combined with other signals.

### 9. Address Concentration (Virtual Office Indicator)

- Data source: KRS / CEIDG registered address, aggregated.
- Normalization: number of active entities registered at the exact same address. <5 => 0.0; 5-50 => 0.3; 50-500 => 0.6; >500 => 1.0.
- Weight: 4.
- Rationale: Mass-registration addresses correlate with virtual office services. These are legal but increase the probability of low-substance operations and complicate due diligence.

### 10. Address and Name Changes

- Data source: KRS history.
- Normalization: combined count of address and name changes in the trailing 36 months. 0 => 0.0; 1-2 => 0.3; 3-4 => 0.6; 5+ => 1.0.
- Weight: 5.
- Rationale: Frequent re-domiciling and rebranding is a known evasion pattern.

### 11. Capital to Liabilities Ratio

- Data source: MF e-Reports (latest balance sheet).
- Normalization: equity / total liabilities. Ratio > 1.0 => 0.0; 0.5-1.0 => 0.3; 0.2-0.5 => 0.6; <0.2 => 1.0; negative equity => 1.0.
- Weight: 8 (when available).
- Rationale: Low or negative equity coverage of liabilities is a direct solvency signal. Complements Altman Z'-Score with a simpler ratio that remains interpretable when the full Altman inputs are not all available.

### 12. Court and Enforcement Entries

- Data source: KRS section on enforcement, bankruptcy, and restructuring entries.
- Normalization: presence of any active entry => 1.0; closed entries within the trailing 36 months => 0.5; none => 0.0.
- Weight: 12.
- Rationale: An active enforcement, bankruptcy, or restructuring entry is one of the strongest possible negative signals. It is weighted accordingly.

## Weights Summary

| # | Factor | Weight |
|---|--------|--------|
| 1 | Company age | 10 |
| 2 | Share capital | 6 |
| 3 | Board changes | 7 |
| 4 | VAT whitelist status | 9 |
| 5 | Altman Z'-Score | 12 |
| 6 | Legal form | 3 |
| 7 | Time since last statement | 8 |
| 8 | Connected entities | 5 |
| 9 | Address concentration | 4 |
| 10 | Address/name changes | 5 |
| 11 | Capital/liabilities ratio | 8 |
| 12 | Court/enforcement entries | 12 |

Total weight (full data): 89. The denominator adjusts when factors are excluded due to missing data.

## A-E Rating Thresholds

The Risk Score is bucketed into a letter rating:

| Rating | Risk Score range | Interpretation |
|--------|------------------|----------------|
| A | 0-19 | Very low risk |
| B | 20-39 | Low risk |
| C | 40-59 | Moderate risk |
| D | 60-79 | Elevated risk |
| E | 80-100 | High risk |

## Altman Z'-Score Implementation

Raport360 uses the Altman Z'-Score variant for private non-manufacturing firms, since the majority of Polish SMEs in scope are non-manufacturing. The coefficients are from Altman (2000), which revised the original 1983 private-firm model:

> Altman, E.I. (2000). *Predicting Financial Distress of Companies: Revisiting the Z-Score and Zeta Models*. NYU Stern Working Paper. Available at: https://pages.stern.nyu.edu/~ealtman/Zscores.pdf

```
Z' = 6.56 * X1 + 3.26 * X2 + 6.72 * X3 + 1.05 * X4
```

Where:

- X1 = working capital / total assets
- X2 = retained earnings / total assets
- X3 = EBIT / total assets
- X4 = book value of equity / total liabilities

Interpretation zones:

| Z' | Zone |
|----|------|
| > 2.9 | Safe |
| 1.23 - 2.9 | Grey |
| < 1.23 | Distress |

Note on Polish-market applicability: the original Altman coefficients were derived from US data. The model is widely used for Polish firms in practice but its calibration on the local market is imperfect. Polish researchers (for example, Maczynska, Hadasik) have proposed locally calibrated variants. Raport360 currently reports the standard Altman Z'-Score and treats it as one signal among twelve rather than as a standalone verdict. A Poland-calibrated variant is under evaluation.

## Validation

The current scoring is a rule-based heuristic. The weights and thresholds reflect expert judgment informed by published research and practitioner heuristics, not a fitted statistical model.

Empirical validation against historical bankruptcy and restructuring filings in KRS is in progress. The validation plan covers:

- Backtesting Risk Score against the set of companies that entered formal insolvency proceedings in the trailing 36 months.
- Computing area under the ROC curve (AUC) for the Risk Score as a binary predictor of insolvency within 12 months.
- Per-factor lift analysis to identify factors that contribute little or negatively.
- Recalibration of weights once a sufficient labeled set is available.

Until validation completes, Risk Score and Rating should be treated as a structured aggregation of public signals, not as a predictive probability of insolvency.

## Known Limitations

- Coverage gap for sole proprietorships. CEIDG entries lack the financial-statement signal entirely; their scores rely on a reduced factor set and are inherently less informative than scores for KRS companies.
- Lagging data. Even with daily refresh, the underlying registers themselves lag real-world events by days or weeks (court entries are filed after the event they record).
- Address concentration false positives. Legitimate co-working hubs and serviced offices register many tenants at one address; the address-concentration factor cannot distinguish these from shell-hosting addresses by itself.
- Connected-entities false positives. Holding-company structures legitimately share board members across many subsidiaries; the connected-entities factor is a signal, not a verdict.
- No qualitative inputs. The model does not ingest news, court rulings outside KRS, sanctions lists, or media reports.
- Heuristic weights. Until backtested validation completes, weights are based on expert judgment and may be miscalibrated for specific industries or company sizes.

Users are expected to read the per-factor breakdown alongside the headline score rather than relying on the rating alone.

## References

- Altman, E.I. (1983). *Corporate Financial Distress: A Complete Guide to Predicting, Avoiding, and Dealing with Bankruptcy*. New York: Wiley Interscience.
- Altman, E.I. (2000). *Predicting Financial Distress of Companies: Revisiting the Z-Score and Zeta Models*. NYU Stern Working Paper. https://pages.stern.nyu.edu/~ealtman/Zscores.pdf
- Maczynska, E. (1994). Ocena kondycji przedsiebiorstwa (z zastosowaniem uproszczonych metod analizy). *Zycie Gospodarcze*, 38. [Polish Z-Score adaptation widely cited in Polish insolvency literature]
- Hadasik, D. (1998). Upadlosc przedsiebiorstw w Polsce i metody jej prognozowania. Poznan: Akademia Ekonomiczna. [Polish-calibrated discriminant model based on Warsaw Stock Exchange data]
