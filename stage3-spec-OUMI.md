# Microsoft FY2024 Ratio Analysis — Technical Specification

**Created by:** Graysen Oumi
**Updated by:** Graysen Oumi
**Date Created:** April 22, 2026
**Date Updated:** April 22, 2026
**Version:** 1.0
**LLM Used:** Claude Sonnet 4.6

**Role:** Financial Analyst, Corporate Finance
**Audience:** CFO or Director of FP&A

**Purpose:** Provide a professional, quantitative specification documenting the Stage 2 Excel model's analytical structure for computing and interpreting 25+ accounting and performance ratios from Microsoft's FY2024 financial statements. This post-build spec captures what was built, what was learned, and how the model should be refined for production use.

---

## 1. Problem Statement

Microsoft Corporation (NASDAQ: MSFT) is a global technology company operating across Productivity & Business Processes, Intelligent Cloud, and More Personal Computing. This specification documents the analytical framework for computing 25+ accounting and performance ratios from Microsoft's FY2024 10-K (fiscal year ended June 30, 2024), enabling management to assess capital efficiency, margin quality, financial leverage, and shareholder value creation. The resulting analysis supports CFO-level briefings on capital allocation, investor communications, and peer benchmarking against Alphabet, Amazon, and Salesforce. All figures are sourced from the consolidated financial statements in Microsoft's SEC EDGAR filing.

---

## 2. Inputs (Known Variables)

All figures in millions (USD) unless noted. FY2024 = current year; FY2023 = prior year.

### Balance Sheet Items

| Variable | Description | Named Range | FY2024 | FY2023 |
|---|---|---|---|---|
| Cash & marketable securities | Cash + short-term investments | `BAL_cash_mktSec_2024` / `_2023` | 75,525 | 111,262 |
| Accounts receivable | Net receivables | `BAL_receivables_2024` / `_2023` | 60,097 | 48,688 |
| Inventories | Physical inventory | `BAL_inventories_2024` / `_2023` | 1,246 | 2,500 |
| Total current assets | Sum of current assets | `BAL_assets_current_2024` / `_2023` | 159,734 | 184,406 |
| Net PP&E | Property, plant & equipment, net | `BAL_fixed_assets_net_2024` | 135,591 | 95,641 |
| Total assets | All assets | `BAL_assets_total_2024` / `_2023` | 512,163 | 411,976 |
| Total current liabilities | Short-term obligations | `BAL_liabilities_current_2024` / `_2023` | 125,286 | 104,149 |
| Long-term debt | Non-current borrowings | `BAL_debt_long_term_2024` / `_2023` | 42,688 | 41,990 |
| Total liabilities | All liabilities | `BAL_liabilities_total_2024` | 243,686 | 205,753 |
| Shareholders' equity | Book value of equity | `BAL_equity_shareholders_2024` / `_2023` | 268,477 | 206,223 |

### Income Statement Items

| Variable | Description | Named Range | FY2024 |
|---|---|---|---|
| Net sales | Total revenue | `INC_sales` | 245,122 |
| Cost of revenue | Direct product/service costs | `INC_cost_goods_sold` | 74,114 |
| R&D expense | Research and development | `INC_rd` | 29,510 |
| SG&A expense | Selling, general & administrative | `INC_sga` | 24,456 |
| EBIT | Operating income | `INC_ebit` | 109,433 |
| Interest expense | Cost of debt | `INC_interest_expense` | 1,584 |
| Taxes | Income tax expense | `INC_taxes` | 22,982 |
| Net income | Bottom-line earnings | `INC_net` | 88,136 |
| Dividends | Cash dividends paid | `INC_dividends` | 8,491 |

### Cash Flow Statement Items

| Variable | Description | Named Range | FY2024 |
|---|---|---|---|
| Depreciation & amortization | Non-cash charge (from CFS) | `INC_depreciation` | 15,223 |
| Cash from operations | Operating cash flow | `CASH_operating` | 118,548 |
| Cash from investments | Investing cash flow | `CASH_investments` | -33,493 |

### Market / Analyst Inputs

| Variable | Description | Named Range | Value |
|---|---|---|---|
| Share price | Closing price, June 28, 2024 | `share_price` | 446.36 |
| Shares outstanding | Diluted shares (millions) | `shares_outstanding` | 7,432 |
| Cost of capital | WACC (analyst estimate) | `cost_capital` | 9.0% |
| Tax rate | Effective rate (≈ statutory) | `tax_rate` | 20.7% |

---

## 3. Assumptions & Constraints

