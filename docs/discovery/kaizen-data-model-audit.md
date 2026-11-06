# ART-146 — Kaizen data-model audit

**Evidence revision:** `44101ef2fc772e0903ff926247a01597a439a2a3`
**Scope:** `apps/api/app/models.py`, related services/schemas, 74 Alembic migrations and data-handling runbooks.
**Result:** 118 ORM tables grouped below; no production or local database was opened.

## Executive finding

Kaizen's database is a mature single-club operational model with strong append-only evidence, security and provider-integration patterns. It is not a suitable schema baseline for RiverThames. The new product should reuse invariants and tested service patterns while creating intentional neutral aggregates under ART-151, ART-153 and ART-158.

The principal blockers are not missing tenant IDs. Isolated deployments intentionally make uniqueness local to one organisation. The blockers are the mandatory Guardian→Member ownership path, free-trial Booking→ScheduledClass shape, discipline-driven weekly timetable, family/member membership entitlements, fixed term names, and martial library/grading/licence aggregates.

## Complete table/aggregate inventory

Every current ORM table is included once in this inventory.

| Aggregate | Tables | Count | Classification | RiverThames treatment |
|---|---|---:|---|---|
| Membership import | `membership_import_draft`, `membership_import_apply_batch`, `membership_import_apply_group`, `membership_import_apply_item` | 4 | DEFERRED | Keep validation, dry-run, idempotency and encrypted-draft lessons. Exclude NEST/Kaizen import UI and never import Cygnets data without an approved source/lawful basis. ART-168 owns a generic framework. |
| Timetable, content catalogue and staff/event operations | `location`, `discipline`, `scheduled_class`, `term_break`, `scheduled_class_override`, `timetable_mutation_clock`, `summer_timetable_season`, `instructor`, `credential_type`, `credential`, `credential_expiry_notification`, `franchise_location`, `event`, `event_registration`, `event_registration_rights_action` | 15 | CONFIGURABLE CORE except `franchise_location` compatibility | Reuse venue/profile, override, credential and registration invariants. Replace Discipline/ScheduledClass with Programme→Course Offering→Session; add rooms/resources/staff assignments. Exclude franchise/NEST compatibility. |
| Identity, profile and authority | `person`, `person_profile`, `postal_address`, `person_address`, `guardian`, `member`, `member_training_location`, `person_relationship`, `person_relationship_event`, `person_reconciliation_event`, `person_access_override`, `person_access_override_event` | 12 | CORE concepts plus CONFIGURABLE/legacy rows | Preserve Person and verified authority concepts. Create a clean baseline without mandatory Guardian ownership, duplicate PII or Kaizen reconciliation history. Profile encryption remains controller/DPIA gated. |
| Community and member segmentation | `reaction`, `moderation_item`, `member_trust_score`, `forum_category`, `forum_thread`, `forum_post`, `forum_thread_subscription`, `member_note`, `member_tag`, `member_segment` | 10 | DEFERRED | Disable as a module for MVP. Safeguarding incident/concern handling must not depend on enabling public community. |
| Trial booking, communications and registers | `waiver_version`, `waiver_acceptance`, `message_template`, `comms_message`, `class_waitlist_entry`, `attendance_record`, `booking` | 7 | CONFIGURABLE CORE | Keep versioning, idempotency and audit patterns; rebind learner/payer/authority to neutral enrolment/session aggregates. Extend register and collection states. |
| CMS, media, gallery, video and SEO | `media_asset`, `gallery`, `gallery_item`, `blog_post`, `blog_category`, `blog_tag`, `blog_post_tag`, `content_revision`, `video_asset`, `video_stream_grant`, `seo_meta` | 11 | CORE / CONFIGURABLE CORE; video optional | Extract generic content/media/publishing; no tenant rows or media migrate. Extend rights metadata for artwork/project publishing. Video remains disabled/deferred for MVP. |
| Martial syllabus/technique library | `library_style`, `library_category`, `library_grade`, `technique`, `technique_ai_draft`, `technique_video_asset`, `technique_grade`, `technique_relationship`, `syllabus_collection`, `syllabus_collection_item` | 10 | MARTIAL-ARTS VERTICAL | Do not include in the generic/art baseline. Art projects, media, skills and resources need an original art vertical rather than renamed grade/syllabus tables. |
| Redirect intelligence | `redirect`, `suggested_404` | 2 | CORE | Extract after removing tenant paths/domains/content. |
| Commerce, membership and reminders | `subscription`, `membership_deal`, `term_period`, `term_period_exclusion`, `membership_agreement`, `membership_entitlement`, `membership_entitlement_adjustment`, `stripe_webhook_event`, `payment_transaction`, `bespoke_payment_link`, `billing_chase_action`, `payment_reminder_template`, `payment_reminder`, `member_reminder_template`, `member_reminder`, `member_reminder_action` | 16 | CORE provider/evidence patterns plus CONFIGURABLE CORE commercial model | Preserve webhook idempotency, immutable offers/adjustments and reconciliation. Replace guardian/member membership semantics with offer, enrolment, payer and entitlement models supporting blocks/subscriptions/packs/one-offs/materials. |
| Consent, rights, complaints, audit and martial records | `consent_type`, `consent_text`, `consent_record`, `data_subject_request`, `data_subject_request_event`, `data_protection_complaint`, `data_protection_complaint_event`, `access_audit_log`, `grading`, `licence`, `media_consent` | 11 | CORE compliance; `grading`/`licence` MARTIAL-ARTS VERTICAL; `media_consent` CONFIGURABLE | Extract generic compliance patterns under a new controller. Exclude grading/licence. Extend media consent to artwork, attribution and purpose/channel-specific withdrawal. |
| Accounts, sessions and MFA | `account`, `auth_session`, `mfa_challenge`, `mfa_throttle`, `mfa_break_glass_approval`, `auth_throttle` | 6 | CORE | Reuse with new roles, vault keys, provider configuration and rollout evidence. Do not carry account rows or factors. |
| Public intake, privacy-safe action links and email tokens | `public_intake`, `public_intake_conversion`, `public_intake_conversion_event`, `subject_handle_request`, `subject_handle`, `staff_preview_token`, `staff_preview_target`, `staff_preview_effect`, `public_intake_throttle`, `public_intake_action_token`, `public_intake_contact_suppression`, `intake_comms_effect`, `email_token`, `pending_guardian_email_change` | 14 | CORE / CONFIGURABLE CORE | Reuse token, suppression, preview and effect-idempotency patterns. Rename guardian-specific email change and integrate intake into the neutral CRM/enrolment lifecycle. |

