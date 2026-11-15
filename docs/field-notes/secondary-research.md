# Secondary Desk Research — School Fees Tracker

**Project:** School Fees Tracker — private nursery and primary school in Cameroon (CEF 345, Week 4)
**Prepared:** 2026-11-15
**Method:** **Secondary desk research only** — i.e., published reports, official statistics, reputable press and market commentary, all listed as Sources below with access dates. **This is not field research. No school visit, interview, survey or observation of any kind took place, and nothing in this document may be read as "the school said…" or "parents/bursars reported…".**
**Purpose:** Provide the evidence base for the requirements in `docs/srs.md`. These notes inform context and constraints; they are not a substitute for primary research and do not certify any specific school's actual fee policy.

Where sources disagree, the discrepancy is reported rather than smoothed over. Where a figure comes from a single secondary (non-official) source, it is labelled as such.

---

## 1. Education context (Cameroon)

- Cameroon operates a bilingual (French/English) education system. The levels relevant to this project are **pre-primary (nursery/kindergarten, roughly ages 3–5)** and **primary** (a six-year cycle) [2][4].
- Per World Bank analysis, public schools enroll the majority of primary pupils (about **75–76%**), while **most pre-primary pupils are in the private sector (about 61%)** [1].
- Primary enrollment in private schools is roughly a **quarter** of the total: **~26.86% in 2023** (figures compiled from World Bank WDI, via Aubsp) and **~25.34% in 2024** (CEIC, same WDI series) [2][6].
- Pre-primary gross enrollment ≈ **35.62% (2019)**; primary gross enrollment ≈ **112.64% (2023)** with ≈ **5.16 million primary pupils**; primary completion rate ≈ **71%** — all per Aubsp / World Bank WDI as reported [2][3][4].
- Fee levels (indicative only, in XAF, **not** audited data for any specific school):
  - International private primary, Yaoundé: ~**1,016,855 XAF/year** average; range roughly 300,057–2,000,369 XAF/year (combien-coute.net, 2026, self-reported/estimated) [7].
  - Local private nursery & primary examples in Yaoundé: ~**170,885–228,782 XAF/year** (Grace Dara Nursery and Primary School) and ~**187,233–240,601 XAF/year** (Awae Bilingual Nursery and Primary School), per directory listings [8][9].
  - Derived takeaway (an interpretation, not a sourced figure): local private nursery/primary fees run to a few hundred thousand XAF per child per year, often paid in instalments. **Actual fees and instalment rules for the target school are unknown and must be configurable data, not hard-coded constants.**

**Implication for the SRS:** the system must model XAF amounts, academic years, per-class fee structures, instalments and per-student balances — not a single flat fee.

## 2. Currency

- The currency is the **Central African CFA franc (XAF)**, issued for the CEMAC zone under the **BEAC** (Bank of Central African States) framework [2][10].
- XAF is used without a minor subunit in practice (amounts are whole francs). Monetary values in the system should be stored and displayed as whole XAF amounts.

## 3. Payment landscape in Cameroon

### 3.1 Mobile money is mainstream (official anchor source)

According to the **2024 Annual Observatory of the Electronic Communications Market** (as reported by Business in Cameroon, 2025-12-10) [5]:

- Estimated transaction volumes (measured via SMS sent) grew from **1,237,402,354 (2023)** to **1,822,959,071 (2024)**, a **+47.3%** increase.
- **MTN Mobile Money Corporation** revenue ≈ **CFA 70.2bn (2024)**, up from CFA 55.1bn (2023), **+27%**.
- **Orange Money Cameroon** revenue ≈ **CFA 65bn (2024)**, versus CFA 44.5bn (2022) (no 2023 comparator published).
- MTN + Orange counted **11.429 million active mobile money subscriptions** in 2024 (+772,141 vs 2023); **38.84%** of all subscribers across both networks hold an e-money account.
- Total mobile money sector revenue ≈ **CFA 135.84bn (2024)**, +146.4% year-on-year (partly because Orange Money revenue entered the data set for the first time).

### 3.2 Market structure — two operators dominate (sources disagree on shares)

