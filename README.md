# LA-City-Employee-Payroll-using-R
Understand and interpreting data, analyze the results using statistical techniques and provide appropriate visualizations using ggplot2. 

This project has been developed to find the pattern for the salary range and to understand which department offers the highest amount of salary with benefits. Employees have their own payroll management needs, and this project can help the HR department be aware of different department offers and improve the hiring process. 

## The dataset for this project has been taken from the URL link:
[LA_City Employee Payroll](https://controllerdata.lacity.org/Payroll/City-Employee-Payroll-Current-/g9h8-fvhu/data)

## Data Cleaning
Removing Unnecessary Columns
```
> setwd("~/Grad Courses/5250 Visual Analytics/r_script_project")
> 
> payroll<-read.csv("payroll.csv", header=T)
> 
> usable_columns <- subset(payroll, select=c(PAY_YEAR,
+                                            DEPARTMENT_TITLE,
+                                            MOU_TITLE,
+                                            EMPLOYMENT_TYPE,
+                                            REGULAR_PAY,
+                                            GENDER,
+                                            BENEFIT_PAY,
+                                            OVERTIME_PAY,
+                                            ALL_OTHER_PAY))
> 
> View(usable_columns)
```
Convert Column Data Type
```
> library(tidyverse)
> 
> usable_columns <- usable_columns %>%
+   mutate(PAY_YEAR=as.character(PAY_YEAR),
+          DEPARTMENT_TITLE = as.character(DEPARTMENT_TITLE),
+          MOU_TITLE = as.character(MOU_TITLE),
+          EMPLOYMENT_TYPE = as.character(EMPLOYMENT_TYPE),
+          GENDER = as.character(GENDER),
+          REGULAR_PAY=as.integer(REGULAR_PAY), 
+          BENEFIT_PAY=as.integer(BENEFIT_PAY),
+          OVERTIME_PAY=as.integer(OVERTIME_PAY),
+          ALL_OTHER_PAY=as.integer(ALL_OTHER_PAY))
>
```
Removing the NA Values
```
> benefit_pay_by_gender<-  usable_columns %>%  
+ group_by(PAY_YEAR, GENDER) %>%  
+ summarise(Benefit_Pay=mean(BENEFIT_PAY))
> benefit_pay_by_gender <- benefit_pay_by_gender %>%
+ na.omit
> benefit_pay_by_gender <- subset(benefit_pay_by_gender, GENDER!="",GENDER!="UNKNOWN")
```
