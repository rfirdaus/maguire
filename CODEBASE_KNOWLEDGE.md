# Maguire / Tendigi codebase knowledge base

Last verified: 17 August 2026

This is a living engineering map of the current prototype. It records what the code actually does, where each kind of data lives, and which product behaviors are only simulated. It is not a product specification.

## Executive summary

- The active product brand is **Tendigi**. The folder name `maguire`, [archive/index.html](archive/index.html), and [dashboard-preview.html](dashboard-preview.html) preserve the older **Maguire.id** name.
- This is a browser-only static prototype: 17 HTML files and 3 CSS files, with page-specific JavaScript embedded directly in each HTML page.
- There is no framework, package manifest, build step, backend, database, API client, test suite, deployment configuration, or shared JavaScript module.
- The only shared raster asset is the transparent Tendigi wordmark at `assets/tendigi-logo.png`. Icons are inline SVG/text, and `Inter` is requested as a font-family without loading a webfont, so the browser normally uses a system fallback.
- Data and most domain behavior are hard-coded per page. `localStorage` supplies limited persistence and acts as a demo session/store, but it is not a coherent application data model.
- Only one tender has a complete journey: **Pengadaan Pembangunan Gardu Induk 150kV** for PT PLN. Other tender links eventually render the same PLN detail/workspace because query parameters are not read.
- The scan found no Git metadata in this directory, so this snapshot is not currently a Git working tree.

## Product and demo context

Tendigi is positioned as tender intelligence and preparation software for Indonesian business-development, procurement, legal/compliance, engineering, finance, and delivery teams. The intended product loop is discovery, preliminary company matching, tender-document analysis, requirement traceability, readiness decision support, and collaborative submission preparation.

The demo identity is fixed throughout the markup:

- User: **Budi Santoso**
- Workspace access role: **Superadmin** (protected current-user account with all permissions)
- Company: **PT Wijaya Infrastruktur Nusantara**
- Primary tender ID: `pln-150kv`
- Primary tender: **Pengadaan Pembangunan Gardu Induk 150kV**
- Buyer: **PT PLN (Persero)**
- Estimated value: **Rp48.2B**
- Submission deadline: **21 Aug 2026**
- Preliminary match: **94%**
- Fixed readiness result: **82%, At Risk**
- Tender package: **4 files / 184 pages**
- Requirement summary: **44 total / 38 matched / 3 attention / 3 gaps**
- Mandatory eligibility gaps: third qualifying project and Project Manager SKA

Marketing claims about AI document analysis, encryption, organization isolation, and audit trails describe product direction. The prototype now has a locally persisted role/permission editor, but it does not enforce authorization outside the browser UI or provide server-side security.

## Runtime model

Serve the folder with any static HTTP server and open `index.html`. For example:

```sh
python3 -m http.server 8000
```

Then open `http://localhost:8000/`. Serving over HTTP is preferable to opening `file://` URLs because browser handling of `localStorage` for local files varies.

Demo credentials:

```text
demo@tendigi.id
demo123
```

No network requests are made after pages load: there are no `fetch`, XHR, WebSocket, dynamic import, external script, or external stylesheet calls. A strict Content Security Policy would require refactoring because every active application page uses inline JavaScript, and the archived landing page also uses inline CSS.

## Repository map

### Public and session entry points

| File | Responsibility | Real behavior |
| --- | --- | --- |
| [index.html](index.html) | Current Tendigi marketing site | Static sections, anchor navigation, mail links, and login link; no JavaScript |
| [login.html](login.html) | Demo sign-in | Validates one hard-coded credential pair, writes demo keys, and routes to Dashboard |
| [dashboard.html](dashboard.html) | Authenticated overview | Static KPIs, recommendations, deadlines, and pipeline; only greeting and shared shell controls are dynamic |

### Core tender journey

