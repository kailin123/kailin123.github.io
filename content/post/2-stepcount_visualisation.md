---
title: "Visualising Step Count"
date: 2025-01-01T12:01:23+08:00
lastmod: 2025-01-01T12:01:23+08:00
draft: false
categories: ["visualisation", "python","plotly"]

menu:
  main:
    parent: "archives"
    weight: 1
---

I wanted to get a view of my daily step count data across years. Currently, the Apple Health app only offers aggregated views when looking across longer time periods. 

Fortunately, there is an option to export out the data.

{{< blockquote >}}
  I used Python to parse and read in the .xml file, then used plotly-calplot package to generate a heatmap.
{{< /blockquote >}}

### Heatmap
<b>The below chart is best viewed on desktop. </b>
&nbsp;

<span style="color:blue;">Hover mouse on chart to see data for specific dates</span>

{{< include-html "static/images/stepcount_visualisation.html">}}

&nbsp;

This enables me to easily see my step count for any specific date between 2022 - 2024. 

<i>The trends align closely with key moments and activities in my life:
- Covid-19 and work from home practices led to a much lower step count in 2022
- I now know that my peak daily step count occured when I climbed Hallasan on Jeju Island in Sept 2023
- Other trends are mostly related to work from home schedules, hiking trips, travelling overseas... </i>

### Average daily step count by years
For a clearer measure of which year saw the most activity, I also calculated the average daily step count for each year.

<table border="2" class="dataframe">
  <thead>
    <tr>
      <th>year</th>
      <th style="text-align: right;">average daily step count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>2022</td>
      <td style="text-align: right;">3,354</td>
    </tr>
    <tr>
      <td>2023</td>
      <td style="text-align: right;">6,412</td>
    </tr>
    <tr>
      <td>2024</td>
      <td style="text-align: right;">6,208</td>
    </tr>
  </tbody>
</table>

So it seems my steps activity has fallen slightly in 2024. Time to catch up in 2025?

### Codes

```python
# import libraries
import os
import xml.etree.ElementTree as ET
import pandas as pd
import datetime as dt
import plotly
from plotly_calplot import calplot
```

```python
# create element tree object
tree = ET.parse('export.xml')
# for every health record, extract the attributes
root = tree.getroot()
record_list = [x.attrib for x in root.iter('Record')]

# create dataframe
df = pd.DataFrame(record_list)
```

```python
# convert to datetime
for col in ['creationDate', 'startDate', 'endDate']:
    df[col] = pd.to_datetime(df[col])

# convert value to numeric
df['value'] = pd.to_numeric(df['value'], errors='coerce')

# make observation names shorter
df['type'] = df['type'].str.replace('HKQuantityTypeIdentifier', '')
df['type'] = df['type'].str.replace('HKCategoryTypeIdentifier', '')

# filter out relevant data
stepcount = df[(df['type'] == 'StepCount') & (df['sourceName'] == 'Kailin') & (df['startDate'] >= '2022-01-01')
              & (df['startDate'] <= '2025-01-01')]

# final dataframe for use
stepcount = stepcount.groupby([stepcount['startDate'].dt.date])['value'].sum().reset_index()
```

```python
# plot heatmap
fig = calplot(stepcount, x="startDate", y="value",colorscale = 'reds', years_title=True, space_between_plots=0.1
             , name = 'Daily Step Count', showscale = True)
fig.update_layout(width=800, height=600)
fig.show()

plotly.offline.plot(fig, filename='result.html')
```

```python
# calculate average stepcount
stepcount['year'] = stepcount['startDate'].dt.year
yearly_avg = stepcount.groupby('year')['value'].mean().reset_index()
print(round(yearly_avg,2))
```