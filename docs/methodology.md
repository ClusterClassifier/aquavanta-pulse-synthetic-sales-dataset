# Dataset Generation Methodology

## 1. Purpose and scope

The AquaVanta Pulse 750 dataset is a rule-based stochastic simulation of an omnichannel retail system. It was created for forecasting, inventory, pricing, marketing, SQL, dashboard, and reinforcement-learning experiments where genuine commercial data is unavailable or unsuitable for public release.

AquaVanta Pulse 750 is a fictional 750 ml insulated smart water bottle with an LED temperature display and hydration reminder. No real company, customer, transaction, campaign, or operational record is represented.

The generator covers 1 January 2023 through 31 December 2025. Its main table contains one observation for every combination of:

- 1,096 calendar dates
- Eight metropolitan markets
- Three sales channels
- Four colour SKUs

This produces 105,216 rows at the grain `date × city × channel × SKU`.

## 2. Reproducibility

The public release uses NumPy random seed `750`. A single seeded random-number generator controls the simulation, while separate child generators isolate inventory and raw-data corruption stages. Running the notebook with the pinned dependency versions in `requirements.txt` should reproduce the same clean dataset.

The generator records the following in `metadata/generation_summary.json`:

- Random seed
- Date range
- Row count
- Number of markets, channels, and variants
- Total fulfilled units
- Validation results
- SHA-256 checksum of `clean/model_ready_daily.csv`

Changing the seed creates a different but structurally comparable edition. Changing generation rules, dependency versions, schema, or rounding behaviour may change the checksum and should be recorded as a new dataset version.

## 3. Market configuration

The simulation uses Delhi NCR, Mumbai, Bengaluru, Hyderabad, Chennai, Kolkata, Pune, and Ahmedabad. Each market has its own demand scale, annual temperature mean, temperature amplitude, rainfall factor, humidity baseline, AQI baseline, and supply lead-time adjustment.

The three channels behave differently:

| Channel | Designed behaviour |
|---|---|
| Brand Website | Stronger margin, direct advertising, moderate volume, and medium delivery time |
| Marketplace | Highest volume, greater discounting, platform fees, and higher return probability |
| Physical Retail | Lower digital traffic, minimal delivery time, and greater sensitivity to rainfall |

The four product variants are Graphite, Ocean Blue, Coral, and Sage. They have different demand factors, advertising allocation shares, and manufacturing costs. Ocean Blue receives a summer preference effect, while Graphite receives a small physical-retail preference effect.

## 4. Calendar and weather generation

The calendar includes day of week, weekend status, payday windows, public holidays, and modelled festival periods. Festival windows are explicitly assigned for Holi, Eid, Diwali, and Christmas because their commercial effects do not all follow fixed Gregorian dates.

Temperature is generated with a city-specific annual cosine wave that peaks around May, plus daily Gaussian variation. Rainfall uses a city- and season-dependent occurrence probability followed by a positively skewed gamma distribution for rainfall amounts. Chennai receives an additional October–November monsoon effect.

Humidity responds to each city's baseline climate, rainfall occurrence, temperature, and random variation. AQI rises during winter, with a stronger penalty in Delhi. A separate May 2024 heatwave adds an extreme but localized demand condition in Delhi and Ahmedabad.

These variables are synthetic environmental drivers. They are not reconstructed historical weather observations.

## 5. Events, pricing, and competition

Controlled events create structural breaks that are useful for anomaly detection and event studies:

| Period | Event | Main effect |
|---|---|---|
| January 2023 | Product launch burst | Decaying traffic and demand increase |
| December 2023 | Marketplace listing outage | One-day marketplace traffic reduction |
| April 2024 | Influencer campaign | D2C traffic, demand, and discount increase in selected cities |
| May 2024 | North-west heatwave | Higher temperatures in Delhi and Ahmedabad |
| June 2024 | Supplier component delay | Seven additional replenishment lead-time days |
| August 2024 | Battery-cap quality issue | Lower ratings and greater delayed returns for Coral and Sage |
| September 2024 | Targeted product recall | Temporary demand reduction for affected variants |
| July 2025 | Western courier strike | Longer delivery times and more cancellations in Mumbai and Pune |
| December 2025 | Competitor product launch | Lower competing price and conversion pressure |
| December 2025 | Counter-promotion | Higher AquaVanta traffic and discounts |

The list price is ₹2,499 during 2023–2024 and ₹2,599 during 2025. The final selling price combines channel discounting, weekend effects, festival promotions, coupons, and campaign-specific discounts. Discounts are rounded and capped at 30%.

Competitor prices follow mild inflation, festival promotions, daily noise, and a December 2025 price shock. `price_gap_pct` measures AquaVanta's selling-price difference relative to the simulated competitor median.

## 6. Reputation, advertising, and shopping funnel

