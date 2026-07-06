---
type: Note
---
# Paradigm

Here are all the challenges we faced at Paradigm:

## Business context

- Their core business was very similar: clients buy a set of reports that are built based on answers to demographic and bias questionnaires.
- There were many types of reports, some focused on personality, others on leadership, others on team building.

## Legacy modernization

- We took a legacy system built on .NET Core 2.0 and we updated it to its latest version.
- The web-application was built using Angular JS; we updated it to the latest version.

## Features we built

- We created and implemented a shopping cart where end-clients can create customized report packages by their own.
- We rewrote the score engine to be an isolated module that, given a set of answers of a test, produced a result (no UI).
- We implemented a serverless and async process to render reports in the client language.

## Architecture & scale

- A big challenge was that reports must be delivered all at once, so the architecture we designed with queues has to support hundreds of reports being rendered at the same time to be zipped and sent to the end-client.
- The system supported more than 10 languages, not only the web but also the reports. This came with the challenge of building a responsive design for the PDF to support different lengths of text.

## Compliance & security

- Because their clients were all over the world, we had to be compliant with GDPR and HIPAA.
- Once, somebody filed a complaint against GDPR, and because of the tracing system we built, plus the implemented constraints to the PII (encryption in transit and at rest) plus the rotation of keys, it was easily proven there was no leak of any kind of information.

## Data & experimentation

- Another important challenge was how data was stored and retained. We used Azure SQL because of its point-in-time restore capabilities and backup strategies, where we backed up incremental backups (daily, weekly, monthly) with different retention policies.
- We also learned how questions are presented to the client made a significant difference to their score, because the layout we built was less distracting than the previous one. This was implemented using A/B testing.
