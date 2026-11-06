# ART-146 — Kaizen capability and coupling audit

**Audit date:** 2026-11-06
**Kaizen evidence revision:** `44101ef2fc772e0903ff926247a01597a439a2a3` (`origin/main`, 2026-11-05)
**Target repository:** `https://github.com/duncandeveloper/RiverThamesArtSchool`
**Linear project:** P-ART-85 — Franchise Ecosystem — Vanilla Club Platform
**Reference tenant working name:** RiverThames Art School

## Decision summary

Kaizen is a mature, tightly integrated single-organisation platform, not a safe copy-and-rename template. Its strongest reusable assets are the security, authentication, capability-RBAC, audit, compliance, content, media, publishing, Stripe, transactional-email, telemetry and Azure deployment patterns. Its weakest reuse boundary is the operational domain: `Guardian` owns `Member`; `Booking` is a free trial against a weekly `ScheduledClass`; membership and term coverage are family/member-centric; public content and infrastructure contain extensive Kaizen identity; martial-arts library, grading and licence concepts are first-class.

The evidence supports the working architecture: **one versioned reusable core + explicit vertical packs + isolated per-organisation deployments**. It does not support a permanent Kaizen fork or a shared-database SaaS conversion at this stage.

## Evidence and method

- The canonical Kaizen checkout was dirty and its local branch was 173 commits behind `origin/main`. No audit conclusion is based on that stale branch.
- A clean detached worktree was created at `44101ef2` and used read-only.
- Repository instructions and the required handover/ADR documents were read before inspection.
- Web routes, API routers, mounted router objects, endpoint decorators, services, schemas, ORM tables, Alembic migrations, jobs, providers, Bicep modules, workflows, package configuration and hard-coded tenant markers were counted from source.
- The live site, `https://kaizenkaikan.fit`, its sitemap, robots file and build identity were checked on 2026-11-06. No personal data or secret value was accessed.
- Azure was inspected read-only through the authenticated CLI. No resource, role, secret or configuration value was changed.
- Linear P-ART-85, P-ART-86, ART-132 and ART-144 through ART-205 where relevant were cross-checked. Existing issues are used below; no duplicate ticket is proposed.

## Verified implementation size and handover drift

| Surface | Code-backed count | Documentation drift |
|---|---:|---|
| Next.js page files | 70 | Handover route examples cover only the early public/admin subset. |
| Next.js layouts | 9 | Current nested auth/admin/portal/content layouts are not fully enumerated in handover. |
| Next.js route handlers | 1 | `video-sitemap.xml/route.ts` is omitted from the old route map. |
| FastAPI router modules | 45 | `AGENTS.md` says approximately 35. |
| Mounted router objects | 62 | Multiple modules mount public/admin/member/safeguarding variants. |
| Router HTTP operation decorators | 358 | Confirms a much larger API surface than the four-router handover example. |
| Generated OpenAPI paths / operations | 300 / 362 | The four additional operations are application-level health/readiness/root/CSP surfaces. |
| Service modules | 81 | Handover describes only the original router→service shape. |
| Schema modules | 41 | Generated shared types cover a substantially expanded contract. |
| ORM tables | 118 | `AGENTS.md`/handover still describe approximately 53 tables. |
| Alembic migrations | 74 | The handover tree still illustrates migrations `0001`–`0003`. |
| API test files | 141 | Security/compliance/import/billing coverage is materially broader than the overview suggests. |
| Web unit/E2E files | 40 | Current auth, portal, import, timetable and compliance journeys exceed the overview. |
| Bicep files | 27 | Current private-data-plane, DSAR and video-job modules exceed the original module list. |
| GitHub workflows | 5 | Current count agrees with `AGENTS.md`; older handover passages mention fewer. |

Two further drift points matter to extraction:

1. The overview says the video transcode worker is not built, but current source and Azure contain video ingest/transcode workers and Container Apps Jobs. Deployment/readiness still needs verification, but it is no longer an absent module.
2. The data-model handover says most tables carry `legacy_nest_id`; current source has it only on `Location`, `ScheduledClass`, `Grading` and `Licence`. NEST coupling is now concentrated, while legacy Guardian/Member ownership remains widespread through foreign keys and workflows.