| File | Responsibility | Real behavior |
| --- | --- | --- |
| [discover.html](discover.html) | Opportunity discovery | Renders 14 inline tender records; search, category/location/deadline filters, sorting, tabs, and saved state work locally |
| [tender-detail.html](tender-detail.html) | Opportunity review | Static PLN detail, save toggle, optional Analyze transition, and direct Participate confirmation that creates a workspace without analysis state |
| [analysis.html](analysis.html) | Analysis process | Simulates four phases with a timer, animated counters, fixed findings, and cached completion state |
| [readiness.html](readiness.html) | Decision-level result | Static 82%/At Risk result, source-reference modals, and Participate confirmation |
| [requirements.html](requirements.html) | Detailed requirement traceability | Contains the fullest 44-item requirement dataset; live search/filtering, drawer, and source preview |
| [workspace.html](workspace.html) | Submission preparation | Six hash-addressable tabs, task persistence/filtering, and a mode-aware direct-participation state that starts without AI-derived readiness/requirement/document results |
| [my-tenders.html](my-tenders.html) | Tender portfolio | Seven inline records with search, status/owner filtering, sorting, summary modal, and a direct-participation override for the PLN row |

### Evidence, organization, and support

| File | Responsibility | Real behavior |
| --- | --- | --- |
| [documents.html](documents.html) | Reusable evidence library | 18 seeded metadata records, including three explicitly attributed to Budi Santoso; current-user uploads support confirmed deletion while other seeded records remain protected |
| [company-profile.html](company-profile.html) | Structured company profile | Hash tabs; company edits and added projects persist; added capabilities last only until reload |
| [notifications.html](notifications.html) | Notification inbox | Nine seeded notifications, search/type/status filtering, and locally persisted read IDs |
| [settings.html](settings.html) | Workspace/account preferences and access administration | Five hash sections, auto-saved preferences, persisted member/role management, role assignment, and an 11-permission access editor; security/data operations remain simulated |

### Development and historical artifacts

| File | Meaning |
| --- | --- |
| [analysis-preview.html](analysis-preview.html) | Same as `analysis.html` except the auth redirect is deliberately disabled; no page links to it |
| [dashboard-preview.html](dashboard-preview.html) | Older Maguire.id-branded Dashboard preview; no auth guard and stale `maguire_demo_*` logout keys |
| [archive/index.html](archive/index.html) | Older self-contained Maguire.id marketing page with approximately 680 lines of embedded CSS |

No active page links to these artifacts, but they will still be publicly addressable if the whole directory is deployed unchanged.

### Stylesheets

| File | Responsibility |
| --- | --- |
| [css/styles.css](css/styles.css) | Global reset, design tokens, typography, buttons, cards, common grids, and utility colors |
| [css/marketing.css](css/marketing.css) | Current public marketing-page layouts and responsive behavior |
| [css/app.css](css/app.css) | Authentication, shared application shell, and all authenticated page styles; 7,649 lines organized mostly in page-order sections plus a final shared readability/spacing layer |

## Intended navigation

```text
index -> login -> dashboard
                     |
                     +-> discover -> tender detail -> participate directly -> workspace
                                         |                                      |
                                         +-> optional analysis -> readiness     +-> optional analysis
                                                                  |
                                                                  +-> requirements
                                                                  +-> participate -> workspace
                     +-> my tenders ------------------------------------------------------+
                     +-> documents
                     +-> company profile
                     +-> notifications
                     +-> settings
```

The conceptual product says Company Profile and Documents power discovery/readiness. In the implementation, those pages do not recompute discovery scores, analysis findings, readiness, or requirements. Their data remains isolated except for the narrow integrations listed below.

## JavaScript and data architecture

Every interactive page follows roughly the same pattern:

1. Render nearly all markup up front.
2. Run an end-of-body auth check on normal application pages.
3. Declare page-specific arrays/objects inline.
4. Read any relevant `localStorage` keys.
5. Render dynamic lists with template strings and attach event listeners.
6. Duplicate the user-menu, notification-popover, mobile-sidebar, and logout handlers.

