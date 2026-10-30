# Schematic Version Control for Site Construction Drawings

Spec and architecture for a React Native + Firebase mobile app (iOS + Android)
that gives a construction site a single, permissioned source of truth for
drawings shared across trades — stopping the "I'm working off an outdated
PDF" problem that costs rework and causes clashes between trades.

Status: pre-build architecture/spec. No code in this repo yet — this doc is
the thing to agree on before screens get built.

## 1. Problem

On a build site (a new warehouse shell-and-fit-out, a high-street store
renovation, anything in between), many trades each hold their own drawings of
the same space, at different revisions:

- Groundworks / structural / steel erectors
- Concrete
- Electricians
- HVAC
- Fire systems
- Data cabling
- CCTV / camera install
- Lighting / light hangers
- The site manager, who is accountable for all of it

Drawings get superseded, trades work from stale copies, and there's no shared
place to flag a clash ("your duct run is where my cable tray goes") or see
who signed off on the latest change. The app's job is to make the *drawing*
itself version-controlled and permissioned the way code is, while staying
usable by non-developers on a phone, often outdoors with patchy signal.

## 1.1 Validated against a real site

Grounding this in a specific, lived example rather than a generic persona:
on a large 3PL warehouse build, the founder installed structured/data
cabling and CCTV. Confirmed directly (not assumed) about how that site
actually ran:

- **What went wrong, concretely**: wrong install location (mounting/routing
  done against a drawing that no longer matched what was physically there)
  and wasted site visits / rework redoing the work once the mismatch
  surfaced — not primarily clashes with other trades' installed equipment
  or failed inspections, though those are adjacent risks of the same root
  cause.
- **How updates actually moved**: email. Per-revision PDF attachments, no
  shared platform in consistent use by the trades. This is the baseline
  the app has to beat — not "better than Procore," but "better than a PDF
  in an inbox with no way to tell if it's the latest one."
- **Who owned "current"**: each trade owned and published its own
  drawings independently. There was no single cross-trade authority
  reconciling them — which validates the access-grant model in §3.4/§4 as
  designed (per-Drawing/per-Set grants, not one site-wide publisher role)
  and means the **Drawing Set / amalgamation feature carries the real
  value-add**: it's the only place a cross-trade view exists at all, since
  no human role was filling that function on this job.
- **Highest-leverage fix, by the installer's own ranking**: a push alert
  the moment a drawing relevant to their trade changes — ranked above
  trade-filtered views, in-drawing clash flagging, and offline access.
  This reprioritizes the roadmap: §3.3's publish notification is not a
  nice-to-have alongside the other v1 features, it's the single feature
  that would have prevented the specific failure described (wrong install
  location from a stale drawing) on its own, independent of whether
  amalgamation or review threads ever got used. Build and validate it
  first.

## 1.2 External evidence — trade-specific pain & competitive gap

Research findings below are web-search-sourced (vendor blogs, trade forums,
case studies), not peer-reviewed — treated as directional corroboration,
not proof. A second pass verifying hard rework-cost figures is still
running and will be appended as §1.3.

**This is a recognized, recurring pattern, not a one-off:**
- Fieldwire's document-control guide opens with a mechanical sub roughing
  in a floor from a drawing two weeks out of date, framing it as routine.
- LetsBuild/Aproplan case study: a warehouse-type project built racking, a
  mezzanine, and an office in the wrong place off outdated drawings;
  remediation estimated near £5M on an £8M contract.
- building-code-forum.com: an electrician working off the original bid set
  (not the current revision) installed ~$34,000 of the wrong fixtures on a
  warehouse job.
