# Week 4: Model Comparison

Tested 4 AI models on 5 cybersecurity text samples to evaluate their suitability for a security alert classification / triage component.

## Models Tested

- HF Sentiment: distilbert-sst-2
- HF Zero-Shot: bart-large-mnli
- HF NER: bert-large-NER
- Groq Llama 3.1 8B Instant

## Finding

Recommended Groq Llama 3.1 8B Instant for the security alert triage component because it gave the most context-aware severity classifications and included useful reasoning. Zero-Shot classification is also useful as a faster lightweight routing layer.

See `report.md` for full analysis.
