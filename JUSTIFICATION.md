# JUSTIFICATION — Ethical and Legal Design

## 1. Purpose and Permitted Use

The AI system is designed to assist the fictional Tulpar Scholarship Fund with the initial screening of scholarship applications.

The system may:
- Read valid applicant information.
- Evaluate academic information using the supplied model.
- Produce a preliminary shortlist or rejection recommendation.
- Assign a suitability score.
- Identify uncertain applications for human review.
- Record decisions and the controls applied to them.

The AI system must not make final scholarship decisions independently. A human reviewer must retain responsibility for the final outcome.

## 2. Refusal Boundaries

The system must refuse requests in the following situations:

- The requested purpose is not permitted, including ranking applicants by ethnicity.
- The applicant is under 18.
- Consent to automated processing is missing.
- Input data contains invalid values.
- The input contains an instruction attempting to manipulate the model.

The custom refusal rule detects suspicious instructions such as "ignore all previous instructions", "system prompt" and "reveal instructions".

A refusal prevents the model from processing the request. The reason is recorded for later auditing.

## 3. Input Validation and Data Minimization

Input validation checks whether the applicant information has valid types and values.

The accepted ranges include:
- GPA: 0.0–4.0.
- Entrance examination score: 0–140.
- Household income bracket: 1–5.
- Achievements: 0–5.
- Age: 16–40.

Invalid information is refused rather than passed to the model.

The system also hides the applicant's name and university from the model. These attributes should not determine scholarship eligibility.

This decision follows the principles of fairness, non-discrimination and data minimization.

The experiment found that the model produced 4 different outcomes among 40 gender-coded name pairs, 2 among 40 name-language pairs and 1 among 40 university pairs. However, these results did not establish effects above the corresponding noise floors. Hiding these fields is a preventive design decision rather than proof of confirmed discrimination.

## 4. Human Review and Escalation

Human review is required when:
- The model's score is between 45 and 60, inclusive.
- The model produces an invalid or unparseable response.
- The academic indicators conflict according to the additional escalation rule.

The additional rule detects:
- GPA at least 3.7 with an examination score no higher than 70.
- GPA no higher than 2.3 with an examination score at least 125.

These cases are sent to a human instead of being automatically accepted or rejected.

In the experiment, 86 of 400 applicants (21.5%) were escalated.

Human oversight is important because the model cannot fully understand an applicant's circumstances or guarantee a correct assessment.

## 5. Accountability and Decision Logging

The system uses `decisions.jsonl` to keep a structured record of decisions.

Each record includes:

- Record ID and applicant ID.
- SHA-256 hash of the model-visible input.
- Fields shown to the model.
- Fields withheld from the model.
- Model ID and prompt version.
- Active harness configuration.
- Raw model output.
- Decision and score.
- Controls that were activated.
- Final outcome and reason code.
- Whether the application was escalated to a human.
- Timestamp.

The log makes it possible to investigate why a decision was produced and which controls were applied.

The input hash supports integrity checking of the recorded model-visible text, but it does not reconstruct the original application. Access to the log must be restricted because it can still contain sensitive operational information.

## 6. Ethical and Legal Foundations

The design follows the following ethical principles and legal instruments.

### Fairness and Non-Discrimination

The principle of fairness requires that applicants should not receive different treatment because of irrelevant personal characteristics.

The UNESCO Recommendation on the Ethics of Artificial Intelligence (2021), paragraph 28, establishes fairness and non-discrimination as core principles. Paragraph 29 calls for efforts to prevent discriminatory or biased AI outcomes.

The Republic of Kazakhstan's Law on Artificial Intelligence, Article 6, establishes fairness and equality as requirements for AI systems.

Application: the system hides names and universities from the model to reduce the possibility of irrelevant attributes influencing its recommendations.

Official sources:
- UNESCO Recommendation, paragraphs 28–29: https://www.unesco.org/en/legal-affairs/recommendation-ethics-artificial-intelligence
- Law of the Republic of Kazakhstan on Artificial Intelligence, Article 6: https://old.adilet.zan.kz/rus/docs/Z2500000230

