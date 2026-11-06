# ART-146 — Kaizen route and API map

**Audit date:** 2026-11-06
**Kaizen evidence revision:** 44101ef2fc772e0903ff926247a01597a439a2a3 (origin/main, 2026-11-05)
**Reference tenant working name:** RiverThames Art School
**Related evidence:** kaizen-capability-audit.md and kaizen-data-model-audit.md

## Executive finding

Kaizen has a substantially larger delivery surface than the old handover route examples imply: 70 Next.js pages, 9 layouts, 1 Next.js route handler, 45 FastAPI router modules, 62 mounted router objects, 358 router operation decorators, 81 service modules and 4 application-level operational endpoints. Generated OpenAPI contains 300 paths and 362 operations.

The public/CMS, authentication, security, audit, media, publishing, redirect and provider-boundary patterns are strong extraction candidates. The web shell, portal, timetable, booking, member and billing surfaces require organisation and domain configuration. Grading, licence, martial syllabus/library and NEST compatibility must remain outside RiverThames. Community, reactions, moderation and the video pipeline are deferred for the first art-school MVP.

This is a source audit, not a contract to preserve every current URL. RiverThames routes should be selected through organisation and module configuration, with disabled modules absent from navigation, route registration, OpenAPI and background jobs.

## Evidence and counting rules

