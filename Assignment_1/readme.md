## Setup instructions ####
1. Create azure keyvault.
    --> az keyvault create --name dipanjans-kv --resource-group Tredence-Batch1.
2. Create OpenAI service:
        --> az cognitiveservices account create \
                --name dipanjan-oai \
                --resource-group Tredence-Batch1 \
                --location eastus \
                --kind OpenAI \
                --sku s0
3. Deploy GPT 4o model:
        Create azure openai resources- GPT-4o
        --> az cognitiveservices account deployment create \
            --name dipanjan-oai \
            --resource-group Tredence-Batch1 \
            --deployment-name gpt-4o-mini  \
            --model-name gpt-4o-mini \
            --model-version "2024-07-18" \
            --model-format OpenAI \
            --sku-capacity 1 \
            --sku-name Standard
4. Deploy langfuse locally / host it in cloud:
        PS. Since ACIs are being deleted by your team on a regular basis, deploying it locally.
        - download the docker-compose.yaml file from https://github.com/langfuse/langfuse
        - docker-compose -up
        - Get sk and pk from langfuse UI.

5. Create and push secrets to keyvault , install pip dependencies:
        - push_secrets_and_pip_dependencies.ipynb (run all)

## Sample command to run the chain.
get_company_report('Bajaj financial services') --> Takes company name as input and generates a report in json. 
The json is also saved in the format -> sample_output_json_<company_name>.json 

-- This function can be independently be served if put in a py file and containerised / with a fast api type endpoint

