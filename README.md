---
title: "README"
author: "Beata-C"
date: "4/29/2020"
output: word_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

To read the final file use:

```{r}
tidy_data <- read.table("/data/TidyDataUCIHAR.txt", check.names = FALSE, stringsAsFactors = FALSE, header = TRUE)
```

The output file is "/data/TidyDataUCIHAR.txt"


The script begins here:

```{r}
run_analysis <- function(zip_file){
```

Save the link to download file

```{r}
zip_file <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
```

create a data directory

```{r}
if(!file.exists("data")){dir.create("data")}
```

create a place for zip file in the directory

```{r}
dest <- "C:/Users/Craftemall/Desktop/datasciencecoursera/Getting and Cleaning Data Course Project/data/data.zip"
```

download zip file

```{r}
download.file(url = zip_file, destfile = dest, mode = "wb")
```

recorded download date Mon Apr 27 15:48:24 2020

unzip the data into our directory

```{r}
unzip(zipfile = dest, exdir = "/data")
```

1. Merge the training and test sets to create one data set:

1.a Create a full training data set:

read features (column names for both sets)

```{r}
features <- read.table("/data/UCI HAR Dataset/features.txt")
```

read training data adding column names from features file

```{r}
training_set <- read.table("/data/UCI HAR Dataset/train/X_train.txt", col.names = as.vector(features[,2]))
```

add training labels (Activity) and subject labels (Subject)

```{r}
training_labels <- read.table("/data/UCI HAR Dataset/train/y_train.txt")
training_subject <- read.table("/data/UCI HAR Dataset/train/subject_train.txt")
library(dplyr)
training_set <- training_set %>% mutate(Activity = training_labels$V1) %>% mutate(Subject = training_subject$V1)
```

1.b Create a full test data set:

read test data adding column names from features file

```{r}
test_set <- read.table("/data/UCI HAR Dataset/test/X_test.txt", col.names = as.vector(features[,2]))
```

add training labels (Activity) and subject labels (Subject)

```{r}
test_labels <- read.table("/data/UCI HAR Dataset/test/y_test.txt")
test_subject <- read.table("/data/UCI HAR Dataset/test/subject_test.txt")
test_set <- test_set %>% mutate(Activity = test_labels$V1) %>% mutate(Subject = test_subject$V1)
```

1.c Merge test_set and training_set

```{r}
library(data.table)
merged_data <- merge(training_set, test_set, all = TRUE)
```

2. Extract measurements on the mean and standard deviation (adding Activity and Subject)

```{r}
extracted_data <- select(merged_data, contains(c("Activity","Subject","mean","std")))
```

3. Use descriptive activity names to name the activities in the data set

```{r}
activity_labels <- read.table("/data/UCI HAR Dataset/activity_labels.txt")
extracted_data$Activity <- extracted_data$Activity %>% {gsub("1", "Walking", .)} %>%
  {gsub("2", "WalkingUpstairs", .)} %>% {gsub("3", "WalkingDownstairs", .)} %>%
  {gsub("4", "Sitting", .)} %>% {gsub("5", "Standing", .)} %>% {gsub("6", "Laying", .)}
```

4. Appropriately label the data set with descriptive variable names:

```{r}
names(extracted_data) <- names(extracted_data) %>% {gsub("BodyAcc", "BodyAcceleration", .)} %>% 
  {gsub("GravityAcc", "GravityAcceleration", .)} %>% {gsub(".mean", "(mean)", .)} %>% 
  {gsub(".std", "(std)", .)} %>% {gsub("^t", "Time", .)} %>% 
  {gsub("^f", "Frequency", .)} %>% {gsub("Mean", "(mean)", .)} %>% 
  {gsub("Mag", "Magnitude", .)} %>% {gsub(")Freq", ")Frequency", .)} %>% 
  {gsub("angle.", "(angle)", .)} %>% {gsub("tB", "TimeB", .)} %>% 
  {gsub(".gravity", "AndGravity", .)} %>% {gsub("\\.*", "", .)}
```

5. From data set in step 4, create a second, independent tidy data set with the average of each variable for each activity and each subject:

```{r}
tidy_data_UCI_HAR <- extracted_data %>% group_by(Subject, Activity) %>% summarise_all(mean) %>%
  unite(ActivityAndSubjectNumber, Activity, Subject, sep = "")
```

Create output file:

```{r}
output_file <- "/data/TidyDataUCIHAR.txt"
write.table(tidy_data_UCI_HAR, output_file)
```


}

