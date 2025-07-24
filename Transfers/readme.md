
#1. Instead of hard coding, can we use something like this:
 
relevant_query = []
relevant_sql_query = []
description_sql = []
relevant_table = []
relevant_mschema = []
 
for item in result["result"]["data_array"]:
    if item["similarity_score"] > config["QQA_THRESHOLD"]:
        relevant_query.append(item["question"].strip())
        formatted_sql = item["place_holder_query"].format(**eval(item["place_holder_query_values"]))
        relevant_sql_query.append(formatted_sql)
        description_sql.append(item["query_Description"])
        relevant_table.append(item["table_names"])
        relevant_mschema.append(item["mschema_descriptions"])
 
#2. Test for scenarios where sql_query has join of multiple tables. In that case, mschema should correspond to those tables and it should work.
 
#3. Check below prompt once:
 
"ENTITY_MATCH_MISMATCH": '''
You are an expert in the mining domain. Your task is to:
1. Extract key entities from the given `user_query` and `similar_query(s)`.
2. Compare each `similar_query` against the `user_query` to find **only the entities that are present in the user query but missing in the similar query**. This is calculated as:  
**entities(user_query) − entities(similar_query)**
 
---
 
Key entities include:
- **KPIs** (e.g., ROM, Stripping Ratio, OB Removal)
- **Mine/site names** (e.g., GP3, Talabira, PEKB, all sites)
- **Time references** (e.g., "last month", "FY23 Q4")
- **Other domain-specific entities or filters**
 
---
 
**KPI Mismatch Logic:**
You must also determine the **KPI mismatch** flag for each similar query.
 
This check looks for KPIs in the similar query that are **not present in the user query**. 
It is directional and should be computed as:  
`KPIs(similar_query) − KPIs(user_query)`
 
- If the `similar_query` contains **any KPI not present** in the `user_query`, return `"kpi_mismatch": 1`
- If all KPIs in the `similar_query` **are present** in the `user_query`, return `"kpi_mismatch": 0`
 
---
 
**Instructions:** 
- DO NOT modify or rephrase any query text in the output.
- The `<similar query>` keys must be exact string matches—no edits or paraphrasing.
- Return only entities missing **from the similar query** in the `mismatch_list`.
- Return `kpi_mismatch: 1` only if there are **extra KPIs** in the similar query not found in the user query.
- If no mismatches exist, return:
  ```json
  "mismatch_list": [],
  "kpi_mismatch": 0
  ```
- Do NOT include explanations, comments, or extra fields in the output.
 
---
 
**Output format:**
{{
  "mismatches": {{
    "<similar query 1 (as is, no modification)>": {{
      "mismatch_list": ["..."],
      "kpi_mismatch": 0 or 1
    }},
	...
  }}
}}
 
**Example:** 
user_query: "Provide ROM and OB removal data for Talabira and PEKB sites for last quarter"
 
similar_queries:  
1. "ROM and OB data for Talabira"  
2. "Stripping Ratio for PEKB and GP3 last quarter"
3. "ROM and OB removal data for Talabira and PEKB in the last quarter"
 
**Output**:
```json
{{
  "mismatches": {{
    "ROM and OB data for Talabira": {{
      "mismatch_list": ["PEKB", "last quarter"],
      "kpi_mismatch": 0
    }},
    "Stripping Ratio for PEKB and GP3 last quarter": {{
      "mismatch_list": ["ROM", "OB removal", "Talabira"],
      "kpi_mismatch": 1
    }},
	"ROM and OB removal data for Talabira and PEKB in the last quarter": {{
      "mismatch_list": [],
      "kpi_mismatch": 0
    }}
  }}
}}
```
 
Now, process the following:
 
User query:
{user_query}
 
Similar queries:
{similar_queries}
 
KPI examples by domain:
{kpi_examples}
'''
 
#4. can we make it more explicit as below:
class entity_mismatch(BaseModel):
    mismatches: Dict[str, Dict[str, List[str]]] = Field(description= """A mapping of query to a dictionary with two keys -
                                                            1. 'mismatch_list' — a list of entities that are present in the user query but missing in the similar query. 
                                                            2. 'kpi_mismatch' — set to 1 if any KPI in the similar query is missing from the user query; otherwise, set to 0.""")
