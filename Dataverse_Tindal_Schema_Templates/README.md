# Ecothought Passive Infrastructure — Dataverse Schema (with Base dimension)

This package contains CSV templates and implementation notes for deploying the schema in Microsoft Dataverse and wiring it into a Power Apps canvas app. It supports multiple locations (e.g., **Tindal**, etc.) via a master **Base** table.

## Tables (entities)

### 1) Bases
- **BaseId** (GUID, Primary Key)
- **BaseCode** (Text, *Alternate Key*, unique) e.g., TIN, EDN
- **BaseName** (Text) e.g., RAAF Base Tindal
- **State** (Text) e.g., NT
- **Timezone** (Text) e.g., Australia/Darwin
- **Notes** (Multiline Text)

**Purpose:** Tenant dimension so all data is partitioned by base.

---

### 2) Nodes
- **NodeId** (GUID, Primary Key)
- **BaseId** (Lookup → Bases)
- **NodeCode** (Text, *Alternate Key* with BaseId) e.g., ILSS019, DPN-460-G-5-01
- **NodeType** (Choice) Pit | Cabinet | Endpoint | Building Entry | Other
- **Building** (Text)
- **Room** (Text)
- **Latitude** (Number, optional)
- **Longitude** (Number, optional)
- **Status** (Choice) Draft | Active | Retired | Demolished
- **Notes** (Multiline Text)

**Alternate Key:** (BaseId, NodeCode)

---

### 3) Segments
- **SegmentId** (GUID, Primary Key)
- **BaseId** (Lookup → Bases)
- **FromNodeId** (Lookup → Nodes)
- **ToNodeId** (Lookup → Nodes)
- **SegmentKey** (Text, *Alternate Key* with BaseId) convention `FromCode|ToCode`
- **Distance_m** (Decimal Number, required)
- **Status** (Choice) Draft | Surveyed | Approved | Retired
- **SurveyDate** (DateOnly)
- **ApprovedBy** (Text / Lookup to Users)
- **Notes** (Multiline Text)

**Rules:**
- FromNodeId ≠ ToNodeId
- Base consistency: Segment.BaseId == FromNode.BaseId == ToNode.BaseId
- **Alternate Key:** (BaseId, SegmentKey)
- **Direction policy:** Choose either (a) directional (store both A|B and B|A if needed), or (b) canonical ordering: SegmentKey = MIN(NodeCodeA,NodeCodeB) & "|" & MAX(...)

---

### 4) Routes
- **RouteId** (GUID, Primary Key)
- **BaseId** (Lookup → Bases)
- **RouteName** (Text, *Alternate Key* with BaseId)
- **Purpose** (Text)
- **CreatedOn** (DateTime)
- **CreatedBy** (Text / Lookup to Users)
- **Notes** (Multiline Text)

**Rollup:** `TotalDistance_m` (Decimal, Rollup) — see below.

---

### 5) RouteSegments (intersect table)
- **RouteSegmentId** (GUID, Primary Key)
- **BaseId** (Lookup → Bases)
- **RouteId** (Lookup → Routes)
- **SegmentId** (Lookup → Segments)
- **SequenceNumber** (Whole Number) order of traversal
- **Notes** (Multiline Text)

**Calculated column:** `Distance_m_calc` (Decimal, Calculated) = Segment.Distance_m  
**Relationship rules:** Route.BaseId == Segment.BaseId == RouteSegment.BaseId

---

## Calculated & Rollup Columns

Because `TotalDistance` is on **Routes** but the raw value lives on **Segments** through **RouteSegments**, use a two-step approach:

1. On **RouteSegments** create **Calculated** column `Distance_m_calc` = `Segment.Distance_m`.
2. On **Routes** create **Rollup** column `TotalDistance_m` = **SUM** of related `RouteSegments.Distance_m_calc` (via the Route → RouteSegments relationship).

This ensures totals stay current if a segment distance is corrected.

---

## Business Rules / Constraints

- Prevent saving a Segment if FromNodeId = ToNodeId.
- Prevent saving a Segment if FromNode.BaseId ≠ Segment.BaseId (same for ToNode).
- Prevent adding a RouteSegment if Route.BaseId ≠ Segment.BaseId.
- Optionally lock `Distance_m` when Status = Approved (editable only by a role).

---

## Power Apps (Canvas) — key formulas

**Route total on a screen (without waiting for rollups):**
```powerfx
With(
    { segs: Filter(RouteSegments, RouteId = varRouteId) },
    Sum(segs, Segment.Distance_m)
)
```

**Gallery of ordered segments:**
```powerfx
SortByColumns(
    Filter(RouteSegments, RouteId = varRouteId),
    "SequenceNumber", Ascending
)
```

**Add a segment to a route (simplified):**
```powerfx
Patch(
  RouteSegments,
  Defaults(RouteSegments),
  {
    BaseId: drpBase.Selected.BaseId,
    RouteId: varRouteId,
    SegmentId: galSegments.Selected.SegmentId,
    SequenceNumber: CountRows(Filter(RouteSegments, RouteId = varRouteId)) + 1
  }
)
```

**Prevent duplicates (example OnSelect with existence check):**
```powerfx
If(
    CountRows(
      Filter(
        RouteSegments,
        RouteId = varRouteId && SegmentId = galSegments.Selected.SegmentId
      )
    ) > 0,
    Notify("Segment already in route", NotificationType.Error),
    Patch(...)
)
```

---

## Alternate Keys (Dataverse)

Create these Alternate Keys to enforce uniqueness:
- **Bases:** BaseCode
- **Nodes:** (BaseId, NodeCode)
- **Segments:** (BaseId, SegmentKey)
- **Routes:** (BaseId, RouteName)

---

## Security & Audit

- Use security roles: *Field Engineer* (create nodes/segments, not approve), *Design Engineer* (approve & lock), *Viewer*.
- Enable auditing on Segments.Distance_m and Status.
- Optionally add Power Automate flows:
  - On Segment status change → notify, write audit note.
  - On Route publish → render PDF summary & archive.

---

## Data Load (from existing Excel)

1. Prepare **Bases.csv** with at least one row for Tindal (TIN).
2. Extract unique nodes from your workbook (“Cable Segments” row nodes) into **Nodes.csv**.
3. Build **Segments.csv**: for each adjacent pair, compose `SegmentKey = FromCode|ToCode`, set `Distance_m` from survey.
4. Build **Routes.csv** and **RouteSegments.csv** from your existing “runs” rows.

This package includes **empty CSV templates** you can fill and import via Power Query/Dataflows or the Dataverse Import Wizard.

---

## GIS & QR (optional)

- Add latitude/longitude to Nodes to enable map visuals in Power BI.
- Generate QR codes from NodeCode and attach to pits/cabinets; Power Apps can scan to select nodes on-site.

---

## Files in this folder

- Bases.csv
- Nodes.csv
- Segments.csv
- Routes.csv
- RouteSegments.csv
- This README.md