- Jonas Construction blog ("Drawing Updated Friday, Field Missed It
  Monday"): explicitly argues that emailing/uploading a revision does not
  guarantee the field crew sees it before acting on the old one.

**Competitive landscape — where Procore/Autodesk Build/Bluebeam
Studio/Fieldwire/Revizto leave a gap for specialty/low-voltage trades:**
- **Cost**: priced for GC-scale contract volume (Procore reviews cite
  ~$6-13k/yr for a $5-10M/yr sub; Revizto requires per-seat annual minimums)
  — hard to justify for a small cabling/CCTV crew on one job.
- **GC-centric licensing**: subs are typically invited as guest seats
  project-by-project on the GC's instance, not running their own
  cross-project system of record — which matches our founder's
  experience of no shared platform actually being used.
- **Workflow mismatch**: Autodesk Build assumes a Revit/AutoCAD-centric
  design workflow; Fieldwire is noted as the better fit for small/mid
  specialty trades, but is still deployed GC-first.
- No source found describes a **sub-owned, cross-project, trade-to-trade
  notification layer** — i.e. a tool the cabling company itself runs
  across all its jobs/GCs, rather than a hub it's a guest in. That's this
  app's lane.

**Amalgamation feature's gap check**: clash detection as practiced
(Navisworks, Revizto, BIM 360/ACC) is a 3D/BIM, pre-construction activity
run by VDC coordinators, not installers. Bluebeam Studio's 2D overlay/
compare is the closest desktop equivalent but is a paid-tier, manually-
fiddly power-user workflow — no source found describes a mobile-first,
installer-usable, cross-trade 2D overlay used *during* installation
(vs. pre-construction coordination). Supports amalgamation as genuinely
differentiated rather than a weaker clone of existing BIM tooling.

**Push-notification gap check**: only BuildPass ("Smart Notify" — marketed
as "coming soon," not confirmed live) and Fieldwire (push/email digest)
were found offering anything close to trade-targeted revision alerts;
PlanGrid flags stale drawings reactively rather than pushing proactively.
Corroborates push-on-publish (§1.1) as a real, largely unmet gap even
among named competitors, not just this app's own assumption.

## 1.3 External evidence — quantified cost of the problem

Verified via multiple independent press recaps (primary PDF host was
unreachable from the research sandbox; a direct fetch of
pg.plangrid.com/.../Construction_Disconnected.pdf is recommended before
using these externally, e.g. in a pitch deck):

- **FMI Corporation & PlanGrid, "Construction Disconnected" (2018)**,
  ~599 US construction leaders surveyed: **$177.5B/year** total US labor
  cost of rework + searching for project data + conflict resolution.
  Of that, **$31.3B/year** is rework specifically caused by poor project
  data + miscommunication (48% of US rework). Field workers reported
  ~5.5 hrs/week searching for project information and ~5 hrs/week on
  conflict resolution — roughly a third of a working week on exactly the
  class of problem this app targets.
- **Autodesk/FMI, "Harnessing the Data Advantage in Construction" (2021)**,
  3,900+ global professionals, 2020 data: "bad data" (inaccurate,
  incomplete, inaccessible, inconsistent, or untimely) caused **$88.69B**
  in global rework (14% of all rework that year), inside a **$1.85T**
  global headline cost of bad data overall.
- **McKinsey Global Institute, "Reinventing Construction" (2017)**: solid
  macro framing (construction near the bottom of all sectors on
  digitization; ~1%/year productivity growth for two decades; $1.6T global
  productivity gap) but does **not** itself contain a rework/document cost
  figure — any "McKinsey" rework number in circulation is actually the
  FMI/PlanGrid figure misattributed. Use McKinsey only for the
  digitization-lag framing, not a cost number.
- **Specialty/low-voltage trades specifically**: no study found isolates
  electrical, cabling, fire/life-safety, HVAC, or CCTV subcontractors'
  document-control costs — a real gap in public literature. The founder's
  firsthand account (§1.1) is currently the best evidence for this exact
  population; frame it as qualitative validation filling a quantification
  gap, not as an anecdote standing in for data that exists elsewhere. One
  adjacent, not-fully-verified data point: a circulated 2019 figure that
  >70% of subcontractors don't use mobile document-management tech, which
  is at least directionally consistent with "trades work off emailed
  PDFs, not a GC platform."
- **Baseline-cost framing**: CII's project-benchmarking work (frequently
  cross-cited in academic papers, primary report not directly accessed)
  puts direct rework around ~5% of total construction cost, with older CII
  industrial-project data as high as >12% and design-related deviations
  driving ~80% of that. Use as an order-of-magnitude anchor, not a precise
  figure.
