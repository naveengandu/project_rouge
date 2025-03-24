# Rouge 

Objective: Learn and experiment with ML concepts and tools while leveraging my Data Engineering skills. 

Approach/Plan: 
1. Curate a dataset containing images corresponding to at least 10 different classes of rice leaf diseases from multiple open data sources (to improve robustness and eliminate bias). 
2. Ingest and organise this data onto a data lake storage (ADLSg2 for instance). 
3. Employ a transfer learning approach by using a pre-trained CNN model such as ResNet50 to train a CNN on the curated dataset using Azure ML to reliably identify and classify rice leaf diseases. 
4. Experiment, evaluate and iterate over the process to fine-tune the performance of the model. 
5. Deploy the model using a managed instance (a custom-defined instance would be the next step) to process real-time requests accessible via REST API.

Execution:
1. Ingested 3 datasets from Kaggle using Azure Databricks. The data was streamed into VM's memory and saved in ADLSg2. A 4th dataset was excluded because the size of the dataset was too high to ingest via streaming with the limitations of the free trial subscription (both Azure and Databricks).
2. The raw data, which was stored as 3 separate directories (1 for each dataset) contained sub-directories, each representing one class of disease. ADF was used to merge all of this data to obtain a single dataset with distinct classes of diseases, represented by their respective directories inside a staging container. 
