---
type: Note
_organized: true
---
# ArtsAVL Overview

- Arts agency in Buncombe County, North Carolina
- Promotes, advocates, and funds artists, arts organizations, and creative businesses
- Current platform: Ruby on Rails monolith with four modules:
  - Artist/org directory
  - Events calendar
  - Opportunities board (jobs, grants, scholarships)
  - Member portal (login, access, membership fees)
- Revenue from membership fees and ads
- Stripe integration already in place; CRM integration needed (name not specified in RFP)

## RFP Scope and Requirements

- Seeking a long-term strategic partner, not just an executor
- Six-month implementation timeline from contract execution (no hard start date)
- Proposals due August 15
- Key deliverables:
  - UX/UI redesign: search, filtering, mobile responsiveness, accessibility (WCAG)
  - User workflow improvements: profiles, events, opportunities
  - Admin tools: content moderation, management, settings
  - Analytics dashboards and member engagement tools
  - Third-party integrations: CRM, marketing platforms, cloud consolidation
  - Multitenancy and white-label architecture
  - API and content distribution tools
  - Ongoing maintenance (separate monthly/annual fee, not in project budget)
- Budget: $150K to $250K (all-in: discovery, design, dev, PM, travel)
- Client wants to keep and improve existing platform, not rebuild from scratch

## Key Technical Questions

- Current hosting unknown: AWS, Azure, GCP, or bare metal matters for scoping
- Data cleanliness and volume are major variables
- Ruby on Rails monolith: rebuilding on a new stack (e.g., .NET) could enable multitenancy out of the box and save cost, but client preference is to preserve what exists
  - If rebuilding is genuinely cheaper/better, this is a valid recommendation to bring to the August 11 meeting with justification
- CRM name not specified in RFP: confirm before the meeting

## Prep Strategy for August 11 Meeting

- Goal: arrive with recommendations and a position, not just questions
  - Show what can/can’t be done within the $150K-$250K budget
  - Prioritize scope if budget is tight; propose rescoping if needed
- Liam will be out next week; Rodrigo and Peter to lead prep
  - Rodrigo starts Monday after finishing current due diligence work
  - Peter aligned on the objective and available to collaborate
- Two value-add angles to explore before the meeting:
  - Accessibility audit: Liam’s contact (UX/accessibility expert) can do a WCAG compliance audit in ~2 hours as a sales asset
  - Loop in an Echo team member early given the explicit UX upgrade ask; bring them into a strategy review before the call

## Next Steps

- **Review full RFP and scope against budget** (Rodrigo)
- **Make intro to ArtsAVL contact and confirm Rodrigo/Peter are leading** (Liam)
- **Create a ticket for Rodrigo for this project** (Liam)
- **Arrange accessibility audit with Liam's contact** (Liam)
- **Loop in an Echo team member for UX strategy review**
