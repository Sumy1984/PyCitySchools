# PyCitySchools

#Dependencies
import pandas as pd
import os
import numpy as np

#define path
schools_file = os.path.join("schools_complete.csv")
students_file = os.path.join("students_complete.csv")

#read csv
schools_df = pd.read_csv(schools_file)
students_df = pd.read_csv(students_file)

# Combine the data into a single dataset.  
school_data_complete = pd.merge(schools_df, students_df, how="left", on=["school_name", "school_name"])
school_data_complete.head()

## District Summary

# Calculate the total number of unique schools
unique_school_names = schools_df['school_name'].unique()
print(unique_school_names)

school_count = len(unique_school_names)
print(school_count)

# Calculate the total number of students
student_count = schools_df['size'].sum()
print (student_count)

# Calculate the total budget
total_budget = schools_df['budget'].sum()
print(total_budget)

# Calculate the average (mean) math score
average_math_score = students_df['math_score'].mean()
print(average_math_score)

# Calculate the average (mean) reading score
average_reading_score =  students_df['reading_score'].mean()
print( average_reading_score)

# calculate the percentage of students who passed math (math scores greather than or equal to 70)
passing_math_count = school_data_complete[(school_data_complete["math_score"] >= 70)].count()["student_name"]
passing_math_percentage = passing_math_count / float(student_count) * 100
passing_math_percentage

# Calculate the percentage of students who passed reading 
passing_reading_count = school_data_complete[(school_data_complete["reading_score"] >= 70)].count()["student_name"]
passing_reading_percentage = passing_reading_count / float(student_count) * 100
passing_reading_percentage

# Use the following to calculate the percentage of students that passed math and reading
passing_math_reading_count = school_data_complete[
    (school_data_complete["math_score"] >= 70) & (school_data_complete["reading_score"] >= 70)
].count()["student_name"]
overall_passing_rate = passing_math_reading_count /  float(student_count) * 100
overall_passing_rate

# Create a high-level snapshot of the district's key metrics in a DataFrame
district_summary = pd.DataFrame({
    
    "Total Schools": [school_count],
    "Total Students": [student_count],
    "Total Budget": [total_budget],
    "Average Math Score": [average_math_score],
    "Average Reading Score": [average_reading_score],
    "Passing Math%": [passing_math_percentage],
    "Passing Reading%":[passing_reading_percentage],
     "Overall Passing Rate": [overall_passing_rate]})

# Formatting
district_summary["Total Students"] = district_summary["Total Students"].map("{:,}".format)
district_summary["Total Budget"] = district_summary["Total Budget"].map("${:,.2f}".format)

# Display the DataFrame
district_summary

## School Summary 

# school type
school_types = schools_df.set_index(["school_name"])["type"]
school_types

# Calculate the total student count
by_school = school_data_complete.set_index('school_name').groupby(['school_name'])
stu_per_sch = by_school['Student ID'].count()
stu_per_sch

# school budget
sch_budget = schools_df.set_index('school_name')['budget']
sch_budget

#student budget 
stu_budget = schools_df.set_index('school_name')['budget']/schools_df.set_index('school_name')['size']
stu_budget

# Calculate the average test scores
per_school_math =  by_school['math_score'].mean()
per_school_math

# Calculate the average test scores
per_school_reading = by_school['reading_score'].mean()
per_school_reading



# Calculate the number of schools with math scores of 70 or higher

pass_math = school_data_complete[school_data_complete['math_score'] >= 70]
pass_math

# Calculate the number of schools with reading scores of 70 or higher
pass_read = school_data_complete[school_data_complete['reading_score'] >= 70]
pass_read

# Use the provided code to calculate the schools that passed both math and reading with scores of 70 or higher
passing_math_and_reading = school_data_complete[
    (school_data_complete["reading_score"] >= 70) & (school_data_complete["math_score"] >= 70)]
passing_math_and_reading

# Use the provided code to calculate the passing rates
per_school_passing_math = pass_math.groupby(["school_name"]).count()["student_name"] / stu_per_sch * 100
per_school_passing_reading = pass_read.groupby(["school_name"]).count()["student_name"] / stu_per_sch * 100
overall_passing_rate = passing_math_and_reading.groupby(["school_name"]).count()["student_name"] / stu_per_sch * 100

per_school_summary = pd.DataFrame({
    "School Type": school_types,
    "Total Students": stu_per_sch,
    "Per Student Budget": stu_budget,
    "Total Budget": sch_budget,
    "Average Math Score": per_school_math,
    "Average Reading Score": per_school_reading,
    '% Passing Math': per_school_passing_math,
    '% Passing Reading': per_school_passing_reading,
    "Overall Passing Rate": overall_passing_rate})

# Formatting
per_school_summary["Total Students"] = per_school_summary["Total Students"].map("{:,}".format)
per_school_summary["Total Budget"] = per_school_summary["Total Budget"].map("${:,.2f}".format)
per_school_summary["Per Student Budget"] = per_school_summary["Per Student Budget"].map("${:,.2f}".format)

# Display the DataFrame
per_school_summary

## Highest-Performing Schools (by % Overall Passing)

# sort values by passing rate and then only print top 5 
top_schools = per_school_summary.sort_values("Overall Passing Rate", ascending = False)
top_schools.head()

