# Rouge

## Table of Contents

- [Overview](#overview)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Cloning the repository](#cloning-the-repository)
- [Reproduce the results](#reproduce-the-results)
- [Project Details](#project-details)
- [Future Work](#future-work)
- [Contributing](#contributing)
- [Contact](#contact)

***

## Overview

Rouge uses a Convolutional Neural Network (CNN) to classify images of rice leaves into various disease categories. The project leverages Jupyter Notebooks to explore, prototype, and evaluate the model. \

_**Note**: The project currently focuses on the data processing and model training aspects, future versions may include model deployment and API integrations._

***

## Repository Structure

The repository is organized into several folders and files to support different aspects of the project:

<details>
<summary>Repository Structure</summary>

```text
.
├── dataset/                                          # Config files (.json) for the Azure Dataset objects used in the Azure Data Factory (ADF) instance (rougedf2).
│   ├── raw_ds.json
│   ├── rouge_map_classes.json
│   └── stage_ds.json
│
├── factory/
│   └── rougedf2.json                                 # ADF instance config file
│
├── integrationRuntime/
│   └── rougeIR2.json                                 # Config file for the Integration Runtime used in the ADF instance.
│
├── linkedService/
│   └── rouge_adls2.json                              # Config file for the linked service object used in the ADF instance.
│
├── pipeline/
│   └── merge_dataset_classes.json                    # ADF pipeline used to merge the datasets.
│
├── rouge4_databricks/                                # Notebooks and configurations for running experiments on Databricks.
│   ├── create_ext_location.ipynb                     # Create an external location in Databricks which points to the dataset stored on the Data lake (ADLSg2). Execute after merging the dataset using the ADF pipeline "merge_dataset_classes".
│   ├── create_processkaggledata_vol.dbquery.ipynb    # Ignore this file.
│   ├── ingest kaggle datasets.ipynb                  # Ingest the datasets (stated in the notebook) from Kaggle onto ADLSg2. Execute first. Execute the ADF pipeline "merge_dataset_classes" next.
│   ├── prep_dataset.ipynb                            # Data augmentation and preprocessing steps. Execute after executing "create_ext_location.ipynb"
│   └── train_model.ipynb                             # Train the model. Execute last, i.e. after executing "prep_dataset.ipynb".
│
├── rougedf2/*                                        # ADF templates.
│
├── README.md                                         # README
│
├── publish_config.json                               # ADF linked publish branch config
│
└── rouge_classes.json                                # Config file that defines the classes for mapping. Required for merging the datasets. 

```
</details>

***


## Getting Started

### Prerequisites

- **Python 3.8+** (or later) is recommended.
- Active Azure subscription (Free trial or Pay-as-you-go is fine).
- Active Databricks subscription (standard tier works).

### Cloning the repository

```bash
git clone https://github.com/naveengandu/project_rouge.git
cd project_rouge
```

***

## Reproduce the results

### Azure Data Lake (ADLSg2):

- Create a resource group for the project.
- Create and initialize a storage account with default options and list the storage account under the resource group.
- Create a landing/raw container.
- Manually upload the `rouge_classes.json` file to the landing/raw container.
- Create a staging container.
- Create a processing/training container.

### Azure Databricks:

- Create and initialize an Azure Databricks workspace using default options, under the project's resource group. A free trial premium tier was used for this project.
- Toggle on git and link the workspace to the cloned repo.
- Navigate to rouge4_databricks to find all notebooks.
- Execute the notebooks in the following order:
  1. `ingest kaggle datasets.ipynb` (execute the ADF pipeline after this)
  2. `create_ext_location.ipynb` (execute after successfully running the ADF pipeline)
  3. `prep_dataset.ipynb`
  4. `train_model.ipynb`
- Dependicies, as they pertain to each notebook, are listed in the notebooks and can be installed at the time of execution. 
- More granular instructions can be found within each notebook, as it pertains to their execution.

### Azure Data Factory (ADF):
1. Initialize and set up
	- Create and initialize an ADF workspace under the project's resource group.
	- Navigate to the pipeline directory in the repo, download the config file and manually upload it to your workspace. Do not link ADF to this repo, link it to Azure DevOps, a 	fresh repo or other git services to implement version control, if necessary.
	- Create an Integration runtime with appropriate options.
	- Create a linked service between ADF and ADLSg2.
	- Create a JSON dataset to access `rouge_classes.json` from ADLSg2.
	- Create a binary dataset to access the images from the landing/raw container in ADLSg2. Refer to the raw_ds dataset. Note the dataset parameters!
	- Create a second binary dataset to access the staging container in ADLSg2. Refer to the stage_ds dataset. Note the dataset parameters!

2. Configuring the pipeline
   - Navigate to the author section in the ADF workspace.
   - Set the pipeline variables (src_container_name = landing/raw container & sink_container_name = staging container)
   - Under the Lookup activity's settings, select the JSON dataset created earlier as the source pointing to the `rouge_classes.json` file.
   - Navigate to the Copy activites nested in the IF condition activities inside the For loop activity, under source settings select "raw_ds" as the dataset and "stage_ds" as the dataset under sink settings.
   - Assign dynamic variables for the dataset properties. Refer to the pipeline config on how to set up properly.
    
3. Validate, Publish and Trigger/Run the pipeline.     

***

## Project Details

- **Data Ingestion & Processing**:
Data ingestion, preprocessing steps and data augmentation techniques are implemented within the notebooks. The dataset/ folder contains Azure Dataset objects (.json) used within the Azure Data Factory (ADF) instance in this project. 

- **Model Architecture**:
The CNN model is designed to capture features from rice leaf images and classify them into distinct categories based on disease symptoms. The model expects normalized square images (160 x 160 px). The image augmentation steps are implemented in the `prep_dataset.ipynb` notebook to ensure reproducibility. The image size can be tweaked if necessary. Detailed experiments and model evaluations are **not** documented in this repository.

- **Current Focus**:
The primary focus is on building an accurate classifier. As noted, deployment pipelines and API integrations are not covered in this version of the project.

***

## Future Work

- **Model Deployment**:
Plans to implement model serving and deployment pipelines (e.g., REST API integration) in future versions.

- **Optimization**:
Further optimize the model architecture with hyperparameter tuning and advanced techniques.

- **Enhanced Documentation**:
Expand tutorials and add usage examples to guide new contributors and users.

- **Additional Experiments**:
Investigate alternative deep learning architectures and training methodologies.

***

## Contributing

Contributions are welcome! To contribute:

1. Fork the Repository
  
2. Create a New Branch:

```bash
git checkout -b feature/your-feature-name
```

3. Commit Your Changes

```bash
git commit -am "Add feature or fix issue description"
```

4. Push the Branch and Open a Pull Request

For major changes, please open an issue first to discuss your ideas.

***

## Contact

For questions, suggestions, or issues, please open an issue in the repository or contact [Naveen Gandu](mailto:naveenkg676@gmail.com).

---


