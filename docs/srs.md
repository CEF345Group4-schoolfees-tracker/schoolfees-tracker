# Software Requirements Specification — School Fees Tracker

**SRS v0.2** · 2026-11-15 · Status: Draft for review
**Project:** School Fees Tracker — private nursery and primary school in Cameroon (CEF 345, Week 4 Requirements assignment)
**Document standard:** modelled on IEEE 830-1998 and ISO/IEC/IEEE 29148:2018

| Version | Date | Change |
|---|---|---|
| 0.2 | 2026-11-15 | Complete draft: full requirement set (FR-01…FR-16, NFR-01…NFR-08), MoSCoW table, open questions, references. |

Conventions:

- The requirement text has been rewritten to be specific and testable, following ISO/IEC/IEEE 29148 ("shall" statements, measurable criteria). **IDs and MoSCoW priorities are unchanged from the approved requirement list.** No requirement was silently added, removed, or re-scoped.
- Where a proposed clarification would extend — not merely restate — the original wording, the original meaning is retained and the proposal is flagged in **Open questions**. Such items are marked *(flagged)* at the point of use.
- Verification methods: **T** = test, **D** = demonstration, **I** = inspection, **A** = analysis.

---

## 1. Introduction

### 1.1 Purpose

This document specifies the software requirements for the **School Fees Tracker**, a responsive web application that lets a private nursery and primary school in Cameroon record fee payments for pupils and track outstanding balances. It is the baseline for design, implementation, and verification of the system, and the reference for the Week 4 Requirements deliverable (CEF 345).

Intended audience: the development team, the course assessor, and the school stakeholders who will review the system's scope.

### 1.2 Scope

The School Fees Tracker will:

- register pupils and their classes;
- let staff configure fees per class and academic year;
- let the Bursar record manual payments (cash, bank transfer, Mobile Money);
- accept payer-initiated online payments via Mobile Money (MTN MoMo / Orange Money);
- compute remaining balances, generate receipts, and produce expected/received/outstanding reports;
- restrict staff access to two roles — **Administrator** and **Bursar**; the **Payer** (parent/guardian) is an external actor for online payments only and holds no staff account.

All monetary amounts are in **XAF** (Central African CFA franc), the sole currency of the system.

The following are **explicitly out of scope** (see §7): SMS notifications, parent self-service accounts, payroll, exam results, inventory.

Evidence base for this specification is **secondary desk research only** (see `docs/field-notes/secondary-research.md`); no school visit or interview was performed. Residual uncertainties are collected in **Open questions** (§9).

### 1.3 Definitions, acronyms and abbreviations

| Term | Definition |
|---|---|
| SRS | Software Requirements Specification |
| FR / NFR | Functional / Non-Functional Requirement |
| MoSCoW | Must / Should / Could / Won't prioritisation |
| RBAC | Role-Based Access Control |
| XAF | Central African CFA franc (BEAC zone); whole francs, no minor subunit |
| MTN MoMo / Orange Money | The two dominant mobile money schemes in Cameroon |
| PSP | Payment Service Provider |
| EMI | Electronic Money Institution |
| BEAC / COBAC / CEMAC | Bank of Central African States / its Banking Commission / Central African Economic and Monetary Community |
| CENTIF / CNDP | Cameroon's financial intelligence unit (AML) / national data protection authority |
| Bursar | Staff member who records payments and manages day-to-day fee operations |
| Administrator | Staff member who configures the system (users, classes, fees) |
| Payer | Parent/guardian initiating an online payment; external actor, no staff account |
| Manual payment | A payment recorded in the system by the Bursar after money is received outside the app |
| Online payment | A payer-initiated Mobile Money payment confirmed electronically by the provider |
| Receipt number | Unique identifier generated per successful payment (FR-06) |
| Academic year | The school's fee/academic cycle against which fees are configured (FR-03) |

### 1.4 References

See §8.

---

## 2. Overall Description

### 2.1 Product perspective

The School Fees Tracker is a new, standalone responsive web application (greenfield; it is assumed not to replace an existing production system — *flagged, see OQ-9*). It does not hold payer funds or act as an e-money issuer; online payments flow through licensed MTN MoMo / Orange Money rails, directly or via a licensed aggregator (see Constraints, §2.6). It coexists with the school's existing manual processes, which it replaces for fee recording only.

Context (actors and external systems):

- **Administrator** — configures classes, fees, user accounts.
- **Bursar** — records payments, reviews online payments, views balances and reports.
- **Payer (parent/guardian)** — initiates an online Mobile Money payment for a specific pupil; never authenticates as staff.
- **External systems:** MTN MoMo and Orange Money payment APIs (or an aggregator providing both), used for payer-initiated online payments.

### 2.2 Product functions (summary)

