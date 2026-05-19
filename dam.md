# SMART Permission Tickets Narrative Domain Analysis Model

Status: Working narrative DAM

Source basis: Interviews with 31 stakeholders representing patient app developers,
patients, caregivers, EHR vendors, provider organizations, payers, CMS, public
health, registries, consent management, identity architecture, and health
information networks.

Purpose: This document captures the domain problem, stakeholder expectations,
conceptual workflows, information concepts, requirements, constraints, and open
questions for SMART Permission Tickets. It is intended as a narrative foundation
for later generation of formal artifacts such as use cases, storyboards, UML
models, sequence diagrams, requirements matrices, and implementation guide
material.

This is not an implementation specification. It deliberately focuses on what the
domain needs and why, before deciding how the final token format, API profiles,
conformance language, and certification behavior should be written.

## 0. Method and DAM Posture

This DAM is written from a domain-analysis perspective:

- It uses stakeholder language and workflow narratives before technical design.
- It separates requirements from implementation choices.
- It captures both static semantics (actors, concepts, relationships) and
  dynamic semantics (current and future workflows).
- It treats requirements as traceable to interview evidence, but does not claim
  statistical representativeness.
- It intentionally preserves tensions and minority concerns rather than forcing
  premature consensus.
- It identifies which problems should be solved by Permission Tickets and which
  should remain in adjacent workstreams.

Some interviews were partial, exploratory, or interrupted. Those signals are
used cautiously as directional evidence. The stronger conclusions in this DAM
come from themes repeated across stakeholder groups or from comments grounded in
specific operational experience.

## 1. Executive Summary

Healthcare authorization is fragmented across portals, app registration
processes, network participation agreements, patient consent forms, proxy
paperwork, faxed records, and local policy engines. Patients and caregivers are
asked to repeat identity and authorization ceremonies at every data holder.
Payers and public health agencies often depend on broad network trust,
site-specific onboarding, or manual follow-up. EHR vendors and provider
organizations must protect sensitive data, satisfy privacy obligations, and
retain local control, but the mechanisms available to them are often coarse,
static, or operationally expensive.

SMART Permission Tickets are being explored as a portable authorization
mechanism. Conceptually, a Permission Ticket is a signed, scoped authorization
artifact issued by a trusted party and presented to a data holder or intermediary.
It carries verifiable context such as:

- Who is requesting access
- Whose data is involved
- Who or what authorized the request
- What data is requested
- Why the data is requested
- What constraints apply
- How long the permission lasts
- Which client or system may use the permission
- How the access should be audited or revoked

The ticket is not a promise that all requested data will be released. The ticket
sets an authorization ceiling. The data holder remains responsible for validating
the ticket, applying local law and policy, honoring applicable consent and
sensitivity rules, resolving the patient safely, and issuing a down-scoped access
token or denial.

The interviews show strong support for the problem being solved, but also reveal
a consistent boundary condition: a Permission Ticket will succeed only if it is
part of a broader trust, policy, consent, and implementation ecosystem. The token
mechanics alone are not enough.

## 2. Domain Problem

The current authorization environment creates avoidable friction and risk.

For patients, access usually depends on repeated portal discovery, account
creation, username and password recall, MFA, consent screens, and refresh-token
maintenance. Several interviewees described users simply giving up, especially
when a health system's Patient Access API is not enabled, when patients never
created portal credentials, or when multiple portals are required.

For caregivers, legal authority often has to be re-proven for every provider and
every call. Faxed powers of attorney, drivers' licenses, custom forms, and
front-desk interpretation create delays, denials, and emotional burden.

For patient app developers, the N-portal model creates drop-off and operating
cost. Developers maintain thousands of endpoints and relationships, handle site
quirks, and still cannot reliably connect users to all records.

For provider organizations, external data access is often treated as a
mini-project. Business owners, informatics, API teams, HIM, privacy, security,
BAAs, IRBs, and legal agreements may all be involved before data can flow.

For payers, requests for clinical documentation for claims, prior authorization,
risk adjustment, and quality often remain portal-heavy and document-heavy. PDFs,
images, text files, scanned records, and phone or fax follow-up are common.

For public health and registries, initial reporting may be electronic, but
follow-up often falls back to phone calls, email, fax, letters, or remote EHR
access negotiated site by site.

For EHR vendors, broad backend access and all-or-nothing app approval create
security, privacy, and customer-control concerns. Several vendors emphasized that
validating a signed artifact is not the main challenge. The harder work is
integrating the authorization context into every relevant API, policy engine, and
data filtering path.

For health information networks and intermediaries, there is a lack of mature,
shared guidance on how to carry authorization context through a network-mediated
FHIR exchange. OIDs, purpose of use, user role, network agreements, and scope
filters are assembled differently across networks.

For consent management services, the consent decision is often trapped in a
static document. Even when consent is digitized, variation in forms and limited
EHR segmentation capability constrain how computable consent can be enforced.

The shared domain problem is therefore not only "how to create a token." It is:

How can healthcare actors express, convey, validate, enforce, audit, and revoke
portable authorization decisions in a way that reduces repeated ceremonies while
preserving local policy, privacy, patient agency, and implementability?

## 3. Business Objectives and Success Outcomes

The purpose of SMART Permission Tickets is not only to introduce a new
cryptographic artifact. The business objective is to make healthcare data access
more portable, scoped, governable, and auditable across organizational
boundaries.

### 3.1 Business Objectives

The domain requires a mechanism that can:

- Reduce repeated identity and authorization ceremonies for patients and
  caregivers
- Reduce site-by-site manual onboarding for recurring, well-governed access
  patterns
- Allow authorization context to travel with a request in a verifiable form
- Preserve data-holder control over final release decisions
- Support finer-grained access than broad network participation or all-or-nothing
  app approval
- Provide a common audit anchor for distributed access events
- Enable networks and intermediaries to carry authorization context consistently
- Support patient-facing permission experiences with more consistent data
  categories
- Avoid requiring current EHRs to enforce segmentation they cannot safely support
- Create a practical migration path from today's portal, file, fax, and
  agreement-heavy workflows

### 3.2 Success Outcomes

A Permission Ticket approach is successful if:

- Patients can connect apps to more data holders with fewer failed login flows.
- Caregivers can prove authority once and have it recognized consistently.
- Payers can request claim- or prior-authorization-specific evidence without
  broad standing access.
- Public health agencies can obtain condition-specific follow-up data without
  routine phone, email, or fax.
- Registries can obtain case-completion data more consistently.
- Provider organizations can approve trusted issuers or categories at a high
  level while retaining local policy control.
- EHR vendors can validate tickets through standard infrastructure and apply
  constraints consistently across APIs.
- Networks can represent "on behalf of" requests in a portable, verifiable way.
- Consent services can produce computable outputs that data holders can evaluate
  alongside local rules.
