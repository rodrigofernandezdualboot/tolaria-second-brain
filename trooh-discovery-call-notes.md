---
type: Note
related_to: "[[trooh]]"
status: Active
---

# Trooh — Discovery Call Notes

## Meeting Purpose and Scope

- Goal: understand Trooh’s ad deployment process before scheduling a deeper workflow session
- Trooh’s pain point: ad trafficking team deployment process is manual, complex, takes months to train staff
- Two sub-processes identified:
  - New campaign deployment (creative complete through to system deployment)
  - Campaign optimization (managing underdelivering campaigns, currently done weekly)
- Lean toward optimization as the POC focus: clearer process, more discrete, easier to demonstrate value

## Tech Stack and Data

- Primary platforms: Perion (formerly HiveStack) and Ayuda (by BroadSign)
  - Ayuda syncs screen inventory and properties with Perion
  - Ayuda also tracks player online status and delivers proof-of-play
- Trooh maintains two environments:
  - Production: live connection to screens, Perion UI, real-time reporting
  - Internal copy: SQL-queryable database replica, no UI, no live screen connection, data only
- Trooh also pulls Perion data into their own database for historical reporting
- Optimization decisions currently driven by weekly Perion reports; process feels like it could be more live

## POC Plan and Commercials

- POC hosted in Dualboot’s own demo environment (not Trooh’s), to reduce infrastructure cost and spin-up time
- Precursor steps before the 1-hour stakeholder session:
  - Execute NDA so Trooh can share sample reports and Perion/Ayuda data ahead of time
  - Explore bulk export of Ayuda historical data to use as dummy data for the POC
- Full engagement (post-POC) would be deployed in Trooh’s own AWS or Azure account
- Pricing model:
  - Professional services upfront: 2 workflows delivered in ~5 weeks
  - Ongoing: queue-based, ~$4,000/month per queue (covers platform + AI token costs)
  - 1 to 2 queues typical for client workflows; roughly equivalent to $0.50/hour vs. ~$50/hour for manual labor
- Next meeting: Trooh to bring subject matter experts focused on the optimization workflow

## Next Steps

- **Send NDA to Trooh**
- **Arrange 1-hour session with Trooh's digital/revenue ops team**
- **Explore bulk Ayuda data export for POC dummy data**
