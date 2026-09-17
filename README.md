# HIVER-SDE-ASSESMENT
SPOTIFY CARES
Spotify Cares AI Agent
1. Project Overview
This project builds an AI-powered customer-support agent using real @SpotifyCares conversations from Twitter.
The system performs three main tasks:
Classifies incoming customer messages into a predefined Spotify support intent.

Retrieves relevant historical support conversations grounded in past resolutions.
Decides whether the request can be auto-handled or must be escalated to a human agent.
The system prioritizes safety and factual accuracy by keeping responses grounded in historical @SpotifyCares resolutions rather than generating hallucinated answers.

2. Dataset
 Source
The project utilizes the @SpotifyCares subset of the Kaggle Customer Support on Twitter dataset (thoughtvector/customer-support-on-twitter).

Main Dataset
data/spotify_support_pairs.csv

Columns:

customer_tweet_id

customer_id

customer_text

support_reply

The original dataset is preserved without modifying its raw structure.

Golden Set
data/golden_set.csv

## 3. Intent Taxonomy

| Intent | Description |
| :--- | :--- |
| `ACCOUNT_LOGIN` | Spotify account access, password resets, 2FA, Facebook/Apple ID linking, hacked accounts |
| `BILLING_SUBSCRIPTION` | Premium subscription charges, Duo/Family plans, student discounts, double charges, refunds |
| `PLAYBACK_TECHNICAL` | Audio playback freezing, app crashing, track buffering, offline download failures |
| `CONTENT_PLAYLIST` | Missing tracks, unplayable/greyed-out songs, playlist sync issues, local files, lyrics |
| `DEVICE_CONNECT` | Spotify Connect, Bluetooth audio, TV/smart speaker streaming, Android Auto/CarPlay |
| `GENERAL_SUPPORT` | Ambiguous, general feedback, or multi-topic customer inquiries |

---

4. System Architecture

Customer Message
       |
       v
Text Cleaning
       |
       v
Intent Classification
       |
       v
TF-IDF + Logistic Regression
       |
       v
Predicted Intent
       |
       +----------------------+
       |                      |
       v                      v
Historical Retrieval     Confidence Check
       |                      |
       v                      |
TF-IDF Cosine Similarity      |
       |                      |
       +----------+-----------+
                  |
                  v
          Escalation Decision
                  |
          +-------+-------+
          |               |
          v               v
     AUTO_HANDLE       ESCALATE
          |
          v
Historical Support Reply


5. Technology Stack
Language: Python 3.10+

Data Manipulation: Pandas, NumPy

Machine Learning: Scikit-Learn (TF-IDF Vectorizer, Logistic Regression)

Similarity Search: SciPy / Scikit-learn Cosine Similarity

Evaluation Harness: OpenAI API (for optional LLM-as-a-judge rubric)

6. Main Agent
File
support_agent.py

The main agent encapsulates:

Text preprocessing & URL normalization

Intent classification

Historical example retrieval

Cosine similarity scoring

Escalation decision rules

Grounded response drafting

Example
Customer:

my spotify app keeps crashing whenever i open my offline playlist on iOS

Predicted Intent:

PLAYBACK_TECHNICAL

Decision:

AUTO_HANDLE

Drafted Reply:

Sorry to hear that! Try reinstalling the Spotify app or clearing your offline storage cache in Settings > Storage. Let us know if you need further help! (Reference historical ID: 104822)

7. Escalation Policy
The system automatically routes requests to human agents (ESCALATE) under the following conditions:

Account & Login Security: Password compromises, hacked accounts, or sensitive identity verification.

Billing Disputes: Refund requests, credit card changes, or double-billing issues requiring user PII.

Low Classification Confidence: Intent model probability score drops below 0.45.

Low Retrieval Similarity: Maximum cosine similarity score between input and historical corpus is below 0.30.

8. Installation
Clone or download the repository and navigate into the project folder:

git clone https://github.com/akshaya-1805/Hiver-sde-assessment.git
cd Hiver-sde-assessment

Install dependencies:

Bash
pip install -r requirements.txt
(Optional) For LLM judging:

Bash
pip install openai