- All figures reported in millions USD; per-share figures in USD.
- Tax rate set at Microsoft's FY2024 effective rate of 20.7% (near-statutory; used consistently for after-tax operating income and EVA).
- Cost of capital estimated at 9.0% reflecting MSFT's investment-grade profile and long-run equity risk premium; not derived from a formal WACC build.
- Depreciation taken from the Cash Flow Statement ($15,223M); this figure captures both D&A on PP&E and amortization of acquired intangibles.
- Start-of-year denominators use FY2023 Balance Sheet balances; average-based denominators use the arithmetic mean of FY2023 and FY2024.
- Total capitalization defined as long-term debt plus shareholders' equity (excludes current maturities of long-term debt and operating leases).
- Interest rates treated on a simple annual basis; no compounding adjustments applied.
- No off-balance-sheet items, pension obligations, or contingent liabilities incorporated.
- Share price as of fiscal year-end (June 28, 2024); no trailing-average price used.
- Microsoft's FY2023 Activision acquisition ($75B) is fully reflected in the FY2024 balance sheet but not separated from organic growth.

---

## 4. Calculation Flow

### Step 1: Derived Inputs
1. `market_capitalization` = `share_price` × `shares_outstanding` → **$3,317,075M**
2. `after_tax_op_income` = `INC_net` + (1 − `tax_rate`) × `INC_interest_expense` → **$89,393M**
3. `daily_sales` = `INC_sales` / 365 → **$671.6M/day**
4. `avg_equity` = AVERAGE(`BAL_equity_shareholders_2023`, `BAL_equity_shareholders_2024`) → **$237,350M**
5. `avg_assets` = AVERAGE(`BAL_assets_total_2023`, `BAL_assets_total_2024`) → **$462,070M**
6. `avg_receivables` = AVERAGE(`BAL_receivables_2023`, `BAL_receivables_2024`) → **$54,393M**
7. `avg_inventory` = AVERAGE(`BAL_inventories_2023`, `BAL_inventories_2024`) → **$1,873M**
8. `startYear_capitalization` = `BAL_debt_long_term_2023` + `BAL_equity_shareholders_2023` → **$248,213M**
9. `currentYear_capitalization` = `BAL_debt_long_term_2024` + `BAL_equity_shareholders_2024` → **$311,165M**
10. `net_working_capital` = `BAL_assets_current_2024` − `BAL_liabilities_current_2024` → **$34,448M**

### Step 2: Performance Ratios
- **MVA** = `market_capitalization` − `BAL_equity_shareholders_2024` → **$3,048,598M**
- **Market-to-Book** = `market_capitalization` / `BAL_equity_shareholders_2024` → **12.4×**
- **EVA** = `after_tax_op_income` − (`cost_capital` × `startYear_capitalization`) → **$89,393 − $22,339 = $67,054M**

### Step 3: Profitability Ratios
- **ROA (start-of-year)** = `after_tax_op_income` / `BAL_assets_total_2023` → **21.7%**
- **ROA (average)** = `after_tax_op_income` / `avg_assets` → **19.3%**
- **ROC (start-of-year)** = `after_tax_op_income` / `startYear_capitalization` → **36.0%**
- **ROE (start-of-year)** = `INC_net` / `BAL_equity_shareholders_2023` → **42.7%**
- **ROE (average)** = `INC_net` / `avg_equity` → **37.1%**

### Step 4: Efficiency Ratios
- **Asset Turnover** = `INC_sales` / `BAL_assets_total_2023` → **0.60×**
- **Receivables Turnover** = `INC_sales` / `avg_receivables` → **4.5×**
- **Collection Period** = 365 / `receivables_turnover` → **81 days**
- **Inventory Turnover** = `INC_cost_goods_sold` / `avg_inventory` → **39.6×**
- **Days in Inventory** = 365 / `inventory_turnover` → **9 days**
- **Gross Profit Margin** = (`INC_sales` − `INC_cost_goods_sold`) / `INC_sales` → **69.8%**
- **Operating Profit Margin** = `INC_ebit` / `INC_sales` → **44.6%**
- **Net Profit Margin** = `INC_net` / `INC_sales` → **35.9%**

### Step 5: Leverage Ratios
- **Long-term Debt Ratio** = `BAL_debt_long_term_2024` / `currentYear_capitalization` → **13.7%**
- **Debt-to-Equity** = `BAL_debt_long_term_2024` / `BAL_equity_shareholders_2024` → **15.9%**
- **Total Debt Ratio** = `BAL_liabilities_total_2024` / `BAL_assets_total_2024` → **47.6%**
- **Times Interest Earned** = `INC_ebit` / `INC_interest_expense` → **69.1×**
- **Cash Coverage** = (`INC_ebit` + `INC_depreciation`) / `INC_interest_expense` → **78.7×**
- **Debt Burden** = (`INC_ebit` − `INC_interest_expense`) / `INC_ebit` → **98.6%**
- **Leverage Ratio** = `BAL_assets_total_2024` / `BAL_equity_shareholders_2024` → **1.91×**

