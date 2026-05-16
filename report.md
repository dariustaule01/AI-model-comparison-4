# Model Comparison Report — Week 4

**Name:** Darius Taule  
**Date:** May 15, 2026  
**Capstone Project:** AI-Capstone-Threat-Intel  
**Project Description:** This project builds an automated threat intelligence system that collects, analyzes, and prioritizes cybersecurity threats from public sources.  
**My Component:** Automated threat triage and model-assisted severity classification

## Test Setup

**Input dataset:** 5 cybersecurity alert samples covering:

- 3 clearly concerning or high-severity records: unauthorized login, phishing email, and repeated failed SSH attempts
- 2 routine or benign records: scheduled firewall maintenance and normal system resource utilization

**Models tested:**

1. **distilbert-base-uncased-finetuned-sst-2-english** — sentiment analysis
2. **facebook/bart-large-mnli** — zero-shot classification
3. **dslim/bert-large-NER** — named entity recognition
4. **Groq Llama 3.1 8B Instant** — LLM-based cybersecurity severity classification

**Evaluation criteria:** I evaluated the models based on label usefulness, confidence score, whether the classification made sense for cybersecurity triage, whether extracted entities were useful, and how practical the model would be inside an automated n8n workflow.

## Results Summary

| Record | Input Summary | Sentiment | Zero-Shot | NER Entities | Groq |
|--------|---------------|-----------|-----------|--------------|------|
| 1 | Unauthorized login from Moscow targeting John Miller | NEGATIVE (0.9961) | possible anomaly (0.9000) | Moscow | CRITICAL — unauthorized login targeting a user may threaten credentials and data integrity |
| 2 | Routine firewall rule update during scheduled maintenance | NEGATIVE (0.9986) | routine activity (1.0000) | None | INFORMATIONAL — standard scheduled activity with no threat indicated |
| 3 | Phishing email using spoofed Amazon domain | NEGATIVE (0.9959) | possible anomaly (0.8000) | Amazon | CRITICAL — credible phishing attempt with risk of financial loss and unauthorized access |
| 4 | Multiple failed SSH attempts from Beijing IP on Cloudflare server | NEGATIVE (0.9994) | possible anomaly (0.8000) | SS | CRITICAL — likely brute-force authentication attack against production infrastructure |
| 5 | Normal system resource utilization, no anomalies | NEGATIVE (0.9880) | routine activity (1.0000) | None | INFORMATIONAL — expected normal condition with no security issue indicated |

## Record-by-Record Analysis

### Record 1: Unauthorized login from Moscow

The sentiment model marked the alert as **NEGATIVE** with high confidence, which correctly reflected that the unauthorized login was concerning. The zero-shot model labeled it as **possible anomaly**, which was sensible but not as severe as the alert deserved. The NER model extracted **Moscow**, which was useful location context, but it missed the IP address and the user name John Miller. Groq classified the record as **CRITICAL** and gave the strongest security-focused explanation. Overall, all classification models treated the record as concerning, but Groq produced the most useful threat-triage output.

### Record 2: Routine firewall rule update

The sentiment model marked this record as **NEGATIVE**, which was incorrect because the record described planned maintenance. The zero-shot model labeled it as **routine activity** with full confidence, which was correct. The NER model returned no useful entities, which was expected because the record did not contain a major named person, place, or organization. Groq classified it as **INFORMATIONAL**, matching the zero-shot result. This record shows that sentiment analysis is not reliable for security operations because technical language can be interpreted as negative even when the event is benign.

### Record 3: Phishing email with spoofed Amazon domain

The sentiment model marked the phishing alert as **NEGATIVE** with high confidence, which was directionally correct. The zero-shot model labeled it as **possible anomaly**, which made sense, although a stronger severity label would also be justified. The NER model extracted **Amazon** as an organization, which was useful because the domain was spoofed. However, it did not extract the target email address. Groq classified the alert as **CRITICAL** and gave the most useful explanation because it connected the spoofed domain to financial loss and unauthorized access. For threat intelligence triage, Groq produced the strongest output.

### Record 4: Multiple failed SSH attempts

The sentiment model marked this record as **NEGATIVE** with very high confidence, which correctly reflected that the alert was concerning. The zero-shot model labeled it as **possible anomaly**, which was sensible but understated the severity of 47 failed attempts in 5 minutes. The NER model extracted **SS** as a miscellaneous entity, which was not useful and appears to be a bad partial extraction from SSH. Groq classified the alert as **CRITICAL** and correctly described the event as a possible brute-force authentication attack. This was one of the clearest examples where the LLM gave the most operationally useful cybersecurity analysis.

### Record 5: Normal utilization, no anomalies

