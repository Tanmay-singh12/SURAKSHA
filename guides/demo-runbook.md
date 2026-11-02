# RESQ-ZONE — Hackathon Demo Runbook

## 1. Demo objective

Demonstrate one connected flood/heavy-rainfall scenario across citizen app, government dashboard, backend, AI/GIS and BLE offline communication.

## 2. Pre-demo checklist

- [ ] Backend running and connected to database.
- [ ] AI/GIS service healthy.
- [ ] Demo seed data loaded.
- [ ] Dashboard logged in with authorized demo account.
- [ ] Citizen app logged in with citizen account.
- [ ] Shared habitation and safe-site IDs match across modules.
- [ ] Risk data visibly labelled as demo/simulated where applicable.
- [ ] Two Android phones charged and BLE permissions granted.
- [ ] Gateway device has internet access for final synchronization.
- [ ] Network toggle tested before the presentation.
- [ ] Screenshots/video backup prepared in case hardware fails.

## 3. Suggested demonstration sequence

1. Open citizen app and show the risk map.
2. Submit a flood report from the citizen app.
3. Show the new pending report in the government dashboard.
4. Verify the report using an authorized officer account.
5. Show the habitation risk and priority output.
6. Open safe-site evaluation and show capacity/services.
7. Generate a relocation proposal.
8. Approve it through the authorized dashboard workflow.
9. Publish an official alert.
10. Show the alert in the citizen app.
11. Disable internet on the SOS-originating phone.
12. Create an SOS and show it queued locally.
13. Transfer it to a nearby Android phone using BLE.
14. Re-enable internet on the gateway device.
15. Synchronize the queued packet.
16. Show the SOS on the dashboard as unverified.

## 4. Narration guidance

Explain:
- Hazard identification alone does not determine who needs help first.
- Risk assessment combines hazard exposure with habitation and vulnerability context.
- A nearby site is not necessarily safe; capacity and services matter.
- AI/GIS generates decision-support recommendations.
- Authorities retain approval and evacuation control.
- BLE is a localized best-effort fallback, not unlimited-range connectivity.
- The backend deduplicates and records the synchronized SOS.

## 5. Fallback plan

If a live integration fails:
- Clearly state which part is simulated.
- Show a recorded physical-device BLE test if available.
- Use preloaded demo data for GIS outputs.
- Do not describe a mock as a live system.
- Keep the end-to-end story accurate and transparent.
