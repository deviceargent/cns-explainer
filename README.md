# Contextual Navigation Signal (CNS) — Explainer


**Status:** Early Proposal  
**Date:** 2026  
**Author:** Miguel Okstein 
---

## Abstract

Users browsing the web frequently experience interruptions from 
contextually adjacent but temporally irrelevant content — 
advertisements, extension notifications, and feed items that 
relate to their general interest area but not to their current 
task or moment of intent.

This proposal defines a lightweight, privacy-preserving signaling 
layer that allows browsers to construct a local model of a user's 
active navigation context, derived from publisher-declared semantic 
metadata and observable in-page behavioral signals. This context 
signal is used by the browser to gate and prioritize ambient 
content delivery — including advertisements, notifications, and 
feed items — through a standardized surface, without exposing 
user behavior or intent to any external party.

The primary objective is to reduce disruption to the user's 
cognitive workflow. Improved relevance for publishers and 
advertisers is a consequent benefit, not the driving goal.

---

## Problem Statement

When a user is engaged in a focused browsing task — research, 
study, professional work — the ambient content delivered to their 
screen is frequently derived from their general interest profile 
rather than their current moment of intent.

This ambient content takes multiple forms, all affected by the 
same misalignment:

- **Advertisements** served by publisher ad networks
- **Web Push Notifications** from subscribed sites 
  (e.g. a video platform notifying a new upload while 
  the user is mid-research)
- **Browser extension notifications** operating within 
  the content layer (e.g. a game deals extension announcing 
  a free title during an unrelated work session)
- **On-site feed items** and recommended content widgets
- **Sidebar and ambient UI elements** populated by 
  third-party services

The current Web Push API (W3C Working Draft, December 2025) 
standardizes notification delivery efficiently, but carries 
no mechanism for the receiving browser to evaluate whether 
a notification is appropriate to surface at this moment, 
given the user's active task context. Throttling exists at 
the browser vendor level but is not part of any open standard 
and operates on volume, not semantic relevance.

This produces a specific failure mode: content that is 
topically related but temporally misaligned. A user 
researching appliance purchase trends may be shown a 
seasonal promotion for that same category — technically 
relevant, but disruptive and unconvertible at that moment.

The cost of this misalignment is triple:

- **For the user:** involuntary interruption of cognitive 
  flow, forcing a context-switch and recovery cost that 
  compounds across a browsing session.
- **For the advertiser:** a wasted impression delivered 
  to a user whose current intent does not match the offer.
- **For the web:** unnecessary bandwidth, compute, and 
  energy consumed to deliver content that will be ignored.

No existing open standard captures or communicates the 
user's active task context at session granularity — the 
level at which temporal relevance can actually be assessed.

---

## Use Cases

The following scenarios illustrate the problem at the user level. 
They are drawn from real browsing patterns and represent distinct 
user profiles and task types.

---

### Use Case 1 — Market Research (Professional)

A user is preparing a presentation and is actively researching 
which home appliance category has seen the highest consumer 
adoption in their country during the current month. They have 
multiple tabs open: a statistics portal, a retail trends report, 
and a search results page.

During this session, an advertisement is served promoting a 
seasonal sale on home appliances — topically identical to the 
research subject, but temporally misaligned. The user is in 
an information-gathering phase, not a purchase-intent phase. 
The ad is ignored. The impression is wasted. The user's 
reading flow is interrupted.

**What CNS would change:** The browser's local context model 
identifies an active informational research session on a 
commercial topic. It defers purchase-intent content until 
signals indicate a transition toward comparative or 
transactional intent.

---

### Use Case 2 — Creative Work Session (Extension Notification)

A user is engaged in an active music production or music 
research workflow. A browser extension subscribed to a 
gaming deals platform fires a push notification announcing 
a free game title available for a limited time.

The extension is functioning as designed. The notification 
is potentially of interest to this user in other contexts. 
But the interruption arrives during a focused creative 
session on an unrelated domain, breaking concentration 
at no benefit to the user, the platform, or the advertiser.

Under the current Web Push standard, the browser has no 
mechanism to evaluate whether this notification is 
appropriate to surface now. Throttling, where implemented, 
operates on delivery volume — not on the semantic distance 
between the notification content and the user's active context.

