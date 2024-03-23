# Electoral_Bond_2024-Analysis

### Project Overview :
Analysed two datasets-company purchases and  political party transactions using python 
Identified top parties and companies based on total denomination and the monthly denomination over years and visualizes these insights through tables and plots.

### Data Sources:
Company Data : contains the companies data that purchase the electoral bonds,named as "Company.csv"

Political Parties Data : contains the political parties transactions data of the electoral bonds,named as "Party.csv"

### Data Analysis

```python
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
from tabulate import tabulate
```

```python
company=pd.read_csv("Company.csv")
party=pd.read_csv("Party.csv")
```
```python
#Denomination to crores and column names changed for company data set
company['Denomination']=pd.to_numeric(company['Denomination'])
company['Denomination']=company['Denomination']/1e7
company.rename(columns={'Denomination':'Denomination(Crore)'},inplace=True)
company.head(2)
```
```python
#Denomination to crores and column names changed for company data set
party['Denomination']=pd.to_numeric(party['Denomination'])
party['Denomination']=party['Denomination']/1e7
party.rename(columns={'Date of\nEncashment':'Date','Name of the Political Party':'Political Party','Denomination':'Denomination(Crore)'},inplace=True)
party.head(2)
```
```python
# which party appears maximum number of times 
party['Political Party'].value_counts()
```
```python
# Which company purchases maximum
company['Purchaser Name'].value_counts()
```

```python
# New column created in both data sets that stores the year 
party['Year']=pd.to_datetime(party['Date']).dt.year
company['Year']=pd.to_datetime(company['Date of Purchase']).dt.year
```
```python
# to calculate in which year parties transactions more
party['Year']=pd.to_numeric(party['Year'])
party['Year'].value_counts()
```
```python
# to calculate in which year company purchase most
company['Year']=pd.to_numeric(company['Year'])
company['Year'].value_counts()
```

```python
# in each year count which party transacts how many times
party_counts = party.groupby(['Year', 'Political Party']).size()
print(party_counts)
```
```python
# in each year count which company purchase how many time
company_counts = company.groupby(['Year', 'Purchaser Name']).size()
print(company_counts)
```
```python
#calculates the total denomination for that party for that year
party_counts = party.groupby(['Year', 'Political Party'])['Denomination(Crore)'].sum().reset_index(name='Total Denomination')
print(party_counts)
```
```python
#represents above data in tabular form considering the top 5 parties for each year having maximum denomination .
party_top5_per_year = party_counts.groupby('Year').apply(lambda x: x.nlargest(5, 'Total Denomination')).reset_index(drop=True)

for year, group in party_top5_per_year.groupby('Year'):
    print(f"\n{int(year)}:")
    print(tabulate(group.drop(columns='Year'), headers='keys', tablefmt='grid', showindex=False))
```
```python

colors = [(0.1, 0.1, 0.9, 0.5), (0.1, 0.9, 0.1, 0.5), (0.9, 0.1, 0.1, 0.5), (0.1, 0.9, 0.9, 0.5), (0.9, 0.1, 0.9, 0.5)]

# Plotting of top 5 parties for each year having maximum denomination 
num_years = len(party_top5_per_year['Year'].unique())
num_rows = (num_years + 1) // 2  
fig, axes = plt.subplots(nrows=num_rows, ncols=2, figsize=(10, 5 * num_rows))

for i, (year, group) in enumerate(party_top5_per_year.groupby('Year')):
    row_index = i // 2
    col_index = i % 2
    ax = axes[row_index, col_index]
    
    bars = ax.bar(group['Political Party'], group['Total Denomination'], color=colors[:len(group)])

    ax.set_title(f"Top 5 Parties with Highest Total Denomination in {int(year)}")
    ax.set_xlabel('')  
    ax.set_ylabel('Total Denomination')
    ax.tick_params(axis='x', which='both', bottom=False, top=False, labelbottom=False) 

    legend = ax.legend(bars, group['Political Party'], loc='upper right', fontsize='xx-small') 

    legend.set_title('Political Party') 

plt.tight_layout()
plt.show()
```


