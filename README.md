# CDC2023_Business
Data analysis for the 2023 Carolina Data Challenge with the Business dataset

# Analysis Steps:
## 1. Data Importation
The code begins by importing necessary libraries and then uploading a CSV file containing the dataset. This is done using Google Colab's **files.upload()** method.

```python
import numpy as np
import matplotlib.pyplot as plt
from google.colab import files
import io
import pandas as pd
import re


data = files.upload()
```
## 2. 