## Bottom Performing Schools (By % Overall Passing)

# Sort the schools by `% Overall Passing` in ascending order and display the top 5 rows.
bottom_schools = per_school_summary.sort_values(["Overall Passing Rate"], ascending=True)
bottom_schools.head()

## Math Scores by Grade

ninth_math = students_df.loc[students_df['grade'] == '9th'].groupby('school_name')["math_score"].mean()
tenth_math = students_df.loc[students_df['grade'] == '10th'].groupby('school_name')["math_score"].mean()
eleventh_math = students_df.loc[students_df['grade'] == '11th'].groupby('school_name')["math_score"].mean()
twelfth_math = students_df.loc[students_df['grade'] == '12th'].groupby('school_name')["math_score"].mean()

math_scores = pd.DataFrame({
        "9th": ninth_math,
        "10th": tenth_math,
        "11th": eleventh_math,
        "12th": twelfth_math
})
math_scores_by_grade= math_scores[['9th', '10th', '11th', '12th']]
math_scores_by_grade.index.name = None

#show 
math_scores_by_grade

## Reading Score by Grade

#creates grade level average reading scores for each school
ninth_reading = students_df.loc[students_df['grade'] == '9th'].groupby('school_name')["reading_score"].mean()
tenth_reading = students_df.loc[students_df['grade'] == '10th'].groupby('school_name')["reading_score"].mean()
eleventh_reading = students_df.loc[students_df['grade'] == '11th'].groupby('school_name')["reading_score"].mean()
twelfth_reading = students_df.loc[students_df['grade'] == '12th'].groupby('school_name')["reading_score"].mean()

#merges the reading score averages by school and grade together
reading_scores_by_grade = pd.DataFrame({
        "9th": ninth_reading,
        "10th": tenth_reading,
        "11th": eleventh_reading,
        "12th": twelfth_reading
})
reading_scores_by_grade = reading_scores_by_grade[['9th', '10th', '11th', '12th']]
reading_scores_by_grade.index.name = None

#show
reading_scores_by_grade


## Scores by School Spending

# Sample bins
spending_bins = [0, 585, 615, 645, 675]
group_names = ["<$585", "$585-615", "$615-645", "$645-675"]

# Create a copy of the school summary since it has the "Per Student Budget" 
school_spending_df = per_school_summary.copy()
school_spending_df["Spending Ranges (Per Student)"] = pd.cut(stu_budget, spending_bins, labels = group_names, right = False)
school_spending_df                                                                              

#  Calculate averages for the desired columns. 
spending_math_scores = school_spending_df.groupby(["Spending Ranges (Per Student)"]).mean()["Average Math Score"]
spending_reading_scores = school_spending_df.groupby(["Spending Ranges (Per Student)"]).mean()["Average Reading Score"]
spending_passing_math = school_spending_df.groupby(["Spending Ranges (Per Student)"]).mean()["% Passing Math"]
spending_passing_reading = school_spending_df.groupby(["Spending Ranges (Per Student)"]).mean()["% Passing Reading"]
overall_passing_spending = school_spending_df.groupby(["Spending Ranges (Per Student)"]).mean()["Overall Passing Rate"]

spending_summary = pd.DataFrame({
    "Average Math Score": spending_math_scores,
    "Average Reading Score": spending_reading_scores,
    '% Passing Math': spending_passing_math,
    '% Passing Reading': spending_passing_reading,
    "Overall Passing Rate": overall_passing_spending})
spending_summary

## Scores by School Size

# Establish the bins.
size_bins = [0, 1000, 2000, 5000]
labels= ["Small (<1000)", "Medium (1000-2000)", "Large (2000-5000)"]

per_school_summary["Total Students"] = per_school_summary["Total Students"].str.replace(',','').astype(int)
per_school_summary["Total Students"]

# Categorize the spending based on the bins
# Use `pd.cut` on the "Total Students" column of the `per_school_summary` DataFrame.
size_summary = per_school_summary.copy()
size_summary["School_Size"] = pd.cut(per_school_summary["Total Students"], size_bins, labels=labels, right = False)

#  Calculate averages for the desired columns. 
size_math_scores = size_summary.groupby(["School_Size"]).mean()["Average Math Score"]
size_reading_scores = size_summary.groupby(["School_Size"]).mean()["Average Reading Score"]
size_passing_math = size_summary.groupby(["School_Size"]).mean()["% Passing Math"]
size_passing_reading = size_summary.groupby(["School_Size"]).mean()["% Passing Reading"]
overall_passing_size = size_summary.groupby(["School_Size"]).mean()["Overall Passing Rate"]

size_summary = pd.DataFrame({
    "Average Math Score": size_math_scores,
    "Average Reading Score": size_reading_scores,
    '% Passing Math': size_passing_math,
    '% Passing Reading': size_passing_reading,
    "Overall Passing Rate": overall_passing_size})
size_summary

## Scores by School Type

# Create a new data frame with our desired columns
scores_type = per_school_summary[['School Type','Average Math Score',
                                  'Average Reading Score','% Passing Math',
                                  '% Passing Reading','Overall Passing Rate',]]
# Create a group based off of the school type
scores_type = scores_type.groupby('School Type').mean()
scores_type.head()
