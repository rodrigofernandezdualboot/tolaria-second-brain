---
type: Note
belongs_to: "[[wg-henschen]]"
status: Active
_organized: true
---

# WG Henschen — RAG Scope Call Notes

Internal call notes (Granola capture) covering the new scope to be estimated. Source spells the client "WG Henshon" — spelling to confirm.

## Client and prior engagement

WG Henschen supplies aerospace parts and manages a large, complex inventory of components and part numbers.

Prior Dualboot engagement: a tool that processes technical drawings (PDFs) into structured CSV/Excel data.

- Built on AWS Bedrock (Claude + Gemini) specifically for security — drawings contain sensitive and patented data that cannot leave a controlled environment.
- Delivered as a Windows native app wrapping a Python connector to Bedrock.
- ~400 hours committed, ~$80K.

The client attempted to replicate this internally with GPT. Results were inferior, which brought them back to us. That is the reference point for what "good" looks like on the new engagement.

## New engagement — RAG knowledge and inventory tool

NetSuite is the single source of truth. The ask is a chat interface to query inventory, accounting, customer, and other business data.

**Interface.** Chat UI embedded inside NetSuite, likely a React embed.

**Accuracy posture.** Must answer "I don't have that information" rather than hallucinate. Guardrails on both inbound and outbound.

**Architecture direction discussed.** A RAG engine on AWS pulling data from NetSuite into domain-specific agents.

- Proposed business slices: inventory management, accounting, customer data.
- Routing layer directs a query to the correct agent — the user should never have to pick one.
- Permission/access layer resolves user identity and applies token-based filtering *before* the request reaches the LLM.

**Infrastructure.** Client already has AWS configured with Bedrock in place, and is open to continuing on AWS.

## Open questions carried out of the call

- How to extract data from NetSuite. It is an Oracle-native DB and no IT contact has been identified on the client side.
- Which NetSuite modules are in scope beyond inventory.
- Whether output is chat-only or also needs to generate exportable files (CSV, etc.).
- Scope of user roles and the data access permissions that follow from them.

## Next steps

| Action | Owner | Due |
|---|---|---|
| Sketch preliminary architecture; research NetSuite APIs/docs; identify data extraction options and business domain slices to scope agent design | Rodrigo | — |
| Draft clarifying questions for WG Henschen and send to David | Rodrigo | Fri 14 Aug 2026 |
| Unblock client AWS account access — Matías Corvela raised tickets; account was originally created by TDE Syntax | Maxim | Fri 14 Aug 2026 |
| Demo the existing AWS/Bedrock setup for Rodrigo (pending AWS access being restored) | Maxim | after access restored |

No pricing needed yet. The goal for the call next week is a high-level picture plus the open questions.
