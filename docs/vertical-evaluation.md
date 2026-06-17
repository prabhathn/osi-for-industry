# OSI Enterprise Model — Vertical Fit Evaluation

> Evaluated against six representative end-customer archetypes.  
> Model scope: **Core** (party, legal entity, product, address, date, currency) + **HR** + **Finance** + **Sales** + **Marketing** + **Procurement** + Add-ons: **Sites, Inventory, Manufacturing, Projects, Support, Compliance**

---

## Legend

| Symbol | Meaning |
|--------|---------|
| ✅ | Strong fit — entity/metric maps naturally with little or no extension |
| ⚠️ | Partial fit — needs field additions, new status values, or light extension |
| ❌ | Gap — important capability not present in the model at all |

---

## 1. Retailer (e.g., Target, Walmart, Zara)

### Domain Fit Assessment

| Domain | Fit | Notes |
|--------|-----|-------|
| Core (party, product, date, currency) | ⚠️ | Product lacks SKU/UPC/barcode; no price-book entity |
| HR | ✅ | Hourly vs salaried distinction handled via `job.grade`; large shift-based workforce can use existing model |
| Finance | ✅ | GL, cost center, journal entry, budget all map cleanly |
| Sales | ⚠️ | `sales_order` is B2B-centric; retail needs a POS/transaction model (basket, receipt) as the dominant pattern |
| Marketing | ✅ | Campaign, segment, and response model fits promotional/loyalty marketing well |
| Procurement | ✅ | Supplier, PO, and contract model is core to retail buying |
| Sites | ✅ | `site_type` covers store / warehouse / DC natively |
| Inventory | ✅ | Balance, location, and transaction model is a strong fit for store-level stock management |
| Manufacturing | ❌ | Not applicable for pure retail; relevant only for private-label production |
| Support | ⚠️ | `support_case` covers post-purchase issues; no returns/exchange workflow |
| Compliance | ✅ | PCI-DSS, consumer data privacy regulations map to existing control/audit model |

### Key Discrepancies & Missing Needs

**❌ POS / Transaction model**  
The `sales_order → sales_order_line` pattern models B2B orders (quote → order → fulfillment). Retail needs a flat basket/receipt model: `transaction → transaction_line` with a POS terminal identifier, cashier, payment tender, and timestamp. The current order lifecycle (`draft → confirmed → shipped`) does not translate.

**❌ Pricing & promotion engine**  
No price-book, promotional rule, markdown event, or clearance price entity. Retail pricing is dynamic — regular price, promotional price, clearance price, and customer-tier price must all be traceable per SKU per date.

**❌ Loyalty program**  
`customer_tier` is a single field; it cannot represent a loyalty ledger (points earned, points redeemed, tier status). A `loyalty_account`, `loyalty_transaction`, and `tier_qualification_rule` entity are needed.

**❌ Returns / RMA detail**  
`order_status = 'returned'` signals a return but there is no `return_order`, `return_reason`, or refund/exchange tracking entity.

**❌ Replenishment rules**  
The inventory module records current balance and transactions but has no `reorder_rule` (min/max, safety stock, lead-time) entity. Replenishment planning requires this.

**⚠️ Product model gaps for retail**  
`enterprise_core.product` has no `barcode`, `upc`, `sku`, `vendor_item_number`, `size`, `color`, or `style` fields. Retailer catalog management requires these plus a variant model (parent product → SKU variants).

**⚠️ Channel tracking**  
Online vs. in-store vs. mobile-app channel is not represented on sales records. Omnichannel retailers need channel attribution throughout the order and return flow.

**⚠️ Inventory shrinkage**  
`inventory_transaction.txn_type` presumably covers adjustment transactions, but there is no dedicated `shrinkage_event` or cycle-count reconciliation entity to support loss-prevention reporting.

---

## 2. Bank / Financial Institution (e.g., JPMorgan Chase, Deutsche Bank, HSBC)

### Domain Fit Assessment

