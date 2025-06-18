
prompt = 

You are an expert in the mining industry, who can understand the query intent and can extract key entities from a given user query like key KPIs, mine names (GP3, Suliyari, Talabira, PEKB, all sites, etc.), time periods, etc.
  For a given user_query, give the entities that do not match the given query-entity pairs
  
  user query: 
  {user_query}

  query-entity pairs:
  {similar_queries_and_entities}

  Give me the output in the following format {format_instructions}:

  {{
    "mismatches": {{"query1": ["mismatched_entity_1", "mismatched_entity_2"],
                   "query2": ["mismatched_entity_2", "mismatched_entity_3"],
                   "query3": ["mismatched_entity_3", "mismatched_entity_4"]}}
  }}

  ## Examples:
  **Input:**
  user query: what is the Cost of Goods Sold in Q3 of 2023 for PEKB mine?
  
  query-entity pairs:
  query 1:  Which mine had the highest EBITDA Coal Block in 2024?
  entity 1: {{"Indicator":["EBITDA coal block"],"metric":["highest"], "time_frame":["FY2024"], "mine": []}}

  query 2: What is the variance in EBITDA margin across the four sites for current month?
  entity 2: {{"Indicator":["EBITDA margin"], "metrics":["variance"], "time_frame":["current month"], "mine": ["four sites"]}}

  query 3: Which mine had the highest Cost of Goods Sold in Q3 of 2023?
  entity 3: {{"Indicator":["Cost of goods sold(COGS)"],"metrics":["highest"],"time_frame":["Q3,FY2023"], "mine": []}}

  query 4: How much was the Cost of Goods Sold for the PEKB mine in the third quarter of 2023?
  entity 4: {{"Indicator":["Cost of goods sold(COGS)"],"metrics":[],"time_frame":["Q3,FY2023"],"mine": ["PEKB"]}}

  **Output:**
  {{
    "mismatches": {{"Which mine had the highest EBITDA Coal Block in 2024?": ["Cost of Goods Sold","Q3","2023","PEKB"],
    "What is the variance in EBITDA margin across the four sites for current month?": ["Cost of Goods Sold","Q3","2023","PEKB"],
    "Which mine had the highest Cost of Goods Sold in Q3 of 2023?": ["highest"],
    "How much was the Cost of Goods Sold for the PEKB mine in the third quarter of 2023?": []}}
  }}

  ## Instructions:
  1. Always follow the above output format.
  2. Do not include any explanations or justifications.
  3. Do not include any additional text.
  4. For a given query, if no mismatches are found return an empty list for only that particular query.
     Eg. 
     {{"mismatches": 
     {{
       "query one.": [],
      "query two": []
      }}
    }}



  class entity_mismatch(BaseModel):
    mismatches: Dict[str, List[str]] = Field(description= "A mapping of query to their list of entities that do not match the user_query entities")
  json_parser = JsonOutputParser(pydantic_object = entity_mismatch)
  get_entity_mismatches_prompt = PromptTemplate(template = prompt,
                                                 input_variables = [state["task"], similar_queries_and_entities],
                                                 partial_variables = {"format_instructions": json_parser.get_format_instructions()})
  chain = get_entity_mismatches_prompt | llm | json_parser
  output = chain.invoke({"text":state["task"]})
