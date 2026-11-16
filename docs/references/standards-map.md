# Standards Map - AI SDLC Skills

Date: 2026-08-01

Ce document cartographie les standards, frameworks et bonnes pratiques à utiliser pour peupler la suite `agentic-coding-suite`. Il ne remplace pas les sources originales. Il sert de boussole pour créer des skills originaux, cohérents et publiables.

## Principes de sourcing

| Type de source | Usage dans la suite | Règle |
|---|---|---|
| Standard normatif | Définir les exigences, vocabulaires, contrôles et checklists | Citer, paraphraser, ne pas copier |
| Framework reconnu | Structurer la méthode et les livrables | Adapter au format skill et au mode Lean/Standard/Enterprise |
| Skill public existant | Benchmark, inspiration, éventuellement dépendance recommandée | Vérifier licence, provenance, install count, contenu, scripts et risques |
| Pratique marché | Compléter les standards trop abstraits | Transformer en procédure vérifiable |
| Création originale | Relier les sources en workflow repo-first | Préserver une interface de skills stable |

## Références transverses

| Référence | Rôle | Skills concernés |
|---|---|---|
| ISO/IEC/IEEE 12207 - Software life cycle processes | Cadre complet de cycle de vie logiciel | Tous, surtout `delivery-orchestrator`, `` |
| ISO/IEC/IEEE 29148 - Requirements engineering | Exigences, attributs de qualité des exigences, information items | `prd-requirements`, `bdd-specifications`, `test-strategy` |
| ISO/IEC/IEEE 42010 - Architecture description | Description d'architecture, viewpoints, concerns, stakeholders | `architecture-rfc`, `application-architecture` |
| ISO/IEC 25010:2023 - Product quality model | Modèle de qualité produit: sécurité, fiabilité, maintenabilité, performance, utilisabilité, etc. | `test-strategy`, `code-review`, `` |
| DORA metrics | Mesure delivery: throughput, instability, amélioration continue | `delivery-orchestrator`, `ci-cd-platform`, `observability-reliability` |

Sources:

- ISO/IEC/IEEE 12207: https://standards.ieee.org/ieee/12207/11416/
- ISO/IEC/IEEE 29148: https://www.iso.org/standard/72089.html
- ISO/IEC/IEEE 42010: https://www.iso.org/standard/74393.html
- ISO/IEC 25010:2023: https://www.iso.org/standard/78176.html
- DORA metrics: https://dora.dev/guides/dora-metrics/

## Produit et discovery

| Référence | Ce qu'on reprend | Skills concernés |
|---|---|---|
| PRD agile Atlassian | PRD concis, collaboratif, vivant, orienté objectifs, hypothèses, user stories, design, questions ouvertes, hors périmètre | `prd-requirements`, `idea-framing` |
| Shape Up | Shaping, appetite, pitch, risques, no-gos, fixed time variable scope | `idea-framing`, `delivery-roadmap-backlog`, `implementation-planning` |
| Dual-track discovery/delivery | Discovery continue séparée de l'exécution, mais connectée aux décisions | `market-user-discovery`, `delivery-orchestrator` |

Sources:

- Atlassian PRD: https://www.atlassian.com/agile/product-management/requirements/
- Shape Up pitch: https://basecamp.com/shapeup/1.5-chapter-06
- Shape Up shaping principles: https://basecamp.com/shapeup/1.1-chapter-02

## Domain modeling, bounded contexts et architecture

| Référence | Ce qu'on reprend | Skills concernés |
|---|---|---|
| Domain-Driven Design | Ubiquitous language, domain model, strategic design, complex domain logic | `domain-modeling` |
| Bounded Context | Découpage du domaine, cohérence du modèle, relations entre contextes | `bounded-context-map` |
| C4 model | Diagrams system context, containers, components, code; diagrams as communication and onboarding | `architecture-rfc`, `application-architecture`, `documentation` |
| ADR | Context, decision, consequences, alternatives, lifecycle des décisions | `architecture-rfc`, `delivery-retrospective` |
| Architecture styles | Modular monolith, hexagonal, clean, microservices, event-driven, CQRS, event sourcing, serverless | `architecture-rfc`, `application-architecture` |

Sources:

- DDD overview: https://martinfowler.com/bliki/DomainDrivenDesign.html
- Bounded Context: https://martinfowler.com/bliki/BoundedContext.html
- C4 model: https://c4model.com/
- AWS ADR guidance: https://docs.aws.amazon.com/prescriptive-guidance/latest/architectural-decision-records/adr-process.html
- Google Cloud ADR overview: https://docs.cloud.google.com/architecture/architecture-decision-records

## BDD comportemental

| Référence | Ce qu'on reprend | Skills concernés |
|---|---|---|
| Cucumber BDD | Discovery, formulation, automation; BDD comme collaboration par exemples, pas seulement tests | `bdd-specifications`, `test-strategy` |
| Gherkin | Given/When/Then lisible par humains et machines | `bdd-specifications` |
| Example mapping | Rules, examples, questions, edge cases | `bdd-specifications`, `prd-requirements` |

Source:

- Cucumber BDD: https://cucumber.io/docs/bdd/

## UX, UI et accessibilité

