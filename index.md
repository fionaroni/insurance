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

```python
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
```python
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
![Image](https://github.com/fionaroni/insurance/blob/master/num_observations_per_qtr.png)


### Missing Data
If we assume that the sex and race of each employee did not change over time, we can impute this missing data with values from other rows associated with the same employee. But because these employees are NaN for all observations, we cannot impute any missing values.

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

## Employee Characteristics

```markdown
# Sex Characteristic
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
![Image](https://github.com/fionaroni/insurance/blob/master/sex_time.png)

Sex of employees is roughly 50% male and 50% female throughout 12 quarters, with higher proportion of females (50.2%-51.5%) in Quarters 1 to 3 and higher proportion of males (50.6%-51.3%) in Quarters 4 to 12.

As we've observed from running df.describe(), age has outliers that are erroneous. Filter outliers using is_outlier() function.

```markdown
def is_outlier(points, thresh=3.5):
    """"""
    Returns a boolean array with True if points are outliers and False 
    Otherwise.
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
# Age histogram
fig, ax3 = plt.subplots(figsize=(10,5))
ax3.hist(filtered, density=True, color='steelblue')
plt.ylabel(""Frequency"")
plt.xlabel(""Age"")
plt.grid(b=True, which='major', color='#666666', linestyle='-')
plt.title('Age of Employees in Quarter 12')
plt.savefig('age_q12_filtered.png')
plt.show()
plt.close()
```
![Image](https://github.com/fionaroni/insurance/blob/master/age_q12_filtered.png)

```markdown
# Race over 12 Quarters 
race_12q = df.groupby(['Quarter'])['Race'].value_counts(normalize=True) * 100
race_12q.unstack().plot.bar(figsize=(10,5)).set_title('Race of Employees Over Time')
plt.ylabel('Percentage of Sample Population')
plt.legend(bbox_to_anchor=(1.0, 0.8))
plt.xticks(rotation = 0)
plt.savefig('race_time.png')
print(race_12q)
```
![Image](https://github.com/fionaroni/insurance/blob/master/race_time.png)

```markdown
# Salary Over 12 Quarters
fig, ax5 = plt.subplots(1,1,figsize=(10,5))
sns.set_style(""whitegrid"")
sns.boxplot(x=""Quarter"", y=""Salary"", data=df, color='olivedrab', ax=ax5).set_title('Salary of Employees Over 12 Quarters')
means = df.groupby(['Quarter'])['Salary'].mean().values.astype(int) # calculate means
mean_labels = [str(np.round(s, 2)) for s in means] # add median labels
pos = range(len(means))
for tick, label in zip(pos, ax5.get_xticklabels()):
    ax5.text(pos[tick], means[tick] + 0.1, mean_labels[tick], horizontalalignment='center', size='medium', color='w')
```
![Image](https://github.com/fionaroni/insurance/blob/master/salary_time.png)

## Relationship between Health Score and Employee Characteristics

```markdown
# Heatmap: All Attributes
fig = plt.figure(figsize=(10,5))
hm = df.loc[:,['Sex (Male=1)','Quarter','Salary','Age','Race','Health Score','Hospital Visit This Quarter (1=Yes)']]
hm.rename(columns={""Sex (Male=1)"":""Sex"",""Hospital Visit This Quarter (1=Yes)"":""Hospital Visit""}, inplace=True) # rename columns for easier-to-read display
ax7 = sns.heatmap(hm.corr(), linewidth=0.5, annot=True, cbar_kws={""label"": ""Co-occurrence frequency""})
ax7.set_title('Heatmap of All Attributes')  
ax7.set_xlabel('Attributes')  
ax7.set_ylabel('Attributes')  
plt.show()
```
![Image](https://github.com/fionaroni/insurance/blob/master/heatmap_all-attributes.png)

Salary and sex have a correlation of 0.47. Having a high salary and being male are positively correlated.
There is also a relatively high correlation, 0.48, between quarter and salary, indicating that employees have higher salaries in the latter part of the 3-year period.

There is a correlation of 0.19 between age and quarter, meaning that employees are older (as a whole) in the latter part of the 3-year period compared to earlier quarters.

Health score and sex are positively correlated (males are associated with higher health scores; 0.093 correlation).
Health score and age are positively correlated (older employees are associated with higher health scores; 0.16 correlation).
Health score and hospital visits are positively correlated (which makes sense- if you weren’t at least fairly sick, your health condition would not warrant a hospital visit; 0.13 correlation). Perhaps the presence of a health condition might prompt an employee to make a visit to the hospital in order to get well.
Health score and salary are positively correlated (as salaries of employees increase, health scores increase as well; 0.06 correlation).

Health score and race are negatively correlated (a race of 1.0 is associated with a higher health score, while a race of 3.0 is associated with a lower health score; -0.03 correlation).


```markdown
# Scatterplot: Age (filtered out outliers) and Health Score
sns.set(style=""darkgrid"")
fig, ax5 = plt.subplots(figsize=(10,5))
sns.regplot('Age','Health Score', data=age_df, scatter_kws={'s':2})
sns.despine()
```
![Image](https://github.com/fionaroni/insurance/blob/master/regplot_hs-age.png)

There is a slight positive slope. Age and Health Score are directly correlated. 
This might indicate that the older employees are less healthy.

```markdown
# Scatterplot: Salary and Health Score
sns.set(style=""darkgrid"")
fig, ax6 = plt.subplots(figsize=(10,5))
sns.regplot('Salary','Health Score', data=df, scatter_kws={'s':2}, color='olivedrab')
sns.despine()
```
![Image](https://github.com/fionaroni/insurance/blob/master/regplot_hs-salary.png)

There is a very slight positive slope. Salary and Health Score are directly correlated.
This might indicate that employees with higher salaries and are less healthy. Likely data quality issues with the anomalies at health score of 10.

```markdown
# Facet grid: Sex, Race, Hospital Visit, and Health Score
copy=df.copy()
copy['Sex (Male=1)'] = copy['Sex (Male=1)'].map({1: ""Male"", 0: ""Female""})
copy['Hospital Visit This Quarter (1=Yes)'] = copy['Hospital Visit This Quarter (1=Yes)'].map({1: ""Yes"", 0: ""No""})
copy.rename(columns={""Sex (Male=1)"":""Sex"", ""Hospital Visit This Quarter (1=Yes)"":""Hospital Visit""}, inplace=True)
sns.factorplot(x='Race', y='Health Score', row='Sex', col='Hospital Visit',
               kind='bar', data=copy, row_order=['Female','Male'], col_order=['Yes','No'])"
```
![Image](https://github.com/fionaroni/insurance/blob/master/facetgrid_health-score_race-hv-sex.png)

Males have a higher health score (3.6-3.8) than females (3.3-3.4) across all races, suggesting that the male employees overall are less healthy than the female employees.
Employees who have a hospital visit in a given quarter have higher health scores in that quarter (4.1-4.3) than those who do not have a hospital visit (3.2-3.3) across all races.
Employees with a Race of 1.0 have a higher health score (unhealthier) than employees with a race of 2.0 or 3.0.


## Evaluating the Claim
InsurAHealth's claim implies that the employees at Company A (who are represented by each sample in each quarter) are becoming more ill over time.

However, boxplot below shows mean health scores of 3.4 in Q1, 3.5 in Q6, and 3.9 in Q12. There is a steady increase in health scores over time.

```markdown
# Mean Health Scores Over Time
fig, ax8 = plt.subplots(1,1,figsize=(10,5))
sns.boxplot(x='Quarter', y='Health Score', data=df, ax=ax8, color='indianred').set_title('Health Scores Over Time')
means = df.groupby(['Quarter'])['Health Score'].mean().values # calculate means
mean_labels = [str(np.round(s, 2)) for s in means] # add mean labels
pos = range(len(means))
for tick, label in zip(pos, ax8.get_xticklabels()):
    ax8.text(pos[tick], means[tick] + 0.05, mean_labels[tick], horizontalalignment='center', size='large', color='k')
```
![Image](https://github.com/fionaroni/insurance/blob/master/health-scores_time_medians.png)

The heatmaps and facet grid show that Age, Sex, and Salary are attributes that are positively correlated with Health Score. 
In other words, older people tend to have higher health scores than younger people. Males tend to have higher health scores than females. And people with higher salaries tend to have higher health scores than people with lower salaries.

The barplot, “Sex of Employees Over Time,” and the boxplots, “Age of Employees Over Time” and “Salary of Employees Over Time”, from Question 1 show how over the 12 quarters, the employees at Company A have a greater proportion of male employees, older employees, and employees who have higher salaries. Considering that these attributes increase in frequency over the 12 quarters and that they also tend to be associated with a higher health score (less healthy), it is possible that the increase of Mean Health Scores results from changes in employee characteristics/demographics at Company A, rather than from the employees becoming more ill.

For example, an older employee is more likely to develop diseases or illnesses than a younger employee. The boxplot “Age of Employees Over Time” shows that the mean age of employees increases from Q1 to Q12. Perhaps in the latter quarters, Company A hired new employees who were older and thus already experiencing some health problems. Or perhaps in the latter quarters, some young employees left the company, causing the existing “old” employees’ health scores to have a greater impact on the mean. Or perhaps both of these happened simultaneously. These occurrences could increase the mean age of the employees, while the health statuses of the employees remain completely unchanged.

High salaries could be linked with higher-position jobs that are more demanding and more stressful. The “Salary of Employees Over Time” boxplot shows that the mean salary of employees increases from Q1 to Q12. Perhaps in the latter quarters, Company A hired new employees who came from high-position jobs at their previous companies and already exhibited signs of poor health. The company experienced a large increase in new hires in executive positions with high salaries and nearly no increase in new hires in low-stress roles with few repercussions on health. The employees at Company A are not “getting sicker.” Rather, Company A now has a greater proportion of employees who are sickly due to their jobs.

Males tend to have more health issues than females, perhaps because males are not as diligent about exercising certain self-care habits such as diet and exercise. The facet grid shows that males have higher health scores than females. The barplot, “Sex of Employees Over Time,” shows the sample population becoming slightly more male as time progresses. Rather than the employees at Company A “getting sicker,” the proportion of males has increased, which has contributed to the increased Mean Health Score. We have also observed from the heatmap that being male and having a high salary are positively correlated, which further explains how average health scores might increase over time, and that employees are not necessarily getting sicker.

In conclusion, while it is true that the mean Health Score increases from Quarter 1 to Quarter 12, we should not take InsurAHealth’s claim at face value because the Health Scores might not account for several confounding variables. Unless we control for these variables over time, we cannot be certain that the mean Health Score is a reliable indicator of the worsening or improvement of the health conditions of employees at Company A.
