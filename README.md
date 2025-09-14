# HR Analytics: Job Change of Data Scientists
## Intorduction
 Context and Content

A company which is active in Big Data and Data Science wants to hire data scientists among people who successfully pass some courses which conduct by the company.

Many people signup for their training. Company wants to know which of these candidates are really wants to work for the company after training or looking for a new employment because it helps to reduce the cost and time as well as the quality of training or planning the courses and categorization of candidates.

Information related to demographics, education, experience are in hands from candidates signup and enrollment.

##  DATA SOURCES
### 1. Enrollies' data
As enrollies are submitting their request to join the course via Google Forms, we have the Google Sheet that stores data about enrolled students, containing the following columns:

- enrollee_id: unique ID of an enrollee
- full_name: full name of an enrollee
- city: the name of an enrollie's city
- gender: gender of an enrollee
- The source: https://docs.google.com/spreadsheets/d/1VCkHwBjJGRJ21asd9pxW4_0z2PWuKhbLR3gUHm-p4GI/edit?usp=sharing

```
google_sheet_id = '1VCkHwBjJGRJ21asd9pxW4_0z2PWuKhbLR3gUHm-p4GI'
url = 'https://docs.google.com/spreadsheets/d/' + google_sheet_id + '/export?format=xlsx'
enrollies = pd.read_excel(url, sheet_name = 'enrollies')
```

### 2. Enrollies' education
After enrollment everyone should fill the form about their education level. This form is being digitalized manually. Educational department stores it in the Excel format here: https://assets.swisscoding.edu.vn/company_course/enrollies_education.xlsx

This table contains the following columns:

- enrollee_id: A unique identifier for each enrollee. This integer value uniquely distinguishes each participant in the dataset.

- enrolled_university: Indicates the enrollee's university enrollment status. Possible values include no_enrollment, Part time course, and Full time course.

- education_level: Represents the highest level of education attained by the enrollee. Examples include Graduate, Masters, etc.

- major_discipline: Specifies the primary field of study for the enrollee. Examples include STEM, Business Degree, etc.

``` 
url = 'enrollies_education.xlsx'
enrollies_education = pd.read_excel(url, sheet_name = 'enrollies_education')
```

### 3. Enrollies' working experience
Another survey that is being collected manually by educational department is about working experience.

Educational department stores it in the CSV format here: https://assets.swisscoding.edu.vn/company_course/work_experience.csv

This table contains the following columns:

- enrollee_id: A unique identifier for each enrollee. This integer value uniquely distinguishes each participant in the dataset.

- relevent_experience: Indicates whether the enrollee has relevant work experience related to the field they are currently studying or working in. Possible values include Has relevent experience and No relevent experience.

- experience: Represents the number of years of work experience the enrollee has. This can be a specific number or a range (e.g., >20, <1).

- company_size: Specifies the size of the company where the enrollee has worked, based on the number of employees. Examples include 50−99, 100−500, etc.

- company_type: Indicates the type of company where the enrollee has worked. Examples include Pvt Ltd, Funded Startup, etc.

- last_new_job: Represents the number of years since the enrollee's last job change. Examples include never, >4, 1, etc.

``` working_experience = pd.read_csv('work_experience.csv') ``` 

### 4. Training hours
From LMS system's database you can retrieve a number of training hours for each student that they have completed.

Database credentials:

- Database type: MySQL
- Host: 112.213.86.31
- Port: 3360
- Login: etl_practice
- Password: 550814
- Database name: company_course
- Table name: training_hours

```
!pip install pymysql
from sqlalchemy import create_engine
import pymysql

engine = create_engine('mysql+pymysql://etl_practice:550814@112.213.86.31:3360/company_course')
training_hours = pd.read_sql_table('training_hours', engine)
```

### 5. City development index
Another source that can be usefull is the table of City development index.

- The City Development Index (CDI) is a measure designed to capture the level of development in cities. It may be significant for the resulting prediction of student's employment motivation.

- It is stored here: https://sca-programming-school.github.io/city_development_index/index.html

```
url = 'https://sca-programming-school.github.io/city_development_index/index.html'
tables = pd.read_html(url)
cities = tables[0]
```

### 6. Employment
From LMS database you can also retrieve the fact of employment. If student is marked as employed, it means that this student started to work in our company after finishing the course.

Database credentials:

- Database type: MySQL
- Host: 112.213.86.31
- Port: 3360
- Login: etl_practice
- Password: 550814
- Database name: company_course
- Table name: employment