There is no central router or store. Hashes are interpreted manually by Workspace, Settings, and Company Profile. No page reads `location.search`, so `?tender=...` is currently decorative.

### Current data sources of record

These are the most detailed copies, not guaranteed cross-page canonical sources:

- Tender discovery records: `const tenders` in [discover.html](discover.html)
- PLN tender detail/package/timeline: markup in [tender-detail.html](tender-detail.html)
- Requirement records: `const requirements` in [requirements.html](requirements.html)
- Workspace tasks: `const tasks` in [workspace.html](workspace.html)
- Portfolio rows: `const tenders` in [my-tenders.html](my-tenders.html)
- Evidence records: `const baseDocuments` in [documents.html](documents.html)
- Company project history: `const defaultProjects` in [company-profile.html](company-profile.html)
- Notification feed: `const notifications` in [notifications.html](notifications.html)
- Preference defaults: `const defaults` in [settings.html](settings.html)
- Organization member/role seeds: `const defaultAccessState` in [settings.html](settings.html)
- Role permission catalog and dependency rules: `const permissionGroups` and `const permissionDependencies` in [settings.html](settings.html)

The 44 requirement objects break down as follows:

| Category | Total | Matched | Attention | Gap |
| --- | ---: | ---: | ---: | ---: |
| Administrative | 9 | 9 | 0 | 0 |
| Legal | 7 | 7 | 0 | 0 |
| Technical | 13 | 10 | 1 | 2 |
| Financial | 8 | 6 | 2 | 0 |
| Experience | 7 | 6 | 0 | 1 |
| **Total** | **44** | **38** | **3** | **3** |

Thirty-seven requirements are marked mandatory and seven optional. The 82% readiness score is a fixed value; the code contains no formula deriving it from these records.

Other useful seed counts:

- Discovery: 14 rendered opportunities, 12 marked recommended
- My Tenders: 7 portfolio records
- Documents: 18 seed records (15 ready, 1 expiring, 2 review); three Project Experience records are tagged as Budi Santoso demo uploads and can be deleted
- Workspace: 19 tasks in analyzed mode (14 initially complete, 5 open); 16 initially open tasks in direct mode because the analysis task and two analysis-derived gap tasks stay hidden until analysis
- Notifications: 9 feed records (1 deliberately unread)
- Team & Access: 6 active members and 6 roles; Budi is the only protected Superadmin, and Sales, Finance, Legal, Compliance, and Engineering are assignable roles

## Browser persistence

All state is origin-global and unscoped by user, company, or tender. The historical `octen_*` key prefix is intentionally retained after the Tendigi rebrand so existing browser state is not orphaned.

