# System Overhead Analysis

We further examine the computational overhead of MRSQLGen compared with SelfCheckGPT and LLM-as-a-judge, using GPT-4o as the underlying model for all methods. As shown in Table 4, MR-SQLGen consumes more tokens due to generating metamorphic variants for verification, yet its overall cost remains comparable to LLM-as-a-judge and within a practical range for NL2SQL evaluation. Importantly, the per-task latency of MRSQLGen is moderate, only slightly higher than LLM-as-a-judge on both Spider and BIRD, while still offering substantially better detection accuracy.

![image-20251123180031689](System Overhead Analysis.assets/image-20251123180031689.png)