- Audit trails can explain who accessed what, why, under which authority, and
  within which constraints.

### 3.3 Non-Success Outcomes

The project should not measure success by whether tickets can carry arbitrary
fine-grained constraints that systems cannot enforce. A beautifully expressive
ticket that causes inconsistent or unsafe enforcement is not a successful DAM
outcome.

The project should also not measure success by replacing every current trust or
legal arrangement. Many workflows will continue to require DUAs, BAAs, IRBs,
state reporting laws, network participation agreements, and local policy review.

## 4. DAM Scope

### 4.1 In Scope

This DAM covers:

- Portable authorization artifacts for healthcare data access
- Patient-directed access to records through apps
- Caregiver, proxy, and authorized representative access
- Payer requests tied to claims, prior authorization, quality, risk adjustment,
  or other covered workflows
- Public health follow-up after reportable events
- Registry follow-up, including cancer registries and other condition-specific
  reporting programs
- Care coordination and referral workflows, including community-based
  organizations and downstream providers
- Backend system-to-system access where patient-level or context-level scoping
  is needed
- Intermediary and network-mediated exchange
- Issuer trust, requester identity, organization identity, subject identity, and
  client binding
- Scoping, duration, revocation, auditability, and discoverability
- Interaction with local policy, consent, DS4P-style segmentation, and
  jurisdictional privacy rules at a conceptual level

### 4.2 Out of Scope for This DAM

This DAM does not define:

- A final JWT schema or grant type
- A required identity proofing vendor or workflow
- A universal patient identifier
- A complete consent form standard
- A complete clinical data segmentation standard
- A national trust framework
- Replacement for BAAs, DUAs, IRBs, state reporting laws, HIPAA, 42 CFR Part 2,
  or other legal authority
- A guarantee that receiving systems will integrate returned data into clinical
  workflow
- A general write-back model for all EHR data
- Governance for autonomous AI agents, beyond identifying it as an emerging
  requirement

### 4.3 Boundary Principle

Permission Tickets should standardize the portable authorization mechanism and
the core context needed to evaluate a request. They should not attempt to solve
every downstream scoping vocabulary, clinical relevance, segmentation, consent
form, or workflow integration problem in v1.

Where possible, the work should lean on existing or parallel specifications:
SMART, FAST Security, UDAP, HL7 Consent, HL7 Scalable Consent Management, DS4P,
FHIR security labels, US Core, Da Vinci, TEFCA, and relevant public health or
registry implementation guides.

## 5. Stakeholder Landscape

### 5.1 Patients and Self-Representatives

Patients want simpler access to their records without needing to remember every
portal account. They want portability, transparency, and enough control to
protect sensitive areas without making clinical sharing impossible.

Interview themes:

- Repeated portal login is a major drop-off point.
- Many patients never create portal accounts.
- Patients may trust payers, providers, or familiar identity platforms more than
  organizations whose incentives involve data monetization.
- Most patients are willing to share broadly with clinicians, but want finer
  control for sensitive categories such as behavioral health, mental health,
  sexual health, substance use, or other delicate information.
- Patients want to know who accessed their data, what they accessed, and when.
- Access alone is not enough if receiving systems cannot integrate the data.

### 5.2 Caregivers and Authorized Representatives

Caregivers often act with full legal authority, but must repeatedly prove that
authority to each organization. The burden is administrative, emotional, and
practical.

Interview themes:

- Providers may refuse access even when legal documents exist.
- Staff turnover and lack of digital capture cause repeated re-verification.
- Permission artifacts need plain-language explanations for front-line staff.
- Credentials must be secure enough that only the authorized caregiver can use
  them.
- A primary representative may need full access, while other caregivers or
  providers need scoped access.
- The administrative burden compounds the stress of caregiving.

### 5.3 Patient App Developers

Patient app developers need to reduce connection friction and maintain
integrations across many systems. They see portable authorization as
transformational if acceptance is broad and discoverable.

Interview themes:

- Current flows fail at institution discovery, portal login, API availability,
  consent screens, and refresh-token expiration.
- Some vendors or sites do not enable Patient Access APIs by default.
- Developers need machine-readable knowledge of which data holders accept which
  issuers.
- Partial adoption is still useful if apps can fall back to portal login.
- Ticket creation must not become a new drop-off point.
- Rolling refresh patterns are important for ongoing record updates.
- App developers may want to become trusted issuers if governed under the same
  standards as other issuers.
- Conformance must be tight enough to avoid TEFCA-like underspecification
  problems.

### 5.4 EHR Vendors and Authorization Server Implementers

EHR vendors generally see ticket validation as implementable, but emphasize that
real value requires deeper API-layer enforcement.

Interview themes:

- The authorization server changes may be additive.
- The harder work is updating APIs and data access controls to use ticket
  constraints.
- Local customer policy, sensitive data filters, DS4P rules, and consent logic
  must remain authoritative.
- Data holders must be able to down-scope or deny.
- Demographic patient matching at enforcement time is risky and may be a security
  hole.
- Organization identity is a missing input in many SMART flows and may need to be
  solved both inside and outside Permission Tickets.
- Alignment with FAST Security and UDAP is critical.
- Tickets must be bound to the intended client or system.
- A generic, extensible ticket structure may be preferable for some platforms,
  while other implementers need predictable use-case-specific guidance.

### 5.5 Provider Organizations and Data Holders

Provider organizations want reduced operational burden but cannot give up local
control, privacy review, or sensitivity filtering.

Interview themes:

- Current approval processes may take weeks or months.
- Security, HIM, privacy, and API owners need to review new mechanisms.
- Pre-approving trusted issuers is more acceptable than reviewing each ticket.
- High-level policy overlays are needed per issuer or category.
- Sensitive data filtering remains local.
- Provider organizations will not trust requesters to self-filter sensitive data.
- Business rules must be considered and preserved.

### 5.6 Payers and CMS API Stakeholders

Payers see value in scoped, automated access for claims, prior authorization, and
member access. CMS stakeholders also see a need for standardized data categories
and patient-facing permission language.

Interview themes:

- RFI loops are slow, portal-heavy, and unstructured.
- Clinical documentation often arrives as PDFs, images, documents, or scans.
- Payers need a way to ask for specific data tied to a claim or workflow.
- Self-pay and sensitive data filtering are real concerns.
- Permission screens vary across the ecosystem, increasing patient confusion and
  abandonment.
- Patients need common, meaningful data categories, not only FHIR resource names.
- Implementations must avoid becoming vendor toll roads.
- Open-source, deployable reference services could be important for lean teams.

### 5.7 Public Health and Registries

Public health programs and registries need timely, condition-specific follow-up
data. They see Permission Tickets as useful if scopes match investigation needs
and governance aligns with public health authority.

Interview themes:

- Initial reports often arrive electronically through ELR, eICR, pathology feeds,
  or abstracts.
