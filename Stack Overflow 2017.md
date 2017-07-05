
# Stack Overflow Dataset 2017: A look into the career of Data Science
Every year, Stack Overflow conducts a massive survey of people on the site, covering all sorts of information like programming languages, salary, code style and various other information. This year, they amassed more than 64,000 responses fielded from 213 countries. Find the dataset [here](https://www.kaggle.com/stackoverflow/so-survey-2017)

## Introduction
This is an open-ended Exploratory Data Analysis project for the Data Science program at [K2 Data Science](http://k2datascience.com). 

A minimum valuable product (MVP) of this research was conducted using the [MTA turnstile dataset](http://web.mta.info/developers/turnstile.html) and its findings can be found on the [MVP page](MVP.md).

## Goals
The overall goal is to gain insight into the careers of Data Scientist. Of particular focus are the factors that influence a data scientist's salary. I adress this goal through 5 specific questions: 
1. How do data science salaries compare to other career salaries? 
2. How are data science salaries effected by years of programming experience? 
3. What are the most popular languages and tools among data scientists?
4. As a data scientist does the language you know impact your salary?
5. Across all stack overflow users what are the most popular IDEs and how does it break down by experience level?


## Assumptions
* Stack overflow's data is representative of the community of data scientists
* All salaries listed are in USD
* Years spent programming is a good indicator of professional experience as opposed to years spent coding as a job

## What's in the data 
survey_results_schema.csv lists each Question in long form. There are a total of 155 different Questions. The questions used for analysis are:
* DeveloperType -> Which of the following best describe you?
* Salary -> What is your current annual base salary, before taxes, and excluding bonuses, grants, or other compensation?    
* YearsProgram -> How long has it been since you first learned how to program?      
* HaveWorkedLanguage -> Which of the following languages have you done extensive development work in over the past year, and which do you want to work in over the next year? 

There are numerous useful Questions in the dataset and a plethora of insight questions beyond what is listed in the goals section. Further analysis insight questions can be found in the concluding section.

## Cleaning and Exploring the Data
The data is relatively clean in its native form, however, there were two important preprocessing tasks:
1. Convert YearsProgram from its current form of "5 to 6 years" to a true numeric value
2. Convert DeveloperType and HaveWorkedLanguage whose responses are multianswer separated by ; into dummy variables

Task 2) could not be completed using pandas builtin [get_dummies](https://pandas.pydata.org/pandas-docs/stable/generated/pandas.get_dummies.html) since the columns contained lists. Therefore, a custom function createDummies was created.


```python
%matplotlib inline
import matplotlib.pyplot as plt
import pandas as pd
import seaborn as sns
plt.style.use('ggplot')
pd.options.display.float_format = "{:.2f}".format # two decimal places

### helper functions ##
flatten = lambda l: [item for sublist in l for item in sublist]  ## takes a list(list) -> list


def createDummies(column, _data):
    _data[column].fillna("", inplace=True)
    uniqueVals = set(flatten(list(map(lambda str: str.split("; "),
                                      list(_data[column].unique())))))
    uniqueVals.remove("")

    _dummies = pd.DataFrame(0, index=_data.index, columns=uniqueVals)

    for dummyCol in _dummies.columns:
        dummies = so_data[column].map(lambda row: dummyCol in row)
        _dummies[dummyCol] = dummies

    return _dummies, uniqueVals


############# Preprocess Data #############
### Read data ###
so_data = pd.read_csv("data/survey_results_public.csv")

### common column names and variables ###
devType = "DeveloperType"
salary = "Salary"
dScien = "Data scientist"
experience = "YearsProgram"
language = "HaveWorkedLanguage"
jobT = "Job Title"
years = "YearsProgram"
yearsN = "YearsProgramNumeric"
ide = "IDE"
stats_long_name = 'Developer with a statistics or mathematics background'
stats_short_name = "Stats/Math Background"
emb_long_name = 'Embedded applications/devices developer'
emb_short_name = "Embedded Systems"

job_dummies, jobTitles = createDummies(devType, so_data)
language_dummies, languageTitles = createDummies(language, so_data)

year_conversions = {"Less than a year": 0,
                    "1 to 2 years": 2,
                    "2 to 3 years": 3,
                    "3 to 4 years": 4,
                    "4 to 5 years": 5,
                    "5 to 6 years": 6,
                    "6 to 7 years": 7,
                    "7 to 8 years": 7,
                    "8 to 9 years": 9,
                    "9 to 10 years": 10,
                    "10 to 11 years": 11,
                    "11 to 12 years": 12,
                    "12 to 13 years": 13,
                    "13 to 14 years": 14,
                    "14 to 15 years": 15,
                    "15 to 16 years": 16,
                    "16 to 17 years": 17,
                    "17 to 18 years": 18,
                    "18 to 19 years": 19,
                    "20 or more years": 20
                    }
years_numeric = so_data[years].map(year_conversions)
years_numeric.name = yearsN
so_data = so_data.join(years_numeric)
```

### How do data science salaries compare to other career salaries?
We begin with the question: How do data science salaries compare to other career salaries? To begin answering this question, we first calculate the average salary for different job types. Next, we look at the distribution for each job type in the form of a histogram.



```python
sal_v_type = pd.melt(so_data.join(job_dummies).loc[:, [salary] + list(jobTitles)], id_vars=salary).dropna()
sal_v_type = sal_v_type[sal_v_type["value"]]
del sal_v_type["value"]
sal_v_type.columns = [salary, jobT]

sal_v_type.loc[sal_v_type[jobT] == stats_long_name, jobT] = stats_short_name
sal_v_type.loc[sal_v_type[jobT] == emb_long_name, jobT] = emb_short_name

output = sal_v_type.groupby(jobT)[salary].mean().sort_values(ascending=False).to_frame()
output.columns = ["Average Salary"]
output
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Salary</th>
    </tr>
    <tr>
      <th>Job Title</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Other</th>
      <td>72344.57</td>
    </tr>
    <tr>
      <th>DevOps specialist</th>
      <td>66158.20</td>
    </tr>
    <tr>
      <th>Machine learning specialist</th>
      <td>66023.10</td>
    </tr>
    <tr>
      <th>Stats/Math Background</th>
      <td>62455.50</td>
    </tr>
    <tr>
      <th>Data scientist</th>
      <td>61137.33</td>
    </tr>
    <tr>
      <th>Embedded Systems</th>
      <td>58524.35</td>
    </tr>
    <tr>
      <th>Quality assurance engineer</th>
      <td>56423.11</td>
    </tr>
    <tr>
      <th>Desktop applications developer</th>
      <td>56352.86</td>
    </tr>
    <tr>
      <th>Systems administrator</th>
      <td>56331.94</td>
    </tr>
    <tr>
      <th>Web developer</th>
      <td>54968.02</td>
    </tr>
    <tr>
      <th>Database administrator</th>
      <td>53744.81</td>
    </tr>
    <tr>
      <th>Graphics programming</th>
      <td>53212.89</td>
    </tr>
    <tr>
      <th>Mobile developer</th>
      <td>50773.34</td>
    </tr>
    <tr>
      <th>Graphic designer</th>
      <td>45964.05</td>
    </tr>
  </tbody>
</table>
</div>




```python
g = sns.FacetGrid(sal_v_type, col=jobT, col_wrap=4, sharey=False)
g = g.map(plt.hist, salary)
```


![png](output_9_0.png)


#### Findings
In general, it seems that Data Scientists, Machine Learning eingeers, and stats/math programmers have some of the highest average salaries of the surveyed users. Seems like you made the right decision getting into data science with K2! Also, the distribution of salary for each job type is visually similar. The only exception seems to be DevOps specialists whose distribution is not left skewed.  

### How are data science salaries effected by years of programming experience?
It is not enough to say that Data Scientists make more money than other technical professionals without incorporating experience. What if all the Data Scientists surveyed were 10+ year veterans of the industry? In this analysis, years programming is used as a surrogate to indicate expereience level. To question how experience effects salary we answer two questions:
1. What is the average salary of data scientists at each experience level and how does this compare to the average across all other job types? 
2. What is the relationship between experience and salary for each job type individually?


```python
diff_in_salary = so_data.join(job_dummies).groupby([years, dScien])[salary].mean()
gain = (diff_in_salary.shift(-1) - diff_in_salary)[::2].reset_index(1, drop=True).sort_values(ascending=False).to_frame()
gain.columns = ["Average gain in salary: average(DS) - average(All other groups)"]
gain
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average gain in salary: average(DS) - average(All other groups)</th>
    </tr>
    <tr>
      <th>YearsProgram</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Less than a year</th>
      <td>31647.34</td>
    </tr>
    <tr>
      <th>1 to 2 years</th>
      <td>15494.99</td>
    </tr>
    <tr>
      <th>3 to 4 years</th>
      <td>10563.22</td>
    </tr>
    <tr>
      <th>14 to 15 years</th>
      <td>10504.60</td>
    </tr>
    <tr>
      <th>18 to 19 years</th>
      <td>10149.81</td>
    </tr>
    <tr>
      <th>6 to 7 years</th>
      <td>7529.56</td>
    </tr>
    <tr>
      <th>7 to 8 years</th>
      <td>5817.80</td>
    </tr>
    <tr>
      <th>19 to 20 years</th>
      <td>5714.87</td>
    </tr>
    <tr>
      <th>16 to 17 years</th>
      <td>4881.80</td>
    </tr>
    <tr>
      <th>2 to 3 years</th>
      <td>4756.40</td>
    </tr>
    <tr>
      <th>20 or more years</th>
      <td>3883.56</td>
    </tr>
    <tr>
      <th>17 to 18 years</th>
      <td>3635.32</td>
    </tr>
    <tr>
      <th>5 to 6 years</th>
      <td>3065.51</td>
    </tr>
    <tr>
      <th>11 to 12 years</th>
      <td>1868.25</td>
    </tr>
    <tr>
      <th>10 to 11 years</th>
      <td>450.32</td>
    </tr>
    <tr>
      <th>4 to 5 years</th>
      <td>263.20</td>
    </tr>
    <tr>
      <th>12 to 13 years</th>
      <td>-173.51</td>
    </tr>
    <tr>
      <th>8 to 9 years</th>
      <td>-1069.39</td>
    </tr>
    <tr>
      <th>15 to 16 years</th>
      <td>-1527.67</td>
    </tr>
    <tr>
      <th>9 to 10 years</th>
      <td>-6147.88</td>
    </tr>
    <tr>
      <th>13 to 14 years</th>
      <td>-12261.37</td>
    </tr>
  </tbody>
</table>
</div>




```python
joiners = [salary, yearsN]
sal_exp_v_type = pd.melt(so_data.join(job_dummies).loc[:, joiners + list(jobTitles)],
                         id_vars=joiners).dropna()
sal_exp_v_type = sal_exp_v_type[sal_exp_v_type["value"]]
del sal_exp_v_type["value"]
sal_exp_v_type.columns = [salary, yearsN, jobT]

sal_exp_v_type.loc[sal_exp_v_type[jobT] == stats_long_name, jobT] = stats_short_name
sal_exp_v_type.loc[sal_exp_v_type[jobT] == emb_long_name, jobT] = emb_short_name

sal_exp_v_type[yearsN] = sal_exp_v_type[yearsN].astype(int)

# In general it seems being a Data Scientist, machine learning eingeering, or stats/math programmer increases your pay
sns.factorplot(x="YearsProgramNumeric", y="Salary", col="Job Title", data=sal_exp_v_type, col_wrap=4).savefig(
    "stuff.png")
```


![png](output_13_0.png)


#### Findings
The table and chart above show some interesting and exciting conclusions for Data Scientists.

First, when looking at the difference in the average salary between Data Scientists and all other groups, it seems that getting a data science title early in your programming career can boost your salary. The boost in salary is most intense for very new programmers (<= 4 years), and very experienced programmers. The only negative gain in average salary appears in programmers well into their careers (9+ years). 

The second chart is a point plot in [seaborn](http://seaborn.pydata.org/). It shows point estimates and confidence intervals. For web, mobile, and desktop applicationd developers salary linearly increases with the number of years of programming experience. Furthermore, there is a very tight confidence interval for the estimate of the mean salary.

For data scientists, the relationship between programming experience and salary is much less linear and the confidence interval at each experience level is much wider. This makes sense because programming experience is not the only prerequisite for the job of a Data Scientist. Data Science incorporates statistics, machine learning, visualization, and data analytics. Looking solely at programming experience does not give us a good estimate of the salary of a Data Scientist. This is great for new programmers who are interested in breaking into the field. The data shows that even with little programming experience you can gain the title of Data Scientist and there are still opportunities for high paying salaries.  

### What are the most popular languages among data scientists?
First we will look at the most popular languages among data scientists and then the most popular languages among all over those surveyed users.


```python
datascience_data = so_data[so_data.loc[:, devType].str.contains(dScien, na=False)]

dscien_langauge_dummies, dscien_lang = createDummies(language, datascience_data)
ds_lang = (dscien_langauge_dummies.sum(0).sort_values(ascending=False) / len(dscien_langauge_dummies) * 100.).to_frame()
ds_lang.columns = ["% Data Scientists reported working with the language in the last year"]
ds_lang


```

    /anaconda/lib/python3.6/site-packages/pandas/core/generic.py:3660: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      self._update_inplace(new_data)





<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>% Data Scientists reported working with the language in the last year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Java</th>
      <td>55.47</td>
    </tr>
    <tr>
      <th>SQL</th>
      <td>46.17</td>
    </tr>
    <tr>
      <th>JavaScript</th>
      <td>45.29</td>
    </tr>
    <tr>
      <th>C</th>
      <td>43.91</td>
    </tr>
    <tr>
      <th>Python</th>
      <td>38.49</td>
    </tr>
    <tr>
      <th>C#</th>
      <td>25.12</td>
    </tr>
    <tr>
      <th>C++</th>
      <td>22.30</td>
    </tr>
    <tr>
      <th>PHP</th>
      <td>20.72</td>
    </tr>
    <tr>
      <th>R</th>
      <td>19.11</td>
    </tr>
    <tr>
      <th>Scala</th>
      <td>7.65</td>
    </tr>
    <tr>
      <th>TypeScript</th>
      <td>7.29</td>
    </tr>
    <tr>
      <th>Ruby</th>
      <td>7.29</td>
    </tr>
    <tr>
      <th>Matlab</th>
      <td>6.90</td>
    </tr>
    <tr>
      <th>VBA</th>
      <td>5.58</td>
    </tr>
    <tr>
      <th>VB.NET</th>
      <td>5.48</td>
    </tr>
    <tr>
      <th>Perl</th>
      <td>5.48</td>
    </tr>
    <tr>
      <th>Go</th>
      <td>5.32</td>
    </tr>
    <tr>
      <th>Swift</th>
      <td>4.99</td>
    </tr>
    <tr>
      <th>Objective-C</th>
      <td>4.96</td>
    </tr>
    <tr>
      <th>Assembly</th>
      <td>4.93</td>
    </tr>
    <tr>
      <th>Groovy</th>
      <td>3.48</td>
    </tr>
    <tr>
      <th>CoffeeScript</th>
      <td>3.28</td>
    </tr>
    <tr>
      <th>Visual Basic 6</th>
      <td>3.19</td>
    </tr>
    <tr>
      <th>Lua</th>
      <td>2.96</td>
    </tr>
    <tr>
      <th>Haskell</th>
      <td>2.43</td>
    </tr>
    <tr>
      <th>Clojure</th>
      <td>1.81</td>
    </tr>
    <tr>
      <th>F#</th>
      <td>1.74</td>
    </tr>
    <tr>
      <th>Elixir</th>
      <td>1.28</td>
    </tr>
    <tr>
      <th>Rust</th>
      <td>1.25</td>
    </tr>
    <tr>
      <th>Smalltalk</th>
      <td>1.18</td>
    </tr>
    <tr>
      <th>Common Lisp</th>
      <td>1.08</td>
    </tr>
    <tr>
      <th>Julia</th>
      <td>1.05</td>
    </tr>
    <tr>
      <th>Erlang</th>
      <td>0.99</td>
    </tr>
    <tr>
      <th>Dart</th>
      <td>0.62</td>
    </tr>
    <tr>
      <th>Hack</th>
      <td>0.49</td>
    </tr>
  </tbody>
</table>
</div>




```python
_langauge_dummies, _lang = createDummies(language, so_data)
_lang = (_langauge_dummies.sum(0) / len(_langauge_dummies) * 100).sort_values(ascending=False).to_frame()
_lang.columns = ["% ALL USERS reported working with the language in the last year"]
_lang
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>% ALL USERS reported working with the language in the last year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Java</th>
      <td>54.77</td>
    </tr>
    <tr>
      <th>JavaScript</th>
      <td>44.51</td>
    </tr>
    <tr>
      <th>C</th>
      <td>41.20</td>
    </tr>
    <tr>
      <th>SQL</th>
      <td>36.49</td>
    </tr>
    <tr>
      <th>C#</th>
      <td>24.28</td>
    </tr>
    <tr>
      <th>Python</th>
      <td>22.77</td>
    </tr>
    <tr>
      <th>PHP</th>
      <td>20.02</td>
    </tr>
    <tr>
      <th>C++</th>
      <td>15.87</td>
    </tr>
    <tr>
      <th>R</th>
      <td>9.83</td>
    </tr>
    <tr>
      <th>TypeScript</th>
      <td>6.79</td>
    </tr>
    <tr>
      <th>Ruby</th>
      <td>6.47</td>
    </tr>
    <tr>
      <th>Swift</th>
      <td>4.61</td>
    </tr>
    <tr>
      <th>Objective-C</th>
      <td>4.57</td>
    </tr>
    <tr>
      <th>VB.NET</th>
      <td>4.42</td>
    </tr>
    <tr>
      <th>Assembly</th>
      <td>3.55</td>
    </tr>
    <tr>
      <th>Perl</th>
      <td>3.08</td>
    </tr>
    <tr>
      <th>VBA</th>
      <td>3.06</td>
    </tr>
    <tr>
      <th>Matlab</th>
      <td>3.05</td>
    </tr>
    <tr>
      <th>Go</th>
      <td>3.03</td>
    </tr>
    <tr>
      <th>Scala</th>
      <td>2.55</td>
    </tr>
    <tr>
      <th>Groovy</th>
      <td>2.32</td>
    </tr>
    <tr>
      <th>CoffeeScript</th>
      <td>2.32</td>
    </tr>
    <tr>
      <th>Visual Basic 6</th>
      <td>2.08</td>
    </tr>
    <tr>
      <th>Lua</th>
      <td>2.02</td>
    </tr>
    <tr>
      <th>Haskell</th>
      <td>1.26</td>
    </tr>
    <tr>
      <th>F#</th>
      <td>0.89</td>
    </tr>
    <tr>
      <th>Rust</th>
      <td>0.81</td>
    </tr>
    <tr>
      <th>Clojure</th>
      <td>0.76</td>
    </tr>
    <tr>
      <th>Elixir</th>
      <td>0.74</td>
    </tr>
    <tr>
      <th>Smalltalk</th>
      <td>0.64</td>
    </tr>
    <tr>
      <th>Erlang</th>
      <td>0.55</td>
    </tr>
    <tr>
      <th>Common Lisp</th>
      <td>0.53</td>
    </tr>
    <tr>
      <th>Dart</th>
      <td>0.28</td>
    </tr>
    <tr>
      <th>Julia</th>
      <td>0.27</td>
    </tr>
    <tr>
      <th>Hack</th>
      <td>0.21</td>
    </tr>
  </tbody>
</table>
</div>



#### Findings
Data Scientists and all users use similar languages. Java and javascript appear in the top languages in both groups. However, we see an approximately 16% increase in the use of python, 10% increase in the use of R, and 10% increase in the use of SQL. This makes sense. R and python are considered the most data science specific languages, and data scientists often need to interact with databases and use SQL.

### As a data scientist does the language you work in impact your salary?
As we saw in the last section R and Python are more popular among Data Scientists compared to all stack overflow users. Let's look at the salary breakdown for R and Python. For this analysis, 3 groups were formed: 
1) use R and not Python 
2) use Python and not R
3) use Python and R


```python
rName = "r_only"
pName = "python_only"
bothName = "python_and_r"
r_only = _langauge_dummies["R"] & (~_langauge_dummies["Python"])
r_only.name = rName
python_only = (_langauge_dummies["Python"]) & (~_langauge_dummies["R"])
python_only.name = pName
python_and_r = (_langauge_dummies["Python"]) & (_langauge_dummies["R"])
python_and_r.name = bothName

s_ds_lang = so_data.join([r_only, python_only, python_and_r]).loc[:, [salary, yearsN, rName, pName, bothName]].melt(
    id_vars=[salary, yearsN])
s_ds_lang = s_ds_lang[s_ds_lang["value"]]
del s_ds_lang["value"]
s_ds_lang.columns = [salary, "Years Programming", "Type"]
output = s_ds_lang.groupby("Type")[salary].mean().sort_values(ascending=False).to_frame()
output.columns = ["Mean Salary"]
output
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Mean Salary</th>
    </tr>
    <tr>
      <th>Type</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>python_and_r</th>
      <td>66394.96</td>
    </tr>
    <tr>
      <th>r_only</th>
      <td>64807.30</td>
    </tr>
    <tr>
      <th>python_only</th>
      <td>60361.45</td>
    </tr>
  </tbody>
</table>
</div>




```python
sns.boxplot(x="Type", y=salary, data=s_ds_lang)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11f75ef60>




![png](output_21_1.png)



```python
s_ds_lang.groupby("Type")["Years Programming"].mean().sort_values(ascending=False).to_frame()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Years Programming</th>
    </tr>
    <tr>
      <th>Type</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>r_only</th>
      <td>10.91</td>
    </tr>
    <tr>
      <th>python_and_r</th>
      <td>10.64</td>
    </tr>
    <tr>
      <th>python_only</th>
      <td>10.56</td>
    </tr>
  </tbody>
</table>
</div>




```python
sns.boxplot(x="Type", y="Years Programming", data=s_ds_lang)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11f514048>




![png](output_23_1.png)


#### Findings
For both median and mean salaries, Python and R > R only > Python only. Maybe it pays to learn more than one data science language. Additionally, the mean and median years programming is about the same for each group, so it cannot be explained away with experience. 

### Across all stack overflow users what are the most popular IDEs and how does it break down by experience level?
First we start by looking at the most popular IDEs among data scientists and all users. Next we look at IDE choice when accounting for years spent programming.



```python
dscien_ide_dummies, ides = createDummies(ide, datascience_data)
output = (dscien_ide_dummies.sum(0) / len(dscien_ide_dummies) * 100).sort_values(ascending=False).to_frame()
output.columns = ["% Data Scientists regularly use IDE"]
output
```

    /anaconda/lib/python3.6/site-packages/pandas/core/generic.py:3660: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      self._update_inplace(new_data)





<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>% Data Scientists regularly use IDE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Visual Studio</th>
      <td>29.69</td>
    </tr>
    <tr>
      <th>Notepad++</th>
      <td>26.08</td>
    </tr>
    <tr>
      <th>Vim</th>
      <td>25.85</td>
    </tr>
    <tr>
      <th>Sublime Text</th>
      <td>20.43</td>
    </tr>
    <tr>
      <th>Eclipse</th>
      <td>18.69</td>
    </tr>
    <tr>
      <th>IntelliJ</th>
      <td>18.33</td>
    </tr>
    <tr>
      <th>IPython / Jupyter</th>
      <td>13.40</td>
    </tr>
    <tr>
      <th>Atom</th>
      <td>13.30</td>
    </tr>
    <tr>
      <th>PyCharm</th>
      <td>12.61</td>
    </tr>
    <tr>
      <th>Android Studio</th>
      <td>11.89</td>
    </tr>
    <tr>
      <th>Visual Studio Code</th>
      <td>11.82</td>
    </tr>
    <tr>
      <th>RStudio</th>
      <td>8.51</td>
    </tr>
    <tr>
      <th>Xcode</th>
      <td>7.09</td>
    </tr>
    <tr>
      <th>Emacs</th>
      <td>6.96</td>
    </tr>
    <tr>
      <th>NetBeans</th>
      <td>6.93</td>
    </tr>
    <tr>
      <th>PHPStorm</th>
      <td>5.45</td>
    </tr>
    <tr>
      <th>RubyMine</th>
      <td>1.41</td>
    </tr>
    <tr>
      <th>TextMate</th>
      <td>1.28</td>
    </tr>
    <tr>
      <th>Komodo</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>Coda</th>
      <td>0.66</td>
    </tr>
    <tr>
      <th>Zend</th>
      <td>0.53</td>
    </tr>
    <tr>
      <th>Light Table</th>
      <td>0.33</td>
    </tr>
  </tbody>
</table>
</div>




```python
_ide_dummies, ides = createDummies(ide, so_data)
output = (_ide_dummies.sum(0) / len(_ide_dummies) * 100).sort_values(ascending=False).to_frame()
output.columns = ["% ALL USERS regularly use IDE"]
output
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>% ALL USERS regularly use IDE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Visual Studio</th>
      <td>31.25</td>
    </tr>
    <tr>
      <th>Notepad++</th>
      <td>24.68</td>
    </tr>
    <tr>
      <th>Sublime Text</th>
      <td>19.42</td>
    </tr>
    <tr>
      <th>Vim</th>
      <td>18.83</td>
    </tr>
    <tr>
      <th>Eclipse</th>
      <td>15.37</td>
    </tr>
    <tr>
      <th>IntelliJ</th>
      <td>14.43</td>
    </tr>
    <tr>
      <th>Visual Studio Code</th>
      <td>13.28</td>
    </tr>
    <tr>
      <th>Atom</th>
      <td>12.73</td>
    </tr>
    <tr>
      <th>Android Studio</th>
      <td>11.58</td>
    </tr>
    <tr>
      <th>Xcode</th>
      <td>7.49</td>
    </tr>
    <tr>
      <th>PyCharm</th>
      <td>6.36</td>
    </tr>
    <tr>
      <th>PHPStorm</th>
      <td>5.77</td>
    </tr>
    <tr>
      <th>NetBeans</th>
      <td>5.66</td>
    </tr>
    <tr>
      <th>Emacs</th>
      <td>3.81</td>
    </tr>
    <tr>
      <th>IPython / Jupyter</th>
      <td>3.72</td>
    </tr>
    <tr>
      <th>RStudio</th>
      <td>1.94</td>
    </tr>
    <tr>
      <th>RubyMine</th>
      <td>1.19</td>
    </tr>
    <tr>
      <th>TextMate</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>Komodo</th>
      <td>0.49</td>
    </tr>
    <tr>
      <th>Coda</th>
      <td>0.44</td>
    </tr>
    <tr>
      <th>Zend</th>
      <td>0.30</td>
    </tr>
    <tr>
      <th>Light Table</th>
      <td>0.14</td>
    </tr>
  </tbody>
</table>
</div>




```python
joinerz = list(ides) + [yearsN]
ide_v_yearsN = so_data.join(_ide_dummies).loc[:, joinerz].melt(id_vars=yearsN)
ide_v_yearsN = ide_v_yearsN[ide_v_yearsN["value"]]
del ide_v_yearsN["value"]
ide_v_yearsN.columns = [yearsN, ide]

popular_ides = list(_ide_dummies.sum().sort_values(ascending=False)[0:11].index)

ide_across_years = pd.crosstab(index=ide_v_yearsN[yearsN], columns=ide_v_yearsN[ide])
ide_across_years = ide_across_years.div(ide_across_years.sum(1), axis=0)
popular_ides_v_years = ide_across_years[popular_ides]

handlez = plt.plot(popular_ides_v_years)
plt.title("% IDE use by years programming")
plt.legend(handles=handlez, labels=popular_ides, bbox_to_anchor=(1.05, 1), loc=2, borderaxespad=0.)
```




    <matplotlib.legend.Legend at 0x11c1ec6a0>




![png](output_28_1.png)


#### Findings
Not suprisely we see an approximately 6% increase in the use of PyCharm, 10% increase in the use of juypter notebooks, and a 7% increase in the use of RStudio. Finally, looking at IDE use across experience levels: ["VIM is for old people"] (https://meta.stackoverflow.com/questions/350926/2017-survey-vim-is-for-old-people-joking/350935#350935)


## Conculsion
This notebook was largely concerned with exploring what effects a person's data science salary. It seems like it is an exciting time to be a data scientist and you are often paid more than your peers. Furthermore, data science salaries are less influenced by programming experience than other job types. This is a good sign for inexperienced coders and those trying to break into the field. Future analysis should focus on incorporating 
