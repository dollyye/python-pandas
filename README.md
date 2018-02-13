
# PyCity Schools Analysis

Observed Trend 1: Higher Spending per student does not lead necessarily lead to higher scores. In fact, we see that the schools in the lower spending ranges, have higher average math and reading scores as well as higher pass rates.

Observed Trend 2: Smaller schools in size have higher average math and reading scores and higher math and reading pass rates while larger schools have lower scores and lower pass rates.

Observed Trend 3: Charter schools have higher average math and reading scores and have higher math and reading pass rates than District schools.


```python
import pandas as pd
import numpy as np
```


```python
schools_df = pd.read_csv("schools_complete.csv")
students_df = pd.read_csv("students_complete.csv")
```

# District Summary


```python
# Count number of schools
total_schools = schools_df["name"].count()

# Count number of students
total_students = students_df["name"].count()

# Sum for total budget
total_budget = schools_df["budget"].sum()

# Average for Math score
avg_math_score = students_df["math_score"].mean()

# Average for Reading score
avg_reading_score = students_df["reading_score"].mean()

# Find location where Math score is "Pass", assuming passing is score of 65 and up
pass_math_df = students_df.loc[students_df["math_score"]>=65,["school","name","Student ID"]]
pass_math_df["pass_math"] = "Pass"
count_pass_math = pass_math_df["name"].count()
percent_pass_math = count_pass_math / total_students

# Find location where Reading score is "Pass", assuming passing is score of 65 and up
pass_reading_df = students_df.loc[students_df["reading_score"]>=65,["school","name","Student ID"]]
pass_reading_df["pass_reading"] = "Pass"
count_pass_reading = pass_reading_df["name"].count()
percent_pass_reading = count_pass_reading / total_students

# Overall Pass Rate is average of Math Pass Rate and Reading Pass Rate
avg_pass_rate = (percent_pass_math + percent_pass_reading) / 2

# Create dataframe for district summary table
district_summary = pd.DataFrame({"Total Schools":[total_schools],
                                       "Total Students":[total_students],
                                       "Total Budget":[total_budget],
                                       "Average Math Score":[avg_math_score],
                                       "Average Reading Score":[avg_reading_score],
                                       "% Passing Math":[percent_pass_math],
                                       "% Passing Reading":[percent_pass_reading],
                                       "Overall Passing Rate":[avg_pass_rate]})

# Format values
district_summary["Total Students"] = district_summary["Total Students"].map("{:,}".format)
district_summary["Total Budget"] = district_summary["Total Budget"].map("${:,.2f}".format)
district_summary["Average Math Score"] = district_summary["Average Math Score"].map("{:,.2f}".format)
district_summary["Average Reading Score"] = district_summary["Average Reading Score"].map("{:,.2f}".format)
district_summary["Overall Passing Rate"] = (district_summary["Overall Passing Rate"]*100).map("{:,.2f}%".format)

# Arrange column names in order of table display
col_arrange = ["Total Schools","Total Students","Total Budget","Average Math Score","Average Reading Score","Overall Passing Rate"]
district_summary = district_summary[col_arrange]

district_summary
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
      <th>Total Schools</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15</td>
      <td>39,170</td>
      <td>$24,649,428.00</td>
      <td>78.99</td>
      <td>81.88</td>
      <td>90.46%</td>
    </tr>
  </tbody>
</table>
</div>



# School Summary


```python
# Merge students_df with pass_math_df and pass_reading_df
students2_df = pd.merge(pd.merge(students_df, pass_math_df, how="left", on=["school","name","Student ID"]),
                        pass_reading_df, how="left", on=["school","name","Student ID"])

# Merge new students datafram with schools_df
merge_table = pd.merge(students2_df,schools_df, how="left", left_on="school", right_on="name")

# Rename column to student_name and delete name_y
merge_table = merge_table.rename(columns={"name_x":"student_name"})
del merge_table["name_y"]
```


```python
# Group by School Name and School Type
groupby_table = merge_table.groupby(["school","type"],as_index=False)