| Key | Format and responsibility | Actual consumers / caveats |
| --- | --- | --- |
| `octen_demo_authenticated` | String `"true"` demo auth flag | Read by normal app pages; trivial to forge and not access control |
| `octen_demo_user`, `octen_demo_company` | Demo identity strings | Written by Login and removed on logout, but displayed names are hard-coded instead of reading them |
| `octen_cached_email`, `octen_cached_password` | Remembered credentials | Login only; password is stored in plaintext when Remember is checked |
| `octen_saved_tenders` | JSON array of tender IDs | Shared only by Discover and PLN Tender Detail |
| `octen_active_tender`, `octen_analysis_tender` | Tender pointer strings | Repeatedly overwritten with `pln-150kv`; not used as routing sources |
| `octen_analysis_status`, `octen_analysis_status_cached` | `started`/`completed` state | Written by the Analysis animation/replay flow; Readiness requires completed state instead of creating it |
| `octen_readiness_score`, `octen_requirement_summary` | Fixed result values | Written by Analysis but not used to generate the result views |
| `octen_tender_participation`, `octen_workspace_tender` | Fixed PLN participation pointers | Written only by the Tender Detail direct confirmation or Readiness participation confirmation; Workspace validates the pair before loading |
| `octen_participation_mode` | String `direct` or `analyzed` | Distinguishes a workspace created before AI analysis from the analyzed readiness route; Tender Detail writes it, Analysis/Readiness enforce the analyzed state after completion, and Workspace/My Tenders use it to suppress premature AI-derived results |
| `octen_workspace_tasks` | JSON object of task-ID booleans | Workspace task completion only |
| `octen_workspace_tasks_direct` | JSON object of task-ID booleans | Isolates direct/manual task progress; merged into the analyzed task state after optional analysis completes |
| `octen_workspace_tab` | Tab name string | Workspace only; mirrored to the URL hash |
| `octen_uploaded_documents` | JSON array of browser-created document metadata | Documents only; uploads can be deleted with confirmation and no file bytes are selected or stored |
| `octen_deleted_document_ids` | JSON array of deleted seeded-document IDs | Documents only; sanitized to the three seeded Project Experience records explicitly attributed to Budi Santoso, so all other seeded evidence remains protected |
| `octen_profile_company` | JSON object of editable fields | Company Profile only; updates only selected profile fields |
| `octen_profile_projects` | JSON project array | Company Profile only |
| `octen_notification_read_ids` | JSON array of IDs | Notifications and the Settings shell badge; most other page badges ignore it |
| `octen_settings_v1` | JSON preference object | Settings; Notifications reads a few channel values for its summary |
| `octen_team_access_v1` | Revisioned JSON object containing roles, permission IDs, and active members | Settings only; seeded with 6 members/6 roles, sanitized/capped on read, always restores Budi as the exclusive protected Superadmin, and rejects stale whole-state writes from another tab |
| `maguire_demo_authenticated`, `maguire_demo_user`, `maguire_demo_company` | Stale legacy keys | Only removed by `dashboard-preview.html`; no current page writes them |

Logout removes only the three `octen_demo_*` identity/auth keys. Saved tenders, cached credentials, analysis state, tasks, profile data, document metadata, notifications, settings, and team-access configuration survive logout.

## What is genuinely interactive

- Client-side search, filters, sorts, tabs, empty states, and save/read toggles
- Direct tender participation from Tender Detail, including confirmation, persisted workspace state, and an analysis-optional workspace mode
- Analysis progress animation and cached replay state
- Requirement and document detail drawers/modals
- Workspace task completion and last-tab persistence
- Company field edits and project additions
- Document metadata additions and confirmed deletion of browser-created or explicitly Budi-owned demo uploads
- Settings autosave; adding/removing members; assigning roles; creating/editing/deleting roles; and managing role permissions, including atomic reassignment when deleting an in-use role
- Responsive off-canvas navigation

## What is simulated or disconnected

- Authentication, server-enforced authorization, real organization invitations, and sessions
- Tender aggregation, source portal links, and query-based tender routing
- PDF/file upload, storage, preview, replacement, parsing, and source retrieval
- AI analysis and the workspace assistant
- Readiness calculation and eligibility evaluation
- Recalculation after company, project, personnel, or document changes
- Password changes, exports, 2FA, retention, and workspace deletion
- Submission lifecycle and non-PLN tender workspaces
- Dynamic deadlines, match scores, notification generation, and dashboard KPIs

## Styling and UI conventions