```
employment = pd.read_sql_table('employment', engine)
```

## TRANSFORM DATA
### 1. Enrollies' data

```
cat_cols = ['city','gender']
enrollies['full_name'] = enrollies['full_name'].astype('string')
enrollies[cat_cols] = enrollies[cat_cols].astype('category')
enrollies.info()
```
<class 'pandas.core.frame.DataFrame'>

RangeIndex: 19158 entries, 0 to 19157

Data columns (total 4 columns):

| # | Column      | Non-Null Count | Dtype    |
|---|-------------|----------------|----------|
| 0 | enrollee_id | 19158 non-null | int64    |
| 1 | full_name   | 19158 non-null | string   |
| 2 | city        | 19158 non-null | category |
| 3 | gender      | 14650 non-null | category |

dtypes: category(2), int64(1), string(1)  
memory usage: 342.1 KB

```
Add 'unkown' to the categories
enrollies['gender'] = enrollies['gender'].cat.add_categories('unkown')

Fill missing values with 'unkown'
enrollies['gender'] = enrollies['gender'].fillna('unkown')
```
```
enrollies.info()
```

<class 'pandas.core.frame.DataFrame'>

RangeIndex: 19158 entries, 0 to 19157

Data columns (total 4 columns):

| # | Column      | Non-Null Count | Dtype    |   
|---|-------------|----------------|----------|
|0  | enrollee_id | 19158 non-null | int64    |
| 1 |  full_name  |  19158 non-null|  string  |
| 2 |  city       |  19158 non-null|  category|
| 3 |  gender     |  19158 non-null| category |

dtypes: category(2), int64(1), string(1)

memory usage: 342.1 KB

```
standardize_cols = ['full_name','city']
for clo in standardize_cols:
    enrollies[clo] = enrollies[clo].str.lower()

# Now convert gender to string and then to lowercase
enrollies['gender'] = enrollies['gender'].astype('str').str.lower()
  ## capitalize(), upper()
```
```
enrollies.head()
```

|#  |enrollee_id   |	full_name   |city	   |gender|
|---|-------------|-------------|--------|-------|
|0  | 8949	      |mike jones	  |city_103|male   |
|1  |	29725	      |laura jones	|city_40 |male   |
|2  |	11561	      |david miller	|city_21 |unkown |
|3  |	33241    	  |laura davis	|city_115|unkown |
|4  |	666	        |alex martinez|city_162|male   |


### 2. Enrollies' education

```
enrollies_education.info()
```

<class 'pandas.core.frame.DataFrame'>

RangeIndex: 19158 entries, 0 to 19157

Data columns (total 4 columns):

| # | Column               | Non-Null Count | Dtype |   
|---|----------------------|----------------|-------|
| 0 |  enrollee_id         | 19158 non-null | int64 |
| 1 |  enrolled_university | 18772 non-null | object|
| 2 |  education_level     | 18698 non-null | object|
| 3 |  major_discipline    | 16345 non-null | object|
 
dtypes: int64(1), object(3)
memory usage: 598.8+ KB
```
enrollies_education.head()
```

|# |enrollee_id|enrolled_university	 |education_level	|major_discipline|
|--|--------------|------------------|----------------|----------------|
|0 |	8949	      |  no_enrollment	 |   Graduate	    |     STEM       |
|1 |	29725	      |  no_enrollment	 |   Graduate	    |     STEM       |
|2 |	11561	      |  Full time course|	  Graduate	  |     STEM     |
|3 |	33241	      |  NaN	Graduate	 |   Business     |     Degree    |
|4 |	666	        |  no_enrollment	 |   Masters	    |     STEM     |

```
cat_cols = ['enrolled_university','education_level','major_discipline']

enrollies_education[cat_cols] = enrollies_education[cat_cols].astype('category')
enrollies_education.info()
```

<class 'pandas.core.frame.DataFrame'>

RangeIndex: 19158 entries, 0 to 19157

Data columns (total 4 columns):

| # | Column              | Non-Null Count | Dtype    |   
|---|---------------------|----------------|----------|  
| 0 |  enrollee_id        |  19158 non-null| int64    |
| 1 |  enrolled_university|  18772 non-null| category |
| 2 |  education_level    |  18698 non-null| category |
| 3 |  major_discipline   |  16345 non-null| category |

dtypes: category(3), int64(1)
memory usage: 206.5 KB