- **Flagged as unverified / do not cite externally**: "~70% of rework from
  design-construction disconnect," "$280B annual US rework," and "~55% of
  rework from inaccurate documentation" (the last traces to a vendor press
  release with no named underlying study) all appear repeatedly in
  secondary blog posts without a traceable primary source.

**Bottom line for the pitch**: lead with the verified $31.3B/$177.5B
(FMI/PlanGrid 2018) and $88.69B/$1.85T (Autodesk/FMI 2021) figures as the
industry-scale problem, McKinsey for "construction is uniquely undigitized"
context, and the founder's own account (§1.1) plus the trade-forum/case-study
evidence (§1.2) as the specific, underserved-population proof point that no
existing quantified study covers.

## 2. Core concepts

| Concept | Definition |
|---|---|
| **Drawing** | A single versioned document (PDF, DWG export, image) tied to one trade/purpose. Has an ordered history of **Revisions**. |
| **Revision** | One staged state of a Drawing: `draft → in_review → published → superseded`. Immutable once published; a new change is always a new revision, never an edit in place. |
| **Drawing Set (the "linchpin")** | A named grouping of Drawings that all depict the same physical area/subject, used for cross-trade reference. The linchpin is the set's designated base/reference drawing. |
| **Amalgamation** | A read-only composite view generated by overlaying the current published revisions of every Drawing in a Set, aligned to a common reference (origin point + scale, set once per Set). Used for size reference / clash spotting across trades — it is a *view*, not a new editable document. |
| **Review Thread / Comment** | A comment pinned to an (x, y) point or region on a specific revision. Supports replies, `@mentions`, and a resolved/open status (snagging). |
| **Access Grant** | `(Users subset) × (Drawings subset)` with a role (`viewer`, `commenter`, `editor`, `approver`). This is the union described in the brief — a grant names which people can do what on which drawings, not a single site-wide role. |
| **Site** | Top-level tenant. Users, Drawing Sets, and Access Grants all live under a Site. |

## 3. Feature set (v1)

### 3.1 Amalgamation
- Create a Drawing Set, pick a linchpin drawing, add member Drawings.
- One-time per-drawing calibration: two reference points + known real-world
  distance (or a known scale ratio) — this is what lets drawings from
  different trades/sources overlay correctly. Stored per Drawing, reused
  across revisions unless the drawing's extents change.
- Amalgamated view renders each member drawing's **current published**
  revision as a toggleable layer (opacity + show/hide per trade), so a
  site manager can ask "does the cable tray clash with the duct?" without
  opening five PDFs.
- Amalgamation is computed client-side from already-fetched revisions —
  no new server-side document is created, so there's nothing extra to keep
  in sync.

### 3.2 Peer / self review
- Any user with `commenter`+ on a Drawing can pin a comment to a revision.
- Comments carry a status: `open → resolved`, plus optional `severity`
  (snag/defect vs. query vs. FYI) — same shape for a self-review before
  submitting a revision and a peer/client review after.
- Visibility follows the Access Grant on the Drawing: a user who can't see
  the drawing can't see its comments, full stop.
- Review is advisory by default: comments don't block publishing unless
  the Drawing Set is configured to `require_resolved_before_publish`.

### 3.3 Staged updates / version control
- Upload → `draft` (only visible to uploader + editors on that Drawing).
- `draft → in_review`: notifies users with `approver` role on that Drawing.
- `in_review → published`: requires an approver action; the prior published
  revision flips to `superseded` but stays readable (full history, never
  deleted).
- Every revision keeps: uploader, timestamp, change note (required, free
  text — "what changed and why"), and optionally a diff reference to the
  prior revision's calibration so amalgamated views don't silently drift.