**What CNS would change:** Extensions operating within the 
content layer and complying with CNS would submit their 
notification to the browser's context gate. The browser 
evaluates semantic distance between the notification topic 
and the active session context. Non-urgent, low-relevance 
notifications are held and batched for delivery at a 
contextually neutral moment — a tab switch, an idle period, 
or a session boundary.

---

### Use Case 3 — Social Platform Browsing (Ad Density)

A user is browsing a major social platform. The platform 
operates its own ad delivery system, which is contextually 
aware at the profile level but not at the session level. 
The result is a high-density ad surface where multiple 
competing offers appear in a single scroll session, many 
of which are redundant or mutually irrelevant to the 
user's current browsing intent.

This is not a failure of the platform's targeting — it is 
a structural consequence of optimizing for impression volume 
rather than moment-level relevance. The user develops 
banner blindness. Conversion rates decrease. The advertiser 
pays for ignored inventory.

**What CNS would change:** The platform, as a CNS-compliant 
publisher, exposes a standardized content surface. The 
browser's context model limits the ad inventory surfaced 
at any moment to those semantically aligned with the 
active session. The result is a smaller, higher-quality 
selection — analogous to a curated shelf rather than 
an unordered stack. Initial impression volume decreases. 
Conversion probability per impression increases.

This may appear to conflict with volume-based business 
models. In practice, it realigns incentives: a user who 
engages with a contextually appropriate ad is more 
valuable than ten users who scroll past irrelevant ones.

---

## Proposed Solution — Contextual Navigation Signal (CNS)

CNS is a three-layer architecture that constructs a local model 
of the user's active browsing context and uses it to gate 
ambient content delivery. No user data leaves the device. 
The browser emits decisions, not profiles.

---

### Layer 1 — Declarative Semantic Metadata (Publisher)

Publishers annotate their content using structured metadata 
that describes the topical and intentional nature of the page 
or content section. This annotation extends existing open 
standards rather than replacing them.

**Proposed format — JSON-LD extension via Schema.org vocabulary:**

```json
{
  "@context": "https://schema.org",
  "@type": "WebPage",
  "cns:topic": "home-appliances",
  "cns:intentType": "informational",
  "cns:taskPhase": "research",
  "cns:temporalContext": "current-month",
  "cns:taxonomy": "https://cns-taxonomy.org/v1"
}
```

**intentType** admits the following base values, derived from 
established information retrieval taxonomy:

- `informational` — the user is seeking to understand something
- `comparative` — the user is evaluating options
- `transactional` — the user is prepared to act or purchase
- `navigational` — the user is moving between resources
- `creative` — the user is producing, not consuming

**taskPhase** describes position within a session arc:
- `research` / `evaluation` / `decision` / `execution`

Publishers are not required to annotate all fields. 
Unannotated pages fall back to Layer 3 classification.

---

### Layer 2 — Browser Context Engine (Local Inference)

The browser constructs a real-time session context model 
using observable behavioral signals. This process is 
entirely local — no signal is transmitted externally.

**Input signals:**

| Signal | What it indicates |
|---|---|
| Scroll depth and velocity | Engagement level and reading intent |
| Lazy load progression | Content consumption pace |
| In-page search queries | Specific sub-topic focus |
| Tab switching patterns | Multi-source research behavior |
| Time-on-section | Depth of interest per content block |
| Internal site navigation | Task continuity vs. exploration |

**Output — Active Context Object (ACO):**

The browser maintains a local, ephemeral Active Context 
Object that aggregates Layer 1 metadata and Layer 2 
behavioral inference into a single session descriptor:

```json
{
  "sessionTopic": "home-appliances",
  "inferredIntent": "informational",
  "inferredPhase": "research",
  "confidence": 0.87,
  "contextAge": 142,
  "boundarySignals": ["tab-switch-pending", "idle-detected"]
}
```

The ACO is held in browser memory only. It is never 
serialized, transmitted, or exposed to any web API 
accessible by page scripts.

**The Context Gate:**

When an ambient content delivery event occurs — an ad 
request, a push notification, an extension notification, 
a feed item — the browser evaluates it against the ACO 
before surfacing it to the user.

The gate computes a semantic distance score between the 
incoming content's declared topic and the ACO. Content 
above a relevance threshold is surfaced immediately. 
Content below threshold is either:

- **Held** — queued for delivery at the next contextually 
  neutral moment (idle period, tab switch, session boundary)
- **Dropped** — if the content carries an expiry signal 
  and the hold period exceeds it