The sentiment model marked this record as **NEGATIVE**, which was incorrect because the input clearly said that utilization was normal and no anomalies were detected. The zero-shot model labeled it as **routine activity** with full confidence, which was correct. The NER model returned no entities, which was expected for a generic system-status message. Groq classified it as **INFORMATIONAL**, agreeing with the zero-shot result. This record further confirms that sentiment analysis alone is a poor fit for cybersecurity triage.

## Analysis

**Where models agreed:** The models mostly agreed on Records 1, 3, and 4 as concerning events. Sentiment, zero-shot, and Groq all treated these records as suspicious or negative. Groq was the most specific because it assigned operational severity levels such as **CRITICAL** instead of only identifying negative tone or possible anomaly status.

**Where models disagreed:** The largest disagreements occurred on Records 2 and 5. These were benign records, but the sentiment model still classified them as **NEGATIVE**. Zero-shot and Groq correctly recognized them as routine or informational. This disagreement matters because an automated threat intelligence system needs to reduce false positives, not simply flag every technical-sounding event as negative.

**Most accurate model overall:** Groq Llama 3.1 8B Instant was the most accurate and useful model overall for this capstone project. It correctly separated critical alerts from informational records and gave short explanations that matched cybersecurity reasoning. It was especially strong for interpreting context, such as recognizing failed SSH attempts as a potential brute-force attack.

**Fastest or most practical model:** The zero-shot classifier was very practical because it returned simple labels and confidence scores that are easy to store in Airtable and route through n8n. However, Groq was more useful for the actual capstone task because it produced severity labels and explanations that could directly support threat prioritization.

**Most limited model:** The sentiment model was the least reliable for cybersecurity triage. It correctly flagged dangerous records as negative, but it also incorrectly flagged routine records as negative. The NER model was useful only as a supporting model because it extracted some entities but missed important cybersecurity indicators such as IP addresses, email addresses, and usernames.

## Recommended Models for My Capstone Component

**Component:** Automated threat triage and model-assisted severity classification for AI-Capstone-Threat-Intel

**Primary model:** **Groq Llama 3.1 8B Instant** — This should be the primary model because it can classify threat severity, explain its reasoning in plain language, and interpret cybersecurity context better than the simpler Hugging Face models.

**Secondary model:** **facebook/bart-large-mnli** — This should be used as a secondary model for fast routing into categories such as critical threat, possible anomaly, and routine activity. It is useful for structured classification and confidence-based routing inside n8n.

**Supporting model:** **dslim/bert-large-NER** — This model can support enrichment by extracting named entities such as organizations and locations, but it should not be responsible for threat severity decisions.

**Rejected model:** **distilbert-base-uncased-finetuned-sst-2-english** — This model is not a good fit for the capstone’s main triage task because sentiment does not equal cybersecurity severity. It produced false positives on routine records, which would create unnecessary analyst workload.

## Failure Cases and Limitations

The clearest failure case was the sentiment model’s handling of routine security records. It labeled both the scheduled firewall update and the normal system utilization record as **NEGATIVE**, even though both were benign. This shows that general-purpose sentiment analysis is not reliable for cybersecurity workflows. Security text often contains words that sound negative or technical, but the actual event may be normal.

The NER model also had limitations. It extracted **Moscow** and **Amazon**, which were useful, but it missed important cyber indicators such as IP addresses, email addresses, usernames, and infrastructure names. It also incorrectly extracted **SS** from SSH as a miscellaneous entity. This means that a production version of the capstone system would need cybersecurity-specific entity extraction, indicator-of-compromise parsing, or custom regex extraction for IP addresses, domains, hashes, emails, and hostnames.

Groq performed best, but it still has limitations. It can produce strong explanations, but those explanations should be treated as model-generated judgments, not final truth. In a real threat intelligence system, the LLM output should be combined with source reputation, threat feeds, confidence scoring, and analyst review for high-impact decisions.

## Next Steps

If I had more time, I would test a larger dataset with more realistic threat intelligence records from public sources. I would include indicators of compromise such as IP addresses, domains, file hashes, malware names, CVE numbers, usernames, and organization names. I would also test more specific labels, such as **credential attack**, **phishing**, **malware**, **vulnerability exploitation**, **benign**, and **needs review**.

For the next version of the workflow, I would add confidence-based routing. Records classified as critical or high confidence could be sent to a high-priority review table, while routine records could be logged without urgent review. I would also add a separate extraction step for cyber indicators so the capstone system can collect and prioritize actionable intelligence, not just classify text.

## Reflection

What surprised me most was how poorly the sentiment model handled routine cybersecurity text. I expected it to separate concerning alerts from benign events, but it treated normal maintenance and normal utilization as negative. This made it clear that threat intelligence needs domain-aware classification, not just general language sentiment. Groq was the most useful model because it understood the operational meaning of the alerts and produced severity labels that would fit the AI-Capstone-Threat-Intel workflow.
