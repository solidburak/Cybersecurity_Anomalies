Here is a formal **Technical Case Study / Project Report**.

This format is distinct from a `README`. While a `README` tells people *how to run* the code, this report tells the story of the *engineering challenges* you solved. It is perfect for a standalone `REPORT.md` file in your repo or to attach as a PDF to job applications.

---

# Technical Report: Mitigating Shortcut Learning in Deep Malware Detection

**Author:** [Your Name]
**Date:** February 2026
**Subject:** Improving Model Robustness via Stochastic Token Masking

## 1. Executive Summary

This project involved the development of a PyTorch-based neural network for detecting malicious process behavior. While the initial baseline model achieved high validation accuracy (99%), it demonstrated near-zero recall (< 10%) on unseen attack data.

This report details the diagnosis of a **"Shortcut Learning"** failure mode, where the model over-relied on categorical "Unknown" tokens as a proxy for safety. The solution involved engineering a custom training loop with **Stochastic Token Masking**, forcing the model to learn robust, secondary features (specifically `userId`). The final intervention improved test set recall from **~5%** to **~98%** while maintaining generalization.

## 2. The Challenge: The "Perfect" Model That Failed

The initial model architecture utilized Entity Embeddings for categorical features (`processId`, `parentProcessId`, `userId`, `returnValue`) and a fully connected network.

**Initial Metrics:**

* **Validation Accuracy:** >99%
* **Validation Recall:** High
* **Test Set Recall (Real Attacks):** < 10% (Critical Failure)

The model was excellent at identifying "safe" traffic but completely ignored the simulated attacks in the test set, classifying them as benign.

## 3. Root Cause Analysis

A deep-dive investigation into the data distribution revealed the cause of the discrepancy.

### 3.1 The "Unknown" Token Bias

The dataset contained high cardinality fields (Process IDs). Rare or unseen values were mapped to an index of `0` (Unknown).

* **Observation:** In the training and validation sets, legitimate background system processes frequently resulted in `parentProcessId` being mapped to `0`.
* **Learned Heuristic:** The model learned a lazy shortcut: *"If Parent Process is Unknown (0), the sample is Safe."*

### 3.2 The Test Set Collision

The test set consisted of malicious attacks that also generated "Unknown" parent process IDs. Because the model had learned `Unknown == Safe`, it confidently misclassified these attacks as benign.

### 3.3 The Tie-Breaker Investigation

Further analysis was conducted to find features that distinguished "Safe Unknowns" from "Malicious Unknowns."

* **`argsNum` Analysis:** Both groups had similar average argument counts (~2.88 vs 2.94), making this feature insufficient for separation.
* **`userId` Analysis:**
* **Safe Unknowns:** Overwhelmingly ran under `userId: 0` (System/Admin).
* **Malicious Unknowns:** Overwhelmingly ran under `userId: 1001` (Attacker Account).



The signal existed in the data (`userId`), but the model was ignoring it in favor of the easier `parentProcessId` shortcut.

## 4. Engineering Solution: Stochastic Token Masking

To break the model's reliance on the "Unknown" shortcut, I implemented a custom "poisoning" strategy within the PyTorch training loop.

### 4.1 The Algorithm

I introduced a stochastic mask that randomly overwrites the `parentProcessId` and `returnValue` features with `0` (Unknown) during training.

**Logic:**

1. Generate a random probability tensor for the current batch.
2. Create a boolean mask where `probability < Threshold` (tuned to ~0.32).
3. Overwrite selected tokens with `0`, regardless of their true label (Safe or Malicious).

### 4.2 Implementation Snippet

```python
# Inside the training loop:
# Generate random noise probabilities
probs = torch.rand_like(labels_flat, dtype=torch.float)

# Universal Noise Injection: Target ~32% of all data
mask = (probs < 0.32)

# Force the model to see "Unknown" in both Safe and Malicious contexts
b_parent[mask] = 0
b_ret[mask] = 0

```

### 4.3 The Effect

This intervention "poisoned the well." The model could no longer assume that `Parent=0` implied safety, as 32% of clear attacks now also had `Parent=0`. This forced the gradient descent process to optimize weights for the `userId` embedding, the only remaining reliable discriminator.

## 5. Results

The implementation of Stochastic Token Masking yielded a dramatic shift in model behavior.

| Metric | Baseline Model | Robust Model (Final) | Impact |
| --- | --- | --- | --- |
| **Validation Precision** | High | High | Maintained Safety |
| **Test Set Recall** | **~5%** | **~98%** | **+1860% Improvement** |
| **False Positive Rate** | Low | Low | Stable |

## 6. Conclusion

This project demonstrated that high accuracy metrics can mask critical failures in generalization, particularly when "shortcut features" are present. By diagnosing the data distribution mismatch and implementing a custom **Stochastic Token Masking** strategy, I transformed a brittle model into a robust detection engine capable of identifying attacks even when primary indicators are missing.

This approach highlights the importance of **Adversarial Training** and **Data-Centric AI** in cybersecurity applications.
