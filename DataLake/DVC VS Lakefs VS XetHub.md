When considering machine learning workflows, managing different versions and copies of data incurs inherent costs when cleaning data, creating new features, and splitting the data for different datasets.
# Deduplication Approaches
- DVC (Data Version Control) is a free and open-source version system for data, machine learning models, and experiments which manage files with .dvc pointers which are managed by git while the file bytes are stored elsewhere.
- Git LFS Is an extension to git that enables git to version large files by replacing the files themselves with pointers.
- LakeFS is a DataLake middleware between your code and blob-store which implement Git operations on the data.

All of them implement file-level deduplication, which helps ensure that only one copy of the data is stored locally at any given point. However, it still requires you to store, upload, and download the entire file whenever there is a minor change (adding a new row or column).

This is where XetHub's block-level deduplication comes into play. It offers an alternative approach that minimizes storage and network costs by deduplicating blocks.

![[Pasted image 20240708084815.png]]

# Performance


![[Pasted image 20240708083929.png]]