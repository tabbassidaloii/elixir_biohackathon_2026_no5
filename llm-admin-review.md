### Assigning terms to answers

Respondents answer in free text, so each answer has to be turned into registry and ontology terms before it can be compared, reused or checked. The review service does this in three steps.

**1. LLM proposal.** The service sends each answer, together with its question and the resource classes that question expects, to a large language model. It calls the model through an OpenAI-compatible API at temperature 0 and asks for JSON output. The model returns the concrete resources the answer names (software, databases, standards, file formats, roles and EDAM topics). For each one it gives:

- the resource class;
- the name;
- an identifier, if one is known;
- a one-line reason;
- a confidence score.

The prompt tells the model to report only things the answer mentions, never to invent an identifier, and to return an empty list when the answer names nothing. Candidates whose class the question does not accept are dropped. When no model is configured, a rule-based extractor proposes the candidates instead, and each one records which method produced it.

**2. Registry verification.** The model's output is treated as a proposal, never as a mapping. Each candidate is looked up in a registry:

- bio.tools for software;
- OLS4 for EDAM terms;
- FAIRsharing, through DataCite, for standards;
- the RDMkit role pages for roles.

A registry match pins the candidate to a resolvable identifier. The registry name must also agree with the name in the answer. If it does not, the identifier is dropped and the disagreement is shown, so a look-alike record is not mapped with confidence. Confidence comes from the registry check, not from the model; the model's own score is kept only when no registry could check the candidate.

**3. Review.** Each stored proposal keeps the excerpt of the answer it came from, the model's reason and the verification result. A reviewer can therefore see why a term was proposed and correct the term before approval. Approved terms feed the curated set and the dropdown options offered to later respondents.

### Checking that answers fit their question

A verified term can still answer the wrong question. For example, a file format can end up under analysis operations, or a software tool where a community standard is expected. The respondent may have typed the answer into the wrong box, or the model may have extracted a term from a passage about another topic. To detect such cases, each question can carry an expectation profile. The profile lists the resource classes the question accepts. It also lists the EDAM branches an answer should fall under, given as root terms, and branches that are known wrong answers for that question.

Each proposed item is checked in two ways. The class check compares the kind of item with the accepted classes. The region check retrieves the term's ancestors from the OLS4 `hierarchicalAncestors` endpoint and tests whether the term is, or descends from, an expected or excluded root. Each item receives one of four verdicts: `fit`, `check` (needs review), `mismatch`, or `unassessable` when no term is attached. These checks are deterministic and use no LLM, so a model error cannot hide itself. The check uses only IRIs and a subclass hierarchy, so other ontologies in OLS4 can be added as further regions: SPDX for licences, DUO for usage conditions, or a role ontology. At present only EDAM is used. Five of the 41 questions have an explicit profile, and the rest use a profile derived from their mapped EDAM terms and resource types.

### Admin review

The verdicts are shown on the admin page next to the model's reason and the answer excerpt. Submissions pass from respondent upload through researcher confirmation to admin approval and publication. Each flagged item shows:

- the verdict;
- the reason, for example that the term is a format and the question excludes formats;
- a plain statement of what the question expects.

Flagged items are listed first, and the submission list shows the number of `mismatch` and `check` items per submission. When an item fits another question, the page suggests that question. Among the questions whose region contains the term, it prefers the one with the most specific matching root, then a question in the same lifecycle stage.

With one action each, the admin can:

- move the item to the suggested question;
- keep it in place, which can be undone;
- replace the proposed term;
- re-run the registry lookups for items that could not be verified, for instance during a bio.tools outage. Re-verification only upgrades a result and never removes a confirmation.

Moves are recorded in the admin note and carried into the published, curated set. The LLM proposes terms and the registries and ontology check them, but a person makes every final decision.

### Calibration

The fit check was calibrated against the curated practices:

- **Positives:** 203 items, each under its own question.
- **Negatives:** 609 cases made by moving those same items to a random question in another lifecycle stage.

| What counts as a flag | Precision | Recall |
|---|---|---|
| `mismatch` only | 0.91 | 0.10 |
| `mismatch` or `check` | 0.86 | 0.28 |

So a flag is usually correct, but most misplaced items pass, for two reasons:

- **Tools pass anywhere.** A tool fits any question that accepts tools.
- **Few answers carry ontology terms.** Only 25 of 235 practices have EDAM terms directly. Another 37 name a tool registered in bio.tools, whose record could supply such terms.

Calibration also found profiles narrower than real answers. Question 4.1, for example, excludes tools, yet the curated practices list tools there. In such cases the question or its profile needs revising, not the answer.

### Still to be done

- a verdict for the whole answer;
- similarity scores for nearby terms;
- mappings between ontologies;
- an overview grid of questions × communities;
- warnings in the submission form that do not block it.