- Pupil registration and class management (FR-01, FR-02)
- Fee configuration per class and academic year (FR-03)
- Manual payment recording: cash / bank transfer / Mobile Money (FR-04)
- Automatic balance calculation (FR-05)
- Unique receipt generation (FR-06)
- Payment history, outstanding-balance identification, and reporting (FR-07…FR-10)
- User account and role management (FR-11)
- Audit logging of payment creation and corrections (FR-12)
- Online Mobile Money payment: initiation, automatic recording on provider confirmation, failure handling and bursar review (FR-13…FR-16)

### 2.3 Stakeholders

| Stakeholder | Interest |
|---|---|
| School ownership / management (sponsor) | Fee collection integrity, auditability, cash-flow visibility |
| Administrator | Configuration (pupils, classes, fees, users) |
| Bursar | Fast, accurate payment recording; receipt and reconciliation workflow |
| Payer (parent/guardian) | Pay a child's fees from a phone without visiting the school |
| Payment providers (MTN MoMo, Orange Money) / aggregator | Integration partner; confirmation callbacks/webhooks |
| Course assessor (CEF 345) | Conformance to the Week 4 requirements exercise |

### 2.4 User characteristics

- **Administrator:** office staff; comfortable with web applications; infrequent-to-moderate use (setup and termly changes).
- **Bursar:** office staff; heavy daily use; not an IT specialist. NFR-08 targets recording a manual payment in under 1 minute after training.
- **Payer:** general public; variable digital literacy; uses a basic smartphone and a Mobile Money wallet (possibly via USSD); has no account on the system.
- All staff users are expected to operate within a single school (single-tenant) — *flagged, see OQ-10*.

### 2.5 Operating environment

- Responsive web application, accessible over HTTPS from current versions of Chrome, Edge, Firefox and Safari, on desktop, tablet and smartphone viewports (NFR-05).
- Server-side persistence in a relational database (choice of DBMS deferred to design); internet connectivity required for online payments and for system access.
- No client installation; no card terminal; no direct handling of PINs (NFR-06).

### 2.6 Constraints

**Research-derived constraints** (from `docs/field-notes/secondary-research.md`; these inform design and are *not* additional FR/NFR IDs):

- **C-1. Currency and arithmetic:** all amounts are XAF, whole francs; no decimal/fractional-unit handling anywhere in storage, computation or display.
- **C-2. Two mobile money rails:** online payment integration must support **both** MTN MoMo and Orange Money; single-operator integration is not acceptable (sources disagree on exact shares — Orange ~52% / MTN ~45% per 2026 market commentary; ~40/40 in 2019 official citations — but convergence is unambiguous).
- **C-3. Licensed rails only:** payments flow via licensed operator APIs or a licensed aggregator; the system shall not issue e-money, hold payer balances, or bypass operator authorization (BEAC/COBAC framework). Confirmation must rely on provider callback/reconciliation, not client-side claims.
- **C-4. AML/record transparency:** transaction records (reference, method, amount, timestamp, identity) must be retained and auditable, anticipating CENTIF reporting expectations; access is role-restricted (NFR-02) and payment events are logged (FR-12).
- **C-5. Data protection:** personal data (pupil/guardian names, phones) is stored and processed only as needed; processing respects CNDP (national data protection authority) expectations — no sharing beyond the school and its payment providers.
- **C-6. Configurable fees, never constants:** fee amounts, classes per academic year and academic-year boundaries change annually and by class; they must be configuration data entered by an authorised user (FR-03), never hard-coded.
- **C-7. Fee scale and instalments:** desk research indicates annual local private fees on the order of a few hundred thousand XAF with instalment payments common — the data model and balance computation must support part-payments (FR-04, FR-05) and per-academic-year fee assignment (FR-03).
- **C-8. Network realities:** both staff and payers may rely on mobile data; online confirmation depends on provider latency and availability, so FR-14/FR-15/FR-16 must tolerate delayed, duplicated or out-of-order provider notifications.
- **C-9. No primary evidence:** all requirements rest on desk research and the approved requirement list; no school-specific practice is assumed as fact (see Assumptions and Open questions).
- **C-10. Public repository:** the source repository is public; secrets must never be committed (reinforced by NFR-06).

**Technical/project constraints:**

- **C-11.** Two staff roles only (Administrator, Bursar); no additional roles shall be introduced without a change request.
- **C-12.** Payer is external; Payer shall have no staff account and no access to staff functions.
- **C-13.** No SMS notifications (see Out of scope, §7) — online payment UX must not depend on SMS being sent by this system.

### 2.7 Assumptions and dependencies