- Follow-up still often requires phone, email, fax, letters, or remote EHR
  access.
- Data needs vary by condition.
- Existing case investigation forms can define condition-specific data profiles.
- Shorter tickets renewed as investigations progress may be preferable to very
  long-lived access.
- Public health and registry systems often have limited direct FHIR capability,
  making intermediaries important.
- Data use agreements and state reporting laws remain relevant after access.
- Organization identity cannot depend only on NPIs; domain-based identifiers and
  registries may be needed.

### 5.8 Health Information Networks and Intermediaries

Networks already carry authorization context, but often without a dedicated
standard for the intermediary pattern.

Interview themes:

- Intermediaries may pass OIDs, purpose of use, user role, and filtered scopes.
- There is no mature, universally adopted intermediary IG for FHIR network flows.
- A ticket could formalize what networks already do in two-token or brokered
  models.
- The three-party pattern must be first-class: requester -> network/intermediary
  -> target.
- CMS-0057 and burden reduction may drive FHIR enablement.

### 5.9 Consent Management Services

Consent services see the promise of computable permission artifacts but warn that
consent form variation and limited EHR segmentation are practical blockers.

Interview themes:

- Static consent documents are often not searchable or enforceable.
- Forms-agnostic capture is costly when every form differs.
- A standardized output format could help, but inputs remain variable.
- Confidence in EHRs' ability to segment data at a clinical level is low.
- v1 should start with constraints systems can actually enforce.

### 5.10 Identity, Trust, and Agentic Systems

Identity architects emphasize proofing, trust frameworks, scope control, and the
emerging challenge of autonomous agents.

Interview themes:

- IAL2 or equivalent identity proofing may be a hard requirement for some
  organizations.
- Trust must be established before honoring requests, whether pre-established or
  verified at request time.
- Scope-aware architecture is essential to prevent over-provisioning.
- Network frameworks may be broader than local IAM principles prefer.
- Autonomous or agentic requesters will change how trust and scope are evaluated.

## 6. Current-State Narratives

### 6.1 Patient App Connection

A patient opens a health app and chooses to add a provider. The app helps them
search for an institution. The patient may not remember the institution name, may
not find the institution, or the institution may not expose a usable API. If the
institution is found, the patient is sent to a portal login page. The patient may
not have an account, may not remember credentials, may fail MFA, or may abandon
the flow. If login succeeds, the patient sees consent screens. The app may not
have visibility into drop-off inside the secure web view. If the flow succeeds,
data flows until the refresh token expires or is rejected.

Pain points:

- Institution discovery
- Portal account creation
- Username/password/MFA
- Multiple repeated ceremonies
- API availability and endpoint configuration
- Refresh-token expiration
- Lack of consistent permission screen language

### 6.2 Caregiver Proof of Authority

A caregiver managing a parent's care contacts a provider for records, billing
corrections, lab follow-up, coverage questions, or sharing with another provider.
The provider asks for identity and proof of authority. The caregiver faxes a
drivers' license, power of attorney, and custom authorization forms. On a later
call, the provider cannot retrieve the prior documents and asks again. A staff
member may refuse access despite the caregiver's legal authority. The caregiver
explains privacy law, information blocking, and legal authority, sometimes to a
new staff member after turnover. At times the caregiver gives up.

Pain points:

- Repeated proof
- Non-digital records of authority
- Staff uncertainty
- Inconsistent recognition of legal documents
- Privacy concerns in busy, multilingual, front-line environments
- Emotional and time burden

### 6.3 Payer Documentation Request

A provider submits services and CPT-coded packages to a payer portal. The request
enters adjudication or prior authorization review. If more evidence is needed,
the payer issues an RFI. Documentation arrives in mixed formats such as PDF, TXT,
DOCX, PNG, JPG, scanned records, or portal uploads. Staff review the material
manually. If the request is accepted, payment proceeds. If rejected, appeals may
repeat the loop.

Pain points:

- Portal-heavy exchange
- Unstructured documents
- Manual review
- Slow RFI loops
- Difficulty requesting only claim-specific clinical evidence
- Risk of over-broad payer access if backend access is granted

### 6.4 Public Health Follow-Up

An initial case report arrives electronically through ELR or eICR. A public
health investigator needs additional data such as symptoms, hospitalization,
pregnancy status, immunization history, demographics, exposure information, or
condition-specific details. The investigator contacts the provider, ordering
clinician, infectious disease coordinator, or facility staff by phone or email.
FHIR query capability is emerging through pilots and connectors, but direct
production connections are still early.

Pain points:

- Electronic initial reporting, manual follow-up
- Condition-specific data needs
- Provider responsiveness variation
- Scaling one-to-one connections
- Need to align with statewide agreements, HIEs, and public health authority

### 6.5 Registry Follow-Up

A central cancer registry receives initial reporting through pathology feeds,
hospital abstracts, or other reporting paths. To complete cases, the registry
needs specific data elements such as treatment, staging, outcomes, labs, and
facility-specific information. Follow-up may involve letters, remote EHR access,
phone, or fax. Remote EHR access requires approvals and signed agreements. Access
may not include all data needed.

Pain points:

- Slow data collection
- Variation across facilities
- Missing specific registry data elements
- Limited FHIR adoption in registries
- Need for DUAs and state-law alignment

### 6.6 Provider Integration Approval

A third-party integration request enters a provider organization's review
process. API owners, HIM, privacy, security, and business owners evaluate
accuracy, appropriateness, integrity, legal agreements, BAAs, and IRB needs. The
approval process may take several weeks. Testing and production rollout may take
additional weeks or months.

Pain points:

- High operational overhead
- Site-specific review
- Legal and privacy dependency
- Need to preserve local guardrails

### 6.7 Health Information Network Intermediary

A network receives requests from participants and forwards them to targets. It
may use a two-token process, with scopes carried or duplicated from requester to
intermediary and from intermediary to target. It may include requester identity,
OIDs, purpose of use, user role, and onboarding-defined scope filters. Targets
decide what data to return. There is no single mature intermediary guidance
covering all of this behavior.

Pain points:

- Fragmented network behavior
- Lack of common intermediary authorization artifact
- Per-network interpretation
- Need for targets to understand who is represented and why

### 6.8 Consent Capture and Enforcement

A patient signs a consent document on paper or digitally. The document is stored
in an EHR or document system, but may not be searchable, computable, or
consistently located. HIEs and providers often reduce consent to opt-in or
opt-out because granular enforcement is hard. Some consent services can extract
structured decisions into JSON, but form variation and EHR segmentation limits
remain.

Pain points:

- Static consent documents
- Form variation
- Limited computable enforcement
- Low confidence in fine-grained EHR segmentation
- Binary consent defaults

## 7. Future-State Conceptual Narrative