The gate does not modify content. It only controls 
the moment of delivery.

---

### Layer 3 — Autonomous Classification (Scalable Annotation)

The majority of web content will not carry Layer 1 
annotations, particularly in early adoption phases. 
Layer 3 addresses this through autonomous classification 
agents trained on Layer 1-annotated content.

These agents operate at the browser or platform level 
and assign inferred CNS metadata to unannotated pages, 
using the CNS taxonomy as their output vocabulary. 
Over time, as Layer 1 adoption grows, Layer 3 agents 
become more accurate and their classifications more 
granular.

Layer 1 annotations authored by publishers and developers 
serve as ground truth training signal. This creates a 
virtuous cycle: manual annotation improves autonomous 
classification, which reduces the burden of manual 
annotation.

---

### Delivery Surface

CNS-gated content is delivered through one of two 
standardized surfaces:

**Surface A — Browser Sidebar (Vendor-implemented)**  
A persistent, toggleable sidebar panel within the browser 
chrome, analogous to existing sidebar implementations in 
Chromium-based browsers. This surface is controlled by 
the browser, not the page. Publishers and advertisers 
declare content available for this surface; the browser 
decides what to render and when, based on the ACO.

**Surface B — Declarative Page Canvas (Publisher-implemented)**  
Publishers may reserve a standardized content area within 
their page layout — a declared CNS canvas — which the 
browser populates with contextually gated content. The 
publisher defines the existence and dimensions of the 
surface. The browser controls its content.

In both cases, the publisher does not control what is 
shown — only that a surface exists. This separation is 
intentional and is the architectural guarantee of 
CNS neutrality.

---

### Extension Compliance Model

Browser extensions that deliver ambient content — 
notifications, badges, feed updates — and that operate 
within the web content layer are subject to CNS gating 
if they declare themselves as content-layer operators.

Under Manifest V3, extensions already operate under 
significantly restricted background execution models. 
CNS extends this constraint with a semantic layer: 
a compliant extension submits its notification payload 
to the browser's context gate before delivery, 
and respects the gate's hold or drop decision.

Non-compliant extensions are not blocked by CNS. 
Compliance is opt-in at the extension level and 
enforced at the surface level — a CNS-aware browser 
may choose to visually distinguish gated from 
non-gated notifications, giving users the ability 
to make informed decisions about their notification 
permissions.

---

### Contextual Urgency — Time-Sensitive Content

CNS gating operates bidirectionally. While its primary 
function is to defer or drop ambient content that is 
contextually misaligned, the same architecture supports 
the prioritized delivery of content that is both 
semantically aligned and temporally urgent.

**Scenario:** A user is researching flight options or 
travel logistics. A breaking news event — an airport 
disruption, a security incident affecting travel — 
is published by a compliant news publisher with a 
high-urgency contextual signal.

The browser's context gate recognizes:
- High semantic alignment with active session topic
- Publisher-declared urgency and short expiry window
- Content classification: `intentType: informational`, 
  `temporalContext: breaking`

The content is surfaced immediately through the CNS 
delivery surface, without waiting for a neutral boundary 
moment.

**The editorial quality implication:**

Breaking news events generate high content volume around 
a single trending topic. Current delivery systems surface 
this volume indiscriminately — rewarding speed of 
publication over relevance or quality, and creating 
conditions for low-quality or misleading content to 
compete on equal footing with verified reporting.

CNS introduces a structural quality filter: content 
that is semantically aligned with the user's active 
context and carries a verified publisher annotation 
is prioritized. Content that is topically adjacent 
but contextually opportunistic — produced to exploit 
a trending topic without genuine relevance to the 
user's session — scores low on semantic distance 
and is held or dropped.

This does not constitute editorial curation by the 
browser. The browser applies no judgment about content 
quality or truthfulness. It applies only the contextual 
relevance model — but that model, applied consistently, 
produces an emergent editorial effect: content that 
genuinely matches the user's active context rises; 
content that exploits trending keywords without 
contextual grounding does not.

For publishers, this creates a structural incentive 
toward precision over volume. A news publisher whose 
content is consistently well-annotated and contextually 
accurate will achieve higher surface rates than one 
that publishes high volumes of loosely related content 
around trending topics. Audience capture depends on 
contextual adequacy, not publication frequency.

---

## Privacy & Security Considerations

CNS is designed with privacy as an architectural constraint, 
not a policy afterthought. The following principles govern 
the entire system.

