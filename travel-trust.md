---
type: Project
status: Active
related_to:
  - "[[rodrigo-fernandez]]"
  - "[[aixle-flow]]"
url: https://www.traveltrust.com
_organized: true
---

# Travel Trust

> **Status:** discovery. Process doc received 6 Oct 2026. Next: clarifying questions + 30-min working session with the coordinator who owns the process, then a fixed-bid scope with an AWS funding estimate.

## The deal

- AWS-originated (Gianluca Lachini, AWS). Deal owner: Blaine Wagner. DBP team: Mike Dennis, Rodrigo Fernandez.
- Blaine flagged it as an **Aixle Flow use case**.
- Slack: #traveltrust-dealteam-sales

## Who they are

- Corporate travel agency founded 1983. President: Richard Meyerson (Richard@traveltrust.com).
- ~$8M revenue, ~20 staff, offices in the US, UK, Australia and NZ.
- AWS spend is minimal (~$4K/yr). A third-party provider handles their IT and AWS connectivity.
- Booking system is Sabre (GDS). A PNR is the reservation record in the GDS.

## The problem

One corporate client prepays all its travelers' hotel rooms. For every stay, a coordinator must get the hotel's credit card authorization completed and confirmed before check-in. Today it is fully manual:

1. Agent books the reservation and places it on **Queue 298** in Sabre.
2. Coordinator pulls the queue for check-ins within the next week.
3. Logs each one in an Excel tracker (Date, PNR, Name, Company, Agent, Conf #, HOD, Hotel Name - Phone, Check In, Check Out, CC Used, Notes).
4. Calls the hotel and asks for the auth form to be emailed to `cc@traveltrust.com`. Chases for days when needed; hunts for an email address when the phone goes nowhere.
5. Some hotels have a web auth link, but Travel Trust's confirmation number is rejected there. The coordinator knows which hotels do this and asks for the correct number first.
6. On receipt, reviews the PNR for coverage (room + tax vs. all charges), form of payment, card details and billing address, then completes the form.
7. Extra hurdles: verification codes sent to the cardholder (must be forwarded back within a short window, and she only learns a code is coming after submitting); **Canary** flows that require a QR scan + driver's license or a temporary test deposit. Sometimes she asks the hotel to waive the step, which triggers a new auth request.
8. Calls the hotel at least one day before check-in to confirm the auth is attached. Many hotels do not attach it unless prompted.
9. Updates Excel and adds notes to the Sabre PNR so agents and after-hours staff can verify.

**Cost of the problem:** 1–2 hours of staff time every weekday (~250–500 hrs/yr). About 5% of the time the traveler arrives and the hotel has no record of the prepayment, which embarrasses Travel Trust in front of the client.

## Working solution shape

AI agent handles the admin: pull the queue, track, email and chase hotels, pre-fill forms, send pre-arrival confirmations, write notes back to Sabre. A human stays in the loop for card numbers, verification codes and ID checks, to stay out of PCI scope and keep the build small.

## Commercial constraints

- Richard is cost-sensitive: one fixed price, no hourly billing. Explicitly does not want ~$35K for setup.
- Budget may not open until 2027. A priced proposal now is something he keeps in his back pocket.
- He knows off-the-shelf travel tools exist, so we must justify custom.
- AWS funding available to offset cost.

## What we need to investigate

### Open questions from the deal team

- [ ] Weekly authorization volume, and whether this covers only the one client.
- [ ] Sabre API access: do they have it, or does the third-party provider control it?
- [ ] Email platform behind `cc@traveltrust.com` (M365, Google Workspace, other).
- [ ] Share of hotels that respond by email vs. phone only; how often Canary-style flows appear.
- [ ] What "HOD" means in the tracker.

### Sabre integration

- [ ] Which Sabre product they use (Sabre Red 360 desktop only, or API credentials too). Who owns the PCC and the contract.
- [ ] Can we read Queue 298 and write PNR remarks programmatically? Any extra Sabre fees for API access?
- [ ] Which confirmation number lives in the PNR vs. the one the hotel's system expects. Is the "wrong conf #" problem fixable at the source (booking channel) rather than worked around?
- [ ] Check DBP's internal Sabre experience (several team members have worked with it) for a head start.

### The phone problem

- [ ] Steps 4 and 8 are phone calls. An email-only agent leaves the pre-arrival confirmation call manual, and that call is the control against the 5% failure. Decide: keep it human, use a voice agent, or push hotels to confirm by email. Each changes scope and price.
- [ ] Where does the 5% failure come from? Hotel never attached the auth, auth expired, wrong conf #, or a call that never happened? Do they log these incidents?

### PCI and card data

- [ ] Completed auth forms carry full card numbers. If they flow through `cc@traveltrust.com`, that inbox is already in card-data scope, and an agent that reads it may be too. Confirm what the agent would touch.
- [ ] Who holds the card details today (PNR, separate vault, emailed by the client)?
- [ ] Is there a virtual card or a single corporate card per traveler?

### Build vs. buy

- [ ] Which off-the-shelf tools Richard has looked at, and why they did not fit.
- [ ] Whether a virtual card / hotel-payment provider, or a Sabre add-on, removes most of the manual work without a custom build.

### Delivery and hosting

- [ ] Is Aixle Flow the delivery platform? Check it has (or can get) connectors for Sabre and their email platform.
- [ ] Where it runs: their AWS account (managed by the third party) or DBP-hosted. Who operates it after go-live.
- [ ] Run cost target, given ~$4K/yr AWS spend today.
- [ ] Where the Excel tracker lives (local, SharePoint/OneDrive). Replace it or keep it as the system of record?
- [ ] Does the process run from one office or across US/UK/AU/NZ time zones?

### Knowledge capture

- [ ] The coordinator holds tacit knowledge: which hotels reject the conf #, which departments handle auths, hotel email contacts. Is any of it written down? This becomes the agent's hotel directory.

### Commercial

- [ ] AWS funding program and expected amount.
- [ ] Price ceiling that clears Richard's bar (below the ~$35K he rejected).
- [ ] Baseline metrics to prove ROI: hours per week, auths per week, failure rate.

## Next steps

1. Send the clarifying questions to Richard.
2. 30-min working session with the coordinator who owns the process.
3. Fixed-bid scope with AWS funding estimate.

## Sources

- Slack: Blaine Wagner's deal brief, #traveltrust-dealteam-sales, 7 Oct 2026
- Email: "TravelTrust & Dualboot/AWS" thread (process doc from Richard, 6 Oct 2026)