In the future-state model, authorization context can travel with a request as a
signed artifact. The ticket allows a data holder, network, or intermediary to
evaluate a request based on verifiable facts rather than only local accounts,
paperwork, or broad network participation.

The expected lifecycle is:

1. An authority basis is established.
   This may be a patient's explicit app authorization, a caregiver's legal
   authority, a public health reporting obligation, a claim-specific payer
   workflow, a registry mandate, a referral, a CDS Hooks invocation, or a network
   use case agreement.

2. A trusted issuer creates a Permission Ticket.
   The issuer signs a ticket containing the requester, subject, authorization
   basis, purpose, allowed data categories or scopes, constraints, expiration,
   client binding, audit metadata, and revocation or renewal information.

3. The requester presents the ticket.
   The requester sends the ticket to a token endpoint, gateway, or intermediary
   along with ordinary client authentication material.

4. The data holder or intermediary validates the ticket.
   Validation includes signature, issuer trust, audience, time bounds, client
   binding, revocation status, conformance, and subject resolution.

5. The data holder applies local policy.
   The ticket is intersected with local rules: consent, sensitive data policy,
   DS4P or security labels, jurisdictional law, customer configuration, use-case
   restrictions, and technical capability. The result may be a narrower access
   token, a partial response, or a denial.

6. The resource server enforces the decision.
   Access decisions are applied consistently across FHIR reads, searches, bulk
   export, operations, subscriptions, or other supported access patterns.

7. Access is audited and visible.
   The ticket and derived access token create clear audit events. Patients,
   caregivers, issuers, requesters, and data holders can trace who accessed what,
   when, under which permission, and with which outcome.

8. Permissions expire, refresh, or revoke.
   Some workflows require short-lived tickets. Others require long-lived grants
   that produce short-lived access tokens. Revocation and renewal must be
   supported without relying on patients or staff to repeat unnecessary
   ceremonies.

## 8. Conceptual Information Model

This section names core domain concepts. It is intentionally conceptual and does
not define a final schema.

### 8.1 Core Concepts

| Concept | Description |
| --- | --- |
| Permission Ticket | A signed, scoped authorization artifact that carries verifiable authorization context to a data holder or intermediary. |
| Issuer | The trusted party that creates and signs the Permission Ticket. Could be a data holder, network, identity service, consent service, payer, public health authority, app under governance, or other trusted entity. |
| Authorizing Party | The person, organization, policy, or legal authority that provides the basis for access. Examples: patient, caregiver, provider, public health law, registry mandate. |
| Subject | The patient, member, person, or record subject whose data is requested. |
| Requester | The application, system, organization, user, agent, or intermediary seeking access. |
| Data Holder | The organization or system that controls the requested data and decides what to release. |
| Intermediary | A network, HIE, QHIN, HIN, gateway, or platform that carries or helps evaluate authorization context between requester and data holder. |
| Trust Framework | The governance structure under which issuers, requesters, data holders, and intermediaries decide whom to trust. |
| Issuer Registry | A machine-readable or governed list of trusted issuers and their identifiers, keys, roles, categories, or accreditation status. |
| Organization Identity | The identity of the organization on whose behalf a request is made. This may be domain-based, OID-based, NPI-associated, or otherwise registered. |
| Client Binding | A mechanism tying the ticket to the intended client, key, or system so it cannot be replayed by another party. |
| Authorization Context | The clinical, operational, legal, or workflow context that explains why access is needed. Examples: claim, case, referral, app authorization, public health investigation. |
| Purpose of Use | A coded or otherwise standardized statement of why data is requested. |
| Scope | The requested data boundary. May include FHIR scopes, data categories, resource groups, time periods, condition-specific profiles, security labels, or security attributes. |
| Security Attribute | A data attribute used for access filtering, such as organization, department, encounter, referral, case, group, or claim association. |
| Local Policy Overlay | The data holder's own rules that constrain or deny access even when the ticket is valid. |
| Consent / Preference / Authorization | A rule or decision expressing what the patient, representative, law, or organization permits. These must be evaluated consistently, not as disconnected artifacts. |
| Subject Resolution | The process of matching or resolving the ticket's subject to the data holder's local patient/member/person record. |
| Access Token | The down-scoped token issued by the data holder after validating the Permission Ticket and applying local policy. |
| Audit Event | A record of ticket issuance, presentation, validation, token issuance, data access, denial, revocation, or partial response. |
| Revocation Event | A change indicating that a prior permission is no longer valid. |
| Data Response | The data returned, withheld, partially returned, redacted, or denied as a result of access enforcement. |
| Discovery Metadata | Machine-readable information indicating which data holders accept which issuers, ticket types, purposes, scopes, or trust frameworks. |

### 8.2 Relationships

- A Permission Ticket is issued by exactly one issuer.
- A Permission Ticket is based on one or more authority sources.
- A Permission Ticket is presented by a requester or intermediary.
- A Permission Ticket applies to one or more subjects or a defined group.
- A Permission Ticket is bound to one or more intended clients, systems, or keys.
- A Permission Ticket contains one or more scopes or data constraints.
- A data holder validates the Permission Ticket against issuer trust and local
  policy.
- A valid Permission Ticket may result in a down-scoped access token.
- The access token must not exceed the Permission Ticket.
- The access token may be narrower than the Permission Ticket.
- Data access must be auditable back to the Permission Ticket and authority
  basis.
- Revocation or expiration invalidates future use of the Permission Ticket or
  derived tokens according to the applicable lifecycle rules.

## 9. Use Case Narratives

### 9.1 Patient Longitudinal Record Access

Goal: A patient authorizes an app to retrieve records from multiple data holders
without logging into every portal separately.

Current pain:

- Patients forget or never create portal accounts.
- Multiple EHR accounts, even within one vendor ecosystem, create friction.
- API endpoints may be unavailable.
- Refresh tokens expire or fail.
- Permission screens are inconsistent and confusing.

Ticket role:

- A trusted issuer verifies identity and captures authorization.
- A signed ticket is presented to participating data holders.
- Data holders issue down-scoped access tokens based on accepted scopes and local
  policy.
- Apps use machine-readable discovery to decide where ticket-based connection is
  available.

Key requirements:

- Ticket creation must be simpler than portal login.
- Partial adoption must be supported with fallback to existing flows.
- Periodic refresh must be supported, potentially using rolling windows.
- Scopes should preserve existing SMART semantics while allowing more
  patient-meaningful data categories.
- Patients need clear revocation and access logs.

Open issues:

- Who is the trusted issuer for broad patient access?
- Can a provider the patient already uses bootstrap tickets for other providers?
- What discoverability metadata is required at scale?
- How are non-enabled FHIR endpoints handled?

### 9.2 Caregiver and Authorized Representative Access

Goal: A caregiver proves authority once and uses a portable credential across
providers.

Current pain:

