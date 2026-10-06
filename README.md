# Store Sales — Data Cleaning, Analysis & Dashboard

## Overview

This project works with a 5,000-row retail transaction dataset (`store_sales.csv`, [source](https://www.kaggle.com/datasets/hassanjameelahmed/store-sales?select=store_sales.csv)), covering customer demographics (`Customer_ID`, `Age`, `Gender`), purchase details (`Category`, `Item_Purchased`, `Amount`, `Season`, `Payment_Method`, `Item_Rating`, `Discount_Applied(%)`), and loyalty signal (`Previous_Purchases`).

The project's main challenge wasn't missing data — it was a **correctness question with no obvious answer upfront**: does `Discount_Applied(%)` need to be factored into revenue calculations, or is `Amount` already the final figure? Getting this wrong in either direction would have meant systematically over- or under-stating revenue across the entire dashboard, so resolving it properly was the central piece of this project.

## Problem statement

1. Understand what `Amount` actually represents — a per-item list price, or the actual transaction value — before using it in any revenue calculation.
2. Determine whether `Discount_Applied(%)` needs to be applied on top of `Amount`, or whether it's a separate descriptive variable.
3. Build a verified Calculations sheet cross-checking category, season, gender, and payment-method breakdowns.
4. Build a dashboard summarizing orders, payment behavior, and demographic patterns.

## The core difficulty: does Amount already include the discount?

### The observation that triggered it

Early in the data, the same item appeared with very different `Amount` values. For example, "Handbag" appeared at R115.50 (18% discount) in one row and R153.31 (13% discount) in another. The natural first guess was that `Amount` is a discounted price derived from one fixed underlying item price, and that backing out the discount would reveal that shared original price.

### Testing the hypothesis

A diagnostic column, `Implied_Original_Price = Amount / (1 - Discount_Applied% / 100)`, was built to test this directly. If the hypothesis were true, every row for the same item should resolve to roughly the same implied original price once its discount was removed.

A PivotTable comparing **Min** and **Max** implied price per item showed the opposite: every item showed a massive spread, not a converging value —

| Item | Min implied price | Max implied price |
|---|---|---|
| Handbag | 21.86 | 261.13 |
| Headphones | 608.87 | 3,992.83 |
| Laptop | 568.52 | 4,148.19 |
| Smart Watch | 594.01 | 3,670.10 |
| Rice Pack | 6.97 | 107.28 |

This disproved the original hypothesis: `Amount` is **not** derived from one shared per-item list price minus a discount. There is no stable "original price" hiding underneath this dataset.

### Resolving it with source documentation, not assumption

Rather than guessing which interpretation to default to, the dataset's Kaggle description was checked directly. It states that `Amount` represents **"the total transaction value."** That settled the question: `Amount` is already the final, actual amount charged — not a pre-discount list price.

### Conclusion and its effect on the rest of the project

- **`Revenue = Amount`**, used directly with no further calculation (`SUM(Amount)`, `SUMIF` by category, etc.).
- The `Implied_Original_Price` helper column was **dropped entirely** once it had served its diagnostic purpose — keeping it in the final sheet would have misleadingly implied it was a real, usable figure.
- `Discount_Applied(%)` was reclassified as an **independent, descriptive variable** (useful for questions like "does discount correlate with previous purchases or item rating?"), not an input for adjusting revenue.

This investigation — forming a hypothesis, testing it against the data rather than assuming it, and resolving the remaining ambiguity against the dataset's own documentation instead of guessing — was the most valuable piece of analytical work in this project, and is the main reason this README exists in this level of detail.

## Verification (Calculations sheet)

A dedicated Calculations sheet was built to independently confirm every number before it reached the dashboard, following the same discipline used in earlier projects:

- **Category breakdown** (Accessories, Beauty, Electronics, Footwear, Groceries, Home, Mens Clothing, Sports, Womens Clothing) with item-level subtotals, reconciling to 5,000 total orders.
- **Season breakdown** (Autumn, Spring, Summer, Winter), each further broken down by category, all reconciling to 5,000.
- **Gender breakdown**: Female 2,504 / Male 2,496 — reconciling to 5,000.
- **Gender × Payment Method**: cross-tabulated by season and overall, consistently reconciling to 5,000 and to the overall Card/Cash on Delivery totals.
- **Payment Method by Season**: Card consistently dominates in every season (roughly 80% of orders), with no season showing meaningfully different payment behavior.
- **KPI totals**: Total Orders (5,000), Average Age (45), Total Card Payment (4,009), Total Cash on Delivery Payment (991) — all independently checked against the pivot breakdowns above rather than trusted at face value.

## Dashboard

Built in Excel with a Season slicer and the following visuals:

- KPI cards: Total Orders, Average Age, Total Card Payment, Total Cash on Delivery Payment
- Payment Method by Season (bar chart)
- Orders by Gender (donut)
- Orders by Gender and Season (clustered bar)
- Payment Method by Gender (bar chart)
- Orders by Category and Season (bar chart)

## Key insights

- **5,000 total orders**, average customer age **45** — an established rather than youth-skewed customer base.
- **Card is the dominant payment method** (4,009 vs. 991 Cash on Delivery, roughly 80/20), and this split holds consistently across all four seasons — season does not meaningfully influence how customers choose to pay.
- **Gender split is nearly even** (2,504 Female / 2,496 Male), with Card preferred by both genders.
- **Order volume is fairly stable across seasons**, with no single season standing out sharply from the others.
- **`Amount` already reflects the final transaction value** — confirmed against source documentation after the data itself disproved the simpler "fixed price minus discount" assumption. Revenue figures throughout this project use `Amount` directly.

## Difficulties faced

- The biggest difficulty wasn't a data-quality issue but an **interpretation risk**: the dataset gave no explicit column-level documentation at first glance, and the wrong assumption about `Amount` vs. `Discount_Applied(%)` could have silently produced incorrect revenue figures across the entire project without any error ever appearing in the numbers themselves.
- Resolving it required building a dedicated diagnostic column, testing it against real data, recognizing when the test disproved the initial hypothesis, and then tracking down the dataset's actual source description rather than defaulting to a convenient assumption.

## Caveats and limitations

- `Discount_Applied(%)` should not be used to adjust or recalculate `Amount` anywhere in this dataset — doing so would double-discount revenue that is already net of discount.
- `Item_Purchased` names are generic labels (e.g. "Handbag", "Laptop") that can represent very different actual price points — the dataset does not support deriving a standard price per item.
- This dataset reflects a single snapshot of transactions with no date range specified; seasonal patterns describe the `Season` field only, not a time-series trend.

## Tools used

- Microsoft Excel (PivotTables, PivotCharts, slicers, helper-column hypothesis testing)
- Source documentation review (Kaggle dataset description) to resolve ambiguity the data alone couldn't settle
