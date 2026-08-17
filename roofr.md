---
type: Note
_width: wide
---
# Roofr

## Statement
The provided sources outline a strategic partnership opportunity with **Roofr**, a manufacturing technology company transitioning its infrastructure from **Heroku to AWS**. This migration is time-sensitive, requiring a **zero-downtime database transfer** before their peak seasonal demand begins in December. Beyond infrastructure, the company seeks to **optimize its AI spending** by routing model usage through Amazon Bedrock to improve cost attribution and governance. While they are eager to integrate advanced AI into their **CRM platform**, they have explicitly rejected any efforts to modernize or replatform their existing **Laravel-based codebase**. The upcoming collaboration involves high-level stakeholders from both organizations and **AWS** to finalize a technical methodology for these transitions. Overall, the deal focuses on providing **specialized engineering support** for cloud migration and disciplined AI implementation.

## Core Application Stack

**Laravel (PHP)**: Roofr's core B2B SaaS platform is built as a monolith using modern **Laravel** design patterns (such as Data Transfer Objects, or DTOs). They have explicitly declined any "PHP modernization" efforts and are highly committed to their current Laravel framework.

**LAMP-stack**: They operate on a traditional **LAMP (Linux, Apache, MySQL, PHP)** architecture, stating they prefer handling "usual LAMP-stack scaling problems" rather than adopting a microservices architecture.

**Roofing CRM Platform**: This is their primary software product where they are currently integrating advanced machine learning and AI capabilities.

**Cloud & Hosting Infrastructure**

**Heroku**: Their current application hosting platform, which they are planning to exit fully in early December before their seasonal customer peak.

**AWS (Amazon Web Services)**: Their target cloud destination. They are partnering with AWS to move their infrastructure and database hosting over from Heroku.

**Data Formats & Database**

**GeoJSON**: They store **GeoJSON geospatial data** within their primary database. Managing a zero-downtime migration of this specific spatial data is highlighted as a primary complication of their Heroku exit.

**AI & Machine Learning Services**

**Anthropic**: They use **Anthropic** models directly, which currently consumes roughly 70% of their engineering AI budget.

**Amazon Bedrock**: They are actively planning to route their AI requests through **Bedrock** to solve their scaling challenges with Anthropic, implement guardrails for non-engineers, and establish clear cost attribution by team and workstream
