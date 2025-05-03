
# Data-Science-Portfolio-Final
Background
This data analysis project is for my Data Science Senior Portfolio. This is the analysis of the data colllected from an IRB approved study with the Department of Psychological Sciences at Belmont University. In this study, data was collected on students of African descent around the country and their mental health status, percieved social support, and technology usage for mental health services. The students werre also asked to identify what cultures they identified with the most and their responses for each of the categories were compared for black immigrants and African-Americans.

# Goals
- Conduct an IRB approved study
- Perform comparative analyses of the minority groups of African descent
- Create visualizations of the data collected
- Apply advanced data science skills
# Part I: Collecting and Cleaning Data
Data was collected from the online survey platform Qualtrics. The survey consisted of demographic questions, and four scientifically verified psychological questionnaires: Patient Health Questionnaire (PHQ-9), Multidimensional Scale of Perceived Social Support (MSPSS), Coping Inventory for Stressful Situations (CISS-21), and Unified Theory of Acceptance and Use of Technology (UTAUT-2). The data was collected by sending the survey to black oriented organizations across different university campuses such as the Black Student Association and the African Student Organization at Belmont University. Thirty participants had taken the survey at the time of analysis. The data was downloaded as a CSV file. The data was cleaned to only keep the columns that contained the demographic question answers and the survey question answers. NA responses for any question were also dropped. 

# Part II: Questionnaire Scoring
Data analysis was conducted in Python using Google Colab.

Importing Packages and Loading Data
Below are the packages used for this project:
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from statistics import mode
from scipy.stats import norm as norm
!pip install openpyxl
from google.colab import files

Number of Rows and Unique Members
How many rows are in the dataset? How many unique members make up these rows?
````
num_rows = len(df) #number of rows in dataset
print(num_rows)
````
There were 30 participants who sent responses for the survey.

Distribution of ethnicities:
First, te counts of people who identified as Africa, African-American, Carribean, or other were counted
````
counts_AA = df['Q5'].value_counts().get('African-American', 0)
print("Occurrences of 'African-American':", counts_AA)
counts_A = df['Q5'].value_counts().get('African', 0)
print("Occurrences of 'African':", counts_A)
counts_C = df['Q5'].value_counts().get('Carribbean', 0)
print("Occurrences of 'African':", counts_C)
````
There were 18 African-American participants, 8 African participants, and 3 Carribean participants, and one identified as other which they specified they identified as Afro-Cuban.

## Scoring the PHQ-9
The PHQ-9 is a depression scoring questionnaire that uses 9 questions on a 4-point likert scale to evaluate participants. The participants would answer the questions on how accurately the statements reflected them. The data was recorded as the string responses and they had to recoded into numeric values for scoring. 

First I created a seperate dataset just for the PHQ-9 Questions:
````
phq9_columns = []
for i in range(len(columns)):
  if columns[i].find("Q11") >= 0 :
    phq9_columns.append(columns[i])
phq9_columns
````
Then I recoded the values:
````
phq9_df = df[phq9_columns]
phq9_df
phq9_mapping = {
    "Not at all" : 0,
    "A few days" : 1,
    "More than half of the days" : 2,
    "Nearly every day" : 3
}
phq9_df = phq9_df.replace(phq9_mapping)
````
Then they were scored by summing up the total points for each participant:
````
phq9_score = phq9_df.sum(axis=1)
df['PHQ9 Score']= phq9_score
df.head()
````
## Scoring the CISS
The CISS questionnaire was a series of statements separated into three categories. The participants were asked to select the statements that they agree with. The amount of statements selected for each category is totaled and the category with the highest total is the participant's dominant coping style.

A separate dataframe was created for the CISS questions:
````
ciss_columns = []
for i in range(len(columns)):
  if columns[i].find("Q12") >= 0 :
    ciss_columns.append(columns[i])
````
Each question was coded for each category and coded into String arrays:  
````
ciss_question_list= ['Take some time off and get away from the situation',...,'Phone a friend']
ciss_task_oriented= ['Focus on the problem and see how I can solve it',..., 'Analyze my problem before reacting']
ciss_emotion_oriented=['Blame myself for ha\u200eving gotten into this situation',..., 'Focus on my general inadequacies']
ciss_avoidance_oriented= ['Take some time off and get away from the situation',..., 'Phone a friend']
for index, row in ciss_df.iterrows():
    for col in ciss_df.columns:
        value = row[col]
        ciss_df.loc[index, col] = [x.strip() for x in str(value).split(',')]
