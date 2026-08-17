---
type: Note
belongs_to: "[[run-with-hal]]"
related_to: "[[run-with-hal]]"
_width: wide
_organized: true
---

# Run With Hal — Client Question Set

**For:** August 20 call with the Higdon family (David, Spence, Hal's sons) plus Ekko
**Prepared from:** agent debate session, App Store sentiment analysis, Figma flow board, `#run-with-hal-dealteam-sales`

---

## How to use this

The 20th is a reveal, not a discovery call. The family should hear questions that demonstrate we've already understood their business; the forensic questions belong with Spence, offline, before or after.

Three tiers:

- **Tier 1 — Ask Spence before the 20th.** Blocking. Answers change what we can credibly present.
- **Tier 2 — Ask in the room.** Framed to show insight, not extract data. These make us look like we've done the work, because we have.
- **Tier 3 — Ask once engaged.** Real questions, wrong moment.

A note on tone for Tier 1: every one of these can be asked as "here's what we'd need to protect you," never as "prove you own your own product." The family may not know the answers, and finding out they don't own their developer account in front of their father is not the moment we want to create.

---

# TIER 1 — Spence, before the 20th

## The contract with Peaksware

This is the single highest-value item in this document. Alex's "the data and the code is the IP" suggests the incumbent is asserting ownership. Whether that assertion holds is a contract question, and no architecture we propose can fix an unfavourable answer.

1. Can we see the master services agreement between the Higdon entity and Peaksware / TrainingPeaks — specifically the IP assignment, data ownership, and termination clauses?
2. Does the agreement contain a transition-assistance or transition-services clause? Any obligation to return data or assist a successor vendor on termination?
3. Is there a data-processing agreement or privacy addendum? If Peaksware is a processor rather than a controller, they may have an obligation to return or delete data on instruction — a very different legal position from owning it.
4. What's the notice period and termination date? Is the relationship in its term, in renewal, or already terminable?
5. Has anyone put the request for data and account transfer in writing yet? If not, we should — a written refusal is more useful than an ambiguous conversation.
6. Has outside counsel reviewed the agreement in light of the Garmin acquisition?

## Developer accounts and the store listings

Camila's note says Peaksware holds the developer accounts. Confirming precisely how is worth more than any other technical fact.

7. Which legal entity is named on the Apple Developer Program membership and the Google Play developer account? Not who built the app — whose name is on the team.
8. **Who receives the App Store and Play Store payouts?** This is the cleanest proxy for account ownership, and it's a question about money rather than a question about law, so it usually gets a faster and more honest answer.
9. Does anyone on the Higdon side have any level of access to App Store Connect or Play Console today — even read-only App Analytics?
10. If the answer to 9 is no: the family currently has no visibility into their own install base, retention, or subscriber count. Are they aware of that, and is it something we should raise on the 20th?
11. Who owns the `Run With Hal` app name and trademark, and who owns the domain and website?

## The Garmin acquisition

Spence raised this himself on July 29 — "it puts the connection with Garmin in risk." Worth understanding what he actually knows versus fears.

12. What has Peaksware or Garmin communicated since the July 22 acquisition closed? Anything in writing?
13. Who is the counterparty now — still the Peaksware account team, or has this moved to Garmin corp dev or legal?
14. Does Spence's read that the Garmin integration is at risk come from something they were told, or is it his inference?
15. Is there a separate agreement between the Higdon entity and Garmin for the Connect integration, or does that integration sit under Peaksware's developer credentials?

**Why 15 matters:** if the Garmin OAuth credentials belong to Peaksware, then Garmin now owns the integration, the app's code, the user data, *and* a competing product in TrainingPeaks. We should assume that integration does not survive and design around it.

## The Hal Higdon content licence

16. Who holds the licence to the training programs — the Higdon family entity directly, or was it granted to Peaksware?
17. Same question for the book content: who holds digital and audio rights to the books, and is a publisher involved?

**Why 17 matters:** the custom-podcast concept from the internal thread depends on book rights that may sit with a publisher, not the family. Worth knowing before we present it.

## RFP and process

18. Is the RFP still happening, or has the August 20 meeting replaced it?
19. Who decides? Spence, David, Hal, the sons collectively — and is there anyone not in the room on the 20th whose sign-off is required?
20. Is Peaksware still being invited to bid? If they know they're being replaced, that's the worst possible posture for the cooperation we need from them.
21. Is there a budget range and a target date, and is the date driven by anything external — a contract expiry, a race season, a book release?

---

# TIER 2 — In the room, on the 20th

These are framed as insight-led. Each one signals that we've studied the product.

## Opening the migration conversation

22. "We've assumed you should plan for the scenario where Peaksware hands over nothing at all. Is that how you're thinking about it, or do you see a cooperative path?"

*Sets the frame honestly and positions the harder scenario as prudence rather than pessimism.*

23. "The thing we'd want to protect hardest is your ability to *reach* your existing users. Today, is there any channel that gets to app users who never signed up on halhigdon.com — a newsletter, a CRM, anything?"

*This is the real constraint from the debate. Ask it as protection, not as a gap.*

24. "If we can only recover part of a user's history, which part matters more to your runners — their run log, or the record of which plans they completed and which races they trained for?"

*Tom versus Sofía, handed to the client. Their answer sets the first-release priority.*

25. "How would you want to communicate to a runner mid-plan that some of their history didn't survive? We think being early and explicit beats being discovered."

## Users and seasonality

26. Roughly how many active users, and how many paying Hal+ subscribers?
27. What's the seasonal shape of the base — is there a genuine off-season, or does the southern hemisphere and autumn-marathon population keep it flat year-round?

*Drives cutover timing. The debate's conclusion was migrate in the off-season; we need to know whether one exists.*

28. What proportion of users are mid-plan at any given time, and how long is the typical plan — are most people on 18-week marathon builds or shorter 5K and 10K programs?
29. Do you have any sense of the sign-in split across Apple, Google and email?

*Sizing the unrecoverable-identity population. Frame it as "this determines how many people we can bring across.")*

