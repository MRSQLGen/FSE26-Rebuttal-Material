# Details of HKB Construction and MR Rules Usage

## HKB Construction and Statistics

This section introduces the construction and scale of HKB. The construction of the HKB is grounded in our human-annotated hallucination corpus (available in our GitHub repository [EmpiricalStudyResult](https://github.com/MRSQLGen/MRSQLGen/blob/main/EmpiricalStudyResults/HallucinationTaxonomy_Intent-violatingHallucination.jsonl)), including question, correct query, predicted query by llms,  hallucination types. To ensure consistent comparison and retrieval, we standardize (detailed process in Section 5.2) the prompt x and query q, then we obtain the processed and structured HKB cases, with their scale shown below.

* Number of cases: 3,968 annotated cases , half of Spider dataset
* Storage space:  4709KB
* Storage format: json format as follows. Each entry (x, q, H ) consists of three elements: (1) the standardized natural language question x, (2) the standardized corresponding LLM-generated SQL query q, and (3) a set of hallucination types H (marked as true indicates the presence of this hallucination type) identified in q. 

``` json
{  
    "index": 0,  
    "node_type": {  
        "C1": true, 
        "C2": false,
        "C3": false,
        "C4": true,
        "C5": true,  
        "C6": true,
        "C7": true,
        "C8": false,
        "C9": false,
        "C10": false
    },  
    "question": "How many [NOUN] of the [NOUN] are older than [NUMBER] ?",  
    "query": "SELECT COUNT(*) FROM [TableName] WHERE [ColumnName] > [Integer]"  
}
```

* Hallucination types:  involve ten hallucination types as follows.
  * C1: Operator Misuse
  * C2: Limit Error
  * C3: Join Logic Hallucination
  * C4: Violating Value Specification
  * C5: Column Selection Error
  * C6: Condition Logic Hallucination
  * C7: Aggregation Function Misuse
  * C8: Distinct Error
  * C9: OrderBy Misuse
  * C10: GroupBy Misuse



## MR Rules Construction and Usage

**We manually designed 26 targeted MR rules, each aligned with a specific hallucination type.** Once MRSQLGen retrieves the predicted hallucination type from the HKB (Fig. 7), the corresponding rule set is applied to transform the original prompt accordingly. The full rule set is available in our GitHub repository ([MetamorphicPromptList](https://github.com/MRSQLGen/MRSQLGen/tree/main/MetamorphicPromptList)) as follows:

* [C1: Operator Misuse](https://github.com/MRSQLGen/MRSQLGen/blob/main/MetamorphicPromptList/MR_Rules_C1.md)
* [C2: Limit Error](https://github.com/MRSQLGen/MRSQLGen/blob/main/MetamorphicPromptList/MR_Rules_C2.md)
* [C3: Join Logic Hallucination](https://github.com/MRSQLGen/MRSQLGen/blob/main/MetamorphicPromptList/MR_Rules_C3.md)
* [C4: Violating Value Specification](https://github.com/MRSQLGen/MRSQLGen/blob/main/MetamorphicPromptList/MR_Rules_C4.md)
* [C5: Column Selection Error](https://github.com/MRSQLGen/MRSQLGen/blob/main/MetamorphicPromptList/MR_Rules_C5.md)
* [C6: Condition Logic Hallucination](https://github.com/MRSQLGen/MRSQLGen/blob/main/MetamorphicPromptList/MR_Rules_C6.md)
* [C7: Aggregation Function Misuse](https://github.com/MRSQLGen/MRSQLGen/blob/main/MetamorphicPromptList/MR_Rules_C7.md)
* [C8: Distinct Error](https://github.com/MRSQLGen/MRSQLGen/blob/main/MetamorphicPromptList/MR_Rules_C8.md)
* [C9: OrderBy Misuse](https://github.com/MRSQLGen/MRSQLGen/blob/main/MetamorphicPromptList/MR_Rules_C9.md)
* [C10: GroupBy Misuse](https://github.com/MRSQLGen/MRSQLGen/blob/main/MetamorphicPromptList/MR_Rules_C10.md)



**For the usage of MR rules, we adopt a LLM-based structured transformation protocol.** To apply metamorphic rules in MRSQLGen, we use an LLM-driven transformation protocol, where the LLM rewrites the original prompt under explicit constraints and few-shot exemplars. This enables precise, hallucination-type–guided prompt transformations.

For MR rules in **QMR-1**, **QMR-2**, and **QMR-3**, we follow a standardized five-step transformation process. We illustrate this using an MR rule from C1: Operator Misuse, which belongs to QMR-1: Intent Perturbation & Reversal:

1. **Task Specification**
    Define the transformation objective—e.g., generating semantically equivalent, strengthened, or relaxed questions—according to the corresponding MR rule category.
2. **Constraint Setting**
   - *Hallucination-type constraint:* the LLM may rewrite only the linguistic spans associated with the targeted hallucination type (e.g., operator phrases for C1).
   - *MR constraint:* enforce a specific metamorphic relation (e.g., equivalence, subset, superset).
3. **Few-shot Exemplars**
    Provide 2–3 example transformations that demonstrate the exact linguistic pattern required for this hallucination type and MR. These exemplars guide the LLM to rewrite the prompt in a controlled, type-consistent way.
4. **Question Under Transformation**
    Insert the original NL question that should be rewritten according to the selected MR rule.
5. **Output Formatting**
    Constrain the output format (e.g., “return N transformed questions only”) to ensure consistency and determinism.

Using this procedure, the LLM produces MR-guided prompt variants tailored to the predicted hallucination type. The corresponding prompt template is shown below:

``` python
'''
C1: Operator Misuse
	QMR-1: Intent Perturbation & Reversal
		MR: equivalence
'''
prompt = (  
    # Task Specification
    f"Generate {n} simple English questions that are semantically equivalent to the following question but different from each other, and whose corresponding SQL queries would also be logically equivalent. "    
    
    # Constraint Setting
    f"You must achieve this by modifying only the arithmetic, comparison, or logical operation descriptions without changing the query's overall meaning or result set.\n\n" 
    
    # Few-shot exemplars
    f"Example 1:\n"  
    f"Original question: Show each employee and their total compensation(positive number), calculated as `base_salary + bonus`.\n"  
    f"Equivalent question: Display all employees with `total_compensation = base_salary + bonus`(positive number).\n"  
  
    f"Example 2:\n"  
    f"Original question: Find employees with a salary greater than 5000.\n"  
    f"Equivalent question: Retrieve all employees whose salary is strictly above five thousand dollars.\n"  
  
    f"Example 3:\n"  
    f"Original question: Find customers who are from New York or Los Angeles.\n"  
    f"Equivalent question:Retrieve customers whose city is either New York or Los Angeles.\n"  
    
    # Question
    f"Now generate {n} equivalent questions for the following question:\n"  
    f"Question: {self.question}\n"   
    
    # Output Formatting
    f"Do not include any explanations in your response. Just return the {n} equivalent questions, each separated by a newline.")
```

For **QMR-4**, transformation does not involve rewriting the NL question.  Instead, the LLM is asked to generate a corrected SQL query directly and self-verify whether the generated SQL introduces the targeted hallucination type.

``` python
metamorphic_question = f"{self.question}\n" + \  
f"### Important rules: 1.After generating the query, you must verify whether the query introduces a {hallu_type_name} hallucination." + \  
"2. If the query introduces such error, fix it automatically." + \  
"3. The final output must only be the corrected query, with no explanation.\n"
```



## Example