- National statistics cited in 2019 put the segment at **MTN ~40.7%, Orange ~40.3%**, Afrikpay ~8.5% [11].
- Recent market commentary (2026, **non-official blog sources**) puts Orange Money at ~52% (≈6.5–6.85M active users) and MTN MoMo at ~45% (≈5.8–5.94M active users), with ~43% adult penetration and ~67,000 agent points [12][13].
- CamPay's blog claims roughly **60% MTN / 40% Orange** [14].
- **Consistent finding across all sources: MTN MoMo and Orange Money together dominate Cameroon's mobile money market.** New entrants (e.g., Wave, Blue Money) are expected, per Business in Cameroon [5].
- **Implication:** any online payment integration for school fees should cover **both** MTN MoMo and Orange Money; supporting only one excludes a large share of payers.

### 3.3 Regulatory and integration context

- Cameroon follows the **CEMAC** framework, administered by **BEAC** and supervised by **COBAC** (Banking Commission). Mobile money services require coordination with telecommunications regulators and specific **BEAC authorization** [10][15].
- An entity that merely **integrates** MTN/Orange APIs (or a licensed aggregator) generally does **not** need its own EMI (Electronic Money Institution) licence; EMI licensing applies to issuers of e-money/wallets [10][16]. School fee collection should therefore ride on licensed operators' rails (direct APIs or an aggregator) rather than attempt to hold payer funds. This is secondary-source legal commentary, not legal advice — confirm with the payment providers before production launch.
- Online e-payments outside CEMAC are limited to **XAF 1,000,000 per user per month** absent written authorization (Legal 500 fintech guide, 2026) [15]. Note this is outbound-outside-CEMAC; fee payments by domestic payers are domestic, but integrators should confirm operator-side per-transaction and per-day limits (one 2026 blog source lists ~2,000,000 FCFA/day per operator, unverified) [13].
- AML/compliance: PSPs must monitor transactions and report suspicious activity to **CENTIF**; merchants are expected to maintain transparent transaction records [10]. Data protection falls under **CNDP** (national data protection authority) [10].
- **Implication:** the SRS should require auditable payment records, role-restricted access, and retention of transaction references/receipts — consistent with AML transparency expectations — without holding payer funds itself.

### 3.4 Integration options observed

- Direct operator APIs (MTN MoMo, Orange Money) or licensed aggregators such as CamPay that unify both operators are how Cameroonian merchants typically integrate [14][16]. Exact API terms are commercial and out of scope for this SRS; integration requirements should be expressed as "licensed MTN MoMo / Orange Money rails (direct or via an aggregator)" with an idempotent, callback/reconciliation-based confirmation flow.

## 4. Derived design constraints (desk-research-based)

1. Currency: **XAF**, whole francs; no decimal handling.
2. Payment channels: **cash**, **bank transfer**, and **Mobile Money** recorded manually by the Bursar, **plus** payer-initiated online payments via **MTN MoMo / Orange Money**.
3. Two flows must both reconcile to one student balance: staff-recorded (manual) and gateway-confirmed (online). Online payments need transaction references and reconciliation, not just a "paid" flag.
4. Private (fee-paying) nursery/primary schools are a real, sizable segment (~a quarter of primary enrollment; most pre-primary) [1][2][6], and local fee levels suggest per-child annual totals of a few hundred thousand XAF [7][8][9], often paid in instalments — hence FR-04's "full or partial payment" and FR-05's running balance.
5. Two staff roles only (**Administrator**, **Bursar**); the **Payer** (parent/guardian) is an external actor for online payments with **no staff account**.
6. Record-keeping and access control should anticipate AML/data-protection expectations (CENTIF/CNDP context): audit trails, least-privilege roles, receipt traceability [10].
7. Fee structures, terms, instalment plans, and any waiver/discount rules vary by school and change year to year — they must be **configurable data**, never hard-coded constants.
8. Public primary is described in policy as free, yet private primary remains common [1][2] — do not assume the target school is free or subsidised; confirm its fee policy directly (primary research) before production configuration.

## 5. Limitations