- Repeated faxing of identity and authority documents.
- Legal authority not recognized consistently.
- Staff lack confidence interpreting documents.
- Providers cannot digitally retain or retrieve prior proof.

Ticket role:

- A trusted service verifies the caregiver's identity and relationship or legal
  authority.
- The ticket states the caregiver's authority, scope, duration, and basis.
- Front-line staff and systems can understand the permission in plain language
  and machine-readable form.

Key requirements:

- The authority basis must be explicit: power of attorney, patient delegation,
  parent/guardian, policy, court order, or other basis.
- Scope may be full representative access or limited caregiver access.
- Revocation and change in authority must be supported.
- The ticket must be usable only by the intended caregiver or system.
- Plain-language explanatory text is required for staff-facing workflows.

Open issues:

- How are legal documents verified?
- How is authority transitioned when a minor reaches majority?
- How do providers handle conflicts among multiple representatives?

### 9.3 Payer Claim, Prior Authorization, and RFI Data Access

Goal: A payer requests specific clinical documentation for a claim, prior
authorization, or similar workflow through a scoped, verifiable mechanism.

Current pain:

- Documentation arrives through portals, fax, or unstructured files.
- RFI loops are slow and manual.
- Backend payer access may be broad and difficult to filter.
- Self-pay and sensitive data may be disclosed inappropriately if local filtering
  is weak.

Ticket role:

- The ticket ties the request to a claim, prior authorization, service, member,
  date range, and data category.
- The provider validates the ticket and issues an access token limited to that
  workflow.
- Local policy excludes data the payer should not receive.

Key requirements:

- Ticket constraints must include "who", "what", and workflow context.
- The data holder must be able to down-scope or deny.
- Organization identity of the payer and any acting app must be reliable.
- The model should support patient-meaningful data categories and technical FHIR
  mappings.
- Audit must show why payer access was granted.

Open issues:

- How are self-pay filters represented and enforced?
- How much claim context is required?
- How does this interact with CMS-0057 and Da Vinci workflows?

### 9.4 Public Health Follow-Up

Goal: A public health agency queries for follow-up data after an initial report
without manual phone, email, or fax follow-up.

Current pain:

- Initial reporting is electronic but follow-up is manual.
- Data needs vary by condition.
- Provider responsiveness varies.
- Direct FHIR query infrastructure is early.

Ticket role:

- The initial case report, case workflow, or public health system triggers a
  ticket for condition-specific follow-up.
- The ticket authorizes query access for the relevant patient, condition,
  investigation, and data profile.
- The public health agency presents the ticket to the provider, HIE, or
  intermediary.

Key requirements:

- Scope must be condition-specific.
- Existing case investigation forms can define data needs.
- Tickets may need renewal as investigations progress.
- Public health authority and purpose must be explicit.
- Organization identity must handle agencies and non-NPI entities.

Open issues:

- Should the ticket travel with eICR, ELR, a FHIR case report, or a separate
  workflow?
- Who issues the ticket: data holder, public health agency, HIE, or network?
- How are outbreak-level tickets represented?

### 9.5 Cancer Registry Follow-Up

Goal: A central cancer registry obtains specific follow-up data needed for case
completion more consistently and quickly.

Current pain:

- Follow-up may use letters, fax, phone, or remote EHR access.
- Remote access requires site agreements.
- Access often misses specific data elements.
- FHIR adoption in registries is limited.

Ticket role:

- A registry or intermediary presents a ticket tied to a cancer case, state
  authority, and required data categories.
- Facilities validate the ticket and return scoped data.
- Intermediaries may bridge between FHIR query and registry-native intake.

Key requirements:

- Governance must align with state reporting laws.
- DUAs may still govern downstream storage and use.
- Scope must include specific registry needs such as treatment, staging, and
  outcomes.
- Intermediary patterns are likely necessary for adoption.

Open issues:

- How does the ticket relate to state mandates?
- Which intermediary organizations can issue or carry tickets?
- How are registry systems upgraded or bridged?

### 9.6 Care Coordination and Referral Workflows

Goal: A downstream provider or community-based organization receives scoped
access to referral-related data and possibly updates referral status.

Current pain:

- Referrals and status updates often rely on Direct, fax, portals, or custom
  workflows.
- Write-back is poorly supported by many systems.
- Broad access is unnecessary and risky.

Ticket role:

- The ticket authorizes access to a specific referral, service request, task, or
  care journey context.
- The downstream actor can view relevant data and, where supported, update
  permitted workflow status.

Key requirements:

- v1 may focus on read and workflow triggers.
- Write-back requires clinical oversight and system functionality.
- Scope should follow the care journey context, not the whole record.
- Data holder local policy must decide whether updates are accepted.

Open issues:

- What write operations are safe for v1?
- How is clinical oversight represented?
- How are CBO identities and roles governed?

### 9.7 Intermediary and Network-Mediated Exchange

Goal: A HIN, HIE, QHIN, or other intermediary carries verifiable authorization
context between requester and target in a standardized way.

Current pain:

- Networks use different models.
- Intermediaries assemble identity, purpose, user role, and scope context from
  onboarding agreements and token extensions.
- Targets may not know how to interpret network-specific context.

Ticket role:

- The intermediary may issue, carry, transform, or validate a ticket.
- The ticket formalizes "this request is made on behalf of this participant for
  this purpose under this trust framework."
- The target can verify the artifact independent of the intermediary's internal
  token model.

Key requirements:

- The three-party pattern must be explicit.
- Requester identity must not be hidden behind the network.
- Purpose of use, role, organization, and scope must survive transit.
- The model should align with FAST Security and UDAP where applicable.

Open issues:

- When is the intermediary the issuer versus the carrier?
- What metadata must be preserved end to end?
- How are network-level agreements linked to transaction-level tickets?

### 9.8 CDS Hooks and On-Demand Access

Goal: A CDS service receives a narrowly scoped authorization artifact and
exchanges it for access only when needed.

Current pain:

- Generic tokens may be issued up front.
- Tokens may be unused or too broadly scoped.

Ticket role:

- The EHR or decision service receives a ticket linked to a hook invocation.
- The service exchanges it for a short-lived token only if it needs data.

Key requirements:

- Ticket must be bound to the intended CDS service.
- Scope must reflect the hook context.
- Alignment with current CDS Hooks token exchange patterns is needed.

Open issues:

- Does this replace or complement existing CDS Hooks token exchange?
- What hook context is required in the ticket?

## 10. Business Rules and Requirements

These requirements are expressed narratively. They can be converted later into
formal SHALL/SHOULD/MAY language.

### 10.1 Trust and Issuer Governance

R-1. A data holder must be able to determine whether the ticket issuer is trusted
for the requested use context.

R-2. Trust may be pre-established, discovered through a governed registry, or
validated at request time through a trusted framework. In all cases, trust must be
established before access is honored.