- Global variables live in `css/styles.css`; the core palette is slate/navy with `#2563eb` blue, green/amber/red statuses, white cards, and a light slate background.
- Shared UI tokens include a 12px compact-caption floor, 14px metadata, 16px control text, a 44px minimum control target, and a 4/8/12/16/24/32px spacing scale. The public site retains an 18px body size.
- `css/app.css` is loaded after `css/styles.css` and owns all authenticated UI. Its major sections are Authentication/Shell, Dashboard, Discovery, Tender Detail, Analysis, Readiness, Requirements, Workspace, My Tenders, Documents, Company Profile, Notifications, and Settings.
- A final `READABILITY + SPACING SYSTEM` section deliberately overrides older page-local density rules. Keep shared floors and breakpoint corrections there unless a component genuinely requires a page-specific exception.
- Some shared navigation badges, popovers, mobile-menu, and overlay rules are located inside the stylesheet's Dashboard section even though every authenticated page depends on them.
- Authenticated pages use a fixed 248px desktop sidebar and sticky top bar above 1,024px. At 1,024px and below the shell becomes an off-canvas sidebar controlled by duplicated page JavaScript; 760px and 520px remain phone-focused content breakpoints.
- Page-specific classes are usually namespaced (`discover-*`, `readiness-*`, `workspace-*`, and so on), which limits accidental selector collisions but makes the stylesheet large and page-coupled.
- Responsive behavior is implemented with many page-local breakpoints from 480px to 1,280px. Notable layout boundaries are Analysis at 1,200px, Tender Detail at 1,100px, Workspace dense rows at 1,050px, Settings navigation/access at 1,180px, My Tenders cards at 1,250px, Documents cards at 1,180px, and the shared shell at 1,024px. The marketing page mainly uses 900px and 600px.
- The code includes reduced-motion CSS, semantic headings, many accessible labels/live regions, and Escape/backdrop support for several dialogs. Focus trapping/restoration, menu keyboard handling, and complete tab semantics are not implemented consistently.
- At 900px and below the marketing navigation links disappear without a mobile-menu replacement. The off-canvas app sidebar is visually translated away but is not made `inert`, so closed links may remain in keyboard order.
- Active application CSS contains no explicit text declaration below 12px. Compact badges/captions may use 12px, meaningful metadata is generally 14px, primary controls/form fields use 16px, and primary headings use a 32/20/16px hierarchy. Meaningful text formerly using low-contrast `#94a3b8` now uses `--text-light` (`#64748b`).
- Buttons, form fields, navigation links, icon controls, tabs, and modal/drawer close controls use a 44px interaction baseline. Dense tables remain horizontally scrollable inside their own containers when their columns cannot collapse safely.
- The readability pass was computed-style checked across all 14 active public/authenticated pages at 320, 375, 500, 768, 1,024, 1,280, and 1,440px. The audit found no visible text below 12px, no tested primary control below 44px, and no page-level horizontal scrolling. Representative Dashboard, Workspace, My Tenders, Documents, Company Profile, Notifications, and Settings screens were also visually inspected at breakpoint boundaries.
- Font weights such as 750 and 850 are common, but no variable font file is supplied; synthesis and appearance can vary by platform. There is also no global skip link or authored `:focus-visible` style for ordinary links/buttons.

## Known integration gaps and data drift

Treat these as current facts when planning changes, not as intentional product rules.

