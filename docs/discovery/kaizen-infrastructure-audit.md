# ART-146 — Kaizen infrastructure and delivery audit

**Evidence revision:** `44101ef2fc772e0903ff926247a01597a439a2a3`
**Inspection:** source, GitHub metadata and authenticated Azure CLI read-only checks on 2026-11-06.
**Safety:** no resources, roles, settings, files or secrets were changed; no secret value or firewall address was read.

## Executive finding

Kaizen has a strong infrastructure-as-code and passwordless deployment foundation, but its current deployment is an integrated Kaizen product stack, not a reusable organisation template. Naming, domains, image ownership, database identity, email, CORS, GitHub/OIDC and several runtime modules are fixed. Generic media storage is also entangled with the martial-library video pipeline.

ART-152 must turn this into a validated per-organisation contract and prove isolation before ART-170 provisions anything. ART-170 must then create a wholly separate RiverThames DEV stack with synthetic data only. Cloning `kia-dev-rg` or changing a prefix is not sufficient.

## Source inventory

| Surface | Verified inventory | Classification / action |
|---|---:|---|
| Bicep | 27 files: `main.bicep` + 26 modules; 98 resource declarations and 42 module declarations | CORE structure; extensive CONFIGURABLE CORE; martial video and private-stage pieces are optional vertical/deferred modules |
| Bicep parameter/config | `main.dev.bicepparam`, `main.parameters.json`, three checked-in JSON configs | Replace Kaizen policy/schema names and enforce organisation-specific validation |
| `azure.yaml` | One infra-only azd project with mandatory Kaizen private-plane preflight | Parameterise/replace under ART-152 |
| GitHub Actions | 5 workflows, 13 jobs, 37 action uses | Reuse validation/security patterns; make deploy workflows reusable and organisation-scoped |
| Action supply chain | All 37 remote action uses are commit-SHA pinned | CORE control to preserve |
| Scripts | 10 files | Split generic deployment helpers from Kaizen inventory/private-stage operations |
| Shared package | 8 files; `index.ts` 4,623 lines; generated client 23,870 lines | Split/regenerate after ART-150/151; it contains martial and provider surfaces |

## Bicep module map

| Slice | Files | Classification | RiverThames action |
|---|---|---|---|
| Composition | `infra/main.bicep` | CONFIGURABLE CORE | Replace fixed environment, prefix, region, tags, image, DB, domains, email and scale choices with a validated organisation/deployment contract. |
| Core runtime | `containerApp.bicep`, `containerAppsEnv.bicep`, `staticWebApp.bicep` | CONFIGURABLE CORE | Preserve managed identity/ingress/scale patterns; module-gate feature env and parameterise all names/origins/images. |
| Data/secrets | `postgres.bicep`, `storage.bicep`, `keyVault.bicep`, `emailSecrets.bicep` | CONFIGURABLE CORE | Create distinct per-organisation resources; split generic media from martial P8/video assets; define provider bootstrap ownership. |
| Monitoring/security | `appInsights.bicep`, `logAnalytics.bicep`, `security.bicep`, `sensitiveDataAlerts.bicep` | CORE + CONFIGURABLE CORE | Preserve telemetry/security patterns; parameterise retention/caps/metric namespaces and define privacy-safe telemetry. |
| Jobs | `containerAppMigrationJob.bicep`, `containerAppDsrErasureJob.bicep`, `containerAppJob.bicep`, `containerAppIngestJob.bicep` | Mixed | Migration is core; DSR is approval-gated; ingest/transcode belong to an optional video/media module. Parameterise schedules and activation. |
| Job handoff | `dsrJobHandoff.bicep`, `videoJobHandoff.bicep`, `dsrErasureBreakGlassSecrets.bicep` | DEFERRED / module-specific | Keep out of base deployment unless owning compliance/media module and activation gate are selected. |
| Protected-data preparation | `memberDataValidation.bicep`, `memberDataRecoveryPreparation.bicep` | DEFERRED pattern | Revisit only after ART-156 defines neutral protected-profile data and key/recovery policy. |
| Private network/runtime | `privateMemberDataNetwork.bicep`, `privateMemberDataEndpoints.bicep`, `privateContainerAppsEnv.bicep`, `privateStorageEndpoints.bicep` | DEFERRED CONFIGURABLE CORE | Avoid copying fixed CIDR/names/DNS assumptions; decide whether first DEV needs a parallel private runtime. |
| Private-stage governance | `privateMemberDataPlaneStageStamp.bicep`, `privateStageApprovalGate.bicep` | DEFERRED pattern | Rename schemas and make approval evidence organisation/module-specific. |

