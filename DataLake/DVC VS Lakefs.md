When considering machine learning workflows, managing different versions and copies of data incurs inherent costs when cleaning data, creating new features, and splitting the data for different datasets.
![[Pasted image 20240728103811.png|600]]

## Lakefs
LakeFS is a DataLake middleware between your code and blob-store which implement Git operations on the data.
- Provide S3 endpoint
- Create Bucket for each branch
- Save Diff or File Changes and metadata in postgres 
- Support action and pipeline
- Has well User Interface

![[Pasted image 20240728103045.png|600]]


## DVC
DVC (Data Version Control) is a free and open-source version system for data, machine learning models, and experiments which manage files with .dvc pointers
- Create clone of data on each `commit` and each `branch`
- Save metadata on Git as a file 
- Support pipeline 
- Support Model Registery
- Has VSCode extension
![[Pasted image 20240728104925.png|600]]

# Performance
I start with a single 8 GB file (dataset) and then repeat the following 10 times:
- I carry out a simulated iteration of feature engineering by adding random new columns to the dataset
- I commit and push the changes to the central repository

![[Pasted image 20240708083929.png|600]]