---

### 1. No Data Leaves the Device

The Active Context Object (ACO) is ephemeral, held in 
browser memory only, and is never:

- Transmitted to any external server
- Exposed to page scripts via any Web API
- Logged, persisted, or associated with a user identity
- Shared between browsing sessions

The browser emits decisions — surface this content now, 
hold it, drop it — not the context model that produced 
those decisions. Advertisers and publishers receive 
delivery outcomes, not user signals.

This is a Privacy by Architecture guarantee, not a 
policy-level promise. The architecture makes exfiltration 
of the ACO structurally impossible, not merely prohibited.

---

### 2. Publisher Annotation is One-Way

Layer 1 metadata flows from publisher to browser. 
The browser uses it to build context. No information 
flows back to the publisher about how their annotation 
was used, what the ACO contained, or what other content 
was evaluated alongside theirs.

Publishers learn only what they would learn from any 
standard delivery system: whether their content was 
surfaced and whether the user interacted with it.

---

### 3. The Annotation Honesty Problem

A publisher may declare misleading Layer 1 metadata — 
annotating clickbait content as `intentType: informational` 
or `taskPhase: research` to attract higher-value 
contextual matches.

CNS addresses this through two mechanisms:

**Behavioral cross-validation:** The browser's Layer 2 
inference operates independently of Layer 1 declarations. 
If a publisher declares `informational` but user behavior 
on that page consistently signals low engagement, rapid 
exit, and no depth indicators, the ACO weights Layer 2 
signals over Layer 1 declarations. Persistent 
mismatches between declared and inferred context 
reduce the publisher's annotation authority score 
over time, locally within the browser.

**Taxonomy governance:** Layer 1 annotations reference 
a versioned, openly governed CNS taxonomy. Publishers 
using the taxonomy agree to its usage terms. Systematic 
abuse can be flagged through the governance process, 
which operates independently of any browser vendor.

This does not eliminate annotation abuse entirely. 
It structurally reduces its effectiveness and creates 
a reputational signal that persists locally.

---

### 4. Extension Permissions Model

Extensions that opt into CNS compliance submit their 
notification payloads to the context gate. This 
requires no new permission beyond what the extension 
already holds.

Extensions do not gain any new visibility into the ACO 
or the user's session context. The gate is a one-way 
evaluation: the extension submits, the browser decides, 
the extension receives a delivery outcome only.

Non-compliant extensions retain their current behavior 
and permission model. CNS does not break existing 
extension functionality.

---

### 5. Vendor Neutrality and Monopoly Risk

CNS is an open standard. Any browser vendor may 
implement it. The specification defines behavior — 
not implementation details — leaving vendors free 
to optimize their context engines competitively.

The risk that a dominant browser vendor uses CNS 
to favor its own advertising inventory is real and 
acknowledged. CNS addresses this through:

**Surface separation:** The CNS delivery surface is 
specified as a neutral canvas. The standard defines 
what the surface is, not what fills it. No vendor 
may write into the specification a preference for 
its own content network.

**Regulatory scope:** Implementation-level competitive 
behavior — a vendor prioritizing its own ad inventory 
within a CNS surface — falls within the existing 
jurisdiction of competition regulators (EU DMA, 
US antitrust, UK CMA). CNS does not attempt to 
replicate regulatory enforcement within the standard. 
It relies on the separation of specification and 
implementation, a principle established across 
the W3C's existing standards portfolio.

**Multi-vendor governance:** The CNS taxonomy and 
core specification are proposed for governance under 
a multi-stakeholder body, analogous to the model 
used by Schema.org — founded by competing vendors 
under a shared open license.

---

### 6. User Control

Users retain full control over CNS behavior through 
browser settings:

- **Disable CNS gating entirely** — all ambient content 
  reverts to current delivery behavior
- **Adjust hold thresholds** — how aggressively the 
  gate defers low-relevance content
- **View and clear session context** — the ACO is 
  inspectable and clearable on demand, analogous 
  to clearing browsing history
- **Opt out of Layer 3 classification** — pages 
  visited are not submitted to autonomous 
  classification agents

No CNS feature requires user action to protect privacy. 
Privacy is the default. User controls exist to expand 
CNS functionality, not to restrict data collection 
that would otherwise occur.

---

## Alternatives Considered

