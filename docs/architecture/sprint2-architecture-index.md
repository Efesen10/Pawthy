# Pawthy — Sprint 2 Architecture Deliverable Index (SRS Update)

**Owner:** Efe (Software Architect / Technical Lead)  
**Reconciled:** 20 November 2026  
**Head:** `Efesen10/Pawthy:feature/sprint2-srs-20261120-sync`; **PR target:** `HUARGE/Pawthy:develop`  
**Live API baseline:** `api/openapi.yaml` — OpenAPI 3.1.0, `info.version: 0.6.3`  
**New source evidence:** Deniz's 69-page Functional Requirements SRS + 5-page Section 5.2 Software Interfaces supplement (shared externally; not committed as SRS files).  
**Review state:** Updated architect-owned **design proposal**; API spec remains inconsistent until PO/API owners reconcile, version and QA test. No code has been implemented by these documents.

## 1. Architect-owned files in this follow-up

| Deliverable | Path | Contents |
|---|---|---|
| C4 Level 1, 2, 3 | [C4 Architecture](c4-architecture.md) | Updated actors/containers/components, SRS-selected Django/MySQL, guided tour, notifications, private owner calendar, record soft-delete/restore |
| UML design | [UML Diagrams](../uml-diagrams/uml-diagrams.md) | Use Case, two Class, first login/appointment/reminder/availability/guided-tour Sequence, three State diagrams |
| Formal Use Case source | [PlantUML](../uml-diagrams/use-cases.puml) | Explicit vet vs owner rights, US-14 tour, provisional US-08 gap |
| Architecture/API/SRS traceability | [Alignment Review](api-srs-alignment.md) | 14 story identifier coverage with US-08 source gap, current API contract mismatches and NFR design mapping |
| Full issue ledger | [SRS 20 Nov Review](srs-2026-11-20-review.md) | Detailed requirement IDs, permissions, conflicts, necessary PO decisions and supplementary 5.2 SI-01–SI-07 |
| API-owner proposal **only** | [Suggested API Delta](../../api/srs-sync-proposal-2026-11-20.md) | Current 0.6.3 vs proposed owner phone/ID login, clinical restore, calendar, appointments, reminders, US-14; candidate URLs expressly not normative |

## 2. SRS system boundaries now clarified

- Technology selected in the five-page SRS supplement: **HTML/CSS/JavaScript frontend**, **Python/Django backend**, **MySQL database**, **OpenAPI/YAML specification**. MySQL must be accessed **via the backend only**, with server-side validation, authorization and referential integrity.
- **Soft deletion and restoration** required for clinical-record lifecycle; restoration must preserve independent vaccination and owner diary records. Archive, ordinary modification and deceased state must not be confused with deletion.
- New **US-14 Optional Guided Tour** is P2/Should Have, **not a mandatory initial Sprint 3 MVP feature**. State is per owner account; four steps and Help replay are documented.
- Vet only creates appointments after phone/in-person agreement. Hard appointment overlaps are rejected even if hours are overridden. Out-of-hours/unavailable intervals require **explicit vet warning confirmation**; successful override does not unlock adjacent hours.
- Owner can only view **available** weekly times and own appointment information (read-only), never other owners' records or private vet scheduling information.
- Public clinic contact remains; whether the existing *public BUSY/CLOSED* calendar endpoint remains available is **open for PO/API decision**.
- Vaccination read status and clinical completion are independent. Deceased pet suppresses reminders while retaining history; owner archiving hides personal active reminders but should not hide vet monitoring.

## 3. Critical unresolved decisions

1. **PO editorial:** Updated 69-page SRS duplicates **US-07/section 3.7** and omits **US-08/section 3.8**. Sprint 1's read-history use case remains provisional.
2. **API owner:** Current 0.6.3 requires email, lacks required owner phone/Owner ID login, chip/DOB/life-stage fields, clinical restore, reminder mark-read, Guided Tour, appointment notices/Completed state and hours override.
3. **PO + security/API:** Decide whether weekly owner-only AVAILABLE calendar should replace public BUSY/CLOSED or coexist as a separately scoped API view.
4. **Backend:** Design atomic conflict checking, exception warning ACK, protected media authorization, owner/pet relationships and clinical soft-delete/restore audit and retention policies.
5. **QA:** Revise baseline v0.6.3 tests to conform to approved SRS/API changes. No tests executed or claimed passed.

## 4. Submission policy

This branch edits **only architect-owned documents and adds a nonnormative API proposal Markdown file**. It does not change `api/openapi.yaml`, `non-functional-requirements.md`, `test-scenarios.md`, teammate SRS source, frontend, CI/CD code or runtime application files.

- [x] New 69-page + supplementary 5-page SRS evidence incorporated.
- [x] C4 and UML updated and API discrepancies documented rather than hidden.
- [x] Critical US-08 source gap, clinic calendar privacy and soft-delete/restore highlighted.
- [ ] Product Owner and API owner confirm final contract decisions.
- [ ] Diagram rendering/compilation and runtime/QA tests (not executed).
- [ ] PR reviewer approval and merge.