- **A-1.** The school will hold (or obtain) merchant arrangements with MTN MoMo and Orange Money, directly or via an aggregator, before online payments go live. API credentials are provisioned outside the repository (C-10, NFR-06).
- **A-2.** Pupils/guardians pay against a per-pupil balance; each payment (manual or online) is attached to exactly one pupil.
- **A-3.** The school provides the fee schedule per class per academic year at configuration time (data entry, not development).
- **A-4.** Academic-year boundaries and instalment structure are school-defined and entered as configuration.
- **A-5.** Single school, single tenant; no multi-campus consolidation in this revision.
- **A-6.** Staff devices have a current browser and internet access; the app is not required to work fully offline.
- **A-7.** Receipts are digital records; whether they must be printable/PDF is unspecified (see OQ-8).
- **A-8.** Guardians' MoMo accounts and phones are available to initiate online payments (FR-13); the system does not verify wallet ownership beyond the provider flow.

---

## 3. Functional Requirements

Each requirement keeps the source ID and MoSCoW priority. Statements are rewritten with ISO/IEC/IEEE 29148-style "shall" wording and measurable acceptance criteria. Where the rewrite had to fix or extend the original meaning, the item is marked *(flagged → OQ-n)* and the original intent is not silently changed.

### FR-01 — Pupil registration — Priority: Must — Verification: T

**FR-01.1** The system shall provide a pupil registration function capturing at minimum: pupil full name, admission number, class, guardian full name, and guardian phone number.
**FR-01.2** The system shall reject registration when the admission number already exists for the school; the duplicate admission number shall not be stored and an explanatory error shall be shown.
**FR-01.3** The system shall reject registration when any of the five fields is missing or blank.
**FR-01.4** The guardian phone number shall be stored in a syntactically valid international form (e.g., +237…); invalid input shall be rejected with an explanatory message. *(The exact validation rule is proposed, not in the source list — flagged → OQ-1)*
**Acceptance:** registering "Ada Lovelace / ADM-001 / Class 3 / guardian 'Grace Lovelace' / +2376…" succeeds and the record is retrievable; re-registering ADM-001 fails with a duplicate-admission-number error.

### FR-02 — Class management — Priority: Must — Verification: T, D

**FR-02.1** An Administrator shall be able to create a class with a unique name (e.g., "Nursery 1", "Class 6A").
**FR-02.2** An Administrator shall be able to update a class's details (name and any class metadata defined at design time) and to deactivate/reactivate a class.
**FR-02.3** A Bursar shall not be able to create, update or deactivate classes; any such attempt shall be denied and the refusal shall be verifiable by automated test (see NFR-02).
**FR-02.4** Deactivation shall not delete historical payment records or balances associated with the class. *(Hard-delete policy unspecified — flagged → OQ-2)*
**Acceptance:** admin creates and renames a class; bursar attempt to create a class is denied.

### FR-03 — Fee configuration per class and academic year — Priority: Must — Verification: T

