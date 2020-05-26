## Background Information

A large company, Company A, provides health insurance to its employees. Every four years, Company A’s insurer, InsurAHealth, reviews the health status of the employees. To do this, InsurAHealth calculates a health score between 0 and 6 for each employee on a quarterly basis.
   • 0 denotes a very healthy person, and 6 denotes a very sick person. 
   • The ‘health score’ is a proprietary tool used by InsurAHealth. The items that go into its formula are not public.

This past review cycle InsurAHealth claimed that the employees have gotten sicker. 
   • Mean Health Score in Quarter 1 was 3.4, in Quarter 6 it was 3.5, and Quarter 12 it was 3.9.

Company A has hired you to evaluate InsurAHealth's claim that employees are sicker. To facilitate your analysis, InsurAHealth has provided you with data for 12 quarters that includes 2,000 employees from Company A. 
   • Each quarter is a representative sample of the employees at Company A in that quarter.
   • The demographic information included in this data is not part of InsurAHealth's health score calculation."


## Understanding the Data

```markdown
import datetime as dt
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.dates as mdates
import seaborn as sns

df = pd.read_excel("Insurance_Exercise.xlsx", sheet_name="Data")
print('columns: ', list(df.columns.values))
print('shape: ', df.shape) # 19103 rows, 9 columns

df.describe()
```
```markdown
def find_missing_employees(lst): 
    return [x for x in range(lst[0], lst[-1]+1)  
                               if x not in lst] 
l = df[""Employee Id""].tolist()
missing_employee_IDs = find_missing_employees(l)
print(len(missing_employee_IDs)) # returns 38
```
```markdown
n_observations_per_quarter = df[""Quarter""].value_counts()
n_observations_per_quarter.sort_index(inplace=True)
n_observations_per_quarter.plot.bar(figsize=(10,5), color='lightslategray')
plt.title(""Total Observations Per Quarter"")
plt.xticks(rotation=0)
plt.xlabel(""Quarter"")
plt.ylabel('Number of Observations')
plt.savefig('num_observations_per_qtr.png')
plt.show()
plt.close()
```

### Missing Data
```markdown
df_nan = df[df.isnull().any(axis=1)]
race_nan = df_nan[df_nan['Race'].isnull() == True] # all rows with NaN for Race
race_emps = race_nan['Employee Id'].drop_duplicates().values.tolist()
df_race = df[df['Employee Id'].isin(race_emps)]
np.nanmax(df_race['Race'].values) # check to see if these 220 employees have a non-NaN value for Race in any other row
np.nanmin(df_race['Race'].values) # check to see if these 220 employees have a non-NaN value for Race in any other row

```
```markdown
sex_nan = df_nan[df_nan['Sex (Male=1)'].isnull() == True] # all rows with NaN for Sex 
sex_emps = sex_nan['Employee Id'].drop_duplicates().values.tolist()
df_sex = df[df['Employee Id'].isin(sex_emps)]
np.nanmax(df_sex['Sex (Male=1)'].values) # check to see if these 7 employees have a non-NaN value for Sex in any other row
np.nanmin(df_sex['Sex (Male=1)'].values) # check to see if these 7 employees have a non-NaN value for Sex in any other row
```

### Employee Characteristics
```markdown
sex_12q = df.groupby(['Quarter'])['Sex (Male=1)'].value_counts(normalize=True) * 100
sex_12q.unstack().plot.bar(figsize=(10,5), colors=('pink','chocolate')).set_title('Sex of Employees Over Time')
plt.legend(labels=('female', 'male'), bbox_to_anchor=(1.0, 0.8))
plt.xticks(rotation = 0)
plt.xlabel('Quarter')
plt.ylabel('Percentage of Sample Population')
plt.savefig('sex_time.png')
plt.show()
plt.close()

print(sex_12q)
```
```markdown
"def is_outlier(points, thresh=3.5):
    """"""
    Returns a boolean array with True if points are outliers and False 
    Otherwise.

    References:
    ----------
        Boris Iglewicz and David Hoaglin (1993), ""Volume 16: How to Detect and
        Handle Outliers"", The ASQC Basic References in Quality Control:
        Statistical Techniques, Edward F. Mykytka, Ph.D., Editor. 
    """"""
    if len(points.shape) == 1:
        points = points[:,None]
    median = np.median(points, axis=0)
    diff = np.sum((points - median)**2, axis=-1)
    diff = np.sqrt(diff)
    med_abs_deviation = np.median(diff)

    modified_z_score = 0.6745 * diff / med_abs_deviation

    return modified_z_score > thresh
```
```markdown
"fig, ax3 = plt.subplots(figsize=(10,5))
ax3.hist(filtered, density=True, color='steelblue')
plt.ylabel(""Frequency"")
plt.xlabel(""Age"")
plt.grid(b=True, which='major', color='#666666', linestyle='-')
plt.title('Age of Employees in Quarter 12')
plt.savefig('age_q12_filtered.png')
plt.show()
plt.close()"
```

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/fionaroni/insurance/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.



