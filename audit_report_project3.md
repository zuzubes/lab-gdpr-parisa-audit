# Audit Report: TestVet / Project 3

Auditor - Mudit Airan
Client - Parisa
Project github URL - https://github.com/Parisa-QA/Project-3
Project name - TestVet

> Note: this file contains the audit case framing and the answer key in one place so a reviewer can scan the structure quickly.

## 1. System Summary

TestVet is a QA tool vetting agent with a Gradio front end and a LangGraph backend. The user-facing form collects a tool name, a free-text context, and an optional GitHub repo URL or `owner/repo` slug. The backend then resolves the repository, queries GitHub, News API, and Hacker News via Algolia, loads a local QA guidance corpus, and synthesizes a structured recommendation report.

The team likely sees the system as useful because it saves time when comparing QA tools and makes the recommendation feel safer by grounding it in external evidence rather than a purely generative answer. That does not remove GDPR duties, but it does explain the product intent.

Input, output, human review, and use case are clearer when separated:

- Inputs: tool name, context, optional GitHub repo reference.
- Outputs: score, recommendation, confidence, evidence summary, and detailed criterion notes.
- Human review: the report is produced automatically, but the current design does not show a required human approval gate before output.
- Who uses it: QA leads or automation engineers deciding whether a testing tool is worth adopting.

The interface is not obviously AI-driven from the page itself. A reviewer can tell from the repo and architecture files that the backend is automated and model-like, but an end user seeing the form would likely just see a structured utility, not an obvious AI interaction.

## 2. Data and Role Map

### Personal Data Summary

| Data category | Source | Purpose(s) | Crosses EU border? | Special category? |
|---|---|---|---|---|
| Tool name | User input | Identify the product to evaluate | No clear indication | No |
| Free-text context | User input | Describe technical QA requirements, but may also contain incidental personal data | Likely yes if sent to GitHub / News / HN APIs | No obvious special-category data, but it could be entered incidentally |
| GitHub repo URL or slug | User input | Resolve the repo and fetch evidence | Likely yes | No |
| GitHub repository metadata | GitHub API | Evidence for the recommendation | Likely yes | No |
| News article metadata | News API | Evidence for the recommendation | Likely yes | No |
| Hacker News discussion metadata | Algolia HN API | Evidence for the recommendation | Likely yes | No |
| Generated report and criteria notes | TestVet backend | Internal evaluation output | Depends on hosting and export path | No |

### Role Map

| Entity | Role | Processing activity | DPA needed? |
|---|---|---|---|
| TestVet operator / project team | Controller, and provider if the system is shared outside the team | Defines the purpose of the evaluation, receives user input, and decides which external sources to call | Yes, if any vendor truly acts as a processor on its behalf |
| Builder team | Processor only if working under a separate client’s instructions; otherwise part of the controller | Builds and maintains the workflow | Not by default |
| GitHub | Third-party vendor; likely independent controller for public repo data, but role should be confirmed | Receives repo queries and returns repository metadata | Not clearly a DPA relationship from the brief |
| News API | Third-party vendor; role not confirmed | Receives search terms and returns news metadata | Not clearly a DPA relationship from the brief |
| Hacker News via Algolia | Third-party vendor; role not confirmed | Receives search terms and returns discussion metadata | Not clearly a DPA relationship from the brief |

The provider/deployer distinction matters here. If the team hands this system to a client company, the team is the provider of the tool, while the client company becomes the deployer and controller for its own use. That split is not documented in the brief, so responsibility boundaries are still fuzzy.

No special-category data is intentionally processed from the documented flow. The only realistic route to special-category data is accidental entry in the free-text context field, which is a documentation and minimisation issue rather than an intended feature.

## 3. Clarifying Questions Log

- Do users ever type personal data into the context field, or is the field constrained to product requirements only? This matters for lawful basis, minimisation, and retention. Provisional assumption: incidental personal data may be entered unless the UI blocks it.
- Do any of the external APIs store query strings or logs, and where are those logs hosted? This matters for international transfers, disclosure, and vendor role analysis. Provisional assumption: the query strings leave the EU and may be retained by the vendor according to their own terms.
- Is the generated report ever used to make decisions about named people, vendors, or employees outside the chat? This matters for whether Article 22 or a DPIA-level review becomes relevant. Provisional assumption: the output is advisory only unless a downstream decision process is documented.

### Clarification responses

- The context field is intended only for technical QA requirements, but the free-text design means incidental personal data can still be entered unless additional UI and technical controls are added.
- TestVet sends search terms derived from the tool name and context field to GitHub, News API, and Hacker News via Algolia. TestVet itself does not store these queries in a database, but the external providers may retain request metadata, timestamps, IP addresses, and logs.
- The report is advisory only. It supports decisions about QA tools and vendors, but it does not make decisions about named individuals or produce legal or similarly significant effects on people, so Article 22 is not currently engaged.

## 4. Compliance Findings

### Finding 1 - Article 50 transparency for direct AI interaction

**Severity:** Significant

**Description:** The user-facing interface does not make the AI nature of the workflow obvious from context. The backend is clearly automated and AI-assisted from the repo structure, but the front end reads like a conventional form and reporting tool. Article 50 is sharper when the AI nature is not obvious to the person interacting with it, so the platform-level label alone is not enough.

**Recommended action:** Add a visible disclosure at the point of interaction, not just in the code or architecture notes. A short banner or pre-submit notice should say that the report is generated by an AI-assisted evaluation workflow and may contain automated synthesis.

**Escalation needed?** Yes - product owner and legal/DPO review before launch.

**Reviewer note:** I put this in Significant rather than Blocking because the core workflow can still run, but the user-facing disclosure gap is real and easy to fix.

### Finding 2 - International transfer mechanism and vendor role ambiguity