#5. if mismatch gets All / All Sites / Other Sites, can we think to handle this?
 
#6. in "has_any_date" function see if these changes makes sense:
 
fy_pattern = r"\b(?:fy[\s\-]?)?\d{2,4}[-–/]\d{2,4}\b" # FY 2023-24, FY2024/25, FY-24–25, 2024-2025 (including spaces and dashes)
qtr_pattern = r"\b(?:qtr|quarter)[\s\-–]*[1-4]\b|\bq[1-4]\b" # "Q1", "Quarter 3", "Qtr-4" (including spaces and dashes)
 
compact_fy_quarter_pattern = r'\b(fy[\s\-]?\d{2,4}[\s\-]?q[1-4]|q[1-4][\s\-]?fy[\s\-]?\d{2,4})\b' #FY24Q4, Q3FY25, FY2025Q1 (including spaces and dashes)
 
# march–july, april to june, nov–dec (including spaces and dashes)
month_range_pattern = r"\b(?:jan(?:uary)?|feb(?:ruary)?|mar(?:ch)?|apr(?:il)?|may|jun(?:e)?|jul(?:y)?|aug(?:ust)?|sep(?:tember)?|oct(?:ober)?|nov(?:ember)?|dec(?:ember)?)[\s\-–to]+(?:jan(?:uary)?|feb(?:ruary)?|mar(?:ch)?|apr(?:il)?|may|jun(?:e)?|jul(?:y)?|aug(?:ust)?|sep(?:tember)?|oct(?:ober)?|nov(?:ember)?|dec(?:ember)?)\b"
 
 
ambiguous_date_keywords = [
    "end of year", "quarter end", "year to date", "month to date",
    "weekly summary", "monthly report", "this month", "next month",
    "week ending", "month ending", "first half", "second half",
    "current fiscal", "last fiscal", "H1", "H2", "YTD", "MTD",
	"current financial", "last financial", "YTM", "FTM", "FTD"
]
#7. REFINER_PROMPT ->> check below simplified version
 
 
"REFINER_PROMPT": '''
You are an expert SQL refiner.
 
You will be provided with the following inputs:
- A natural language **User Query**
- A list of **Similar Queries**
- Their corresponding **SQL Queries**
- Their respective **Descriptions**
- The associated **Schema**
- A dictionary of **Mismatched Entities**, structured as:
  {{mismatched_entities: 
	{{"<similar_query>":[<list_of_mismatched_entities>]}}
  }}
 
## Objective:
**Refine** the provided SQL query or queries to answer the User Query by making **minimal adjustments only**. Your changes must be restricted to filtering for **mine names** and **date ranges**—if this alone is insufficient, return:
```json
{ "final_sql_query": "N" }
```
 
## Rules:
 
1. You may adjust:
   - `WHERE` clauses for **mine names** or **time frames**
 
2. You must return `{ "final_sql_query": "N" }` if:
   - The query is about a **different KPI**
   - There are **more than two mismatched entities**
   - Any mismatches involve KPI-specific entities
   - Answering the query requires changes beyond **mine/date filters**
   - The required columns to be modified include:
     - `"attribute"`, `"sub_attribute"`, `"sub_attributes"`, `"sub_attribute_new"`,
     - `"sub_attribute_split"`, `"sub_attribute_level_1"`, `"sub_attribute_level_2"`, `"sub_attribute_level_3"`, or `"particulars"`
 
3. You must not:
   - Invent new columns, filters, or logic
   - Modify existing logic beyond what is shown
   - Use tables not found in the provided schema
 
If you cannot produce an accurate SQL query using only acceptable minimal changes, return:
```json
{ "final_sql_query": "N" }
```
 
**Output**:
```json
{
  "final_sql_query": "<refined SQL or 'N'>"
}
```
 
## Now, process the following:
 
User Query: 
{refiner_input}
'''