R-3. Issuer identity should be machine-verifiable. Domain-based identifiers,
published JWKS endpoints, DNS-anchored trust, OIDs, NPIs, and network registries
may all play roles, but the model must not depend only on NPIs.

R-4. Data holders must be able to limit which ticket types, scopes, purposes, and
requester categories a trusted issuer may authorize.

R-5. Category-based approval must preserve data-holder control. Adding new apps
or organizations to an approved category without notice or re-approval may create
unacceptable trust risk.

R-6. Issuer governance must address conflicts of interest if an app developer,
network, payer, or data holder also acts as an issuer.

### 10.2 Requester and Organization Identity

R-7. The ticket must identify the requester and, where applicable, the
organization on whose behalf the requester acts.

R-8. Organization identity is needed even outside Permission Ticket flows and
should be coordinated with broader SMART and FAST identity work.

R-9. A requester's app identity, user identity, organization identity, and purpose
may all be distinct and should not be collapsed into a single client ID.

R-10. Tickets must support client binding so that a ticket can only be used by the
intended system, key, or client.

### 10.3 Subject Resolution

R-11. Solid identifiers are strongly preferred for subject resolution.

R-12. Demographic matching at authorization enforcement time should be avoided or
treated as high risk. If demographic matching is necessary, it should be a
separate workflow with clear ambiguity handling before access is granted.

R-13. If subject resolution is ambiguous, the data holder should deny or require a
separate resolution step rather than guessing.

R-14. The ticket should record what subject identifier or matching basis was used
so access can be audited and challenged.

### 10.4 Scope and Data Categories

R-15. The ticket must express the maximum access being requested or authorized.

R-16. The data holder must be able to down-scope, filter, redact, or deny based on
local policy and capability.

R-17. v1 scoping should focus on what can be consistently understood and enforced
today: requester, recipient, subject, purpose, broad data category, resource
group, time period, encounter, claim, case, referral, condition, or organization.

R-18. The scoping vocabulary should not be limited to FHIR resource names. It
should support patient-meaningful and business-meaningful categories such as
"lab results", "coverage", "claims", "medications", "mental health information",
"immunization history", or "treatment records", with technical mappings.

R-19. Sensitive data segmentation should start with broad enforceable categories
and evolve toward finer clinical segmentation as EHR capabilities mature.

R-20. External or existing specifications should define detailed clinical
relevance vocabularies where possible. Permission Tickets should carry such
constraints without owning every clinical scoping taxonomy.

R-21. Tickets should support condition-specific or use-case-specific profiles,
especially for public health and registry follow-up.

R-22. Tickets should support group-based or attribute-based access where the
server can dynamically evaluate group membership or security attributes at query
time.

### 10.5 Consent, Preferences, and Local Policy

R-23. The ticket must state the authority basis: patient preference, consent,
legal representative, provider action, public health authority, state mandate,
network agreement, payer workflow, or other basis.

R-24. The expression of rules inside a Permission Ticket should be coordinated
with HL7 Consent and related consent management work. Divergent rule formats
should be intentional and justified.

R-25. Consent to share must be explicit where required by the use case.

R-26. DS4P-style segmentation, local sensitivity rules, jurisdictional privacy
rules, and organizational policies must not be bypassed by the presence of a
ticket.

R-27. For patient-directed app access, data holders should distinguish the
patient's own preference from other institutional data-sharing restrictions and
apply the correct legal and policy context.

R-28. For non-patient-directed access, the ticket should be evaluated alongside
other applicable rules.

### 10.6 Duration, Refresh, and Revocation

R-29. The model should distinguish between a long-lived grant and a short-lived
access token.

R-30. Patient app access may need rolling refresh behavior similar to existing
refresh token expectations.

R-31. Referral and notification workflows may require long-lived grants, but each
actual data access event should be short-lived and scoped.

R-32. Public health investigations may prefer renewed tickets as a case evolves
rather than a broad long-lived ticket from the start.

R-33. Revocation must be supported and discoverable by the data holder at the time
of token issuance or access.

R-34. Patients and caregivers should have a practical way to stop sharing and to
understand what stopping means.

### 10.7 Audit and Transparency

R-35. Ticket issuance, presentation, validation, token issuance, access, denial,
partial response, and revocation must be auditable.

R-36. Audit records should link access back to the ticket, issuer, requester,
subject, purpose, and constraints.

R-37. Patients and caregivers should be able to view meaningful access logs for
patient-directed and representative access.

R-38. Data holders and issuers should be able to trace misuse or disputes across
distributed systems.

R-39. The ticket itself is valuable as an audit event and should be preserved or
referenced in audit logs according to policy.

### 10.8 Discovery and User Experience

R-40. Apps need machine-readable discovery of which data holders accept which
issuers and ticket profiles.

R-41. Users should not be offered a ticket-based flow when the data holder will
not accept the relevant issuer.

R-42. Patient-facing permission screens should use common, understandable data
categories.

R-43. Patient-facing screens should preserve contextual information such as why
the app wants data, what data is shared, how to revoke, and whom to contact.

R-44. Front-line staff workflows for caregiver/proxy tickets should include
plain-language explanations and multilingual support where practical.

### 10.9 Interoperability and Alignment

R-45. Permission Tickets must align with SMART, FAST Security, UDAP, and related
work to avoid duplicating existing trust and endorsement mechanisms.

R-46. If Permission Tickets and UDAP Certifications/Endorsements solve different
problems, their formats and concepts should be harmonized where possible.

R-47. Intermediary flows must be first-class, not treated as edge cases.

R-48. The model should support direct, intermediary, and network-mediated
exchange.

R-49. The specification should define conformance tightly enough to avoid
per-issuer or per-network underspecification.

### 10.10 Implementation Feasibility

R-50. Implementers need a high-quality reference implementation, test suite, and
deployment guidance.

R-51. Lean organizations may need an open-source, containerized service rather
than a vendor-only offering.

R-52. The implementation burden should be clear: token validation, trust
management, subject resolution, policy decision, API-layer enforcement,
redaction, audit, and discovery are separate concerns.

R-53. Data holders must not be required to implement fine-grained segmentation
that their systems cannot safely enforce.

R-54. Partial adoption must be allowed. Existing SMART flows, portal login,
network exchange, and manual processes will coexist during transition.

## 11. Business Rules by Actor

### 11.1 Data Holder Rules

- A data holder never grants more than the ticket authorizes.
- A data holder may grant less than the ticket authorizes.
- A data holder applies local sensitive data rules regardless of requester
  claims.
- A data holder may require issuer pre-approval or trust-framework membership.
- A data holder must be able to reject tickets from untrusted issuers.
- A data holder must be able to reject tickets with unresolved or ambiguous
  subjects.
- A data holder must audit every accepted, denied, or partially honored ticket.

### 11.2 Issuer Rules