| Domain | Fit | Notes |
|--------|-----|-------|
| Core (party, legal entity, address) | ⚠️ | Party model is adequate but needs counterparty / beneficial-owner extensions for KYC |
| HR | ✅ | Employee, department, payroll model maps cleanly |
| Finance | ⚠️ | GL/journal entry model is generic but bank-specific sub-ledgers (loan, deposit, trading) are absent |
| Sales | ❌ | B2B sales pipeline is not the primary model; banking "sales" is relationship and product origination |
| Marketing | ⚠️ | Campaign and segment model fits retail banking marketing; wholesale banking has different needs |
| Procurement | ✅ | Internal IT and operations procurement maps adequately |
| Sites | ✅ | Branches, data centres, and offices can be modelled as site types |
| Inventory | ❌ | Physical inventory is irrelevant; securities/collateral inventory is a wholly different domain |
| Manufacturing | ❌ | Not applicable |
| Support | ⚠️ | `support_case` can cover complaints and disputes but lacks regulatory escalation workflow |
| Compliance | ✅ | Best-fit add-on; regulation / control / audit model directly supports Basel, SOX, AML, GDPR |

### Key Discrepancies & Missing Needs

**❌ Core banking data model entirely absent**  
The model has no `account` (demand deposit, savings, term deposit, loan, credit card), `financial_product` (distinct from OSI physical `product`), `account_balance`, or `account_transaction` entity. These are the foundational entities of any banking data model.

**❌ Product model incompatible**  
`enterprise_core.product` is designed for tangible goods. A bank's "products" are financial instruments (home mortgage, business loan, credit card, FX forward). These need `financial_product_type`, `pricing_curve`, `maturity_date`, `interest_rate_type`, and `collateral` fields.

**❌ KYC / AML / Sanctions**  
No `due_diligence_record`, `watchlist_screening`, `beneficial_owner`, `PEP_flag`, or `risk_rating` entity. Financial crime compliance is a regulatory necessity and is entirely outside the current compliance module (which focuses on operational controls).

**❌ Transaction / payment model**  
Wire transfers, ACH payments, SWIFT messages, card transactions, and inter-account transfers require a dedicated `payment_instruction` and `payment_leg` model. The `journal_entry` is an accounting record, not a payment record.

**❌ Risk & capital**  
Credit risk (PD, LGD, EAD), market risk (VaR), and liquidity metrics (LCR, NSFR) are foundational for regulatory reporting (FINREP, COREP, Call Report) and are not modelled.

**⚠️ Party / customer model for relationship banking**  
Banking customers require `relationship_manager` assignment, `client_segment` (retail, private banking, corporate, institutional), `credit_score`, and `KYC_status` fields. The existing `customer_tier` field is insufficient.

**⚠️ Revenue recognition for banking**  
Fee income, net interest income, and trading income are recognised differently from product sales. The current `invoice` + `sales_order_line` revenue path does not apply.

---

## 3. SaaS Company (e.g., Salesforce, Snowflake, Twilio, HubSpot)

### Domain Fit Assessment

| Domain | Fit | Notes |
|--------|-----|-------|
| Core (party, product, date, currency) | ✅ | Strong baseline; product hierarchy works for editions/SKUs |
| HR | ✅ | Standard corporate HR needs are well served |
| Finance | ✅ | GL, invoice, and journal entry model covers SaaS financials; deferred revenue posting needs careful mapping |
| Sales | ✅ | B2B SaaS sales motion (lead → opportunity → order) is exactly what this model was designed for |
| Marketing | ✅ | Demand gen, campaign, and segment model maps cleanly to SaaS GTM |
| Procurement | ✅ | Cloud vendor, infrastructure, and software vendor procurement fits |
| Sites | ⚠️ | Physical offices only; cloud regions/data centres are not physical-site-managed entities in SaaS |
| Inventory | ❌ | No physical inventory |
| Manufacturing | ❌ | Not applicable |
| Support | ✅ | SLA-tiered `support_case` model is a very strong fit for SaaS customer support |
| Compliance | ✅ | SOC 2, GDPR, ISO 27001, SOX controls map directly to the compliance module |

### Key Discrepancies & Missing Needs

**❌ Subscription / entitlement model**  
The most critical gap. There is no `subscription`, `license`, `entitlement`, `renewal`, or `contract_term` entity. SaaS revenue is subscription-driven, not one-time-order-driven. ARR, MRR, churn, expansion, and contraction metrics cannot be derived from the current model.

**❌ Usage metering**  
Usage-based billing (e.g., Snowflake credits, Twilio API calls, AWS compute) requires a `usage_event`, `usage_aggregation`, and `rate_card` model. None of these exist.

**❌ MRR / ARR / churn metrics**  
The model's metrics are order- and pipeline-centric. SaaS-critical metrics — MRR, ARR, Net Revenue Retention (NRR), Gross Revenue Retention (GRR), logo churn, expansion MRR — require a subscription-based data model.