```
enrollies_education['enrolled_university'] = enrollies_education['enrolled_university'].cat.add_categories('unkown')
enrollies_education['enrolled_university'] = enrollies_education['enrolled_university'].fillna('unkown')
enrollies_education['education_level'] = enrollies_education['education_level'].cat.add_categories('unkown')
enrollies_education['education_level'] = enrollies_education['education_level'].fillna('unkown')
enrollies_education['major_discipline'] = enrollies_education['major_discipline'].cat.add_categories('unkown')
enrollies_education['major_discipline'] = enrollies_education['major_discipline'].fillna('unkown')
```


### 3. Enrollies' working experience

```
cat_cols = ['experience','company_size','company_type','relevent_experience']
working_experience['last_new_job'] = working_experience['last_new_job'].astype('string')
working_experience[cat_cols] = working_experience[cat_cols].astype('category')
working_experience.info()
```
<class 'pandas.core.frame.DataFrame'>

RangeIndex: 19158 entries, 0 to 19157

Data columns (total 6 columns):

| # | Column              | Non-Null Count | Dtype    |   
|---|---------------------|----------------|----------|  
| 0 |  enrollee_id        |  19158 non-null|  int64   |
| 1 |  relevent_experience|  19158 non-null|  category|
| 2 |  experience         |  19093 non-null|  category|
| 3 |  company_size       |  13220 non-null|  category|
| 4 |  company_type       |  13018 non-null|  category|
| 5 |  last_new_job       |  18735 non-null|  string  |

dtypes: category(4), int64(1), string(1)

memory usage: 375.7 KB

```
working_experience['experience',] = working_experience['experience'].cat.add_categories('unkown').fillna('unkown')
working_experience['company_size'] = working_experience['company_size'].cat.add_categories('unkown').fillna('unkown')
working_experience['company_type'] = working_experience['company_type'].cat.add_categories('unkown').fillna('unkown')
working_experience['last_new_job'] = working_experience['last_new_job'].fillna('unkown')
```
```
working_experience.info()
```
| # | Column              | Non-Null Count | Dtype    |   
|---|---------------------|----------------|----------|  
| 0 |  enrollee_id        |  19158 non-null|  int64   |
| 1 |  relevent_experience|  19158 non-null|  category|
| 2 |  experience         |  19158 non-null|  category|
| 3 |  company_size       |  19158 non-null|  category|
| 4 |  company_type       |  19158 non-null|  category|
| 5 |  last_new_job       |  19158 non-null|  string  |


### 4. Training hours

```
training_hours.info()
```

<class 'pandas.core.frame.DataFrame'>

RangeIndex: 19158 entries, 0 to 19157

Data columns (total 2 columns):

| # | Column              | Non-Null Count | Dtype |   
|---|---------------------|----------------|-------|  
| 0 |  enrollee_id        |19158 non-null  | int64 | 
| 1 |  training_hours     |19158 non-null  | int64 |

dtypes: int64(2)
memory usage: 299.5 KB


### 5. City development index

```
cities.info()
```

<class 'pandas.core.frame.DataFrame'>

RangeIndex: 123 entries, 0 to 122

Data columns (total 2 columns):

| # | Column                 | Non-Null Count | Dtype |   
|---|------------------------|----------------|-------|   
| 0 |  City                  |  123 non-null  |object |
| 1 |  City Development Index|  123 non-null  |float64|

dtypes: float64(1), object(1)
memory usage: 2.1+ KB


### 6. Employment

```
employment.info()
```

<class 'pandas.core.frame.DataFrame'>
RangeIndex: 19158 entries, 0 to 19157
Data columns (total 2 columns):
| # | Column              | Non-Null Count | Dtype   |   
|---|---------------------|----------------|---------|  
| 0 |  enrollee_id        |19158 non-null  | int64   | 
| 1 |  employed           |19158 non-null  | float64 |

dtypes: float64(1), int64(1)

memory usage: 299.5 KB


## LOAD DATA

```
db_path = 'data_warehouse.db'

engine = create_engine(f'sqlite:///{db_path}')

enrollies.to_sql('enrollies', engine, if_exists = 'replace', index = False)
enrollies_education.to_sql('enrollies_education', engine, if_exists = 'replace', index = False)
working_experience.to_sql('working_experience', engine, if_exists = 'replace', index = False)
training_hours.to_sql('training_hours', engine, if_exists = 'replace', index = False)
cities.to_sql('cities', engine, if_exists = 'replace', index = False)
employment.to_sql('employment', engine, if_exists = 'replace', index = False)
```























