---
type: course-note
date: 2026-10-09
tags:
  - course
  - ppia
  - privacy
course: "[[Privacy Preserving Information Access]]"
sources:
  - "[[Macrodata and Microdata]]"
  - "[[Intro]]"
  - "(see -> maps/NotebookLM Mind Map.png)"
ai-first: true
---

## For Future Agent

Study note for the [[Privacy Preserving Information Access]] course covering the foundational concepts of data, security, and privacy. Created 2026-10-09 by translating and merging the fundamentals sections of [[Macrodata and Microdata]] and [[Intro]]. Key points: attribute taxonomy (identifiers / quasi-identifiers / confidential / non-confidential), security vs. privacy, the legal frameworks (HIPAA, GDPR, EDPB), and privacy as an evolutionary and architectural concern.

## 1. Data: Definition and Analytical Vectors

Data is information - specifically facts or numbers - collected to be examined, considered, and used to support decision-making. Individual data points gain meaningful context only when evaluated collectively.

Anonymization strategies evaluate data through three analytical vectors:

- **Record Isolation:** preventing single records from being distinctly isolated.
- **Inference Control:** managing authorized versus unauthorized inferences.
- **Linkage:** preventing unauthorized linkage between disparate datasets.

## 2. Attribute Taxonomy

Attributes within a dataset are classified into four structural types:

| Attribute Type | Definition | Examples |
|---|---|---|
| **Identifiers** | Allow direct or (almost) univocal identification of a person or entity. | Name, SSN/tax code, student ID, fingerprints, email address, phone number. |
| **Quasi-identifiers** | Do not uniquely identify an individual in isolation, but can re-identify when combined. | Postal/zip code, gender, Date of Birth (DoB), workplace name. |
| **Confidential Attributes** | Pertain to the private or personal sphere of the respondent. | Medical diagnoses/diseases, income, financial balance. |
| **Non-confidential Attributes** | Currently cannot be used to disclose identity or sensitive information. | General non-sensitive observations (subject to change over time). |

This taxonomy is the backbone of the whole course: quasi-identifiers drive re-identification risk (see [[Microdata Disclosure Risks and Sanitization]]), confidential attributes are what protection techniques try to hide, and identifiers are removed first during sanitization.

## 3. Security vs. Privacy Protection

- **Data Security:** protecting physical or digital data from unauthorized access, theft, or corruption. Security manages data *access* and serves as a **prerequisite** for privacy.
- **Privacy Protection:** protecting the underlying information contained within or inferable from the data. Privacy concerns arise when data (or a subset thereof) is released, shared, or processed to extract value.

Security keeps attackers out; privacy survives even once data is legitimately in someone's hands.

## 4. Privacy: Definition and Legal Framework

The *Cambridge Dictionary* defines privacy as the right of a person to keep their personal matters and relationships secret. Three constitutive elements follow:

1. **Juridical nature** - privacy is a *right*, not a preference.
2. **Personal sphere** - it pertains to individual life.
3. **Secrecy** - it guarantees protection of that sphere.

Protection against arbitrary interference is solemnly enshrined in **Article 12 of the Universal Declaration of Human Rights**.

### Normative frameworks and sanctions

At international and EU level, confidentiality is regulated by:

- **HIPAA** (US health data)
- **GDPR** (EU data protection)
- **EDPB** (European Data Protection Board)

GDPR violations carry administrative fines of up to **EUR 20 million or 4% of the organization's global annual turnover**, whichever is higher. This turns privacy from a moral imperative into a hard architectural and legal constraint for engineers.

## 5. Privacy as an Evolutionary and Psychological Need

Why does privacy feel innate? (reflection prompts):

- **Evolutionary roots:** ancestrally, avoiding the view of predators; today, avoiding social stigma attached to lawful-but-private behaviours.
- **Design failures erode trust:** lack of software transparency or invasive ad algorithms produce the feeling of being watched ("*my phone listens to me*"), which translates into loss of credibility of the service.
- **Efficiency vs. ethics conflict:** the pursuit of the *top performing algorithm* collides with the ethical-legal duty to preserve confidentiality; severe GDPR sanctions redefine the engineer's responsibility (open question).

## 6. Glossary (this note's terms)

| Term | Definition |
|---|---|
| **Privacy** | Fundamental right of the individual to keep personal matters and relationships secret; rooted in evolutionary/psychological needs; protected by Art. 12 UDHR. |
| **GDPR** | EU framework on personal data protection; violations fined up to EUR 20M or 4% of global annual turnover. |
| **Security vs. Privacy** | Security protects access to data; privacy protects the information inside/inferable from data once accessed. |

## Related

- [[Database Foundations and Architecture]] - architecture/database terms live there
- [[SQL Injection and Defenses]] - SQLi terms live there
- [[Microdata Disclosure Risks and Sanitization]] - where the privacy need turns into concrete threats
- [[Macrodata Protection]] - how the privacy requirement is enforced on real releases

## Sources

- [[Macrodata and Microdata]] (AY 2026/2027)
- [[Intro]] (AY 2026/2027)
- (see -> maps/NotebookLM Mind Map.png)
