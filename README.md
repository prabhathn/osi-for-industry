# OSI Enterprise Semantic Models

A comprehensive, open enterprise semantic model library built on the [Open Semantic Interoperability (OSI)](https://github.com/open-semantic-interchange/OSI) specification (v0.1.1). Defines canonical business entities — from core reference data through domain-specific modules to industry verticals — as portable, AI-ready YAML.

## Why

Enterprise data platforms need a shared language. OSI Enterprise provides:

- **Canonical entities** — Party, Product, Currency, Calendar form a universal foundation
- **Composable modules** — pick the domains and add-ons relevant to your business
- **AI-ready metadata** — `ai_context` blocks give LLMs the instructions they need to generate correct queries
- **Portable** — vendor-neutral by default, with dialect/vendor extension points for Snowflake, Databricks, Salesforce, dbt

## Structure

```
models/
├── enterprise_core.yaml          # Party, LegalEntity, Address, Product, Currency, Calendar
├── enterprise_hr.yaml            # Employee, Department, Job, Payroll
├── enterprise_finance.yaml       # GL, Invoice, Budget, Journal
├── enterprise_sales.yaml         # Customer, Lead, Opportunity, Order
├── enterprise_marketing.yaml     # Campaign, Segment, Response
├── enterprise_procurement.yaml   # Supplier, PO, Contract
├── addons/
│   ├── compliance.yaml           # Regulation, Control, Audit, Risk
│   ├── inventory.yaml            # Location, Balance, Transaction
│   ├── manufacturing.yaml        # BOM, Work Center, Work Order
│   ├── subscriptions.yaml        # Plan, Subscription, Entitlement
│   ├── projects.yaml             # Project, Task, Resource
│   ├── sites.yaml                # Site, SiteType
│   ├── support.yaml              # SLA, Case, Interaction
│   ├── loyalty.yaml              # Programme, Account, Transaction
│   └── returns.yaml              # Return Request, Return Line
└── verticals/
    ├── saas/
    │   └── customer_success.yaml # CSM Assignment, Health Score, Renewal Forecast
    ├── telecom/
    │   └── subscriber.yaml       # Service Line, SIM, Device
    ├── banking/
    │   └── accounts.yaml         # Account, Balance, Transaction
    ├── retail/
    │   ├── pos.yaml              # Terminal, Transaction, Payment Tender
    │   └── pricing.yaml          # Price Book, Promotion, Promotion Rule
    ├── manufacturing/
    │   ├── quality.yaml          # Inspection, Defect, Corrective Action
    │   └── production.yaml       # Production Order, Production Step
    └── hospitality/
        └── reservations.yaml     # Room Type, Rate Plan, Reservation, Stay
```

## Architecture

The model follows an OOP-style inheritance pattern:

1. **Core** — `enterprise_core.yaml` defines the abstract `Party` base entity
2. **Domains** — Customer, Employee, Supplier extend Party via `party_id` foreign keys
3. **Add-ons** — cross-industry modules that compose on top of any domain
4. **Verticals** — industry-specific entities that further extend the domain layer

Cross-file joins resolve through consistent source naming (`enterprise.<domain>.<table>`).

## Schema Validation

All model files conform to the OSI JSON Schema (`.osi-schema.json`). Validate with:

```bash
pip install -r requirements.txt
python -c "
import yaml, jsonschema, json

with open('.osi-schema.json') as f:
    schema = json.load(f)

with open('models/enterprise_core.yaml') as f:
    model = yaml.safe_load(f)

jsonschema.validate(model, schema)
print('Valid')
"
```

## Getting Started

1. Clone the repo
2. Pick your modules — start with `enterprise_core.yaml` plus any domains relevant to your use case
3. Map `source:` references to your physical tables (e.g., `enterprise.core.party` → `MY_DB.CORE.PARTY`)
4. Load into your semantic layer tool of choice (Snowflake Cortex Analyst, dbt Semantic Layer, etc.)

## Contributing

Contributions welcome. Please:

- Follow existing naming conventions (`snake_case` entities and fields)
- Include `ai_context` blocks with `instructions` and `synonyms`
- Add foreign keys to Core entities where applicable
- Validate against `.osi-schema.json` before submitting

## License

Apache 2.0 — see [LICENSE](LICENSE).