Expected product reputation begins around 4.3 stars with small channel differences. The August 2024 defect reduces expected ratings for Coral and Sage. Reputation recovers gradually rather than immediately. Conversion uses a seven-day lagged rolling reputation value, preventing future reviews from affecting past demand.

Advertising spend is calculated at the city-channel level and allocated across SKUs. It varies with market size, season, festivals, and campaigns. Cost per thousand impressions converts expenditure into impressions, and a stochastic click-through rate produces clicks.

Traffic contains paid visits and organic visits. Physical-retail `product_page_views` should be interpreted as a digital-assisted footfall proxy rather than literal web-page activity.

The funnel is simulated in order:

```text
impressions → clicks → product-page views → add-to-cart actions → potential demand
```

Each stage is bounded by the previous stage. Add-to-cart probability responds to discount, temperature, payday timing, holidays, lagged reputation, and relative price. Checkout responds to discount, weekends, payday timing, rainfall, competition, reputation, and event effects.

`potential_demand` represents customer demand before cancellation and inventory constraints. It is therefore more suitable than fulfilled sales for some forecasting tasks.

## 7. Sequential inventory and fulfilment

Inventory is simulated separately for every city-channel-SKU location in chronological order. Each location starts with approximately 15–21 days of early demand coverage.

The simulator maintains:

- Opening inventory
- Scheduled replenishment arrivals
- New purchase orders
- Supplier lead time
- Closing inventory
- Estimated stockout duration
- Inventory cover days

A slowly adapting exponentially weighted demand forecast determines the reorder point and target inventory position. This slow response allows sudden campaigns to create plausible short-term stockouts.

Cancellations occur before fulfilment and become more likely during delivery disruptions or heavy rain. Fulfilled sales follow the constraint:

```text
units_sold = min(available_inventory, potential_demand - cancellations)
```

Lost sales equal fulfilment demand that could not be supplied. Daily inventory must satisfy:

```text
opening_inventory + replenishment_units - units_sold = closing_inventory
```

Inventory cover uses only prior sales through a lagged 14-day average, avoiding look-ahead leakage.

## 8. Delivery, returns, reviews, and economics

Delivery time depends on channel, market lead-time adjustment, rainfall, random operational variation, and the July 2025 courier strike.

Feedback is deliberately delayed:

- Returns are associated with purchases made approximately 14 days earlier.
- Reviews are associated with purchases made approximately 10 days earlier.

Return probability rises with marketplace purchasing, purchase-time discounts, late delivery, and the battery-cap defect. Review averages become less noisy when more reviews are observed. Negative-review share increases when expected ratings fall, delivery becomes late, or a defective batch is involved.

Gross revenue equals current fulfilled sales multiplied by the selling price. Net revenue subtracts refunds recognized on the current day for earlier purchases. Gross margin additionally subtracts product cost, channel fees, advertising expenditure, and return-handling cost. A day can therefore have negative net revenue or margin after a promotion or quality incident.

## 9. Raw data-cleaning edition

The clean table is the ground truth. The raw edition introduces controlled business-data problems:

| Issue | Approximate rate |
|---|---:|
| Missing values in selected fields | 0.8% per selected field |
| Inconsistent channel labels | 0.6% |
| Mixed date formats | 0.3% |
| Inventory reconciliation errors | 0.2% |
| Duplicate source records | 0.15% |

The raw edition should be used for cleaning exercises, not for validating generator constraints before correction.

## 10. Validation

The notebook checks:

- Expected 105,216-row output
- Unique date-city-channel-SKU grain
- Complete date range
- Non-negative sales
- Sales not exceeding fulfilment demand
- Monotonic funnel counts
- Daily inventory conservation
- Discount values between 0% and 30%
- Ratings between one and five
- Complete clean identifiers

The v1.0.0 generation produced 519,136 fulfilled units and passed all built-in checks.

## 11. Modelling guidance

Use chronological splits instead of random row splits:

- Training: January 2023–December 2024
- Validation: January–June 2025
- Test: July–December 2025

When predicting future outcomes, exclude variables unavailable at prediction time. In particular, same-day `return_units`, `return_rate`, `average_rating`, `new_review_count`, `net_revenue`, and `gross_margin` are outcomes rather than valid advance predictors.

For stockout-aware demand forecasting, prefer `potential_demand` as the target or explicitly model censoring. `units_sold` is lower than true fulfilment demand when stock is unavailable.

## 12. Limitations

The generator encodes plausible educational relationships, not empirically estimated consumer behaviour. It simplifies households, individual orders, competitors, logistics networks, suppliers, marketing attribution, taxes, and macroeconomic conditions. Markets are represented through aggregate factors and are not intended for geographic comparison.

Results obtained with this dataset do not establish real-world model performance. The dataset must not be used for commercial, financial, legal, or policy decisions about actual products, companies, or populations.
