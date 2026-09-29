# AZURE_DATA_FACTORY_HTTP_TO_BRONZE_LAYER
<img width="1908" height="932" alt="Screenshot 2026-09-29 163641" src="https://github.com/user-attachments/assets/edaafdb7-e916-4991-a3dd-856873c38210" />
# Azure Data Factory HTTP to Bronze Layer Pipeline

## 📌 Project Overview

This project demonstrates an **Azure Data Engineering ingestion pipeline** built using **Azure Data Factory (ADF)**.

The pipeline extracts CSV datasets from a public HTTP/GitHub source and loads the raw data into the **Bronze layer of Azure Data Lake Storage Gen2 (ADLS Gen2)**.

The project also implements a **dynamic metadata-driven ingestion pipeline** using ADF parameters, Lookup activity, ForEach activity, and parameterized datasets. This allows multiple files to be ingested using a single reusable pipeline instead of creating a separate pipeline for every dataset.

---

## 🏗️ Architecture

```text
GitHub / HTTP Source
        │
        ▼
Azure Data Factory
        │
        ├── HTTP Linked Service
        │
        ├── Lookup Activity
        │
        ├── ForEach Activity
        │
        └── Copy Data Activity
        │
        ▼
Azure Data Lake Storage Gen2
        │
        ▼
     Bronze Layer
```

For the dynamic pipeline:

```text
Parameter JSON File
        │
        ▼
Lookup Activity
        │
        ▼
ForEach Activity
        │
        ▼
Dynamic HTTP Dataset
        │
        ▼
Copy Data Activity
        │
        ▼
Dynamic ADLS Dataset
        │
        ▼
Bronze Layer
```

---

## 🚀 Technologies Used

- Microsoft Azure
- Azure Data Factory
- Azure Data Lake Storage Gen2
- HTTP / REST-based data ingestion
- GitHub Raw Content
- JSON
- CSV
- ADF Parameters
- Lookup Activity
- ForEach Activity
- Copy Data Activity
- Dynamic Datasets

---

# 🔄 Pipeline Workflow

## 1. Source Data

The source files are hosted on GitHub and accessed through the raw GitHub content endpoint.

The HTTP Linked Service uses:

```text
https://raw.githubusercontent.com/
```

ADF then uses relative URLs to locate individual CSV files.

Example:

```text
prathambhavsar220-ai/Azure-data-set/refs/heads/main/AdventureWorks_Products.csv
```

---

## 2. HTTP Linked Service

The project contains an HTTP Linked Service:

```text
Httplinkedservice
```

It connects Azure Data Factory to:

```text
https://raw.githubusercontent.com/
```

Authentication is configured as:

```text
Anonymous
```

Since the datasets are publicly available, authentication credentials are not required.

---

## 3. Azure Data Lake Linked Service

The second Linked Service connects Azure Data Factory to **Azure Data Lake Storage Gen2**.

```text
storagelake
```

This connection allows ADF to write the ingested datasets into the Data Lake.

The project primarily stores raw data inside the:

```text
bronze
```

container.

---

# 📥 Basic HTTP to Bronze Pipeline

The project contains the pipeline:

```text
git_to_raw
```

This pipeline demonstrates a simple ingestion workflow.

```text
GitHub CSV
     │
     ▼
HTTP Dataset
     │
     ▼
Copy Data Activity
     │
     ▼
ADLS Gen2
     │
     ▼
Bronze Layer
```

The pipeline uses the `dshttp` dataset as the source and `dsraw` as the destination dataset.

---

## Source Dataset

```text
dshttp
```

The dataset reads the AdventureWorks product CSV file from GitHub.

It uses:

```text
Httplinkedservice
```

and reads:

```text
AdventureWorks_Products.csv
```

as a delimited CSV file.

---

## Copy Data Activity

The Copy Data activity transfers the dataset from the HTTP source to Azure Data Lake Storage.

The source configuration uses:

```text
DelimitedTextSource
HttpReadSettings
GET Request
```

The sink configuration uses:

```text
DelimitedTextSink
AzureBlobFSWriteSettings
```

---

## Bronze Dataset

The destination dataset is:

```text
dsraw
```

The data is stored inside ADLS Gen2 using the structure:

```text
bronze/
└── products/
    └── products.csv
```

The Bronze layer keeps the ingested data close to its original source format.

---

# ⚙️ Dynamic Metadata-Driven Pipeline

The repository also contains a more scalable pipeline:

```text
dynamic_piprline
```

Instead of creating separate Copy activities for every source file, this pipeline uses metadata and parameters to dynamically ingest multiple datasets.

The workflow is:

```text
git.json
   │
   ▼
Lookup
   │
   ▼
ForEach
   │
   ▼
Dynamic Copy Activity
   │
   ├── Dynamic Source URL
   │
   └── Dynamic Destination Path
   │
   ▼
Bronze Layer
```

---

# 🔎 Lookup Activity

The first activity in the dynamic pipeline is:

```text
Lookup
```

The Lookup activity reads configuration information from:

```text
git.json
```

The configuration file is stored in the ADLS Gen2 filesystem:

```text
parameters
```

The dataset used to access the configuration is:

```text
lookup_Json
```

The Lookup activity is configured with:

```text
firstRowOnly = false
```

This allows it to return multiple configuration records.

Each record can define information such as:

```text
p_relative_url
p_sink_folder
p_file_name
```

These values are later passed to the ForEach activity.

---

# 🔁 ForEach Activity

After the Lookup activity succeeds, the output is passed to the ForEach activity.

The ForEach expression is:

```text
@activity('Lookup').output.value
```

This means ADF loops through every configuration record returned by the Lookup activity.

The pipeline currently executes the loop sequentially.

