title: <this Sample of Filter Expressions (Python)> | Microsoft Docs
description: <this Gives samples of python filter expressions>
author: <your cforbe>
ms.author: <your cforbe@microsoft.com>
ms.date: <today’s date: 9/7/2017>

Before reading this appendix read [Python Extensibility Overview](data-prep-python-extensibility-overview.md)
# Sample of Filter Expressions (Python) #

Filter in only those rows the value of (numeric) Col2 is great than 4

```python
    row.Col2 > 4
```

Filter in only those rows where Col1 contains the value 'Good" and Col2 contains the (numeric) value 1
```python
    row.Col1 == 'Good' and row.Col2 == 1
```

Filter in only those rows where Col1 has a null value
```python
    pd.isnull(row.Col1)
```
