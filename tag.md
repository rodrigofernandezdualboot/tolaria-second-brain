---
type: Project
_organized: true
---
# TAG (The Aldridge Group)

## Value proposition — response to Ryan \(walks his 8 questions\)

> Draft answer to Ryan's 2026-07-03 ask: how Dualboot is uniquely positioned to win beyond price. Pricing/margin posture is a leadership call and deliberately out of scope here — this is the technical de-risking and differentiation story to message the client. See also [[tag-open-questions]] and [[sales-engineering-role]].

**Framing.** Everyone will scope this near ~2,000 hours, so hours-and-rate is a losing game. TAG isn't buying engineering; they're buying the removal of an existential risk on their \#1 revenue system — 20 years of validated scoring IP, run by a retiring solo developer, that has to come out identical. We win by being the firm that most credibly makes this a *safe* decision. We aim the message where the RFP puts its points: IP preservation & validation \(25%\), legacy .NET/SQL modernization \(20%\), team quality \(15%\), build-vs-buy judgment \(10%\).

**1 · How we're uniquely positioned \(at any price\).** Proven expertise in *their* business, not generic engineering. We have done a near-identical engagement \(Paradigm\): took over a proprietary personality scoring-and-report platform, isolated the scoring engine, multilingual reports, .NET + Angular. Client-facing line: *"We take a 20-year, revenue-critical, retiring-developer system off your risk register — with a proven method to prove your scoring comes out identical before any cutover, and an architecture sized to a team without an ops function."* \(Pricing/margin posture is set at the leadership level.\)

**2 · How we technically de-risk it.** Concrete method, not a promise:

- In **Knowledge Transfer \(1A\)**, assess unit-test coverage of the scoring module \(already partially isolated → wrappable\). With a solid test-case set we guarantee **bit-to-bit parity** between the unwrapped legacy scoring and the new wrapped version.
- Propose a **CI/CD pipeline with quality gates that run the unit tests on every change** as a non-negotiable correctness guarantee; use our **proprietary test-automation tooling** to accelerate coverage where automation is needed.
- Golden test cases built from known input→expected-output sets \(the Paradigm approach\), a **parallel run** reconciling new vs. legacy, and a **reversible cutover** \(feature toggle\) so we can fall back with no downtime.
- Hold a **re-baseline checkpoint** after 1A source review \(code released only post-award\) and a reserve if the WCF scoring service needs porting rather than a clean wrap.
- **Honorlock** \(see below\) is de-risked: we build a vendor-agnostic adapter now, stub-verified, and connect the live vendor in Phase 3.
- Where a client won't expose the algorithm itself, we have a stricter validation model \(Mossa\): keep the secret parameters in a **client-controlled Azure Key Vault** and run a **client-led fine-tuning / deviation-reporting** cycle — proving correctness without us ever holding the IP.

**3 · How we build credibility/authority.** Lead the top-3 call with three "crown jewels": \(a\) we've already isolated a proprietary scoring engine and proven parity; \(b\) our data governance survived a real GDPR complaint \(tracing + encryption in transit/at rest + key rotation proved no leak\); \(c\) we know from A/B testing that question layout can move scores — so our UI refresh is psychometrically aware, not cosmetic. Supporting assets: a working **prototype** \(TAG questionnaire UI using Paradigm UX\) and Paradigm sample reports. We cite a **senior solution-architect bench with 15+ years on .NET/Azure/SQL** rather than naming individuals \(no confirmed start date, can't commit specific people\).

**4 · What we see \(technical read\).** Legacy core is VB.NET WebForms / .NET 3.5 \(2007\) with no .NET Core upgrade path → the Test Delivery app is a genuine rebuild. Scoring is a C\# **WCF** service \(`Service1.svc.cs`\) TAG lists as a *wrap* candidate while also flagging WCF as *rebuild* — the single biggest technical unknown, resolvable only with the code. Everything runs on one server \(`berniedesktop11`\) — a single point of failure with no CI/CD, monitoring, or alerting. The SuccessFactors API contract is fixed and must be mimicked exactly. Phase 1 must run in parallel with zero downtime. Honorlock cannot attach to the legacy stack — the technical catalyst for the whole effort.

**5 · What the client must avoid.**

