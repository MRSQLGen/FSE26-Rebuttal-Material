# Failure Cases Analysis

We randomly sampled 100 cases in which MRSQLGen failed to correctly detect hallucinations, focusing on the underlying technical causes that lead to false positives and false negatives. Our analysis reveals that the majority of these failures stem from two primary sources: (i) imperfect HKB retrieval, which leads to unrelated hallucination-type prediction and misaligned MR rule selection; and (ii) hallucinations introduced during metamorphic query generation, where the auxiliary LLM deviates from the intended transformation constraints and produces metamorphic SQL that violates the expected MR relations.



## Failure Mode 1: Inaccurate HKB Retrieval Leading to Incorrect MR Rule Selection

**Description**

MRSQLGen relies on accurate hallucination-type retrieval from the HKB to select the correct metamorphic rules. When the retrieved neighborhood does not contain the hallucination type exhibited in the input query, the system will select an irrelevant MR category. In such cases, the generated metamorphic prompts do not target the faulty span of the SQL, meaning the metamorphic queries fail to amplify the hallucination and the MR relationship remains trivially satisfied. This yields a **false negative**, where a real hallucination remains undetected because MRSQLGen is “testing the wrong thing.”

**Example**

**User question:**
“For each customer, compute the total amount they spent on items priced above 100.”

**LLM-generated SQL (contains a real hallucination):**

```sql
Q: 
SELECT C.customer_name, SUM(O.amount)
FROM Customers C
JOIN Orders O ON C.id = O.customer_id
JOIN Items I ON O.item_id = I.id
GROUP BY C.customer_name;
```

❌ **C6: Condition Logic Hallucination** — the required predicate `I.price > 100` is missing.

**How MRSQLGen Fails**

**Step 1:  Incorrect hallucination-type retrieval.**  Because the prompt and SQL involve a multi-table JOIN and a single aggregate, the joint embedding tends to cluster with HKB cases dominated by **C3: Join Logic Hallucination**. The voting stage therefore predicts C3 instead of the correct C6, leading MRSQLGen to select the wrong MR family.

**Step 2:  Selecting an irrelevant MR rule.**  MRSQLGen applies a QMR-1 rule for C3  (Join Reversal / Join Perturbation).
However, the JOIN structure is correct in this example—only the missing price filter is erroneous—so the chosen MR does not target the actual faulty logic span.

**Step 3：Generating hallucination-insensitive metamorphic prompts.**  Following the C3 rule, MRSQLGen produces a metamorphic prompt such as:

> “Compute the total spending per customer by allowing customer and order records to match more broadly, while still calculating the same total amount spent on items priced above 100.”

The auxiliary LLM then generates:

```sql
Q1:
SELECT C.customer_name, SUM(O.amount)
FROM Customers C
LEFT JOIN Orders O ON C.id = O.customer_id
JOIN Items I ON O.item_id = I.id
GROUP BY C.customer_name;
```

This transformation relaxes the join condition, but—critically—it still omits the required predicate `I.price > 100`. Therefore, it does not expose the missing-filter hallucination.  Because both Q and Q1 sum all item amounts (due to the same missing predicate), their results satisfy the expected superset MR relation:  Result(Q1)⊇Result(Q).   Thus MRSQLGen incorrectly concludes that the original query is semantically stable, and the C6 hallucination remains undetected.



## Failure Mode 2: LLM Hallucination During Metamorphic Prompt Execution

**Description**

Even when MRSQLGen retrieves the correct hallucination type and selects the appropriate metamorphic rule, the LLM responsible for generating metamorphic query may introduce new hallucinations. This issue is more pronounced when metamorphic prompts become structurally complex. These hallucinations violate the MR rule constraints, causing the resulting metamorphic queries to fail the expected MR relation. (In short, it is a hallucination introduced by the metamorphic query, not by the original query.) While cross-validation mitigates some of these unstable generations, such cases can still lead to **false positives**.

**Example**

**User question:**
 “List all cities with more than 1,000 residents.”

**LLM-generated SQL (correct):**

```sql
Q:
SELECT city_name
FROM City
WHERE population > 1000;
```

**Hallucination type retrieval (correct)**

MRSQLGen correctly identifies a potential **C1: Operator Misuse** risk and selects the appropriate QMR-1 rule and generates the metamorphic prompt：

> “Show all cities whose population exceeds 5,000.”

**LLM-generated SQL (hallucinated):**

```sql
Q1:
SELECT city_name
FROM City
WHERE population > 5000
  AND region = 'North';   -- hallucinated region filter
```

Although MRSQLGen expected the transformed query to satisfy a subset MR relation  Result(Q1)⊆Result(Q), the hallucinated predicate region = 'North' fundamentally alters the filtering semantics. As a result, Q1 no longer represents a valid threshold-tightening transformation, and its output is no longer guaranteed to be a subset of the original results. Consequently, the MR check fails—even though the original SQL is perfectly correct.
