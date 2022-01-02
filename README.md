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
## Analysis and Visualizations
**Which departments had the highest pay each year from 2013 to 2021?** 
```
> total_by_department<-  usable_columns %>%
+   group_by(PAY_YEAR, DEPARTMENT_TITLE) %>%
+   summarise(TOTAL_PAY=sum(REGULAR_PAY + 
+                             OVERTIME_PAY +
+                             BENEFIT_PAY +
+                             ALL_OTHER_PAY))
`summarise()` has grouped output by 'PAY_YEAR'. You can override using the `.groups` argument.
> 
> max_pay_by_year <-  total_by_department %>% 
+   group_by(PAY_YEAR) %>% 
+   slice(which.max(TOTAL_PAY))
> 
> library(ggplot2)
> 
> ggplot(data = max_pay_by_year) +
+   geom_col(mapping=aes(x=PAY_YEAR, y=TOTAL_PAY, fill=DEPARTMENT_TITLE)) +
+   theme(panel.background = element_blank(), 
+         axis.text.x = element_text(vjust=15),
+         axis.ticks.x = element_blank(),
+         axis.title.x = element_text(vjust=8),
+         plot.title = element_text(hjust=.92))+
+   scale_y_continuous(name="Total Pay", labels=scales::comma) +
+   scale_x_discrete(name="Pay Year") +
+   ggtitle("The Police Department has had the highest payroll from 2013 to 2021 with the exception of 2020.") +
+   scale_fill_manual(values= c("lightblue", "gray"),
+                     guide = guide_legend(title="Department"))
```

![image](https://user-images.githubusercontent.com/75762778/147885345-54a7a635-b9a8-4f84-ad96-e715398c253e.png)

**Which department generated the most overtime pay as per year?**
```
> User_Defined_Function <- function(usable_columns){
+ overtime_by_department <- aggregate(OVERTIME_PAY ~ PAY_YEAR + DEPARTMENT_TITLE, data=usable_columns, FUN = sum)
+  group_data <- overtime_by_department[order(overtime_by_department$OVERTIME_PAY, decreasing=TRUE),]
+ return(head(group_data,20))
+ }
> ggplot(User_Defined_Function(usable_columns), aes(PAY_YEAR, OVERTIME_PAY, fill =DEPARTMENT_TITLE)) +  geom_bar( stat = "identity") + ggtitle("Water and Power department has generated the highest overtime pay in the year 2018,2019 and 2020.") + scale_y_continuous(name= "Over-Time Pay",labels=scales::comma) + scale_fill_brewer()
```
![image](https://user-images.githubusercontent.com/75762778/147885466-f813d54c-14f8-4258-9f88-d1df3bb82bbe.png)