- Push notification to every user with ≥`viewer` on that Drawing (or that
  Drawing's Set) when a revision is published — this is the mechanism that
  actually solves "the site manager had the old PDF."

### 3.4 Access control
- Enforced at two layers: Firestore Security Rules (server-side, can't be
  bypassed by a modified client) mirrored by UI-side checks (so the app
  doesn't show controls a user can't use).
- Grants are additive and most-specific-wins is *not* used — a user's
  effective permission on a Drawing is the highest role granted to them
  directly on that Drawing, or via any Drawing Set / Site-wide grant that
  includes it. Simpler to reason about than an override hierarchy, at the
  cost of not supporting "block this one user from this one drawing" —
  acceptable for v1; revisit if a real case shows up.

## 4. Data model (Firestore)

```
sites/{siteId}
  name, address, createdAt

sites/{siteId}/users/{userId}          -- profile + site-wide default role
  displayName, trade, email, siteRole: viewer|commenter|editor|approver|admin

sites/{siteId}/drawingSets/{setId}
  name, linchpinDrawingId, requireResolvedBeforePublish: bool

sites/{siteId}/drawingSets/{setId}/drawings/{drawingId}
  title, trade, setId, currentPublishedRevisionId, calibration: {p1, p2, realDistance, unit}

sites/{siteId}/drawingSets/{setId}/drawings/{drawingId}/revisions/{revisionId}
  storageRef, uploadedBy, uploadedAt, status: draft|in_review|published|superseded,
  changeNote, supersedesRevisionId, approvedBy?, approvedAt?

sites/{siteId}/drawingSets/{setId}/drawings/{drawingId}/revisions/{revisionId}/comments/{commentId}
  authorId, body, x, y, status: open|resolved, severity, createdAt, replies[]

sites/{siteId}/accessGrants/{grantId}
  userIds: [...], scope: {type: "site"|"set"|"drawing", id}, role
```

Binary files (PDF/image) live in **Firebase Storage**, path-mirrored to the
revision doc (`sites/{siteId}/drawings/{drawingId}/revisions/{revisionId}.pdf`);
Firestore holds only metadata + the storage reference.

Security Rules sketch (resolved against `accessGrants`, not hardcoded roles):
a read/write on a revision or comment first resolves the caller's effective
role by checking for a matching grant at `drawing` scope, then `set` scope,
then `site` scope, and allows the action if the highest role found meets the
operation's minimum (`viewer` for read, `commenter` for comment-write,
`editor` for new revision, `approver` for publish).

## 5. Platform & backend

- **Frontend**: React Native via Expo (managed workflow) — one TypeScript
  codebase, OTA updates for non-native-code fixes (useful for a field app
  where you can't always push through app-store review before a site
  manager needs a fix).
- **Backend**: Firebase — Auth, Firestore, Storage, Cloud Functions (for
  fan-out push notifications on publish, and for thumbnail/preview
  generation from uploaded PDFs), FCM for push.
- **Offline**: Firestore's offline persistence is enabled by default on
  mobile — reads/writes queue locally and sync on reconnect, which matters
  on a site with patchy signal. Storage downloads are cached explicitly
  (expo-file-system) so a drawing already opened once stays viewable
  offline.
- **PDF/image rendering & annotation**: `react-native-pdf` (or
  `pdf-lib` + a WebView renderer) for viewing, a custom Skia
  (`@shopify/react-native-skia`) overlay for comment pins and the
  amalgamation layering, since that needs pixel-accurate, scalable
  coordinates independent of the PDF renderer's own zoom/pan state.

## 6. React Native and platform-specific design patterns

This is the open question from the kickoff: React Native shares one
JS/TS codebase, but iOS (Human Interface Guidelines) and Android (Material
Design) have different navigation, chrome, and interaction conventions.
Approach: **shared data/business logic layer, platform-aware UI shell** —
don't try to force one visual language onto both OSes, and don't fork the
app into two codebases either.

