Ecothought Dataverse Prefill — Tindal (BaseCode=TIN)

Publisher Prefix: def_

- BaseId: 597f54cb-cb58-4fa0-9625-11b671e6535a
- Nodes: 306 records prefilled from Pits, Cabinets, and Building Endpoints.
- Segments, Routes, RouteSegments: left empty for import-ready template.

Next steps:
1. Import Bases.csv, then Nodes.csv into Dataverse.
2. Create Segments linking FromNodeId→ToNodeId with fixed Distance_m.
3. Create Routes and RouteSegments in sequence.
4. Add calculated/rollup columns on def_routesegment.distance_m_calc and def_route.totaldistance_m.