- A **big-bang cutover** — must run in parallel with a reversible fallback \(legacy-displacement pattern\).
- **Over-engineering** — full multi-tenant SaaS or fine-grained microservices would hand a team with no ops function an unmanageable burden \(right-size to a modular monolith\).
- **Losing bit-identical parity/validation fidelity** — scoring feeds EEOC/Uniform-Guidelines defensibility; "close enough" is a legal problem.
- **Deferring the regression harness** — build golden-case parity checks *early*, not at the end \(Paradigm lesson\).
- **Assuming the retiring developer is available indefinitely** — front-load structured KT.
- **Refreshing the test UI without re-validating scores** — layout can shift outcomes; changes must be A/B tested and parity-checked.
- **Leaving deploys manual** — the legacy publish-by-hand process \(VS publish, zip, IIS off, cert updates, App Pool permissions\) is fragile; it persists early but must give way to CI/CD.

**6 · What they may not be thinking of.**

- The **WCF scoring service may not wrap cleanly** — biggest hidden cost.
- **No client-side SuccessFactors sandbox** concentrates the riskiest integration late in 1B.
- **Transitional scaffolding** \(parallel-run harness, cloud-to-legacy connectivity\) has a *removal* cost, not just a build cost — with no ops team it calcifies if unmanaged.
- **Who operates this in 2027** once Bernie is gone — the recurring support need nobody is pricing.
- **Multi-client parameterization** must be "just enough" — under-do it and it's a future rebuild; gold-plate it and it burns the Phase 1 budget.
- **Honorlock** adds a mandatory candidate-side Chrome extension and contract-gated credentials \(see [[tag-open-questions]] OQ-02\).

**7 · Paradigm — challenges & lessons \(transferable proof\).** Same core business \(clients buy report sets from questionnaire answers\), B2B assessment-creation integrations, many report types. We rewrote the score engine as an **isolated, parameterized module** \(scores varied by client population — like TAG's client-specific config\). Key lessons that map directly to TAG:

- The client provided **known answer sets for each scenario with expected results** → we built tests from them. This is exactly TAG's parallel-run/golden-case parity approach.
- Paradigm was a **phased transition off an old provider** \(what TAG needs\), starting with a full technical discovery: stand up local environments, inventory every credential, map the infrastructure, document how a deployment is actually executed, and catalog all the undocumented "cheat-codes" — **per environment**, since they always differ.
- The **legacy deploy was heavy handwork** \(VS publish, zip, IIS off, cert updates, App Pool permissions\); those pains persist in the initial period, then a robust **CI/CD pipeline** removes them over time.
- **Multilingual \(10+ languages\)** across web *and* PDF — responsive layout for varying text length/direction. Relevant to TAG's Spanish path \(doubles QA surface\).
- **Compliance/security** held up under a real GDPR complaint. *Caveat: Paradigm cannot be confirmed as a nameable reference yet; present anonymized. Specific metrics are not available.*

**Second IP proof — Mossa \(our most restrictive IP engagement\).** A quotation platform whose output price came from a **client-patented algorithm the team was never allowed to see**. We knew only the formula's *shape* \(roughly `A + B·X`, where `X = Zⁿ`\) and placed the secret parameters in a **client-controlled Azure Key Vault** — only the client could read or tune them. Because we couldn't build unit tests against real values, we agreed a **client-led validation phase**: the client fine-tuned the algorithm and reported any deviation from expected results, and we corrected against their findings. Directly relevant to TAG — if they're reluctant to expose scoring internals, we can preserve the IP behind client-controlled secrets and still prove correctness: IP preservation *and* validation without us ever holding the crown jewels.

**8 · Technical advantages vs. other engineering firms.** **.NET / Azure / SQL is one of our strongest stacks**, with senior solution architects \(15+ years\) involved on every project. A psychometric scoring-and-report-engine takeover already in our portfolio \(Paradigm\). Proven scoring-engine isolation and parity discipline. Secret-IP isolation via a client-controlled Azure Key Vault with client-led validation \(Mossa\). Security that survived a regulatory complaint. **HIPAA experience proven** \(GDPR as needed depending on their client base\). Multilingual assessment + responsive PDF. A design system plus UX-that-respects-validity. Proprietary test-automation tooling. And a **prototype already built** — a tangible "we've started" that commodity firms won't match.

**Open decisions for leadership:** \(a\) how much of Paradigm we can name vs. present anonymized; \(b\) whether to share the prototype with the client; \(c\) pricing/margin posture.

[[tag-open-questions]]
