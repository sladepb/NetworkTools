
# Ecothought — Passive Infrastructure (Tindal) Dataverse Solution Checklist
**Generated:** 2025-09-02T22:47:12

> **Publisher Prefix:** `def` (adjust if your solution uses a different prefix)

---

## 1) Tables (Entities) and Columns

### 1.1 Bases (`def_base`)
- **Primary Name:** Base Name (`def_basename`) — Single line of text (max 200)
- **Primary Key:** `def_baseid` (GUID)
- **Columns:**
  - `def_basecode` — Single line of text (max 10), **Required**, **Alternate Key**
  - `def_basename` — Single line of text (max 200), Required
  - `def_state` — Single line of text (max 10)
  - `def_timezone` — Single line of text (max 100)
  - `def_notes` — Multiline text

- **Alternate Keys:**
  - `def_basecode`

---

### 1.2 Nodes (`def_node`)
- **Primary Name:** Node Code (`def_nodecode`) — Single line of text (max 100)
- **Primary Key:** `def_nodeid` (GUID)
- **Columns:**
  - `def_baseid` — **Lookup** → Bases
  - `def_nodecode` — Single line (max 100), **Required**
  - `def_nodetype` — **Choice** (Option Set): Pit | Cabinet | Endpoint | Building Entry | Other
  - `def_building` — Single line (max 50)
  - `def_room` — Single line (max 50)
  - `def_latitude` — **Decimal Number** (Precision: 6, Scale: 6) *(or Floating Point)*
  - `def_longitude` — **Decimal Number** (Precision: 6, Scale: 6)
  - `def_status` — **Choice**: Draft | Active | Retired | Demolished
  - `def_notes` — Multiline text

- **Alternate Keys:**
  - Composite: `def_baseid` + `def_nodecode`

---

### 1.3 Segments (`def_segment`)
- **Primary Name:** Segment Key (`def_segmentkey`) — Single line of text (max 201)
- **Primary Key:** `def_segmentid` (GUID)
- **Columns:**
  - `def_baseid` — **Lookup** → Bases
  - `def_fromnodeid` — **Lookup** → Nodes
  - `def_tonodeid` — **Lookup** → Nodes
  - `def_segmentkey` — Single line (max 201), **Required**
    - **Convention:** `FromNodeCode|ToNodeCode` (or canonical min|max ordering if non-directional)
  - `def_distance_m` — **Decimal Number** (Precision: 18, Scale: 2), **Required**
  - `def_status` — **Choice**: Draft | Surveyed | Approved | Retired
  - `def_surveydate` — Date Only
  - `def_approvedby` — **Lookup** → System User *(or Single line of text if preferred)*
  - `def_locked` — **Two Options** (Yes/No) — controls editability in forms
  - `def_distance_m_approved` — **Decimal Number** (18,2) — snapshot at approval
  - `def_notes` — Multiline text

- **Business Rules (recommended):**
  - Prevent save if `def_fromnodeid` = `def_tonodeid`.
  - Prevent save if Bases differ across Segment and the two Nodes.
  - Make `def_distance_m` **read-only** if `def_locked` = Yes.

- **Alternate Keys:**
  - Composite: `def_baseid` + `def_segmentkey`

---

### 1.4 Routes (`def_route`)
- **Primary Name:** Route Name (`def_routename`) — Single line of text (max 200)
- **Primary Key:** `def_routeid` (GUID)
- **Columns:**
  - `def_baseid` — **Lookup** → Bases
  - `def_routename` — Single line (max 200), **Required**
  - `def_purpose` — Multiline text
  - `def_status` — **Choice**: Draft | Published | Archived
  - `def_totaldistance_m` — **Rollup** (Decimal 18,2) — see §2
  - `def_notes` — Multiline text

- **Alternate Keys:**
  - Composite: `def_baseid` + `def_routename`

---

### 1.5 Route Segments (`def_routesegment`)
- **Primary Name:** Route Segment (`def_routesegmentname`) — Autonumber or concatenation (optional)
- **Primary Key:** `def_routesegmentid` (GUID)
- **Columns:**
  - `def_baseid` — **Lookup** → Bases
  - `def_routeid` — **Lookup** → Routes
  - `def_segmentid` — **Lookup** → Segments
  - `def_sequencenumber` — **Whole Number**
  - `def_distance_m_calc` — **Calculated** (Decimal 18,2) — see §2
  - `def_notes` — Multiline text

- **Relationships:**
  - Routes (1) → RouteSegments (N)
  - Segments (1) → RouteSegments (N)
  - Bases (1) → RouteSegments (N)

---

## 2) Calculated & Rollup Columns

### 2.1 Calculated Column on RouteSegments
- **Table:** Route Segments (`def_routesegment`)
- **Column:** `def_distance_m_calc` (Decimal 18,2)
- **Calculation:** *Related* Segment → `def_distance_m`
  - In the calculated column editor: **Add related entity** → Segment (via `def_segmentid`) → choose `def_distance_m` as the result.

### 2.2 Rollup Column on Routes
- **Table:** Routes (`def_route`)
- **Column:** `def_totaldistance_m` (Decimal 18,2)
- **Rollup Definition:**
  - **Related entity:** Route Segments (relationship `def_route` → `def_routesegment`)
  - **Aggregate:** **SUM** of `def_distance_m_calc`
  - **Filters:** none (or add `def_routesegment.def_segmentid` not null)
  - **Frequency:** default (12 hrs) — you can trigger on-demand or via Flow (see §4).

---

## 3) Choice (Option Set) Definitions

### 3.1 Node Type (`def_nodetype`)
- Pit (100000000)
- Cabinet (100000001)
- Endpoint (100000002)
- Building Entry (100000003)
- Other (100000004)

### 3.2 Node Status (`def_status`)
- Draft (100000000)
- Active (100000001)
- Retired (100000002)
- Demolished (100000003)

### 3.3 Segment Status (`def_status` on Segments)
- Draft (100000000)
- Surveyed (100000001)
- Approved (100000002)
- Retired (100000003)

### 3.4 Route Status (`def_status` on Routes)
- Draft (100000000)
- Published (100000001)
- Archived (100000002)

> **Note:** You can share a global option set for Status across tables or keep separate lists if the lifecycles differ.

---

## 4) On-Demand Rollup Recalc

To force-refresh `def_totaldistance_m` after edits:
- Use **Power Automate** “Dataverse → Perform a bound action”
  - **Action name:** `Microsoft.Dynamics.CRM.CalculateRollupField`
  - **Target:** Routes record (via RouteId)
  - **FieldName:** `def_totaldistance_m`
- Or open a Route record in a model-driven app and click **Recalculate** on the rollup field.

---

## 5) Security & Audit

- Roles: **Field Engineer**, **Design Engineer**, **Viewer**.
- Enable **Auditing** on `def_segment.def_distance_m`, `def_status`, and `def_locked`.
- Business rule: make Distance read-only if Locked = Yes.
- Optional: **Access Teams** per Base to partition access.

---

## 6) Keys & Referential Integrity (Summary)

- Alternate Keys:
  - Bases: `def_basecode`
  - Nodes: `def_baseid` + `def_nodecode`
  - Segments: `def_baseid` + `def_segmentkey`
  - Routes: `def_baseid` + `def_routename`

- Referential checks (Business Rules or Plugins):
  - Segment.Base = FromNode.Base = ToNode.Base
  - RouteSegment.Base = Route.Base = Segment.Base