### Privacy and Data Protection

The UNESCO Recommendation, paragraphs 32–34, addresses privacy and data protection throughout the AI lifecycle.

The Law of the Republic of Kazakhstan on Personal Data and their Protection, Article 7, establishes consent requirements for the collection and processing of personal data, subject to the exceptions provided by law.

The Law on Artificial Intelligence, Article 10, also addresses data protection and confidentiality.

Application: the system validates consent and minimizes the information shown to the model.

Official sources:
- Law on Personal Data and their Protection, Article 7: https://old.adilet.zan.kz/rus/docs/Z1300000094
- Law on Artificial Intelligence, Article 10: https://old.adilet.zan.kz/rus/docs/Z2500000230
- UNESCO Recommendation, paragraphs 32–34: https://www.unesco.org/en/legal-affairs/recommendation-ethics-artificial-intelligence

### Human Oversight

The UNESCO Recommendation, paragraphs 35–36, states that human responsibility must remain possible and that AI cannot replace ultimate human accountability.

The Law on Artificial Intelligence, Article 8, establishes the responsibility of AI owners, holders and users to maintain control over AI systems.

Article 7(2) also recognizes the right of a person affected by an AI-assisted decision to be informed about automated processing and its consequences and to raise an objection.

Application: uncertain and conflicting applications are sent to a human reviewer.

Official sources:
- UNESCO Recommendation, paragraphs 35–36: https://www.unesco.org/en/legal-affairs/recommendation-ethics-artificial-intelligence
- Law on Artificial Intelligence, Articles 7–8: https://old.adilet.zan.kz/rus/docs/Z2500000230

### Accountability and Transparency

The UNESCO Recommendation, paragraphs 42–43, establishes responsibility, oversight, auditing and traceability as important requirements.

The Law on Artificial Intelligence, Article 8, establishes accountability and control. Article 22 addresses machine-readable forms that support transparency and accountability.

Application: every screening operation is recorded in a structured JSONL decision log.

Official sources:
- UNESCO Recommendation, paragraphs 42–43: https://www.unesco.org/en/legal-affairs/recommendation-ethics-artificial-intelligence
- Law on Artificial Intelligence, Articles 8 and 22: https://old.adilet.zan.kz/rus/docs/Z2500000230

These instruments provide the legal and ethical basis for the selected controls. The legal provisions are applied as design requirements relevant to the proposed system; this assignment does not establish a final legal opinion on the applicability of every provision to a real scholarship fund.

## 7. Trade-Off Analysis

The design accepts several trade-offs.

First, hiding names and universities improves protection against irrelevant attributes, but it also removes information that might sometimes provide useful context. This can change individual scores and decisions.

Second, human review improves oversight but creates additional work. In the measured run, 86 of 400 profiles were escalated.

Third, the refusal mechanism improves safety but may prevent some requests from being processed. Therefore, refusal rules should be tested regularly to avoid incorrectly blocking legitimate applicants.

Fourth, the decision log improves accountability but creates data protection responsibilities. Logs should be protected, access-controlled and retained only as long as necessary.

Finally, counterfactual checking was not enabled. This avoided doubling model calls, but it means the submitted harness does not perform an additional model-based comparison for every applicant.

I consider these trade-offs acceptable for a prototype because protecting applicants and preserving human responsibility are more important than maximizing automation.

## 8. AI-Use Declaration

I used ChatGPT as an AI assistance tool during this assignment. It helped me understand the assignment requirements, interpret the experiment outputs, organize the measurement results, prepare documentation and identify relevant ethical and legal provisions for further verification.

Google Colab was used as the execution environment. The Qwen2.5-3B-Instruct model supplied by the assignment was used to perform the scholarship screening experiments. The model ran locally in the Colab runtime without an external API key.

I reviewed the experiment outputs and used them as the basis for the report. I also checked the cited legal instruments and their provisions using the official legal texts available through the Adilet legal information system and UNESCO's published Recommendation.

The AI tools assisted with analysis and drafting, but responsibility for the submitted code, results, legal references and final decisions remains with me.