**Severity:** Significant

**Description:** The workflow sends search terms derived from the tool name and context field to GitHub, News API, and Hacker News via Algolia. TestVet does not store these queries in its own database, but the vendors may retain API requests, IP addresses, timestamps, usage data, and logs. The brief still does not show any transfer mechanism, data residency decision, SCCs, or completed vendor-role analysis. That is a material gap because the data likely leaves the EU and the legal basis for the transfer is undocumented.

**Recommended action:** Produce a vendor map with each service classified as controller, processor, or independent controller. Document the transfer mechanism for each service, keep query strings as short as possible, and confirm the hosting region or contractual safeguard before launch.

**Escalation needed?** Yes - legal/DPO and procurement or vendor management.

**Reviewer note:** I marked this Significant because the application can continue to exist, but cross-border vendor use without a documented transfer story is too material to ignore.

### Finding 3 - Lawful basis, purpose limitation, and minimisation for free-text input

**Severity:** Minor

**Description:** The system is designed for QA-tool evaluation, not for profiling people. Even so, the free-text context field can absorb personal data if a user types names, emails, employee details, or other identifiers, and the UI does not technically prevent that. The brief confirms that this data is not required for the tool’s purpose, so users should be told not to enter it and the system should avoid sending the full context externally when it is not necessary. The brief does not document a lawful basis, a retention rule, or a warning telling users not to include unnecessary personal data.

**Recommended action:** Document the lawful basis for any incidental personal data, add a short prompt telling users not to paste personal data unless it is necessary, and define a retention rule for prompts, logs, and generated reports.

**Escalation needed?** Yes - product owner and DPO.

**Reviewer note:** I kept this Minor because the system does not look like it is intentionally collecting personal data at scale, and there is no sign of special-category processing or Article 22 decision-making.

## 5. Specific GDPR Obligations Checklist

| Obligation | Assessment | Note |
|---|---|---|
| Lawful basis identified for each processing purpose | Gap identified | No documented basis for incidental personal data in the context field or query logs |
| Purpose limitation respected (no incompatible reuse) | Gap identified | The brief does not say whether user inputs or fetched metadata are reused outside QA evaluation |
| Data minimisation (only necessary data collected) | Appears met | Inputs are narrow for the stated purpose, but the free-text field should be constrained or warned against unnecessary personal data |
| Controller/processor roles mapped and DPAs in place | Gap identified | Vendor status is unclear and no DPA / controller analysis is documented |
| International transfer mechanism documented | Gap identified | GitHub, News API, and Algolia queries likely cross borders, but no transfer mechanism is shown |
| DPIA conducted if required | Appears met | No obvious DPIA trigger is visible from the brief: no special-category data, no systematic monitoring of people, and no high-risk profiling of natural persons |
| Article 22 safeguard in place if automated decisions affect people | Appears met | The tool produces advisory recommendations about QA tools and vendors, not decisions about individuals |
| Privacy notice covers AI processing | Gap identified | The user-facing disclosure is not obvious from the interface |
| Data subject rights can be operationalised within deadlines | Cannot determine from brief | No retention, contact point, or workflow for access/erasure requests is documented |

## 6. Overall Recommendation

**Proceed with conditions**

The current brief does not show a blocking GDPR issue, but it does leave two material gaps: the AI disclosure is not obvious in the user interface, and the transfer story for external API calls is undocumented. The data footprint is otherwise limited, the system does not appear to make decisions about people, and no special-category processing is evident. I would allow a limited launch only after the disclosure text, vendor-role map, and transfer mechanism are written down, plus a warning not to enter unnecessary personal data in the context field.

## 7. What This Report Is Not

This report is not a legal opinion, not a DPIA, and not a certification of compliance. It is an audit based on the project brief and the codebase visible in this repository. The team should still obtain legal review before relying on it for a production launch.

## 8. Stretch Remediation Plan

**Most significant gap chosen:** Article 50 transparency

**Artifact to create:** A short in-product disclosure and a one-paragraph AI notice for the README or privacy page.

**Owner:** Product owner, with legal or DPO review.

**Timeline:** 1 sprint, or sooner if the tool is already in pre-launch review.

**What the fix should say:** The interface should clearly state that the report is generated by an AI-assisted workflow and that the output is advisory, not a human-authored assessment.

**Evidence that the gap is closed:** A screenshot of the notice in the UI, the final disclosure copy approved by the owner, and a short implementation note showing where the message is rendered before submission.

**Operational priority:** Disclosure first, transfer documentation second, then any broader rollout after review.

## 9. Phase 5: Debrief Conversation

1. **Auditor presents** - The report treats TestVet as a QA tool evaluator, not a people-profiling system. The main gaps are the unclear AI disclosure, the missing transfer story for external APIs, and the need to warn users not to paste unnecessary personal data.
2. **Builder responds** - The intent is technical QA only. The free-text field is convenient, but I agree it should be constrained or clearly warned, and the vendor logging story should be documented before launch.
3. **Compare lawful basis selections** - I would use legitimate interests for the primary QA-evaluation purpose, with minimisation and user guidance for incidental personal data. That is stronger than relying on consent for a workflow users may need to complete.
4. **Compare DPIA conclusions** - We both land on no DPIA for the current version. The system is advisory, does not target named people, and does not do systematic monitoring or high-risk profiling.
5. **Compare gap lists** - The external audit caught the UI transparency gap more sharply. The self-audit was closer on intent and vendor handling, but it underplayed how easily the free-text field can drift into personal data.
6. **Joint closing note** - Self-assessment misses the gaps that feel obvious to the builder because they sit inside the intended use case. External review is better at catching interface disclosures, transfer documentation gaps, and places where a flexible field quietly expands the data footprint.
