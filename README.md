# DeepSeek-R1/V3 — Chinese Chain-of-Thought Trigger Investigation

**An independent empirical study into why DeepSeek switches its internal reasoning to Chinese on English prompts — and what exactly causes it.**

---

## The Finding

DeepSeek-R1/V3 spontaneously switches its Chain-of-Thought (CoT) reasoning to Chinese when processing certain English-language prompts. This happens silently — the final response may still appear in English, but the internal reasoning that produced it is entirely in Chinese.

After 50 controlled tests, I identified the root cause: **the co-occurrence of high technical detail density and numeric scoring formats** in a single prompt triggers DeepSeek's MoE router to assign Chinese-language reasoning experts to the task — before any user instruction can intervene.

---

## Key Results

| Test | Condition | Override: Chinese CoT | No Override: Chinese CoT |
|------|-----------|----------------------|--------------------------|
| T1 | Full GRC prompt (dense compliance jargon + scoring /10) | 5/5 (100%) | 5/5 (100%) |
| T2 | Same structure, stripped of dense prose | 0/5 (0%) | 5/5 (100%) |
| T3 | Mobile dev languages — scoring out of 10 | 2/5 (40%) | 5/5 (100%) |
| T4 | Mobile dev languages — scoring out of 100 | 4/5 (80%) | 4/5 (80%) |
| T5 | Mobile dev languages — qualitative H/M/L rating | 0/5 (0%) | 5/5 (100%) |

> Each condition: 5 runs. "Override" = prompt prepended with `SYSTEM OVERRIDE: Respond exclusively in US English.`

---

## What the Data Proves

- A System Override instruction **fully controls final output language** but has **zero effect on CoT language** when detail density is high
- Scoring `out of 100` triggers Chinese CoT **more** than `out of 10`
- Replacing numeric scores with qualitative ratings (High / Medium / Low) **eliminates** Chinese CoT under override conditions
- Structural elements alone (tiers, time-boxing, tool names) do **not** trigger the behavior — dense explanatory prose is required

---

## Root Cause Hypothesis

DeepSeek-R1 uses a Mixture-of-Experts (MoE) architecture. The router classifies task type in the first few transformer layers and assigns domain-specific experts accordingly.

Prompts combining **dense enterprise/compliance/infrastructure jargon** with **numeric scoring formats** are statistically overrepresented in Chinese technical training corpora (CSDN, Zhihu). The router classifies them as Chinese-coded tasks and assigns Chinese-language reasoning experts — before the system prompt override can influence the generation path.

This is consistent with DeepSeek's published RLVR findings, which document a **5.6% accuracy drop** when single-language CoT is enforced, and with community-reported GitHub issues [#1240](https://github.com/deepseek-ai/DeepSeek-R1/issues/1240) and [#1295](https://github.com/deepseek-ai/DeepSeek-R1/issues/1295).

This is not a bug — it is a **design trade-off between reasoning accuracy and language consistency**.

---

## Practical Workarounds (Until an API Fix Exists)

- **Avoid** combining dense technical jargon with numeric scoring in the same prompt
- **Replace** `score out of 10` / `score out of 100` with `High / Medium / Low` ratings
- **Reduce** explanatory prose density — a structural skeleton prompt with the same logical elements will not trigger Chinese CoT
- A System Override instruction is sufficient to **guarantee English final output**, even when it cannot control CoT language

The only complete fix is an API-level parameter (e.g., `reasoning_language: "en"`). No user-side prompt engineering can fully override the router's early-stage classification.

---

## Methodology

- **Platform:** DeepSeek web interface (chat.deepseek.com) — not the API
- **Isolation:** Each run conducted in a fresh, separate browser window with no prior conversation history
- **Detection:** Chinese CoT identified by direct visual inspection of the model's thinking block and response text
- **Total runs:** 50 (5 prompts × 10 runs each — 5 with override, 5 without)
- **Testing window:** June 2026

---

## Report

The full formal investigation report (methodology, hypothesis table, complete results, root cause analysis, limitations) is available as a Word document in this repository.

📄 [`DeepSeek_CoT_Report_Final.docx`](./DeepSeek_CoT_Report_Final.docx)

---

## Author

**Abu Horairah Ansari**  
Independent researcher — B.Tech Computer Science student  
This investigation was conducted independently, without institutional affiliation.

---

## License

This research is shared openly for community reference. If you build on these findings, a credit or link back to this repository would be appreciated.