**Total: 118 tables.**

## Identity and adult self-enrolment

### Reusable foundation

- `Person` is an opaque canonical identity, and account holder, learner and responsible-adult capabilities are conceptually non-exclusive.
- `PersonRelationship` records verified, scoped and time-bounded authority with append-only decision snapshots and safeguarding restrictions.
- Person access overrides, age/authority services and denial-first authorization provide valuable reusable tests.
- `PersonProfile`, `PostalAddress` and `PersonAddress` represent the intended encrypted-record boundary, but their effective storage mode is still gated/disabled and must not be described as a live RiverThames solution.

### Blocking legacy shape

- `Member.guardian_id` is non-null and cascades from `Guardian`.
- The model has 24 `guardian_id` fields and 21 `member_id` fields, so the coupling is systemic rather than confined to registration.
- `Guardian` and `Member` hold plaintext profile copies; `Guardian.email` is unique.
- `Booking`, `ClassWaitlistEntry`, `MembershipAgreement`, `MembershipEntitlement`, portal queries and several reminder/import flows require Guardian/Member IDs.
- A self-booking adult therefore has to be represented through legacy family roles even though Person can conceptually be both account holder and learner.
- `Account.guardian_id` and legacy authorization paths preserve compatibility and complicate direct learner access.
- Public free-trial booking upserts a Guardian by an untrusted email and creates a child Member, while public account registration deliberately refuses to adopt an existing Guardian/Member/Person by email. Those conflicting rules make email-based migration or identity linking unsafe.