````
Then the subtotals for each category were added up and made into individual scores columns which were added to the main dataframe:
````
ciss_task_score = []
ciss_emotion_score = []
ciss_avoidance_score = []
for participant in ciss_df['Q12']:
  task_score = 0
  emotion_score = 0
  avoidance_score = 0
  for response in participant:
      if response in ciss_task_oriented:
            task_score += 1
      elif response in ciss_emotion_oriented:
            emotion_score += 1
      elif response in ciss_avoidance_oriented:
            avoidance_score += 1
      else:
            print("error:", response)
  ciss_task_score.append(task_score)
  ciss_emotion_score.append(emotion_score)
  ciss_avoidance_score.append(avoidance_score)
````
````
df['Task Oriented Score'] = ciss_task_score
df['Emotion Oriented Score'] = ciss_emotion_score
df['Avoidance Oriented Score'] = ciss_avoidance_score
````
Then the overall coping style was determined for each participant by analyzing their highest score:
````
overall_style = []
for i in range(len(df)):
  individual_style = []
  task_score = df['Task Oriented Score'].iloc[i]
  emotion_score = df['Emotion Oriented Score'].iloc[i]
  avoidance_score = df['Avoidance Oriented Score'].iloc[i]
  max_score = max(task_score, emotion_score, avoidance_score)
  if max_score == task_score:
    individual_style.append("Task Oriented Coping")
  if max_score == emotion_score:
    individual_style.append("Emotion Oriented Coping")
  if max_score == avoidance_score:
    individual_style.append("Avoidance Oriented Coping")
  overall_style.append(individual_style)

df["Overall Coping Style"] = overall_style
````
## Scoring the MSPSS
The MSPSS is a questionnaire about social support that asks participants to rate the accuracy of statements to them on a seven point Likert scale. Agreeing with the statements indicates that they percieve themself to have high social support in that area.

A separate dataframe was created for the MSPSS questions and the responses to each statement were mapped to a Likert scale:
````
mspss_columns = []
for i in range(len(columns)):
  if columns[i].find("Q13") >= 0 :
    mspss_columns.append(columns[i])
mspss_df = df[mspss_columns]

mspss_mapping = {
    "Very Strongly Disagree" : 1,
    "Strongly Disagree" : 2,
    "Mildly disagree" : 3,
    "Neutral" : 4,
    "Mildly Agree": 5,
    "Strongly Agree" : 6,
    "Very Strongly Agree" : 7
}
mspss_df = mspss_df.replace(mspss_mapping)
````
The total MSPSS score was calulated by adding up all the question response scores and theses scores were added to the main dataframe:
````
total_mspss = mspss_df.sum(axis=1)
df['Total MSPSS Score']= total_mspss
````
## Scoring the UTAUT-2
The UTAUT is a questionnaire that asks participants to rate statements about their use of technology for a specific goal on a five point Likert scale. For this study, we modified the UTAUT to ask about mental health technology specifically. Agreeing with the statements indicates that the participant utilizes technology for that specific goal.

A separate dataframe was created for the UTAUT questions and the responses to each statement were mapped to a Likert scale:
````
utaut_columns = []
for i in range(len(columns)):
  if columns[i].find("Q14") >= 0 :
    utaut_columns.append(columns[i])
utaut_df = df[utaut_columns]

utaut_mapping = {
    "Strongly Disagree" : 1,
    "Disagree" : 2,
    "Neither agree or disagree" : 3,
    "Agree": 4,
    "Strongly Agree" : 5,

}
utaut_df = utaut_df.replace(utaut_mapping)
````
The total UTAUT score was calulated by adding up all the question response scores and theses scores were added to the main dataframe:
````
total_utaut = utaut_df.sum(axis=1)
df['Total UTAUT Score']= total_utaut
````
A separate dataframe was then created with all of the scores from all of the questionnaires and the demographic information needed for ease of use:
````
scores_df = df[['Q4','Q5','PHQ9 Score', 'Total MSPSS Score', 'Total UTAUT Score', 'Task Oriented Score','Emotion Oriented Score', 'Avoidance Oriented Score', 'Overall Coping Style', 'Total Family MSPSS Score' ]]
````
# Part III: Exploratory Data Analysis
## Linear Regression
A linear regression was conducted between Total MSPSS Score and Total UTAUT score to analyze the relationship between these two values.
````
phq9 = scores_df['Total MSPSS Score']
utaut = scores_df['Total UTAUT Score']

