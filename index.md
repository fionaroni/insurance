## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/fionaroni/insurance/edit/master/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

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
## Understanding the Data
Total quarters is 12.
Minimum Employee ID is 1, and maximum is 2000. It seems the intention was to have 1 observation per quarter for each employee ranging from Employee ID 1 to Employee ID 2000, totaling 24000 observations (2000 employees*12 observations).
Maximum value in Age column is 172, and the minimum value is 7. Both of these are too extreme, so there are some data-quality issues with this dataset.
We are assuming that the 1st quarter is the earliest point in time, and that 12th is the latest point in time. 
Race attribute doesn't make sense without the coding for what 1, 2, and 3 represent.

```markdown
def find_missing_employees(lst): 
    return [x for x in range(lst[0], lst[-1]+1)  
                               if x not in lst] 
l = df[""Employee Id""].tolist()
missing_employee_IDs = find_missing_employees(l)
print(len(missing_employee_IDs)) # returns 38
```

# Mean Health Scores Over Time
fig, ax8 = plt.subplots(1,1,figsize=(10,5))
sns.boxplot(x='Quarter', y='Health Score', data=df, ax=ax8, color='indianred').set_title('Health Scores Over Time')
means = df.groupby(['Quarter'])['Health Score'].mean().values # calculate means
mean_labels = [str(np.round(s, 2)) for s in means] # add mean labels
pos = range(len(means))
for tick, label in zip(pos, ax8.get_xticklabels()):
    ax8.text(pos[tick], means[tick] + 0.05, mean_labels[tick], horizontalalignment='center', size='large', color='k')
```