- **Navigation**: `@react-navigation/native-stack` and `@react-navigation/
  bottom-tabs` already render platform-native primitives under the hood —
  iOS gets the native `UINavigationController` push/pop with edge
  swipe-back; Android gets the Material back-transition and respects the
  hardware/gesture back button automatically. This one choice covers most
  of "feels native" for free.
- **Component styling**: keep one component tree, branch styling with
  `Platform.OS` / `Platform.select()` rather than separate components,
  for things like: iOS action sheets (`ActionSheetIOS` / bottom sheet with
  iOS-style buttons) vs. Android's Material menu/snackbar; iOS swipe-to-
  reveal row actions vs. Android long-press context menu; haptic feedback
  (`expo-haptics`) on iOS interactions where Android would use a ripple
  instead.
- **Typography & iconography**: let the system default font apply (San
  Francisco on iOS, Roboto on Android — Expo/RN does this automatically
  when no custom font is set); use `expo-symbols` (SF Symbols) on iOS and
  Material Symbols/`react-native-vector-icons` on Android behind one icon
  abstraction component, so call sites don't know which they got.
- **Permissions & OS integration**: file picking, camera access, and push
  registration are configured per-platform through `app.json`/Expo config
  plugins (`NSPhotoLibraryUsageDescription` etc. for iOS,
  `android.permissions` for Android) — this is config, not code fork.
  Opening/sharing a drawing out of the app uses the native share sheet on
  iOS and an Android `Intent` chooser; `expo-sharing` abstracts both.
- **Push notifications**: Expo Notifications abstracts APNs vs FCM
  transport, but each still needs its own credentials set up (Apple Push
  key, FCM server key) — an infra step, not a UI fork.
- **What stays identical**: auth flow, Firestore queries, the comment/
  review data model, the amalgamation/overlay math, and the calibration
  workflow. None of that is visual-language-specific, so it lives in
  plain hooks/services shared by both platforms.

Net effect: expect maybe 10-15% of UI code to carry a `Platform.OS` branch
or a platform-specific library pick; the rest — screens, data fetching,
the overlay canvas — is identical.

## 7. Screens (v1)

1. **Sign in** — Firebase Auth (email/password or SSO if the site mandates it)
2. **Site / Drawing Set list** — sets the user has any access to
3. **Drawing Set detail** — member drawings, linchpin indicator, "View amalgamated"
4. **Amalgamated view** — layered canvas, per-layer opacity/visibility toggle
5. **Drawing detail** — revision history list, current published revision
6. **Revision viewer** — PDF/image render + comment pins + comment thread panel
7. **Upload / stage revision** — pick file, calibration step (first revision only), change note, submit as draft
8. **Review queue** — revisions `in_review` for users with `approver` role
9. **Access management** — per-Site/Set/Drawing grant editor (admin only)
10. **Notifications** — publish events, @mentions, resolved snags

## 8. Open questions for next pass

Resolved by direct founder input (§1.1): the install-location/rework
failure mode, email as the baseline to beat, per-trade drawing ownership
(no cross-trade authority), and publish-notification as the top-priority
fix. Still open:

- Push alert scope: notify on *every* publish to a Drawing/Set the user
  has access to, or only when the change falls within a region/area the
  user has flagged as relevant to them (avoids alert fatigue once a Set
  has many trades in it)?
- Do revisions need a formal diff/overlay (old vs. new) beyond "view both
  side by side," or is that a v2 feature?
- Should `require_resolved_before_publish` be a hard block or a warning an
  approver can override with a reason?
- CAD source files (DWG/RVT) — out of scope for v1 (PDF/image only), or do
  we need a conversion step (e.g. via a Cloud Function calling an external
  conversion API) before upload?
- Multi-site / multi-tenant admin (a contractor working across several
  sites) — same account, switch Site context, or separate accounts per site?
- Given no cross-trade authority existed on the reference site, who in the
  app's model is allowed to *create* a Drawing Set / designate the
  linchpin — any user, or does that need its own permission tier so Sets
  don't end up duplicated/conflicting across trades?