1. **Only PLN is routed.** Discovery and Dashboard generate several `?tender=...` URLs, but Tender Detail and Workspace ignore the query and always render/set `pln-150kv`.
2. **Workflow state is enforced only for the PLN prototype.** Workspace requires an existing participation pair and Readiness requires completed analysis, but all keys remain origin-global and forgeable, and other tenders still have no real routed workflow.
3. **Profile/evidence updates do not affect results.** Adding a qualifying project or document cannot resolve a requirement, readiness risk, workspace warning, or KPI.
4. **Summary copies have drifted.** Readiness category totals differ from the 44-object requirement array, and Workspace’s unresolved items/source pages differ from Requirements.
5. **Portfolio identities conflict.** `jakarta-drainage` and `pertamina-substation` describe different values/statuses/scopes in Discover versus My Tenders despite reusing IDs.
6. **Certificate facts conflict.** Company Profile and Documents contain different validity dates/statuses for some certificates, including SMK3 and ISO 9001.
7. **Task completion is mostly isolated.** Direct-mode task completion updates its overall progress, but analyzed-mode fixed progress, team workloads, Documents, Readiness, and most cross-page KPIs do not recalculate.
8. **Notification state is inconsistent.** Notifications and Settings observe read IDs, while most page shells retain a hard-coded unread badge/popover. Initially-read notifications are re-seeded as read on reload.
9. **Deep-link support is uneven.** Workspace and Settings hashes work. Company Profile expects `#projects`, while several links use `#project-experience`; Documents does not interpret `#certifications` or `#financial` at all.
10. **Dates are snapshot text.** “Today,” “days remaining,” deadlines, update timestamps, and expiry warnings are mostly hard-coded around 8–10 Aug 2026 and will age.
11. **Auth is prototype-only.** The guard runs after markup, redirects without stopping the rest of the script, stores persistent plaintext demo credentials, and can be bypassed with localStorage or preview pages.
12. **Stored JSON is not uniformly defensive.** Most parsers catch invalid JSON, but Company Profile parses `octen_profile_company` without a guard, and Workspace can dereference a stored JSON `null` task state.
13. **Inactive artifacts and placeholder links can leak into deployment.** Preview pages bypass normal auth, while several marketing Roadmap/Solutions/About/Privacy/Terms links are only `#` anchors and Request Demo jumps to the footer rather than a form.
14. **Organization access and tender collaboration are separate datasets.** Settings manages a roster seeded with six organization accounts in `octen_team_access_v1`, while Workspace, My Tenders, document owners, and Company Profile personnel still use independent hard-coded people and do not enforce the configured permissions.

## Maintainability constraints

- Shared shell markup and JavaScript are copied into each authenticated page. A navigation, auth, popover, badge, logout, or mobile-menu change must currently be applied and checked across all of them.
- Domain facts are copied into page markup and multiple independent JavaScript arrays. Changing a tender, requirement, certificate, deadline, score, task, or person can require coordinated edits in several files.
- `css/app.css` is effectively append-only page CSS. Reuse existing tokens/primitives before adding another page-local variant, and check for earlier selectors before introducing generic names.
- Template-string rendering generally escapes variable content using page-local helpers. Preserve that behavior whenever values can originate from forms or storage.
- Treat Preview and Archive files as non-production artifacts unless a task explicitly includes them.
- The current active name is Tendigi. Do not propagate the legacy Maguire.id brand from the folder/archive/preview into active pages without an explicit rebrand decision.

## Safe verification checklist for future changes

1. Serve the directory over HTTP and test with both fresh and populated `localStorage`.
2. Verify public entry, valid/invalid login, unauthenticated redirects, and logout persistence expectations.
3. Walk the PLN path: Discover -> Detail -> Analysis -> Readiness -> Requirements -> Workspace.
4. Check any changed cross-page fact in Dashboard, Discover, Detail, Readiness, Requirements, Workspace, My Tenders, Documents, and Company Profile as applicable.
5. Test desktop plus 1,024px off-canvas-shell, 768px tablet, and 500px/phone layouts; also check any changed page-specific intermediate breakpoint.
6. Test keyboard Escape/backdrop behavior and `prefers-reduced-motion` for changed dialogs or animation.
7. Recheck relative links and dynamic hashes. Static local `href`/`src` targets all resolved at the time of this scan.
8. Remember that there is no automated regression suite; browser behavior must currently be verified manually.

## Scan notes

- Files inspected: all HTML and CSS files in the folder, including Preview and Archive artifacts.
- Static local `href` and `src` file targets were checked and resolve.
- No repository instruction file (`AGENTS.md`) was present.
- The initial scan added only this knowledge file. The 16 August 2026 readability pass subsequently updated the three active stylesheets and synchronized the 1,024px sidebar resize condition across normal application pages and the two preview pages.
- The 17 August 2026 Team & Access implementation was syntax-checked across all active inline scripts and its access-state normalization invariants were exercised in Node. The in-app browser runtime was unavailable for that change, so its new interaction/layout flow still needs a live click-through at the checklist widths above.
