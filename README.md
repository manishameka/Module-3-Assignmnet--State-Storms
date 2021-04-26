Module-3-Assignmnet-State Storms
title: "Module 3 Assignment"
author: "Manisha Meka"
date: "4/25/2021"

---

#Move this into a good local directory for your current working directory and read it in to R using read_csv from the readr package.
```{r}
library(readr)
library(dplyr)
Stormdetails <- read_csv ("StormEvents_details-ftp_v1.0_d1992_c20170717.csv")
Stormdetails
```

#Limit the dataframe to: the beginning and ending dates and times, the episode ID, the event ID, the state name and FIPS, the “CZ” name, type, and FIPS, the event type, the source, and the beginning latitude and longitude and ending latitude and longitude (10points)
```{r}
myvars <- c("BEGIN_YEARMONTH","BEGIN_DAY", "BEGIN_TIME", "END_YEARMONTH","END_DAY","END_TIME","EPISODE_ID", "EVENT_ID", "STATE","STATE_FIPS","EVENT_TYPE","CZ_TYPE","CZ_NAME", "CZ_FIPS",	"CZ_NAME","SOURCE","BEGIN_LAT","BEGIN_LON","END_LAT","END_LON")
StormDetails_Limit<- Stormdetails[myvars]
StormDetails_Limit
```

#Convert the beginning and ending dates to a “date-time” class (there should be one column for the beginning date-time and one for the ending date-time) (5 points)
#Unite Begining Date and Time
```{r}
library(tidyr)
library(lubridate)
library(stringi)
newdata1 <- unite(StormDetails_Limit, BEGIN_DATE, BEGIN_YEARMONTH,BEGIN_DAY, sep="")
newdata2 <- unite(newdata1, BEGIN_DATE_TIME, BEGIN_DATE, BEGIN_TIME, sep=" ")
head(newdata2)
```
#Unite End Date and Time
```{r}
library(tidyr)
library(lubridate)
newdata3 <- unite(newdata2, END_DATE, END_YEARMONTH,END_DAY, sep="")
newdata4 <- unite(newdata3, END_DATE_TIME, END_DATE, END_TIME, sep=" ")
head(newdata4)
```

#Change state and county names to title case (e.g., “New Jersey” instead of “NEW JERSEY”) (5 points)
```{r}
library(stringr)
state.data <- StormDetails_Limit[, c("STATE","CZ_NAME")]
state.data$state_title_case=str_to_title(state.data$STATE)
state.data$county_title_case=str_to_title(state.data$CZ_NAME)
state.data
```

#Limit to the events listed by county FIPS (CZ_TYPE of “C”) and then remove the CZ_TYPE column (5 points)
```{r}
StormDetails_CType= filter(newdata4, CZ_TYPE == "C")
StormDetails_CType$CZ_TYPE <- NULL
head(StormDetails_CType)
```

#Pad the state and county FIPS with a “0” at the beginning (hint: there’s a function in stringr to do this) and then unite the two columns to make one fips column with the 5-digit county FIPS code (5 points)
```{r}
STATE_FIPS <- str_pad(StormDetails_CType$STATE_FIPS, width= 3,side ="left", pad ="0")
CZ_FIPS <- str_pad(StormDetails_CType$CZ_FIPS, width= 3, side ="left",pad ="0")
newdata5 <- unite(StormDetails_CType, "FIPS","STATE_FIPS" , "CZ_FIPS", sep="")
head(newdata5)
```

#Change all the column names to lower case (you may want to try the rename_all function for this) (5 points)
```{r}
rename_all(newdata5, tolower)
```

#There is data that comes with R on U.S. states (data("state")). Use that to create a dataframe with the state name, area, and region
```{r}
data("state")
us_state_info<-data.frame(state=state.name, region=state.region, area=state.area)
us_state_info$state_upper_case=str_to_upper(us_state_info$state)
us_state_info$state <- NULL
us_state_info <- rename(.data=us_state_info, state= state_upper_case)
us_state_info

```

#Create a dataframe with the number of events per state in the year of your birth. Merge in the state information dataframe you just created. Remove any states that are not in the state information dataframe. (5 points)
#number of events per state in the year of your birth
```{r}
newset<- data.frame(table(Stormdetails$STATE))
newset1<-rename(newset, c("state"="Var1", "num_events"="Freq"))
newset1
```
#Merge in the state information dataframe 
```{r}
merged <- merge(x=newset1,y=us_state_info,by.x="state", by.y="state")
head(merged)
```

#PLOT
```{r}
library(ggplot2)

storm_plot <- ggplot(merged, aes(x=area,y=num_events)) + geom_point(aes(col = region))+ labs(x="Land area (square miles)", y= "# of storm events in 1992")
storm_plot                                                                                   
```

