# Solidus Labs

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Solidus Labs is a crypto-native market-integrity and compliance technology company founded in 2018,
headquartered in New York with offices in Tel Aviv and London. Its **HALO** platform is a unified
compliance control hub spanning trade surveillance, transaction monitoring, stablecoin monitoring,
token monitoring, execution quality, case management and agentic-AI workflows, used by regulated
exchanges, broker-dealers, market makers, custodians, OTC desks, stablecoin issuers and regulators.

- Website: https://www.soliduslabs.com/
- Developer surface: https://www.soliduslabs.com/tokensniffer/api
- API reference: https://tokensniffer.readme.io/

## What is public

The publicly documented developer surface is the **TokenSniffer API**, the programmatic face of the
HALO Token Monitoring module — smart-contract scam detection, the TokenSniffer "Smell Test" 0-100
risk score, malicious token / pair / deployer-address feeds across 15 chains, and a v3 webhook
subscription API. The HALO platform itself is delivered to contracted customers through regional
dashboards and is not publicly documented.

| Artifact | What's here |
|---|---|
| `openapi/` | 5 OpenAPI 3.1.0 documents, 15 operations — assembled verbatim from the per-operation OpenAPI blocks Solidus Labs publishes on `tokensniffer.readme.io` |
| `skills/` | The provider's **own** Agent Skill (`SKILL.md` + 6 references + evals), harvested verbatim from `github.com/SolidusLabsExternal/tokensniffer-skills` |
| `llms/` | Both published `llms.txt` files — `soliduslabs.com` and `tokensniffer.readme.io` |
| `asyncapi/` | Webhook event catalog (no AsyncAPI document is published) |
| `vocabulary/` | The exploit typology, chain registry and event-type vocabularies |
| `packages/`, `cli/`, `components/` | npm widget library, the Python CLI (git-only, not on PyPI) |
| `plans/`, `rate-limits/`, `conventions/`, `errors/`, `lifecycle/`, `changelog/` | Pricing tiers, the 5 req/s limit, auth and pagination semantics, the status-code table, versioning |
| `security/`, `well-known/`, `conformance/` | Probe results — including what is **absent**: no security.txt, no status page, no trust center, no agent card |

Nothing in this repository was authored on the provider's behalf. Where a probe missed, the miss is
recorded with the HTTP status it actually returned.
