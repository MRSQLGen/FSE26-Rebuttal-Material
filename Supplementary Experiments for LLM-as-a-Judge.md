## Supplementary Experiments for LLM-as-a-Judge

We evaluate two LLM-as-a-Judge settings: **LAJ-Self**, where the model evaluates its own outputs, and **LAJ-G5**, which uses a stronger external judge GPT-5.1.  The experimental setup is the same as in Section 7.1.4 of the paper, with the addition of a stronger judge model, GPT-5.1.The results are listed as follows.

![table3.png](attachments/table3.png)

Across all models and both datasets, using GPT-5.1 as an external judge consistently improves the F1 of LLM-as-a-judge: On Spider (F1), LAJ-G5 outperforms LAJ-Self for every model. For example, with GPT-4o the F1 rises from 0.123 → 0.487, and with GPT-4o-mini from 0.275 → 0.470. On BIRD (F1), the same trend holds: GPT-4o improves from 0.446 → 0.518, and CodeLlama from 0.254 → 0.486. Averaged over all five models, LAJ-G5 brings an absolute F1 gain of roughly 0.17 on Spider and 0.20 on BIRD compared with LAJ-Self. This improvement mainly comes from much higher recall (e.g., for CodeLlama on BIRD, recall increases from 0.220 → 0.791), while precision can slightly increase or decrease depending on the model.


Even with a stronger judge, LAJ-G5 still lags substantially behind our metamorphic method MRSQLGen: For Spider (F1), MRS remains clearly better (e.g., GPT-4o: 0.653 vs. 0.487, CodeLlama: 0.805 vs. 0.409). For BIRD (F1), the gap is similar or even larger (e.g., CodeLlama: 0.944 vs. 0.486, Deepseek-Coder: 0.870 vs. 0.602). This indicates that, while a strong external judge can help calibrate LLM-as-a-judge and recover many missed hallucinations, it still cannot match the precision–recall balance achieved by MR-SQLGen.