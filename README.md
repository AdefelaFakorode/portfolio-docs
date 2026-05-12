# Adefela Fakorode — Software Engineer

> Houston, TX | [LinkedIn](https://linkedin.com/in/adefelafakorode) | adefakorode@gmail.com | [Portfolio](https://adefela.com)

## 📊 Impact at a Glance

| Metric                                   | Result                                                                                     |
| ---------------------------------------- | ------------------------------------------------------------------------------------------ |
| Production SPAs shipped                  | 7 (intake, declaration, client-docs, client-mgmt, case-type-admin, LMS-admin, LMS-student) |
| AWS Lambdas owned                        | 5 (three-day-process, attendance, get-client-info, LMS-lambda, intake-API)                 |
| USCIS forms unified into a single intake | 6 (I-130, I-485, I-864, I-131, I-765, G-28)                                                |
| API modules owned in core platform       | 5+ (documentType, clientFolder, courses+S3, document-route, baseCreateSchema)              |                                                         |
---

## Applications Worked On

### 1. USCIS Intake Form Platform

**May 2026 | Meneses Law PLLC**

Designed and built a unified intake system that collects client information once through a guided wizard, then maps the answers to multiple USCIS forms (I-130, I-485, I-864, I-131, I-765, G-28) — replacing a fragmented per-form intake workflow. Reduced paralegal data-entry duplication and gave clients a single coherent experience across an entire family-based filing.

- Built a 7-step React Hook Form wizard ( Petitioner → Beneficiaries → Financial/I-864 → Sponsorship Outlook → Joint Sponsor 1 → Joint Sponsor 2 → Review) with free step-jumping and validation deferred to the Review page
- Implemented a reactive I-864 sponsorship allocator: applies 2025 HHS poverty guidelines, factors in assets-as-income-substitute, prioritizes the principal beneficiary, distributes the remainder across up to 2 joint sponsors, and renders a terminal "filing cannot proceed" state when 3 sponsors still can't cover all beneficiaries
- Made joint-sponsor wizard steps allocator-driven — Joint Sponsor 1 appears only when needed, Joint Sponsor 2 only when Joint Sponsor 1 is still insufficient, and slots auto-clear when the petitioner's updated income makes them redundant
- Aligned frontend Zod schemas 1:1 with backend Pydantic models across petitioner / beneficiary / I-130 / I-485 / I-864 / shared enums, eliminating silent submit 422s
- Killed a class of data-integrity bugs by replacing stubbed joint-sponsor fields (gender, biographic info, LPR details) with real user-collected data and adding lock-in tests so the stubs can't reappear
- Wrote shared input sanitizers (E.164 phone normalization, ZIP+4 formatting, digit-only IDs, lowercased emails) applied uniformly across petitioner / beneficiary / joint-sponsor surfaces
- Rebuilt the Review page from a 44-line stub into a section-by-section summary with pencil "Edit this section" jumps that resolve step IDs dynamically so the Review stays correct as wizard steps reorder
- Added a `DEV`-gated debug panel surfacing live RHF draft, serialized BE payload, submit-schema validation issues, and allocator output — gated behind `import.meta.env.DEV` so it never ships to prod
- Shipped 48 passing vitest cases covering sanitizers, allocator behavior, and a regression test asserting submits fail when biographic info is missing

**Impact:** 6 USCIS forms generated from a single intake, fully reactive sponsorship eligibility engine, FE↔BE schema parity, dynamic per-beneficiary state isolation for multi-applicant filings

**Next:** localStorage autosave with versioned shape key, real submit handler / API POST wiring, petitioner employment UI page, `address.inCareOf` capture, I-864A household contributor flow, field-level (not section-level) edit jumps from Review

`React` `TypeScript` `React Hook Form` `Zod` `Vite` `Vitest` `TailwindCSS` `React Router` `Pydantic (BE contract)` `Docker` `MongoDB`

---

### 2. Three-Day Process Lambda — Document Aggregation Service

**Feb – Mar 2026 | Meneses Law PLLC**

Refactored a serverless aggregation pipeline that surfaces client OneDrive folder/document state to the case-management frontend. The legacy aggregation pulled from scattered collections with inconsistently shaped payloads; the refactor made `folder_templates` the source of truth for folder structure and `document_route` the source of truth for the documents that populate them.

- Rebuilt `getClientOneDriveItems` aggregation using a newly authored aggregation builder, sourcing folder structure from `folder_templates` and document population from `document_route`
- Added relation-aware filtering so documents tied to a specific relationship (spouse, child, etc.) only appear under the selected relation
- Added `required` flag propagation per-document so the frontend can render mandatory-vs-optional state without a second round-trip
- Reshaped the payload sent to the frontend so the SPA no longer has to reconcile mismatched collections client-side
- Cast `caseTypeId` / `clientId` to `ObjectID` to fix MongoDB query mismatches that were silently returning empty results
- Trimmed unused `folder_5`–`folder_8` slots from the aggregation since the product only uses folders 1–4

**Impact:** Single source of truth for folder/document structure, relation-aware filtering, consistent payload shape consumed by `client-documents-spa`

`Node.js 22` `AWS Lambda` `AWS SAM` `MongoDB` `OneDrive API` `Microsoft Graph` `esbuild` `TypeScript` `Zod`

---

### 3. Client Documents SPA

**Sep 2025 – Mar 2026 | Meneses Law PLLC**

Frontend for clients to view, request, and upload case documents into the structured folder template surfaced by the three-day-process Lambda. Owned the refactor that brought the SPA in line with the new backend folder structure plus a comprehensive code-quality cleanup pass.

- Refactored the SPA against the updated folder-structure backend so documents render in the correct slots after the API reshape
- Cleaned up search logic and removed a duplicate client-fetch that was double-firing on mount
- Standardized naming and type conventions across the project to eliminate inherited inconsistency
- Fixed Linux CI failures by correcting filename casing on `Loader` and `Modal/` imports (case-insensitive macOS dev was silently passing)
- Shipped a document-request + upload-validation pass covering the upload flow end-to-end
- Closed out remaining SonarQube findings to bring the project to a clean static-analysis baseline

**Impact:** SPA aligned 1:1 with refactored backend, search/duplicate-request bug eliminated, naming standardized across the codebase, CI green on Linux

`React 19` `Vite` `MUI X DataGrid` `Flowbite` `Tailwind` `JWT` `TypeScript`

---

### 4. Meneses Law Core API ("Voltron")

**Sep 2025 – Feb 2026 | Meneses Law PLLC**

Core Hono-based API serving every internal SPA at the firm. Contributed modules and refactors across the courses/SOPs (LMS), client-folder, document-type, document-route, and admin-UI domains.

- Built `documentType` and `clientFolder` modules end-to-end to unblock the Admin-UI workstream
- Created a shared `baseCreateSchema` used by both Courses and SOPs to eliminate duplicated DTO definitions and drift between sibling resources
- Implemented S3-backed course creation (presigned uploads + metadata persistence)
- Refactored the `document_route` module — added `required` field support and removed unused DTOs flagged in review
- Adjusted the enrollment schema to accept `endTime: null` so the quiz-attempt-time-tracking pipeline could persist in-progress attempts

**Impact:** 5+ API modules owned end-to-end, shared base schema reduced DTO drift across sibling resources, S3-integrated course assets

`Hono` `Bun` `Prisma` `TypeScript` `AWS S3` `AWS Secrets Manager` `Zod` `JWT (aws-jwt-verify)` `OpenAPI/Swagger`

---

### 5. Case Type Admin Management SPA

**Jan 2026 | Meneses Law PLLC**

Internal admin dashboard for defining and managing case types — the configuration that drives folder/document structure across every other case-management surface. Built the v1 dashboard plus a case-type cloning feature so admins can duplicate similar case types instead of rebuilding from scratch.

- Built Admin Dashboard v1 from a blank repo (React 19 + Vite + Tailwind 4)
- Implemented drag-and-drop ordering for case-type configuration via `sortablejs`
- Shipped case-type cloning — admins can duplicate an existing case type as a starting point and tweak from there
- Restructured the project's file/folder layout once the feature surface stabilized

**Impact:** Foundational admin tool for case-type configuration, cloning workflow cuts setup time for similar case types

`React 19` `TypeScript` `Vite` `Tailwind CSS 4` `SortableJS` `Zod` `react-icons`

---

### 6. Universal Navbar — Stencil Web Component

**Oct 2025 – Jan 2026 | Meneses Law PLLC**

Shared `<side-header>` web component built with Stencil and consumed by every internal Meneses Law SPA. Replaces per-app nav implementations with a single distributable bundle.

- Built the Stencil component v2 from a redesigned spec (and cleaned out legacy build outputs from the prior iteration)
- Wrote setup/integration docs covering both local dev (`stencil serve`) and CDN-based consumption
- Made the navbar `position: fixed` with `z-index: 100` so it stays anchored and isn't covered by overlays in consuming SPAs
- Distributed via npm/CDN with both ESM and CJS entrypoints plus typed `.d.ts` exports

**Impact:** Single-source navbar across the entire internal application portfolio, eliminates copy-pasted nav drift between SPAs

`Stencil` `Web Components` `TypeScript`

---

### 7. Learning Management System — Admin + Student + Lambda

**Aug 2025 – Jan 2026 | Meneses Law PLLC**

Internal LMS spanning three repos: admin portal, student-facing SPA, and the AWS Lambda backend that aggregates dashboards and powers course/quiz state. Contributed across all three.

**Admin Portal (`learning-management-system-admin-spa`)**

- Built the SOP-creation feature + payload contract
- Toast notifications for course/SOP creation and unassigned states via `react-hot-toast`
- Naming-convention cleanup pass across the codebase
- SonarQube error remediation pass

**Student SPA (`learning-management-system-spa`)**

- Results page UI refactor — now pulls from API instead of reading stale `sessionStorage`
- Quiz attempt time tracking — added `startTime` / `endTime` to the submit payload, wired through to the API + enrollment schema
- Dynamic current-year display, fixed enrollment progress bugs

**Lambda Backend (`learning-management-system-lambda`)**

- Refactored `getDashBoard` / `getEmployeeDashBoard` to pull from canonical `courses` + `enrollment` collections on `meneses_db` (were reading from an out-of-date LMSP source)
- Added 3 new aggregation metrics: average attempts, attempt-vs-score, first-attempt pass rate
- Implemented `usedAttempts` + score-history aggregation, fixed top-user calculation, and clamped scores to 100 with DESC ordering

**Impact:** Three-repo product owned across UI + API + aggregation, dashboard metrics rebuilt against single source of truth, full quiz-time telemetry pipeline

`React 18` `Vite` `TypeScript` `MUI` `Chart.js` `Plotly` `Recharts` `Node.js 22` `AWS Lambda` `AWS SAM` `MongoDB` `AWS Secrets Manager` `Microsoft Graph` `Mailchimp Transactional` `Vitest`

---

### 8. Attendance Automation Lambda

**Nov 2025 – Jan 2026 | Meneses Law PLLC**

Scheduled Lambda that scrapes employee attendance from a third-party portal and persists it to MongoDB. Originally implemented with Selenium; rewrote to Puppeteer once Selenium proved too slow on multi-row datasets.

- Bootstrapped the Lambda end-to-end: AWS SAM + Docker for local dev, Node 22 runtime, esbuild bundling
- Integrated AWS Secrets Manager for portal credentials so no creds live in code or environment variables
- Got the original Selenium scraper working against the live portal authenticated via Secrets Manager
- **Rewrote scraper from Selenium → Puppeteer** using `@sparticuz/chromium` for a Lambda-friendly headless Chrome build — cut per-row execution time dramatically on multi-row datasets

**Impact:** Production attendance automation, Selenium→Puppeteer rewrite for speed, secrets-manager-backed credentials

`Node.js 22` `AWS Lambda` `AWS SAM` `Puppeteer + Sparticuz Chromium` `MongoDB` `AWS Secrets Manager` `esbuild`

---

### 9. Get Client Info Lambda

**Dec 2025 | Meneses Law PLLC**

Lambda backing the mobile/portal client experience — presigned S3 uploads, document review-queue insertion, client messaging, and SNS-based mobile push notifications.

- Stood up the Lambda from a reusable Node 22 template
- Integrated AWS Secrets Manager and switched from inlined credentials to SDK-backed secret fetches
- Wired up AWS SNS for mobile push notifications (device registration + topic publishing)
- Sanitized debug output — removed all `console.log` calls that were leaking client tokens and PII before the Lambda shipped

**Impact:** Production-ready client-info Lambda with presigned uploads, push notifications, and zero PII in logs

`Node.js 22` `AWS Lambda` `AWS SAM` `AWS S3` `AWS SNS` `AWS Secrets Manager` `MongoDB` `crypto-js`

---

### 10. Client Management SPA

**Oct – Nov 2025 | Meneses Law PLLC**

Internal SPA for browsing and managing the firm's client base — searchable, filterable, paginated views with case-type and office associations. Owned a major filtering/pagination correctness pass and the migration of core domain-shape definitions to Zod.

- Built the "Apply Filters" feature end-to-end: multi-filter selection, apply/clear, working correctly alongside search and pagination
- Fixed a pagination bug that returned only a partial client list when filters were active
- Added slide-in/slide-out modal transitions for the client detail flyout
- Introduced Zod schemas for `caseType`, `client`, and `office` to enforce data shape at the API boundary
- Migrated all `interface` declarations to TypeScript `type` aliases for project-wide consistency
- Added a shared date-formatter utility for the client list
- Switched API base from hard-coded `localhost` to env-configured URL so non-local environments work

**Impact:** Multi-filter + pagination + search working together correctly, Zod-validated client/case-type/office domain models

`React 19` `Vite` `TypeScript` `Tailwind CSS 4` `Zod` `Material Design Icons` `typed-usa-states`

---

### 11. Declaration Form SPA

**Sep 2025 | Meneses Law PLLC**

Standalone SPA for collecting client declarations. Built questions 1–8 with mixed input types (single/multi-select, tag inputs, comboboxes) plus the validation and payload-output pipeline that hands data off to the backend.

- Built questions 1–8 with state management that adapts which sub-questions appear based on prior answers
- Integrated Mantine `TagInput` for question 8 (free-text multi-tag entry)
- Integrated shadcn/Radix `Combobox` for questions 4 and 7 (searchable select)
- Added validation surfacing missing required fields, plus toast notifications for save state
- Fixed phone-number validation and status-constant inconsistencies
- Cleaned up payload outputs so the API receives canonical shape regardless of UI input variations

**Impact:** 8-question declaration intake with adaptive UI, multi-input-type collection, validated payload contract

`React 19` `TypeScript` `Vite` `Mantine` `shadcn/Radix` `Headless UI` `Heroicons` `Tailwind CSS 4` `react-hot-toast` `cmdk`

---

## 🛠️ Tech Stack

```
Frontend:       React 18/19 | TypeScript | Vite | Tailwind CSS 4
                React Hook Form | Zod | React Router | MUI / MUI X DataGrid
                Mantine | shadcn / Radix | Headless UI | Heroicons | Flowbite
                Chart.js | Plotly | Recharts | react-hot-toast
Web Components: Stencil (universal navbar shared across all internal SPAs)
Backend:        Hono | Bun | Node.js 22 | Prisma | OpenAPI / Swagger
                JWT (aws-jwt-verify) | Pydantic (consumed contract)
Serverless:     AWS Lambda | AWS SAM | esbuild | Puppeteer (@sparticuz/chromium)
                Selenium (legacy, rewritten)
AWS:            S3 | Secrets Manager | SNS | STS | Lambda | SAM
Databases:      MongoDB | MongoDB Atlas
Microsoft:      Microsoft Graph | OneDrive API | Azure Identity
Email/Comms:    Mailchimp Transactional | SNS push notifications
Testing:        Vitest | Playwright | Testing Library
Tooling:        ESLint | Prettier | SonarQube | GitHub Actions | Docker
```

---

## 📜 Certifications

_All listed below are actively in progress — currently studying for and scheduled to sit for each._

| Certification                    | Level        | Status         |
| -------------------------------- | ------------ | -------------- |
| AWS Certified Cloud Practitioner | Professional | In Progress 🚧 |
| Claude Certified Architect       | Professional | In Progress 🚧 |

---

## 📅 Project Timeline

| Date                | Project                          | Key Contribution                                                                                |
| ------------------- | -------------------------------- | ----------------------------------------------------------------------------------------------- |
| Mar – May 2026      | USCIS Intake Form Platform       | 6-form unified intake, reactive I-864 allocator, FE↔BE schema parity                            |
| Feb – Mar 2026      | Three-Day Process Lambda         | `folder_templates` as single source of truth, relation-aware doc filtering                      |
| Sep 2025 – Mar 2026 | Client Documents SPA             | Folder-structure refactor, search/duplicate-request fix, Linux CI casing                        |
| Sep 2025 – Feb 2026 | Meneses Law Core API ("Voltron") | 5+ modules: document-type, client-folder, courses-S3, baseCreateSchema, document-route refactor |
| Jan 2026            | Case Type Admin Management SPA   | Admin dashboard v1, drag-drop ordering, case-type cloning                                       |
| Oct 2025 – Jan 2026 | Universal Navbar (Stencil)       | Shared web component used across every internal Meneses Law SPA                                 |
| Aug 2025 – Jan 2026 | Learning Management System       | 3-repo product (admin + student + lambda), dashboard aggregations, quiz time tracking           |
| Nov 2025 – Jan 2026 | Attendance Automation Lambda     | Selenium → Puppeteer rewrite, Secrets Manager integration                                       |
| Dec 2025            | Get Client Info Lambda           | Presigned uploads, SNS mobile push, PII redaction                                               |
| Oct – Nov 2025      | Client Management SPA            | Apply-filter feature, pagination correctness, Zod domain models                                 |
| Sep 2025            | Declaration Form SPA             | 8-question adaptive form, Mantine/shadcn inputs, validation pipeline                            |

---

_Last updated: May 2026 | 11 projects documented_