The 26-module count includes all files listed above. `storage.bicep` currently couples core media containers with P8 video queue/Event Grid resources; `containerApp.bicep` always emits video/ffmpeg configuration. ART-150 must separate those ownership boundaries before ART-152 composes a vanilla stack.

## CI/CD and OIDC map

| Surface | Current behavior | Reuse decision |
|---|---|---|
| `.github/workflows/api.yml` | Tests/builds API, publishes the fixed Kaizen image and deploys fixed DEV resources | Extract reusable validation/build/deploy calls; bind RiverThames to its own image, RG/app and protected environment. |
| `.github/workflows/web.yml` | Tests/builds web and deploys to fixed Kaizen SWA/domain | Parameterise organisation config/origin/API endpoint; use a distinct SWA and approval environment. |
| `.github/workflows/security.yml` | Security/SBOM/IaC checks; controls are documented as report-only in places | Preserve, define blocking thresholds before production. |
| `.github/workflows/personal-data-registry.yml` | Validates personal-data registry discipline | Preserve and retarget to neutral modules. |
| `.github/workflows/research.yml` | Validates 68 martial research records | MARTIAL-ARTS VERTICAL; exclude from vanilla/ART. |
| Workflow reuse | No `workflow_call`; no GitHub `environment:` gate | ART-152 should create reusable workflows and DEV/production approval boundaries. |
| OIDC | Live app trusts Kaizen repository main/PR subjects and has Contributor on the Kaizen RG | Create a separate ART app/service principal, exact repository/environment subjects and least-scope role assignment. Never reuse Kaizen federation. |
| OIDC bootstrap | Live name/credential names differ from `setup-oidc.ps1` defaults | Choose an explicit, reproducible bootstrap/ownership process and reconcile all drift. |

`azure.yaml` fixes `kiazen-ecosystem` as project/template and always invokes the Kaizen private-member-data preflight. The root package and `@kaizen/shared` names, image path and lockfiles are also coupled. The large generated OpenAPI client (300 paths / 362 schemas in the checked-in snapshot) must be regenerated from the selected module composition, not copied as the core contract.

## Hard-coded configuration inventory

Primary hotspots are `infra/main.bicep`, virtually every resource module, `azure.yaml`, API/web deployment workflows, `scripts/setup-oidc.ps1` and the private-plane preflight.

| Concern | Current coupling | ART-152 requirement |
|---|---|---|
| Environment | Only `dev\|prod` accepted | Validated deployment stages without assuming shared resources; promotion policy explicit. |
| Region | Fixed defaults/allow-lists | Organisation/environment parameter with service-availability validation and data-residency decision. |
| Resource identity | `kia-*`, Kaizen tags and fixed RG/vault patterns | Collision-safe prefix/name factory; organisation, environment, release and management tags. |
| Database | Fixed DB/admin identifiers | Per-deployment database/server identity; credentials sourced/rotated outside Git. |
| Image/package | Fixed Kaizen GHCR repository and package names | Sanitised repository-owned image/package coordinates and provenance. |
| Web/domain | Fixed Kaizen domains, origins and CORS | Validated primary/alternate origins; custom domains managed declaratively or explicitly reconciled. |
| Email | Fixed sender, Reply-To, subject prefix and partial provider setup | Organisation-specific sender/domain/provider/config, approval gate and safe non-production recipient policy. |
| Stripe | API contracts and live vault object names exist; no Bicep ownership | ART-171-owned test account/catalogue/webhook namespace; define IaC vs runbook boundary without exposing values. |
| Scaling/cost | Fixed CPU/memory/replicas/SKUs, schedules and telemetry caps | Environment cost profile, budget/alerts, schedule ownership and scale-to-zero decision. |
| Network | Fixed private CIDR/names and public-cloud DNS suffixes | Explicit public/private DEV decision, parameter validation and no overlap. |
| Storage/video | Generic media plus P8 queue/Event Grid/video configuration | Separate core media from optional advanced-video module. |
| Telemetry | Kaizen metric names, fixed sampling/retention/cap | Neutral namespaces, release/config version, privacy review and per-organisation workspace. |
| Repository trust | Fixed GitHub repository and federation subjects | Per-repository/per-environment OIDC subjects and least-privilege scopes. |

## Read-only Azure snapshot

The Kaizen DEV resource group contained 14 top-level resources at inspection:

- 12 tagged Bicep-managed resources covering Log Analytics, storage, shared and member-validation vaults, Static Web App, Container Apps environment, Application Insights, API Container App, PostgreSQL, Event Grid, video ingest and video transcode;
- 2 untagged provider-created Application Insights smart-detection resources.

