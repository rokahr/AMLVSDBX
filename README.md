# AMLVSDBX
Comparison between Azure Machine Learning (AML) and Databricks (DBx). All tasks in AML are done in a non-distrbuted fashion and written in Python. The ones in DBx are written in Scala. 

Comparisons that are made:
- Read data from Azure Data Lake Gen2 (ADLS2), add three columns (concat, calculate and if then else), write data back
- Read data from ADLS2, groupby and aggregate, plot
- Read data from ADLS2, create ML model, write ONNX to ADLS2

Fake test data was generated with Python's Faker package: 1GB, 5GB, 10GB, 50GB, 100GB. Files are written in chunks of approx. 100MB and are CSV's. They are stored in separate folders in a container (no ADLS2 premium)

AML setup
- Small Machine: D13_V2 (8cores, 56GB RAM, 400GB disk) -> $1.20/hour
- Medium Machine: D15_V2 (20cores, 140GB RAM, 1000GB disk) -> $2.79/hour

DBX setup
- Small cluster: 2 workers, DS3_V2 (4cores, 14GB RAM, 0.75DBU)
- Medium cluster: 4 workers, DS3_V2 (4cores, 14GB RAM, 0.75DBU)

## Comparison 1: Read, transform and write
| Data Size | AML Medium | AML Big | DBX small | DBX medium |
|---|---|---|---|---|
| 1GB |  |  |  |  |
| 5GB | 7.36 min |  | 3.17 min | 1.67 min |
| 10GB | failed | 14.58 min | - | 5.29 min |
| 50GB |  |  |  |  |
| 100GB |  |  |  |  |