# Count Student Names and number Passing Math and Reading
schools_summ_count = groupby_table[["student_name","pass_math","pass_reading"]].count()

# Calculate for % Passing Math, Reading, and Overall Passing Rate
schools_summ_count["% Passing Math"] = ((schools_summ_count["pass_math"])/(schools_summ_count["student_name"]))*100
schools_summ_count["% Passing Reading"] = ((schools_summ_count["pass_reading"])/(schools_summ_count["student_name"]))*100
schools_summ_count["% Overall Passing Rate"] = ((schools_summ_count["% Passing Math"])+(schools_summ_count["% Passing Reading"]))/2

# Create array of which columns to display
col_i = ["school","type","student_name","% Passing Math","% Passing Reading","% Overall Passing Rate"]
schools_summ_count = schools_summ_count[col_i]

# Rename columns 
schools_summ_count.rename(columns={"school":"School Name",
                                   "type":"School Type",
                                   "student_name":"Total Students"},inplace=True)                                                    
```


```python
# Use mean to calculate Total Budget, Average Math Score, Average Reading Score
schools_summ_mean = groupby_table[["budget","math_score","reading_score"]].mean()
schools_summ_mean["Per Student Budget"] = schools_summ_mean["budget"]/schools_summ_count["Total Students"]

# Rename columns
schools_summ_mean.rename(columns={"school":"School Name",
                                  "type":"School Type",
                                  "budget":"Total School Budget", 
                                  "math_score":"Average Math Score",
                                  "reading_score":"Average Reading Score"},inplace=True)
```


```python
# Merge Count and Mean tables on School Name and School Type
schools_summary = pd.merge(schools_summ_count,schools_summ_mean,how="left",on=["School Name","School Type"])

# Create Array of column names to display
summ_col = ["School Name","School Type","Total Students","Total School Budget","Per Student Budget",
           "Average Math Score","Average Reading Score","% Passing Math","% Passing Reading","% Overall Passing Rate"]
schools_summary = schools_summary[summ_col]

# Set School Name as index and index name=None
schools_summary.set_index("School Name", drop=False, inplace=True)
schools_summary.index.name = None

# Format column names 
schools_summary["Total Students"] = schools_summary["Total Students"].map("{:,}".format)
schools_summary["Total School Budget"] = schools_summary["Total School Budget"].map("${:,.2f}".format)
#schools_summary['Per Student Budget'] = schools_summary['Per Student Budget'].map('${:,.2f}'.format)
#schools_summary['Average Math Score'] = schools_summary['Average Math Score'].map('{:,.2f}'.format)
#schools_summary['Average Reading Score'] = schools_summary['Average Reading Score'].map('{:,.2f}'.format)
#schools_summary['% Passing Math'] = (schools_summary['% Passing Math']).map('{:,.2f}%'.format)
#schools_summary['% Passing Reading'] = (schools_summary['% Passing Reading']).map('{:,.2f}%'.format)
#schools_summary['% Overall Passing Rate'] = (schools_summary['% Overall Passing Rate']).map('{:,.2f}%'.format)

