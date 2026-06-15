# 🚀 FINETUNING: Vulnerability-Aware Java Model

Base Large Language Models (LLMs) pose a significant risk in enterprise DevSecOps pipelines due to their tendency to hallucinate syntax, offer generalized advice, and fail strict corporate compliance standards. 

This project solves that by fine-tuning the **80-billion parameter Qwen3-Coder-Next (MoE)** model into a strict, highly specialized **Lead Java AppSec Reviewer**. When fed a vulnerable code snippet, it abandons conversational fluff, automatically classifies the CWE, generates mathematical proof of its applied patch, and outputs a production-ready fix.

---

## ⚙️ Model Architecture & Hardware
* **Base Model:** `Qwen/Qwen3-Coder-Next` (80B Mixture-of-Experts, bfloat16)
* **Framework:** Unsloth / Hugging Face `trl`
* **Optimization:** Parameter-Efficient Fine-Tuning (PEFT) via Rank-64 LoRA
* **Hardware Environment:** AMD MI300X GPU (192GB VRAM)
* **Compute Optimization:** `UNSLOTH_MOE_BACKEND="native"` implemented to ensure ROCm kernel stability.

---

## 🗄️ Dataset Curation & CVE Fixes
To teach the model rigorous DevSecOps logic, we curated a triple-filtered dataset focusing exclusively on Java vulnerabilities. 

1. **CVEfixes:** Real-world vulnerable-to-fixed Java code diffs mapped to specific CVEs.
2. **All-CVE-Records:** Foundational vulnerability knowledge and mitigation strategies.
3. **JavaVFC Extended:** A massive 531MB database of raw Git commits fixing Java vulnerabilities.

**Preprocessing Pipeline:**
* Filtered exclusively for Java context using strict RegEx constraints.
* Transformed all raw data into `ChatML` format, anchoring the model to the system prompt: *"You are an expert Java security engineer..."*
* Formatted Git diffs to force the model to read raw code changes and predict the security patch explanation.
* **Volume Control:** Capped at a randomized, balanced split of **5,000 samples** to respect the hackathon time budget, followed by a 90/10 train-test split.

---

## ⚡ Training Details (Epoch & Time Strategy)
Due to strict hackathon time constraints, the pipeline was engineered for maximum alignment in minimum time.

* **Epochs:** 1
* **Training Time:** ~40 minutes (dependent on AMD MI300X I/O speeds)
* **Effective Batch Size:** 16 (Batch size 2 × 8 Gradient Accumulation Steps)
* **Learning Rate:** `2e-4` with Cosine Scheduler and 3% Warmup
* **Target Modules:** Standard attention layers (`q_proj`, `k_proj`, `v_proj`, `o_proj`)

---

## 📊 Evaluation & Known Limitations
The model was evaluated against strict test cases including **CWE-89 (SQLi)**, **CWE-78 (Command Injection)**, **CWE-22 (Path Traversal)**, and **CWE-611 (XXE)**.

### 🏆 Key Successes
The fine-tuning successfully eliminated "chatbot fluff" and aligned the model to enterprise DevSecOps standards.
* **Path Traversal (CWE-22):** The base model offered generic string-matching tips. The fine-tuned model deployed a production-grade fix utilizing `getCanonicalPath()` and `File.separator` boundary checks.
* **Format Adherence:** Successfully generated complex, 6-part DevSecOps vulnerability reports detailing Threat Classification, Attack Vectors, and Fix Verification without prompt degradation.

### ⚠️ Limitations (The "Low Epoch" Tax)
Because training was constrained to **1 epoch** and **5,000 samples**, the model's reasoning on highly complex novel inputs remains brittle.
* On some diff-analysis tasks, the model learned the *format* perfectly but hallucinated the contextual commit message (e.g., confusing a SQL injection patch for a UI crash fix). 
* **Future Work:** Scaling the training run to 3 epochs over the full 22,000+ sample dataset would eliminate these edge-case hallucinations and solidify reasoning capabilities.

---

## 🛠️ Replication Steps
To run the evaluation matrix or train the model yourself:

1. Clone this repository.
2. Ensure you have access to an AMD MI300X or an Nvidia GPU with >80GB VRAM.
3. Open `FINETUNING_001_Java_AppSec.ipynb`.
4. Run Phases 1 through 10 to execute the pipeline from dependency installation to comparative output generation.
5. Run Phase 11 to witness the live generation of a Deep-Dive Security Report.

---
