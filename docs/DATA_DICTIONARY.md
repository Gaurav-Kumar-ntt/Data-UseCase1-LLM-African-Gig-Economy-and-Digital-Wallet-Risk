# Data Dictionary

**Theme:** African Gig-Economy and Digital Wallet Risk
**Generated:** 2026-07-10 15:51:16
**Date Range:** 2023-01-01 to 2024-12-31
**Currency:** USD

---

## Table of Contents

- [dim_channel](#dim_channel)
- [dim_date](#dim_date)
- [dim_market](#dim_market)
- [dim_worker](#dim_worker)
- [fact_transactions](#fact_transactions)

---

## dim_channel

**Type:** DIMENSION
**Primary Key:** `channel_id`
**Estimated Rows:** 12

### Description

Captures key business metrics and dimensions.

### Columns

| Column Name | Data Type | Nullable | Unique | Description |
|---|---|---|---|---|
| `channel_id` | integer | No | No | Unique channel identifier |
| `channel_type` | string | No | No | USSD app agent card |
| `channel_subtype` | string | Yes | No | Channel subcategory detail |
| `is_digital` | boolean | No | No | Digital vs physical channel |
| `avg_fraud_rate` | float | No | No | Historical fraud rate |
| `requires_internet` | boolean | No | No | Needs internet connection |

### Data Generation

| Column | Generator | Parameters |
|---|---|---|
| `channel_id` | sequence | — |
| `channel_type` | choice | values=['app', 'ussd', 'agent', 'card', 'bank_transfer'], weights=[0.38, 0.28, 0.18, 0.1, 0.06] |
| `channel_subtype` | choice | values=['android_app', 'ios_app', 'web_app', 'ussd_shortcode', 'agent_network', 'card_pos', 'card_online', 'bank_api'], weights=[0.22, 0.1, 0.06, 0.28, 0.18, 0.06, 0.04, 0.06] |
| `is_digital` | choice | values=[True, False], weights=[0.76, 0.24] |
| `avg_fraud_rate` | numpy | distribution=uniform, low=0.02, high=0.15 |
| `requires_internet` | choice | values=[True, False], weights=[0.54, 0.46] |

## dim_date

**Type:** DIMENSION
**Primary Key:** `date_id`
**Estimated Rows:** 730

### Description

Captures key business metrics and dimensions.

### Columns

| Column Name | Data Type | Nullable | Unique | Description |
|---|---|---|---|---|
| `date_id` | integer | No | No | Surrogate date key |
| `full_date` | date | No | No | Calendar date |
| `year` | integer | No | No | Calendar year |
| `month` | integer | No | No | Month number 1-12 |
| `quarter` | integer | No | No | Quarter number 1-4 |
| `day_of_week` | string | No | No | Day name |
| `is_weekend` | boolean | No | No | Weekend flag |
| `is_month_end` | boolean | No | No | Month end flag |
| `week_number` | integer | No | No | ISO week number |

### Data Generation

| Column | Generator | Parameters |
|---|---|---|
| `date_id` | sequence | — |
| `full_date` | date_range | — |
| `year` | derived | — |
| `month` | derived | — |
| `quarter` | derived | — |
| `day_of_week` | derived | — |
| `is_weekend` | derived | — |
| `is_month_end` | derived | — |
| `week_number` | derived | — |

## dim_market

**Type:** DIMENSION
**Primary Key:** `market_id`
**Estimated Rows:** 4

### Description

Captures key business metrics and dimensions.

### Columns

| Column Name | Data Type | Nullable | Unique | Description |
|---|---|---|---|---|
| `market_id` | integer | No | No | Unique market identifier |
| `country` | string | No | No | Country name |
| `iso_code` | string | No | No | ISO country code |
| `local_currency` | string | No | No | Local currency code |
| `dominant_channel` | string | No | No | Most used payment channel |
| `regulatory_tier` | string | No | No | Regulatory complexity tier |
| `usd_fx_rate` | float | No | No | USD exchange rate |
| `market_fraud_index` | float | No | No | Relative fraud exposure index |

### Data Generation

| Column | Generator | Parameters |
|---|---|---|
| `market_id` | sequence | — |
| `country` | choice | values=['Nigeria', 'Kenya', 'Ghana', 'South Africa'] |
| `iso_code` | choice | values=['NG', 'KE', 'GH', 'ZA'] |
| `local_currency` | choice | values=['NGN', 'KES', 'GHS', 'ZAR'] |
| `dominant_channel` | choice | values=['agent', 'ussd', 'app', 'card'] |
| `regulatory_tier` | choice | values=['high', 'medium', 'medium', 'high'] |
| `usd_fx_rate` | numpy | distribution=uniform, low=1.0, high=1600.0 |
| `market_fraud_index` | numpy | distribution=uniform, low=0.5, high=2.0 |

## dim_worker

**Type:** DIMENSION
**Primary Key:** `worker_id`
**Estimated Rows:** 5,000

### Description

Captures key business metrics and dimensions.

### Columns

| Column Name | Data Type | Nullable | Unique | Description |
|---|---|---|---|---|
| `worker_id` | integer | No | No | Unique worker identifier |
| `worker_name` | string | No | No | Full name |
| `gig_segment` | string | No | No | Worker gig category |
| `account_tenure_days` | integer | No | No | Days since account opened |
| `kyc_tier` | string | No | No | KYC verification level |
| `gender` | string | Yes | No | Worker gender |
| `age_band` | string | No | No | Age group band |
| `risk_score` | float | No | No | Account risk score 0-100 |
| `is_active` | boolean | No | No | Active account flag |
| `preferred_channel` | string | No | No | Primary channel used |

### Data Generation

| Column | Generator | Parameters |
|---|---|---|
| `worker_id` | sequence | — |
| `worker_name` | faker | method=name |
| `gig_segment` | choice | values=['ride_hailing_driver', 'delivery_rider', 'freelance_creative', 'market_trader'], weights=[0.3, 0.28, 0.2, 0.22] |
| `account_tenure_days` | numpy | distribution=exponential, scale=300 |
| `kyc_tier` | choice | values=['tier_1', 'tier_2', 'tier_3'], weights=[0.45, 0.35, 0.2] |
| `gender` | choice | values=['Male', 'Female', 'Prefer not to say'], weights=[0.62, 0.34, 0.04] |
| `age_band` | choice | values=['18-24', '25-34', '35-44', '45-54', '55+'], weights=[0.18, 0.38, 0.27, 0.12, 0.05] |
| `risk_score` | numpy | distribution=beta, a=2, b=5 |
| `is_active` | choice | values=[True, False], weights=[0.88, 0.12] |
| `preferred_channel` | choice | values=['app', 'ussd', 'agent'], weights=[0.48, 0.32, 0.2] |

## fact_transactions

**Type:** FACT
**Primary Key:** `transaction_id`
**Estimated Rows:** 50,000
**Grain:** one row per wallet transaction

### Description

Captures key business metrics and dimensions.

### Columns

| Column Name | Data Type | Nullable | Unique | Description |
|---|---|---|---|---|
| `transaction_id` | string | No | No | Unique transaction identifier |
| `worker_id` | integer | No | No | Gig worker FK |
| `date_id` | integer | No | No | Date dimension FK |
| `channel_id` | integer | No | No | Channel dimension FK |
| `market_id` | integer | No | No | Market dimension FK |
| `transaction_type` | string | No | No | Cash-in out transfer payment |
| `amount_local` | float | No | No | Transaction amount local currency |
| `amount_usd` | float | No | No | Amount in USD |
| `transaction_outcome` | string | No | No | Success failed reversed pending |
| `is_fraud_flagged` | boolean | No | No | Fraud flag indicator |
| `is_disputed` | boolean | No | No | Dispute raised flag |
| `is_reversed` | boolean | No | No | Transaction reversed flag |
| `velocity_score` | float | Yes | No | Transactions per hour score |
| `fraud_loss_usd` | float | Yes | No | Loss amount if fraud |
| `processing_time_ms` | integer | Yes | No | Processing latency milliseconds |

### Data Generation

| Column | Generator | Parameters |
|---|---|---|
| `transaction_id` | uuid | — |
| `worker_id` | foreign_key | — |
| `date_id` | foreign_key | — |
| `channel_id` | foreign_key | — |
| `market_id` | foreign_key | — |
| `transaction_type` | choice | values=['cash_in', 'cash_out', 'peer_transfer', 'merchant_payment', 'bill_payment'], weights=[0.22, 0.2, 0.25, 0.2, 0.13] |
| `amount_local` | numpy | distribution=lognormal, mean=5.2, sigma=1.4 |
| `amount_usd` | numpy | distribution=lognormal, mean=3.8, sigma=1.4 |
| `transaction_outcome` | choice | values=['success', 'failed', 'reversed', 'pending'], weights=[0.78, 0.1, 0.08, 0.04] |
| `is_fraud_flagged` | choice | values=[True, False], weights=[0.07, 0.93] |
| `is_disputed` | choice | values=[True, False], weights=[0.05, 0.95] |
| `is_reversed` | choice | values=[True, False], weights=[0.08, 0.92] |
| `velocity_score` | numpy | distribution=uniform, low=0.0, high=10.0 |
| `fraud_loss_usd` | numpy | distribution=lognormal, mean=2.5, sigma=1.2 |
| `processing_time_ms` | numpy | distribution=normal, mean=850, sigma=300 |

---

## Relationships

### Foreign Key Constraints

| From Table | From Column | To Table | To Column | Type |
|---|---|---|---|---|
| fact_transactions | `worker_id` | dim_worker | `worker_id` | Many To One |
| fact_transactions | `date_id` | dim_date | `date_id` | Many To One |
| fact_transactions | `channel_id` | dim_channel | `channel_id` | Many To One |
| fact_transactions | `market_id` | dim_market | `market_id` | Many To One |

### Relationship Descriptions

- **fact_transactions** has many to one **dim_worker**: `fact_transactions.worker_id` references `dim_worker.worker_id`
- **fact_transactions** has many to one **dim_date**: `fact_transactions.date_id` references `dim_date.date_id`
- **fact_transactions** has many to one **dim_channel**: `fact_transactions.channel_id` references `dim_channel.channel_id`
- **fact_transactions** has many to one **dim_market**: `fact_transactions.market_id` references `dim_market.market_id`

---

## Data Quality & Characteristics

### Missing Values
Columns marked as 'Nullable: Yes' may contain null values representing missing data.

### Unique Constraints
Columns marked as 'Unique: Yes' have no duplicate values (excluding nulls).

### Primary Keys
Primary keys are unique identifiers with no null values.

### Temporal Coverage
All dates fall within: 2023-01-01 to 2024-12-31

---

*Dictionary generated on 2026-07-10 15:51:16*