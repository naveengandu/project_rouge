# Rouge 

**Objective:** Learn and experiment with ML concepts and tools while leveraging my Data Engineering skills. 


**Approach/Plan:**
1. Curate a dataset containing images corresponding to at least 10 different classes of rice leaf diseases from multiple open data sources (to improve robustness and eliminate bias). 
2. Ingest and organise this data onto a data lake storage (ADLSg2 for instance). 
3. Employ a transfer learning approach by using a pre-trained CNN model such as ResNet50 to train a CNN on the curated dataset using Azure ML to reliably identify and classify rice leaf diseases. 
4. Experiment, evaluate and iterate over the process to fine-tune the performance of the model. 
5. Deploy the model using a managed instance (a custom-defined instance would be the next step) to process real-time requests accessible via REST API.


**Execution:**
1. Ingested 3 datasets from Kaggle using Azure Databricks. The data was streamed into VM's memory and saved in ADLSg2. A 4th dataset was excluded because the size of the dataset was too high to ingest via streaming with the limitations of the free trial subscription (both Azure and Databricks).
2. The raw data, which was stored as 3 separate directories (1 for each dataset) contained sub-directories, each representing one class of disease. ADF was used to merge all of this data to obtain a single dataset with distinct classes of diseases, represented by their respective directories inside a staging container.
3. The images were then processed for training within Databricks, which primarily involved cropping the images. 
4. EfficientnetB0 (instead of Resnet50) was used to employ the trasfer learning approach due to limited compute resources available for training in the free trial.



**Training parameters:**
1. base model = EfficientNetB0
2. image size = 160 x 160 (optimal size for training which captures features to a reasonable degree while balancing training time and compute.
3. batch size = 4
4. epochs = 10 (can achieve reasonable accuracy)




**Challenges:**

_Image files larger than 160 x 160_

The problem
   - The dataset contained images whose sizes and resolutions varied accross the datasets and are generally larger than the size specified for training (i.e. 160 x 160).
   - Resizing the images to fit the specified size introduces errors as the features get distorted/get downsized drastically to be detected.

The task
   - The images had to be cropped but doing so at random defeats the purpose.

The action
   - The images were converted to a HSV color space representation (using the cv2 python module), followed by parsing the image with a window size of 160 x 160 and a sliding step size/stride of 32 to compute the array sums of each window to identify feature-dense regions / regions of interest (ROIs). 
   - Each image was then cropped according to the coordinates of the window which yielded the highest sum and saved to a different location, which was then used for training.

The result
   - This drastically improved the performance (65%) and accuracy of the model (training accuracy: 37.7% -> 93.3% ; validation accuracy: 39.9% -> 91.2%).
   - In addition, the validation accuracy increased with every iteration/epoch, which is a good indicator that the model isn't overfitting.



  
**Limitations, reflections and future scopes:**

1. Although the HSV-based feature extraction method improved the performance and accuracy of the model, it is not the most optimal solution. For instance, from a random check in the 'brownspot' class, it is clear that this method didn't extract the ROIs for a small portion of the images. This might've affected the model's accuracy to a certain degree. There are 2 ways to circumvent this particular issue:
   1. Experiment with varying parameters for the feature extraction process, i.e. using different window sizes and stride/sliding step sizes. From my limited experimentation, I observed that the model performed better, in terms of extracting the features, when the window size was larger. However, I had compute limitations to consider. 
   2. Use another filtering method in tandem with HSV or try setting thresholds for the hue values (not tested).

2. The size of the dataset used is relatively small. I had to discard a dataset I originally intended to use for this project due to compute limitations. Perhaps, there're ways to circumvent the compute limitations for ingesting a larger dataset (at least an order of magnitude larger than the largest dataset I've used), but I haven't fully explored this aspect. Having a larger dataset introduces diversity to the image sources and thereby the robustness of the model.

3. This model has not been deployed, for instance to an online endpoint, as a result I can't verify the accuracy of the model manually. I'd like to explore this domain in the future once my understanding of the underlying concepts and familiarity with the upstream tools is more solidified.  
