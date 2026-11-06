# ART-146 — RiverThames Art School gap analysis

**Evidence revision:** Kaizen `44101ef2fc772e0903ff926247a01597a439a2a3`
**Working product:** RiverThames Art School
**Architecture constraint:** one versioned reusable core, explicit vertical packs, and an isolated deployment per organisation.

## Executive finding

Kaizen proves the platform's security, operational and deployment patterns, but it does not prove the art-school domain. RiverThames needs a neutral education model rather than renamed martial-arts entities. The first release should favour safe enrolment and delivery of weekly child and adult programmes; later formats and engagement features should be enabled only through explicit modules.

No new Linear issue is justified by this audit. The identified work is already represented by ART-144–ART-173. Two dependency links should be corrected before UAT planning: ART-172 should explicitly depend on the role/MFA work in ART-154 and the sensitive-data work in ART-156.

## Target capability map

| RiverThames need | Kaizen evidence | Gap / reuse decision | MVP position | Existing ART ticket |
|---|---|---|---|---|
| Organisation identity, brand, domains and legal links | Brand, metadata, JSON-LD, navigation and policy copy are compiled around Kaizen | Move safe organisation settings to validated configuration; write new approved content | Required | ART-149 |
| Children's weekly art classes | Weekly `ScheduledClass`, timetable, booking and attendance exist | Replace Discipline/Class semantics with Programme → Offering → Session and neutral eligibility rules | Required | ART-158, ART-160 |
| Adult art classes and daytime programmes | Person can conceptually hold several roles, but booking requires Guardian → Member | Design independent adult learner/payer and responsible-adult relationships greenfield | Required | ART-153, ART-158, ART-160 |
| School-hosted clubs | Locations/classes exist | Need organisation/customer context, host school, collection rules and term-specific offer configuration | Post-foundation module | ART-158, ART-159, ART-161, ART-163 |
| Holiday workshops | Events and registrations exist; membership is term-centric | Reuse event/checkout patterns but model workshop occurrence, materials, capacity and consent | Required if included in charter | ART-158, ART-160, ART-161 |
| Trials | Public free-trial booking exists | Generalise to configurable trial products; do not retain Guardian-only or weekly-class-only shape | Required | ART-153, ART-160 |
| Waitlists and capacity | `ClassWaitlistEntry` exists and scheduled classes have one capacity | Add atomic holds, promotion, expiry, per-session capacity and cancellation policy | Required | ART-159, ART-160 |
| Parties and private sessions | No suitable aggregate | New optional offer type, enquiry/quote and private-session scheduling | Deferred unless charter selects it | ART-158, ART-161, ART-166 |
| Tutors, teachers and assistants | Instructor profile, credentials and expiry notifications exist | Generalise staff roles, assurance status, qualifications and session assignments; support multiple staff | Required | ART-154, ART-155, ART-158 |
| Venues, rooms and resources | Location profile and one location per class | Add room/resource/equipment constraints, accessibility, travel and conflict detection | Required | ART-159 |
| Programme/course/session scheduling | Weekly discipline timetable with overrides and summer seasons | Greenfield hierarchy and recurrence/exception model; avoid martial terminology | Required | ART-158 |
| Booking/enrolment lifecycle | Free-trial booking and portal cancellation | Add offer enrolment, holds, transfer, catch-up, waitlist, capacity and policy evidence | Required | ART-160 |
| Attendance | Present/absent/late register exists | Add authorised collection, delivery staff/rota and exception evidence | Required | ART-163 |
| Guardian/responsible-adult authority | Guardian owns Member; Person relationships provide useful evidence patterns | Preserve verified, evented authority but remove mandatory ownership and duplicated PII | Required | ART-153 |
| Independent adult learners | Unsupported without synthetic Guardian/Member roles | Must be first-class and tested through registration, booking, payment and portal | Required | ART-153, ART-160, ART-161 |
| Authorised collection | Not implemented as a delivery workflow | Add approved collectors, time-bounded authority, denial paths and collection record | Required for child sessions | ART-153, ART-163 |
| Emergency contacts | No art-school-appropriate intake | Design purpose limitation, access, retention and emergency disclosure | Required, approval-gated | ART-156 |
| Allergy, medical and accessibility needs | No suitable neutral model | Store only approved minimum, encrypt where required and restrict to delivery need-to-know | Required, approval-gated | ART-156 |
| Consent and policy evidence | Versioned consent, waiver and media consent exist | Reuse immutable/versioned evidence patterns; create RiverThames purposes and texts | Required | ART-157 |
| Artwork/media consent | Media consent exists but is person/media oriented and carries a legacy tenant field | Separate artwork display, photography, promotional use and withdrawal effects | Required where publishing occurs | ART-157, ART-165 |
| Safeguarding controls | Public policy, moderation/reporting and evented compliance workflows exist | Safeguarding design, DSL role and escalation rules must be separately approved; forums do not substitute for it | Required | ART-154–ART-157 |
| Staff assurance | Credentials/expiry model exists | Generalise checks, evidence, renewal and scope without copying personal records | Required | ART-155 |
| Stripe payments | Checkout, webhooks, idempotency and transaction evidence exist | Reuse provider boundary; use a separate account/config and neutral offer identifiers | Required | ART-151, ART-161, ART-162, ART-171 |
| Term/block payments | Three fixed named terms and family/member entitlements | Model configurable offerings, blocks, price components, concessions and payer relationships | Required | ART-161 |
| Subscriptions | Monthly subscriptions exist | Retain only if charter selects them; keep provider and product IDs per deployment | Optional | ART-161, ART-162 |
| Workshops/drop-ins | One-off event registration partly fits | Unify capacity/payment/attendance under an explicit offering type | Required if charter selects it | ART-158, ART-160, ART-161 |
| Refunds and payment lifecycle | Transactions, adjustments, billing chase and portal exist | Add RiverThames policy, partial/full refund state and auditable provider reconciliation | Required | ART-162 |
| Transactional communications | Email adapters, templates and reminders exist | Keep provider abstraction/outbox/idempotency patterns; create new sender identity, copy and recipients | Required | ART-166, ART-171 |
| CMS, blog and media | Mature blog, event, media, redirect and publishing flows | Reuse mechanics; never copy tenant copy, biographies, legal text or media | Required subset | ART-165 |
| SEO and local discovery | Metadata, JSON-LD, sitemap, areas and location pages exist | Make origins/schema/navigation configurable and author genuine programme/venue content | Required subset | ART-149, ART-164, ART-165 |
| Enquiries and CRM/trials | Contact page has no form; trial booking is narrow | Add consented enquiry capture, source, follow-up and conversion without building a broad CRM | Required | ART-166 |
| Galleries/projects | Martial library/video system is the wrong domain | Design a later art vertical with artwork consent, ownership and moderation; do not repurpose syllabus tables | Deferred | ART-150, ART-157, ART-165 |
| Privacy-safe reporting | Dashboard, exports, logs and telemetry exist | Define minimum aggregates, role access and anti-reidentification rules | Required subset | ART-167 |
| Audit | Append-only access and domain evidence is mature | Reuse pattern; retarget neutral subjects and capabilities | Required | ART-151, ART-167 |
| DSR, complaint and retention | Strong DSR/complaint workflows; retention ownership is incomplete | Reuse workflow patterns only after controller/DPIA/process approval; define module-owned schedules/jobs | Required | ART-151, ART-156, ART-157, ART-197 |
| Import/export | NEST membership import/reconciliation is Kaizen-specific | Build source-mapped, dry-run, idempotent generic import only when an approved source exists | Deferred until source approved | ART-168 |
| Recovery and backup | IaC, Postgres/storage and runbooks provide patterns | Validate an isolated restore, key rotation and recovery gate using synthetic data | Required before production | ART-170, ART-173 |
| Community/forums/reactions | Full feature exists | High safeguarding/moderation burden and no first-MVP need | Deferred | ART-150 |
| Advanced video pipeline | Storage, queue, scan/transcode and grants exist | Cost and operational burden without an MVP art-school need | Deferred | ART-150, ART-152 |