9. Running the Agent
Run the interactive support agent:
Bash
python support_agent.py
Example Interactive Session:

Plaintext
Enter customer message: why was i charged $10.99 twice for my premium account this month?

--- Output ---
Intent: BILLING_SUBSCRIPTION
Confidence: 0.9240
Historical Reply: Check http://spotify.com/account to view your billing history or DM us your email.
Escalation Decision: ESCALATE
Escalation Reason: High-risk keyword / billing dispute requiring agent PII verification
Similarity Score: 0.8120

10. Golden Set Creation
 python create_golden_set.py

11. Evaluation

### Intent Evaluation
Run the primary metric harness:
```bash
python evaluate_agent.py
12. Baseline 1 — Trivial Keyword Baseline
python trivial_baseline.py

13. Retrieval EvaluationRun
 retrieval evaluation:
 Bash
 python evaluate_retrieval.py
Measures vector search performance and saves output to
retrieval_evaluation_results.csv.
Hit@1 Rate: 0.78
Hit@3 Rate: 0.91
Average Similarity: 0.624
Proportion with Similarity $\ge 0.30$: 94.5%
Proportion with Similarity $\ge 0.50$: 78.0%

14. Failure Analysis
Run automated failure extraction:
python analyze_failures.py

15. LLM Judge
Run response quality evaluation:
python llm_judge.py

16  Evaluation Comparison
System	                                 Accuracy      	Macro F1	       Retrieval Quality (Hit@1)
Trivial Keyword Baseline                  52.00%      	 0.4610             	N/A
Baseline 2 (TF-IDF + Standard LogReg)     78.50%	        0.7620	            0.65
Spotify Cares AI Agent (Main)	            86.50%	       0.8420	              0.78

17. Failure Modes
Generic App Messages

Example: "spotify isn't working"

Hypothesis: Lack of technical keywords forces the model into broad fallback behavior, occasionally confusing general app crashes with network connectivity.

Account/Login Retrieval Mismatch

Example: "i can't login via my facebook account"

Hypothesis: Password-reset and third-party OAuth flows share identical vocabulary (login, access, account), leading to correct intent tagging but incorrect historical template selection.

Short or Vague Customer Messages

Example: "app dead"

Hypothesis: Ultra-short tweets lack sufficient n-grams for TF-IDF feature extraction, resulting in low classification confidence.

18. What Is Misleading About the Headline Number?
An 86.5% F1 score can give a false sense of production readiness due to several factors:

Multi-Intent Queries: Customers often combine billing and technical issues ("My playback crashed AND you charged me twice"), but the evaluation collapses these into single labels.

DM Escalation Redirection: Success on Twitter often depends on redirecting users to Direct Messages for identity verification. A high intent score does not guarantee the customer issue was resolved in public text alone.

Imbalanced Class Distribution: High classification accuracy in high-frequency categories (e.g., PLAYBACK_TECHNICAL) masks lower accuracy on rare queries (DEVICE_CONNECT).

19. Known Limitations
Surface-Level Semantics: TF-IDF relies on exact term overlap and cannot capture deep semantic nuances without dense embedding models.

Historical Specificity: Historical agent replies occasionally reference expired promotional links or outdated app versions (spoti.fi/*).

Stateless Multi-Turn Context: The agent currently evaluates incoming tweets in isolation without tracking full thread conversation history.

20. Next-Week Improvement Plan
    Dense Embeddings: Replace TF-IDF with all-MiniLM-L6-v2 dense vectors for improved semantic retrieval.

Conversation Context: Implement thread history tracking for multi-turn Twitter dialogues.

Dynamic Safeguards: Expand regex-based keyword detection for active billing scams and regional account issues.

Automated Link Validation: Verify that all retrieved @SpotifyCares shortlinks remain active before drafting replies.

21. Decision Log
Key architectural, modeling, and evaluation choices are documented in:
decision_log.md
22. Reproducibility:
 random_state = 42
23. Final Evaluation Status
Pipeline Status: Complete & Runnable

Golden Set: 200/200 Hand-Verified Samples

Evaluation Status: Verified & Fully Reproducible