# Drop School Name for cleaner look
schools_summary_clean = schools_summary.drop('School Name', axis=1)
schools_summary_clean
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
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>District</td>
      <td>4,976</td>
      <td>$3,124,928.00</td>
      <td>628.0</td>
      <td>77.048432</td>
      <td>81.033963</td>
      <td>77.913987</td>
      <td>94.553859</td>
      <td>86.233923</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1,858</td>
      <td>$1,081,356.00</td>
      <td>582.0</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>100.000000</td>
      <td>100.000000</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2,949</td>
      <td>$1,884,411.00</td>
      <td>639.0</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>77.178705</td>
      <td>94.540522</td>
      <td>85.859613</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2,739</td>
      <td>$1,763,916.00</td>
      <td>644.0</td>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>78.203724</td>
      <td>93.866375</td>
      <td>86.035049</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1,468</td>
      <td>$917,500.00</td>
      <td>625.0</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>100.000000</td>
      <td>100.000000</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4,635</td>
      <td>$3,022,020.00</td>
      <td>652.0</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>77.734628</td>
      <td>94.606257</td>
      <td>86.170442</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>Charter</td>
      <td>427</td>
      <td>$248,087.00</td>
      <td>581.0</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>100.000000</td>
      <td>100.000000</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2,917</td>
      <td>$1,910,635.00</td>
      <td>655.0</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>77.716832</td>
      <td>94.480631</td>
      <td>86.098732</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4,761</td>
      <td>$3,094,650.00</td>
      <td>650.0</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>77.966814</td>
      <td>94.475950</td>
      <td>86.221382</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858.00</td>
      <td>609.0</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>100.000000</td>
      <td>100.000000</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3,999</td>
      <td>$2,547,363.00</td>
      <td>637.0</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>77.944486</td>
      <td>94.623656</td>
      <td>86.284071</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>Charter</td>
      <td>1,761</td>
      <td>$1,056,600.00</td>
      <td>600.0</td>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>100.000000</td>
      <td>100.000000</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1,635</td>
      <td>$1,043,130.00</td>
      <td>638.0</td>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>100.000000</td>
      <td>100.000000</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2,283</td>
      <td>$1,319,574.00</td>
      <td>578.0</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>100.000000</td>
      <td>100.000000</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>Charter</td>
      <td>1,800</td>
      <td>$1,049,400.00</td>
      <td>583.0</td>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>100.000000</td>
      <td>100.000000</td>
      <td>100.000000</td>
    </tr>
  </tbody>
</table>
</div>



# Top Performing Schools (By Passing Rate)


```python
# Because there are multiple schools with Overall Passing Rate of 100%, I use a combined average of math and reading scores as the second sort
schools_summary["combined_average"] = (schools_summary["Average Math Score"]+schools_summary["Average Reading Score"])/2
# Sort schools by Overall Passing Rate and Combined Average score
sort_schools = schools_summary.sort_values(["% Overall Passing Rate","combined_average"], ascending=False)
# Find the 5 top performing schools
top_schools = sort_schools.nlargest(5,"% Overall Passing Rate",keep='first')
# Drop columns not needed in final output
top_schools = top_schools.drop(["combined_average","School Name"],axis=1)
top_schools
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
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858.00</td>
      <td>609.0</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>Charter</td>
      <td>1,800</td>
      <td>$1,049,400.00</td>
      <td>583.0</td>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>Charter</td>
      <td>427</td>
      <td>$248,087.00</td>
      <td>581.0</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1,635</td>
      <td>$1,043,130.00</td>
      <td>638.0</td>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2,283</td>
      <td>$1,319,574.00</td>
      <td>578.0</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
  </tbody>
</table>
</div>



# Bottom Performing Schools (By Passing Rate)


```python
# Find the 5 bottom performing schools by Overall Passing Rate
bottom_schools = sort_schools.nsmallest(5,"% Overall Passing Rate",keep="last")
# Drop columns not needed in final output
bottom_schools = bottom_schools.drop(["combined_average","School Name"],axis=1)
bottom_schools
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
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2,949</td>
      <td>$1,884,411.00</td>
      <td>639.0</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>77.178705</td>
      <td>94.540522</td>
      <td>85.859613</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2,739</td>
      <td>$1,763,916.00</td>
      <td>644.0</td>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>78.203724</td>
      <td>93.866375</td>
      <td>86.035049</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2,917</td>
      <td>$1,910,635.00</td>
      <td>655.0</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>77.716832</td>
      <td>94.480631</td>
      <td>86.098732</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4,635</td>
      <td>$3,022,020.00</td>
      <td>652.0</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>77.734628</td>
      <td>94.606257</td>
      <td>86.170442</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4,761</td>
      <td>$3,094,650.00</td>
      <td>650.0</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>77.966814</td>
      <td>94.475950</td>
      <td>86.221382</td>
    </tr>
  </tbody>
</table>
</div>



# Math Scores by Grade


