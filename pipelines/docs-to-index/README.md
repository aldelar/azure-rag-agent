# Azure ML enrichment pipeline for a RAG Agent in Azure

Load generic documents (PDF, images, Word, PowerPoint, Excel, HTML, etc.) into an AI search index with contextual chunking and vectorization supported by the following services:
- Azure Docuemnt Intelligence
- Azure OpenAI
- Azure AI Search
- Azure Machine Learning

# Project intent

This is an [Azure ML CLIv2](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-configure-cli?view=azureml-api-2&tabs=public) template project demonstrating the use of the parallel job type to do large scale documents processing.

We here have a simple pipeline with 2 steps:
- doc2json:
	- input: folder containing a set of documents to process
	- output: one json file per document, containing the extracted text and metadata
- create-index:
	- output: log file with the index creation status
- json2idx:
	- inputs:
	  - log_file of create-index step to introduce a dependency between the 2 steps
	  - folder of json files
	- output: a search index containing the extracted text and metadata

![pipeline](doc/pipeline.png)

Each step is configured with a different level of parallelism, and could potentially use a different compute type/scale. See the control parameters for retries, timeouts, etc. in the pipeline.yml file.

For more details check [How to use parallel job in pipeline (V2)](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-use-parallel-job-in-pipeline?tabs=cliv2)

## Services Setup

To run this project, you need to have an Azure subscription and the following services setup:
- Azure ML
- Azure OpenAI
- Azure AI Search

The credentials of these services are assumed to be present in the default Azure Key Vault of the Azure ML Workspace. See codebase for keys used to locate these secrets.

## Local Environment Setup

1. [Install the Azure CLI + the Azure CLI 'ml' extension](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-configure-cli?tabs=public)

2. Login to your Azure subscription:
```bash
az login
```

3. Setup your azure CLI defaults to connect to your Azure ML workspace:
```bash
az configure --defaults group=<resource-group> workspace=<workspace-name> region=<region>
```

4. Rename the following file '.azureml/config.json.template' to '.azureml/config.json' and fill in the values with your environment values. This will enable the code to run locally and connect to the AML workspace to retrieve its default key vault and the necessary secrets.

5. Setup the following secrets in your Azure Key Vault:
```bash
AZURE-AI-SEARCH-ADMIN-API-KEY
AZURE-AI-SEARCH-ENDPOINT
AZURE-DOCUMENT-INTELLIGENCE-ENDPOINT
AZURE-DOCUMENT-INTELLIGENCE-KEY
AZURE-OPENAI-API-KEY
AZURE-OPENAI-API-VERSION
AZURE-OPENAI-EMBEDDING-MODEL-NAME
AZURE-OPENAI-ENDPOINT
```

## How to create and run the pipeline

Run the following commands to create the steps runtime environment (repeat this any time you need to modify the 'conda' files in each step folder to support code changes).

Create the 2 environments supporting each pipeline step (we could have used a single environment for both steps but it is best practice to have a specific runtime per task/step as they tend to eventually diverge and use different packages/versions; if these environments happened to be the same, it would be detected and would only create one backend image anyway):
```
az ml environment create -f doc2json/environment.yml
az ml environment create -f json2idx/environment.yml
```
To trigger a pipeline creation/update/run, execute the following:

```
az ml job create -f pipeline.yml
```
Then go to the Azure portal to monitor your pipeline.