The following existing technologies and proposals address 
adjacent problems. None resolves the core issue of 
session-level temporal relevance for ambient content delivery.

---

### 1. Google Topics API (Privacy Sandbox, discontinued 2025)

Topics API assigned interest categories to users based on 
browsing history and exposed those categories to advertisers 
via the browser. It was the most direct prior attempt to 
replace third-party cookies with a privacy-preserving 
contextual signal.

**Why it does not address the CNS problem:**

Topics operated at the interest-profile level — a 
persistent, cross-session representation of the user's 
general interests. It had no mechanism for session-level 
or task-level context. A user interested in home appliances 
would receive appliance-related ads regardless of whether 
they were actively researching, casually browsing, or 
doing something entirely unrelated.

Topics API was also architecturally centralized: the 
taxonomy was defined and controlled by Google, the 
implementation was exclusive to Chrome, and the 
initiative was discontinued in October 2025 following 
low adoption and sustained regulatory pressure.

CNS addresses the session-level gap that Topics never 
attempted to fill, under an open governance model that 
Topics explicitly avoided.

---

### 2. Schema.org Structured Data

Schema.org provides a widely adopted vocabulary for 
describing the static content of web pages. It is 
the closest existing standard to CNS Layer 1.

**Why it does not address the CNS problem:**

Schema.org describes what a page *is* — its topic, 
its author, its publication date, its content type. 
It does not describe what the user is *doing* on 
that page, at what phase of a task, or with what 
temporal intent.

The distinction is between content identity and 
session context. Schema.org solves the former 
comprehensively. CNS extends toward the latter, 
building on Schema.org's vocabulary rather than 
competing with it.

---

### 3. Web Push API (W3C Working Draft, December 2025)

The Web Push API, now incorporating Declarative Web Push, 
standardizes the delivery of notifications from web 
applications to users. It is efficient, privacy-aware, 
and actively maintained by Apple, Mozilla, and the W3C.

**Why it does not address the CNS problem:**

Web Push defines how notifications are delivered. 
It does not define when they should be delivered 
relative to the user's active context. A notification 
that arrives via a fully compliant Web Push 
implementation may still be temporally misaligned 
with the user's current task.

CNS does not replace or modify Web Push. It sits 
above it as a context gate — evaluating delivery 
timing without altering the delivery mechanism. 
The two standards are complementary and are designed 
to coexist without conflict.

---

### 4. Browser-level Notification Throttling

Major browser vendors — particularly those in the 
Chromium ecosystem — implement internal notification 
throttling mechanisms that limit delivery volume 
from abusive or overly frequent sources.

**Why it does not address the CNS problem:**

Vendor throttling operates on volume, not semantic 
relevance. It limits how many notifications a site 
can send per unit of time. It does not evaluate 
whether any individual notification is appropriate 
given the user's current session context.

Additionally, these mechanisms are not open standards. 
They are proprietary implementations, undocumented 
at the specification level, inconsistent across 
vendors, and not auditable by publishers, advertisers, 
or regulators.

CNS proposes to standardize the intent behind 
throttling — contextual appropriateness — and 
make it transparent, consistent, and governable 
across all compliant browsers.

---

### 5. Contextual Advertising Networks

Existing contextual ad networks — which serve ads 
based on the topic of the current page rather than 
user profiles — represent the closest behavioral 
analogy to CNS in the advertising industry.

**Why they do not address the CNS problem:**

Contextual networks operate at the page level, 
not the session level. They know what the page 
is about but not what the user is doing on it, 
how long they have been doing it, or what phase 
of a task they are in.

They are also proprietary, fragmented, and 
incompatible with each other — each network 
defines its own taxonomy, its own signals, 
and its own delivery logic. There is no open 
standard that a publisher, advertiser, or 
browser can rely on across networks.

CNS proposes exactly that open standard — 
not as a replacement for contextual networks, 
but as the shared semantic layer they currently lack.

---

## Open Questions

The following questions are acknowledged as unresolved 
at this stage of the proposal. They are presented 
as open problems for community discussion, not as 
gaps that invalidate the proposal.

---

### 1. Taxonomy Governance Structure

CNS requires a shared, versioned taxonomy of topics, 
intent types, and task phases. Who governs this taxonomy, 
how it evolves, and how conflicts between stakeholders 
are resolved are open questions.