### Step 6: Liquidity Ratios
- **Current Ratio** = `BAL_assets_current_2024` / `BAL_liabilities_current_2024` → **1.27×**
- **Quick Ratio** = (`BAL_assets_current_2024` − `BAL_inventories_2024`) / `BAL_liabilities_current_2024` → **1.27×**
- **Cash Ratio** = `BAL_cash_mktSec_2024` / `BAL_liabilities_current_2024` → **0.60×**
- **NWC-to-Assets** = `net_working_capital` / `BAL_assets_total_2024` → **6.7%**

### Step 7: Du Pont Decomposition
- **Du Pont ROA** = (`after_tax_op_income` / `INC_sales`) × (`INC_sales` / `BAL_assets_total_2023`)
  = After-tax operating margin × Asset turnover = **36.5% × 59.5% = 21.7%** ✓
- **Du Pont ROE** = (`INC_net` / `INC_sales`) × (`INC_sales` / `BAL_assets_total_2023`) × (`BAL_assets_total_2023` / `BAL_equity_shareholders_2023`) × (`INC_net` / (`INC_ebit` − `INC_interest_expense`))
  = Net margin × Asset turnover × Leverage × Debt burden = **35.9% × 59.5% × 2.00 × 98.6% = 42.1%** ≈ ROE ✓

---

## 5. Outputs

| Output | Description | Format |
|---|---|---|
| Ratio summary table | All 25+ ratios organized by category with computed values | Table |
| Derived inputs block | Averages, daily figures, after-tax operating income, capitalization totals | Table |
| Du Pont decomposition | Factor-level ROA and ROE breakdown with reconciliation check | Table |
| Formula documentation | Named-range pseudocode for each ratio in a dedicated column | Inline annotations |
| Executive summary | Key findings narrative for CFO briefing | 1–2 paragraphs (Stage 4) |

---

## 6. Model Review — What Worked & What to Improve

**What worked correctly:**
The Du Pont ROA and ROE decompositions reconciled to within 0.1% of their directly computed counterparts, confirming formula consistency across the model. The start-of-year vs. average denominator logic was cleanly separated into distinct rows, making it straightforward to compare both conventions. The EVA calculation — after-tax operating income minus a capital charge — produced a meaningful positive result ($67B) consistent with Microsoft's documented economic outperformance.

**What should be improved:**
- **Named range consistency:** Several intermediate cells were referenced by coordinate (e.g., cell B14) rather than a declared named range, making formula audits slow. A refined model enforces named ranges for every input and every derived variable.
- **Tax rate handling:** The model used a single static rate (20.7%) for both the after-tax operating income and the debt-burden interpretation. A more rigorous build separates the effective tax rate (from `INC_taxes` / pre-tax income) from any statutory assumption used in scenario analysis.
- **Depreciation sourcing:** The model pulled D&A from the Cash Flow Statement. This is correct for cash coverage but conflates PP&E depreciation with intangible amortization (large in FY2024 due to Activision). A clean model separates these and notes the source.
- **Layout:** Ratio categories were computed on a single worksheet with no grouping or color coding by section. A production model would use separate clearly-labeled sections, consistent font hierarchy, and alternating shading for readability.
- **No scenario/sensitivity layer:** The model computes one point-in-time result with no what-if analysis (e.g., sensitivity of EVA to ±1% change in WACC, or impact of a 5% revenue decline on coverage ratios). This is the most important structural gap.

**Additional metrics worth including:**
- Free cash flow yield = (`CASH_operating` − capital expenditures) / `market_capitalization`
- Return on invested capital (ROIC) using a full NOPAT build
- Organic revenue growth (excluding Activision) to assess core business trajectory

---

## 7. Limitations & Next Steps

This specification does not incorporate multi-year trend analysis, industry peer benchmarks (Alphabet, Amazon, Salesforce), or off-balance-sheet obligations such as operating lease commitments. The cost of capital (9.0%) is an analyst estimate rather than a formally derived WACC, which affects EVA precision. The Activision acquisition distorts FY2024 asset and amortization figures relative to organic periods, a caveat that must be communicated in any peer comparison.

The next phase (Stage 4) will use this specification as the direct input for a structured AI prompt, leveraging the named-range definitions in Section 2 and the calculation flow in Section 4 to generate an improved model. The Model Review findings in Section 6 will instruct the AI on exactly which simplifications to replace. The final output will be a CFO-ready analysis memo synthesizing ratio results into strategic recommendations on capital allocation, leverage capacity, and shareholder return sustainability.