**Decision:** ART-153 must design Person capabilities, Learner, Responsible Adult and Payer greenfield. Do not make `guardian_id` nullable as a shortcut and call adult enrolment solved. Positive and denial tests must cover adults, siblings, staff-learners, restricted relationships and age/competence transitions.

## Scheduling, programmes and delivery

### Current limits

- `ScheduledClass` is one weekly recurrence tied to one `Location`, one `Discipline` and at most one `Instructor`.
- Recurrence is day-of-week plus start/end time and effective dates. It has no first-class Programme, Course Offering or generated Session.
- `age_group` is a free-text display label alongside min/max age; audience is a small closed field.
- Capacity is a single class value; there are no room, equipment, material, ratio or multi-staff constraints.
- `TermBreak` and per-date overrides are useful, but they operate around the weekly class rather than an arbitrary course calendar.
- `SummerTimetableSeason` is a year-specific presentation snapshot, not a generic operational offering/session model.
- Attendance is only `present`, `absent` or `late` and lacks expected/arrived/left-early/no-show/cancelled and authorised-collection evidence.

### Greenfield boundary

ART-158 should own Programme→Course Offering→Session with arbitrary block names, one-offs, drop-ins, workshops, school clubs, closures, substitutions, cancellation and historical preservation. ART-159 adds Venue→Room and constrained resources. ART-163 adds multiple staff/assistants, ratios, attendance/collection and session materials. The current recurrence/override materializer is useful reference logic, not the schema to migrate.

## Commerce and fixed-term assumptions

- `MembershipDeal.category` is `term | monthly | bundle | payg`; `billing_period` is `seasonal_term | monthly | one_off`.
- `TermPeriod.name` is constrained to `autumn | spring | summer`; active commercial coverage is designed to remain contiguous between named periods.
- `MembershipAgreement` is guardian/family-level; `MembershipEntitlement` is member-level and duplicates deal/term snapshots for audit.
- These immutable snapshots, append-only adjustments, webhook event idempotency and payment ledger are strong reusable patterns.
- The fixed family/member/term shape cannot directly express an independent adult payer/learner, course-specific block, workshop ticket, class pack, materials fee, bursary or mixed payer relationship.

ART-161 should generalise immutable Offer/Price Version and policy. ART-162 should retain the payment/refund/reconciliation evidence. Entitlement activation must remain server/provider verified and never rely on a client success redirect.

## NEST and compatibility fields

Current source contains `legacy_nest_id` on exactly four tables:

- `location`
- `scheduled_class`
- `grading`
- `licence`

`franchise_location`, Guardian/Member ownership, import/reconciliation tables and several service paths remain compatibility surfaces even without that column name. ART-151 should exclude them from the clean baseline unless a neutral product need is independently demonstrated. Existing Kaizen migrations and data remain untouched.

## Uniqueness and tenancy assumptions

Examples of deployment-global uniqueness include:

- `Location.slug`, `Discipline.slug`, `Instructor.slug`, `Event.slug`, blog/category/tag slugs;
- `Guardian.email` and `Account.email`;
- waiver versions, template slugs, term codes and membership deal code/version;
- media/blob paths, canonical technique/syllabus paths, redirects and Stripe event/session identifiers.

The model contains 59 `unique=True` declarations and 35 named `UniqueConstraint` declarations. For the selected **one database per organisation** topology, uniqueness per database is appropriate. Adding `tenant_id` to every table is not required for the first RiverThames deployment and would move the design toward the explicitly deferred shared-database SaaS model. Only `media_consent` currently carries a default `tenant_id=1`, a remnant of Kaizen's deferred multi-tenancy plan; it should not drive the new architecture.