| Référence | Ce qu'on reprend | Skills concernés |
|---|---|---|
| WCAG 2.2 | Accessibilité web, niveaux A/AA/AAA, exigences de conformité | `ux-product-design`, `ui-design-system`, `test-strategy` |
| Nielsen Norman Group | Journey mapping, usability, heuristiques UX | `ux-product-design` |
| Design systems | Tokens, composants, états UI, responsive, conventions d'interaction | `ui-design-system` |

Sources:

- WCAG 2.2: https://www.w3.org/TR/WCAG22/
- NN/g journey mapping: https://www.nngroup.com/articles/journey-mapping-101/

## API, events et intégrations

| Référence | Ce qu'on reprend | Skills concernés |
|---|---|---|
| OpenAPI | Contrats REST/HTTP machine-readable, schémas, endpoints, erreurs, sécurité | `api-versioning-deprecation` |
| AsyncAPI | Contrats event-driven/message-driven, channels, messages, operations, bindings | `api-versioning-deprecation`, `application-architecture` |
| Contract testing | Backward compatibility, consumer/provider contracts | `test-strategy`, `api-versioning-deprecation` |

Sources:

- OpenAPI Specification: https://spec.openapis.org/oas/latest.html
- AsyncAPI Specification: https://www.asyncapi.com/docs/reference/specification/latest

## Sécurité, privacy, compliance et supply chain

| Référence | Ce qu'on reprend | Skills concernés |
|---|---|---|
| NIST SSDF SP 800-218 | Secure software development practices intégrables dans tout SDLC | `security-privacy-compliance`, `code-review`, `ci-cd-platform` |
| OWASP SAMM | Maturité AppSec, roadmap d'amélioration, approche risk-driven | `security-privacy-compliance`, `` |
| OWASP ASVS | Exigences de vérification sécurité applicative | `security-privacy-compliance`, `test-strategy` |
| OWASP Cheat Sheets | Bonnes pratiques concrètes par risque | `code-review`, `security-privacy-compliance` |
| SLSA | Supply chain, provenance, niveaux de confiance build/source | `third-party-ip-risk`, `ci-cd-platform` |
| OWASP Top 10 / LLM Top 10 | Risques web et risques spécifiques LLM/agents | `security-privacy-compliance`, `ai-product-architecture` |

Sources:

- NIST SSDF: https://csrc.nist.gov/pubs/sp/800/218/final
- OWASP SAMM: https://owasp.org/www-project-samm/
- OWASP ASVS: https://github.com/OWASP/ASVS
- OWASP Cheat Sheets: https://cheatsheetseries.owasp.org/
- SLSA: https://slsa.dev/spec/v1.2/
- OWASP LLM Top 10: https://genai.owasp.org/resource/owasp-top-10-for-llm-applications-2025/

## DevOps, release, observabilité et reliability

| Référence | Ce qu'on reprend | Skills concernés |
|---|---|---|
| DORA metrics | Change lead time, deployment frequency, failed deployment recovery time, change fail rate, deployment rework rate | `ci-cd-platform`, `observability-reliability`, `` |
| OpenTelemetry | Traces, metrics, logs, instrumentation vendor-neutral | `observability-reliability` |
| Google SRE | SLI, SLO, error budget, alerting utile, production best practices | `observability-reliability`, `definition-of-done` |
| Twelve-Factor App | Config, stateless processes, backing services, dev/prod parity | `ci-cd-platform`, `application-architecture` |

Sources:

- DORA metrics: https://dora.dev/guides/dora-metrics/
- OpenTelemetry docs: https://opentelemetry.io/docs/
- Google SRE SLOs: https://sre.google/sre-book/service-level-objectives/
- Twelve-Factor App: https://12factor.net/

## Performance, scalabilité et coûts

| Référence | Ce qu'on reprend | Skills concernés |
|---|---|---|
| FinOps Framework | Business value drives technology decisions, ownership des coûts, data accessible, collaboration finance/tech/product | `scalability-cost` |
| SRE capacity/reliability practices | Latence, disponibilité, saturation, error budgets | `observability-reliability`, `scalability-cost` |
| Cloud provider architecture centers | Patterns de résilience, coût, scalabilité, IaC, migration | `architecture-rfc`, `ci-cd-platform` |

Source:

- FinOps Framework: https://www.finops.org/framework/

## IA, LLM et agents

| Référence | Ce qu'on reprend | Skills concernés |
|---|---|---|
| NIST AI RMF | Trustworthiness, risk management, governance, measurement | `ai-product-architecture`, `ai-coding-governance` |
| NIST Generative AI Profile | Risques spécifiques genAI, test/evaluation, gouvernance | `ai-product-architecture` |
| OWASP LLM Top 10 | Prompt injection, data leakage, unsafe tool use, supply chain LLM | `ai-product-architecture`, `security-privacy-compliance` |
| Agent Skills open ecosystem | Format SKILL.md, progressive disclosure, distribution multi-agent | Tous les skills, surtout packaging |

Sources:

- NIST AI RMF: https://www.nist.gov/itl/ai-risk-management-framework
- NIST AI RMF 1.0: https://www.nist.gov/publications/artificial-intelligence-risk-management-framework-ai-rmf-10
- OWASP LLM Top 10: https://genai.owasp.org/resource/owasp-top-10-for-llm-applications-2025/
- OpenAI Skills catalog: https://github.com/openai/skills
- GitHub Copilot Agent Skills docs: https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills
- skills.sh docs: https://www.skills.sh/docs

