config = {
  "QQA_THRESHOLD":0.99,
  "QQA_N":10,
  "ENTITY_MATCH_MISMATCH": '''
  You are an expert in the mining industry, who can understand the query intent and can extract key entities from a given user query like key KPIs, mine names (GP3, Suliyari, Talabira, PEKB, all sites, etc.), time periods, etc.
  For a given user_query, give the entities that do not match the given query-entity pairs
  
  user query: 
  {user_query}

  query-entity pairs:
  {similar_queries_and_entities}

  Give me the output in the following format {format_instructions}:

  {{
    "mismatches": {"query1": ["mismatched_entity_1", "mismatched_entity_2"],
                   "query2": ["mismatched_entity_2", "mismatched_entity_3"],
                   "query3": ["mismatched_entity_3", "mismatched_entity_4"]}
  }}

  ## Examples:
  **Input:**
  user query: what is the Cost of Goods Sold in Q3 of 2023 for PEKB mine?
  
  query-entity pairs:
  query 1:  Which mine had the highest EBITDA Coal Block in 2024?
  entity 1: {"Indicator":["EBITDA coal block"],"metric":["highest"], "time_frame":["FY2024"], "mine": []}

  query 2: What is the variance in EBITDA margin across the four sites for current month?
  entity 2: {"Indicator":["EBITDA margin"], "metrics":["variance"], "time_frame":["current month"], "mine": ["four sites"]}

  query 3: Which mine had the highest Cost of Goods Sold in Q3 of 2023?
  entity 3: {"Indicator":["Cost of goods sold(COGS)"],"metrics":["highest"],"time_frame":["Q3,FY2023"], "mine": []}

  query 4: How much was the Cost of Goods Sold for the PEKB mine in the third quarter of 2023?
  entity 4: {"Indicator":["Cost of goods sold(COGS)"],"metrics":[],"time_frame":["Q3,FY2023"],"mine": ["PEKB"]}

  **Output:**
  {{
    "mismatches": {"Which mine had the highest EBITDA Coal Block in 2024?": ["Cost of Goods Sold","Q3","2023","PEKB"],
    "What is the variance in EBITDA margin across the four sites for current month?": ["Cost of Goods Sold","Q3","2023","PEKB"],
    "Which mine had the highest Cost of Goods Sold in Q3 of 2023?": ["highest"],
    "How much was the Cost of Goods Sold for the PEKB mine in the third quarter of 2023?": []
  }}

  ## Instructions:
  1. Always follow the above output format.
  2. Do not include any explanations or justifications.
  3. Do not include any additional text.
  4. For a given query, if no mismatches are found return an empty list for only that particular query.
     Eg. 
     {{"mismatches": 
     {
       "query one.": [],
      "query two": []
      }
    }}
  ''',


  "COT_REFINER_PROMPT_CHASE":'''
  The Schema Structure is as follows: 

    [Table Catalog:<catalog that the table belongs to> 

    Table Schema: <schema that the table belongs to> 

    Table Name: <table name> 

    Table Descriptions: <table description> 

    [(<column_name>,<column_datatype>,<column_description>,<column_example_values>)], 

    [Additional Information: <additional information about the table and rules to follow (consider additional information as the table specific rules to follow)>]] 

##Remember each table schema is separated by '**************************************' 

## Ensure schema descriptions are distinct per table; do not mix schemas. 

##your output should be short and crisp (e.g., avoid conversational filler) 

  

**CORE SQL RULES:** 

- **ALWAYS FOLLOW CATALOG_NAME.SCHEMA_NAME.TABLE_NAME** 

- **SQL must run in Spark Databricks (3.0+ compatible)** 

- **Column names should include units.** 

- **Always use LOWER() for string comparisons.** 

- **Adhere to all provided table-specific rules and additional information.** 

  

You are a highly skilled **SQL Candidate Generator and Refiner Agent**. Your task is to first generate *multiple, diverse candidate SQL queries* based on the provided initial query, and then **meticulously refine each of these candidates** for quality, correctness, and adherence to best practices, all without actual execution. 

  

**Instructions:** 

1.  **Understand Inputs:** Analyze the natural language question, the detailed database schema, any retrieved values, and the initial SQL query provided by the **Divide and Conquer agent**. 

2.  **Generate Diverse Candidates (2-3):** 

    * Based on the core logic and intent of the Divide and Conquer SQL query, generate **2 to 3 distinct candidate SQL queries**. 

    * Explore **alternative valid constructions** to achieve the same or a similar result (e.g., different join types, subquery vs. CTE, alternative filtering methods, different aggregation functions if applicable, different ways to express dates/periods). 

    * Ensure each initially generated candidate conceptually adheres to the schema, respects data types, and broadly follows SQL syntax and the CORE SQL RULES. 

3.  **Refine Each Candidate:** 

    * For **EACH** generated candidate query, critically review it. 

    * **Identify any potential logical flaws, semantic inaccuracies, or areas for improvement.** This includes (but is not limited to): 

        * Ensuring precise adherence to the user's intent. 

        * Optimizing for clarity, readability, and efficiency (e.g., better joins, more precise filtering, clearer column aliases, or ensuring all CORE SQL RULES are strictly met). 

        * Correcting any misinterpretations of the schema or rules. 

    * **Refine the query** to enhance its correctness, readability, and efficiency. 

    * Ensure the *refined* query is syntactically valid and logically consistent with the original natural language question and the database schema. 

4.  **Output Format:** Present your output by clearly listing each *final, refined* candidate query. For each refined candidate, provide a brief explanation outlining its original generation reasoning and any specific improvements made during the refinement step. 

  

**Use all provided rules (CORE SQL RULES, Schema Structure, Additional Information) to strictly guide your generation and refinement.** 

  

--- 
##You need to give the evidences of the table and the column used in the SQL ((table1:col1,col2),(table2:col1,col4))
**Output Format:** 

Refined Candidate 1:  

```sql  

<SQL query 1 - after generation AND refinement> 
<Evidence of the table and the column used>

``` 

Refined Candidate 1 Explanation: <Brief explanation of why this query was generated (alternative logic) and what specific refinements were applied.> 

  

Refined Candidate 2:  

```sql  

<SQL query 2 - after generation AND refinement> 
<Evidence of the table and the column used>

``` 

Refined Candidate 2 Explanation: <Brief explanation of why this query was generated (alternative logic) and what specific refinements were applied.> 



Refined Candidate 3:  

```sql  

<SQL query 3 - after generation AND refinement> 
<Evidence of the table and the column used>

``` 

Refined Candidate 3 Explanation: <Brief explanation of why this query was generated (alternative logic) and what specific refinements were applied.> 

 --- 

 

Now, generate and refine: 

Divide and Conquer approach: 

{} 

Values Retrieved: 

{} 

Database Schema: 

{} 

User Input: 

{} 

  ''',
  "INTENT_CLASSIFICATION_PROMPT_COMMON":'''
  You are an expert Intent Classifier
  You will be given the user query and the domains, your task is to map the user query to the appropriate domain/domains as per the query intent.
  ## Instructions:
  1. Read the user query and the domains.
  2. Identify the intent of the user query.
  3. If the query involves topics like revenue, profit, loss, expenses, cost components (fixed/variable), or financial performance metrics, capital expenditures, cash flow activities from operating , investing and financing,any costs such as logistics costs or any operation related costs, — classify it under **Finance**.
  4. If the query relates to employees, workforce, departments, headcount, HR costs, or staffing issues — classify it under **HR**.
  5. If the query relates to sales, marketing, customer dispatch, production, operations efficiency, or daily KPIs — classify it under **Operations**.
  6. Map the intent to the appropriate domain.
  8. Output the results in the following format:
  ['domain1','domain2']

  Available domains are: ['Finance', 'Operations', 'HR']

  Below are the examples for the user query and corresponding intent.
  Example1: "Has there been an increase in OB rehandling cost this quarter?"
  Intent: Finance
  Example2: "What is the actual and budgeted diesel cost incurred for OB + Subgrade removal in the last month?"
  Intent: Finance
  Example3: "What are the company's main products or services?"
  Intent: Operations

  User Query:
  {}

  ''',
    'VALUE_RETRIEVER_PROMPT_CHASE':''' 
    The Schema Structure is as follows:
    [Table Catalog:<catalog that the table belongs to>
    Table Schema: <schema that the table belongs to>
    Table Name: <table name>
    Table Descriptions: <table description>
    [(<column_name>,<column_datatype>,<column_description>,<column_example_values>)],
    [Additional Information: <additional information about the table and rules to follow (consider additional information as the table specific rules to follow)>]]
    ##Remember each table schema is separed by '**************************************'
## Do not mix the schemas of different tables, ensure that the schema descriptions 
    ##your output should be short and crisp
You are a value retrieval assistant. Your task is to help with identifying and retrieving relevant values from a database schema based on a user’s question.

Instructions:
**Use the provided rules and additional information in the Schema Structure to guide your generation.**
1. Read the following natural language question and the accompanying database schema.
2. Extract all key keywords that might refer to table names, column names, or literal values within the database. These keywords might be subject to typos or slight spelling variations.
3. For each extracted keyword:
   - Use techniques similar to locality-sensitive hashing (LSH) to generate a list of candidate values from the database schema that are syntactically similar.
   - Re-rank these candidates based on semantic embedding similarity and edit distance to ensure robustness against typos.
4. Output the results in the following JSON format:
{{
  "keyword": "<extracted keyword>",
  "candidates": [
      {{"value": "<candidate value>", "similarity_score": <score>}},
      ...
  ]
}}

Example:
User Question: "Show me all employes working in NewYork"
Extracted Keywords might include: "employes" and "NewYork".  
For "NewYork", candidates might include "New York", "NewYork", etc., along with their similarity scores.

Please now process the following input:
Database Schema:
{}
User Input:
{}

'''
,
'GENERATOR_DIVIDE_AND_CONQUER_PROMPT_CHASE' :'''
The Schema Structure is as follows:
    [Table Catalog:<catalog that the table belongs to>
    Table Schema: <schema that the table belongs to>
    Table Name: <table name>
    Table Descriptions: <table description>
    [(<column_name>,<column_datatype>,<column_description>,<column_example_values>)],
    [Additional Information: <additional information about the table and rules to follow (consider additional information as the table specific rules to follow)>]]
    ##Remember each table schema is separed by '**************************************'
  
## Do not mix the schemas of different tables, ensure that the schema descriptions are kept distinct for each table
##your output should be short and crisp
**ALWAYS FOLLOW CATALOG_NAME.SCHEMA_NAME.TABLE_NAME**
**sql should run in a spark databricks environment**
**write a sql that is compatible with spark 3.0 and above**
**column names should have units**
**Always do string comparisons using lower**
    You are an expert SQL generator that uses a divide-and-conquer strategy to handle complex questions.
**Use the provided rules and additional information in the Schema Structure to guide your generation.**
Your task is to generate a correct SQL query by following these steps:

### Step 1: Divide
- Analyze the provided natural language question and identify its distinct sub-problems. Each sub-problem should reflect a component of the final SQL query (e.g., filtering conditions, join operations, aggregations).
- Express each sub-problem as a pseudo-SQL query that outlines the intended operation.
**Use the provided rules and additional information in the Schema Structure to guide your generation.**

### Step 2: Conquer
- For each sub-problem, generate a corresponding partial SQL query that addresses that specific aspect.
- Ensure each partial query is syntactically valid and fits within the provided database schema.
**Use the provided rules and additional information in the Schema Structure to guide your generation.**
### Step 3: Assemble
- Combine all the partial SQL queries into one final SQL query.
- Optimize the final SQL query by eliminating any redundant clauses and ensuring logical coherence.

Finally, provide a brief explanation of how the sub-problems contributed to the final solution.

Example:
Question: "Find the average salary of employees in the Sales department who joined after 2015."
- Sub-problem 1: Identify employees in the Sales department.
- Sub-problem 2: Filter those who joined after 2015.
- Sub-problem 3: Compute the average salary.

Now, please process the following:
Values Retrieved:
{}
Database Schema:
{}
User Input:
{}

''',
'GENERATOR_COT_PROMPT_CHASE' :'''
The Schema Structure is as follows:
    [Table Catalog:<catalog that the table belongs to>
    Table Schema: <schema that the table belongs to>
    Table Name: <table name>
    Table Descriptions: <table description>
    [(<column_name>,<column_datatype>,<column_description>,<column_example_values>)],
    [Additional Information: <additional information about the table and rules to follow (consider additional information as the table specific rules to follow)>]]
    ##Remember each table schema is separed by '**************************************'
## Do not mix the schemas of different tables, ensure that the schema descriptions are kept distinct for each table
##your output should be short and crisp
**ALWAYS FOLLOW CATALOG_NAME.SCHEMA_NAME.TABLE_NAME**
**column names should have units**
**sql should run in a spark databricks environment**
**write a sql that is compatible with spark 3.0 and above**
**Always do string comparisons using lower**
You are the Candidate Generator Agent. With the contextual information provided (including the retrieved values, database metadata, and schema details), your task is to generate multiple candidate SQL queries that could address the user's request. Please:
- Create at least three candidate SQL queries.
- Ensure that each query adheres to the schema, respects data types, and follows SQL syntax.
- Optionally, include a brief explanation for each candidate query to outline your reasoning behind its construction.
**Use the provided rules and additional information in the Schema Structure to guide your generation.**
Your output should clearly list the candidate queries, each separated by a header or bullet point.
Divide and Conquer approach:
{}
Values Retrieved:
{}
Database Schema:
{}
User Input:
{}
''',
'QUERY_REFINER_PROMPT_CHASE' :'''
The Schema Structure is as follows:
    [Table Catalog:<catalog that the table belongs to>
    Table Schema: <schema that the table belongs to>
    Table Name: <table name>
    Table Descriptions: <table description>
    [(<column_name>,<column_datatype>,<column_description>,<column_example_values>)],
    [Additional Information: <additional information about the table and rules to follow (consider additional information as the table specific rules to follow)>]]
    ##Remember each table schema is separed by '**************************************'
## Do not mix the schemas of different tables, ensure that the schema descriptions are kept distinct for each table
##your output should be short and crisp
**ALWAYS FOLLOW CATALOG_NAME.SCHEMA_NAME.TABLE_NAME**
**sql should run in a spark databricks environment**
**write a sql that is compatible with spark 3.0 and above**
**column names should have units**
**Always do string comparisons using lower**
You are a SQL Query Fixer. Your task is to review a given SQL queries that may contain errors and iteratively fix it based on provided feedback.
**Use the provided rules and additional information in the Schema Structure to guide your generation.**
Instructions:
1. Read the provided SQL queries and the accompanying natural language question.
2. Identify the errors or issues in the SQL query.
3. Correct the query to fix these errors. If multiple errors exist, fix them one at a time and describe the changes you make.
4. After each correction, ensure the query is syntactically valid and logically consistent with the original natural language question.
5. Output the corrected SQL query and a brief explanation of your corrections.

Example:
Input Query: "SELECT name, age FROM Employees WHERE age > '30"
Feedback: "Syntax error: missing closing quote."
Corrected Query: "SELECT name, age FROM Employees WHERE age > 30"
Explanation: "Removed the unmatched quote and corrected the data type for the age filter."

Now, please review and fix the following query:
User Query:
{}
Schema:
{}
SQL Queries:
{}
Important Values Retrieved:
{}
''',
"SELECTION_AGENT_PROMPT_CHASE" :'''
The Schema Structure is as follows:
    [Table Catalog:<catalog that the table belongs to>
    Table Schema: <schema that the table belongs to>
    Table Name: <table name>
    Table Descriptions: <table description>
    [(<column_name>,<column_datatype>,<column_description>,<column_example_values>)],
    [Additional Information: <additional information about the table and rules to follow (consider additional information as the table specific rules to follow)>]]
    ##Remember each table schema is separed by '**************************************'
## Do not mix the schemas of different tables, ensure that the schema descriptions are kept distinct for each table
##your output should be short and crisp
**ALWAYS FOLLOW CATALOG_NAME.SCHEMA_NAME.TABLE_NAME**
**column names should have units**
**sql should run in a spark databricks environment**
**write a sql that is compatible with spark 3.0 and above**
**Always do string comparisons using lower**
You are a selection agent responsible for choosing the best SQL query from a pool of candidate queries.
**Use the provided rules and additional information in the Schema Structure to guide your generation.**
Instructions:
1. You are provided with:
   - A set of candidate SQL queries: C = {{c1, c2, ..., cn}}.
   - The target database schema (D).
   - Execution results for each candidate query.
2. For each distinct pair of candidate queries (ci and cj), perform the following:
   - If both queries produce the same execution result on the database, mark one as the winner arbitrarily.
   - If the execution results differ, create a union of the schemas used in both queries.
   - Evaluate which query better satisfies the natural language question by considering:
       * Relevance to the question.
       * Correct usage of the schema.
       * Overall logical output.
   - Use a binary comparison (as if using a fine-tuned classifier) to decide which query is more likely correct.
3. Tally the scores for each candidate based on pairwise wins.
4. Select the candidate query with the highest overall score as the final SQL query.
5. Provide the final selected SQL query along with a brief justification explaining why it was chosen over the others.

Output format:
Final Selected SQL Query: "<selected SQL query>"
Justification: "<explanation of the decision process>"

Example:
If comparing:
- Candidate 1: "SELECT name FROM Employees WHERE salary > 50000;"
- Candidate 2: "SELECT name, salary FROM Employees WHERE salary > 50000;"
And both return the same result, additional factors such as simplicity or specificity to the question should be considered.

Now, please process the following:

User Query:
{}
Database Schema:
{}
Candidate SQL Queries:
{}
Important Values Retrieved:
{}
''',
'SELECTOR_PROMPT_MAC' : '''
The Schema Structure is as follows:
    [Table Catalog:<catalog that the table belongs to>
    Table Schema: <schema that the table belongs to>
    Table Name: <table name>
    Table Descriptions: <table description>
    [(<column_name>,<column_datatype>,<column_description>,<column_example_values>)],
    [Additional Information: <additional information about the table and rules to follow (consider additional information as the table specific rules to follow)>]]
    ##Remember each table schema is separed by '**************************************'
## Do not mix the schemas of different tables, ensure that the schema descriptions are kept distinct for each table
            ##your output should be short and crisp
              You are an intelligent database assistant. Your job is to analyze a complex database schema and identify the most relevant tables, columns, and relationships required to answer a user's query. The goal is to keep the input concise by excluding irrelevant elements while ensuring no critical information is missed.
**Use the provided rules and additional information in the Schema Structure to guide your generation.**
              Database Schema:
              {}

              User Query:
              {}

              Instructions:
              1. Analyze the user's query to understand its requirements (e.g., filters, aggregations, joins, or groupings).
              2. Identify and list only the pertinent tables and columns based on the query.
              3. Highlight the relationships (e.g., primary-key/foreign-key links) between tables that are necessary for the query.
              4. Exclude irrelevant schema components to maintain efficiency.
              5. Provide the output in the following format:

              Output Format:
              Relevant Tables: [List relevant tables]
              Relevant Columns: [List relevant columns]
              Important Relationships: [List relationships between tables]


  ''',
  'DECOMPOSER_PROMPT_MAC' :'''
The Schema Structure is as follows:
    [Table Catalog:<catalog that the table belongs to>
    Table Schema: <schema that the table belongs to>
    Table Name: <table name>
    Table Descriptions: <table description>
    [(<column_name>,<column_datatype>,<column_description>,<column_example_values>)],
    [Additional Information: <additional information about the table and rules to follow (consider additional information as the table specific rules to follow)>]]
    ##Remember each table schema is separed by '**************************************'
## Do not mix the schemas of different tables, ensure that the schema descriptions are kept distinct for each table
                  ##your output should be short and crisp
                    **sql should run in a spark databricks environment**
                    **write a sql that is compatible with spark 3.0 and above**
                    **ALWAYS FOLLOW CATALOG_NAME.SCHEMA_NAME.TABLE_NAME**
                    **column names should have units**
                    **Always do string comparisons using lower**
                      You are an expert in decomposing complex natural language queries into smaller, manageable sub-questions to construct accurate SQL queries. Your role is to systematically refine the query into logical steps that can be individually solved and then combined into a final SQL query.
**Use the provided rules and additional information in the Schema Structure to guide your generation.**
                      ***
                      User Query:
                      {}
                      ***
                      Schema:
                      {}
                      Instructions:
                      1. Analyze the query to identify its key components, such as filters, conditions, aggregations, and relationships between tables.
                      2. Break down the query into a sequence of sub-questions that can be solved step-by-step.
                      3. Structure the steps so that each one logically builds on the previous.
                      4. Clearly explain how these steps will lead to the final SQL query.
                      5. Provide the output in the following format:

                      Output Format:
                      Logical Steps:
                      1. [First sub-question or step]
                      2. [Second sub-question or step]
                      3. [Third sub-question or step]
                      ...
  ''',
  "REFINER_PROMPT_MAC" :'''
The Schema Structure is as follows:
    [Table Catalog:<catalog that the table belongs to>
    Table Schema: <schema that the table belongs to>
    Table Name: <table name>
    Table Descriptions: <table description>
    [(<column_name>,<column_datatype>,<column_description>,<column_example_values>)],
    [Additional Information: <additional information about the table and rules to follow (consider additional information as the table specific rules to follow)>]]
    ##Remember each table schema is separed by '**************************************'
## Do not mix the schemas of different tables, ensure that the schema descriptions are kept distinct for each table
  ##your output should be short and crisp
        **sql should run in a spark databricks environment**
        **write a sql that is compatible with spark 3.0 and above**
        **do not give comments**
        **ALWAYS FOLLOW CATALOG_NAME.SCHEMA_NAME.TABLE_NAME**
        **column names should have units**
        **Always do string comparisons using lower**
          You are an expert SQL assistant tasked with validating, diagnosing, and refining SQL queries. Your goal is to ensure that the SQL query is syntactically correct, aligns with the database schema, and produces meaningful results.
        **Use the provided rules and additional information in the Schema Structure to guide your generation.**
          Schema:
          {schema}
          relevant schema details:
          {schema_dict}

          SQL Query steps :
          {decomposer}

          Instructions:
          1. convert the users query to lower case and compare the table values in lower case
          2. Validate the SQL query for:
            - Syntactic correctness.
            - Compatibility with the database schema (tables, columns, relationships).
            - Execution feasibility (e.g., it should not produce errors or empty results unless intended).
          3. If the query has errors:
            - Identify the issues (e.g., syntax errors, missing joins, incorrect column references).
            - Correct the query and explain the changes made.
          4. If the query is valid:
            - Confirm its validity and explain why no changes were needed.
          5. Re-evaluate the corrected query until it is error-free or the maximum number of iterations is reached.
          6. Provide the output in the following format:

          Output Format:
          Validation: [Validation result, e.g., "Query is valid" or "Query has errors."]
          Issues Detected: [List of detected issues, if any.]
          Corrections Made: [List of changes made, if any.]
          Refined SQL Query: [Final SQL query]
          {format_instructions}
  ''',
  'EXTRACT_QUERY_PROMPT_COMMON':'''

        Given the Following text return the final sql query in JSON format.
        **remember that the returned text should not be in markdown format only in a simple string **

        Example: 
        {{'final_sql_query':"SELECT * FROM cars where id = 10"}}

        Text:
        {}
        ''',
    'INFORMATION_RETRIEVER_PROMPT_CHESS':'''
    ##your output should be short and crisp
    You are an Information Retriever Agent. Your task is to analyze a natural language query and provide a structured, comprehensive response. To do this, follow these steps:
    **you do not have to give sql queries**

    Extract Keywords:
    Identify the primary keywords and key phrases from the input query that could be linked to elements within a database.

    Identify Entities:
    Based on the extracted keywords, determine which ones correspond to entities in the database. Match these keywords to similar database values by considering both literal and semantic similarities. For each match, note the corresponding column or field information.

    Gather Context:
    Retrieve additional contextual information from the database catalog. This may include schema metadata, column descriptions, extended column names, or value descriptions that help clarify the meaning of the entities.
    **Use the provided rules and additional information in the Schema Structure to guide your generation.**
    Inputs:
    User Input: 
    {}

    Database Schema:
    {}

    Output:
    Keywords:
    - [List of extracted keywords]

    Entities:
    - [List of identified entities along with corresponding columns or field matches]

    Context:
    - [Relevant contextual information from the database catalog]

    


    ''',
 "SCHEMA_SELECTOR_PROMPT_CHESS":''' 
The Schema Structure is as follows:
    [Table Catalog:<catalog that the table belongs to>
    Table Schema: <schema that the table belongs to>
    Table Name: <table name>
    Table Descriptions: <table description>
    [(<column_name>,<column_datatype>,<column_description>,<column_example_values>)],
    [Additional Information: <additional information about the table and rules to follow (consider additional information as the table specific rules to follow)>]]
    ##Remember each table schema is separed by '**************************************'
## Do not mix the schemas of different tables, ensure that the schema descriptions are kept distinct for each table
 ##your output should be short and crisp
        You are a Schema Selector Agent. Your objective is to streamline the provided database schema by identifying and returning only the tables and columns essential for constructing a correct SQL query to answer the given question. Follow these steps:

        Understand the Query:
        Carefully analyze the natural language question to determine the required data elements.

        Analyze the Schema:
        Review the provided schema, which includes a list of tables and their corresponding columns.

        Filter Irrelevant Elements:
        Identify and discard columns that do not pertain to the question's requirements.

        Select Necessary Tables:
        Determine which tables contain the relevant information needed to answer the query.

        Narrow Down Columns:
        Within the selected tables, choose only the columns that are critical for forming the SQL query.

        Provide a Streamlined Schema:
        Return the reduced schema in a structured format that includes only the necessary tables and their essential columns.
**Use the provided rules and additional information in the Schema Structure to guide your generation.**
        Output Format:
        Tables:
        - [Table1]: [ColumnA, ColumnB, ...]
        - [Table2]: [ColumnC, ColumnD, ...]
        ...

        User Query:
        {}

        Schema:

        {}
    ''',
    "CANDIDATE_GENERATOR_PROMPT_CHESS":'''
The Schema Structure is as follows:
    [Table Catalog:<catalog that the table belongs to>
    Table Schema: <schema that the table belongs to>
    Table Name: <table name>
    Table Descriptions: <table description>
    [(<column_name>,<column_datatype>,<column_description>,<column_example_values>)],
    [Additional Information: <additional information about the table and rules to follow (consider additional information as the table specific rules to follow)>]]
    ##Remember each table schema is separed by '**************************************'
## Do not mix the schemas of different tables, ensure that the schema descriptions are kept distinct for each table
    ##your output should be short and crisp
            **sql should run in a spark databricks environment**
            **ALWAYS FOLLOW CATALOG_NAME.SCHEMA_NAME.TABLE_NAME**
            **sql should run in a spark databricks environment**
            **write a sql that is compatible with spark 3.0 and above**
            **column names should have units**
            **Always do string comparisons using lower**
            You are a Candidate Generator Agent. Your task is to generate 3 candidate SQL queries that accurately answer the provided natural language question, using the given database schema and contextual information. Follow these steps:

            Analyze the Input:
            Carefully review the natural language question along with the database schema and any provided context (such as entities and their descriptions). Understand which data elements are needed to answer the question.

            Generate a Candidate Query:
            Formulate an initial SQL query based on your analysis. Ensure that the query logically corresponds to the schema and effectively incorporates the relevant context.

            Validate the Query:
            Evaluate the candidate query for any issues such as syntax errors or the possibility of returning an empty result. Consider how the query would execute against the database to check for correctness and completeness.

            Iterative Revision:
            If the candidate query is found to be faulty—either due to syntax errors or because it produces an empty result—revisit your reasoning and revise the query. Repeat this process as needed until the query is both syntactically correct and capable of returning meaningful data, or until a maximum number of revisions is reached.

            Provide the Final Query:
            Once a robust candidate query is produced, output the final SQL query as your answer.
**Use the provided rules and additional information in the Schema Structure to guide your generation.**
            Input:

            Question: 
            {}
            Schema: 
            {}
            Context (Entities and Descriptions): 
            {}
            Important Values Retrieved:
            {}
            Output Format:

            Candidate SQL Query:
            [CANDIDATE 1]
            [CANDIDATE 2]
            [CANDIDATE 3]
    ''',
    'TESTER_PROMPT_CHESS': '''
    The Schema Structure is as follows:
    [Table Catalog:<catalog that the table belongs to>
    Table Schema: <schema that the table belongs to>
    Table Name: <table name>
    Table Descriptions: <table description>
    [(<column_name>,<column_datatype>,<column_description>,<column_example_values>)],
    [Additional Information: <additional information about the table and rules to follow (consider additional information as the table specific rules to follow)>]]
    ##Remember each table schema is separed by '**************************************'
## Do not mix the schemas of different tables, ensure that the schema descriptions 
    ##your output should be short and crisp
        **sql should run in a spark databricks environment**
        **ALWAYS FOLLOW CATALOG_NAME.SCHEMA_NAME.TABLE_NAME**
        **sql should run in a spark databricks environment**
        **write a sql that is compatible with spark 3.0 and above**
        **column names should have units**
        **Always do string comparisons using lower**

        You are a Unit Tester Agent. Your task is to select the most accurate SQL query from a set of candidate queries based on how well they perform on a series of tests. Follow these steps:

        Generate Unit Tests:

        Develop a set of k unit tests (with k specified as needed) designed to highlight the semantic differences among the candidate queries.
        Each unit test should be crafted so that only a correct SQL query will successfully handle the scenario presented.
        Evaluate Candidate Queries:

        For each generated unit test, assess every candidate query to determine if it meets the test criteria.
        Carefully consider how each query behaves and whether it returns the expected results under the conditions described by the test.
        Scoring and Selection:

        Assign a score to each candidate query based on the number of unit tests it passes.
        Compare the scores, and select the candidate with the highest score as the final SQL query to answer the given question.
        If needed, use further reasoning to resolve any ties or close scores by considering the criticality of each test.
        Output the Final Result:

        Present the final selected SQL query along with a summary of the evaluation, including details on test outcomes and scores for transparency.

        **Use the provided rules and additional information in the Schema Structure to guide your generation.**
        Input Provided:

        User Query: 
        {}
        Candidates: 
        {}
        Schema: 
        {}
        Important Values Retrieved:
        {}

        Output Format Example:

        Final SQL Query:
        [The best candidate SQL query]
            ''',
"Operations_rules":
  '''
  #always use all the rules 

 

{     

  "rules": [     

    {     

      "rule_id": 1,     

      "title": "**Interpreting Year and Quarter References**",     

      "description": "Interpret all year and quarter references using the **indian financial year** format (**April to March**). NOTE: Calendar year concept is not applicable.",     

      "sub_rules": [     

        {"a": "**Vague Year References**: If a vague reference like 'last 2 years' is used and today is **May 2025**, interpret it as:"},     

        {"a1": "April 2023 to March 2024 (**FY23–24**)"},     

        {"a2": "April 2024 to March 2025 (**FY24–25**)"},     

        {"b": "**Vague Quarter References** (e.g., 'last 3 quarters' from May 2025):"},     

        {"b1": "Q4 **FY24–25**: Jan–Mar 2025"},     

        {"b2": "Q3 **FY24–25**: Oct–Dec 2024"},     

        {"b3": "Q2 **FY24–25**: Jul–Sep 2024"}     

      ]     

    },     

{    

      "rule_id": 2,    

      "title": "**Use of `sapbpc` Tables**",    

      "sub_rules": [    

        {"a": "Use `sapbpc` tables when the question asks for data **older than the current or previous month**."},    

        {"b": "`sapbpc` tables have **month-level granularity**."},    

        {"c": "Use them when data is requested for **monthly**, **yearly** periods."},    

        {"d": "Filtering Duration in `sapbpc` Tables"},    

        {"d1": "For **one or more months** or   **year to current month** or **till date**, filter with:"},    

        {"code_example": "```sql duration = 'FTM' ```"}   

              ]    

    },    

{    

      "rule_id": 3,    

      "title": "**Use of `mineshot` Tables**",    

      "sub_rules": [    

        {"a": "Use `mineshot` tables when the question asks for data **within the current or previous month**."},    

        {"b": "`mineshot` tables have **day-level granularity**."},    

        {"c": "Use them when the question refers to **specific days or weeks**."}   

      ]    

    },    

{    

      "rule_id": 4,    

      "title": "**Data Requested for Days or Weeks**",    

      "sub_rules": [    

        {"a": "Do **not** use **month-level** tables."},    

        {"b": "Do **not** use **MTD**, **FTM**, **YTD**, or **YTM** columns."},    

        {"c": "If table is **day-level**, filter by **exact days or week dates**."},    

        {"example": "If the user asks for data between **May 1, 2024** and **May 7, 2024**, filter rows where the date column is between '`2024-05-01`' and '`2024-05-07`'."}    

      ]    

    },    

{    

      "rule_id": 5,    

      "title": "**Filtering Timestamp Columns**",    

      "description": "Always use SQL functions to extract parts from timestamp columns:",    

      "code_example": " ```sql YEAR(timestamp_column), MONTH(timestamp_column), DATE(timestamp_column) ```",    

      "example": "For `2025-03-31T00:00:00.000+00:00`, use: ```sql YEAR('2025-03-31T00:00:00.000+00:00'), MONTH('2025-03-31T00:00:00.000+00:00') ```"      

    },    

{    

      "rule_id": 6, 

      "title": "**Filtering Timeframes in `sapbpc` Tables**",    

      "sub_rules": [ 

        {"a": "When filtering by **date, period, or time** columns in `sapbpc` tables:"},    

        {"a1": "Extract only **month and year** — avoid filtering by full date."},    

        {"a2": "Refer below for how to extract year/month from timestamps using SQL."},    

        {"code_example": " ```sql YEAR(timestamp_column), MONTH(timestamp_column), DATE(timestamp_column) ```"},    

        {"example": "For `2025-03-31T00:00:00.000+00:00`, use: ```sql YEAR('2025-03-31T00:00:00.000+00:00'), MONTH('2025-03-31T00:00:00.000+00:00') ```"}   

      ]    

    },    

{    

      "rule_id": 7,    

      "title": "**Capturing **Mine Name** in Coal Mining Datasets**",    

      "sub_rules": [    

        {"a": "Always include the mine name column in the output or filtering if applicable. (e.g., `dr_mine_name`, `DR_Mine_Name`, `Mine_name`, `mine_name`, `Dr_Mine_Name`, `Dr_Mine`)."},    

        {"b": "Exclude values like `'All'`, `'All Sites'` using wildcards like `%all%`, `%All%`."},    

        {"c": "Include only **valid mine names** from the data. (e.g., `talabira`, `suliyari`, `pekb`, `gp3`)."}    

      ]    

    },    

{    

      "rule_id": 8,    

      "title": "**Including Relevant Columns in Output**",    

      "description": "Always include columns that are used in the **WHERE clause** in the **SELECT clause** as well.",    

      "columns": ["`attribute`", "`dr_mine_name`", "`DR_Mine_Name`", "`Mine_name`", "`mine_name`", "`Dr_Mine_Name`", "`Dr_Mine`", "`sub_attribute`", "`sub_attributes`", "`particulars`", "`Period`", "`Duration`", "`month`", "`year`"]    

    },    

 {    

      "rule_id": 9,    

      "title": "**Comparing Highest or Lowest Metric Values Across Mines**",    

      "sub_rules": [    

        {"a": "When asked to identify the mine with the **highest** or **lowest** value of a metric over a **year** or a **specific quarter**, **do not mix** metric values from different `duration` types (`YTM`, `FTM`)."},    

        {"b": "**Filter and use only the `FTM` (For-The-Month) values** to aggregate metric data for each mine across the requested timeframe (year or quarter)."},    

        {"c": "Aggregate the filtered `FTM` metric values **within each mine** by summing or applying the required aggregation based on the question."},    

        {"d": "After aggregation, **compare the aggregated values across all mines** to determine the mine with the highest or lowest metric value."},    

        {"e": "This approach aligns with the existing rule of **Do Not Aggregate Metric Columns for Different Values of `duration`**"}    

      ]    

    } , 

{    

      "rule_id": 10,    

      "title": "**Rules for Determining the Time Period**",    

      "sub_rules": [    

        {"a": "If a question does not explicitly mention a lower bound for the time period, consider the start of the current  financial year (beginning April 1st) up to the specified date in the question. Example: For the question "What is the Cash flow from Investing Activities till August 2024?", interpret the period as April 2024 to August 2024. "},    

        {"b": "`If no time period is mentioned at all in the question, default to using only yesterday's date as the time reference. example: For the question "Till now how much taxes have been paid and compare with current budget?", interpret the period as yesterday’s date only. "},    

       ]    

    },    

{     

      "rule_id": 11,     

      "title": "**Consider the Absolute Variance Rule**",     

      "sub_rules": [     

        {"a": "If a question involves identifying the highest or lowest variance, always consider the absolute value of the variance first. Then, sort the results in descending order to find the highest variance or in ascending order to find the lowest variance."},     

       ]    

    },   

  ]    

} 
  ''',
"HR_rules":
  '''Placeholder for hr''',

"Finance_rules" :
"""
##Always use all the rules 
### Instructions: 

- Analyze  the user's query. 

- Select the rules which are applicable to the user's query. 

- Append the rules to the user's query . Do not modify the user query just append applicable rules to it.  

- Return the modified query. 

- If no rules are applicable, return the original query. 

 

{    

  "rules": [    

    {    

      "rule_id": 1,    

      "title": "**Interpreting Year and Quarter References**",    

      "description": "Interpret all year and quarter references using the **indian financial year** format (**April to March**). NOTE: Calendar year concept is not applicable.",    

      "sub_rules": [    

        {"a": "**Vague Year References**: If a vague reference like 'last 2 years' is used and today is **May 2025**, interpret it as:"},    

        {"a1": "April 2023 to March 2024 (**FY23–24**)"},    

        {"a2": "April 2024 to March 2025 (**FY24–25**)"},    

        {"b": "**Vague Quarter References** (e.g., 'last 3 quarters' from May 2025):"},    

        {"b1": "Q4 **FY24–25**: Jan–Mar 2025"},    

        {"b2": "Q3 **FY24–25**: Oct–Dec 2024"},    

        {"b3": "Q2 **FY24–25**: Jul–Sep 2024"}    

      ]    

    },    

    {    

      "rule_id": 2,    

      "title": "**Use of `sapbpc` Tables**",    

      "sub_rules": [    

        {"a": "Use `sapbpc` tables when the question asks for data **older than the current or previous month**."},    

        {"b": "`sapbpc` tables have **month-level granularity**."},    

        {"c": "Use them when data is requested for **monthly**, **yearly** periods."},    

        {"d": "Filtering Duration in `sapbpc` Tables"},    

        {"d1": "For **one or more months** or   **year to current month** or **till date**, filter with:"},    

        {"code_example": "```sql duration = 'FTM' ```"}   

              ]    

    },    

    {    

      "rule_id": 3,    

      "title": "**Use of `mineshot` Tables**",    

      "sub_rules": [    

        {"a": "Use `mineshot` tables when the question asks for data **within the current or previous month**."},    

        {"b": "`mineshot` tables have **day-level granularity**."},    

        {"c": "Use them when the question refers to **specific days or weeks**."}   

      ]    

    },    

    {    

      "rule_id": 4,    

      "title": "**Data Requested for Days or Weeks**",    

      "sub_rules": [    

        {"a": "Do **not** use **month-level** tables."},    

        {"b": "Do **not** use **MTD**, **FTM**, **YTD**, or **YTM** columns."},    

        {"c": "If table is **day-level**, filter by **exact days or week dates**."},    

        {"example": "If the user asks for data between **May 1, 2024** and **May 7, 2024**, filter rows where the date column is between '`2024-05-01`' and '`2024-05-07`'."}    

      ]    

    },    

    {    

      "rule_id": 5,    

      "title": "**Filtering Timestamp Columns**",    

      "description": "Always use SQL functions to extract parts from timestamp columns:",    

      "code_example": " ```sql YEAR(timestamp_column), MONTH(timestamp_column), DATE(timestamp_column) ```",    

      "example": "For `2025-03-31T00:00:00.000+00:00`, use: ```sql YEAR('2025-03-31T00:00:00.000+00:00'), MONTH('2025-03-31T00:00:00.000+00:00') ```"      

    },    

    {    

      "rule_id": 6, 

      "title": "**Filtering Timeframes in `sapbpc` Tables**",    

      "sub_rules": [ 

        {"a": "When filtering by **date, period, or time** columns in `sapbpc` tables:"},    

        {"a1": "Extract only **month and year** — avoid filtering by full date."},    

        {"a2": "Refer below for how to extract year/month from timestamps using SQL."},    

        {"code_example": " ```sql YEAR(timestamp_column), MONTH(timestamp_column), DATE(timestamp_column) ```"},    

        {"example": "For `2025-03-31T00:00:00.000+00:00`, use: ```sql YEAR('2025-03-31T00:00:00.000+00:00'), MONTH('2025-03-31T00:00:00.000+00:00') ```"}   

      ]    

    },    

    {    

      "rule_id": 7,    

      "title": "**Capturing **Mine Name** in Coal Mining Datasets**",    

      "sub_rules": [    

        {"a": "Always include the mine name column in the output or filtering if applicable. (e.g., `dr_mine_name`, `DR_Mine_Name`, `Mine_name`, `mine_name`, `Dr_Mine_Name`, `Dr_Mine`)."},    

        {"b": "Exclude values like `'All'`, `'All Sites'` using wildcards like `%all%`, `%All%`."},    

        {"c": "Include only **valid mine names** from the data. (e.g., `talabira`, `suliyari`, `pekb`, `gp3`)."}    

      ]    

    },    

    {    

      "rule_id": 8,    

      "title": "**Including Relevant Columns in Output**",    

      "description": "Always include columns that are used in the **WHERE clause** in the **SELECT clause** as well.",    

      "columns": ["`attribute`", "`dr_mine_name`", "`DR_Mine_Name`", "`Mine_name`", "`mine_name`", "`Dr_Mine_Name`", "`Dr_Mine`", "`sub_attribute`", "`sub_attributes`", "`particulars`", "`Period`", "`Duration`", "`month`", "`year`"]    

    },    

    {    

      "rule_id": 9,    

      "title": "**Do Not Aggregate Metric Columns for Different Values of duration**",    

      "sub_rules": [    

        {"a": "Never sum or aggregate metric columns (like `period_actual`, `period_budget`, etc.) when the `duration` column contains **multiple types of durations**."},    

        {"b": "**Example**: If `duration` has both `YTM` (Year-to-Month) and `FTM` (For-the-Month), do **not** aggregate their metric values."},    

        {"c": "Aggregation is valid **only when** the `duration` value is the same across rows."}    

      ]    

    },    

    {    

      "rule_id": 10,    

      "title": "**Comparing Highest or Lowest Metric Values Across Mines**",    

      "sub_rules": [    

        {"a": "When asked to identify the mine with the **highest** or **lowest** value of a metric over a **year** or a **specific quarter**, **do not mix** metric values from different `duration` types (`YTM`, `FTM`)."},    

        {"b": "**Filter and use only the `FTM` (For-The-Month) values** to aggregate metric data for each mine across the requested timeframe (year or quarter)."},    

        {"c": "Aggregate the filtered `FTM` metric values **within each mine** by summing or applying the required aggregation based on the question."},    

        {"d": "After aggregation, **compare the aggregated values across all mines** to determine the mine with the highest or lowest metric value."},    

        {"e": "This approach aligns with the existing rule of **Do Not Aggregate Metric Columns for Different Values of `duration`**"}    

      ]    

    } , 

{    

      "rule_id": 11,    

      "title": "**Rules for Determining the Time Period**",    

      "sub_rules": [    

        {"a": "If a question does not explicitly mention a lower bound for the time period, consider the start of the current  financial year (beginning April 1st) up to the specified date in the question. Example: For the question "What is the Cash flow from Investing Activities till August 2024?", interpret the period as April 2024 to August 2024. "},    

        {"b": "`If no time period is mentioned at all in the question, default to using only yesterday's date as the time reference. example: For the question "Till now how much taxes have been paid and compare with current budget?", interpret the period as yesterday’s date only. "},    

       ]    

    },      

{     

      "rule_id": 12,     

      "title": "**Consider the Absolute Variance Rule**",     

      "sub_rules": [     

        {"a": "If a question involves identifying the highest or lowest variance, always consider the absolute value of the variance first. Then, sort the results in descending order to find the highest variance or in ascending order to find the lowest variance."},     

       ]    

    },     

  ]    

} 

 """,
"SQL_FIXER_PROMPT_COMMON":
 '''
 You are an expert in fixing SQL Queries.
 You will be provided with a sql query, the database schema and the error message, your task is to return the fixed SQL query.
**sql should run in a spark databricks environment**
**ALWAYS FOLLOW CATALOG_NAME.SCHEMA_NAME.TABLE_NAME**
**sql should run in a spark databricks environment**
**write a sql that is compatible with spark 3.0 and above**
**Always do string comparisons using lower**
##**JUST RETURN THE SQL QUERY IN THE FORMAT INSTRUCTED AND NOTHING ELSE**
User Query:
{}
Schema:
{}
SQL Query:
{}
Error Message:
{}
##Output Format:
{{'final_sql_query':"SELECT * FROM cars where id = 10"}}


 ''',
      "custom_table_flag":0,
      "custom_table_name":["mineshot_ct_operation_summary_report","sapbpc_finance_summary_report","hr_employees_hired"]

}