Inside the ForEach activity is:

```text
dynamic_copy
```

---

# 📤 Dynamic Copy Activity

The `dynamic_copy` activity performs the actual ingestion.

For every record returned from the configuration file, ADF dynamically determines:

- Source URL
- Destination folder
- Destination filename

This makes the pipeline reusable for multiple datasets.

---

# 🔗 Dynamic Source Dataset

The dynamic HTTP source dataset is:

```text
dsgitdynamic
```

It contains the parameter:

```text
p_relative_url
```

The relative URL is dynamically generated using:

```text
@dataset().p_relative_url
```

Inside the Copy activity, its value comes from:

```text
@item().p_relative_url
```

Therefore, every iteration of the ForEach activity can read a different source file.

---

# 📂 Dynamic Sink Dataset

The dynamic destination dataset is:

```text
ds_dynamicsink
```

It contains two parameters:

```text
p_sink_folder
p_file_name
```

The folder path is dynamically generated using:

```text
@dataset().p_sink_folder
```

The filename is dynamically generated using:

```text
@dataset().p_file_name
```

Inside the Copy activity, the values come from:

```text
@item().p_sink_folder
```

and

```text
@item().p_file_name
```

This allows each dataset to be written to its own folder and file inside the Bronze container.

---

# 🧠 Why Use a Dynamic Pipeline?

A static approach would require creating a separate pipeline or Copy Data activity for every dataset.

For example:

```text
Products → Copy Activity 1
Customers → Copy Activity 2
Sales → Copy Activity 3
Employees → Copy Activity 4
```

This becomes difficult to maintain when the number of datasets increases.

The metadata-driven approach used in this project provides:

```text
Configuration
     │
     ▼
Lookup
     │
     ▼
ForEach
     │
     ▼
One Reusable Copy Activity
     │
     ▼
Multiple Bronze Datasets
```

To add another dataset, the configuration can be updated instead of redesigning the complete pipeline.

---

# 📁 Repository Structure

```text
AZURE_DATA_FACTORY_HTTP_TO_BRONZE_LAYER/
│
├── dataset/
│   ├── ds_dynamicsink.json
│   ├── dsgitdynamic.json
│   ├── dshttp.json
│   ├── dsraw.json
│   └── lookup_Json.json
│
├── factory/
│
├── linkedService/
│   ├── Httplinkedservice.json
│   └── storagelake.json
│
├── pipeline/
│   ├── dynamic_piprline.json
│   └── git_to_raw.json
│
├── publish_config.json
│
└── README.md
```

---

# 🥉 Bronze Layer

The Bronze layer represents the **raw ingestion layer** of the data architecture.

Its main responsibilities are:

- Store source data with minimal transformation
- Preserve the original data for reprocessing
- Separate ingestion from downstream transformations
- Provide a reliable historical landing area
- Support future Silver and Gold transformations

Example:

```text
External Source
      │
      ▼
   Bronze
      │
      ▼
   Silver
      │
      ▼
    Gold
```

This project currently focuses on the ingestion stage:

```text
HTTP / GitHub → Azure Data Factory → Bronze Layer
```

---

# ✨ Key Features

- HTTP-based ingestion using Azure Data Factory
- GitHub-hosted CSV ingestion
- Azure Data Lake Storage Gen2 integration
- Bronze-layer architecture
- Dynamic datasets
- Parameterized source URLs
- Parameterized destination folders
- Parameterized filenames
- Metadata-driven ingestion
- Lookup activity
- ForEach activity
- Reusable Copy Data activity
- Scalable pipeline design

---

# 🎯 Project Objective

The objective of this project was to understand how Azure Data Factory can be used to build scalable ingestion pipelines instead of manually creating pipelines for every data source.

The project demonstrates how:

```text
Parameters + Metadata + Lookup + ForEach + Copy Data
```

can be combined to create a reusable ingestion framework.

This is closer to how real-world Data Engineering pipelines handle multiple files and datasets.

---

# 💡 What I Learned

Through this project, I practiced:

- Creating HTTP Linked Services in Azure Data Factory
- Connecting ADF with Azure Data Lake Storage Gen2
- Creating source and sink datasets
- Ingesting CSV files from HTTP endpoints
- Using Copy Data activities
- Creating parameterized datasets
- Passing parameters between pipeline activities and datasets
- Using Lookup activities
- Using ForEach loops
- Building dynamic source paths
- Building dynamic destination paths
- Designing metadata-driven pipelines
- Implementing Bronze-layer ingestion architecture

---

# 🔮 Future Improvements

The pipeline can be extended further by adding:

- Azure Databricks for data transformation
- Silver layer for cleaned and standardized data
- Gold layer for analytics-ready data
- Delta Lake
- Incremental data loading
- Watermark-based ingestion
- Error handling
- Retry mechanisms
- Logging and monitoring
- Email or Teams alerts
- ADF scheduling triggers
- Azure Key Vault for secret management
- CI/CD deployment using Azure DevOps or GitHub Actions

A future architecture could look like:

```text
HTTP / API / GitHub
        │
        ▼
Azure Data Factory
        │
        ▼
ADLS Gen2 - Bronze
        │
        ▼
Azure Databricks
        │
        ▼
Silver Delta Tables
        │
        ▼
Gold Analytics Tables
        │
        ▼
Power BI
```

---

# 👨‍💻 Author

**Pratham Bhavsar**

Data Engineering | Data Analytics | Azure

GitHub:  
https://github.com/prathambhavsar220-ai

---

## ⭐ Support

If you found this project useful, consider giving the repository a ⭐.

This project was created as part of my hands-on learning journey in **Azure Data Engineering and Azure Data Factory**.