## Classification legend

- **CORE** — reusable without tenant or discipline semantics.
- **CONFIGURABLE CORE** — reusable pattern with Kaizen identity, vocabulary, policy, provider or operational assumptions that must move to validated configuration or a neutral model.
- **MARTIAL-ARTS VERTICAL** — grading, licence, syllabus, technique, kata, discipline-specific research/content or similar functionality excluded from RiverThames.
- **ART VERTICAL GAP** — required art-school capability not supplied by Kaizen's current domain.
- **DEFERRED** — intentionally outside the first RiverThames MVP.

## Capability map

| Capability | Kaizen implementation evidence | Classification | RiverThames need | Action | Existing ART ticket |
|---|---|---|---|---|---|
| Password authentication, verification and reset | `routers/auth.py`, `services/auth_app.py`, `Account`, `AuthSession`, `EmailToken` | CORE | Yes | Extract with neutral package/config names and retain denial tests. | ART-154 |
| Revocable sessions and recent-auth step-up | `auth.py`, `services/auth_app.py`, `AuthSession`; server-side idle/absolute bounds | CORE | Yes | Preserve security invariants; reassess browser bearer transport separately. | ART-154, ART-172 |
| TOTP MFA and recovery | `routers/account.py`, `mfa_compliance.py`, `mfa_recovery.py`; Key Vault-backed TOTP | CORE | Yes | Reuse pattern, but configure new vault/key/owners and rollout evidence. | ART-154, ART-170, ART-172 |
| Capability-based RBAC | 17 capabilities in `services/rbac.py`; site admin/editor/instructor/moderator/billing roles plus delegated roles | CONFIGURABLE CORE | Yes | Keep capability enforcement; replace club roles with operator, teacher, assistant, DSL/safeguarding, billing and learner-aware matrix. | ART-154 |
| Canonical person foundation | `Person`, encrypted-profile scaffolding, relationships and immutable decision events | CORE | Yes | Reuse concepts and authorization patterns, not the live schema/migrations. | ART-151, ART-153, ART-156 |
| Legacy Guardian/Member family model | Mandatory `Member.guardian_id`; duplicate plaintext identity fields | CONFIGURABLE CORE | Partly | Do not migrate as the new baseline; design non-exclusive Person capabilities and adult self-enrolment greenfield. | ART-151, ART-153 |
| Relationship authority and restrictions | `PersonRelationship`, events, age/authority and legacy-compatibility services | CORE | Yes | Retain verified/scoped/time-bounded authority and denial tests; remove fuzzy/legacy ownership as the primary path. | ART-153 |
| Staff credential assurance | Instructor, credential type, credential, expiry notification and assignment gates | CONFIGURABLE CORE | Yes | Generalise instructor terminology, audience requirements, DBS/training/first-aid policy. | ART-155 |
| Public timetable | `Location` + `Discipline` + weekly `ScheduledClass`, closures and date overrides | CONFIGURABLE CORE | Yes | Reuse rendering/override lessons; replace the domain with Programme→Offering→Session and flexible calendars. | ART-158, ART-159, ART-164 |
| Summer timetable presentation | `SummerTimetableSeason` and admin/public routers | CONFIGURABLE CORE | Maybe | Generalise to arbitrary seasonal/holiday schedules or defer presentation layer. | ART-158, ART-164 |
| Venue profiles and maps | Location profiles, static `VENUES`, publishable transport/parking data | CONFIGURABLE CORE | Yes | Add rooms, accessibility, facilities, resources, risk state and conflict rules; remove duplicate static venue data. | ART-159 |
| Free-trial booking | `Booking` links Guardian, Member and ScheduledClass; waiver; requested/confirmed/cancelled | CONFIGURABLE CORE | Yes | Retain transactional/idempotency patterns, but replace fixed free-trial/family/class semantics. | ART-153, ART-158, ART-160 |
| Capacity, waitlist and attendance | `ClassWaitlistEntry`, `AttendanceRecord`, booking-capacity and attendance services | CONFIGURABLE CORE | Yes | Rebind to offering/session/enrolment; add holds, expiring offers, broader register states and authorised collection. | ART-160, ART-163 |
| Courses, workshops, drop-ins and school clubs | No neutral Programme/Course Offering/Session aggregate | ART VERTICAL GAP | Yes | Greenfield model; do not force these into Discipline/ScheduledClass. | ART-158 |
| Room/equipment/material constraints | No room/resource inventory or scheduling conflict aggregate | ART VERTICAL GAP | Yes | Add venue rooms, resources and conflict checks. | ART-159, ART-163 |
| Teacher rota and substitution | One optional `ScheduledClass.instructor_id`; no multi-staff rota/ratio model | ART VERTICAL GAP | Yes | Add teacher/assistant assignments, availability, ratios and substitutions. | ART-155, ART-163 |
| Authorised collection | No complete collection/release workflow | ART VERTICAL GAP | Child delivery | Add collector/independent-travel authority and escalation without exposing private details on general registers. | ART-153, ART-163 |
| Stripe Checkout, portal and signed webhooks | `checkout.py`, billing routers/services, subscription and webhook ledger | CORE | Yes | Reuse provider boundary/idempotency; create separate Stripe account/config/catalogue and never copy identifiers. | ART-161, ART-162, ART-171 |
| Membership offers and entitlements | Versioned deals, family agreement, per-member entitlement, adjustments | CONFIGURABLE CORE | Partly | Reuse immutable price/evidence patterns; remodel offers/enrolments for adult learners, blocks, packs, one-offs and materials. | ART-151, ART-161, ART-162 |
| Named term calendar | `TermPeriod` constrained to autumn/spring/summer with contiguous coverage assumptions | CONFIGURABLE CORE | Partly | Replace fixed names/coverage semantics with arbitrary course calendars and configurable commercial policies. | ART-158, ART-161 |
| Refund/reconciliation ledger | Payment transactions, linkage, disputes, reminders and append-only evidence | CORE | Yes | Reuse with organisation-isolated provider namespace and neutral offer/enrolment links. | ART-162 |
| Transactional email | Provider abstraction for console, Mailpit, Resend and ACS; idempotent communication records | CONFIGURABLE CORE | Yes | Retain adapter/outbox policy; configure new sender/domain/allow-list/provider account. | ART-166, ART-171 |
| CMS, blog and publishing workflow | Blog CMS, revisions, publish/schedule, admin editor | CORE | Yes | Extract core; remove Kaizen content and add tenant content-quality/rights gates. | ART-165 |
| Media and gallery | Blob/local storage, EXIF stripping, variants, galleries, consent gates | CONFIGURABLE CORE | Yes | Reuse pipeline; configure storage and extend purpose/rights metadata for artwork and learner projects. | ART-157, ART-165 |
| Video ingest/transcode/streaming | Video rows, services, workers, Blob/Queue/Event Grid/Jobs | DEFERRED | Not MVP | Keep as optional module; no job, route or data collection when disabled. | ART-150; no new issue required |
| SEO, redirects and sitemap | SEO records, redirect/suggested-404 services, metadata, static/dynamic sitemap | CONFIGURABLE CORE | Yes | Reuse engine; replace domains, local-area content and Kaizen defaults. Fix broken library sitemap outputs before extraction. | ART-149, ART-164, ART-165 |
| Local-area discovery pages | Hard-coded towns, martial-art offers, venues and class times in `lib/area-pages.ts` | CONFIGURABLE CORE | Yes | Replace with validated site/programme/location configuration; never copy claims. | ART-149, ART-164 |
| Public intake/contact actions | Public intake, conversions, action tokens, suppression and communication effects | CORE | Yes | Reuse privacy-safe workflow patterns as the CRM foundation. | ART-166 |
| CRM/trial stages | Booking pipeline plus public-intake conversion, but not one neutral lead lifecycle | CONFIGURABLE CORE | Yes | Unify enquiry/trial/waitlist/enrolment stages and separate service from marketing preferences. | ART-160, ART-166 |
| Member/guardian portal | Portal dashboard, bookings, billing and library paths | CONFIGURABLE CORE | Yes | Rebuild presentation/queries around learner/payer/authority capabilities rather than Guardian ownership. | ART-153, ART-160, ART-161 |
| Member directory/import/reconciliation | Legacy member CRUD, encrypted draft import, Person reconciliation | DEFERRED | Only synthetic/approved imports | Preserve generic validation/idempotency lessons; do not carry NEST/Kaizen migration UI into MVP. | ART-151, ART-168 |
| Consent and media consent | Versioned consent text/records; per-channel child media decisions | CONFIGURABLE CORE | Yes | Keep default-deny/version/withdrawal patterns; add artwork/name attribution and controller-approved purposes. | ART-157 |
| DSR, retention and erasure | DSR cases/events, retention services, gated erasure worker, exports | CORE | Yes | Extract with module/store coverage and independent controller decisions. | ART-156, ART-168, ART-173, CAS ART-197 |
| Data-protection complaints | Separate complaint case/event model with restricted handling | CORE | Yes, subject to policy | Keep separate from DSR/safeguarding; RiverThames controller must approve deadlines, retention and roles. | ART-154, ART-168, CAS ART-197 |
| Audit logging | Append-only structured audit, recent-auth no-store access | CORE | Yes | Reuse closed vocabulary/opaque identifiers; map new operations and resources. | ART-154, ART-172 |
| Safe telemetry and analytics | Allow-listed server events/metrics; consented public-route GA4 | CONFIGURABLE CORE | Yes | Reuse fail-closed approach; remove hard-coded Kaizen GA4 page values and create isolated telemetry. | ART-149, ART-152, ART-167, ART-170 |
| Forums, reactions and community moderation | Forum/category/thread/post/subscription/reaction/moderation tables and routes | DEFERRED | No first MVP need | Disable as a complete module: no routes, nav, jobs, permissions or data. | ART-144, ART-150 |
| Martial disciplines | Discipline catalogue and discipline-driven navigation/content | MARTIAL-ARTS VERTICAL | No | Keep in Kaizen vertical; RiverThames uses art programme/medium/skill semantics. | ART-150, ART-158, ART-169 |
| Grading and licences | `Grading`, `Licence`, portal/admin UI and reminders | MARTIAL-ARTS VERTICAL | No | Exclude schema, API, UI, reminders and seeds from art deployment. | ART-150 |
| Technique/syllabus library | Styles, grades, techniques, relationships, syllabus collections, videos and AI drafts | MARTIAL-ARTS VERTICAL | No | Retain only in martial pack. Do not rename grades to art levels. | ART-150 |
| Kata/martial research corpus | 68 files in `docs/research/karate/kyokushin` | MARTIAL-ARTS VERTICAL | No | Do not copy to generic or art repository. | ART-145, ART-150 |
| Art curriculum/projects/media | No art-native project, medium, skill, material or artwork-rights model | ART VERTICAL GAP | Yes | Create original/synthetic art pack after core domain is stable. | ART-157, ART-169 |
| Adult-only life drawing | No separated eligibility, model, privacy or room-control workflow | ART VERTICAL GAP / DEFERRED GATE | Optional, disabled | Seed only a disabled synthetic example after legal/safeguarding approval; test strict separation. | ART-169, ART-172, CAS ART-185 |
| Privacy-safe reporting | Operational data exists; no complete art occupancy/funnel/margin dashboard | ART VERTICAL GAP | Yes | Build closed, role-scoped aggregates with small-cohort protection. | ART-167 |
| Backup/restore and portable import/export | Kaizen provider/backup register and DSR exports; no generic per-org import framework | CONFIGURABLE CORE / GAP | Yes | Create controlled synthetic import/export and prove isolated restores. | ART-168, ART-173 |
| Azure IaC and OIDC delivery | Bicep/azd, managed identity, Key Vault, CA/SWA/Postgres/Storage/telemetry, SHA-pinned Actions | CONFIGURABLE CORE | Yes | Parameterise organisation/repo/resource/domain/provider identity and preserve cost/security gates. | ART-152, ART-170 |

