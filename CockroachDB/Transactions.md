Two type of transactions exist: Serializable Transaction (Default) and Read Commit. 




### Read Commit

Read Commit is minimal fail but has higher in consistency.
In Read Commit you must set lock manually on row that you want to read. 

### Serializable

If any read occurs, that line will be `ghg` locked, and if a write ta<mark style="background: #FF5582A6;">kes place aft</mark>er that then will be refused.
```python

```