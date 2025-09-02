Ecothought Dataverse Prefill — Tindal (BaseCode=TIN)

- BaseId: 86e28873-4287-4ae2-bc9e-709e974ed661
- Nodes: 306 records prefilled from Pit Inventory, DPN/DSN Cabinet Inventory, and Building Endpoints.
- Segments/Routes/RouteSegments: left empty for clean import; populate after segment survey.
- NodeCode conventions:
  - Pits: PIT-#### (zero-padded)
  - Cabinets: as per "Cabinet Numb er" field (e.g., TIN-460-G-5-01)
  - Endpoints: Building Number (e.g., C0460)
- Next steps:
  1) Import Bases.csv, then Nodes.csv into Dataverse.
  2) Create Segments by selecting FromNodeId/ToNodeId and setting Distance_m (fixed).
  3) Create Routes and add RouteSegments in sequence.
  4) Add calculated/rollup columns as per the earlier README (Distance_m_calc on RouteSegments; TotalDistance_m rollup on Routes).

Generated: 2025-09-02T22:31:26