## Hard-coded coupling inventory

The coupling is structural, not cosmetic. A first-party scan (excluding generated OpenAPI/types and dependency folders) found Kaizen identity in 198 files, martial terminology in 111 files, exact Kaizen domains in 19 files, Kaizen Azure/repository names in 28 files and Kaizen-domain email examples/configuration in 22 files. The broad `kia` count is inflated by Linear ticket references, so it is not used as a replacement estimate.

Highest-risk coupling points:

- package/workspace identities: `kiazen-ecosystem`, `@kaizen/web`, `kaizen-api`, `@kaizen/shared`;
- `apps/web/lib/site.ts`: brand, phone, navigation and five tenant venues;
- `apps/web/lib/content.ts`, public pages, image paths and legal pages: Kaizen copy and claims;
- `apps/web/lib/area-pages.ts`: six towns, two martial offers, venue/class schedules and SEO claims;
- `apps/web/components/analytics.tsx`: hard-coded Kaizen page location/title in GA initialisation;
- `apps/web/app/timetable/page.tsx`: two direct Kaizen Stripe payment links;
- API seeds: venues, disciplines, instructors, events, waiver/consent, membership catalogue and martial library;
- `infra/main.bicep`: `kia-*` naming, `app=kaizen`, database `kaizen`, `kiaadmin`, Kaizen image repository, domains and sender identities;
- GitHub deployment workflows: fixed resource group, app/job/vault/storage/SWA names and Kaizen origins;
- `scripts/setup-oidc.ps1`: fixed repository, Entra app and resource group;
- local Compose and examples: Kaizen database/user/package identities.