## Monetization — the reason the ratings fell

30. "Reading the reviews against your own flow, the paywall sits before a new user has seen a single workout. Was that the intended design, or something the partner shipped?"

*Establishes that we found the cause of the Q3 2025 collapse. Also politely finds out whether the family chose the paywall or had it done to them.*

31. What was the commercial thinking behind Hal+, and what did it actually do to revenue? Did the rating collapse cost more than the subscription earned?
32. Are you open to revisiting the model — a genuinely useful free tier with paid depth, rather than a gate at the front door?
33. Who set the current pricing, and is it contractually fixed under the Peaksware arrangement?

## Product, grounded in the flow board

34. "The onboarding wizard asks for easy running pace but never a goal race pace — and 'can't set a target race pace' is a recurring complaint in the 2026 reviews. Was a goal-pace model ever part of the product?"

*The strongest single credibility moment available. It ties their reviews to their own flow diagram.*

35. "Plan Settings routes straight back to the paywall, labelled 'features locked.' So the moment a runner needs to move Thursday's run to Friday, they hit a wall. Is schedule flexibility something you want in the free tier?"
36. What is "Hal Says" meant to be, and is the logic behind it something the family specified or something the partner built? Reviews say the automatic adjustments "seem really off."
37. The wizard is ten steps before a user sees anything. How attached are you to collecting all of it upfront?
38. On attracting a younger demographic — what does that mean concretely? Different distances, different tone, social features, or the gamification direction Ekko would take?

## Brand and Hal himself

39. How involved can Hal be in the product going forward — recorded audio, written guidance, appearances in-app?
40. If we used a synthesized voice for audio content, how would the family feel about that? We'd rather ask than assume.

*Directly relevant to the podcast concept, and asking about voice rights before building is the right instinct.*

41. What does the family want this to be in three years — a training app, or a broader Higdon platform with books, audio and community?

---

# TIER 3 — Once engaged

## Integrations

42. What proportion of users have Garmin connected? Apple Health? Strava?
43. Does the app currently write workouts to HealthKit, or only read from it?

**Why 43 matters:** if it writes, years of run history sits on users' own devices and we can recover it with permission alone, independent of Peaksware and independent of Garmin. This is the most valuable technical question in this section and it may quietly solve the history problem.

44. Does the app push structured workouts to Garmin devices, or only pull completed activities? The push capability requires a separate Garmin approval tier with its own timeline.
45. Any other integrations not visible in the flow board — Apple Watch complications, Siri, widgets, Strava posting?

## Support and operations

46. Who owns the Zendesk instance, and can the ticket history be exported? Support scored 2.40 across 25 reviews and it's the second-worst theme in the dataset.
47. Who answers support today, and what would support capacity look like at cutover? Reviews describe unanswered emails and refund disputes.
48. How have refund requests been handled, and who bears the cost? If we inherit active subscriptions we inherit that queue.
49. Is there analytics instrumentation — Firebase, Amplitude, Mixpanel — and whose account is it in?

## Data specifics

50. If Peaksware does cooperate, what would they actually be able to produce, and in what format? A database dump, a CSV export, an API?
51. Would Peaksware respond to user-initiated data access requests under GDPR or CCPA? Slow and low-completion, but it's a self-service recovery path for the users who care most.
52. Is there a staging or test environment, any documentation, or any source access at all today?

---

# If you only get five minutes with Spence

1. Can we see the Peaksware agreement — IP, data ownership, termination assistance?
2. Whose legal entity is on the Apple and Google developer accounts, and who receives the store payouts?
3. What has Garmin or Peaksware said since July 22, and who's the counterparty now?
4. Who holds the training-plan licence — the family or Peaksware?
5. Is there any channel that reaches app users who aren't on the website list?

---

## Two things to say rather than ask

**Name the fork.** We can present a cooperative-handoff path and a hostile-handoff path with different scope and different prices, rather than one plan that quietly assumes cooperation. That's exactly what Camila proposed on July 21, and it's more credible than optimism.

**Offer the leverage.** The strongest thing we can hand them on the 20th isn't an architecture. It's the observation that the Peaksware contract, not the technology, decides how much this costs them — and a recommendation to get it reviewed this week. Nobody bidding against us will lead with that.