**❌ Customer Success (CS) domain**  
No `csm_assignment`, `health_score`, `renewal_forecast`, `QBR_cadence`, `playbook`, or `at_risk_flag` entity. CS is a primary value driver in SaaS and is entirely unmodelled.

**❌ Product telemetry / adoption**  
Feature usage, DAU/MAU, login events, and product adoption scoring are key inputs to CS and renewal. These require a `product_event` or `adoption_metric` entity.

**⚠️ Partner / channel model**  
Resellers, ISVs, referral partners, and marketplace channels are absent. SaaS companies with channel programs need `partner`, `partner_type`, `deal_registration`, and `partner_commission` entities.

**⚠️ Account hierarchy**  
Enterprise SaaS accounts often have parent companies with multiple subsidiaries each holding their own subscriptions. The current `customer` → `legal_entity` path does not cleanly represent a parent/child account hierarchy.

**⚠️ Deferred revenue**  
Annual subscriptions paid upfront require systematic deferred revenue recognition. While the GL model can hold these entries, there is no `deferred_revenue_schedule` or `revenue_recognition_event` entity to automate ASC 606 waterfall reporting.

---

## 4. Telecom (e.g., AT&T, Verizon, T-Mobile, Deutsche Telekom)

### Domain Fit Assessment

| Domain | Fit | Notes |
|--------|-----|-------|
| Core (party, address, legal entity) | ⚠️ | Party model is usable but needs subscriber-specific extensions |
| HR | ✅ | Large workforce; employee/dept/payroll model applies |
| Finance | ✅ | GL and journal entry model covers telecom financials |
| Sales | ⚠️ | Consumer sales (plan activations, handset bundles) differ significantly from the B2B model |
| Marketing | ✅ | Campaign and segment model fits mass-market telecom marketing |
| Procurement | ✅ | Network equipment, handset, and IT procurement fits well |
| Sites | ✅ | Cell towers, data centres, retail stores, and exchanges all map to `site_type` |
| Inventory | ⚠️ | Handset/SIM stock management fits; network equipment lifecycle does not |
| Manufacturing | ❌ | Not applicable |
| Support | ⚠️ | `support_case` covers customer-facing issues; network operations trouble tickets are a different system |
| Compliance | ✅ | FCC, GDPR, CPNI, and lawful intercept regulations map to the compliance module |

### Key Discrepancies & Missing Needs

**❌ Subscriber / service instance model**  
The core telecom entity is the subscriber (a party holding one or more service lines). A `subscriber_line`, `msisdn`, `imsi`, `iccid`, and `device` (IMEI) model is entirely absent. The `customer` entity is too thin.

**❌ Usage / CDR model**  
Call detail records (CDR), data session records, and SMS/MMS events — the raw material of telecom billing — are not modelled. A `usage_event` entity with duration, volume, rate zone, and termination type is needed.

**❌ Billing & rating**  
Telecom billing is among the most complex in any industry: recurring monthly charges, per-minute/per-MB overage rates, roaming charges, add-on bundles, pro-ration, and taxes. The current `invoice` entity is far too generic. A `billing_cycle`, `charge`, and `rate_plan` model is required.

**❌ Network inventory**  
Physical network assets — base stations, routers, cables, circuits, trunks — form the backbone of telecom operations but are entirely absent. The `sites` add-on covers locations but not the network equipment at those locations.

**❌ Service provisioning lifecycle**  
Activation, suspension, port-in/port-out, plan change, and device upgrade are core telecom service events. These require a `service_order` and `service_lifecycle_event` entity distinct from `sales_order`.

**❌ Interconnect / roaming settlements**  
Inter-carrier settlements, roaming hub agreements, and MVNO billing require `interconnect_agreement` and `settlement_record` entities.

**⚠️ Number portability**  
Porting a subscriber's number between carriers involves regulatory timing windows and a distinct workflow not covered by any current entity.

**⚠️ Consumer vs enterprise segmentation**  
The model assumes a single `customer` model. Telecom has distinct data architectures for consumer (mass market), SMB, and enterprise segments, each with different billing, contract, and provisioning needs.

---

## 5. Industrial Manufacturer (e.g., GE, Caterpillar, Siemens, Honeywell)

### Domain Fit Assessment

