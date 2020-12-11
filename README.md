# AMLVSDBX
Comparison between Azure Machine Learning (AML) and Databricks (DBx). All tasks in AML are done in a non-distrbuted fashion and written in Python. The ones in DBx are written in Scala. 

Comparisons that are made:
- Read data from Azure Data Lake Gen2 (ADLS2), add three columns (concat, calculate and if then else), write data back
- Read data from ADLS2, groupby and aggregate, plot
- Read data from ADLS2, create ML model

Fake test data was generated with Python's Faker package: 1GB, 5GB, 10GB, 50GB, 100GB. Files are written in chunks of approx. 100MB and are CSV's. They are stored in separate folders in a container (no ADLS2 premium)

AML setup
- Small Machine: D13_V2 (8cores, 56GB RAM, 400GB disk) -> $1.20/hour
- Medium Machine: D15_V2 (20cores, 140GB RAM, 1000GB disk) -> $2.79/hour
- Hulk Machine: D64_V3 (64cores, 256GB RAM, 1600GB disk) -> $6.68/hour

DBX setup
- Small cluster: 2 workers, DS3_V2 (4cores, 14GB RAM, 0.75DBU) -> $2.07/hour (3 * VM's * $0.28/hour + 3 * 0.75 DBU * $0.55/hour)
- Medium cluster: 4 workers, DS3_V2 (4cores, 14GB RAM, 0.75DBU) -> $3.46/hour (5 * VM's * $0.28/hour + 5 * 0.75 DBU * $0.55/hour)

Prices are calculated per fully used minute as described on https://azure.microsoft.com/en-us/pricing/calculator/.

## Comparison 1: Read, transform and write
| Data Size | AML Small | AML Medium | AML Hulk | DBX small | DBX medium |
|---|---|---|---|---|
| 1GB | 1 min 33 sec| 1 min 23 sec | 1 min 56 sec| 54 sec | 37.8 sec |
| 5GB | 7 min 21 sec | 6 min 40 sec | 7 min 24 sec | 3 min 10 sec | 1 min 40 sec |
| 10GB | failed | 14 min 35 sec | 12 min 40 sec | 3 min 1 sec | 2 min 17 sec |
| 50GB | failed | failed | failed | 14 min 24 sec | 8 min 40 sec |
| 100GB | failed | failed | failed | 29 min | 19 min 6 sec |

Error message for AML is Kernel died. After some investigation we found that it's not even loading the 50GBs with the standard method provided within AML.

## Comparison 2: Read, groupby country and calculate average salary
| Data Size | AML Small | AML Medium | AML Hulk | DBX small | DBX medium |
|---|---|---|---|---|
| 1GB | 16 sec | 16 sec | n/a | 31 sec | 24 sec |
| 5GB | 57 sec | 51sec | n/a | 55 sec | 27 sec |
| 10GB | 1 min 51 sec | 1 min 44 sec | n/a | 1 min 47 sec | 55 sec |
| 50GB | failed | failed | 10 min 16 sec | 8 min 10 sec | 3 min 56 sec |
| 100GB | failed | failed | failed | 21 min 2 sec | 8 min 36 sec |

We have also used pure python on DBX as we see this quite often in reality. Transforming 1GB took 1 minute and 44 seconds and 5GB failed already. The reason is quite simple: As soon as you start to use pure python, e.g. a pandas dataframe, everything gets loaded to the driver and hence isn't distributed anymore. 

## Comparison 3: Read, create ML model
| Data Size | AML Small | AML Medium | AML Hulk | DBX small | DBX medium |
|---|---|---|---|---|
| 1GB | failed | 3 min | 2 min 12 sec | 2 min | 3 min 48 sec |
| 5GB | failed | failed | failed | 8 min 12 sec | 4 min 18 sec |
| 10GB | failed | failed | failed | 16 min 42 sec | 8 min |
| 50GB | failed | failed | failed | 68 min | 33 min 30 sec |
| 100GB | failed | failed | failed | n/a | 76 min 12 sec |

The main problem for python and pandas is the one-hot encoding for countries. This creates a very big but sparse Matrix which is too big to load into memory. For this reason a simpler and less sparse matrix is used to train a model in the next step.

## Comparison 4: Read, create ML model with less onehot encodings
| Data Size | AML Small | AML Medium | AML Hulk | DBX small | DBX medium |
|---|---|---|---|---|
| 1GB | 30 sec | 36 sec | 23 sec | 24 sec | 1 min 48 sec |
| 5GB | 1 min 48 sec | 1 min 54 sec | 1 min 46 sec | 1 min 40 sec | 5 min 24 sec |
| 10GB | 3 min 30 sec | 4 min | 3 min 16 sec | 11 min 24 sec | 8 min 9 sec |
| 50GB | failed | failed | 18 min 22 sec | 47 min 56 sec | 20 min 42 sec |
| 100GB | failed | failed | failed | n/a |  |

- add Hulk

- add prices