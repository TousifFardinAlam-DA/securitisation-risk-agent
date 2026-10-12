# Auto-Loan Securitisation Surveillance & IFRS 9 ECL Risk Engine

![Power BI](https://img.shields.io/badge/Power_BI-Desktop-yellow?logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/Language-DAX-blue)
![Domain](https://img.shields.io/badge/Domain-Securitisation%20%7C%20Credit%20Risk-darkgreen)
![Standard](https://img.shields.io/badge/Compliance-IFRS%209%20%2F%20Ind%20AS%20109-purple)

An end-to-end institutional risk monitoring, credit migration, and structured finance cash flow simulator built in Microsoft Power BI. The platform models a ₹318M retail auto-loan portfolio backed by a 5-tier Star Schema.

---

## 📌 Executive Summary & Key Metrics

* **Active Pool Balance:** ₹318M across 500 loans
* **Pool Factor:** 0.5819 (Original pool: ₹546.5M)
* **Weighted Average Coupon (WAC):** 10.95%
* **Weighted Average Maturity (WAM):** 45.1 Months
* **Weighted Average LTV:** 67.54%
* **Delinquency Metrics:** 30+ DPD Rate: 14.11% | Gross NPA Rate: 5.57%
* **Total Expected Credit Loss (ECL):** ₹12M (Provision Coverage Ratio: 3.67%)
* **Monthly Pool Collections:** ₹224M

---

## 🏗 Data Model Architecture (Star Schema)

The core relational engine utilizes a 1-to-many Star Schema to prevent circular dependencies and bi-directional ambiguity:

* **Dimensions:**
  * `Dim_Borrower`: Geographic location (State, Region), credit rating, debt-to-income (DTI).
  * `Dim_Vehicle`: Vehicle segment (Compact SUV, MUV, SUV, Sedan, Hatchback), valuation, LTV.
  * `Dim_Date`: Sequential calendar date hierarchy.
* **Facts & Snapshots:**
  * `Fact_Auto_Loans`: Loan-level performance parameters (EAD, PD, LGD, Delinquency Days).
  * `dpd_snapshot_history`: Historical monthly delinquency migrations and payment statuses.
  * `static_pool_vintage_data`: Vintage cumulative loss tables across Months on Book (MOB).
  * `dynamic_loss_monthly`: Dynamic monthly cash collections and recoveries.

---

## 📊 Dashboard Modules & Analytical Deliverables

### 1. Portfolio Overview
* Executive KPIs: Total Balance, Active Loan Count, WAC, WAM, Pool Factor, and Average LTV.
* Exposure stratifications: State-level concentration map and vehicle type distribution.

### 2. Delinquency Surveillance & Roll Rates
* High-level delinquency cards: ₹252.17M Current Performing, ₹45M 30+ DPD, ₹18M 90+ DPD.
* Credit migration tracking: Multi-period line chart plotting DPD bucket shifts.
* **Month-over-Month Roll Rate Transition Matrix:** Heatmap visual calculating transition probabilities (% of row total) between prior and current DPD states.

### 3. Vintage & Static Pool Loss Curves
* Static pool net cumulative loss rates mapped against loan seasoning (**Months on Book**).
* Visualizes classic default seasoning (S-curves) highlighting loss maturation between months 12 and 30.

### 4. IFRS 9 ECL Provisioning Framework
* Impairment reserves segmented by accounting stages:
  * **Stage 1 (Performing):** ₹3M (12-month ECL, <30 DPD)
  * **Stage 2 (Underperforming):** ₹2M (Lifetime ECL, 30–89 DPD)
  * **Stage 3 (Credit-Impaired):** ₹7M (Lifetime ECL, 90+ DPD / Default)
* Provision Coverage Ratio KPI: 3.67% of total balance.

### 5. Cash Flow Waterfall & Stress Testing Simulator
* **Sequential Priority of Payments:** Allocates monthly collections (₹224M) sequentially:
  * Senior Tranche: 85% priority target (₹190.67M)
  * Mezzanine Tranche: 10% target (₹22.40M)
  * Equity Residual: First-loss piece capturing remaining residual cash.
* **Macro Stress Testing Engine:** Dynamic What-If parameters (`PD Stress Factor` and `LGD Stress Factor`) dynamically shock the pool, recalculating Stressed ECL and demonstrating junior tranche impairment under crisis conditions.

---

## 💻 Core DAX Implementations

```dax
Total Portfolio ECL = 
SUMX (
    Fact_Auto_Loans,
    Fact_Auto_Loans[EAD] * Fact_Auto_Loans[PD_Estimate] * Fact_Auto_Loans[LGD_Estimate]
)

Stressed Net Collections = 
MAX ( 0, [Total Monthly Collections] - [Stressed Total ECL] )

Mezzanine Tranche Payout = 
VAR MezzTarget = [Total Monthly Collections] * 0.10
VAR CashAfterSenior = MAX ( 0, [Stressed Net Collections] - [Senior Tranche Payout] )
RETURN
MIN ( CashAfterSenior, MezzTarget )