![top5_politicalparty](https://github.com/bubblepreetkaur06/Electoral_Bond_2024-Analysis/assets/164672202/5f2f091f-803e-4250-88fb-c1375fdf2b3d)



```python
#calculates the total denomination for that company for that year
company_counts = company.groupby(['Year', 'Purchaser Name'])['Denomination(Crore)'].sum().reset_index(name='Total Denomination')
print(company_counts)
```



```python
company_top5_per_year = company_counts.groupby('Year').apply(lambda x: x.nlargest(5, 'Total Denomination')).reset_index(drop=True)
# Display in tabular form
for year, group in company_top5_per_year.groupby('Year'):
    print(f"\n{int(year)}:")
    print(tabulate(group.drop(columns='Year'), headers='keys', tablefmt='grid', showindex=False))
```




```python
#top 5 companies with highest total denomination per year
num_years = len(company_top5_per_year['Year'].unique())

# Create subplots
fig, axes = plt.subplots(num_years, 1, figsize=(5, 4*num_years))

for i, (year, group) in enumerate(company_top5_per_year.groupby('Year')):
    
    wedges, texts, autotexts = axes[i].pie(group['Total Denomination'], autopct='%1.1f%%', startangle=140)
    axes[i].set_title(f'Top 5 Companies with Highest Total Denomination for Year {int(year)}')
    axes[i].axis('equal')  

    axes[i].legend(wedges, group['Purchaser Name'], title='Companies', loc='center left', bbox_to_anchor=(1,0.5))

plt.show()
```

![top5_company](https://github.com/bubblepreetkaur06/Electoral_Bond_2024-Analysis/assets/164672202/40382218-5257-4e88-a082-8fd2bf757991)



```python
#calculates each month total denomination in each year the company data set
company['Date of Purchase'] = pd.to_datetime(company['Date of Purchase'])
company['Year'] = company['Date of Purchase'].dt.year
company['Month'] = company['Date of Purchase'].dt.month

month_names = {
    1: 'January', 2: 'February', 3: 'March', 4: 'April', 5: 'May', 6: 'June',
    7: 'July', 8: 'August', 9: 'September', 10: 'October', 11: 'November', 12: 'December'
}

company['Month'] = company['Month'].map(month_names)
company['Month'] = pd.Categorical(company['Month'], categories=month_names.values(), ordered=True)
company_monthly_denomination = company.groupby(['Year', 'Month'])['Denomination(Crore)'].sum().reset_index()
monthly_denomination_pivot = company_monthly_denomination.pivot(index='Month', columns='Year', values='Denomination(Crore)').fillna(0)

monthly_denomination_pivot = monthly_denomination_pivot.loc[(monthly_denomination_pivot != 0).any(axis=1)]

print("\t\tCompanies Monthly per year \n")
print(tabulate(monthly_denomination_pivot, headers='keys', tablefmt='pretty'))
```





```python

num_years = len(company_monthly_denomination['Year'].unique())
num_months = len(month_names)

bar_width = 1.0 / num_years  # evenly width divided among the years

years = sorted(company_monthly_denomination['Year'].unique())

plt.figure(figsize=(12, 8))
ax = plt.gca()

colors = plt.cm.tab10(np.linspace(0, 1, num_years))

for i, year in enumerate(years):
    data_year = company_monthly_denomination[company_monthly_denomination['Year'] == year]
    positions = np.arange(num_months) + i * bar_width  
    plt.bar(positions, data_year['Denomination(Crore)'], width=bar_width, label=str(year), color=colors[i])

valid_months = company_monthly_denomination['Month'].unique()
plt.xticks(np.arange(len(valid_months)) + bar_width * (num_years - 1) / 2, valid_months, rotation=90)

plt.title('Monthly Denomination Over the Years \n', fontsize=16)
plt.xlabel('Month', fontsize=14)
plt.ylabel('Denomination (Crore)', fontsize=14)
plt.legend(title='Year', fontsize=12)

plt.grid(axis='y',linestyle='--', alpha=0.5)
plt.tight_layout()
plt.show()
```

![monthly_demo_yearly](https://github.com/bubblepreetkaur06/Electoral_Bond_2024-Analysis/assets/164672202/cd9105a9-8d88-429e-8568-54237be513d9)


