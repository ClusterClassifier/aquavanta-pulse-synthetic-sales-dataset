# AquaVanta Pulse 750 Synthetic Sales Dataset

![Python](https://img.shields.io/badge/Python-3.13%2B-3776AB?logo=python&logoColor=white)
![Data](https://img.shields.io/badge/Data-Synthetic-35A7A0)
![License](https://img.shields.io/badge/License-CC0-green)

A synthetic dataset for studying omnichannel sales, demand, pricing, marketing, inventory, fulfilment, returns, and customer feedback for one fictional product.

> **Synthetic-data notice:** AquaVanta Pulse 750 is a fictional smart water bottle. Every transaction, event, operational record, and business relationship in this project is simulated. The dataset does not represent an actual company or observed consumer behaviour.

## Dataset overview

The dataset follows sales of the fictional **AquaVanta Pulse 750**, a ₹2,499 insulated smart bottle with an LED temperature display and hydration reminder. It covers eight Indian metropolitan markets, three sales channels, and four colour variants from January 2023 through December 2025.

| Property | Value |
|---|---:|
| Main table rows | 105,216 |
| Main table columns | 58 |
| Date range | 2023-01-01 to 2025-12-31 |
| Observation grain | Date × city × channel × SKU |
| Markets | 8 |
| Sales channels | 3 |
| Colour variants | 4 |
| Fulfilled units | 519,136 |
| Fixed generation seed | 750 |

The complete data package is intended for Kaggle, while this repository contains the generator, documentation, preview data, and validation figures.

## Why this dataset exists

Real retail and supply-chain data is difficult to share because it can expose customers, commercial strategy, pricing, suppliers, and internal operations. This project offers a transparent alternative for education and experimentation while keeping the assumptions visible in the generator.

It supports:

- Sales and latent-demand forecasting
- Stockout-aware demand estimation
- Price-elasticity and promotion analysis
- Return and cancellation prediction
- Inventory and replenishment analysis
- SQL, dashboard, and business-intelligence projects
- Reinforcement-learning inventory experiments

## Simulated relationships

The generator follows a causal sequence:

```text
Calendar + weather + price + campaigns
                    ↓
          Traffic and shopping funnel
                    ↓
             Potential demand
                    ↓
       Cancellations + inventory limits
                    ↓
              Fulfilled sales
                    ↓
       Delayed delivery, returns, reviews
```

Important patterns include summer demand growth, festival promotions, weekend and payday effects, advertising saturation, price competition, channel-specific margins, stockout censoring, delayed returns, and reputation recovery after a quality incident.

The data also contains controlled events such as an influencer campaign, heatwave, supplier delay, battery-cap defect, product recall, courier strike, marketplace outage, and competitor launch.

## Generated files

Running [data_generator.ipynb](data_generator.ipynb) creates the following package:

| File | Purpose |
|---|---|
| `clean/model_ready_daily.csv` | Flattened table for machine learning and analysis |
| `clean/sales_daily.csv` | Sales, pricing, funnel, revenue, and outcomes |
| `clean/inventory_daily.csv` | Stock, replenishment, lead time, and stockouts |
| `clean/marketing_daily.csv` | City-channel advertising and funnel totals |
| `clean/weather_calendar.csv` | City-level weather, holidays, and festivals |
| `clean/returns_reviews.csv` | Delayed returns, ratings, and review activity |
| `clean/market_events.csv` | Catalogue of injected events |
| `raw/model_ready_daily_raw.csv` | Deliberately messy edition for data-cleaning practice |
| `metadata/data_dictionary.csv` | Column definitions, types, categories, and units |
| `metadata/generation_summary.json` | Seed, dimensions, checksum, and validation results |

## Quick start

```bash
git clone https://github.com/<ClusterClassifier>/aquavanta-pulse-synthetic-sales-dataset.git
cd aquavanta-pulse-synthetic-sales-dataset

python -m venv .venv
```

Activate the environment:

```bash
# Linux or macOS
source .venv/bin/activate

# Windows PowerShell
.venv\Scripts\Activate.ps1
```

Install the dependencies and open the generator:

```bash
pip install -r requirements.txt
jupyter notebook data_generator.ipynb
```

Run all notebook cells in order. The generated package will appear in `aquavanta_pulse_dataset/`.

## Repository structure

```text
.
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── requirements.txt
├── data_generator.ipynb
├── data/
│   └── sample_model_ready_daily.csv
├── docs/
│   ├── methodology.md
│   ├── data_dictionary.md
│   └── kaggle_upload_guide.md
└── figures/
```

## Validation

The generator performs automated checks for:

- Expected row count and complete date coverage
- Unique date-city-channel-SKU observations
- Monotonic shopping funnel counts
- Non-negative demand, sales, and inventory
- Sales not exceeding fulfilment demand
- Daily inventory conservation
- Valid discount and rating ranges
- Complete clean identifiers
- Reproducibility through a fixed seed and SHA-256 checksum

The verified release generated 519,136 fulfilled units and passed all built-in integrity checks.

## Example trends

![Monthly AquaVanta sales trend](figures/monthly_sales_trend.png)

Additional figures show normalized temperature effects, promotion-versus-return trade-offs, channel economics, stockout censoring, and delayed returns after the quality event.

## Documentation

- [Methodology](docs/methodology.md)
- [Data dictionary](docs/data_dictionary.md)
- [Kaggle upload guide](docs/kaggle_upload_guide.md)

## Limitations

- The relationships are simulation assumptions, not empirical findings.
- The data should not be used to make commercial predictions about real products or markets.
- Customer-level identities and real personal information are intentionally absent.
- Weather, pricing, campaigns, fulfilment, and feedback are simplified versions of real systems.
- Model performance on this dataset does not establish performance on real retail data.

## Kaggle

The complete dataset is hosted on:

`[Add Kaggle dataset URL]`


## License

- This project is dedicated to the public domain under the [CC0 1.0 Universal](LICENSE) license. You can copy, modify, distribute, and perform the work, even for commercial purposes, all without asking permission.
