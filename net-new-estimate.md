---
type: Note
belongs_to: "[[park-avenue-partners]]"
_organized: true
---
# Park Avenue Partners estimate

Tue, 07 Jul 26 · [peter.klayman@dualbootpartners.com](mailto:peter.klayman@dualbootpartners.com)

### Project Context: New Estimate

- Net-new client with 24 mobile home parks \(portfolio expected to grow\)
- ~2,400 total mobile homes across the portfolio
- Budget: under $100K, ideally under $50K
- Solution must be low-maintenance and turnkey
- Client uses “AI” language, but this is a data processing project, not AI/agentic
  - OCR/AI may apply narrowly \(e.g., PDF bill parsing\)

### Use Case 1: Leak Detection \(Three Sub-cases\)

- Master meter comparison: city meter vs. owner’s smart master meter
- Leak detection across distribution: master meter vs. 100 sub-meters per park
  - Discrepancy = leak in property pipes, owner’s responsibility
- Individual usage anomaly alerts
  - Compare tenant usage vs. 30-day historical average
  - Notify via API integration into RentCafe \(tenant platform\)
- Leak/discrepancy alerts go to 4 people per park: owner, regional manager, property manager, one other
  - Each park has its own notification contact list

### Use Case 2: Automated Billing

- 24 parks, each in a different water utility area
  - Some utilities use email bills, others require portal login
- Monthly process:
  1. Log in to each of the 24 utility portals and pull the bill
  2. Compare city meter usage \(from bill\) vs. owner’s smart master meter
  3. Apportion improvement fees equally across units
  4. Apply current water rate \(rates change monthly\) to calculate individual bills
  5. Upload billing amounts to RentCafe via API
- Smart meter protocols: research vendor docs \(companies listed in the brief\)

### Rodrigo’s Role on This Estimate

- Primary focus: technical validation, architecture, and proposed approach
  - Not the final estimator \(for now\)
  - A separate estimator \(assigned by Maddie\) will run in parallel for first 6 weeks
- Rodrigo’s deliverables:
  - Review and enrich the brief with technical context useful for estimators
  - Research RentCafe API, smart meter protocols, utility portal access strategies
  - Define technical strategy and approach independently
- Call with Rodrigo, Peter, and the assigned estimator planned for Wednesday or Thursday
  - Rodrigo to review and validate the estimator’s numbers
- Long-term intent: Rodrigo owns estimates fully, but strategy and business context come first

### Open Items

- Hours logging for Project X: unclear how to register now that role has shifted
  - Peter to clarify in a couple of days; to discuss with Felix on Friday
  - For now, only log Project X hours for work still being closed out with the delivery center

### Next Steps

- **Review the brief and begin preliminary technical estimate** \(Rodrigo\)
  - Focus on architecture, RentCafe API, smart meter protocols, and utility portal access strategies.
- **Schedule call with assigned estimator** \(Peter\)
  - Peter to confirm Maddie's estimator assignment; call targeted for Wednesday or Thursday 8th/9th July.
- **Clarify Project X hours logging** \(Peter\)
  - Raise with Felix on Friday 10th July; only close-out work should be logged for now.

---

Chat with meeting transcript: [https://notes.granola.ai/t/c71e6d08-baea-4df1-ad48-807efb0c725e-00best9l](https://notes.granola.ai/t/c71e6d08-baea-4df1-ad48-807efb0c725e-00best9l)
