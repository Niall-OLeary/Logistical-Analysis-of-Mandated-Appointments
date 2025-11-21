# Logistical-Analysis-of-Mandated-Appointments

## Business Question

The core question of this analysis is:  
**What effect does mandating appointment attendance have on participant's attendance and programme outcome (whether a participant reaches their final milestone)?**

Within our programme (and many others) attendance plays a key role in ensuring participants fully engage with the content and receive the intended benefits. If a participant fails to attend appointments, we can mandate them - making attendance mandatory and continuing non-attendance leads to a loss of benefits. The question arises as to whether enforcing mandatory attendance increases participant involvement and improves the program's overall success. This analysis aimed to answer this question by evaluating attendance patterns and program outcomes before and after the implementation of mandatory attendance policies.

## Solution

To address this question, we conducted a logistic regression analysis using Python to evaluate the relationship between attendance mandates and programme success. The key steps involved were:

1. Data Collection & Cleaning  
The dataset consists of two groups:

A. **Mandated Group**: This group consists of participants who were required to attend appointments. Attendance for this group was calculated both **before** and **after** the implementation of the mandatory     attendance policy.  

B. **Control Group**: This group consists of participants who were not subject to the mandatory attendance policy (i.e. voluntary attendance). Attendance for this group was calculated separately for each         **half** of the program duration (12 months), allowing for a comparison of attendance over time.  

For both groups, two binary columns were calculated:
   - Column 1: 0 = participant's attendance rate decreased or stayed the same, 1 = attendance rate **improved**.
   - Column 2: 0 = participant did not reach a Job Outcome, 1 = participant **did** reach a Job Outcome.
<details> 
<summary><strong>Data Collection Script</strong></summary>
   
```sql
with a as (
select 
participant_id,
activity_main_status,
activity_status,
activity_type,
activity_completed_date
from vw_activity_union
where activity_class = 'Appointment' and activity_status in ('Completed - Completed','Cancelled - Failed to Attend')
),

m as (
select * from (
select 
*,
row_number() over(partition by participant_id order by mandation_date asc) as rn
from (
select
pf.participant_id,
Date(pf.file_created_date) as mandation_date
from stg_pics_participant_files pf
where pf.file_type = 'Mandation Letter'
union
select
af.participant_id,
Date(af.file_created_date) as mandation_date
from stg_pics_activity_file_upload af
where af.file_type = 'Mandation Letter'
) as A
) as B
where rn =1 
),

b as (
select 
dr.participant_id,
case when m.mandation_date is not null then '1' else '0' end as mandated,
add_months(dr.start_date,6) as scheme_halfway_point,
m.mandation_date,
milestone_job_out,
milestone_comm_earn,
milestone_job_start
from vw_daily_report dr
left join m on dr.participant_id = m.participant_id
left join vw_daily_stats ds on ds.participant_id = dr.participant_id
where start_date is not null and start_status = 'Started'
)

select
c.participant_id,
mandated,
scheme_halfway_point,
mandation_date,
milestone_job_out,
milestone_comm_earn,
milestone_job_start,
case 
    when C.mandation_date is null then div0(count(case when a.activity_main_status = 'Completed' then a.participant_id else null end), count(a.participant_id))
    else div0(count(case when a_before.activity_main_status = 'Completed' then a_before.participant_id else null end), count(a_before.participant_id))
end as attendance_before,

case 
    when C.mandation_date is null then div0(count(case when ab.activity_main_status = 'Completed' then ab.participant_id else null end), count(ab.participant_id)) 
    else div0(count(case when a_after.activity_main_status = 'Completed' then a_after.participant_id else null end), count(a_after.participant_id)) 
end as attendance_after
from C

left join a on a.participant_id = C.participant_id and a.activity_completed_date < scheme_halfway_point
left join a ab on ab.participant_id = C.participant_id and ab.activity_completed_date >= scheme_halfway_point
left join a a_before on a_before.participant_id = C.participant_id and a_before.activity_completed_date < C.mandation_date
left join a a_after on a_after.participant_id = C.participant_id and a_after.activity_completed_date > C.mandation_date
group by 1,2,3,4,5,6,7
```