- An issuer signs only permissions it is authorized to assert.
- An issuer must identify the authority basis.
- An issuer must publish keys or otherwise enable signature verification.
- An issuer must support revocation or expiration semantics appropriate to the
  ticket type.
- An issuer must not silently expand category membership in ways that violate the
  relying data holder's expectations.

### 11.3 Requester Rules

- A requester must authenticate as the intended client or system.
- A requester must not present a ticket issued to another requester.
- A requester must use data only for the stated purpose and within constraints.
- A requester must handle partial responses and denials.
- A requester must preserve audit context needed to explain downstream use.

### 11.4 Intermediary Rules

- An intermediary must preserve requester identity and purpose.
- An intermediary may add, translate, or constrain authorization context only
  according to its trust framework and role.
- An intermediary must not cause the target to believe the request originates
  solely from the intermediary if it is acting on behalf of another party.
- An intermediary should support verifiable artifacts that the target can
  independently inspect.

### 11.5 Patient and Caregiver Rules

- A patient or caregiver should be able to understand what is being shared in
  ordinary language.
- A patient or caregiver should be able to revoke or limit sharing where the use
  case permits.
- A caregiver's authority should be explicit, scoped, and updateable.
- A patient should have access to meaningful audit history for patient-directed
  sharing.

## 12. Semantic and Vocabulary Considerations

### 12.1 Data Categories

Interviewees repeatedly distinguished technical scopes from human-understandable
data categories. FHIR resource names are convenient for implementers but may not
be the right patient-facing or business-facing vocabulary.

Candidate v1 category patterns:

- Demographics
- Coverage
- Claims / Explanation of Benefits
- Medications
- Lab results
- Imaging reports
- Clinical notes
- Problems / conditions
- Immunizations
- Allergies
- Procedures
- Encounters
- Treatment records
- Care plans
- Referral details
- Social determinants / demographics relevant to care planning
- Sensitive data categories, where technically enforceable

The same category may map to different FHIR resources in different contexts. A
future artifact should define a mapping model rather than assuming a one-to-one
resource relationship.

### 12.2 Purpose and Context

Purpose of use remains necessary but too broad on its own. A ticket should carry
both purpose and context.

Examples:

- Patient-directed app access
- Caregiver representative access
- Claim attachment for a specific claim
- Prior authorization evidence for a specific service
- Risk adjustment retrieval
- Public health investigation for a specific condition or case
- Cancer registry follow-up for a specific case
- Referral coordination
- CDS Hooks invocation
- Notification subscription

### 12.3 Security Labels and Attributes

Security labels are a natural mechanism for representing sensitive categories,
but interviews suggest additional "security attributes" may be needed. These are
filterable data associations that support contextual access control.

Examples:

- Organization
- Department
- Encounter
- Claim
- Case
- Referral
- Care team
- Registry case
- Geographic jurisdiction
- Group membership

The model should allow servers to evaluate such attributes dynamically when
issuing or enforcing access.

### 12.4 Consent and Rule Expression

Permission Tickets should not create an isolated authorization language. The same
core rule expression should be usable or mappable across:

- Consent
- Patient preference
- Authorization
- Release of information
- Permission Ticket
- Jurisdictional rule
- Organizational policy

This does not mean every artifact has identical legal meaning. It means data
holders need a consistent computational format so these rules can be evaluated
together.

## 13. Adoption and Governance

### 13.1 Adoption Drivers

The interviews identified several likely adoption drivers:

- Patient demand for easier access
- Caregiver burden reduction
- CMS mandates and CMS-0057 implementation
- TEFCA, Da Vinci, and other networks needing finer transaction-level context
- Public health follow-up needs
- Registry modernization
- Payer-provider burden reduction
- EHR vendor differentiation
- AI and agent use cases requiring structured patient-authorized data access

### 13.2 Adoption Barriers

The main barriers are not only technical:

- Data holders may not trust external issuers.
- Customers may not ask EHR vendors for granular policy features.
- EHRs may lack reliable clinical segmentation.
- FHIR endpoints may not be enabled.
- Registry and public health systems may not be FHIR-ready.
- Staff may not understand new authorization artifacts.
- Consent form variation makes computable consent expensive.
- Vendors may turn the mechanism into an added-cost product.
- Network frameworks may not support sufficient scoping.
- App categories may change in ways data holders do not expect.
- Receiving systems may not integrate returned data usefully.

### 13.3 Governance Needs

A workable governance model must answer:

- Who may issue tickets?
- For which use cases may each issuer issue tickets?
- How are issuers accredited, audited, suspended, or removed?
- How are issuer keys discovered?
- How are organizations identified?
- How is category membership managed?
- How are patients informed?
- How are revocations distributed?
- How are disputes handled?
- How are misissued tickets detected and remediated?
- How does governance differ for patient-directed, public health, payer,
  caregiver, and network use cases?

### 13.4 Trust Patterns

Potential trust patterns include:

- Data-holder-issued ticket: The data holder issues a ticket for follow-up, as
  in public health case reporting where the data holder initiated the report.
- Patient-preference issuer: A patient uses an independent service to manage app
  preferences and issue tickets.
- Provider-bootstrap issuer: A provider the patient already uses verifies the
  patient and issues a ticket usable elsewhere.
- Network issuer: A HIN, HIE, QHIN, or other network issues tickets based on
  onboarding and use-case agreements.
- Public health issuer: A public health agency issues condition-specific tickets
  under legal authority.
- Consent service issuer: A consent management service issues or informs tickets
  based on computable consent.
- App issuer under governance: A patient app developer becomes an issuer only
  after meeting the same governance and audit requirements as other issuers.

No single trust pattern fits all use cases.

## 14. Risks and Mitigations

### 14.1 Over-Provisioning

Risk: Tickets authorize more data than needed or local systems fail to constrain
access.

Mitigations:

- Treat ticket scope as a ceiling.
- Require data-holder down-scoping.
- Start with enforceable scope levels.
- Require audit and conformance tests for access enforcement.

### 14.2 Under-Provisioning

Risk: Tickets are too narrow, preventing legitimate clinical, public health, or
registry work.

Mitigations:

- Allow renewal or follow-up tickets.
- Define condition-specific and workflow-specific profiles.
- Support partial denial with clear reasons.
- Permit appropriate broader access where legally justified.

### 14.3 Demographic Mis-Matching

Risk: A ticket is matched to the wrong patient.

Mitigations:

- Prefer solid identifiers.
- Separate matching from authorization.
- Deny ambiguous matches.
- Audit matching basis.

### 14.4 Issuer Misuse or Misconfiguration

Risk: A trusted issuer issues invalid, excessive, or unauthorized tickets.

Mitigations:

- Govern issuer eligibility by use case.
- Support issuer suspension.
- Require conformance testing.
- Provide audit trails and dispute processes.

### 14.5 Category Surprise

Risk: A practice approves a category and later an unexpected app or payer is
added.

