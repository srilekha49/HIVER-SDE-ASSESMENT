# Engineering Decision Log (@SpotifyCares Support Agent)

* **Brand Choice**: Selected `@SpotifyCares` due to high query volume, distinct intent taxonomy (Billing vs. Playback vs. Login), and structured resolution links (`spoti.fi/*`).
* **Intent Taxonomy Scope**: Constrained taxonomy to 6 mutually exclusive categories to prevent class overlap and keep Macro F1 score high.
* **Retrieval Architecture**: Used TF-IDF + Cosine Similarity over external vector databases (e.g., Pinecone) to eliminate API latency and maintain zero-cost offline execution.
* **Classification Model**: Selected Logistic Regression with N-gram features over deep networks due to sub-10ms inference and robustness on short Twitter text.
* **Confidence-Based Escalation**: Established a 0.45 confidence threshold; queries below this automatically escalate to human agents.
* **Hard Keyword Safeguards**: Added strict regex triggers (`hacked`, `unauthorized charge`) to bypass automated replies for sensitive compliance/security issues.
* **Golden Set Size**: Curated 200 hand-verified samples with stratified intent representation to minimize sampling bias.
* **LLM-as-a-Judge Framework**: Implemented a 3-axis rubric (Relevancy, Empathy, Grounding) rather than BLEU/ROUGE, which correlate poorly with customer satisfaction.
* **URL Grounding Enforcement**: Forced drafted replies to attach verified `@SpotifyCares` help links (`spoti.fi`) to ensure factual correctness.
* **Handling Ambiguity**: Routed short/vague messages (e.g., "help", "app dead") to `GENERAL_SUPPORT` with clarification prompts.
* **Stateless Initial Turn**: Designed per-tweet stateless classification to align with Twitter's single-turn initial response paradigm.
* **Reproducibility Guarantee**: Locked random seeds (`random_state=42`) across all data splitters and classification models.