```python
# Create pivot_table for average math_schore by school and grade
math_byGrade = students_df.pivot_table(values='math_score',index='school',columns=['grade'],aggfunc=np.mean)
# Do not show School Name index name
math_byGrade.index.name = None
# Create Array of column names to display
column_order = ['9th', '10th', '11th','12th']
math_byGrade = math_byGrade.reindex_axis(column_order, axis=1)
math_byGrade
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
      <th>grade</th>
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>77.083676</td>
      <td>76.996772</td>
      <td>77.515588</td>
      <td>76.492218</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.094697</td>
      <td>83.154506</td>
      <td>82.765560</td>
      <td>83.277487</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>76.403037</td>
      <td>76.539974</td>
      <td>76.884344</td>
      <td>77.151369</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.361345</td>
      <td>77.672316</td>
      <td>76.918058</td>
      <td>76.179963</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>82.044010</td>
      <td>84.229064</td>
      <td>83.842105</td>
      <td>83.356164</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>77.438495</td>
      <td>77.337408</td>
      <td>77.136029</td>
      <td>77.186567</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.787402</td>
      <td>83.429825</td>
      <td>85.000000</td>
      <td>82.855422</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>77.027251</td>
      <td>75.908735</td>
      <td>76.446602</td>
      <td>77.225641</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>77.187857</td>
      <td>76.691117</td>
      <td>77.491653</td>
      <td>76.863248</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.625455</td>
      <td>83.372000</td>
      <td>84.328125</td>
      <td>84.121547</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>76.859966</td>
      <td>76.612500</td>
      <td>76.395626</td>
      <td>77.690748</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>83.420755</td>
      <td>82.917411</td>
      <td>83.383495</td>
      <td>83.778976</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.590022</td>
      <td>83.087886</td>
      <td>83.498795</td>
      <td>83.497041</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.085578</td>
      <td>83.724422</td>
      <td>83.195326</td>
      <td>83.035794</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.264706</td>
      <td>84.010288</td>
      <td>83.836782</td>
      <td>83.644986</td>
    </tr>
  </tbody>
</table>
</div>



# Reading Scores by Grade


```python
# Create pivot_table for average reading_score by school and grade
reading_byGrade = students_df.pivot_table(values='reading_score',index='school',columns=['grade'],aggfunc=np.mean)
# Do not show School Name index name
reading_byGrade.index.name = None
# Create Array of column names to display
column_order = ['9th', '10th', '11th','12th']
reading_byGrade = reading_byGrade[column_order]
reading_byGrade
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
      <th>grade</th>
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>81.303155</td>
      <td>80.907183</td>
      <td>80.945643</td>
      <td>80.912451</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.676136</td>
      <td>84.253219</td>
      <td>83.788382</td>
      <td>84.287958</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>81.198598</td>
      <td>81.408912</td>
      <td>80.640339</td>
      <td>81.384863</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>80.632653</td>
      <td>81.262712</td>
      <td>80.403642</td>
      <td>80.662338</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.369193</td>
      <td>83.706897</td>
      <td>84.288089</td>
      <td>84.013699</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>80.866860</td>
      <td>80.660147</td>
      <td>81.396140</td>
      <td>80.857143</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.677165</td>
      <td>83.324561</td>
      <td>83.815534</td>
      <td>84.698795</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>81.290284</td>
      <td>81.512386</td>
      <td>81.417476</td>
      <td>80.305983</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>81.260714</td>
      <td>80.773431</td>
      <td>80.616027</td>
      <td>81.227564</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.807273</td>
      <td>83.612000</td>
      <td>84.335938</td>
      <td>84.591160</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>80.993127</td>
      <td>80.629808</td>
      <td>80.864811</td>
      <td>80.376426</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>84.122642</td>
      <td>83.441964</td>
      <td>84.373786</td>
      <td>82.781671</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.728850</td>
      <td>84.254157</td>
      <td>83.585542</td>
      <td>83.831361</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.939778</td>
      <td>84.021452</td>
      <td>83.764608</td>
      <td>84.317673</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.833333</td>
      <td>83.812757</td>
      <td>84.156322</td>
      <td>84.073171</td>
    </tr>
  </tbody>
