# 📈 Finance Analysis Guide: DCF & WACC

This guide provides the necessary formulas and instructions for performing Discounted Cash Flow (DCF) and Weighted Average Cost of Capital (WACC) analysis. These can be directly mapped into Excel functions.

---

## 1. Core Formulas for Excel

### A. WACC (Weighted Average Cost of Capital)
Used as the discount rate for future cash flows.
*   **Formula**: `=(E/V * Re) + (D/V * Rd * (1-T))`
*   **Excel Mapping**:
    *   `E`: Market Value of Equity (Market Cap)
    *   `D`: Total Debt
    *   `V`: Total Value (Equity + Debt)
    *   `Re`: Cost of Equity (CAPM: `RiskFreeRate + Beta * EquityRiskPremium`)
    *   `Rd`: Cost of Debt (Interest Expense / Total Debt)
    *   `T`: Effective Tax Rate

### B. FCF (Free Cash Flow)
*   **Formula**: `Operating Cash Flow - Capital Expenditures`
*   **Excel Mapping**: `='Cash Flow'!B5 - 'Cash Flow'!B10`

### C. Terminal Value (Gordon Growth Method)
*   **Formula**: `FCFn * (1+g) / (WACC - g)`
*   **Excel Mapping**: `=(Final_Year_FCF * (1 + Perpetuity_Growth)) / (WACC - Perpetuity_Growth)`

### D. Enterprise Value (EV)
*   **Formula**: `Sum of PV of FCFs + PV of Terminal Value`
*   **Excel Mapping**: `=NPV(WACC, FCF_Range) + (Terminal_Value / (1 + WACC)^n)`

---

## 2. Practical Examples (As of April 2026)

Based on recent 3-quarter growth trends and a market interest rate of **5.0%**.

| Ticker | Growth (Avg Last 3Q) | WACC (Est.) | Valuation Context |
| :--- | :--- | :--- | :--- |
| **NVDA** | **~262% (YoY)** | 9.8% | AI infrastructure dominance; high growth offset by high valuation. |
| **AXTI** | **~18% (QoQ)** | 11.5% | Compound semiconductor recovery; turnaround story. |
| **IREN** | **~185% (YoY)** | 14.2% | High risk/reward; Bitcoin mining + AI Cloud data centers. |

---

## 3. Excel Instruction Steps

1.  **Input Sheet**: Create a "Inputs" tab for `Risk-Free Rate (5.0%)`, `Beta`, and `Tax Rate`.
2.  **Cash Flow Projection**: Project FCF for the next 5-10 years.
    *   Use `=GROWTH()` or simple percentage increments.
3.  **WACC Calculation**: Use the formula above to find your discount rate.
4.  **Discounting**: Use `=NPV(Rate, Values)` for the projection period.
5.  **Terminal Value**: Calculate at the end of the projection period.
6.  **Sensitivity Analysis**: Use Excel's **Data Table** feature (`Data > What-If Analysis > Data Table`) to see how EV changes with different WACC and Growth assumptions.

---
## 🤖 Automation Tip
You can use `gobi.vault.readFile` in your custom homepage to pull these formulas into a calculator applet in the future!

> [!IMPORTANT] Disclaimer
> This is for educational and personal research purposes only. Not financial advice.