- **No school visit, interview, survey or observation took place. No primary data of any kind was collected for this document.**
- Mobile money market-share figures conflict across sources (~40/40 official 2019 data vs ~52/45 or ~60/40 recent commentary) [5][11][12][13][14]; the only verified anchor used is the BEAC-observatory-derived 2024 data via Business in Cameroon [5][11].
- Fee figures are indicative directory listings/estimates, not audited or school-verified data, and drift over time; they are used only to size the problem, not to set requirements.
- Regulatory summaries (BEAC/COBAC/CEMAC, EMI vs PSP, outbound monthly cap) come from secondary legal/fintech commentary and must be confirmed with the school's payment providers and legal counsel before production launch.
- The target school's actual academic calendar, fee policy, instalment schedule, and current record-keeping practice are **unknown** (no fieldwork). These are treated as configurable parameters or open questions in the SRS, never as established fact.
- Language coverage (English and/or French UI) could not be established from secondary sources and is raised as an open question in the SRS.

## 6. Sources

All accessed 2026-11-15.

1. World Bank, *Project Appraisal Document — Cameroon Education Sector Support Project (PAD2366)*, 2018. https://documents1.worldbank.org/curated/en/991921525399281987/pdf/CAMEROON-PAD-03232018.pdf
2. Aubsp, *Age-Wise Education System in Cameroon (2026 Guide for Parents & Students)*. https://www.aubsp.com/age-wise-education-system-in-cameroon/
3. TradingEconomics, *Cameroon — School Enrollment, Preprimary (% Gross)* (World Bank WDI). https://tradingeconomics.com/cameroon/school-enrollment-preprimary-percent-gross-wb-data.html
4. TradingEconomics, *Cameroon — School Enrollment, Primary (% Gross)* (World Bank WDI). https://tradingeconomics.com/cameroon/school-enrollment-primary-percent-gross-wb-data.html
5. Business in Cameroon, *Cameroon's Mobile Money Industry Accelerates, Topping CFA135.84bn*, 2025-12-10 (reporting the 2024 Annual Observatory of the Electronic Communications Market). https://www.businessincameroon.com/finance/1012-15480-cameroon-s-mobile-money-industry-accelerates-topping-cfa135-84bn
6. CEIC, *Cameroon: School Enrollment: Secondary: % Gross* (page listing related WDI series, incl. Primary Private share). https://www.ceicdata.com/en/cameroon/social-education-statistics/cm-school-enrollment-secondary--gross
7. Combien-Coute.net, *Prix d'une école primaire internationale privée à Yaoundé / au Cameroun (2026)*. https://www.combien-coute.net/ecole-primaire/cameroun/yaounde/
8. Inovedu, *Grace Dara Nursery and Primary School, Yaoundé* (school directory listing). https://www.inovedu.net/fr/schooldirectory/schools/cameroon/yaounde/ecole-maternelle-et-primaire/grace-dara-nursery-and-primary-school
9. Inovedu, *Awae Bilingual Nursery and Primary School, Yaoundé* (school directory listing). https://www.inovedu.net/fr/schooldirectory/schools/cameroon/yaounde/ecole-maternelle-et-primaire/awae-bilingual-nursery-and-primary-school
10. PayAtlas, *Cameroon Payments Guide — Compliance & Regulatory Landscape*. https://payatlas.com/countries/cameroon-cm
11. Business in Cameroon, *Orange Cameroon claims 4.5 million MoMo users, against over 5 million for MTN*, 2019-04-15. https://www.businessincameroon.com/telecom/1504-9046-orange-cameroon-claims-4-5-million-momo-users-against-over-5-million-for-mtn
12. Simiz.io, *Marché Mobile Money Cameroun 2026* (non-official market commentary). https://simiz.io/blog/marche-paiement-mobile-cameroun-2026
13. Simiz.io, *Guide Paiement Mobile Cameroun 2026* (non-official market commentary). https://simiz.io/blog/guide-paiement-mobile-cameroun-2026
14. CamPay Blog, *Do You Accept Orange Money? How Payment Method Limitations Cost Cameroon Businesses Annually* (aggregator's own marketing). https://blog.campay.net/do-you-accept-orange-money-how-payment-method-limitations-cost-cameroon-businesses-annually/
15. Legal 500, *Cameroon: Fintech — Country Comparative Guides 2026*. https://www.legal500.com/guides/chapter/cameroon-fintech/
16. Global Law Experts, *Integrate with MTN/Orange Mobile Money or Build Your Own E-wallet in Cameroon? A 2026 Decision Guide*. https://globallawexperts.com/mobile-money-integration-vs-own-ewallet-cameroon/