## Product boundary for the first MVP

The exact product boundary belongs to ART-144. The evidence supports this minimum candidate:

- public programme, teacher and venue discovery;
- child and independent-adult identity/authority paths;
- weekly programmes plus explicitly selected workshops;
- enrolment, trial, waitlist, payment, cancellation and refund evidence;
- session attendance, authorised collection and staff assignment;
- narrowly approved sensitive-data, consent, safeguarding and staff-assurance controls;
- transactional communications, CMS, SEO, audit and privacy-safe operational reporting;
- isolated infrastructure using synthetic data until external-provider and legal gates pass.

Community, reactions, forums, martial library/video, grading, licences, NEST compatibility, social feeds, broad CRM and galleries/projects should not enter the first MVP by inheritance.

## Reuse boundaries

### Reuse as engineering patterns

- session revocation, password reset, invitation, MFA/TOTP and step-up authentication;
- capability checks, denial tests, audit evidence and idempotency;
- versioned policy/consent evidence and append-only events;
- provider abstractions for Stripe, email, storage, queueing and telemetry;
- CMS publication, redirects, media mechanics, health/readiness and build identity;
- Bicep/azd/OIDC structure after full parameterisation.

### Rebuild as neutral product modules

- Person capabilities, learner/responsible-adult/payer authority and adult self-enrolment;
- Programme, Offering, Session, Venue, Room, Resource and staff allocation;
- Enrolment, waitlist, attendance, collection and sensitive delivery information;
- price offers, blocks, workshops, subscriptions and refund rules;
- all organisation-facing content, navigation, emails, legal text and seed data.

### Never migrate automatically

Kaizen/Cygnets people or payment data, databases, exports, recipient lists, secrets, environment/state files, provider identifiers, allow lists, legal copy, biographies, media, research, martial syllabus/kata content, live analytics history, or any other tenant-owned or licensed material.

## Human decisions required

1. ART-144: confirm the first-MVP offer mix—weekly children, weekly/daytime adults, trials and which workshop/private formats.
2. ART-145 / ART-181 / ART-197: confirm source contribution rights, operating entity, controller/processor roles, insurance and DPIA/supplier approvals.
3. ART-149 / ART-182: formalise RiverThames Art School as product/tenant name and choose domains, legal identity and public contact details.
4. ART-156 / ART-157: approve minimum sensitive data, retention, safeguarding/DSL process and consent purposes/texts.
5. ART-161 / ART-162: approve pricing, concessions, cancellations, transfers and refunds.

Until those decisions exist, use conspicuously synthetic fixtures and keep provider delivery and public publication disabled.
