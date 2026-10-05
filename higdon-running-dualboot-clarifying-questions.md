---
type: Note
belongs_to: "[[run-with-hal]]"
related_to: "[[run-with-hal]]"
---
# Higdon Running <> Dualboot Clarifying Questions

Dualboot and Higdon Running clarified RFP scope for the app rebuild: feature matrix, approvals, Flutter, maintenance, hosting, and budget. Dualboot expects $200 to 250 thousand, and user data migration is critical.


### Scope and Feature Matrix

- Feature matrix lists 146 features, 96 marked required; features vary in depth, not effort
  - Blue = in RFP, gray = not in RFP, boxed = implied by RFP
- Jake Higdon: most core historical functionality should carry over, with flows open to redesign (e.g. chat-based onboarding)
- Onboarding is critical to the brand's focus on beginner runners; Dualboot proposes putting the wizard before the paywall
- Plan adaptation draws complaints (e.g. 3 straight intense days to catch up); intent stays, implementation will be redesigned
  - Rodrigo Fernandez suggested persona-based pre-built plans adapted to availability


### Requirements and MVP Approach

- Short runway (joining 6 to 16 weeks out) and too-much-time handling seen as requirements not listed in the RFP
- Golden fixture test harness will validate the plan engine; client help needed for test scenarios
- Variable scope: product must be complete, usable, and migration-ready; MVP may ship a subset with features added later


### Engagement Structure and Technology

- Screens delivered weekly with a 2-day turnaround requested; David Higdon and Jake Higdon will tag team approvals
- Decision chain: Jake Higdon recommends, David Higdon decides with consensus from two siblings
- Native iOS plus Android rejected due to maintenance cost; cross-platform Flutter preferred over React Native for speed
- Spence is not involved going forward; Dualboot may use him as a consultant if desired


### Maintenance, Hosting and Ownership

- Client wants to own the code after the TrainingPeaks experience; Dualboot strongly cautioned against not owning it
- KTLO maintenance needed for 2+ major platform updates yearly plus Garmin and wearable SDK changes
- Scale: around 10,000 paying subscribers; 2025 had 140,000 downloads and 20,000 new subscribers
- Hosting likely AWS (possible incentive funds) or Azure; consolidate web and app in one account; monthly bills will vary


### Budget and Transition from TrainingPeaks

- Peter Klayman: under $150,000 is unrealistic; expected range is $200 to 250 thousand with some features cut
  - Design rethink alone is about 50 to 100 grand; higher risk premium due to open-ended redesign
- User data migration (contacts, training history, subscriptions) is the top priority to secure from TrainingPeaks
- App store listing is second priority; it works with user data, and data alone still allows email-driven migration
- Rodrigo Fernandez flagged subscription equivalence and verifying account ownership during migration


### Next Steps

- (David Higdon) Identify people in the business with deep product and training-plan knowledge for reviews
- (Peter Klayman) Send the feature matrix to the Higdon team
- (Alex Pecorella) Schedule a call to present the RFP response, then send it right after
- (Jake Higdon) Review the feature matrix further and consider which features are true requirements


### Decisions Made

- Approval cadence is weekly, with David Higdon as final decision maker
- Proposal review happens in two steps: David Higdon and Jake Higdon first, then the wider family
- App will be built cross-platform in Flutter
- No format preference for the RFP response (slides or written)
