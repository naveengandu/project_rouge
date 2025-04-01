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
3. EfficientnetB0 (instead of Resnet50) was used to employ the trasfer learning approach due to limited compute resources available for training in the free trial. The following are the parameters set for training the model:
  - image size = 160 x 160 (optimal size for training which captures features to a reasonable degree while balancing training time and compute. 
  - batch size = 4 (lowest possible to optimize for available compute resources. 
  - epochs = 10 (can achieve reasonable accuracy and ensure no model overfitting. 
There were challenges associated with training the model and obtaining acceptable accuracy of the model. These are outlined below with measures taken to circumvent them:
  -  **Image files larger than 160 x 160** - The dataset contained images whose sizes and resolutions varied accross the datasets and are generally larger than the size specified for training (i.e. 160 x 160). Resizing the images to fit the specified size introduces errors as the features get distorted/get downsized drastically to be detected. In order to circumvent this issue, the images had to be cropped. Doing so at random defeats the purpose. Therefore, all images were pushed through a computer vision python module (cv2; opencv) to extract a feature map to obtain a 160 x 160 square of the original images which produced the highest score for saturation. Each image was then cropped according to the coordinates specified by the feature map and saved to a different location, which was then used for training. This drastically improved the performance and accuracy of the model. (Insert metrics here). 
