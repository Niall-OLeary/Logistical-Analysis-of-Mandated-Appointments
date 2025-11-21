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
  Column 1: 0 = participant's attendance rate decreased or stayed the same, 1 = attendance rate **improved**.
  Column 2: 0 = participant did not reach a Job Outcome, 1 = participant **did** reach a Job Outcome.


2. **Logistic Regression Model**:
   - I employed two logistic regression models to evaluate the effect of mandatory attendance on program outcomes and attendance rates.
   - The independent variable in both models was Mandated (mandatory vs. voluntary).
   - The dependant variables were Job_Outcome and Attendance_Improvement.

## Outcome

The logistic regression analysis revealed several key findings:

- **Attendance Rate**: The coefficient for the MANDATED variable is -0.2309, indicating that participants in the mandated group were less likely to show an improvement in attendance compared to participants in the control group. The p-value for MANDATED is 0.000, indicating that the effect of mandatory attendance on attendance improvement is statistically significant. The Pseudo R-squared value is 0.002229, suggesting that the model explains a very small proportion of the variation in the outcome
  
- **Job Outcome**: The coefficient for the Mandated variable was -1.2034, indicating that participants who were mandated to attend appointments had a lower probability of achieving a Job Outcome, compared to those in the control group. The p-value for the Mandated variable is 0.000, which is highly significant. The Pseudo R-squared value of 0.03922 suggests that the model explains a small portion of the variation in the outcome, other factors likely contribute. 

## Impact

The findings from this analysis have substantial implications for program management:

- **Policy Decision**: The data supports the implementation of mandatory attendance policies for programs where attendance is a key factor in achieving success. By enforcing attendance, programs can boost engagement and improve the likelihood of positive outcomes.
  
- **Program Design**: Insights gained from demographic trends can guide the design of future programs. Tailoring policies to specific participant needs (e.g., additional support for less engaged groups) may further enhance participation and success rates.

- **Resource Allocation**: Understanding the impact of attendance mandates allows program managers to allocate resources more effectively. Programs can prioritize strategies that encourage attendance, knowing that increased participation correlates with better outcomes.

## Conclusion

This analysis demonstrates that mandating appointment attendance not only increases participant engagement but also contributes to improved program outcomes. The insights gained from this study can help inform future policy decisions and refine program strategies to maximize success.

For more details, refer to the code and analysis in this repository.