The proposed encrypted `PersonProfile`/address boundary is disabled, has no active reader/writer and still fixes its authenticated-encryption tenant context to Kaizen. It is a design reference, not an approved reusable implementation; ART-156 and ART-197 must define and approve the neutral protected-data boundary first.

## Public-intake convergence

The newer privacy-safe intake aggregate has good token, suppression, preview and idempotent-effect mechanics, but currently accepts only `free_trial` and `event_registration`. Its action routes deliberately return a neutral `503` until purpose adapters exist. In parallel, legacy event registration is retired (`410`) while legacy free-trial booking still writes Guardian/Member records.

RiverThames should not inherit both paths. ART-160 and ART-166 must converge enquiry, trial, waitlist, workshop and private-session intent onto one typed lifecycle backed by the neutral identity/enrolment aggregates.

What is missing is validated **Organisation, Site, Brand and terminology configuration**, not row-level shared-database tenancy. ART-149 owns those concepts; ART-152 proves configuration/deployment isolation. ART-174 reassesses SaaS tenancy only after two isolated deployments are stable.

## Data that should become configuration

- organisation/site/brand names, contacts, logo/theme, navigation and legal links;
- locale, timezone, currency and public domains/origins;
- roles and capability assignment policy, privileged-action assurance and staff credential requirements;
- audience, age/competence, level, delivery format and terminology display labels;
- venue publishable details, room/resources, availability and risk gates;
- term/block names, closures, cancellation/refund/proration/notice rules;
- Stripe catalogue references and email/analytics/provider settings (identifiers supplied per deployment, never committed as tenant defaults);
- consent purposes, channels, notices, retention/legal-hold policy and controller contacts;
- feature/module state and allowed vertical dependencies.

Secrets, personal data and provider customer identifiers must never be supplied through source-controlled tenant configuration.

## Art-school gaps mapped to existing work

| Gap | Evidence in current model | Existing owner |
|---|---|---|
| Neutral Programme/Offering/Session | Absent; Discipline/ScheduledClass only | ART-158 |
| Rooms, facilities, equipment, material constraints and conflicts | Location only; JSON facilities but no constrained resources | ART-159 |
| Adult self-enrolment and neutral payer/authority | Mandatory Guardian→Member chain | ART-153, ART-160 |
| Trials, capacity holds, transfers, catch-ups and expiring vacancy offers | Partial free-trial/waitlist flow | ART-160 |
| Blocks, packs, workshops, drop-ins, materials and concessions | Membership categories insufficient | ART-161 |
| Refund/credit/transfer/dispute operations | Strong partial ledger/reconciliation, incomplete neutral lifecycle | ART-162 |
| Multiple teachers/assistants, substitution, ratios and collection | One instructor and basic attendance | ART-155, ART-163 |
| Emergency, medical/allergy and accessibility profile gates | Proposed encrypted profile boundary, not approved/live | ART-156, CAS ART-197 |
| Artwork consent, attribution and takedown | Child media channels only | ART-157 |
| Art projects, media, skills and resource semantics | Martial technique/syllabus only | ART-169 |
| Privacy-safe occupancy/funnel/margin reporting | No complete art reporting aggregate | ART-167 |
| Generic import/export and isolated restore | Kaizen import/DSR-specific implementations | ART-168, ART-173 |

No new issue is required for these gaps.

## Migration policy recommendation

1. Freeze Kaizen's live schema/history as a source system; do not rewrite it for RiverThames.
2. Under ART-151 create a clean forward-only baseline in the sanitised repository.
3. Port security/compliance/provider tests and service invariants in small modules.
4. Design identity (ART-153) and programme/session (ART-158) before enrolment, commerce or registers.
5. Use only synthetic fixtures until the new controller/DPIA/supplier/safeguarding gates are approved.
6. Treat any future live import as a separately approved, dry-run/idempotent, source-mapped operation under ART-168—not as repository extraction.