</table>
</div>



# Scores by School Spending


```python
# Bins for School Spending
bins = [0, 585, 615, 645, 675]
group_names = ['<$585', '$585-615', '$615-645', '$645-675']
schools_summary['Spending Ranges (Per Student)'] = pd.cut(schools_summary['Per Student Budget'], 
                                                          bins, labels=group_names)
spend_range_df = pd.merge(merge_table, schools_summary, how='left', left_on='school', right_on='School Name')

# Calculate mean of values by Spending Ranges
spend_range_mean = spend_range_df.groupby('Spending Ranges (Per Student)').mean()

# Create Array of column names to display
col_spend_summ = ['Average Math Score','Average Reading Score',
                  '% Passing Math','% Passing Reading','% Overall Passing Rate']
spend_range_summary = spend_range_mean[col_spend_summ]
spend_range_summary
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
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>Spending Ranges (Per Student)</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt;$585</th>
      <td>83.363065</td>
      <td>83.964039</td>
      <td>100.000000</td>
      <td>100.000000</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>$585-615</th>
      <td>83.529196</td>
      <td>83.838414</td>
      <td>100.000000</td>
      <td>100.000000</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>$615-645</th>
      <td>78.061635</td>
      <td>81.434088</td>
      <td>81.701002</td>
      <td>95.412586</td>
      <td>88.556794</td>
    </tr>
    <tr>
      <th>$645-675</th>
      <td>77.049297</td>
      <td>81.005604</td>
      <td>77.820190</td>
      <td>94.526111</td>
      <td>86.173150</td>
    </tr>
  </tbody>
</table>
</div>



# Scores by School Size


```python
# Bins for School Size 
bins = [0, 1000, 3000, 5000]
group_names = ['Small (<1000)', 'Medium (1000-3000)', 'Large (3000-5000)']
merge_table['School Size'] = pd.cut(merge_table['size'], bins, labels=group_names)
school_size_df = pd.merge(merge_table, schools_summary, how='left', left_on='school', right_on='School Name')

# Calculate mean of values by School Size
school_size_mean = school_size_df.groupby('School Size').mean()

# Create Array of column names to display
col_size_summ = ['Average Math Score','Average Reading Score',
                  '% Passing Math','% Passing Reading','% Overall Passing Rate']
school_size_summary = school_size_mean[col_size_summ]
school_size_summary
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
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Size</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Small (&lt;1000)</th>
      <td>83.828654</td>
      <td>83.974082</td>
      <td>100.000000</td>
      <td>100.000000</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>Medium (1000-3000)</th>
      <td>80.450902</td>
      <td>82.626481</td>
      <td>90.108192</td>
      <td>97.475528</td>
      <td>93.791860</td>
    </tr>
    <tr>
      <th>Large (3000-5000)</th>
      <td>77.070764</td>
      <td>80.928365</td>
      <td>77.889064</td>
      <td>94.562082</td>
      <td>86.225573</td>
    </tr>
  </tbody>
</table>
</div>



# Scores by School Type


```python
# Create Array of column names to display
col_type_summ = ['School Type','Average Math Score','Average Reading Score',
                '% Passing Math','% Passing Reading','% Overall Passing Rate']
school_type_summ = schools_summary[col_type_summ]

# Group by School Type and find mean of values
school_type_summ = school_type_summ.groupby('School Type').mean()
school_type_summ
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
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Type</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Charter</th>
      <td>83.473852</td>
      <td>83.896421</td>
      <td>100.000000</td>
      <td>100.000000</td>
      <td>100.00000</td>
    </tr>
    <tr>
      <th>District</th>
      <td>76.956733</td>
      <td>80.966636</td>
      <td>77.808454</td>
      <td>94.449607</td>
      <td>86.12903</td>
    </tr>
  </tbody>
</table>
</div>


