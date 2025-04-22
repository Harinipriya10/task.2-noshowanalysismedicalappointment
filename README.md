*************************************************************BRAZILIAN MEDICAL APPOINTMENT REPORT DATA ANALYSIS REPORT**************************************************************


DATASET DESCRPITION:
The dataset is about medical appointments in BRAZIL, a sample over 100k the information of this dataset is collected from KAGGLE.

Tools Used:  Python (Pandas), Numpy.

This report details the data cleaning and preparation process for the Brazilian medical No-Show Appointments Dataset, which contains medical appointment records with patient demographics, appointment scheduling details, and whether the patient attended ("No") or missed ("Yes") their appointment. 

OBJECTIVES:

•	Identify and handle missing/incorrect data 
•	Remove duplicates 
•	Standardize text and date formats 
•	Rename columns for clarity 
•	Convert data types appropriately 
•	Engineer new features for better analysis 

INITIAL DATA ASSESSMENT:

Dataset Overview 

•	Rows: 110,527 (before cleaning) 
•	Columns: 14 


Key Variables:
  
•	Patient Id, Appointment ID (unique identifiers) 
•	Gender (M/F) 
•	Scheduled Day, Appointment Day (timestamps) 
•	Age (numeric) 
•	Neighbourhood (location) 
•	Medical conditions (`Scholarship, Hipertension, Diabetes, Alcoholism, Handcap) 
•	SMS_received (if a reminder was sent) 
•	No-show (target variable: "Yes" if missed, "No" if attended) 

INITIAL DATA QUALITY ISSUES

Issue                                                          Example                                                         Impact  
Inconsistent column naming (Hipertension, Handcap)     Hipertension (misspelled)  Harder to interpret  
Text case inconsistencies (`No-show` values)         "No" vs. "no"           Affects grouping 
Date columns as strings           2016-04-29T18:38:08Z         Needs datetime conversion 
Possible age outliers               Negative ages, extreme values                    Skews analysis 
Potential duplicates            Same `PatientId` & `AppointmentID`                Overcounting  





DATA CLEANING VALUES 

Handling Missing Values
Value Check:
python -
print(df.isnull().sum())

      Result: No missing values found. 

Removing Duplicates
Action:
python - 
df.drop_duplicates(inplace=True)

     Result: 4 duplicate rows removed
     Final count: 110,523 rows

STANDARDIZING TEXT VALUES

Actions:
1. Gender: Convert to uppercase (`M`/`F`) 
python
   df['Gender'] = df['Gender'].str.upper()
   
2. No-show: Standardize to `Yes`/`No` 
python
   df['No-show'] = df['No-show'].str.title()



COLUMN RENAMING & FORMATTING 

Actions:
Original Column                               Cleaned Column                                                  Reason 
PatientId                                          patient_id                                    Snake case, lowercase 
AppointmentID                   appointment_id                                     Snake case, lowercase 
ScheduledDay                     scheduled_day                                      Snake case, lowercase  
AppointmentDay                appointment_day                                Snake case, lowercase 
Hipertension                       hypertension                                        Correct spelling  
Handcap                              handicap                                                More accurate term  
No-show                              no_show                                                Snake case  

python
df.columns = df.columns.str.lower()
df.rename(columns={
    'patientid': 'patient_id',
    'appointmentid': 'appointment_id',
    'scheduledday': 'scheduled_day',
    'appointmentday': 'appointment_day',
    'hipertension': 'hypertension',
    'handcap': 'handicap',
    'no-show': 'no_show'
}, inplace=True) 

DATE FORMATTING

Actions:
- Convert `scheduled_day` and `appointment_day` to `datetime` 
- Extract useful temporal features: 
  - Day of the week (for no-show trends) 
  - Time between scheduling & appointment


python
df['scheduled_day'] = pd.to_datetime(df['scheduled_day'])
df['appointment_day'] = pd.to_datetime(df['appointment_day'])

Days between scheduling & appointment
df['days_between'] = (df['appointment_day'] - df['scheduled_day']).dt.days
df['days_between'] = df['days_between'].apply(lambda x: 0 if x < 0 else x)  # Fix negative values

Day of week
df['appointment_dow'] = df['appointment_day'].dt.day_name()
df['scheduled_dow'] = df['scheduled_day'].dt.day_name()

HANDLING OUTLIERS (AGE)

Actions: 
-  Negative ages → Set to `0`
-  Unrealistic ages (e.g., >110) → Capped at `110`
-  Age groups created for better analysis

python
df['age'] = df['age'].apply(lambda x: 0 if x < 0 else x)
df['age'] = df['age'].apply(lambda x: 110 if x > 110 else x)

Age groups
bins = [0, 12, 19, 30, 50, 65, 110]
labels = ['Child', 'Teen', 'Young Adult', 'Adult', 'Middle Aged', 'Senior']
df['age_group'] = pd.cut(df['age'], bins=bins, labels=labels, right=False)
 

DATA TYPE CONVERSION

Actions: 

Column                           Original Type                     New Type                                 Reason  

scholarship                    int64                                    int8                            Optimize memory 
hypertension                int64                                    int8                            Optimize memory 
diabetes                         int64                                   int8                            Optimize memory 
alcoholism                     int64                                   int8                            Optimize memory 
handicap                        int64                                  int8                             Optimize memory 
sms_received                int64                                  int8                             Optimize memory 
age                                  int64                                  int8                             Optimize memory 

python
numeric_cols = ['scholarship', 'hypertension', 'diabetes', 'alcoholism', 'handicap', 'sms_received']
df[numeric_cols] = df[numeric_cols].astype('int8')
df['age'] = df['age'].astype('int8')
 

FUTURE DATA STRUCTURE
Cleaned Columns 

Column                                                   Description                                                       Type 

patient_id                                              Unique patient ID                                             int64  
appointment_id                                   Unique appointment ID                                   int64  
gender                                                    Patient gender (M/F)                                     object  
scheduled_day                                      When appointment was scheduled       datetime  
appointment_day                                 When appointment occurred                 datetime 
age                                                           Patient age (0-110)                                            int8 neighbourhood                                     Location of appointment                              object  
scholarship                                             Welfare program (0/1)                                      int8  
hypertension                                          Hypertension status (0/1)                                int8  
diabetes                                                  Diabetes status (0/1)                                         int8  
alcoholism                                              Alcoholism status (0/1)                                     int8  
handicap                                                 Handicap status (0/1)                                        int8 sms_received                                        SMS reminder sent (0/1)                                   int8  
no_show                                                Missed appointment (Yes/No)                     object 
days_between                                      Days between scheduling & appointment  int64 
age_group                                             Age category (Child, Teen, etc.)               category 
appointment_dow                               Day of week (Monday-Sunday)                    object  
scheduled_dow                                    Day of week when scheduled                       object 

Key Improvement
 
•	No missing data
•	No duplicates 
•	Consistent naming & formatting
•	Optimized data types (memory-efficient) 
•	New features (`days_between`, `age_group`) 

Next Steps (Exploratory Analysis Ideas)
No-show rate by:
  - Gender 
  - Age group 
  - Neighbourhood 
  - Days between scheduling & appointment 
  - SMS reminders 


Trends over time:
  - Are no-shows higher on certain days? 
  - Does lead time affect attendance? 


CONCLUSION
The cleaned dataset is now ready for analysis, with: 
Consistent formatting
Correct data types 
Removed duplicates 
New useful features

Cleaned Dataset: 109529 rows, 14 columns, new dataset 19 columns.
Ready for further analysis and data modeling.
Future Additions:
•	EDA
•	Predictive modeling
•	Dashboard visualization of datasets
Prepared by: HARINIPRIYASELVAM 
Date:  22/04/2025
