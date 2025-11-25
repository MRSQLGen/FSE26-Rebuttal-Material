# **Details of Intent-Violating Hallucination Types in LLM-Generated SQL**

### **C1: Operator Misuse (4.9%)**

This type of hallucination refers to the incorrect use of arithmetic, comparison, or logical operators in the generated SQL query, leading to query logic that deviates from the user’s original intent. The hallucination typically stems from the model’s misunderstanding of comparative or quantitative language expressions such as “more than,” “not in,” or “between.” For example, as shown in Figure 1, the model misinterprets the comparative semantics of “reach 80” and incorrectly generates the equality operator (`= 80`) instead of the intended threshold condition (`≥ 80`), resulting in a narrowed and intent-violating query.

![](https://raw.githubusercontent.com/Jasper0209/IMAGES/main/img/CASE1.png)
Fig. 1. C1: Operator Misuse: Faulty Predicate in LLM-Generated SQL

### **C2: Limit Error (6.5%)**

This type of hallucination occurs when the model mishandles result cardinality control in the generated SQL query. It mainly manifests as missing LIMIT clauses, incorrect values for “LIMIT” or “OFFSET”, or the absence of supporting structures such as “ORDER BY”, ultimately causing the result set to deviate from the actual intent expressed in natural language. The hallucination often arises from the model’s misinterpretation or neglect of quantity-related semantics in natural language queries, such as "top 5", "first few rows", "rows 10 to 20", or "at most one result". For example, as shown in Figure 2, the model fails to capture the recency-based cardinality requirement implied by “the track that was opened in the most recent year.” Instead of using an `ORDER BY … DESC LIMIT 1` structure to explicitly restrict the output to a single most recent record, the model rewrites the query using a `WHERE = (SELECT MAX(…))` condition. Although logically related, this formulation bypasses the intended `LIMIT 1` control and may return multiple tracks if several share the same maximum opening year—thus violating the user’s intent of retrieving exactly one result.

![](https://raw.githubusercontent.com/Jasper0209/IMAGES/main/img/CASE2.png)
Fig. 2. C2: Limit Error: Faulty Predicate in LLM-Generated SQL

### **C3: Join Logic Hallucination (38.7%)**

This hallucination refers to the incorrect construction of join logic in SQL queries, where the generated statement deviates from the intended multi-table semantics or introduces irrelevant, spurious, or logically inconsistent relationships between tables. The hallucination often arises from the model’s misunderstanding of entity relationships, attribute ownership, or foreign key dependencies implied in natural language. Typical manifestations include omitted join relations (failing to connect necessary tables), spurious joins (introducing irrelevant tables), join type misuse (e.g., “LEFT JOIN” used instead of “INNER JOIN”), and faulty join predicates. For example, as shown in Figure 3, the model incorrectly joins `election.Counties_Represented` with `county.County_Id` instead of using the correct foreign key predicate `election.District = county.County_id`, leading to logically invalid join results.

![](https://raw.githubusercontent.com/Jasper0209/IMAGES/main/img/CASE3.png)
Fig. 3. C3: Join Logic Hallucination: Faulty Predicate in LLM-Generated SQL

### **C4: Violating Value Specification (12.9%)**

This type of hallucination occurs when the model generates incorrect values for SQL query conditions, due to misunderstanding the valid value range, format, or semantic constraints of the target fields in the database. Such errors commonly appear in WHERE or HAVING clauses. For example, as shown in Figure 4, the model incorrectly generalizes the value constraints by replacing the required exact comparisons (`EVENTS != "Fog"` / `"Rain"`) with pattern-based filters (`NOT LIKE '%Fog%'`), thereby over-expanding the exclusion scope and violating the user’s intended value specification.

![](https://raw.githubusercontent.com/Jasper0209/IMAGES/main/img/CASE4.png)
Fig. 4. C4: Violating Value Specification: Faulty Predicate in LLM-Generated SQL

### **C5: Column Selection Error (15.7%)**

This type of hallucination occurs when the model incorrectly selects columns in the SELECT clause. The issue may involve omitting required columns, selecting extraneous ones, or confusing semantically similar fields. Such hallucinations often arise when the model fails to align entities mentioned in natural language with their corresponding schema fields. They are particularly common when field names are ambiguous across tables, or when the model lacks a precise understanding of column ownership and context. For example, as shown in Figure 5, the model selects all columns (`*`) instead of the specific `truck_details` field requested by the user. This over-selection includes extraneous attributes, deviating from the intended query semantics and potentially returning more data than necessary.

![](https://raw.githubusercontent.com/Jasper0209/IMAGES/main/img/CASE5.png)
Fig. 5. C5: Column Selection Error: Faulty Predicate in LLM-Generated SQL

### **C6: Condition Logic Hallucination (18.6%)**

This type refers to errors in constructing logical conditions, where the SQL query misrepresents the intended filtering logic due to incorrect operators, incomplete expressions, or ill-formed condition structures in clauses like WHERE or JOIN. Common manifestations include spurious conditions that were not mentioned in the prompt, underspecified conditions that fail to capture the complete filtering intent, misconstrained predicates with incorrect values or comparison operators, and faulty logical connections such as misused AND/OR or misplaced NOT. In more subtle cases, malformed condition structures—such as missing parentheses or incorrect nesting—lead to logical parsing errors while still producing executable queries. For example, as shown in Figure 6, the model misinterprets the dual-set condition expressed in natural language and incorrectly merges the two population filters into a single `Population > 1500 AND Population < 500` predicate, which is logically impossible and violates the intended `INTERSECT` semantics.

![](https://raw.githubusercontent.com/Jasper0209/IMAGES/main/img/CASE6.png)
Fig. 6. C6: Condition Logic Hallucination: Faulty Predicate in LLM-Generated SQL

### **C7: Aggregation Function Misuse (4.5%)**

This type of hallucination occurs when the model applies incorrect aggregation operations in SQL queries. Such errors typically arise from misinterpreting the intended aggregation target, misunderstanding the summary method, or applying aggregation to inappropriate attributes. Common manifestations include the use of an aggregation function that does not match the user intent. For example, as shown in Figure 7, the model misinterprets the user’s request and unnecessarily replaces a simple field retrieval with an aggregation (`SUM(Kids)`), thereby altering the intended semantics of the query.

![](https://raw.githubusercontent.com/Jasper0209/IMAGES/main/img/CASE7.png)
Fig. 7. C7: Aggregation Function Misuse: Faulty Predicate in LLM-Generated SQL

### **C8: Distinct Error (14.7%)**

This type of hallucination refers to the incorrect use of the DISTINCT modifier in SQL queries, leading to unintended duplication or unintended loss of result records. Such errors occur when the model fails to correctly interpret whether uniqueness is required in the users’ question. For example, as shown in Figure 8, the model incorrectly adds `DISTINCT`, which removes valid repeated (zip_code, date) pairs and alters the intended result — for instance, if the same zip code reaches ≥80°F on multiple days, using `DISTINCT` will collapse these informative repeated rows and change the original query’s semantics.

![](https://raw.githubusercontent.com/Jasper0209/IMAGES/main/img/CASE8.png)
Fig. 8. C8: Distinct Error: Faulty Predicate in LLM-Generated SQL

### **C9: OrderBy Misuse (9.9%)**

This type of hallucination refers to errors in the construction of the ORDER BY clause, where the sorting logic expressed in natural language is incorrectly translated into SQL, resulting in output that does not match the expected order. Typical forms include omitting the ORDER BY clause when sorting is required, using incorrect sort attributes or directions, or misordering fields in multi-level sorting. For example, as shown in Figure 9, the model orders by `registration_date` in the wrong table (`Student_Course_Registrations`) instead of `date_of_attendance` in `student_course_attendance`, producing results that do not reflect the most recent registration for course 301.

![](https://raw.githubusercontent.com/Jasper0209/IMAGES/main/img/CASE9.png)
Fig. 9. C9: OrderBy Misuse: Faulty Predicate in LLM-Generated SQL

### **C10: GroupBy Misuse (15.9%)**

This type of hallucination refers to the incorrect use of the GROUP BY clause in SQL queries. Such errors typically stem from misinterpretation of the aggregation target, grouping unit, or statistical logic expressed in natural language. Common manifestations include applying aggregation functions without a required GROUP BY clause, grouping by incorrect fields that mismatch the expected result granularity, introducing unnecessary grouping in nonaggregation queries, or failing to aggregate non-grouped fields—potentially causing runtime errors or ambiguous semantics. For example, as shown in Figure 10, when finding students with multiple advisors, the model groups by `student.name` instead of `student.ID`, which can conflate or split students with the same name and produce incorrect aggregation results.

![](https://raw.githubusercontent.com/Jasper0209/IMAGES/main/img/CASE10.png)
Fig. 10. C10: GroupBy Misuse: Faulty Predicate in LLM-Generated SQL