| Domain | Fit | Notes |
|--------|-----|-------|
| Core (party, product, legal entity) | ✅ | Strong baseline; product category hierarchy maps to part/assembly hierarchy |
| HR | ✅ | Factory, engineering, and office workforce all served |
| Finance | ✅ | Cost accounting (standard vs actual cost), CapEx tracking, and budget all fit |
| Sales | ✅ | B2B order management for industrial products is a strong fit |
| Marketing | ⚠️ | Account-based marketing (ABM) more relevant than campaign/response model |
| Procurement | ✅ | Critical and strong fit; direct and indirect materials, strategic sourcing |
| Sites | ✅ | Plants, warehouses, and service depots map cleanly to `site_type` |
| Inventory | ✅ | Raw material, WIP, and finished goods inventory all fit the balance/transaction model |
| Manufacturing | ✅ | BOM and work centre model is a solid foundation; gaps noted below |
| Projects | ✅ | Capital projects and large customer project delivery fit well |
| Support | ⚠️ | `support_case` fits customer issue tracking; field service and warranty are absent |
| Compliance | ✅ | ISO 9001, OSHA, environmental, and export controls map to the compliance module |

### Key Discrepancies & Missing Needs

**❌ Production / work order**  
The manufacturing module has BOM and work centre but there is no `production_order`, `operation_step`, `routing`, or `actual_vs_planned_quantity` entity. Without this, manufacturing execution and production variance reporting are impossible.

**❌ Quality management**  
No `inspection_plan`, `inspection_result`, `non_conformance`, `defect`, or `corrective_action` (CAPA) entity. Quality data is central to ISO 9001 compliance and customer delivery performance.

**❌ Asset / equipment management (EAM)**  
Plant machinery, tooling, and production equipment require `asset`, `asset_class`, `maintenance_work_order`, `preventive_maintenance_schedule`, and `failure_event` entities. This is a full MRO/EAM gap.

**❌ Batch / serial traceability**  
The `product` entity has no `batch_number`, `serial_number`, `lot`, or `expiry_date` field. Traceability for recalls, warranty, and regulatory reporting (e.g., aerospace, automotive) requires a `product_instance` or `batch` entity.

**❌ Engineering change management**  
Engineering change orders (ECO) and change requests (ECR) that update BOMs and routings are standard in manufacturing but not modelled.

**⚠️ Subcontracting**  
Outsourced manufacturing steps (send-to-vendor for plating, painting, etc.) require a `subcontract_operation` entity linked to the production routing and procurement.

**⚠️ Demand planning & forecasting**  
No `demand_forecast`, `master_production_schedule` (MPS), or `material_requirements_plan` (MRP) entity. Manufacturers plan production from forecasts, not just from confirmed sales orders.

**⚠️ Dangerous goods / hazmat**  
Chemicals, composites, and certain industrial materials require MSDS linkage, UN hazmat classification, and handling restriction attributes on the `product` entity.

---

## 6. Hotel / Hospitality (e.g., Marriott, Hilton, IHG)

### Domain Fit Assessment

| Domain | Fit | Notes |
|--------|-----|-------|
| Core (party, address, legal entity) | ✅ | Guest-as-party model works; needs profile extension |
| HR | ✅ | Large hourly and management workforce; shift-based scheduling is the main gap |
| Finance | ✅ | Revenue centre accounting (rooms, F&B, spa) maps to GL/cost centre model |
| Sales | ⚠️ | Group/MICE sales have an opportunity → contract flow that partially fits |
| Marketing | ✅ | Loyalty marketing, campaign, and segment model fits brand marketing |
| Procurement | ✅ | F&B suppliers, linen, amenities, and capital procurement all fit |
| Sites | ✅ | Hotel properties map cleanly; `site_type` can distinguish hotel, resort, and extended-stay |
| Inventory | ⚠️ | F&B and amenity inventory partially fits; room availability is not a stock-balance concept |
| Manufacturing | ❌ | Not applicable |
| Projects | ✅ | Hotel renovation and new property development map to project/task model |
| Support | ⚠️ | `support_case` can handle guest complaints; housekeeping workflow is absent |
| Compliance | ✅ | Fire safety, ADA, PCI-DSS, and data privacy map to compliance module |

### Key Discrepancies & Missing Needs

**❌ Reservation / PMS core entities**  
The entire core of hotel operations is absent: `room_type`, `room`, `rate_plan`, `rate_restriction`, `reservation`, `stay_record`, and `folio`. Without these, occupancy, ADR, and RevPAR — the three fundamental hotel KPIs — cannot be computed.