Runtime/cost characteristics:

- API uses system identity, the exact audited source image, external ingress, 0.25 CPU/0.5 GiB and 0–3 replicas;
- video ingest is event-triggered at 0–3 executions; transcode wakes every 15 minutes (up to 96 times/day);
- PostgreSQL v16 is B1ms/32 GiB, seven-day backup, public/password access, HA and geo-backup off;
- SWA is Free in West Europe; its two live custom domains are not declared in Bicep;
- storage is Standard LRS with network default `Allow`; shared-key access is not explicitly disabled;
- the shared vault is public with seven-day soft delete; the member vault is public with purge protection/90-day soft delete;
- Log Analytics is capped at 1 GB/day/30 days and Application Insights samples 50%; ingestion/query are public;
- there are no resource locks; budget, blob versioning and blob soft-delete/retention are not declared.

PostgreSQL is the fixed DEV cost floor. API/event ingest can scale to zero but cold-start. The parallel private stack adds material runtime/network/DNS cost. ART-152 needs an explicit cost envelope rather than inheriting current SKUs.

## Drift and unmanaged state

- The Azure handover omits Event Grid and both live video jobs.
- Two provider-created monitoring resources contradict the statement that every resource is Bicep-tagged.
- The documented protected-data stage tag is absent from the resource group.
- The member-validation vault remains live while its checked-in deployment gate defaults off; conditional Bicep does not delete retained resources.
- Two legacy PostgreSQL client firewall rules and the two SWA custom domains are not declared in Bicep.
- Stripe credential objects exist by name in the live vault but Stripe has no Bicep composition. Values were not accessed.
- Live OIDC names differ from script defaults; GitHub has the expected Azure variable names and only Kaizen E2E secret-name surfaces.

ART-152 should inventory/import/remove drift deliberately; ART-170 must not clone it.

## Exact ART-152 exit criteria

ART-152 is complete only when:

1. an organisation/deployment schema validates prefix, environment, region, domains, image, SKUs, scaling, telemetry, network and selected modules;
2. module composition excludes routes/jobs/storage/events/provider settings as well as UI;
3. core media is separated from martial/video resources;
4. Bicep lint/build and subscription-scope what-if pass without secret values;
5. two synthetic organisations prove distinct URLs, databases, blobs/queues, vaults, identities, email/provider namespaces, caches and telemetry;
6. reusable workflows use organisation-specific protected environments and exact OIDC subjects;
7. the runtime exposes core release and configuration-schema versions for drift diagnosis;
8. cost/scale, backup/retention, public/private network and drift-ownership decisions are recorded;
9. forbidden Kaizen/tenant assets and identifiers fail an allow-list-based repository scan;
10. no Azure resources have to be provisioned to demonstrate the template contract.

## ART-170 provisioning prerequisites and target

ART-170 remains blocked by ART-152. When unblocked, it must provision from code:

- a distinct ART resource group;
- a distinct database/server and synthetic-only schema/data;
- distinct storage, Key Vault, managed identities and telemetry;
- a distinct deployment app/service principal and exact GitHub OIDC federation;
- selected-module jobs only, with DEV cost/scale settings;
- no route, identity, network or permission capable of reaching Kaizen runtime data.

ART-171 then owns separate domain, sender/DNS policy, email provider and Stripe **test** catalogue/webhook configuration. It is correctly blocked by ART-170 plus the operating-entity/controller/insurance decision (ART-181) and DPIA/ROPA/privacy/retention/supplier approvals (ART-197).

## Recovery and content boundaries

Recovery cannot be claimed from the current stack: planned production is still HA-off with seven-day locally redundant backup, storage recovery settings are incomplete, and no restore rehearsal was evidenced. ART-168/173 own backup/export/DSR foundations and an isolated restore/key/provider recovery rehearsal.

Extraction must exclude Kaizen/Cygnets data, ignored local artefacts, environment/Azure state, secrets, provider identifiers, live resource configuration, 68 martial research files, tenant media/logo assets, seed modules, Kaizen compliance/legal material, handover PDFs/plans/URL maps, WordPress/membership-import artefacts and martial shared types. Use an allow-list, not a copy followed by deletions.

## Human decisions before ART-152/170

- resource prefix, primary region and data-residency constraints;
- sanitised repository/image/package ownership;
- OIDC bootstrap responsibility under the IaC-only rule;
- public versus private first-DEV networking;
- whether generic media and/or advanced video is in MVP;
- monthly DEV cost ceiling and cold-start tolerance;
- backup/retention/HA targets;
- domain, email and Stripe account ownership (activated later through ART-171).
