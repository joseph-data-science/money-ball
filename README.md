# Money Ball
## An analysis of Okaland's spending efficiency during the money ball period

This project aims to analyze the spending efficieny of oakland during the money ball period. During this period oakland was able to perform better than other team by redefining the important metrics of what makes a good player.

More information can be found at:
* [wikipedia](https://en.wikipedia.org/wiki/Moneyball)
* [Moneyball Movie](https://www.imdb.com/title/tt1210166/)
* [Moneyball book](https://www.amazon.com/Moneyball-The-Winning-Unfair-Game/dp/0393324818)
* [Web article: FivethiryEight](https://fivethirtyeight.com/features/dont-be-fooled-by-baseballs-small-budget-success-stories/)


```python
#import all the necessary files for this project
import sqlite3
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

```


```python
# connect to the database 
sqlite_file = 'lahman2014.sqlite'
conn = sqlite3.connect(sqlite_file)
```

## Part1: Wrangling

Query the database to obtain team data and store it in a variable



```python
teams_query = "SELECT yearID, lgID, teamID, franchID, G, W, name FROM Teams"
teams_info = pd.read_sql(teams_query, conn)
teams_info.head()

```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>yearID</th>
      <th>lgID</th>
      <th>teamID</th>
      <th>franchID</th>
      <th>G</th>
      <th>W</th>
      <th>name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1871</td>
      <td>NA</td>
      <td>BS1</td>
      <td>BNA</td>
      <td>31</td>
      <td>20</td>
      <td>Boston Red Stockings</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1871</td>
      <td>NA</td>
      <td>CH1</td>
      <td>CNA</td>
      <td>28</td>
      <td>19</td>
      <td>Chicago White Stockings</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1871</td>
      <td>NA</td>
      <td>CL1</td>
      <td>CFC</td>
      <td>29</td>
      <td>10</td>
      <td>Cleveland Forest Citys</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1871</td>
      <td>NA</td>
      <td>FW1</td>
      <td>KEK</td>
      <td>19</td>
      <td>7</td>
      <td>Fort Wayne Kekiongas</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1871</td>
      <td>NA</td>
      <td>NY2</td>
      <td>NNA</td>
      <td>33</td>
      <td>16</td>
      <td>New York Mutuals</td>
    </tr>
  </tbody>
</table>
</div>



Query the table to obtain the total salaries for each team in each year



```python
salary_query2 = "SELECT yearID, lgID, teamID, sum(salary) FROM Salaries GROUP BY yearID, teamID "   
salary_tbl = pd.read_sql(salary_query2, conn)
salary_tbl.sort_values(by='teamID', axis = 0)
salary_tbl.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>yearID</th>
      <th>lgID</th>
      <th>teamID</th>
      <th>sum(salary)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1985</td>
      <td>NL</td>
      <td>ATL</td>
      <td>14807000.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1985</td>
      <td>AL</td>
      <td>BAL</td>
      <td>11560712.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1985</td>
      <td>AL</td>
      <td>BOS</td>
      <td>10897560.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1985</td>
      <td>AL</td>
      <td>CAL</td>
      <td>14427894.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1985</td>
      <td>AL</td>
      <td>CHA</td>
      <td>9846178.0</td>
    </tr>
  </tbody>
</table>
</div>



Use **yearID, lgID** and **teamID** to **inner join** the salaries table and the teams table to: 
1. obtain the salaries, wins and games played for each team in each year.
2. Ensure that only team that appear in both tables are extracted from the join to prevent the creation of missing values which may happen when we use left/right/outer join. Therefore inner join is the preferred join.
        Note: Teams table has team information starting from the 1871 while Salaries table has team salaries information starting from 1985. Thus an right/outer join will produce too many null fields in the salary column.  Though we have lost team information from the team table from 1871 to 1985, the lost team information wouldn't have been useful to us anyway because there is no corresponding data from 1871 to 1985 in the salaries table.


```python
#inner join the two tables queried from the database
league = teams_info.merge(salary_tbl, how = 'inner', on = ['teamID', 'yearID', 'lgID'])
league.sort_values(by = ['name', 'yearID'])

# drop any rows that contain missing values
league.dropna()
league.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>yearID</th>
      <th>lgID</th>
      <th>teamID</th>
      <th>franchID</th>
      <th>G</th>
      <th>W</th>
      <th>name</th>
      <th>sum(salary)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1985</td>
      <td>AL</td>
      <td>BAL</td>
      <td>BAL</td>
      <td>161</td>
      <td>83</td>
      <td>Baltimore Orioles</td>
      <td>11560712.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1985</td>
      <td>AL</td>
      <td>BOS</td>
      <td>BOS</td>
      <td>163</td>
      <td>81</td>
      <td>Boston Red Sox</td>
      <td>10897560.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1985</td>
      <td>AL</td>
      <td>CAL</td>
      <td>ANA</td>
      <td>162</td>
      <td>90</td>
      <td>California Angels</td>
      <td>14427894.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1985</td>
      <td>AL</td>
      <td>CHA</td>
      <td>CHW</td>
      <td>163</td>
      <td>85</td>
      <td>Chicago White Sox</td>
      <td>9846178.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1985</td>
      <td>AL</td>
      <td>CLE</td>
      <td>CLE</td>
      <td>162</td>
      <td>60</td>
      <td>Cleveland Indians</td>
      <td>6551666.0</td>
    </tr>
  </tbody>
</table>
</div>



**Missing data** has been dealt with by **dropping rows with missing data.**
This **hasn't affected our dataframe** in any way because the number of rows before and after the drop has remained the same


```python
def win_rate(row):
    return (row['W']/row['G']) * 100
    
league['win_rate'] = league.apply(lambda row: win_rate(row), axis = 1)
league = league[league['yearID'] >= 1990 ]
league.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>yearID</th>
      <th>lgID</th>
      <th>teamID</th>
      <th>franchID</th>
      <th>G</th>
      <th>W</th>
      <th>name</th>
      <th>sum(salary)</th>
      <th>win_rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>130</th>
      <td>1990</td>
      <td>AL</td>
      <td>BAL</td>
      <td>BAL</td>
      <td>161</td>
      <td>76</td>
      <td>Baltimore Orioles</td>
      <td>9680084.0</td>
      <td>47.204969</td>
    </tr>
    <tr>
      <th>131</th>
      <td>1990</td>
      <td>AL</td>
      <td>BOS</td>
      <td>BOS</td>
      <td>162</td>
      <td>88</td>
      <td>Boston Red Sox</td>
      <td>20558333.0</td>
      <td>54.320988</td>
    </tr>
    <tr>
      <th>132</th>
      <td>1990</td>
      <td>AL</td>
      <td>CAL</td>
      <td>ANA</td>
      <td>162</td>
      <td>80</td>
      <td>California Angels</td>
      <td>21720000.0</td>
      <td>49.382716</td>
    </tr>
    <tr>
      <th>133</th>
      <td>1990</td>
      <td>AL</td>
      <td>CHA</td>
      <td>CHW</td>
      <td>162</td>
      <td>94</td>
      <td>Chicago White Sox</td>
      <td>9491500.0</td>
      <td>58.024691</td>
    </tr>
    <tr>
      <th>134</th>
      <td>1990</td>
      <td>AL</td>
      <td>CLE</td>
      <td>CLE</td>
      <td>162</td>
      <td>77</td>
      <td>Cleveland Indians</td>
      <td>14487000.0</td>
      <td>47.530864</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Change yearID column and W column to type int
league.loc[:,('yearID')].astype(int)
league.loc[: , ('W')].astype(int)

# Create a variable teamID that is an array of all IDs. 
# variable will be useful later on
teams = league['teamID']
teamID = []
for team in teams:
    if not (team in teamID):
        teamID.append(team)

```

## Part 2: Exploratory data analysis

### Payroll distrubution

#### Problem 2

Plots that show payroll distribution accross time for each team



```python
# for each team plot the payroll vs time
for id in teamID:
    t = league.loc[league['teamID'] == id]
    plt.plot(t['yearID'], t['sum(salary)'])
    plt.xlabel('year')
    plt.ylabel('payroll')
    plt.title(t['name'][1:2] + ' Payroll vs Time')
    plt.show()
```


![png](img/output_16_0.png)



![png](img/output_16_1.png)



![png](img/output_16_2.png)



![png](img/output_16_3.png)



![png](img/output_16_4.png)



![png](img/img/output_16_5.png)



![png](img/output_16_6.png)



![png](img/output_16_7.png)



![png](img/output_16_8.png)



![png](img/output_16_9.png)



![png](img/output_16_10.png)



![png](img/output_16_11.png)



![png](img/output_16_12.png)



![png](img/output_16_13.png)



![png](img/output_16_14.png)



![png](img/output_16_15.png)



![png](img/output_16_16.png)



![png](img/output_16_17.png)



![png](img/output_16_18.png)



![png](img/output_16_19.png)



![png](img/output_16_20.png)



![png](img/output_16_21.png)



![png](img/output_16_22.png)



![png](img/output_16_23.png)



![png](img/output_16_24.png)



![png](img/output_16_25.png)



![png](img/output_16_26.png)



![png](img/output_16_27.png)



![png](img/output_16_28.png)



![png](img/output_16_29.png)



![png](img/output_16_30.png)



![png](img/output_16_31.png)



![png](img/output_16_32.png)



![png](img/output_16_33.png)



![png](img/output_16_34.png)


payroll seem to generally increase over time for most teams


```python
#plot below shows a scatter plot of payroll vs time for all teams
plt.scatter(league['yearID'],league['sum(salary)'])
plt.ylabel('salary')
plt.xlabel('time')
plt.title('line Graph of Salary vs Time for all teams')
plt.show()
```


![png](img/output_18_0.png)


#### Question 2
Statements about payroll distribution accross time:
1. Mean payroll has increased over time for most teams
2. Standard deviation of the payrolls accross teams has increased over time


####  Problem 3

1. Plot that shows mean payroll has increased over time


```python
plt.plot(league.groupby('yearID')['sum(salary)'].mean())
plt.xlabel('Time(years)')
plt.ylabel('mean payroll')
plt.title('Line Graph of mean payroll vs Time (years) for all teams')
plt.show()
```


![png](img/output_22_0.png)


Plot above has a positive slope showing that mean payroll has continously increased over time

2. Plot shows standard deviation of payrolls has increased over time


```python
year = 1990
y = []

# loop through all the 25 years and get the salaries for all teams in each year
# store the salaries in list y
for x in range(0,25):
    t = league.loc[league['yearID'] == (year + x)]['sum(salary)']
    y.append(t)

# plot the 
plt.boxplot(y)
plt.xlabel('time in years')
plt.ylabel('Payroll')
plt.title('Box plots showing Payroll vs time in years')
plt.show()
   
```


![png](img/output_25_0.png)


The boxplots have generally increased in height over time showing that the standard deviation of salaries 
has generally increased over time.

### Correlation between payroll and winning percentage


```python

x = league['sum(salary)']
y = league['win_rate']
plt.scatter( x,y )
plt.xlabel('Payroll')
plt.ylabel('win percentage')
plt.title('Scatter plot for Payroll vs win rate for all teams')

# regression line
fit  = np.polyfit( x, y, 1 )
plt.plot(x, fit[0] * x + fit[1], color = 'red')
plt.show()

```


![png](img/output_28_0.png)


According to the regression line, as payroll increases win percentage increases

#### Problem 4


```python
# Create column category that indicates time period for each row
league.loc[: , 'category'] = pd.cut(league['yearID'],5, precision = 0, labels = ['1', '2', '3', '4', '5'])
league.loc[: , 'category'].astype(int)
league.head()

```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>yearID</th>
      <th>lgID</th>
      <th>teamID</th>
      <th>franchID</th>
      <th>G</th>
      <th>W</th>
      <th>name</th>
      <th>sum(salary)</th>
      <th>win_rate</th>
      <th>category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>130</th>
      <td>1990</td>
      <td>AL</td>
      <td>BAL</td>
      <td>BAL</td>
      <td>161</td>
      <td>76</td>
      <td>Baltimore Orioles</td>
      <td>9680084.0</td>
      <td>47.204969</td>
      <td>1</td>
    </tr>
    <tr>
      <th>131</th>
      <td>1990</td>
      <td>AL</td>
      <td>BOS</td>
      <td>BOS</td>
      <td>162</td>
      <td>88</td>
      <td>Boston Red Sox</td>
      <td>20558333.0</td>
      <td>54.320988</td>
      <td>1</td>
    </tr>
    <tr>
      <th>132</th>
      <td>1990</td>
      <td>AL</td>
      <td>CAL</td>
      <td>ANA</td>
      <td>162</td>
      <td>80</td>
      <td>California Angels</td>
      <td>21720000.0</td>
      <td>49.382716</td>
      <td>1</td>
    </tr>
    <tr>
      <th>133</th>
      <td>1990</td>
      <td>AL</td>
      <td>CHA</td>
      <td>CHW</td>
      <td>162</td>
      <td>94</td>
      <td>Chicago White Sox</td>
      <td>9491500.0</td>
      <td>58.024691</td>
      <td>1</td>
    </tr>
    <tr>
      <th>134</th>
      <td>1990</td>
      <td>AL</td>
      <td>CLE</td>
      <td>CLE</td>
      <td>162</td>
      <td>77</td>
      <td>Cleveland Indians</td>
      <td>14487000.0</td>
      <td>47.530864</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



Below is a scatter plot for Mean win percentage vs Mean win payroll for all team


```python
x = league.groupby(['teamID', 'category'])['sum(salary)'].mean()
y = league.groupby(['teamID', 'category'])['win_rate'].mean()

plt.scatter( x , y)
plt.xlabel('mean payroll')
plt.ylabel('mean win percentage')
plt.title('Mean Win percentage vs mean payroll for all teams')

#regression line
fit  = np.polyfit( x, y, 1 )
plt.plot(x, fit[0] * x + fit[1], color = 'red')
plt.show()


```


![png](img/output_33_0.png)


Below are graphs for win rate vs payroll for each team


```python
mean_payroll = league.groupby(['teamID', 'category'])['sum(salary)'].mean()
mean_winrate = league.groupby(['teamID', 'category'])['win_rate'].mean()

# lists below contain the co-efficicents and constants of the regression lines for each team 
co_eff = []
const = []
t_ids = []
for id in league['teamID']:
    
    t = league.loc[league['teamID'] == id]
    x = mean_payroll[id]
    y = mean_winrate[id]
    
    plt.scatter(x , y )    
    plt.xlabel('mean payroll')
    plt.ylabel('mean win percentage')
    plt.title(t["name"][1:2] + "mean win percentage vs mean payroll")
    
    # regression line
    fit  = np.polyfit( x, y, 1 )
    plt.plot(x, fit[0] * x + fit[1], color = 'red')
    
    if not(id in t_ids):
        t_ids.append(id)
        co_eff.append(fit[0])
        const.append(fit[1])
        
    plt.show()
    

```


![png](img/output_35_0.png)



![png](img/output_35_1.png)



![png](img/output_35_2.png)



![png](img/output_35_3.png)



![png](img/output_35_4.png)



![png](img/output_35_5.png)



![png](img/output_35_6.png)



![png](img/output_35_7.png)



![png](img/output_35_8.png)



![png](img/output_35_9.png)



![png](img/output_35_10.png)



![png](img/output_35_11.png)



![png](img/output_35_12.png)



![png](img/output_35_13.png)



![png](img/output_35_14.png)



![png](img/output_35_15.png)



![png](img/output_35_16.png)



![png](img/output_35_17.png)



![png](img/output_35_18.png)



![png](img/output_35_19.png)



![png](img/output_35_20.png)



![png](img/output_35_21.png)



![png](img/output_35_22.png)



![png](img/output_35_23.png)



![png](img/output_35_24.png)



![png](img/output_35_25.png)



![png](img/output_35_26.png)



![png](img/output_35_27.png)



![png](img/output_35_28.png)



![png](img/output_35_29.png)



![png](img/output_35_30.png)



![png](img/output_35_31.png)



![png](img/output_35_32.png)



![png](img/output_35_33.png)



![png](img/output_35_34.png)



![png](img/output_35_35.png)



![png](img/output_35_36.png)



![png](img/output_35_37.png)



![png](img/output_35_38.png)



![png](img/output_35_39.png)



![png](img/output_35_40.png)



![png](img/output_35_41.png)



![png](img/output_35_42.png)



![png](img/output_35_43.png)



![png](img/output_35_44.png)



![png](img/output_35_45.png)



![png](img/output_35_46.png)



![png](img/output_35_47.png)



![png](img/output_35_48.png)



![png](img/output_35_49.png)



![png](img/output_35_50.png)



![png](img/output_35_51.png)



![png](img/output_35_52.png)



![png](img/output_35_53.png)



![png](img/output_35_54.png)



![png](img/output_35_55.png)



![png](img/output_35_56.png)



![png](img/output_35_57.png)



![png](img/output_35_58.png)



![png](img/output_35_59.png)



![png](img/output_35_60.png)



![png](img/output_35_61.png)



![png](img/output_35_62.png)



![png](img/output_35_63.png)



![png](img/output_35_64.png)



![png](img/output_35_65.png)



![png](img/output_35_66.png)



![png](img/output_35_67.png)



![png](img/output_35_68.png)



![png](img/output_35_69.png)



![png](img/output_35_70.png)



![png](img/output_35_71.png)



![png](img/output_35_72.png)



![png](img/output_35_73.png)



![png](img/output_35_74.png)



![png](img/output_35_75.png)



![png](img/output_35_76.png)



![png](img/output_35_77.png)



![png](img/output_35_78.png)



![png](img/output_35_79.png)



![png](img/output_35_80.png)



![png](img/output_35_81.png)



![png](img/output_35_82.png)



![png](img/output_35_83.png)



![png](img/output_35_84.png)



![png](img/output_35_85.png)



![png](img/output_35_86.png)



![png](img/output_35_87.png)



![png](img/output_35_88.png)



![png](img/output_35_89.png)



![png](img/output_35_90.png)



![png](img/output_35_91.png)



![png](img/output_35_92.png)



![png](img/output_35_93.png)



![png](img/output_35_94.png)



![png](img/output_35_95.png)



![png](img/output_35_96.png)



![png](img/output_35_97.png)



![png](img/output_35_98.png)



![png](img/output_35_99.png)



![png](img/output_35_100.png)



![png](img/output_35_101.png)



![png](img/output_35_102.png)



![png](img/output_35_103.png)



![png](img/output_35_104.png)



![png](img/output_35_105.png)



![png](img/output_35_106.png)



![png](img/output_35_107.png)



![png](img/output_35_108.png)



![png](img/output_35_109.png)



![png](img/output_35_110.png)



![png](img/output_35_111.png)



![png](img/output_35_112.png)



![png](img/output_35_113.png)



![png](img/output_35_114.png)



![png](img/output_35_115.png)



![png](img/output_35_116.png)



![png](img/output_35_117.png)



![png](img/output_35_118.png)



![png](img/output_35_119.png)



![png](img/output_35_120.png)



![png](img/output_35_121.png)



![png](img/output_35_122.png)



![png](img/output_35_123.png)



![png](img/output_35_124.png)



![png](img/output_35_125.png)



![png](img/output_35_126.png)



![png](img/output_35_127.png)



![png](img/output_35_128.png)



![png](img/output_35_129.png)



![png](img/output_35_130.png)



![png](img/output_35_131.png)



![png](img/output_35_132.png)



![png](img/output_35_133.png)



![png](img/output_35_134.png)



![png](img/output_35_135.png)



![png](img/output_35_136.png)



![png](img/output_35_137.png)



![png](img/output_35_138.png)



![png](img/output_35_139.png)



![png](img/output_35_140.png)



![png](img/output_35_141.png)



![png](img/output_35_142.png)



![png](img/output_35_143.png)



![png](img/output_35_144.png)



![png](img/output_35_145.png)



![png](img/output_35_146.png)



![png](img/output_35_147.png)



![png](img/output_35_148.png)



![png](img/output_35_149.png)



![png](img/output_35_150.png)



![png](img/output_35_151.png)



![png](img/output_35_152.png)



![png](img/output_35_153.png)



![png](img/output_35_154.png)



![png](img/output_35_155.png)



![png](img/output_35_156.png)



![png](img/output_35_157.png)



![png](img/output_35_158.png)



![png](img/output_35_159.png)



![png](img/output_35_160.png)



![png](img/output_35_161.png)



![png](img/output_35_162.png)



![png](img/output_35_163.png)



![png](img/output_35_164.png)



![png](img/output_35_165.png)



![png](img/output_35_166.png)



![png](img/output_35_167.png)



![png](img/output_35_168.png)



![png](img/output_35_169.png)



![png](img/output_35_170.png)



![png](img/output_35_171.png)



![png](img/output_35_172.png)



![png](img/output_35_173.png)



![png](img/output_35_174.png)



![png](img/output_35_175.png)



![png](img/output_35_176.png)



![png](img/output_35_177.png)



![png](img/output_35_178.png)



![png](img/output_35_179.png)



![png](img/output_35_180.png)



![png](img/output_35_181.png)



![png](img/output_35_182.png)



![png](img/output_35_183.png)



![png](img/output_35_184.png)



![png](img/output_35_185.png)



![png](img/output_35_186.png)



![png](img/output_35_187.png)



![png](img/output_35_188.png)



![png](img/output_35_189.png)



![png](img/output_35_190.png)



![png](img/output_35_191.png)



![png](img/output_35_192.png)



![png](img/output_35_193.png)



![png](img/output_35_194.png)



![png](img/output_35_195.png)



![png](img/output_35_196.png)



![png](img/output_35_197.png)



![png](img/output_35_198.png)



![png](img/output_35_199.png)



![png](img/output_35_200.png)



![png](img/output_35_201.png)



![png](img/output_35_202.png)



![png](img/output_35_203.png)



![png](img/output_35_204.png)



![png](img/output_35_205.png)



![png](img/output_35_206.png)



![png](img/output_35_207.png)



![png](img/output_35_208.png)



![png](img/output_35_209.png)



![png](img/output_35_210.png)



![png](img/output_35_211.png)



![png](img/output_35_212.png)



![png](img/output_35_213.png)



![png](img/output_35_214.png)



![png](img/output_35_215.png)



![png](img/output_35_216.png)



![png](img/output_35_217.png)



![png](img/output_35_218.png)



![png](img/output_35_219.png)



![png](img/output_35_220.png)



![png](img/output_35_221.png)



![png](img/output_35_222.png)



![png](img/output_35_223.png)



![png](img/output_35_224.png)



![png](img/output_35_225.png)



![png](img/output_35_226.png)



![png](img/output_35_227.png)



![png](img/output_35_228.png)



![png](img/output_35_229.png)



![png](img/output_35_230.png)



![png](img/output_35_231.png)



![png](img/output_35_232.png)



![png](img/output_35_233.png)



![png](img/output_35_234.png)



![png](img/output_35_235.png)



![png](img/output_35_236.png)



![png](img/output_35_237.png)



![png](img/output_35_238.png)



![png](img/output_35_239.png)



![png](img/output_35_240.png)



![png](img/output_35_241.png)



![png](img/output_35_242.png)



![png](img/output_35_243.png)



![png](img/output_35_244.png)



![png](img/output_35_245.png)



![png](img/output_35_246.png)



![png](img/output_35_247.png)



![png](img/output_35_248.png)



![png](img/output_35_249.png)



![png](img/output_35_250.png)



![png](img/output_35_251.png)



![png](img/output_35_252.png)



![png](img/output_35_253.png)



![png](img/output_35_254.png)



![png](img/output_35_255.png)



![png](img/output_35_256.png)



![png](img/output_35_257.png)



![png](img/output_35_258.png)



![png](img/output_35_259.png)



![png](img/output_35_260.png)



![png](img/output_35_261.png)



![png](img/output_35_262.png)



![png](img/output_35_263.png)



![png](img/output_35_264.png)



![png](img/output_35_265.png)



![png](img/output_35_266.png)



![png](img/output_35_267.png)



![png](img/output_35_268.png)



![png](img/output_35_269.png)



![png](img/output_35_270.png)



![png](img/output_35_271.png)



![png](img/output_35_272.png)



![png](img/output_35_273.png)



![png](img/output_35_274.png)



![png](img/output_35_275.png)



![png](img/output_35_276.png)



![png](img/output_35_277.png)



![png](img/output_35_278.png)



![png](img/output_35_279.png)



![png](img/output_35_280.png)



![png](img/output_35_281.png)



![png](img/output_35_282.png)



![png](img/output_35_283.png)



![png](img/output_35_284.png)



![png](img/output_35_285.png)



![png](img/output_35_286.png)



![png](img/output_35_287.png)



![png](img/output_35_288.png)



![png](img/output_35_289.png)



![png](img/output_35_290.png)



![png](img/output_35_291.png)



![png](img/output_35_292.png)



![png](img/output_35_293.png)



![png](img/output_35_294.png)



![png](img/output_35_295.png)



![png](img/output_35_296.png)



![png](img/output_35_297.png)



![png](img/output_35_298.png)



![png](img/output_35_299.png)



![png](img/output_35_300.png)



![png](img/output_35_301.png)



![png](img/output_35_302.png)



![png](img/output_35_303.png)



![png](img/output_35_304.png)



![png](img/output_35_305.png)



![png](img/output_35_306.png)



![png](img/output_35_307.png)



![png](img/output_35_308.png)



![png](img/output_35_309.png)



![png](img/output_35_310.png)



![png](img/output_35_311.png)



![png](img/output_35_312.png)



![png](img/output_35_313.png)



![png](img/output_35_314.png)



![png](img/output_35_315.png)



![png](img/output_35_316.png)



![png](img/output_35_317.png)



![png](img/output_35_318.png)



![png](img/output_35_319.png)



![png](img/output_35_320.png)



![png](img/output_35_321.png)



![png](img/output_35_322.png)



![png](img/output_35_323.png)



![png](img/output_35_324.png)



![png](img/output_35_325.png)



![png](img/output_35_326.png)



![png](img/output_35_327.png)



![png](img/output_35_328.png)



![png](img/output_35_329.png)



![png](img/output_35_330.png)



![png](img/output_35_331.png)



![png](img/output_35_332.png)



![png](img/output_35_333.png)



![png](img/output_35_334.png)



![png](img/output_35_335.png)



![png](img/output_35_336.png)



![png](img/output_35_337.png)



![png](img/output_35_338.png)



![png](img/output_35_339.png)



![png](img/output_35_340.png)



![png](img/output_35_341.png)



![png](img/output_35_342.png)



![png](img/output_35_343.png)



![png](img/output_35_344.png)



![png](img/output_35_345.png)



![png](img/output_35_346.png)



![png](img/output_35_347.png)



![png](img/output_35_348.png)



![png](img/output_35_349.png)



![png](img/output_35_350.png)



![png](img/output_35_351.png)



![png](img/output_35_352.png)



![png](img/output_35_353.png)



![png](img/output_35_354.png)



![png](img/output_35_355.png)



![png](img/output_35_356.png)



![png](img/output_35_357.png)



![png](img/output_35_358.png)



![png](img/output_35_359.png)



![png](img/output_35_360.png)



![png](img/output_35_361.png)



![png](img/output_35_362.png)



![png](img/output_35_363.png)



![png](img/output_35_364.png)



![png](img/output_35_365.png)



![png](img/output_35_366.png)



![png](img/output_35_367.png)



![png](img/output_35_368.png)



![png](img/output_35_369.png)



![png](img/output_35_370.png)



![png](img/output_35_371.png)



![png](img/output_35_372.png)



![png](img/output_35_373.png)



![png](img/output_35_374.png)



![png](img/output_35_375.png)



![png](img/output_35_376.png)



![png](img/output_35_377.png)



![png](img/output_35_378.png)



![png](img/output_35_379.png)



![png](img/output_35_380.png)



![png](img/output_35_381.png)



![png](img/output_35_382.png)



![png](img/output_35_383.png)



![png](img/output_35_384.png)



![png](img/output_35_385.png)



![png](img/output_35_386.png)



![png](img/output_35_387.png)



![png](img/output_35_388.png)



![png](img/output_35_389.png)



![png](img/output_35_390.png)



![png](img/output_35_391.png)



![png](img/output_35_392.png)



![png](img/output_35_393.png)



![png](img/output_35_394.png)



![png](img/output_35_395.png)



![png](img/output_35_396.png)



![png](img/output_35_397.png)



![png](img/output_35_398.png)



![png](img/output_35_399.png)



![png](img/output_35_400.png)



![png](img/output_35_401.png)



![png](img/output_35_402.png)



![png](img/output_35_403.png)



![png](img/output_35_404.png)



![png](img/output_35_405.png)



![png](img/output_35_406.png)



![png](img/output_35_407.png)



![png](img/output_35_408.png)



![png](img/output_35_409.png)



![png](img/output_35_410.png)



![png](img/output_35_411.png)



![png](img/output_35_412.png)



![png](img/output_35_413.png)



![png](img/output_35_414.png)



![png](img/output_35_415.png)



![png](img/output_35_416.png)



![png](img/output_35_417.png)



![png](img/output_35_418.png)



![png](img/output_35_419.png)



![png](img/output_35_420.png)



![png](img/output_35_421.png)



![png](img/output_35_422.png)



![png](img/output_35_423.png)



![png](img/output_35_424.png)



![png](img/output_35_425.png)



![png](img/output_35_426.png)



![png](img/output_35_427.png)



![png](img/output_35_428.png)



![png](img/output_35_429.png)



![png](img/output_35_430.png)



![png](img/output_35_431.png)



![png](img/output_35_432.png)



![png](img/output_35_433.png)



![png](img/output_35_434.png)



![png](img/output_35_435.png)



![png](img/output_35_436.png)



![png](img/output_35_437.png)



![png](img/output_35_438.png)



![png](img/output_35_439.png)



![png](img/output_35_440.png)



![png](img/output_35_441.png)



![png](img/output_35_442.png)



![png](img/output_35_443.png)



![png](img/output_35_444.png)



![png](img/output_35_445.png)



![png](img/output_35_446.png)



![png](img/output_35_447.png)



![png](img/output_35_448.png)



![png](img/output_35_449.png)



![png](img/output_35_450.png)



![png](img/output_35_451.png)



![png](img/output_35_452.png)



![png](img/output_35_453.png)



![png](img/output_35_454.png)



![png](img/output_35_455.png)



![png](img/output_35_456.png)



![png](img/output_35_457.png)



![png](img/output_35_458.png)



![png](img/output_35_459.png)



![png](img/output_35_460.png)



![png](img/output_35_461.png)



![png](img/output_35_462.png)



![png](img/output_35_463.png)



![png](img/output_35_464.png)



![png](img/output_35_465.png)



![png](img/output_35_466.png)



![png](img/output_35_467.png)



![png](img/output_35_468.png)



![png](img/output_35_469.png)



![png](img/output_35_470.png)



![png](img/output_35_471.png)



![png](img/output_35_472.png)



![png](img/output_35_473.png)



![png](img/output_35_474.png)



![png](img/output_35_475.png)



![png](img/output_35_476.png)



![png](img/output_35_477.png)



![png](img/output_35_478.png)



![png](img/output_35_479.png)



![png](img/output_35_480.png)



![png](img/output_35_481.png)



![png](img/output_35_482.png)



![png](img/output_35_483.png)



![png](img/output_35_484.png)



![png](img/output_35_485.png)



![png](img/output_35_486.png)



![png](img/output_35_487.png)



![png](img/output_35_488.png)



![png](img/output_35_489.png)



![png](img/output_35_490.png)



![png](img/output_35_491.png)



![png](img/output_35_492.png)



![png](img/output_35_493.png)



![png](img/output_35_494.png)



![png](img/output_35_495.png)



![png](img/output_35_496.png)



![png](img/output_35_497.png)



![png](img/output_35_498.png)



![png](img/output_35_499.png)



![png](img/output_35_500.png)



![png](img/output_35_501.png)



![png](img/output_35_502.png)



![png](img/output_35_503.png)



![png](img/output_35_504.png)



![png](img/output_35_505.png)



![png](img/output_35_506.png)



![png](img/output_35_507.png)



![png](img/output_35_508.png)



![png](img/output_35_509.png)



![png](img/output_35_510.png)



![png](img/output_35_511.png)



![png](img/output_35_512.png)



![png](img/output_35_513.png)



![png](img/output_35_514.png)



![png](img/output_35_515.png)



![png](img/output_35_516.png)



![png](img/output_35_517.png)



![png](img/output_35_518.png)



![png](img/output_35_519.png)



![png](img/output_35_520.png)



![png](img/output_35_521.png)



![png](img/output_35_522.png)



![png](img/output_35_523.png)



![png](img/output_35_524.png)



![png](img/output_35_525.png)



![png](img/output_35_526.png)



![png](img/output_35_527.png)



![png](img/output_35_528.png)



![png](img/output_35_529.png)



![png](img/output_35_530.png)



![png](img/output_35_531.png)



![png](img/output_35_532.png)



![png](img/output_35_533.png)



![png](img/output_35_534.png)



![png](img/output_35_535.png)



![png](img/output_35_536.png)



![png](img/output_35_537.png)



![png](img/output_35_538.png)



![png](img/output_35_539.png)



![png](img/output_35_540.png)



![png](img/output_35_541.png)



![png](img/output_35_542.png)



![png](img/output_35_543.png)



![png](img/output_35_544.png)



![png](img/output_35_545.png)



![png](img/output_35_546.png)



![png](img/output_35_547.png)



![png](img/output_35_548.png)



![png](img/output_35_549.png)



![png](img/output_35_550.png)



![png](img/output_35_551.png)



![png](img/output_35_552.png)



![png](img/output_35_553.png)



![png](img/output_35_554.png)



![png](img/output_35_555.png)



![png](img/output_35_556.png)



![png](img/output_35_557.png)



![png](img/output_35_558.png)



![png](img/output_35_559.png)



![png](img/output_35_560.png)



![png](img/output_35_561.png)



![png](img/output_35_562.png)



![png](img/output_35_563.png)



![png](img/output_35_564.png)



![png](img/output_35_565.png)



![png](img/output_35_566.png)



![png](img/output_35_567.png)



![png](img/output_35_568.png)



![png](img/output_35_569.png)



![png](img/output_35_570.png)



![png](img/output_35_571.png)



![png](img/output_35_572.png)



![png](img/output_35_573.png)



![png](img/output_35_574.png)



![png](img/output_35_575.png)



![png](img/output_35_576.png)



![png](img/output_35_577.png)



![png](img/output_35_578.png)



![png](img/output_35_579.png)



![png](img/output_35_580.png)



![png](img/output_35_581.png)



![png](img/output_35_582.png)



![png](img/output_35_583.png)



![png](img/output_35_584.png)



![png](img/output_35_585.png)



![png](img/output_35_586.png)



![png](img/output_35_587.png)



![png](img/output_35_588.png)



![png](img/output_35_589.png)



![png](img/output_35_590.png)



![png](img/output_35_591.png)



![png](img/output_35_592.png)



![png](img/output_35_593.png)



![png](img/output_35_594.png)



![png](img/output_35_595.png)



![png](img/output_35_596.png)



![png](img/output_35_597.png)



![png](img/output_35_598.png)



![png](img/output_35_599.png)



![png](img/output_35_600.png)



![png](img/output_35_601.png)



![png](img/output_35_602.png)



![png](img/output_35_603.png)



![png](img/output_35_604.png)



![png](img/output_35_605.png)



![png](img/output_35_606.png)



![png](img/output_35_607.png)



![png](img/output_35_608.png)



![png](img/output_35_609.png)



![png](img/output_35_610.png)



![png](img/output_35_611.png)



![png](img/output_35_612.png)



![png](img/output_35_613.png)



![png](img/output_35_614.png)



![png](img/output_35_615.png)



![png](img/output_35_616.png)



![png](img/output_35_617.png)



![png](img/output_35_618.png)



![png](img/output_35_619.png)



![png](img/output_35_620.png)



![png](img/output_35_621.png)



![png](img/output_35_622.png)



![png](img/output_35_623.png)



![png](img/output_35_624.png)



![png](img/output_35_625.png)



![png](img/output_35_626.png)



![png](img/output_35_627.png)



![png](img/output_35_628.png)



![png](img/output_35_629.png)



![png](img/output_35_630.png)



![png](img/output_35_631.png)



![png](img/output_35_632.png)



![png](img/output_35_633.png)



![png](img/output_35_634.png)



![png](img/output_35_635.png)



![png](img/output_35_636.png)



![png](img/output_35_637.png)



![png](img/output_35_638.png)



![png](img/output_35_639.png)



![png](img/output_35_640.png)



![png](img/output_35_641.png)



![png](img/output_35_642.png)



![png](img/output_35_643.png)



![png](img/output_35_644.png)



![png](img/output_35_645.png)



![png](img/output_35_646.png)



![png](img/output_35_647.png)



![png](img/output_35_648.png)



![png](img/output_35_649.png)



![png](img/output_35_650.png)



![png](img/output_35_651.png)



![png](img/output_35_652.png)



![png](img/output_35_653.png)



![png](img/output_35_654.png)



![png](img/output_35_655.png)



![png](img/output_35_656.png)



![png](img/output_35_657.png)



![png](img/output_35_658.png)


    C:\Users\Owner\Anaconda3\lib\site-packages\numpy\lib\polynomial.py:595: RankWarning: Polyfit may be poorly conditioned
      warnings.warn(msg, RankWarning)



![png](img/output_35_660.png)



![png](img/output_35_661.png)



![png](img/output_35_662.png)



![png](img/output_35_663.png)



![png](img/output_35_664.png)



![png](img/output_35_665.png)



![png](img/output_35_666.png)



![png](img/output_35_667.png)



![png](img/output_35_668.png)



![png](img/output_35_669.png)



![png](img/output_35_670.png)



![png](img/output_35_671.png)



![png](img/output_35_672.png)



![png](img/output_35_673.png)



![png](img/output_35_674.png)



![png](img/output_35_675.png)



![png](img/output_35_676.png)



![png](img/output_35_677.png)



![png](img/output_35_678.png)



![png](img/output_35_679.png)



![png](img/output_35_680.png)



![png](img/output_35_681.png)



![png](img/output_35_682.png)



![png](img/output_35_683.png)



![png](img/output_35_684.png)



![png](img/output_35_685.png)



![png](img/output_35_686.png)



![png](img/output_35_687.png)



![png](img/output_35_688.png)



![png](img/output_35_689.png)


    C:\Users\Owner\Anaconda3\lib\site-packages\numpy\lib\polynomial.py:595: RankWarning: Polyfit may be poorly conditioned
      warnings.warn(msg, RankWarning)



![png](img/output_35_691.png)



![png](img/output_35_692.png)



![png](img/output_35_693.png)



![png](img/output_35_694.png)



![png](img/output_35_695.png)



![png](img/output_35_696.png)



![png](img/output_35_697.png)



![png](img/output_35_698.png)



![png](img/output_35_699.png)



![png](img/output_35_700.png)



![png](img/output_35_701.png)



![png](img/output_35_702.png)



![png](img/output_35_703.png)



![png](img/output_35_704.png)



![png](img/output_35_705.png)



![png](img/output_35_706.png)



![png](img/output_35_707.png)



![png](img/output_35_708.png)



![png](img/output_35_709.png)



![png](img/output_35_710.png)



![png](img/output_35_711.png)



![png](img/output_35_712.png)



![png](img/output_35_713.png)



![png](img/output_35_714.png)



![png](img/output_35_715.png)



![png](img/output_35_716.png)



![png](img/output_35_717.png)



![png](img/output_35_718.png)



![png](img/output_35_719.png)


    C:\Users\Owner\Anaconda3\lib\site-packages\numpy\lib\polynomial.py:595: RankWarning: Polyfit may be poorly conditioned
      warnings.warn(msg, RankWarning)



![png](img/output_35_721.png)



![png](img/output_35_722.png)



![png](img/output_35_723.png)



![png](img/output_35_724.png)



![png](img/output_35_725.png)



![png](img/output_35_726.png)



![png](img/output_35_727.png)



![png](img/output_35_728.png)



![png](img/output_35_729.png)


#### Question 2
 
 Inorder to find teams that are most efficient at paying for wins:
1. Create a dataframe that contains the functions for each team
2. Sort the dataframe according to the co-efficients of the regression line
3. The higher the co-efficient is, the better the team is at paying for wins


```python
d = {'teamID': t_ids, 'co-efficients': co_eff, 'constants': const}
funs = pd.DataFrame(data = d)

#the data will be sorted in descending order starting from the most efficient team at paying for wins
funs.sort_values(by = 'co-efficients', ascending = False, inplace = True )
funs.head(12)

```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>co-efficients</th>
      <th>constants</th>
      <th>teamID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>2.210655e-06</td>
      <td>-17.641063</td>
      <td>CAL</td>
    </tr>
    <tr>
      <th>29</th>
      <td>5.275893e-07</td>
      <td>22.510284</td>
      <td>TBA</td>
    </tr>
    <tr>
      <th>34</th>
      <td>3.317414e-07</td>
      <td>21.399177</td>
      <td>MIA</td>
    </tr>
    <tr>
      <th>33</th>
      <td>2.824453e-07</td>
      <td>27.502610</td>
      <td>WAS</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2.456416e-07</td>
      <td>42.482717</td>
      <td>ML4</td>
    </tr>
    <tr>
      <th>31</th>
      <td>1.636089e-07</td>
      <td>37.517806</td>
      <td>MIL</td>
    </tr>
    <tr>
      <th>28</th>
      <td>1.293287e-07</td>
      <td>43.667352</td>
      <td>ANA</td>
    </tr>
    <tr>
      <th>27</th>
      <td>1.132821e-07</td>
      <td>42.740639</td>
      <td>FLO</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1.101527e-07</td>
      <td>39.569099</td>
      <td>DET</td>
    </tr>
    <tr>
      <th>25</th>
      <td>1.025154e-07</td>
      <td>46.119689</td>
      <td>SLN</td>
    </tr>
    <tr>
      <th>10</th>
      <td>7.418883e-08</td>
      <td>49.160040</td>
      <td>OAK</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5.969963e-08</td>
      <td>47.830827</td>
      <td>CLE</td>
    </tr>
  </tbody>
</table>
</div>



According to the sorted dataframe **Oakland** is roughly the 11th best team at spending efficiency


```python
# From the funs dataframe get the best and worst teams at paying for wins
# by taking the top 5 teams and bottom 5 teams in the sorted dataframe
best_teams_name = []
worst_teams_name = []

#most efficient teams are at the top of funs dataframe
best_teams_id = funs['teamID'][:5]

#least efficient teams are at the bottom of funs dataframe
worst_teams_id = funs['teamID'][-5:]

# Get the names of the teams from their ids
for t in best_teams_id:
    best_teams_name.append( league.loc[league['teamID'] == t]['name'][1:2] )

for t in worst_teams_id:
    worst_teams_name.append( league.loc[league['teamID'] == t]['name'][1:2] )
    
print (" Best teams at paying for wins are: ")
print( best_teams_name)
print("\n")
print (" Worst teams at paying for wins are:")
print (worst_teams_name)

```

     Best teams at paying for wins are: 
    [158    California Angels
    Name: name, dtype: object, 389    Tampa Bay Devil Rays
    Name: name, dtype: object, 819    Miami Marlins
    Name: name, dtype: object, 617    Washington Nationals
    Name: name, dtype: object, 164    Milwaukee Brewers
    Name: name, dtype: object]
    
    
     Worst teams at paying for wins are:
    [170    Atlanta Braves
    Name: name, dtype: object, 169    Toronto Blue Jays
    Name: name, dtype: object, 178    Pittsburgh Pirates
    Name: name, dtype: object, 175    Montreal Expos
    Name: name, dtype: object, 594    Los Angeles Angels of Anaheim
    Name: name, dtype: object]


## Part 3: Data Transformations

### Standardizing accross years
#### Problem 5


```python
#get the salary standard deviation and mean salary for each row
salary_std = league.groupby('yearID')['sum(salary)'].std()
salary_mean = league.groupby('yearID')['sum(salary)'].mean()
```


```python
def standardize_pay(row):
    return (  (row['sum(salary)'] - salary_mean[row['yearID']])/salary_std[row['yearID']]  )

#create a new column that shows the standardized payroll accross a five year time period
league['std_payroll'] = league.apply(lambda row: standardize_pay(row), axis = 1)

league.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>yearID</th>
      <th>lgID</th>
      <th>teamID</th>
      <th>franchID</th>
      <th>G</th>
      <th>W</th>
      <th>name</th>
      <th>sum(salary)</th>
      <th>win_rate</th>
      <th>category</th>
      <th>std_payroll</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>130</th>
      <td>1990</td>
      <td>AL</td>
      <td>BAL</td>
      <td>BAL</td>
      <td>161</td>
      <td>76</td>
      <td>Baltimore Orioles</td>
      <td>9680084.0</td>
      <td>47.204969</td>
      <td>1</td>
      <td>-1.959861</td>
    </tr>
    <tr>
      <th>131</th>
      <td>1990</td>
      <td>AL</td>
      <td>BOS</td>
      <td>BOS</td>
      <td>162</td>
      <td>88</td>
      <td>Boston Red Sox</td>
      <td>20558333.0</td>
      <td>54.320988</td>
      <td>1</td>
      <td>0.924213</td>
    </tr>
    <tr>
      <th>132</th>
      <td>1990</td>
      <td>AL</td>
      <td>CAL</td>
      <td>ANA</td>
      <td>162</td>
      <td>80</td>
      <td>California Angels</td>
      <td>21720000.0</td>
      <td>49.382716</td>
      <td>1</td>
      <td>1.232198</td>
    </tr>
    <tr>
      <th>133</th>
      <td>1990</td>
      <td>AL</td>
      <td>CHA</td>
      <td>CHW</td>
      <td>162</td>
      <td>94</td>
      <td>Chicago White Sox</td>
      <td>9491500.0</td>
      <td>58.024691</td>
      <td>1</td>
      <td>-2.009859</td>
    </tr>
    <tr>
      <th>134</th>
      <td>1990</td>
      <td>AL</td>
      <td>CLE</td>
      <td>CLE</td>
      <td>162</td>
      <td>77</td>
      <td>Cleveland Indians</td>
      <td>14487000.0</td>
      <td>47.530864</td>
      <td>1</td>
      <td>-0.685437</td>
    </tr>
  </tbody>
</table>
</div>




#### Problem 6

Graphs showing mean winnning percentage vs mean standardized payroll for each team


```python
mean_std_payroll = league.groupby(['teamID', 'category'])['std_payroll'].mean()
mean_winrate = league.groupby(['teamID', 'category'])['win_rate'].mean()

for id in league['teamID']:
    t = league.loc[league['teamID'] == id]
    
    x = mean_std_payroll[id]
    y = mean_winrate[id]
    plt.scatter( x, y )    
    plt.xlabel('mean standard payroll')
    plt.ylabel('mean win_rate')
    plt.title(t["name"][1:2] + " mean win_rate vs mean standard payroll")
    
     # regression line 
    fit = np.polyfit(x,y,1)
    plt.plot(x, fit[0]*x + fit[1], color = 'red')
    
    plt.show()
```


![png](img/output_45_0.png)



![png](img/output_45_1.png)



![png](img/output_45_2.png)



![png](img/output_45_3.png)



![png](img/output_45_4.png)



![png](img/output_45_5.png)



![png](img/output_45_6.png)



![png](img/output_45_7.png)



![png](img/output_45_8.png)



![png](img/output_45_9.png)



![png](img/output_45_10.png)



![png](img/output_45_11.png)



![png](img/output_45_12.png)



![png](img/output_45_13.png)



![png](img/output_45_14.png)



![png](img/output_45_15.png)



![png](img/output_45_16.png)



![png](img/output_45_17.png)



![png](img/output_45_18.png)



![png](img/output_45_19.png)



![png](img/output_45_20.png)



![png](img/output_45_21.png)



![png](img/output_45_22.png)



![png](img/output_45_23.png)



![png](img/output_45_24.png)



![png](img/output_45_25.png)



![png](img/output_45_26.png)



![png](img/output_45_27.png)



![png](img/output_45_28.png)



![png](img/output_45_29.png)



![png](img/output_45_30.png)



![png](img/output_45_31.png)



![png](img/output_45_32.png)



![png](img/output_45_33.png)



![png](img/output_45_34.png)



![png](img/output_45_35.png)



![png](img/output_45_36.png)



![png](img/output_45_37.png)



![png](img/output_45_38.png)



![png](img/output_45_39.png)



![png](img/output_45_40.png)



![png](img/output_45_41.png)



![png](img/output_45_42.png)



![png](img/output_45_43.png)



![png](img/output_45_44.png)



![png](img/output_45_45.png)



![png](img/output_45_46.png)



![png](img/output_45_47.png)



![png](img/output_45_48.png)



![png](img/output_45_49.png)



![png](img/output_45_50.png)



![png](img/output_45_51.png)



![png](img/output_45_52.png)



![png](img/output_45_53.png)



![png](img/output_45_54.png)



![png](img/output_45_55.png)



![png](img/output_45_56.png)



![png](img/output_45_57.png)



![png](img/output_45_58.png)



![png](img/output_45_59.png)



![png](img/output_45_60.png)



![png](img/output_45_61.png)



![png](img/output_45_62.png)



![png](img/output_45_63.png)



![png](img/output_45_64.png)



![png](img/output_45_65.png)



![png](img/output_45_66.png)



![png](img/output_45_67.png)



![png](img/output_45_68.png)



![png](img/output_45_69.png)



![png](img/output_45_70.png)



![png](img/output_45_71.png)



![png](img/output_45_72.png)



![png](img/output_45_73.png)



![png](img/output_45_74.png)



![png](img/output_45_75.png)



![png](img/output_45_76.png)



![png](img/output_45_77.png)



![png](img/output_45_78.png)



![png](img/output_45_79.png)



![png](img/output_45_80.png)



![png](img/output_45_81.png)



![png](img/output_45_82.png)



![png](img/output_45_83.png)



![png](img/output_45_84.png)



![png](img/output_45_85.png)



![png](img/output_45_86.png)



![png](img/output_45_87.png)



![png](img/output_45_88.png)



![png](img/output_45_89.png)



![png](img/output_45_90.png)



![png](img/output_45_91.png)



![png](img/output_45_92.png)



![png](img/output_45_93.png)



![png](img/output_45_94.png)



![png](img/output_45_95.png)



![png](img/output_45_96.png)



![png](img/output_45_97.png)



![png](img/output_45_98.png)



![png](img/output_45_99.png)



![png](img/output_45_100.png)



![png](img/output_45_101.png)



![png](img/output_45_102.png)



![png](img/output_45_103.png)



![png](img/output_45_104.png)



![png](img/output_45_105.png)



![png](img/output_45_106.png)



![png](img/output_45_107.png)



![png](img/output_45_108.png)



![png](img/output_45_109.png)



![png](img/output_45_110.png)



![png](img/output_45_111.png)



![png](img/output_45_112.png)



![png](img/output_45_113.png)



![png](img/output_45_114.png)



![png](img/output_45_115.png)



![png](img/output_45_116.png)



![png](img/output_45_117.png)



![png](img/output_45_118.png)



![png](img/output_45_119.png)



![png](img/output_45_120.png)



![png](img/output_45_121.png)



![png](img/output_45_122.png)



![png](img/output_45_123.png)



![png](img/output_45_124.png)



![png](img/output_45_125.png)



![png](img/output_45_126.png)



![png](img/output_45_127.png)



![png](img/output_45_128.png)



![png](img/output_45_129.png)



![png](img/output_45_130.png)



![png](img/output_45_131.png)



![png](img/output_45_132.png)



![png](img/output_45_133.png)



![png](img/output_45_134.png)



![png](img/output_45_135.png)



![png](img/output_45_136.png)



![png](img/output_45_137.png)



![png](img/output_45_138.png)



![png](img/output_45_139.png)



![png](img/output_45_140.png)



![png](img/output_45_141.png)



![png](img/output_45_142.png)



![png](img/output_45_143.png)



![png](img/output_45_144.png)



![png](img/output_45_145.png)



![png](img/output_45_146.png)



![png](img/output_45_147.png)



![png](img/output_45_148.png)



![png](img/output_45_149.png)



![png](img/output_45_150.png)



![png](img/output_45_151.png)



![png](img/output_45_152.png)



![png](img/output_45_153.png)



![png](img/output_45_154.png)



![png](img/output_45_155.png)



![png](img/output_45_156.png)



![png](img/output_45_157.png)



![png](img/output_45_158.png)



![png](img/output_45_159.png)



![png](img/output_45_160.png)



![png](img/output_45_161.png)



![png](img/output_45_162.png)



![png](img/output_45_163.png)



![png](img/output_45_164.png)



![png](img/output_45_165.png)



![png](img/output_45_166.png)



![png](img/output_45_167.png)



![png](img/output_45_168.png)



![png](img/output_45_169.png)



![png](img/output_45_170.png)



![png](img/output_45_171.png)



![png](img/output_45_172.png)



![png](img/output_45_173.png)



![png](img/output_45_174.png)



![png](img/output_45_175.png)



![png](img/output_45_176.png)



![png](img/output_45_177.png)



![png](img/output_45_178.png)



![png](img/output_45_179.png)



![png](img/output_45_180.png)



![png](img/output_45_181.png)



![png](img/output_45_182.png)



![png](img/output_45_183.png)



![png](img/output_45_184.png)



![png](img/output_45_185.png)



![png](img/output_45_186.png)



![png](img/output_45_187.png)



![png](img/output_45_188.png)



![png](img/output_45_189.png)



![png](img/output_45_190.png)



![png](img/output_45_191.png)



![png](img/output_45_192.png)



![png](img/output_45_193.png)



![png](img/output_45_194.png)



![png](img/output_45_195.png)



![png](img/output_45_196.png)



![png](img/output_45_197.png)



![png](img/output_45_198.png)



![png](img/output_45_199.png)



![png](img/output_45_200.png)



![png](img/output_45_201.png)



![png](img/output_45_202.png)



![png](img/output_45_203.png)



![png](img/output_45_204.png)



![png](img/output_45_205.png)



![png](img/output_45_206.png)



![png](img/output_45_207.png)



![png](img/output_45_208.png)



![png](img/output_45_209.png)



![png](img/output_45_210.png)



![png](img/output_45_211.png)



![png](img/output_45_212.png)



![png](img/output_45_213.png)



![png](img/output_45_214.png)



![png](img/output_45_215.png)



![png](img/output_45_216.png)



![png](img/output_45_217.png)



![png](img/output_45_218.png)



![png](img/output_45_219.png)



![png](img/output_45_220.png)



![png](img/output_45_221.png)



![png](img/output_45_222.png)



![png](img/output_45_223.png)



![png](img/output_45_224.png)



![png](img/output_45_225.png)



![png](img/output_45_226.png)



![png](img/output_45_227.png)



![png](img/output_45_228.png)



![png](img/output_45_229.png)



![png](img/output_45_230.png)



![png](img/output_45_231.png)



![png](img/output_45_232.png)



![png](img/output_45_233.png)



![png](img/output_45_234.png)



![png](img/output_45_235.png)



![png](img/output_45_236.png)



![png](img/output_45_237.png)



![png](img/output_45_238.png)



![png](img/output_45_239.png)



![png](img/output_45_240.png)



![png](img/output_45_241.png)



![png](img/output_45_242.png)



![png](img/output_45_243.png)



![png](img/output_45_244.png)



![png](img/output_45_245.png)



![png](img/output_45_246.png)



![png](img/output_45_247.png)



![png](img/output_45_248.png)



![png](img/output_45_249.png)



![png](img/output_45_250.png)



![png](img/output_45_251.png)



![png](img/output_45_252.png)



![png](img/output_45_253.png)



![png](img/output_45_254.png)



![png](img/output_45_255.png)



![png](img/output_45_256.png)



![png](img/output_45_257.png)



![png](img/output_45_258.png)



![png](img/output_45_259.png)



![png](img/output_45_260.png)



![png](img/output_45_261.png)



![png](img/output_45_262.png)



![png](img/output_45_263.png)



![png](img/output_45_264.png)



![png](img/output_45_265.png)



![png](img/output_45_266.png)



![png](img/output_45_267.png)



![png](img/output_45_268.png)



![png](img/output_45_269.png)



![png](img/output_45_270.png)



![png](img/output_45_271.png)



![png](img/output_45_272.png)



![png](img/output_45_273.png)



![png](img/output_45_274.png)



![png](img/output_45_275.png)



![png](img/output_45_276.png)



![png](img/output_45_277.png)



![png](img/output_45_278.png)



![png](img/output_45_279.png)



![png](img/output_45_280.png)



![png](img/output_45_281.png)



![png](img/output_45_282.png)



![png](img/output_45_283.png)



![png](img/output_45_284.png)



![png](img/output_45_285.png)



![png](img/output_45_286.png)



![png](img/output_45_287.png)



![png](img/output_45_288.png)



![png](img/output_45_289.png)



![png](img/output_45_290.png)



![png](img/output_45_291.png)



![png](img/output_45_292.png)



![png](img/output_45_293.png)



![png](img/output_45_294.png)



![png](img/output_45_295.png)



![png](img/output_45_296.png)



![png](img/output_45_297.png)



![png](img/output_45_298.png)



![png](img/output_45_299.png)



![png](img/output_45_300.png)



![png](img/output_45_301.png)



![png](img/output_45_302.png)



![png](img/output_45_303.png)



![png](img/output_45_304.png)



![png](img/output_45_305.png)



![png](img/output_45_306.png)



![png](img/output_45_307.png)



![png](img/output_45_308.png)



![png](img/output_45_309.png)



![png](img/output_45_310.png)



![png](img/output_45_311.png)



![png](img/output_45_312.png)



![png](img/output_45_313.png)



![png](img/output_45_314.png)



![png](img/output_45_315.png)



![png](img/output_45_316.png)



![png](img/output_45_317.png)



![png](img/output_45_318.png)



![png](img/output_45_319.png)



![png](img/output_45_320.png)



![png](img/output_45_321.png)



![png](img/output_45_322.png)



![png](img/output_45_323.png)



![png](img/output_45_324.png)



![png](img/output_45_325.png)



![png](img/output_45_326.png)



![png](img/output_45_327.png)



![png](img/output_45_328.png)



![png](img/output_45_329.png)



![png](img/output_45_330.png)



![png](img/output_45_331.png)



![png](img/output_45_332.png)



![png](img/output_45_333.png)



![png](img/output_45_334.png)



![png](img/output_45_335.png)



![png](img/output_45_336.png)



![png](img/output_45_337.png)



![png](img/output_45_338.png)



![png](img/output_45_339.png)



![png](img/output_45_340.png)



![png](img/output_45_341.png)



![png](img/output_45_342.png)



![png](img/output_45_343.png)



![png](img/output_45_344.png)



![png](img/output_45_345.png)



![png](img/output_45_346.png)



![png](img/output_45_347.png)



![png](img/output_45_348.png)



![png](img/output_45_349.png)



![png](img/output_45_350.png)



![png](img/output_45_351.png)



![png](img/output_45_352.png)



![png](img/output_45_353.png)



![png](img/output_45_354.png)



![png](img/output_45_355.png)



![png](img/output_45_356.png)



![png](img/output_45_357.png)



![png](img/output_45_358.png)



![png](img/output_45_359.png)



![png](img/output_45_360.png)



![png](img/output_45_361.png)



![png](img/output_45_362.png)



![png](img/output_45_363.png)



![png](img/output_45_364.png)



![png](img/output_45_365.png)



![png](img/output_45_366.png)



![png](img/output_45_367.png)



![png](img/output_45_368.png)



![png](img/output_45_369.png)



![png](img/output_45_370.png)



![png](img/output_45_371.png)



![png](img/output_45_372.png)



![png](img/output_45_373.png)



![png](img/output_45_374.png)



![png](img/output_45_375.png)



![png](img/output_45_376.png)



![png](img/output_45_377.png)



![png](img/output_45_378.png)



![png](img/output_45_379.png)



![png](img/output_45_380.png)



![png](img/output_45_381.png)



![png](img/output_45_382.png)



![png](img/output_45_383.png)



![png](img/output_45_384.png)



![png](img/output_45_385.png)



![png](img/output_45_386.png)



![png](img/output_45_387.png)



![png](img/output_45_388.png)



![png](img/output_45_389.png)



![png](img/output_45_390.png)



![png](img/output_45_391.png)



![png](img/output_45_392.png)



![png](img/output_45_393.png)



![png](img/output_45_394.png)



![png](img/output_45_395.png)



![png](img/output_45_396.png)



![png](img/output_45_397.png)



![png](img/output_45_398.png)



![png](img/output_45_399.png)



![png](img/output_45_400.png)



![png](img/output_45_401.png)



![png](img/output_45_402.png)



![png](img/output_45_403.png)



![png](img/output_45_404.png)



![png](img/output_45_405.png)



![png](img/output_45_406.png)



![png](img/output_45_407.png)



![png](img/output_45_408.png)



![png](img/output_45_409.png)



![png](img/output_45_410.png)



![png](img/output_45_411.png)



![png](img/output_45_412.png)



![png](img/output_45_413.png)



![png](img/output_45_414.png)



![png](img/output_45_415.png)



![png](img/output_45_416.png)



![png](img/output_45_417.png)



![png](img/output_45_418.png)



![png](img/output_45_419.png)



![png](img/output_45_420.png)



![png](img/output_45_421.png)



![png](img/output_45_422.png)



![png](img/output_45_423.png)



![png](img/output_45_424.png)



![png](img/output_45_425.png)



![png](img/output_45_426.png)



![png](img/output_45_427.png)



![png](img/output_45_428.png)



![png](img/output_45_429.png)



![png](img/output_45_430.png)



![png](img/output_45_431.png)



![png](img/output_45_432.png)



![png](img/output_45_433.png)



![png](img/output_45_434.png)



![png](img/output_45_435.png)



![png](img/output_45_436.png)



![png](img/output_45_437.png)



![png](img/output_45_438.png)



![png](img/output_45_439.png)



![png](img/output_45_440.png)



![png](img/output_45_441.png)



![png](img/output_45_442.png)



![png](img/output_45_443.png)



![png](img/output_45_444.png)



![png](img/output_45_445.png)



![png](img/output_45_446.png)



![png](img/output_45_447.png)



![png](img/output_45_448.png)



![png](img/output_45_449.png)



![png](img/output_45_450.png)



![png](img/output_45_451.png)



![png](img/output_45_452.png)



![png](img/output_45_453.png)



![png](img/output_45_454.png)



![png](img/output_45_455.png)



![png](img/output_45_456.png)



![png](img/output_45_457.png)



![png](img/output_45_458.png)



![png](img/output_45_459.png)



![png](img/output_45_460.png)



![png](img/output_45_461.png)



![png](img/output_45_462.png)



![png](img/output_45_463.png)



![png](img/output_45_464.png)



![png](img/output_45_465.png)



![png](img/output_45_466.png)



![png](img/output_45_467.png)



![png](img/output_45_468.png)



![png](img/output_45_469.png)



![png](img/output_45_470.png)



![png](img/output_45_471.png)



![png](img/output_45_472.png)



![png](img/output_45_473.png)



![png](img/output_45_474.png)



![png](img/output_45_475.png)



![png](img/output_45_476.png)



![png](img/output_45_477.png)



![png](img/output_45_478.png)



![png](img/output_45_479.png)



![png](img/output_45_480.png)



![png](img/output_45_481.png)



![png](img/output_45_482.png)



![png](img/output_45_483.png)



![png](img/output_45_484.png)



![png](img/output_45_485.png)



![png](img/output_45_486.png)



![png](img/output_45_487.png)



![png](img/output_45_488.png)



![png](img/output_45_489.png)



![png](img/output_45_490.png)



![png](img/output_45_491.png)



![png](img/output_45_492.png)



![png](img/output_45_493.png)



![png](img/output_45_494.png)



![png](img/output_45_495.png)



![png](img/output_45_496.png)



![png](img/output_45_497.png)



![png](img/output_45_498.png)



![png](img/output_45_499.png)



![png](img/output_45_500.png)



![png](img/output_45_501.png)



![png](img/output_45_502.png)



![png](img/output_45_503.png)



![png](img/output_45_504.png)



![png](img/output_45_505.png)



![png](img/output_45_506.png)



![png](img/output_45_507.png)



![png](img/output_45_508.png)



![png](img/output_45_509.png)



![png](img/output_45_510.png)



![png](img/output_45_511.png)



![png](img/output_45_512.png)



![png](img/output_45_513.png)



![png](img/output_45_514.png)



![png](img/output_45_515.png)



![png](img/output_45_516.png)



![png](img/output_45_517.png)



![png](img/output_45_518.png)



![png](img/output_45_519.png)



![png](img/output_45_520.png)



![png](img/output_45_521.png)



![png](img/output_45_522.png)



![png](img/output_45_523.png)



![png](img/output_45_524.png)



![png](img/output_45_525.png)



![png](img/output_45_526.png)



![png](img/output_45_527.png)



![png](img/output_45_528.png)



![png](img/output_45_529.png)



![png](img/output_45_530.png)



![png](img/output_45_531.png)



![png](img/output_45_532.png)



![png](img/output_45_533.png)



![png](img/output_45_534.png)



![png](img/output_45_535.png)



![png](img/output_45_536.png)



![png](img/output_45_537.png)



![png](img/output_45_538.png)



![png](img/output_45_539.png)



![png](img/output_45_540.png)



![png](img/output_45_541.png)



![png](img/output_45_542.png)



![png](img/output_45_543.png)



![png](img/output_45_544.png)



![png](img/output_45_545.png)



![png](img/output_45_546.png)



![png](img/output_45_547.png)



![png](img/output_45_548.png)



![png](img/output_45_549.png)



![png](img/output_45_550.png)



![png](img/output_45_551.png)



![png](img/output_45_552.png)



![png](img/output_45_553.png)



![png](img/output_45_554.png)



![png](img/output_45_555.png)



![png](img/output_45_556.png)



![png](img/output_45_557.png)



![png](img/output_45_558.png)



![png](img/output_45_559.png)



![png](img/output_45_560.png)



![png](img/output_45_561.png)



![png](img/output_45_562.png)



![png](img/output_45_563.png)



![png](img/output_45_564.png)



![png](img/output_45_565.png)



![png](img/output_45_566.png)



![png](img/output_45_567.png)



![png](img/output_45_568.png)



![png](img/output_45_569.png)



![png](img/output_45_570.png)



![png](img/output_45_571.png)



![png](img/output_45_572.png)



![png](img/output_45_573.png)



![png](img/output_45_574.png)



![png](img/output_45_575.png)



![png](img/output_45_576.png)



![png](img/output_45_577.png)



![png](img/output_45_578.png)



![png](img/output_45_579.png)



![png](img/output_45_580.png)



![png](img/output_45_581.png)



![png](img/output_45_582.png)



![png](img/output_45_583.png)



![png](img/output_45_584.png)



![png](img/output_45_585.png)



![png](img/output_45_586.png)



![png](img/output_45_587.png)



![png](img/output_45_588.png)



![png](img/output_45_589.png)



![png](img/output_45_590.png)



![png](img/output_45_591.png)



![png](img/output_45_592.png)



![png](img/output_45_593.png)



![png](img/output_45_594.png)



![png](img/output_45_595.png)



![png](img/output_45_596.png)



![png](img/output_45_597.png)



![png](img/output_45_598.png)



![png](img/output_45_599.png)



![png](img/output_45_600.png)



![png](img/output_45_601.png)



![png](img/output_45_602.png)



![png](img/output_45_603.png)



![png](img/output_45_604.png)



![png](img/output_45_605.png)



![png](img/output_45_606.png)



![png](img/output_45_607.png)



![png](img/output_45_608.png)



![png](img/output_45_609.png)



![png](img/output_45_610.png)



![png](img/output_45_611.png)



![png](img/output_45_612.png)



![png](img/output_45_613.png)



![png](img/output_45_614.png)



![png](img/output_45_615.png)



![png](img/output_45_616.png)



![png](img/output_45_617.png)



![png](img/output_45_618.png)



![png](img/output_45_619.png)



![png](img/output_45_620.png)



![png](img/output_45_621.png)



![png](img/output_45_622.png)



![png](img/output_45_623.png)



![png](img/output_45_624.png)



![png](img/output_45_625.png)



![png](img/output_45_626.png)



![png](img/output_45_627.png)



![png](img/output_45_628.png)



![png](img/output_45_629.png)



![png](img/output_45_630.png)



![png](img/output_45_631.png)



![png](img/output_45_632.png)



![png](img/output_45_633.png)



![png](img/output_45_634.png)



![png](img/output_45_635.png)



![png](img/output_45_636.png)



![png](img/output_45_637.png)



![png](img/output_45_638.png)



![png](img/output_45_639.png)



![png](img/output_45_640.png)



![png](img/output_45_641.png)



![png](img/output_45_642.png)



![png](img/output_45_643.png)



![png](img/output_45_644.png)



![png](img/output_45_645.png)



![png](img/output_45_646.png)



![png](img/output_45_647.png)



![png](img/output_45_648.png)



![png](img/output_45_649.png)



![png](img/output_45_650.png)



![png](img/output_45_651.png)



![png](img/output_45_652.png)



![png](img/output_45_653.png)



![png](img/output_45_654.png)



![png](img/output_45_655.png)



![png](img/output_45_656.png)



![png](img/output_45_657.png)



![png](img/output_45_658.png)


    C:\Users\Owner\Anaconda3\lib\site-packages\numpy\lib\polynomial.py:595: RankWarning: Polyfit may be poorly conditioned
      warnings.warn(msg, RankWarning)



![png](img/output_45_660.png)



![png](img/output_45_661.png)



![png](img/output_45_662.png)



![png](img/output_45_663.png)



![png](img/output_45_664.png)



![png](img/output_45_665.png)



![png](img/output_45_666.png)



![png](img/output_45_667.png)



![png](img/output_45_668.png)



![png](img/output_45_669.png)



![png](img/output_45_670.png)



![png](img/output_45_671.png)



![png](img/output_45_672.png)



![png](img/output_45_673.png)



![png](img/output_45_674.png)



![png](img/output_45_675.png)



![png](img/output_45_676.png)



![png](img/output_45_677.png)



![png](img/output_45_678.png)



![png](img/output_45_679.png)



![png](img/output_45_680.png)



![png](img/output_45_681.png)



![png](img/output_45_682.png)



![png](img/output_45_683.png)



![png](img/output_45_684.png)



![png](img/output_45_685.png)



![png](img/output_45_686.png)



![png](img/output_45_687.png)



![png](img/output_45_688.png)



![png](img/output_45_689.png)


    C:\Users\Owner\Anaconda3\lib\site-packages\numpy\lib\polynomial.py:595: RankWarning: Polyfit may be poorly conditioned
      warnings.warn(msg, RankWarning)



![png](img/output_45_691.png)



![png](img/output_45_692.png)



![png](img/output_45_693.png)



![png](img/output_45_694.png)



![png](img/output_45_695.png)



![png](img/output_45_696.png)



![png](img/output_45_697.png)



![png](img/output_45_698.png)



![png](img/output_45_699.png)



![png](img/output_45_700.png)



![png](img/output_45_701.png)



![png](img/output_45_702.png)



![png](img/output_45_703.png)



![png](img/output_45_704.png)



![png](img/output_45_705.png)



![png](img/output_45_706.png)



![png](img/output_45_707.png)



![png](img/output_45_708.png)



![png](img/output_45_709.png)



![png](img/output_45_710.png)



![png](img/output_45_711.png)



![png](img/output_45_712.png)



![png](img/output_45_713.png)



![png](img/output_45_714.png)



![png](img/output_45_715.png)



![png](img/output_45_716.png)



![png](img/output_45_717.png)



![png](img/output_45_718.png)



![png](img/output_45_719.png)


    C:\Users\Owner\Anaconda3\lib\site-packages\numpy\lib\polynomial.py:595: RankWarning: Polyfit may be poorly conditioned
      warnings.warn(msg, RankWarning)



![png](img/output_45_721.png)



![png](img/output_45_722.png)



![png](img/output_45_723.png)



![png](img/output_45_724.png)



![png](img/output_45_725.png)



![png](img/output_45_726.png)



![png](img/output_45_727.png)



![png](img/output_45_728.png)



![png](img/output_45_729.png)


#### Question 3 
Plot from problem 4 only shows mean winning percentage vs mean payroll for teams but doesn't show how a team's payroll compares to other teams.

Plot from problem 6 not only shows mean winning percentage vs mean payroll for teams **but also shows whether the team has a greater or smaller payroll than other teams. **

 note: Plots from problem 6 may enable us to infer whether  a team may need to have a payroll greater than all other teams inorder to be more effective at paying for wins

### Expected Wins

#### Problem 7

Code below produces a scatter plot showing win percentage vs standardized payroll for all teams in one plot


```python
x = league["std_payroll"]
y = league["win_rate"]

plt.scatter(x, y )
plt.xlabel("standardized payroll")
plt.ylabel("win percentage")
plt.title("Winning percentage vs standardized payroll")

 # regression line 
fit = np.polyfit(x,y,1)
plt.plot(x, fit[0]*x + fit[1], color = 'red')


plt.show()
```


![png](img/output_49_0.png)


### Spending efficiency

#### Problem 8


```python
league["efficiency"] = league["win_rate"] - (50 + (2.5 * league["std_payroll"] ))
league.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>yearID</th>
      <th>lgID</th>
      <th>teamID</th>
      <th>franchID</th>
      <th>G</th>
      <th>W</th>
      <th>name</th>
      <th>sum(salary)</th>
      <th>win_rate</th>
      <th>category</th>
      <th>std_payroll</th>
      <th>efficiency</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>130</th>
      <td>1990</td>
      <td>AL</td>
      <td>BAL</td>
      <td>BAL</td>
      <td>161</td>
      <td>76</td>
      <td>Baltimore Orioles</td>
      <td>9680084.0</td>
      <td>47.204969</td>
      <td>1</td>
      <td>-1.959861</td>
      <td>2.104621</td>
    </tr>
    <tr>
      <th>131</th>
      <td>1990</td>
      <td>AL</td>
      <td>BOS</td>
      <td>BOS</td>
      <td>162</td>
      <td>88</td>
      <td>Boston Red Sox</td>
      <td>20558333.0</td>
      <td>54.320988</td>
      <td>1</td>
      <td>0.924213</td>
      <td>2.010454</td>
    </tr>
    <tr>
      <th>132</th>
      <td>1990</td>
      <td>AL</td>
      <td>CAL</td>
      <td>ANA</td>
      <td>162</td>
      <td>80</td>
      <td>California Angels</td>
      <td>21720000.0</td>
      <td>49.382716</td>
      <td>1</td>
      <td>1.232198</td>
      <td>-3.697779</td>
    </tr>
    <tr>
      <th>133</th>
      <td>1990</td>
      <td>AL</td>
      <td>CHA</td>
      <td>CHW</td>
      <td>162</td>
      <td>94</td>
      <td>Chicago White Sox</td>
      <td>9491500.0</td>
      <td>58.024691</td>
      <td>1</td>
      <td>-2.009859</td>
      <td>13.049338</td>
    </tr>
    <tr>
      <th>134</th>
      <td>1990</td>
      <td>AL</td>
      <td>CLE</td>
      <td>CLE</td>
      <td>162</td>
      <td>77</td>
      <td>Cleveland Indians</td>
      <td>14487000.0</td>
      <td>47.530864</td>
      <td>1</td>
      <td>-0.685437</td>
      <td>-0.755544</td>
    </tr>
  </tbody>
</table>
</div>




```python
# array for teams whose graph will be plotted
team_ids = ["OAK","BOS", "NYA", "ATL", "TBA"]

for id in team_ids:
    t = league.loc[league["teamID"] == id]
    plt.plot(t["yearID"], t["efficiency"]) 
    plt.xlabel("year")
    plt.ylabel("efficiency")
    plt.title(t["name"][1:2] + " Spending efficiency vs time(years)")
    plt.show()
```


![png](img/output_53_0.png)



![png](img/output_53_1.png)



![png](img/output_53_2.png)



![png](img/output_53_3.png)



![png](img/output_53_4.png)


#### Question 4
What can we learn from the plots above compared to plots in question 2 and question 3?
1. This plot shows how good teams are paying for wins compared to other teams accross time. 
2. Question 2 only shows how good team are at paying for wins but doesn't show how good they are accross time and compared to other teams
3. Question 2 shows how good teams are at paying for wins compared to other teams but doesn't show how good they are accross time.

note: in summary this plot shows each team's spending efficiency compared to other teams accross time while question 2 only shows a team's spending efficiency and question 3 only shows team's spending efficiency compared to other teams.


**Oakland was  most efficient at paying for wins during the money ball period (2000 - 2005)** because oakland records the highest spending efficiency during this time period