The Schema.org model — founded by competing vendors 
under a shared open license, governed by community 
consensus — is a candidate reference. However, 
Schema.org's governance has faced criticism for 
being slow to evolve and dominated by its founding 
members. CNS taxonomy governance should learn from 
that experience.

**Open:** Should CNS taxonomy governance be hosted 
under W3C, under an independent multi-stakeholder 
body, or under a federated model where regional 
consortia maintain domain-specific extensions?

---

### 2. Cold Start and Sparse Annotation

In early adoption phases, the majority of web content 
will carry no Layer 1 annotations. However, the adoption 
path for CNS annotation is not expected to depend 
primarily on manual publisher effort.

The most likely adoption vector is tooling integration: 
advertising creation platforms that already classify 
content internally can export that classification as 
CNS-compliant metadata. IDE environments with autonomous 
coding agents — already standard in professional web 
development workflows — can suggest or automatically 
insert CNS annotations at authoring time, analogous 
to how accessibility and SEO hints are surfaced today. 
CMS platforms can generate CNS metadata from their 
existing internal categorization systems.

This mirrors the adoption path of Schema.org, where 
coverage scaled not through individual publisher 
effort but through platform-level integration — 
WordPress, Shopify, and similar tools generating 
compliant markup automatically.

Layer 3 autonomous classification remains the fallback 
for content that predates CNS or originates from 
platforms that have not yet integrated annotation. 
Its accuracy improves progressively as Layer 1 
ground truth accumulates through tooling adoption.

**Open:** What is the recommended minimum CNS 
annotation schema for platform integrators — 
the smallest valid annotation that provides 
useful context signal without requiring 
deep semantic analysis of the content?

---

### 3. Context Boundary Detection

The browser's context engine must determine when 
a user's active session context has changed — 
a transition from research to transactional intent, 
or from one topic domain to another. The behavioral 
signals proposed (scroll depth, tab switching, 
lazy load progression) are proxies, not direct 
measurements of intent.

**Open:** What is the minimum viable set of 
behavioral signals for reliable context boundary 
detection? How should the engine handle ambiguous 
transitions — for example, a user who opens a 
purchase page mid-research without clear intent 
to transact?

---

### 4. Cross-Origin Session Context

A typical research session spans multiple origins — 
a search engine, several publisher sites, a reference 
portal, a retail site. Each origin is isolated under 
current browser security models. The ACO must 
aggregate signals across origins without violating 
the same-origin policy or creating a cross-site 
tracking vector.

**Open:** Can the ACO be constructed from 
per-origin context fragments without exposing 
any single origin to information about the 
others? What is the correct trust boundary 
for cross-origin context aggregation?

---

### 5. Adversarial Autonomous Classification

Layer 3 classification agents learn from Layer 1 
annotations. A coordinated effort by publishers 
to annotate content systematically and dishonestly 
could corrupt the training signal and degrade 
classification quality across all users of 
a given browser.

**Open:** What mechanisms — rate limiting, 
annotation provenance, cross-validation against 
behavioral signals — are sufficient to make 
adversarial annotation training economically 
unattractive? Is a centralized classification 
model inherently more vulnerable than a 
federated one?

---

### 6. Accessibility and Cognitive Load

CNS introduces a new content surface — the 
browser sidebar or declarative page canvas — 
that coexists with existing page content. 
For users with cognitive disabilities, 
attention disorders, or low digital literacy, 
an additional ambient content layer may 
increase cognitive load rather than reduce it.

**Open:** Should CNS surfaces be disabled 
by default for users who have declared 
accessibility preferences? How should 
CNS interact with existing accessibility 
standards such as WCAG and ARIA?

---

### 7. Applicability to Non-Human Agents

As autonomous agents — AI assistants, research 
bots, workflow automation tools — increasingly 
navigate the web on behalf of users, the concept 
of "active session context" becomes more complex. 
An agent performing research does not produce 
the same behavioral signals as a human user. 
Layer 2 inference may produce unreliable ACOs 
for non-human sessions.

**Open:** Should CNS define a separate 
agent-declared context model, where autonomous 
agents explicitly declare their session context 
rather than having it inferred from behavioral 
signals? What are the privacy and abuse 
implications of allowing agents to self-declare 
context?

---

*This explainer is an early-stage proposal submitted 
for community discussion. All technical details are 
subject to revision based on feedback. The author 
welcomes input from browser vendors, publishers, 
advertisers, privacy researchers, and accessibility 
specialists.*