corr, pval = pearsonr(phq9, utaut)

print(f"Correlation: {corr:.3f}, p-value: {pval:.3f}")
````
Correlation: -0.188, p-value: 0.319

Then it was visualized and separated for African Americans and black immigrants to see if there are any significant differences between those two populations.
````
sns.set(style="whitegrid")
scores_df['Group'] = scores_df['African-American or Immigrant'].map({
    1: 'African-American',
    0: 'Immigrants'
})

filtered_df = scores_df[scores_df['Total UTAUT Score'] != 0]

sns.lmplot(
    data=filtered_df,
    x='Total UTAUT Score',
    y='PHQ9 Score',
    hue='Group',
    height=6,
    aspect=1.5,
    markers=['o', 's'],
    palette='Set1',
    ci=95
)

plt.xlim(0, 100)  
plt.ylim(0, None)  
plt.title('Relationship between PHQ-9 Score and UTAUT Score\nby Ethnicity')
plt.xlabel('Total UTAUT Score')
plt.ylabel('PHQ-9 Score')
plt.tight_layout()
plt.show()
````

![image](https://github.com/user-attachments/assets/a523d7fa-c0a0-4b2b-b8d5-291ebb58b3ab)
## Student's T-Test
The next model that was run was a student's t-test to analyze if the MSPSS scores between African-Americans and Black Immigrants had a significant difference
````
african_americans = scores_df[scores_df['African-American or Immigrant'] == 1]['Total MSPSS Score']
immigrants = scores_df[scores_df['African-American or Immigrant'] == 0]['Total MSPSS Score']

t_statistic, p_value = ttest_ind(african_americans, immigrants)
````
Our T-statistic was 1.1297030828596402, our P-value was 0.26818671513459885. Therefore there is no statistically significant difference in MSPSS Score between African-Americans and Immigrants.

This data was visualized in a barplot with error bars.
````
groups = ['African-Americans', 'Immigrants']
means = [np.mean(african_americans), np.mean(immigrants)] 
errors = [np.std(african_americans), np.std(immigrants)]

plt.figure(figsize=(8, 6))
plt.bar(groups, means, yerr=errors, capsize=5, color=['skyblue', 'lightcoral']) 
plt.xlabel('Ethnicity')
plt.ylabel('Mean MSPSS Score')
plt.title(f'Mean MSPSS Score by Ethnicity (t = {t_statistic:.2f}, p = {p_value:.3f})')

plt.tight_layout()
plt.show()
````
![image](https://github.com/user-attachments/assets/c74cf1e2-3204-44ba-8b0e-e1f7c8f5f274)
## Chi-Squared Test
Then a chi-squared test was run on the CISS data of the different coping styles to see if African-Americans and Immirgants had significantly different dominant coping styles. To do this, the tables containing the overall coping style for each participant had to be manipulated
````
style_expanded = scores_df.explode('Overall Coping Style')

# Map readable labels for group
style_expanded['Group'] = scores_df['African-American or Immigrant'].map({
    1: 'African-American',
    0: 'Immigrant'
})

coping_counts = style_expanded.groupby(['Overall Coping Style', 'Group']).size().reset_index(name='Count')
````
Then it was visualized with a barplot.
````
plt.figure(figsize=(10, 6))
sns.barplot(
    data=coping_counts,
    x='Overall Coping Style',
    y='Count',
    hue='Group',
    palette='Set2'
)

plt.title('Coping Style Preference by Group')
plt.xlabel('Overall Coping Style')
plt.ylabel('Number of Participants')
plt.xticks(rotation=20)
plt.tight_layout()
plt.show()
````
![image](https://github.com/user-attachments/assets/01113ec5-a305-4261-830d-fcb3ba942221)