</details>

2. **Logistic Regression Model**:
   - I employed two logistic regression models to evaluate the effect of mandatory attendance on program outcomes and attendance rates.
   - The independent variable in both models was Mandated (mandatory vs. voluntary).
   - The dependant variables were Job_Outcome and Attendance_Improvement.
<details> 
<summary><strong>Regression Model Python Script</strong></summary>
   
```python
import pandas as pd
import statsmodels.api as sm

# Load your data
df = pd.read_csv(r'C:\Users\Niall.OLeary\Downloads\Mandation Testing - milestones.csv')

# Define dependent and independent variables
y = df['MILESTONE_JOB_OUT']                # Outcome (0 or 1)
X = df[['Mandated']]                 # Predictor (0 or 1)

# Add a constant (intercept term)
X = sm.add_constant(X)

# Fit logistic regression model
model = sm.Logit(y, X)
result = model.fit()

# Print results
print(result.summary())

Optimization terminated successfully.
         Current function value: 0.488047
         Iterations 6
                           Logit Regression Results                           
==============================================================================
Dep. Variable:      MILESTONE_JOB_OUT   No. Observations:                96324
Model:                          Logit   Df Residuals:                    96322
Method:                           MLE   Df Model:                            1
Date:                Fri, 07 Nov 2025   Pseudo R-squ.:                 0.03922
Time:                        13:50:00   Log-Likelihood:                -47011.
converged:                       True   LL-Null:                       -48929.
Covariance Type:            nonrobust   LLR p-value:                     0.000
==============================================================================
                 coef    std err          z      P>|z|      [0.025      0.975]
------------------------------------------------------------------------------
const         -1.0551      0.009   -117.988      0.000      -1.073      -1.038
Mandated      -1.2034      0.021    -56.352      0.000      -1.245      -1.162
==============================================================================
```

</details>

## Outcome

The logistic regression analysis revealed several key findings:

- **Attendance Rate**: The coefficient for the mandated variable is -0.2309, indicating that participants in the mandated group were less likely to show an improvement in attendance compared to participants in the control group. The p-value for MANDATED is 0.000, indicating that the effect of mandatory attendance on attendance improvement is statistically significant. The Pseudo R-squared value is 0.002229, suggesting that the model explains a very small proportion of the variation in the outcome
  
- **Job Outcome**: The coefficient for the Mandated variable was -1.2034, indicating that participants who were mandated to attend appointments had a lower probability of achieving a Job Outcome, compared to those in the control group. The p-value for the Mandated variable is 0.000, which is highly significant. The Pseudo R-squared value of 0.03922 suggests that the model explains a small portion of the variation in the outcome, other factors likely contribute. 


## Key Takeaways

Mandating appointments does not appear to be an effective intervention for re-engaging participants who have previously failed to attend, nor does it increase the likelihood a participant achieves a Job Outcome. Participants who are forced to attend may feel bitter or less motivated, which could negatively impact their performance and attendance. The data suggest that alternative strategies should be considered for participants who miss voluntary appointments, as mandation alone often fails to improve attendance and may even worsen it (15% decrease on average).

I could not conclusively attribute this result solely to the mandatory attendance policy. There are several factors that could be influencing these outcomes, such as the nature of the programme itself. Voluntary participants may be more intrinsically motivated, leading to higher engagement and better outcomes. Mandated participants may have differing support needs, which we may not be addressing, and therefore, we might mistakenly interpret non-attendance as a lack of motivation or commitment. In reality, these participants may face barriers such as logistical challenges, lack of engagement with the program content, or even external factors (e.g. personal or financial difficulties) that are preventing their attendance.

Due to the low Pseudo R-squared value of both tests, it may be worthwhile to explore additional variables that could explain more of the variation in attendance improvement and Job Outcomes.