These references belong under validated organisation/site/brand configuration (ART-149), vertical manifests (ART-150), or per-organisation infrastructure parameters (ART-152). A text replacement is unsafe because it would leave the legacy data relationships and enabled modules intact.

## Material that must not migrate automatically

- any Kaizen member, guardian, account, payment, attendance, consent, safeguarding, complaint, audit or communication data;
- `.env` files, `.azure` state, Key Vault values, connection strings, signing/encryption material and local artefacts;
- Stripe customer/subscription/payment identifiers, product/price IDs, webhook secrets and the direct live payment links;
- provider identifiers, mail allow-lists, email recipients and live sender configuration;
- Kaizen legal, waiver, privacy, terms, cookie and safeguarding wording;
- biographies, phone/address content, venue facts, local SEO claims and tenant configuration;
- the 12 Kaizen public images, logos and instructor/class media unless separately licensed for the new controller (default: exclude);
- all 68 martial research files, syllabus/kata data, grading/licence material and martial seeds;
- any Cygnets/RBL/customer/business files in the outer local project directory, including the plainly named credential file and invoice/customer-looking export. These were not opened and are already covered by ART-179.

Synthetic fixtures must use unmistakably fictitious adults, minors, staff, bookings, payments and media. Kaizen's compliance approvals do not transfer to RiverThames.