- Source root: D:\dev-main\Kiazen-Ecosystem-art-146-audit.
- All counts are from clean detached revision 44101ef2.
- Page count includes every apps/web/app/**/page.tsx.
- Router count includes every apps/api/app/routers/*.py module.
- Mount count is the 62 unconditional include_router calls in apps/api/app/main.py.
- Router operation count is the 358 get/post/put/patch/delete decorators in router modules.
- apps/api/app/main.py adds GET /, GET /healthz, GET /readyz and POST /api/v1/security/csp-report, producing 362 OpenAPI operations.
- OpenAPI collapses those operations onto 300 unique path templates because several mounted routers share paths with different methods or audiences.
- No secret or provider value is recorded here.

## Classification legend

- **CORE** — reusable without tenant or discipline semantics.
- **CONFIGURABLE CORE** — reusable mechanics containing Kaizen identity, terminology, policy, provider or operational assumptions.
- **MARTIAL-ARTS VERTICAL** — grading, licence, syllabus, technique, kata, NEST/franchise compatibility or other Kaizen-only functionality.
- **ART VERTICAL GAP** — a RiverThames capability not supplied by the current implementation.
- **DEFERRED** — intentionally outside the first RiverThames MVP.

## Complete web route inventory

Paths in the source column are relative to apps/web.

### Public and content routes — 35

| Route | Source | API or module dependency | Classification | RiverThames equivalent and action |
|---|---|---|---|---|
| / | app/page.tsx | Local lib/content.ts and lib/site.ts; root layout GET /api/v1/locations | CONFIGURABLE CORE | RiverThames homepage; replace all copy, assets, claims and calls to action. |
| /areas | app/areas/page.tsx | lib/area-pages.ts | CONFIGURABLE CORE | Local-discovery landing driven by validated town/programme configuration. |
| /areas/[town]/[offer] | app/areas/[town]/[offer]/page.tsx | lib/area-pages.ts | CONFIGURABLE CORE | Town + art-offer landing; current six towns, two martial offers and schedules do not migrate. |
| /blog | app/blog/page.tsx | lib/public-content.ts; GET /api/v1/blog and SEO resolution | CONFIGURABLE CORE | News/blog listing with RiverThames content. |
| /blog/[slug] | app/blog/[slug]/page.tsx | GET /api/v1/blog/{slug}, GET /api/v1/seo/resolved/{type}/{id}; reactions/report adjuncts | CONFIGURABLE CORE | Article detail; retain CMS/SEO, disable social adjuncts for MVP. |
| /blog/post | app/blog/post/page.tsx | Shared getBlogPost client plus reactions/reporting | CONFIGURABLE CORE | Static-host/runtime fallback for article detail, not a separate product concept. |
| /book | app/book/page.tsx | GET /api/v1/timetable, GET /api/v1/waiver/current, POST /api/v1/bookings | CONFIGURABLE CORE | Trial/enrolment journey; replace Guardian/Member/free-trial assumptions. |
| /build | app/build/page.tsx | Build identity/environment | CORE | Deployment diagnostic. |
| /community | app/community/page.tsx | lib/forums.ts: categories, threads and create-thread | DEFERRED | No first-MVP route. |
| /community/thread | app/community/thread/page.tsx | Forum posts, reactions and moderation report | DEFERRED | No first-MVP route. |
| /contact | app/contact/page.tsx | Local content only | CONFIGURABLE CORE | Contact page plus new enquiry/CRM form under ART-166. |
| /cookies | app/cookies/page.tsx | Static legal copy | CONFIGURABLE CORE | RiverThames controller-approved cookie notice. |
| /disciplines | app/disciplines/page.tsx | lib/content.ts | CONFIGURABLE CORE | Replace with /programmes; do not rename martial disciplines mechanically. |
| /disciplines/[slug] | app/disciplines/[slug]/page.tsx | lib/content.ts | CONFIGURABLE CORE | Replace with /programmes/[slug] backed by the new programme model. |
| /events | app/events/page.tsx | GET /api/v1/events plus public SEO/content services | CONFIGURABLE CORE | Workshops/events listing. |
| /events/[slug] | app/events/[slug]/page.tsx | GET /api/v1/events/{slug}, location profile and SEO; social adjuncts | CONFIGURABLE CORE | Workshop/event detail; disable reactions/reporting for MVP. |
| /grading | app/grading/page.tsx | Martial content | MARTIAL-ARTS VERTICAL | No RiverThames route. |
| /guide | app/guide/page.tsx | lib/content.ts | CONFIGURABLE CORE | First-visit/getting-started guide. |
| /instructors | app/instructors/page.tsx | GET /api/v1/instructors/profile-list | CONFIGURABLE CORE | Prefer /teachers. |
| /instructors/[slug] | app/instructors/[slug]/page.tsx | GET /api/v1/instructors/{slug}/profile and SEO | CONFIGURABLE CORE | Prefer /teachers/[slug]. |
| /library | app/library/page.tsx | lib/library.ts: taxonomy, techniques and collections | MARTIAL-ARTS VERTICAL | Current technique/syllabus library has no direct art equivalent. |
| /library/browse | app/library/browse/page.tsx | Same library APIs; static-host rewrite target | MARTIAL-ARTS VERTICAL | Do not expose; later art resources/projects need an original design. |
| /locations | app/locations/page.tsx | GET /api/v1/locations/profile-list | CONFIGURABLE CORE | Venues listing. |
| /locations/[slug] | app/locations/[slug]/page.tsx | GET /api/v1/locations/{slug}/profile and SEO | CONFIGURABLE CORE | Venue detail extended with rooms, accessibility and resources. |
| /membership | app/membership/page.tsx | GET /api/v1/billing/term-catalogue | CONFIGURABLE CORE | Pricing/offers; remodel for courses, blocks, workshops, packs and subscriptions. |
| /membership/cancel | app/membership/cancel/page.tsx | Checkout return state | CONFIGURABLE CORE | Payment cancellation return. |
| /membership/manage | app/membership/manage/page.tsx | Billing overview and customer-portal session | CONFIGURABLE CORE | Manage payments and enrolments. |
| /membership/success | app/membership/success/page.tsx | Checkout return state | CONFIGURABLE CORE | Payment success return; never activate entitlement from the redirect alone. |
| /our-school | app/our-school/page.tsx | Local content | CONFIGURABLE CORE | About RiverThames. |
| /privacy | app/privacy/page.tsx | Static legal copy | CONFIGURABLE CORE | New controller-approved privacy notice. |
| /privacy/complaints | app/privacy/complaints/page.tsx | POST /api/v1/complaints/intake | CONFIGURABLE CORE | Complaint intake enabled only after controller/process approval. |
| /safeguarding | app/safeguarding/page.tsx | Static policy content | CONFIGURABLE CORE | RiverThames-approved safeguarding page; not dependent on community moderation. |
| /status | app/status/page.tsx | GET /healthz | CORE | Service status. |
| /terms | app/terms/page.tsx | Static Kaizen/payment wording | CONFIGURABLE CORE | New RiverThames terms; source wording must not migrate. |
| /timetable | app/timetable/page.tsx | GET /api/v1/timetable and /timetable/changes; two direct live Stripe links | CONFIGURABLE CORE | Programme/session schedule; remove all Kaizen provider links and redesign filters. |

### Authentication and account routes — 8

| Route | Source | API or module dependency | Classification | RiverThames equivalent and action |
|---|---|---|---|---|
| /accept-invite | app/accept-invite/page.tsx | POST /api/v1/auth/accept-invite | CORE | User/staff invitation acceptance. |
| /account/security | app/account/security/page.tsx | /auth/me, account 2FA, sessions, recovery codes and recent-auth step-up | CORE | Account security. |
| /confirm-email-change | app/confirm-email-change/page.tsx | POST /api/v1/auth/confirm-guardian-email-change | CONFIGURABLE CORE | Neutral account/person email change; remove Guardian-only naming. |
| /forgot-password | app/forgot-password/page.tsx | POST /api/v1/auth/forgot-password | CORE | Same capability. |
| /login | app/login/page.tsx | POST /api/v1/auth/login and /login/totp | CORE | Same capability. |
| /register | app/register/page.tsx | POST /api/v1/auth/register and resend verification | CONFIGURABLE CORE | Support independent adult learners as well as responsible adults. |
| /reset-password | app/reset-password/page.tsx | POST /api/v1/auth/reset-password | CORE | Same capability. |
| /verify-email | app/verify-email/page.tsx | POST /api/v1/auth/verify-email | CORE | Same capability. |

### Portal routes — 4

| Route | Source | API or module dependency | Classification | RiverThames equivalent and action |
|---|---|---|---|---|
| /portal | app/portal/page.tsx | Layout GET /api/v1/portal/me; page GET /portal/overview | CONFIGURABLE CORE | Learner/family dashboard based on neutral person capabilities. |
| /portal/billing | app/portal/billing/page.tsx | Billing overview, checkout and billing-session | CONFIGURABLE CORE | Payments/invoices. |
| /portal/bookings | app/portal/bookings/page.tsx | GET /portal/bookings and booking cancellation | CONFIGURABLE CORE | Enrolments/bookings. |
| /portal/library | app/portal/library/page.tsx | Member syllabus, collections and techniques | MARTIAL-ARTS VERTICAL | No first-MVP equivalent; remove My syllabus navigation. |

### Admin routes — 23

| Route | Source | API or module dependency | Classification | RiverThames equivalent and action |
|---|---|---|---|---|
| /admin | app/admin/page.tsx | GET /api/v1/admin/dashboard/summary | CORE | Module-aware admin dashboard. |
| /admin/attendance | app/admin/attendance/page.tsx | /api/v1/admin/attendance/* | CONFIGURABLE CORE | Session register plus collection/staff-rota extensions. |
| /admin/audit | app/admin/audit/page.tsx | /api/v1/admin/audit-logs/v2 | CORE | Audit log. |
| /admin/blog | app/admin/blog/page.tsx | /api/v1/admin/blog* | CORE | CMS article administration. |
| /admin/bookings | app/admin/bookings/page.tsx | GET /api/v1/bookings and PATCH /api/v1/admin/bookings/{id} | CONFIGURABLE CORE | Enrolment administration against new offering/session model. |
| /admin/complaints | app/admin/complaints/page.tsx | /api/v1/admin/complaints*, deadlines and notifications | CONFIGURABLE CORE | Restricted privacy complaint handling with RiverThames policy. |
| /admin/events | app/admin/events/page.tsx | /api/v1/admin/events*, registrations, CSV and rights actions | CORE | Workshop/event administration. |
| /admin/forums | app/admin/forums/page.tsx | /api/v1/admin/forums* | DEFERRED | No first-MVP route. |
| /admin/instructors | app/admin/instructors/page.tsx | /api/v1/admin/instructors*, profile/account links | CONFIGURABLE CORE | Teachers/assistants and credential assurance. |
| /admin/library | app/admin/library/page.tsx | /api/v1/admin/library/* taxonomy, grades, techniques, AI drafts and collections | MARTIAL-ARTS VERTICAL | Do not include. |
| /admin/locations | app/admin/locations/page.tsx | /api/v1/admin/locations | CONFIGURABLE CORE | Venues, rooms and constrained resources. |
| /admin/media | app/admin/media/page.tsx | /api/v1/admin/media | CORE | Generic media management; no tenant media migrates. |
| /admin/media/video | app/admin/media/video/page.tsx | /api/v1/admin/videos* upload, captions, processing, grants and scan | DEFERRED | Optional video module, disabled for first MVP. |
| /admin/members | app/admin/members/page.tsx | Families, members, gradings, licences and training locations | CONFIGURABLE CORE | Learners/relationships; grading/licence subfeatures remain martial-only. |
| /admin/members/import | app/admin/members/import/page.tsx | /api/v1/admin/membership-imports/* | MARTIAL-ARTS VERTICAL | Kaizen/NEST compatibility; generic import is ART-168. |
| /admin/members/reconciliation | app/admin/members/reconciliation/page.tsx | /api/v1/admin/person-reconciliation/* | MARTIAL-ARTS VERTICAL | Kaizen migration tooling; do not copy into MVP. |
| /admin/moderation | app/admin/moderation/page.tsx | Moderation queue, safeguarding queue and item actions | DEFERRED | Public-community moderation deferred; not the art safeguarding case model. |
| /admin/payments | app/admin/payments/page.tsx | /api/v1/admin/billing/* terms, deals, coverage, links, reminders and exports | CONFIGURABLE CORE | Offers, payments, credits/refunds; replace family/member term assumptions. |
| /admin/seo/redirects | app/admin/seo/redirects/page.tsx | /api/v1/admin/redirects* | CORE | Redirect management. |
| /admin/settings/mfa | app/admin/settings/mfa/page.tsx | GET /api/v1/admin/mfa-compliance | CORE | MFA compliance. |
| /admin/settings/roles | app/admin/settings/roles/page.tsx | /api/v1/admin/roles* | CORE | Role/capability management with a new art-school matrix. |
| /admin/timetable | app/admin/timetable/page.tsx | Classes, locations, disciplines, breaks, overrides, occurrence preview and summer seasons | CONFIGURABLE CORE | Programme/course/session administration; current class ontology is insufficient. |
| /admin/users | app/admin/users/page.tsx | /api/v1/admin/users* | CORE | User administration. |

## Layouts, navigation and special web surfaces

| Surface | Source | Classification | Finding and RiverThames action |
|---|---|---|---|
| Root layout | app/layout.tsx | CONFIGURABLE CORE | Global header/footer, auth, cookies, analytics, MFA banner and JSON-LD. Build-time GET /api/v1/locations has a 3-second timeout and degrades to no locations. Metadata is fixed Kaizen martial copy. |
| Admin layout | app/admin/layout.tsx | CORE | Auth/capability guard through /api/v1/admin/roles/me and AdminNav. |
| Portal layout | app/portal/layout.tsx | CONFIGURABLE CORE | /api/v1/portal/me guard plus member-oriented PortalNav. |
| Blog metadata layout | app/blog/layout.tsx | CONFIGURABLE CORE | Tenant metadata/canonical values. |
| Blog fallback layout | app/blog/post/layout.tsx | CONFIGURABLE CORE | Static-host article fallback metadata. |
| Booking metadata layout | app/book/layout.tsx | CONFIGURABLE CORE | Tenant/trial wording. |
| Events metadata layout | app/events/layout.tsx | CONFIGURABLE CORE | Tenant event metadata. |
| Instructor metadata layout | app/instructors/layout.tsx | CONFIGURABLE CORE | Rename/configure teacher terminology. |
| Timetable metadata layout | app/timetable/layout.tsx | CONFIGURABLE CORE | Tenant schedule metadata. |
| Global navigation | components/site-header.tsx, lib/site.ts | CONFIGURABLE CORE | Hardcoded 12-link menu plus Free trial. ART-149/150 must drive labels, ordering and enabled modules. |
| Admin navigation | components/admin-nav.tsx | CONFIGURABLE CORE | Hardcoded martial, community and video destinations. Module manifest must remove disabled links. |
| Portal navigation | components/portal-nav.tsx | CONFIGURABLE CORE | Hardcoded My syllabus and Community links must be absent when modules are disabled. |
| Account navigation | components/account-nav.tsx | CORE | Generic login/account mechanics. |
| Sitemap | app/sitemap.ts | CONFIGURABLE CORE | Static routes plus CMS and library taxonomy. Build from enabled modules and tenant origin. |
| Robots | app/robots.ts | CONFIGURABLE CORE | Protected/token/API exclusions are reusable; origin and sitemap are tenant config. |
| Video sitemap | app/video-sitemap.xml/route.ts | MARTIAL-ARTS VERTICAL / DEFERRED | Proxies /api/v1/library/video-sitemap.xml; do not register for first MVP. |
| 404 | app/not-found.tsx | CONFIGURABLE CORE | Reusable mechanics; wording hardcodes Contact Kaizen. |
| Route error/loading boundaries | none | ART VERTICAL GAP | No app/error.tsx or loading.tsx exists; add generic safe failure/loading boundaries during extraction. |
| Status | app/status/page.tsx | CORE | GET /healthz. |
| Static publication | lib/static-publish.mjs, next.config.mjs | CONFIGURABLE CORE | CMS redirects/retired/dynamic pages are useful. The /library/:path* rewrite is martial-only. |
| Public CMS loader | lib/public-content.ts | CONFIGURABLE CORE | Build-time fetch, 3-second timeout, fixtures and placeholder slugs. Published content changes require a rebuild. |
| Analytics/consent | lib/analytics.ts, components/analytics.tsx, components/cookie-consent.tsx | CONFIGURABLE CORE | Fail-closed consent boundary is reusable; payload hardcodes Kaizen URL/title and events are library/video focused. |
| Email templates | none in apps/web | CORE API dependency | Web routes trigger backend auth/booking/billing communications; template/provider logic is API-owned. |

The web contains 29 files with forms. They cover authentication, account security, free-trial booking, community, complaint intake and admin CRUD. There is no public enquiry/CRM form, adult self-enrolment, course/workshop/drop-in booking, waitlist offer, room/resource selection, authorised collection, allergy/accessibility intake, or artwork/project consent journey.

## Complete FastAPI router inventory

Every module below is under apps/api/app/routers. Operation is one HTTP decorator in that module; modules with several router objects are mounted into public, member and/or admin prefixes. All 62 router objects are mounted unconditionally today.

| Router module | Ops | Main surface | Classification | RiverThames treatment |
|---|---:|---|---|---|
| account.py | 5 | Account 2FA | CORE | Reuse MFA account flows. |
| admin.py | 21 | Admin locations, disciplines, classes, instructors and related CRUD | CONFIGURABLE CORE | Split neutral venue/staff operations from martial/legacy class concepts. |
| admin_roles.py | 3 | Role/capability matrix | CORE | Reuse enforcement; replace role policy. |
| admin_users.py | 4 | User administration | CORE | Reuse. |
| attendance_admin.py | 4 | Attendance register | CONFIGURABLE CORE | Rebind to Session and add collection/rota states. |
| audit.py | 3 | Audit log access | CORE | Reuse closed evidence model and recent-auth checks. |
| auth.py | 16 | Login, registration, tokens, sessions and step-up | CORE / CONFIGURABLE CORE | Reuse security; remove Guardian-only registration/email semantics. |
| billing.py | 3 | Public plans/quotes/term catalogue | CONFIGURABLE CORE | Replace fixed membership catalogue. |
| billing_admin.py | 24 | Reconciliation, coverage, terms, deals, links and exports | CONFIGURABLE CORE | Preserve ledger/idempotency; redesign offers and neutral payer/learner links. |
| blog_admin.py | 4 | Legacy/admin blog actions | CORE | Consolidate with CMS router during extraction. |
| blog_cms.py | 11 | Public/admin blog CMS | CORE | Reuse publishing workflow with clean tenant content. |
| booking.py | 3 | Public trial booking | CONFIGURABLE CORE | Replace Guardian + Member + ScheduledClass creation path. |
| bookings_capacity.py | 8 | Admin booking, capacity and waitlist actions | CONFIGURABLE CORE | Rebind to Offering/Session/Enrolment and expiring offers. |
| checkout.py | 3 | Stripe checkout, portal and webhook | CONFIGURABLE CORE | Reuse provider boundary; new account/catalogue/secrets only. |
| comms.py | 8 | Communication templates, send/reconcile/outbox operations | CORE | Reuse durable provider-evidence boundary. |
| complaints.py | 9 | Public intake and restricted admin cases | CORE / CONFIGURABLE CORE | Reuse mechanics with new controller policy and role assignments. |
| consent.py | 2 | Consent text/current acceptance | CONFIGURABLE CORE | Reuse version/default-deny pattern; new purposes/text. |
| content.py | 5 | Public instructors, events and blog queries | CONFIGURABLE CORE | Rename instructor and connect to tenant CMS types. |
| dashboard.py | 1 | Admin summary | CONFIGURABLE CORE | Make module-aware. |
| dsr.py | 8 | DSR case/export/erasure operations | CORE | Reuse with isolated stores and controller decisions. |
| events.py | 14 | Public/admin events, registrations and rights actions | CORE / CONFIGURABLE CORE | Reuse for workshops; map registration to neutral person/enrolment model. |
| forums.py | 12 | Public/member/admin forums | DEFERRED | Do not register for first MVP. |
| instructor_profile.py | 4 | Public/admin instructor profiles | CONFIGURABLE CORE | Rename teacher/staff; retain scoped profile editing. |
| intake_actions.py | 5 | Public intake action tokens and conversions | CONFIGURABLE CORE | Reuse kernel, but extend beyond free_trial/event_registration. |
| library.py | 38 | Public/admin/member martial taxonomy, techniques, grades and collections | MARTIAL-ARTS VERTICAL | Exclude module, schema, seeds, routes and generated types. |
| location_profile.py | 5 | Public/admin location profiles | CONFIGURABLE CORE | Extend to rooms/accessibility/resources; geocoder remains unimplemented. |
| media.py | 6 | Admin/public media and galleries | CORE / CONFIGURABLE CORE | Reuse sanitisation/storage; extend artwork rights metadata. |
| media_consent.py | 2 | Member media-consent decisions | CONFIGURABLE CORE | Extend to artwork/attribution/purpose/channel. |
| members.py | 20 | Family/member CRUD, training locations, grading and licence | CONFIGURABLE CORE / MARTIAL-ARTS VERTICAL | Extract neutral learner/person operations; exclude grading/licence. |
| members_directory.py | 9 | Member directory, notes, tags and segments | CONFIGURABLE CORE / DEFERRED | Rebuild only required learner operations; defer segmentation/community trust. |
| membership_imports.py | 8 | Encrypted draft/import/apply | MARTIAL-ARTS VERTICAL / DEFERRED | Do not migrate NEST workflow; ART-168 owns generic imports. |
| mfa_compliance.py | 1 | Admin MFA compliance | CORE | Reuse. |
| mfa_recovery.py | 1 | Restricted MFA recovery | CORE | Reuse with new named approvers and vault. |
| moderation.py | 7 | Public reports, moderator and safeguarding queues | DEFERRED | Community module off; build safeguarding case handling independently. |
| person_reconciliation.py | 4 | Legacy person reconciliation | MARTIAL-ARTS VERTICAL / DEFERRED | Kaizen migration compatibility only. |
| portal.py | 10 | Guardian dashboard, bookings, billing and library | CONFIGURABLE CORE | Rebuild around learner/payer/authority capabilities; exclude syllabus. |
| publishing.py | 9 | Publish/schedule/tick and revision actions | CORE | Reuse; scheduled tick currently admin-triggered. |
| reactions.py | 4 | Public/member reactions | DEFERRED | Do not register for first MVP. |
| redirects.py | 9 | Public/admin redirects and suggested 404s | CORE | Reuse with tenant paths/origin. |
| reminders_admin.py | 10 | Member/payment reminder templates and runs | CONFIGURABLE CORE | Rebind to neutral subjects/offers and controller communication policy. |
| seo.py | 3 | Public/admin SEO metadata | CORE / CONFIGURABLE CORE | Reuse engine; tenant defaults/content only. |
| summer_timetable.py | 7 | Public/admin seasonal timetable | CONFIGURABLE CORE | Generalise to arbitrary programme seasons or retire. |
| timetable.py | 3 | Public timetable and class detail | CONFIGURABLE CORE | Replace ScheduledClass/Discipline contract. |
| timetable_power.py | 10 | Admin breaks, overrides, mutation clock and occurrence preview | CONFIGURABLE CORE | Reuse recurrence/override lessons against the new Session model. |
| video.py | 12 | Admin upload/process plus public streaming grants | DEFERRED | Optional module; no routes, storage, queue or jobs in first MVP. |
| **Router total** | **358** |  |  |  |
| main.py operational endpoints | 4 | Root, health, readiness and CSP report | CORE / CONFIGURABLE CORE | Reuse safe health/readiness/CSP handling; tenant telemetry config. |
| **OpenAPI total** | **362** | **300 unique paths** |  |  |

Generated OpenAPI tag totals corroborate the surface: admin 205, public 44, members 20, auth 16, portal 10, complaints 9, DSR 8, membership-imports 8, content 5, account 5, public-intake 5, person-reconciliation 4, untagged operational 4, timetable/booking/audit/billing/checkout 3 each, and consent/media-consent 2 each.

## Service inventory — 81 modules

There are 75 top-level service modules plus 6 email adapter/policy modules, excluding package initialisers. Every current service is grouped once below.

| Classification | Service modules | Extraction action |
|---|---|---|
| CORE security, compliance, publishing and evidence | age_authority, audit, auth_app, auth_throttle, blog_cms, comms, complaints, dsr, event_registration_rights, event_registration_urls, intake_contact, mfa_policy, payment_ledger, people, person_access_overrides, public_intake, publishing, rbac, redirects, retention, sanitize, seo, subject_selection, totp, totp_secrets | Preserve denial-first security, append-only evidence, idempotency and sanitisation. Rename package/config boundaries and retest with synthetic identities. |
| CONFIGURABLE CORE operations | admin, attendance_register, bespoke_payment_links, billing, billing_linkage, billing_reconciliation, booking, booking_capacity, checkout, consent, content, events, guardian_email_change, instructor_credentials, instructor_links, legacy_authority, location_profile, media, media_consent, media_pipeline, media_storage, member_coverage_admin, member_reminders, member_training_locations, member_training_rights, members, members_directory, memberships, payment_reminders, portal, stripe_catalog, summer_timetable, timetable, timetable_power, timetable_revision | Rebind to neutral Person, Programme, Offering, Session, Enrolment, Offer and Payment concepts. Split mixed admin services at module boundaries. |
| MARTIAL/legacy compatibility | library, membership_import_apply, membership_import_draft_crypto, membership_import_drafts, membership_import_preview, membership_import_review, person_reconciliation | Keep outside generic/art packages. Only generic validation/idempotency lessons may be reimplemented under ART-168. |
| DEFERRED modules | forums, moderation, reactions, video, video_ingest, video_processing, video_storage, video_transcripts | Disabled modules must not register routes, jobs, navigation, storage or provider dependencies. |
| Email provider boundary | email/base, email/policy, email/console, email/mailpit, email/resend, email/acs | Reuse adapter interface and fail-closed cloud policy. RiverThames needs its own sender identity, provider account, allow-list and secrets. |

Key service-boundary blockers:

1. All router mounts are unconditional. Existing flags may fail closed at runtime, but do not remove routes, OpenAPI entries, imports or infrastructure dependencies. ART-150 must introduce true module registration.
2. admin.py mixes neutral venue/staff CRUD with Discipline, ScheduledClass and member compatibility.
3. public_intake is a strong token/effect kernel but its accepted action types are currently closed to free_trial and event_registration.
4. The old booking service creates Guardian and Member records as part of trial booking; it cannot support an independent adult cleanly.
5. Portal, reminders, billing and attendance services depend on legacy Guardian/Member/ScheduledClass identifiers.
6. Library, grading and licence dependencies must not leak into shared types, portal queries or admin navigation.

## Jobs and background processes

### Provisioned jobs

| Process | Source/deployment | Trigger | Classification | RiverThames treatment |
|---|---|---|---|---|
| Database migration | app/db_migrate.py; infra/modules/containerAppMigrationJob.bicep | Manual/deployment job | CORE | Reuse per isolated deployment with forward-only migrations. |
| DSR erasure worker | app/dsr_erasure_worker.py; containerAppDsrErasureJob.bicep | Every 15 minutes; config-gated and default-off | CORE | Retain gated two-person/recent-auth/controller controls; enable only after policy approval. |
| Video ingest worker | app/video_ingest_worker.py; containerAppIngestJob.bicep | Azure Queue scaled | DEFERRED | Do not provision for first MVP. |
| Video transcode worker | app/video_worker.py; containerAppJob.bicep | Every 15 minutes | DEFERRED | Do not provision for first MVP. |

### Manual or admin-triggered processes

- app/retention.py produces retention reporting/action plans; controller policy must be RiverThames-specific.
- app/complaints_deadline_check.py checks complaint deadlines; no scheduler is evidenced in the application deployment.
- app/stripe_catalog_sync.py synchronises the selected Stripe catalogue; RiverThames must use a separate account/catalogue.
- app/legacy_authority_preflight.py is legacy migration assurance and should not enter the clean baseline.
- app/mfa_break_glass.py is a restricted recovery procedure; retain only with named approvals/runbook.
- member_data_* probes and rotation ceremony tools support protected-member-data operations; reuse only after ART-156/170 decisions.
- seed.py, seed_admin.py, seed_consent.py, seed_content.py, seed_demo_member.py and seed_library.py contain Kaizen data/vertical assumptions. RiverThames uses new unmistakably synthetic fixtures under ART-169.
- Publishing /tick and reminder generation are admin-triggered.
- Instructor credential-expiry sweep exists in service/tests but has no evidenced deployed scheduler; staff-assurance scheduling is a blocker for ART-155/163.

## Provider and external-integration map

| Provider/boundary | Source evidence | Classification | RiverThames rule |
|---|---|---|---|
| PostgreSQL/SQLAlchemy/Alembic/pgvector | db.py, models.py, migrations and vector fallback | CORE / CONFIGURABLE CORE | New isolated database and clean baseline; no Kaizen rows or migration history copy. |
| Stripe Checkout, customer portal, catalogue, ledger and signed webhook | routers/checkout.py, services/checkout.py, stripe_catalog.py, payment_ledger.py | CONFIGURABLE CORE | Separate Stripe account/config/catalogue/webhook secret. Never copy customers, IDs or live links. |
| Email: console, Mailpit, Resend and Azure Communication Services | services/email/* selected by EMAIL_PROVIDER | CORE adapter / CONFIGURABLE deployment | Console/Mailpit local only; separate cloud sender identity/provider account/allow-list/secrets. |
| Azure Blob Storage | media_storage.py and video_storage.py | CORE media / DEFERRED video | Separate storage account/containers/identity. Exclude video containers for MVP. |
| Azure Queue | video_ingest_worker.py | DEFERRED | Do not provision until video module is approved. |
| Azure Key Vault | secrets.py, TOTP and protected-member-data modules | CORE | Separate vault, managed identity and keys; no Kaizen secret access. |
| Azure Monitor/OpenTelemetry | observability.py and safe_telemetry.py | CONFIGURABLE CORE | Separate telemetry resource; preserve allow-listed, privacy-safe events. |
| Geocoder | location_profile.py | ART VERTICAL GAP | Current adapter is a 501 stub; decide provider or manual verified geodata. |
| Automated moderation | services/moderation.py | DEFERRED | Current local keyword stub explicitly defers Azure AI Content Safety/OpenAI; no first-MVP dependency. |
| Inbound callbacks | Stripe webhook and CSP report only | CORE / CONFIGURABLE CORE | Preserve signature/size/sanitisation controls and tenant-specific origins/secrets. |

## Permissions and assurance

services/rbac.py defines 17 capabilities:

content.create, content.publish, content.schedule, media.manage, timetable.manage, bookings.manage, members.read, members.write, billing.read, billing.manage, forum.moderate, redirects.manage, roles.manage, audit.read, dsr.erasure.break_glass, complaints.handle and complaints.restricted.

The seven current staff roles are site_admin, editor, instructor, moderator, billing_admin, privacy_erasure_approver and complaints_restricted_reviewer. Legacy aliases admin and contributor remain supported. Guardian grants no admin capability. Unknown roles receive no capabilities.

Reusable controls:

- deny-by-default capability union;
- separate explicitly delegated early-erasure and restricted-complaint roles;
- recent-auth/step-up for sensitive operations;
- service-level row scope for instructors editing their own profile/draft;
- restricted member/safeguarding data paths and append-only audit evidence.

RiverThames gaps under ART-154/155/163:

- operator/manager role separate from platform administrator;
- teacher and assistant distinctions;
- DSL/safeguarding role independent of public-community moderator;
- course/offering/session and venue/room row scope;
- separate collection, emergency, medical/allergy/accessibility and payment access;
- staff assurance/credential gates on assignments and ratios;
- module-aware role/capability presentation so disabled capabilities cannot be delegated.

## Hard-coded web coupling and non-migration boundaries

A tracked-file scan of apps/web, excluding test/e2e directories and test/spec files, found:

| Marker | Files containing marker |
|---|---:|
| Kaizen/Kiazen/kaizenkaikan | 110 |
| @kaizen/shared imports in app/components/lib TypeScript | 48 |
| karate | 13 |
| martial arts | 12 |
| dojo | 6 |
| grading | 18 |
| syllabus | 6 |
| guardian | 28 |
| member | 56 |
| kaizenkaikan.fit or kaizenkaikan.com | 7 |

There is no runtime literal email address in the web app. The only email-regex match was test@example.test in scripts/verify-static-export-links.mjs.

Highest-risk source points:

- lib/site.ts: brand, phone, logo, navigation and five fixed venues.
- lib/content.ts: migrated site copy, Karate/BJJ/MMA/Kobudo disciplines, testimonials and fixed plan copy.
- lib/area-pages.ts: six towns, kids-karate and mma-bjj offers and fixed schedules.
- app/layout.tsx, lib/metadata.ts and lib/jsonld.ts: fixed organisation names, martial descriptions, social handles and served areas.
- components/analytics.tsx: fixed Kaizen URL and page title.
- components/seo/seo-panel.tsx: Kaizen domain previews.
- app/timetable/page.tsx: two direct live Stripe Checkout URLs. Their values are deliberately omitted here and must never migrate.
- public/site.webmanifest and 18 media/icon files: Kaizen name, logos, slogan, class/instructor photography and other tenant assets.
- legal, safeguarding, privacy, waiver, biography, martial research, syllabus and local-SEO content is tenant material, not reusable platform configuration.

## Live-site cross-check

Direct HTTPS checks succeeded on 2026-11-06:

- https://kaizenkaikan.fit/, /sitemap.xml and /robots.txt returned HTTP 200.
- /build reports release 2026.11.2 and commit f1ab2e3c48273e775ad3e6aea9e930790d12ee1e.
- That commit is an ancestor of audited revision 44101ef2.
- git diff f1ab2e3..44101ef -- apps/web packages/shared is empty, so the deployed web/shared implementation matches this audit source.

The live sitemap contains exactly 52 URLs:

- 19 fixed public routes;
- 4 discipline pages;
- 12 town/offer pages;
- 3 blog posts;
- 1 event;
- 2 instructors;
- 6 locations;
- 5 library taxonomy routes;
- no technique or collection detail entries at the snapshot.

Of the 52 sitemap URLs, 47 returned HTTP 200. Five advertised martial-library taxonomy URLs returned HTTP 404 on both HEAD and GET at the snapshot:

- `/library/karate`
- `/library/karate/kata`
- `/library/karate/renraku`
- `/library/karate/kumite-kata`
- `/library/karate/unfiled`

The `/library` landing page and representative public, community, build, status and complaint-intake routes returned 200. The five stale sitemap entries are a live publication regression to fix in Kaizen; they are not routes to migrate to RiverThames. `/community`, `/build`, `/status` and `/privacy/complaints` are live but intentionally absent from the sitemap. Protected auth, portal and admin routes are also absent. `/blog/post` and `/library/browse` are internal static-host targets rather than canonical discovery URLs.

## RiverThames route/API gaps and existing owners

| Gap or blocker | Consequence | Existing ART ticket |
|---|---|---|
| Organisation/brand/domain/legal/navigation configuration | Kaizen identity leaks through metadata, analytics, JSON-LD, legal pages and assets | ART-149 |
| True module registration | Disabled modules still mount routes, appear in OpenAPI and load dependencies | ART-150 |
| Neutral person/learner/responsible-adult/payer model | Adult self-enrolment cannot use the current Guardian→Member path safely | ART-153 |
| Art-school role/capability matrix | Current instructor/moderator/member permissions do not express teacher/assistant/DSL/collection scopes | ART-154 |
| Staff assurance and credential scheduling | Credential checks exist but expiry sweep is not deployed | ART-155 |
| Sensitive learner profile | Emergency, allergy, accessibility and medical data need an approved encrypted boundary | ART-156 |
| Artwork/media consent | Current media consent lacks artwork, attribution and purpose/channel semantics | ART-157 |
| Programme→Offering→Session | Discipline/ScheduledClass cannot model arbitrary courses, blocks or workshops | ART-158 |
| Rooms/resources/equipment/accessibility | Location profile has no schedulable constrained resources | ART-159 |
| Adult/child enrolment, trials, capacity and waitlists | Current booking is a free trial and creates Guardian/Member records | ART-160 |
| Offers for blocks, packs, workshops, drop-ins and materials | Membership/term catalogue is too fixed | ART-161 |
| Refunds/credits/transfers/disputes | Strong ledger exists but needs neutral offer/enrolment links | ART-162 |
| Rota, attendance and authorised collection | One instructor and three attendance states are insufficient | ART-163 |
| Public art discovery and filters | Current discipline/area/timetable pages expose martial vocabulary | ART-164 |
| CMS/media/SEO extraction | Strong engine, but tenant content, domains and library sitemap rules are coupled | ART-165 |
| Enquiries/CRM/transactional communications | Contact page has no form; intake action types are too narrow | ART-166 |
| Privacy-safe operational reporting | No complete art occupancy/funnel/margin surface | ART-167 |
| Generic import/export | Existing import/reconciliation is Kaizen/NEST-specific | ART-168 |
| Synthetic art vertical and fixtures | Current seeds contain Kaizen and martial material | ART-169 |

No new issue is required for these route/API gaps.

## Extraction blockers and acceptance gates

1. ART-149 must establish validated Organisation, Site, Brand, domain, terminology and legal-link configuration before any public UI is copied.
2. ART-150 must make module registration structural. A disabled module must contribute no route, OpenAPI schema, navigation item, job, seed, table dependency or provider infrastructure.
3. ART-151/153/158 must establish neutral Person and Programme/Offering/Session boundaries before booking, portal, attendance, billing or reminder code is ported.
4. Generated shared types must be regenerated from the reduced, module-aware API; @kaizen/shared cannot become the generic package name.
5. Public static publication currently needs an API-backed rebuild for CMS changes. RiverThames must explicitly accept that model or select a different publication trigger.
6. Route-level safe error/loading boundaries are missing from the web app.
7. Direct live payment links, domains, social handles, local storage keys, analytics values, provider identifiers, public assets, legal copy, biographies and seed content must not migrate.
8. Separate databases, storage, Key Vault, managed identity, Stripe/email identity, domains and telemetry remain mandatory; no route/API reuse grants access to Kaizen runtime data.

## Recommended reusable boundary

Extract security/authentication, capability enforcement, audit, complaint/DSR mechanics, CMS publishing, generic media, redirects/SEO, safe telemetry, provider adapters and operational health as versioned core modules. Put organisation/brand/public-shell configuration around them. Build people, programme/session, enrolment, commerce and registers against new neutral aggregates while reusing proven invariants and tests. Keep grading/licence/library/NEST features in a Kaizen martial pack. Keep community/video modules disabled until separately justified.