**❌ Revenue management**  
Dynamic rate setting (BAR, length-of-stay restrictions, overbooking logic) requires `rate_plan`, `availability_snapshot`, `rate_bucket`, and `restriction` entities. These are entirely absent.

**❌ F&B operations**  
Restaurant and bar operations require `outlet`, `menu_item`, `recipe`, `table`, `cover_count`, and `check` entities. The inventory module covers ingredient stock but not the service/point-of-sale layer.

**❌ Loyalty / rewards programme**  
`customer_tier` captures a tier label but cannot represent a loyalty ledger: points balance, points earned per stay, redemption, elite qualifying nights, and tier requalification. A `loyalty_account` and `loyalty_transaction` entity are needed.

**❌ Channel distribution**  
Reservations arrive from OTAs (Expedia, Booking.com), GDS (Amadeus, Sabre), direct web, call centre, and corporate negotiated rates. Each has different commission rates and contract terms. A `booking_channel`, `channel_agreement`, and `commission_payable` entity are absent.

**❌ Event / MICE**  
Meeting rooms, banquet event orders (BEO), catering, and AV requirements are core revenue streams for full-service hotels. No event space, event booking, or BEO entity exists.

**❌ Housekeeping operations**  
Room status (clean, dirty, inspected, out-of-order), turn schedules, and housekeeper assignments are operational necessities with no model coverage.

**⚠️ Folio / billing complexity**  
The generic `invoice` entity does not handle hotel folio complexity: room charges, incidental posting (minibar, room service, parking), split folios, city-ledger accounts, and master account billing for groups.

**⚠️ Shift / scheduling**  
Hotels operate 24/7 with complex shift patterns. The HR module has no `shift`, `schedule`, or `time_attendance` entity; this is the primary HR gap for hospitality.

---

## Cross-Vertical Summary

| Gap Theme | Affects |
|-----------|---------|
| Subscription / recurring revenue model | SaaS, Telecom |
| Usage / consumption metering | SaaS, Telecom |
| POS / transaction-level sales | Retail |
| Product model extension (SKU, serial, batch, variant) | Retail, Manufacturing |
| Customer / subscriber deep profile | Telecom, Bank, Hotel |
| Loyalty & rewards programme | Retail, Hotel, SaaS (trials) |
| Production / work order execution | Manufacturing |
| Quality management (CAPA, inspection) | Manufacturing |
| Asset / equipment management (EAM) | Manufacturing, Telecom |
| Core banking entities (account, instrument, CDR) | Bank, Telecom |
| KYC / AML / financial crime compliance | Bank |
| Reservation / PMS core (room, stay, folio) | Hotel |
| Revenue management & channel distribution | Hotel, Telecom |
| F&B operations | Hotel |
| Return / RMA workflow | Retail, Manufacturing |
| Demand planning / forecasting | Retail, Manufacturing |
| Shift scheduling / time & attendance | Hotel, Retail (hourly workforce) |
| Partner / channel model | SaaS |
| Customer success / health score | SaaS |
| Deferred revenue / ASC 606 schedule | SaaS, Telecom |

---

## Recommended Additions by Priority

### High — Needed by 3+ Verticals
1. **`subscription` + `entitlement`** entity — SaaS, Telecom, and recurring-service businesses
2. **`usage_event` + `rate_card`** — usage-based metering (SaaS, Telecom)
3. **`product` field extensions** — `barcode`, `batch_number`, `serial_number`, `variant_parent_id`, `uom_list`
4. **`loyalty_account` + `loyalty_transaction`** — Retail, Hotel, SaaS
5. **`return_order`** entity with reason codes — Retail, Manufacturing, any product business

### Medium — Needed by 1–2 Verticals but High Business Value
6. **`production_order` + `routing`** — Manufacturing execution gap
7. **`quality_inspection` + `non_conformance`** — Manufacturing, Pharma, Aerospace
8. **`asset` + `maintenance_work_order`** — EAM for asset-intensive industries
9. **`reservation` + `room_type` + `folio`** — Hotel PMS core
10. **`subscriber_line` + `usage_event` (telecom-flavoured)** — Telecom

### Low — Vertical-Specific
11. **`beneficial_owner` + `kyc_record`** — Banking / FinTech
12. **`network_element` + `circuit`** — Telecom network inventory
13. **`beo` + `event_space`** — Hospitality MICE
14. **`shift` + `schedule`** — Hourly workforce industries
15. **`demand_forecast` + `mps`** — Manufacturing planning
