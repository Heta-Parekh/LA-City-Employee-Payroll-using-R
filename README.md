# LA-City-Employee-Payroll-using-R
Understand and interpreting data, analyze the results using statistical techniques and provide appropriate visualizations using ggplot2. 

This project has been developed to find the pattern for the salary range and to understand which department offers the highest amount of salary with benefits. Employees have their own payroll management needs, and this project can help the HR department be aware of different department offers and improve the hiring process. 

## The dataset for this project has been taken from the URL link:
[LA_City Employee Payroll](https://controllerdata.lacity.org/Payroll/City-Employee-Payroll-Current-/g9h8-fvhu/data)

## Data Cleaning
**Removing Unnecessary Columns**
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
**Convert Column Data Type**
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
**Removing the NA Values**
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

**On average, which were the top 10 full-time positions that paid the most from 2013 to 2021?**
```
> pay_by_role <- subset(usable_columns, EMPLOYMENT_TYPE == 'FULL_TIME', select=c(MOU_TITLE,
+                                                                                REGULAR_PAY))
> avg_pay_by_title<-pay_by_role %>%
+   group_by(MOU_TITLE) %>%
+   summarise(AVG_PAY=mean(REGULAR_PAY))
> 
> top_ten <- head(arrange(avg_pay_by_title, desc(AVG_PAY)), n = 10)
> ggplot(top_ten) + 
+   geom_col(aes(x= reorder(MOU_TITLE, AVG_PAY), y=AVG_PAY, fill = MOU_TITLE), show.legend=FALSE) +
+   coord_flip() +
+   scale_fill_manual(values = c(
+     "UNREPRESENTED UNIT - MANAGEMENT BENEFITS" = "#52BE80",
+     "PORT PILOTS" = "lightgrey",
+     "MANAGEMENT ATTORNEYS" = "lightgrey",
+     "FIRE CHIEF OFFICERS" = "lightgrey",
+     "MANAGEMENT EMPLOYEES UNIT" = "lightgrey",
+     "POLICE OFFICERS, CAPTAIN. AND ABOVE" = "lightgrey",
+     "PERSONNEL DIRECTOR" = "lightgrey",
+     "LOS ANGELES PORT POLICE COMMAND OFFICERS" = "lightgrey",
+     "SUPERVISORY PROFESSIONAL UNIT" = "lightgrey",
+     "CONFIDENTIAL ATTORNEYS" = "lightgrey"
+   )) +
+   theme(axis.ticks = element_blank(),
+         panel.background = element_blank(),
+         axis.title.x = element_blank(),
+         axis.title.y = element_blank(),
+         axis.text.x = element_text(size=14),
+         plot.title = element_text(hjust = -1.91, size=20),
+         plot.subtitle = element_text(hjust = -.539)) +
+   scale_y_continuous(limits=c(0,250000),position="right") +
+   ggtitle(label="Top 10 Paying Jobs in Los Angeles from 2013 to 2021",
+           subtitle = "Salary was averaged for all reporting years.") +
+   annotate("text", x=10, y=220000, label="Top paying job earned an 
+ average of $184,739.20", colour= "#247547", fontface=2, size=4.5)
>
```
![image](https://user-images.githubusercontent.com/75762778/147885556-0f2a0aff-1660-4c08-a4f6-c4b747d02fa3.png)

**Which department generated the highest number of benefits pay each year?**
```
> department_by_benefit <- usable_columns %>%
+ group_by(PAY_YEAR, DEPARTMENT_TITLE, EMPLOYMENT_TYPE) %>%
+ summarise (Benefits_Pay = sum(BENEFIT_PAY))
> department_by_benefit <- filter(department_by_benefit, EMPLOYMENT_TYPE %in% c('FULL_TIME'))
> department_by_benefit <- department_by_benefit [head(order(department_by_benefit$Benefits_Pay,decreasing=TRUE),10), ]
library(ggplot2)
library(scales)
> ggplot(department_by_benefit, aes(PAY_YEAR, DEPARTMENT_TITLE, fill=Benefits_Pay)) + geom_tile() + scale_fill_gradient(low="grey",high="brown",name="Benefits_Pay",labels=comma) + ggtitle("Water and Power Department generated the highest amount of Benefits Pay in the year 2020") + theme(legend.background = element_rect(fill="lightblue", size=0.5, linetype="solid"))
```
![image](https://user-images.githubusercontent.com/75762778/147885590-e0fa2e68-d54c-4880-9f40-812a75811468.png)

**Which gender were given the highest amount of benefits each year?**
```
> pivot_benefit_by_gender <- benefit_pay_by_gender %>%
+ pivot_wider(names_from = GENDER, values_from = Benefit_Pay)
> library(ggplot2)
ggplot(pivot_benefit_by_gender) + geom_line(aes(x=PAY_YEAR, y=FEMALE, group=1, colour='FEMALE'))+ geom_line(aes(x=PAY_YEAR, y=MALE, group=1, colour='MALE'))+ labs(y="Benefits Pay", x="Pay Year", title=("Comparison analysis of Male and Female for benefits pay"))
```
![image](https://user-images.githubusercontent.com/75762778/147885652-b1ecec6b-28b3-40c5-a56c-f5ce5feccced.png)


