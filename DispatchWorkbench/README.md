# Dispatch Workbench (D365FO)

Custom D365FO model that provides a **Dispatch Workbench** for planning outbound deliveries over a rolling 1–3 day window.

## Capabilities

| Requirement | Implementation |
|---|---|
| View orders needing dispatch | `DispPendingOrderTmp` temp table populated by `DispRouteOptimizer::loadPendingOrders`, filtered on backorder `SalesLine.ShippingDateRequested` and absence of an existing `DispDispatchLine`. |
| Optimise routes | `DispRouteOptimizer::sequenceDrops` — nearest-neighbour heuristic on great-circle distance with a soft time-window penalty (swap in Bing Maps / OR-Tools later). |
| Locations | Origin picked on the form (`InventLocationId`); drop lat/long resolved from `LogisticsPostalAddress`. |
| Time windows | `TimeWindowFrom` / `TimeWindowTo` per drop; late arrivals penalised in the sequencing score. |
| Fleet availability | `DispVehicleTable.Available` + exclusion of vehicles already booked on the same date in `DispDispatchHeader`. |
| Vehicle config | `VehicleType`, `MaxWeightKg`, `MaxVolumeM3`, `MaxDrops`, `RequiredCompetency`. |
| Driver competency | `DispDriverTable.Competency` (enum) matched against the max required across selected drops. |
| Allocate truck/driver | `DispRouteOptimizer::createDispatch` picks the first eligible vehicle and driver and creates a header + drop lines in one `ttsbegin/ttscommit`. |
| Multi-drop | `DispDispatchLine.DropSequence`; header aggregates weight/volume/drop count. |
| 1–3 day schedule view | Form filters `DispatchDate` range; date controls clamp the window to 3 days with a warning. |

## Object list

- Enums: `DispDispatchStatus`, `DispVehicleType`, `DispDriverCompetency`
- EDTs: `DispDispatchId`, `DispVehicleId`, `DispDriverId`
- Tables: `DispVehicleTable`, `DispDriverTable`, `DispDispatchHeader`, `DispDispatchLine`, `DispPendingOrderTmp` (TempDB)
- Class: `DispRouteOptimizer`
- Form: `DispDispatchWorkbench` (3 tabs: Pending orders, Fleet schedule, Fleet master)
- Menu item: `DispDispatchWorkbench` (display)
- Security: privilege `DispDispatchWorkbenchMaintain`, duty `DispDispatchPlanning`
- Labels: `DispatchWB.label.txt` (en-US)

## Deploy on a Tier-1 dev VM

1. Copy the `DispatchWorkbench` folder into `K:\AosService\PackagesLocalDirectory\` (or your equivalent).
2. In Visual Studio: **Dynamics 365 → Model management → Refresh models**.
3. Open the project, add a **Model reference** to `ApplicationSuite`, `ApplicationPlatform`, `ApplicationFoundation`.
4. Create a number sequence code `Disp_1` (used in `createDispatch`) via **Organization administration → Number sequences**.
5. Build the model, DB sync, then run the menu item `DispDispatchWorkbench`.

## Extension hooks

- Replace `DispRouteOptimizer::sequenceDrops` with a real VRP solver.
- Replace `haversineKm` distance with a road-network service call (Bing Maps `Distance Matrix API`).
- Wire dispatch confirmation to update `SalesLine.ShippingDateConfirmed` and the load/shipment in WHS/TMS if you use them.