**FR-03.1** Authorised users shall be able to set, for each class and each academic year, the fee amount(s) applicable to pupils enrolled in that class for that year.
**FR-03.2** Fee configuration shall be versioned by academic year: configuring fees for 2026/2027 shall not alter amounts already applied to 2025/2026.
**FR-03.3** Authorised users shall be able to update a fee configuration before any payment has been recorded against it; changes after payments exist shall be recorded in the audit log (FR-12) and shall not retroactively alter recorded payments.
**FR-03.4** Only users with the Administrator role shall configure fees, unless the school decides otherwise — *the source text says "authorised users", which role(s) is ambiguous — flagged → OQ-3*.
**FR-03.5** The model shall support either a single annual amount or an instalment breakdown (e.g., per term), per class per year. *(Instalment modelling is implied by "full or partial payment" and the project's instalment context; the exact instalment schema is unspecified — flagged → OQ-4)*
**Acceptance:** set 2026/2027 fee = 250,000 XAF for Class 3; enrol a pupil; record a 100,000 XAF partial payment; outstanding balance shows 150,000 XAF.

### FR-04 — Manual payment recording — Priority: Must — Verification: T, D

**FR-04.1** A Bursar shall be able to record a manual payment against a specific pupil, capturing: amount paid, payment method (exactly one of **cash**, **bank transfer**, **Mobile Money**), date of payment, and a payment reference.
**FR-04.2** The amount may be less than, equal to, or greater than the outstanding balance; full and partial payments shall both be accepted. Overpayment handling (accept vs reject vs credit) is unspecified — *flagged → OQ-5*.
**FR-04.3** The payment date shall not be later than the current system date, and shall not precede the pupil's enrolment date. *(Bounds proposed to make the rule testable — flagged → OQ-6)*
**FR-04.4** The reference shall be mandatory for bank transfer and Mobile Money payments (it is the bank/operator transaction ID). For cash, the system shall auto-generate the reference (the receipt number, FR-06) if none is supplied. *(The source lists "reference" unconditionally; cash handling is proposed — flagged → OQ-6)*
**FR-04.5** On successful recording, the payment shall be committed atomically with the balance update (NFR-04) and shall be assigned a receipt number (FR-06).
**Acceptance:** record 100,000 XAF cash for pupil ADM-001 dated today → balance decreases by 100,000; record a mobile money payment without a reference → rejected with validation error.

### FR-05 — Balance calculation — Priority: Must — Verification: T

**FR-05.1** For each pupil and academic year, the system shall compute: outstanding balance = configured fees for the pupil's class for that academic year (FR-03) − sum of amounts of all recorded successful payments for that pupil in that academic year.
**FR-05.2** The balance shall be recomputed and persisted-or-derived consistently after every recorded payment (manual FR-04, online FR-14) and after any correction (FR-12); displayed values shall never disagree with the underlying records by more than the current transaction.
**FR-05.3** A zero or positive balance is displayed to the user; whether negative balances (overpayment) are displayed or rejected is governed by OQ-5.
**FR-05.4** The scope of "balance" across academic years (per-year only vs carried forward) is unspecified — *flagged → OQ-7*.
**Acceptance:** configured fee 250,000; payments 100,000 + 150,000 recorded in any order → balance 0; deleting/reversing a 100,000 payment (via a recorded correction) → balance returns to 100,000 *(correction semantics per FR-12)*.

### FR-06 — Receipt number — Priority: Must — Verification: T, I

**FR-06.1** Every successful payment (manual or online) shall be assigned, atomically with its recording, a receipt number that is **unique** within the school and never reused, including after corrections or cancellations.
**FR-06.2** The receipt number format shall be deterministic and zero-padded and configurable at deployment (e.g., `RCP-2026-000123`); the format itself is a design choice, not a requirement of this SRS. *(Format proposed — flagged → OQ-8)*
**FR-06.3** The receipt shall record: receipt number, pupil (name, admission number, class), amount, method, date, reference, and the recording channel (manual/online). Whether the receipt must be printable/PDF is unspecified — *flagged → OQ-8*.
**Acceptance:** 1,000 payments created concurrently receive 1,000 distinct receipt numbers; no gaps enforced but no duplicates ever (verifiable by uniqueness constraint + concurrent test).

### FR-07 — Payment history — Priority: Must — Verification: T

**FR-07.1** Authorised users shall be able to view the complete payment history of a pupil: all recorded payments (manual and online), in reverse chronological order by default, showing date, method, amount, reference, receipt number, channel, and status.
**FR-07.2** "Authorised users" here is read as both staff roles (Administrator, Bursar); the source does not restrict it — *flagged → OQ-3*.
**FR-07.3** History shall include payments later marked as corrected, with the correction visible per FR-12.
**Acceptance:** for ADM-001 with three payments (one online), the history screen lists exactly three entries with all fields populated.

### FR-08 — Outstanding balances identification — Priority: Must — Verification: T

**FR-08.1** The system shall produce a list of pupils whose computed outstanding balance (FR-05) is greater than zero, as of today or a user-selected date, showing at minimum: pupil name, admission number, class, academic year, balance.
**FR-08.2** The list shall be filterable at least by class (reusing FR-10's filter mechanism) — *class filter proposed for usefulness; the source specifies no filter — flagged → OQ-11*.
**FR-08.3** The list shall be sortable by balance (descending) and class. *(Proposed — flagged → OQ-11)*
**Acceptance:** given three pupils (balances 0, 150,000, 250,000), only the two non-zero pupils appear.

### FR-09 — Reports — Priority: Must — Verification: T, D

**FR-09.1** The system shall generate, for a selected academic year (and optionally a class), a report containing: (a) **fees expected** (sum of configured fees × enrolled pupils, per FR-03); (b) **payments received** (sum of successful payment amounts in the selected period); (c) **outstanding balances** (a − b), with per-pupil detail available.
**FR-09.2** The report shall be generated on demand within the performance bound of NFR-03 and displayed on screen. Export (PDF/Excel/CSV) and scheduling are unspecified — *flagged → OQ-12*.
**FR-09.3** Figures in the report shall reconcile exactly with the underlying payment records (verification: automated reconciliation test).
**Acceptance:** configured expected 10,000,000 XAF across classes; received 6,000,000; report shows expected 10,000,000 / received 6,000,000 / outstanding 4,000,000.

### FR-10 — Filtering payment records — Priority: Should — Verification: T

**FR-10.1** Authenticated staff shall be able to filter the payment records list by any combination of: **pupil** (name or admission number), **class**, **academic year**, and **date** (single date or date range).
**FR-10.2** Filters shall be combinable (AND semantics) and clearable in one action; applying filters shall re-render the list within the NFR-03 bound.
**Acceptance:** filter pupil=ADM-001 AND year=2026/2027 returns only that pupil's 2026/2027 payments; clearing shows all.

### FR-11 — User account management — Priority: Must — Verification: T

**FR-11.1** An Administrator shall be able to create a user account with: full name, login identifier (unique), initial password (or a set-password mechanism), and exactly one role: **Administrator** or **Bursar**.
**FR-11.2** An Administrator shall be able to update and to deactivate/reactivate user accounts; deactivation shall immediately revoke the user's ability to authenticate and shall not delete their audit history (FR-12) or payment authorship.
**FR-11.3** An Administrator shall be able to reassign a user's role between Administrator and Bursar.
**FR-11.4** The system shall prevent an Administrator from deactivating or demoting the **last active Administrator** (safeguard; proposed, not in the source — flagged → OQ-13).
**FR-11.5** Passwords shall be stored only as salted hashes; plaintext passwords shall never be logged or displayed. *(Security baseline restated from NFR-06 — see also NFR-01.)*
**Acceptance:** admin creates bursar account → account can authenticate with the assigned role; deactivated bursar cannot log in; demoting the sole admin is refused.

### FR-12 — Audit logging — Priority: Should — Verification: T, I

**FR-12.1** The system shall write an audit log entry for every payment **creation** and every **correction** of a payment record, capturing at minimum: performing user identity, timestamp, pupil, payment/receipt reference, amount before and after (for corrections), and the reason for correction if supplied.
**FR-12.2** Audit log entries shall be append-only (no edit/delete via the UI or API) and visible to Administrators — *visibility role proposed — flagged → OQ-14*.
**FR-12.3** Correction authority (who may correct, which fields, and how corrections affect receipts) is unspecified beyond "corrections" implying an edit capability — *flagged → OQ-14*.
**Acceptance:** create a payment as bursar; correct the amount; the log shows two entries (creation, correction) with user, timestamp and before/after amounts; no log entry can be edited via the UI.

### FR-13 — Online payment initiation — Priority: Must — Verification: T, D

**FR-13.1** A Payer (external, unauthenticated staff-wise) shall be able to initiate a Mobile Money payment for a **specific pupil**, via a public payment page reachable over HTTPS, using MTN MoMo or Orange Money.
**FR-13.2** The payer shall identify the pupil (proposed: admission number + guardian phone; actual matching criteria unspecified — *flagged → OQ-15*) and **the amount** to pay (proposed: free entry bounded by the outstanding balance, defaulting to it — *flagged → OQ-15*).
**FR-13.3** On initiation, the system shall create an online payment in status **initiated** with a unique provider transaction reference and call the provider/aggregator API; the payer then completes authorisation on their phone (PIN/USSD — never handled or stored by this system).
**FR-13.4** The payment page shall not expose any data beyond what is needed to complete the payment (no other pupils' data, no staff functions).
**Acceptance:** with the provider in sandbox mode, initiating a 50,000 XAF payment for ADM-001 returns a transaction reference and a pending status; the system never sees a PIN.

### FR-14 — Automatic recording on provider confirmation — Priority: Must — Verification: T

**FR-14.1** Upon receiving a **successful** confirmation from the payment provider (callback/webhook or verified poll), the system shall, atomically and idempotently: (a) record the payment against the pupil with method Mobile Money and channel online; (b) update the pupil's balance (FR-05); (c) assign a receipt number (FR-06); (d) mark the online payment **confirmed**.
**FR-14.2** Duplicate delivery of the same provider confirmation shall have no additional effect (idempotent on provider transaction reference); it shall not create a second payment or receipt.
**FR-14.3** Callback authenticity shall be verified per the provider's mechanism (signature/secret) before any state change (NFR-06).
**Acceptance:** send the same success callback twice → exactly one payment, one receipt, balance reduced once; balance and receipt match FR-05/FR-06 rules.

### FR-15 — Failure, cancellation and duplicates — Priority: Must — Verification: T

**FR-15.1** Online payments reported as **failed**, **cancelled**, or **expired**, and any **duplicate** confirmations, shall not change any pupil balance, under any interleaving of events.
**FR-15.2** Each such event shall be persisted with its provider status and automatically placed in a **bursar review queue** (flagging shall be automatic).
**FR-15.3** The Bursar shall be able to mark a queued item as reviewed (with an optional note); review disposition shall not itself move money. *(Disposition semantics proposed — flagged → OQ-16)*
**Acceptance:** callbacks for failure/cancel/duplicate → balances unchanged; each appears in the review queue with provider payload stored.

### FR-16 — Online payment status and matching — Priority: Must — Verification: T

**FR-16.1** The Bursar shall be able to view a list of online payments with status (at minimum: initiated, confirmed, failed, cancelled/expired, and any exception states from FR-15), filterable/sortable by date and status.
**FR-16.2** Each record shall show: provider transaction reference, amount, pupil (if matched), guardian contact, timestamps, and provider status.
**FR-16.3** The Bursar shall be able to **match** an online payment to a pupil when automatic attribution failed or was wrong (e.g., wrong admission number entered by the payer): selecting the correct pupil shall associate the payment with that pupil; if the payment is confirmed, recording shall then follow FR-14 semantics (recorded once, balance updated, receipt generated) — *this re-matching behaviour is proposed; the source does not state it — flagged → OQ-16*.
**FR-16.4** Re-matching shall be audit-logged (FR-12).
**Acceptance:** a confirmed payment with unmatched pupil appears in the status list; bursar matches it to ADM-001 → balance for ADM-001 decreases once and a receipt is issued; audit log shows the re-match.

---

## 4. Non-Functional Requirements

### NFR-01 — Authentication — Priority: Must — Verification: T

**NFR-01.1** Every staff (protected) feature shall require an authenticated session; unauthenticated requests shall be redirected to the login page or answered 401/403 and shall not return protected data.
**NFR-01.2** Credentials shall travel only over HTTPS (NFR-06); passwords stored only as salted hashes (FR-11.5).
**NFR-01.3** Sessions shall expire after a configurable inactivity period (default 30 minutes proposed — *flagged → OQ-17*) and on explicit logout.
**NFR-01.4** Authentication applies to staff functions. The payer payment page (FR-13) is public-by-design and is explicitly not covered by staff session requirements; its abuse controls (rate limiting, pupil-lookup throttling) are design matters — *flagged → OQ-15/OQ-17*.

### NFR-02 — Role-based access control — Priority: Must — Verification: T

**NFR-02.1** The system shall enforce RBAC server-side with exactly two staff roles: **Administrator** and **Bursar**.
**NFR-02.2** Minimum permission matrix (derived from FRs; changes require change control):

| Function | Administrator | Bursar |
|---|:-:|:-:|
| Register pupils, view history, payments, balances, reports, filters (FR-01, 04–10) | ✔ | ✔ |
| Manage classes (FR-02) | ✔ | ✘ |
| Configure fees (FR-03) | ✔ | ✘ *(per FR-03.4 flag)* |
| Create/manage users and roles (FR-11) | ✔ | ✘ |
| Create payments (FR-04) | ✔ *(per "recorded manually by the bursar" decision, Admin recording is optional — flagged → OQ-18)* | ✔ |
| View/match online payments, review queue (FR-15, FR-16) | ✔ *(view)* | ✔ *(view + match/review)* |
| View audit log (FR-12.2) | ✔ | ✘ |

**NFR-02.3** Any request for a function not permitted to the caller's role shall be denied with no side effects.

### NFR-03 — Performance — Priority: Should — Verification: T, A

**NFR-03.1** "Standard searches" shall be defined as: pupil lookup by name/admission number, payment-record filtering (FR-10), and the outstanding-balances list (FR-08). *(Definition proposed — flagged → OQ-19)*
**NFR-03.2** 95% of standard searches shall complete in **under 3 seconds** with **up to 1,000 pupil records** in the database, on the school's production hardware profile (or equivalent test fixture).
**NFR-03.3** Verification: automated performance test against a seeded 1,000-pupil dataset; report p50/p95 latencies.

### NFR-04 — Transactional integrity — Priority: Must — Verification: T

**NFR-04.1** Every payment-affecting operation (manual record FR-04, online confirmation FR-14, correction FR-12) shall execute as a single ACID transaction: either all effects (payment row, balance, receipt, audit/log) are committed, or none are.
**NFR-04.2** A failed payment operation (crash, validation error, provider inconsistency, duplicate callback) shall leave the database in the pre-operation state — never a payment without a balance update, a balance without a payment, or a duplicate receipt.
**NFR-04.3** Verification: fault-injection tests (forced failure at each step) plus duplicate-callback tests (FR-14.2); zero partial-state occurrences tolerated.

### NFR-05 — Responsiveness — Priority: Should — Verification: D, T

**NFR-05.1** The UI shall adapt to desktop (≥1280px), tablet (~768–1279px) and smartphone (<768px) viewports without loss of function, horizontal scrolling or clipped critical controls.
**NFR-05.2** Core flows (login, record payment, view history, outstanding list) shall be completable on a 360px-wide smartphone screen.
**NFR-05.3** Touch targets on mobile shall be at least 44×44 CSS px (proposed accessibility baseline — *flagged → OQ-20*).

### NFR-06 — Payment security — Priority: Must — Verification: I, T

**NFR-06.1** The application shall be served over **HTTPS only**; HTTP requests shall redirect to HTTPS (or be rejected); TLS 1.2+ required.
**NFR-06.2** The system shall **never store card numbers or Mobile Money PINs**; payer authorisation happens in the provider's channel (app/USSD) and the system stores only non-sensitive artifacts (provider references, statuses, amounts).
**NFR-06.3** Secrets (DB credentials, provider API keys/secrets, signing secrets) shall live in environment/configuration outside the repository; **no secret shall ever be committed to this (public) repo**; `.gitignore` shall exclude secret-bearing files and a committed-secret scan is expected before merge (process, to be wired in CI).
**NFR-06.4** Provider callbacks shall be authenticated/verified before acting on them (FR-14.3); amounts received shall be verified against the initiated amount before recording.

### NFR-07 — Backup and restore — Priority: Should — Verification: I, D

**NFR-07.1** The database shall be backed up **daily** (automated), with backups encrypted at rest and stored separately from the primary database host.
**NFR-07.2** A documented restore procedure shall exist in the repo or the school's operations manual (location to be defined — *flagged → OQ-21*), covering: where backups live, how to verify integrity, and steps to restore to a clean environment.
**NFR-07.3** Retention period and restore-drill cadence are unspecified (proposed: ≥30 daily backups, restore drill each term — *flagged → OQ-21*).

### NFR-08 — Learnability — Priority: Should — Verification: D

**NFR-08.1** A trained Bursar shall be able to record a manual payment — from opening the screen to a saved, receipted payment — in **under 1 minute**.
**NFR-08.2** "Trained" is defined as having completed the school's bursar onboarding (duration to be defined — *flagged → OQ-22*).
**NFR-08.3** Verification: timed task measurement in a usability session with a trained bursar (at least 3 repetitions; median time < 60s). *Note: no usability data exists yet (desk research only); this is a target to be measured, not a measured fact.*

---

## 5. Use Cases

**To be added in the next revision** (use case diagram at `docs/diagrams/usecases.drawio.svg` and detailed descriptions for the online payment and manual payment scenarios are planned as follow-up work).

---

## 6. MoSCoW Prioritisation

| ID | Item (abbreviated) | Priority |
|---|---|---|
| FR-01 | Register pupils | Must |
| FR-02 | Manage classes (Administrator) | Must |
| FR-03 | Configure fees per class & academic year | Must |
| FR-04 | Record manual payment (cash/bank/MoMo) | Must |
| FR-05 | Compute remaining balance | Must |
| FR-06 | Unique receipt number | Must |
| FR-07 | View pupil payment history | Must |
| FR-08 | Identify pupils with outstanding balances | Must |
| FR-09 | Reports: expected / received / outstanding | Must |
| FR-10 | Filter payment records | Should |
| FR-11 | Manage user accounts & roles | Must |
| FR-12 | Audit log payment creation/corrections | Should |
| FR-13 | Payer initiates online MoMo payment | Must |
| FR-14 | Auto-record on provider confirmation | Must |
| FR-15 | Failed/cancelled/duplicate never move money; flagged | Must |
| FR-16 | Bursar views online payment status & matches to pupils | Must |
| NFR-01 | Authentication for protected features | Must |
| NFR-02 | Role-based access control | Must |
| NFR-03 | Search < 3s @ 1,000 pupils | Should |
| NFR-04 | Failed payment never leaves incomplete transaction | Must |
| NFR-05 | Responsive desktop/tablet/smartphone | Should |
| NFR-06 | HTTPS, no card/PIN storage, no committed secrets | Must |
| NFR-07 | Daily backup + documented restore | Should |
| NFR-08 | Trained bursar records payment < 1 min | Should |

**Totals:** 18 Must (14 FR + 4 NFR) · 6 Should (2 FR + 4 NFR) · 0 Could · 0 Won't (this release).

## 7. Out of Scope

Explicitly out of scope for this revision (per the approved scope decision):

1. **SMS notifications** (to payers or staff).
2. **Parent self-service accounts** (a payer portal with login; the Payer actor exists only for online payment initiation, FR-13).
3. **Payroll** for teaching/non-teaching staff.
4. **Exam results** and academic-record management.
5. **Inventory** (uniforms, books, supplies stock).

Additionally, the following follow from project decisions and desk research (not to be added without a change request): card payments and any storage of card data (NFR-06.2); the system acting as an e-money issuer or holding payer funds (C-3); roles beyond Administrator and Bursar (C-11); multi-school/multi-tenant operation (A-5); non-XAF currencies (C-1).

## 8. References

1. IEEE Std 830-1998, *IEEE Recommended Practice for Software Requirements Specifications* (superseded by ISO/IEC/IEEE 29148).
2. ISO/IEC/IEEE 29148:2018, *Systems and software engineering — Life cycle processes — Requirements engineering*.
3. `docs/field-notes/secondary-research.md` — secondary desk research underpinning this SRS (sources with access dates).
4. World Bank, *Project Appraisal Document — Cameroon Education Sector Support Project (PAD2366)*, 2018. https://documents1.worldbank.org/curated/en/991921525399281987/pdf/CAMEROON-PAD-03232018.pdf
5. Business in Cameroon, *Cameroon's Mobile Money Industry Accelerates, Topping CFA135.84bn*, 2025-12-10. https://www.businessincameroon.com/finance/1012-15480-cameroon-s-mobile-money-industry-accelerates-topping-cfa135-84bn
6. Legal 500, *Cameroon: Fintech — Country Comparative Guides 2026*. https://www.legal500.com/guides/chapter/cameroon-fintech/
7. PayAtlas, *Cameroon Payments Guide — Compliance & Regulatory Landscape*. https://payatlas.com/countries/cameroon-cm
8. Global Law Experts, *Integrate with MTN/Orange Mobile Money or Build Your Own E-wallet in Cameroon? A 2026 Decision Guide*. https://globallawexperts.com/mobile-money-integration-vs-own-ewallet-cameroon/

## 9. Open Questions

Numbered for traceability; none of these is silently resolved in this draft.

- **OQ-1** (FR-01.4) Guardian phone validation: exact format rule (e.g., +237 and 9 digits) and whether landlines are allowed?
- **OQ-2** (FR-02.4) Class lifecycle: is hard deletion ever permitted, or deactivation only? What happens to a class with pupils?
- **OQ-3** (FR-03.4, FR-07.2) "Authorised users": which roles may configure fees (Administrator only, or Bursar too) and who may view payment history?
- **OQ-4** (FR-03.5) Instalment model: single annual fee vs instalment plan (per term? per month? fixed counts/dates?); are instalment dates enforced?
- **OQ-5** (FR-04.2, FR-05.3) Overpayment: accept and credit, cap at balance, or reject? And handling of negative balances.
- **OQ-6** (FR-04.3, FR-04.4) Payment date bounds (future dates? back-dating limits?) and reference requirements — cash reference optional as proposed?
- **OQ-7** (FR-05.4) Balance scope: per academic year only, or cumulative with arrears carried into the next year?
- **OQ-8** (FR-06.2, FR-06.3, A-7) Receipt: required output format (screen only / printable PDF?) and receipt number format.
- **OQ-9** (§2.1) Greenfield confirmation: does the school currently use paper/Excel/another system, and is migration of existing balances in scope?
- **OQ-10** (§2.4) Single school only, or will this serve other schools later (affects tenancy and data model)?
- **OQ-11** (FR-08.2, FR-08.3) Outstanding-balances list: required filters/sort beyond class, and any export?
- **OQ-12** (FR-09.2) Reports: export formats needed (PDF/Excel/CSV)? On-screen only acceptable? Any scheduled/emailed reports?
- **OQ-13** (FR-11.4) User lifecycle: deactivation vs deletion, and is the last-Administrator safeguard wanted?
- **OQ-14** (FR-12.2, FR-12.3) Corrections: who may correct (Bursar? Administrator?), which fields, mandatory reason, and who may view the audit log?
- **OQ-15** (FR-13.2, NFR-01.4) Payer flow: how does the payer identify the pupil and amount (admission number + guardian phone? lookup by phone?); full vs partial vs fixed-instalment amounts online; rate-limit/abuse controls; confirmation timeout and session expiry on the payment page?
- **OQ-16** (FR-15.3, FR-16.3) Re-matching: when a confirmed online payment was matched to the wrong/no pupil, should the match action itself record the payment (as proposed) or merely annotate it? What disposition options does the bursar review queue offer?
- **OQ-17** (NFR-01.3) Session lifetime: is 30 minutes inactivity acceptable for staff; any separate rule for the payer page?
- **OQ-18** (NFR-02.2) May the Administrator record manual payments too, or is recording exclusive to the Bursar as the decision text implies?
- **OQ-19** (NFR-03.1) Confirm the definition of "standard search" (pupil lookup + FR-10 filtering + FR-08 list) or provide another.
- **OQ-20** (NFR-05.3) Acceptable to adopt the 44×44px touch-target guideline, or is another baseline preferred?
- **OQ-21** (NFR-07.2, NFR-07.3) Where is the restore procedure documented, what backup retention is required, and how often must a restore be tested?
- **OQ-22** (NFR-08.2) What counts as "trained" for the 1-minute bursar benchmark, and who measures it?
- **OQ-23** (general) UI language(s): English only, or also French (Cameroon is bilingual; school language profile unknown)?
- **OQ-24** (FR-09.1) "Fees expected": calculated from current enrolment × current fee config — correct, or should it use per-term enrolment snapshots?