## Target repository condition

At the start of this audit, the supplied GitHub repository was public and completely empty: no default branch, commits, files, application code, infrastructure or `AGENTS.md`. It was created on 2026-11-06. A local clone now exists at `D:\dev-main\Carrera-cygnets-too-indendant-Ecosystem\RiverThamesArtSchool`.

The originally supplied outer path is not a Git repository and contains business records plus a separate, clean nested `duncandeveloper/teddington-rbl-club` repository. Linear P-ART-85 explicitly says the RBL pop-up is a separate workstream, so no ART files were written there. Existing outer-folder sensitive-looking files were not opened, moved or copied.

The already-created empty GitHub repository creates a governance mismatch: ART-145 says ownership, licensing and contribution/commercial-use rules must be approved before repository creation; ART-147 selects the extraction architecture; ART-148 owns the sanitised bootstrap. This audit writes documentation only. It does not copy Kaizen code, initialise infrastructure, select a licence or claim ART-148 completion. The repository's public/private setting and licence remain human decisions under ART-145.

## Audit conclusion

The reusable value is substantial, but reuse should occur by intentional extraction into a clean baseline, not history-copying the Kaizen repository. Security/compliance/provider patterns are closest to reusable. Public experience is reusable after configuration. People, scheduling, booking and commerce require neutral greenfield aggregates that reuse invariants and tests rather than Kaizen's tables. Martial library/grading/licensing/research remains a Kaizen vertical. Art curriculum and delivery concepts are a new art vertical built only after core boundaries are approved.