Mitigations:

- Notify data holders when category membership changes.
- Lock category membership at approval time or require reapproval for material
  changes.
- Provide clear membership lists.

### 14.6 Sensitive Data Leakage

Risk: Behavioral health, 42 CFR Part 2, sexual health, self-pay, or other
sensitive data is shared inappropriately.

Mitigations:

- Keep local sensitivity filters authoritative.
- Align with DS4P and security labels.
- Avoid claiming clinical segmentation support beyond system capability.
- Require clear partial response behavior.

### 14.7 Static Consent and Form Variation

Risk: Computable ticket constraints cannot be derived consistently from varied
forms.

Mitigations:

- Standardize the output rule format.
- Encourage form harmonization where needed.
- Start with common, enforceable fields.

### 14.8 Data Use After Release

Risk: Once data leaves the data holder, the ticket no longer controls downstream
use.

Mitigations:

- Include purpose, recipient, and constraints in audit and provenance.
- Preserve DUAs, BAAs, policy obligations, and legal agreements where required.
- Explore future object-capability, data-use, or agentic policy models, but do
  not block v1 on solving post-release control fully.

### 14.9 Implementation Burden

Risk: The mechanism is easy to validate but hard to enforce across all APIs.

Mitigations:

- Be explicit about required enforcement surfaces.
- Provide reference implementation and tests.
- Support phased implementation.
- Avoid requiring unsupported fine-grained segmentation in v1.

## 15. Quality Criteria for the DAM and Future Specification

Future artifacts derived from this DAM should be:

- Complete enough to support formal use case and model generation
- Correct with respect to real stakeholder workflows
- Unambiguous about what a ticket does and does not authorize
- Deterministic enough for implementers to test
- Verifiable through conformance tests and audit traces
- Traceable from interview need -> requirement -> model element -> IG behavior
- Internally consistent across patient, caregiver, B2B, public health, registry,
  and intermediary flows
- Technology-neutral at the conceptual level but realistic about SMART/FHIR
  infrastructure
- Explicit about legal and policy boundaries
- Practical for current EHR and network capabilities

## 16. Candidate Formal Artifacts for Next Phase

The following artifacts should be generated from this narrative DAM:

1. Stakeholder and actor model
2. Use case catalog
3. Storyboards for each major use case
4. Activity diagrams for ticket issuance, presentation, validation, access, and
   revocation
5. Sequence diagrams for direct, intermediary, patient app, public health, payer,
   caregiver, and CDS Hooks flows
6. Conceptual class diagram
7. Glossary of domain terms
8. Requirements matrix with priority and traceability
9. Trust model and issuer governance model
10. Subject resolution model
11. Scope and data category model
12. Consent and local policy interaction model
13. Audit and transparency model
14. Revocation and renewal lifecycle model
15. Discovery metadata requirements
16. Threat model and privacy risk analysis
17. Conformance scenarios and negative tests
18. Implementation guide outline

## 17. Initial Glossary

Authorization Ceiling: The maximum access represented by a ticket before local
policy is applied. The data holder may grant less.

Business Rule: A policy, workflow condition, legal requirement, or operational
constraint that affects whether data may be shared.

Client Binding: Cryptographic or registration-based binding that prevents a
ticket from being used by anyone other than the intended client.

Computable Consent: Consent represented in a structured form that can be
processed by software, not only read by humans.

Data Category: A patient-meaningful or business-meaningful grouping of data,
which may map to one or more FHIR resources or elements.

Data Holder: The organization or system responsible for deciding whether to
release data.

Down-Scoped Access Token: An access token issued after ticket validation whose
permissions are equal to or narrower than the ticket.

Issuer: The trusted party that creates and signs a Permission Ticket.

Local Policy Overlay: The data holder's additional policy layer applied after
ticket validation.

Permission Ticket: A signed, portable, scoped authorization artifact carrying the
context needed to evaluate a data access request.

Requester: The app, system, organization, user, or agent seeking access.

Security Attribute: A filterable attribute used for access control, such as
department, organization, claim, case, referral, or group.

Subject Resolution: Matching the person referenced in the ticket to the local
record at the data holder.

Trust Framework: Governance and technical rules defining who is trusted to issue,
present, validate, and rely on tickets.

## 18. Open Questions

1. What exact issuer types should v1 support?
2. Should the ticket have use-case-specific profiles, a generic extensible
   structure, or both?
3. What is the minimum required subject identifier assurance?
4. Should demographic matching be prohibited, profiled separately, or allowed
   with strict constraints?
5. What machine-readable discovery mechanism tells apps which issuers are
   accepted by which data holders?
6. How should patient-meaningful data categories be standardized and mapped to
   FHIR?
7. How should tickets coordinate with HL7 Consent and Scalable Consent
   Management?
8. How does the model distinguish patient-directed access from payer, public
   health, and other B2B access?
9. How should category-based app approval avoid surprise membership changes?
10. What is the minimum audit event set?
11. What should patients and caregivers be able to see in access logs?
12. What revocation semantics are required for long-lived grants?
13. How should intermediaries issue, carry, or transform tickets?
14. What role should DNS-anchored identity and domain-bound JWKS endpoints play?
15. Which parts of UDAP Certifications and Endorsements should be reused or
   aligned?
16. What data segmentation level can EHRs reliably enforce in v1?
17. How should partial responses and redactions be communicated?
18. What open-source reference components are needed for adoption?
19. How should write-back be scoped or deferred?
20. How should autonomous agents be represented as requesters?

## 19. Recommended v1 Posture

The interviews support a pragmatic v1:

- Focus on portable authorization context, not full consent-law automation.
- Support patient-directed access, payer workflow access, public health
  follow-up, caregiver access, and intermediary flows as first-class use cases.
- Use a common core ticket model with profile-specific extensions where needed.
- Make issuer trust, requester identity, organization identity, subject
  resolution, client binding, scope, duration, revocation, and audit explicit.
- Treat the ticket as an authorization ceiling.
- Preserve data-holder local policy and sensitivity filtering.
- Start with broad data categories, resource groups, time periods, case/claim/
  referral context, and other enforceable constraints.
- Do not require fine-grained clinical segmentation that EHRs cannot reliably
  perform.
- Align with SMART, FAST Security, UDAP, HL7 Consent, and DS4P.
- Include the intermediary pattern in the core design.
- Provide machine-readable discovery and conformance tests.
- Provide reference implementation guidance that avoids unnecessary vendor
  lock-in.

The central design balance is clear: Permission Tickets must reduce repeated
authorization ceremonies and manual setup without creating a mechanism for
uncontrolled external authorization. The successful model is not "trust the
ticket and release the data." It is "trust the issuer for this context, validate
the ticket, resolve the subject safely, intersect with local policy, issue a
down-scoped token, enforce consistently, and audit transparently."
