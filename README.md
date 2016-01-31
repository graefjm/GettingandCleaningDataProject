#README
###*Sunday January 31, 2016*

##Getting and Cleaning Data Course Project Background

The purpose of this project is to demonstrate the ability to collect, work with, and clean a data set. The goal is to prepare tidy data that can be used for later analysis. This project includes: 

1. A tidy data set 
2. A link to a Github repository with the scripts for performing the analysis 
3. A code book that describes the variables, the data, and transformations performed to clean up the data called CodeBook.md. Also a README.md that explains how all of the scripts work and how they are connected.

One of the most exciting areas in all of data science right now is wearable computing. Companies like Fitbit, Nike, and Jawbone Up are racing to develop the most advanced algorithms to attract new users. The data linked to from the course website represent data collected from the accelerometers from the Samsung Galaxy S smartphone. A full description is available at the site where the data was obtained:

[http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones](http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones)

Here is the data for the project:

[https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip](https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip)

The R script called **run_analysis.R** does the following.

1. Merges the training and the test sets to create one data set.
2. Extracts only the measurements on the mean and standard deviation for each measurement.
3. Uses descriptive activity names to name the activities in the data set
4. Appropriately labels the data set with descriptive variable names.
5. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.

##What is in this repository
* **CodeBook.md** - information on the data, variables, and transformations performed on the information
* **README.md** - includes the scripts used and how they work
* **run_analysis.R** - the full script used to create the tidy dataset

##Getting Started
###The Setup
First, make sure you download and unzip the data from the link in the 'Project Background' section and put it into your working directory. There should be one folder named **UCI HAR Dataset** that stores all of the information needed.

###Loading Libraries
The libraries used are **'data.table'** and **'dplyr'**.
    
    library(data.table)
    library(dplyr)

###Reading Data
Next read the data in and store them into variables with names of your choice. Keep it simple so you can use these names again later.

    featureNames <- read.table("UCI HAR Dataset/features.txt")
    activityLabels <- read.table("UCI HAR Dataset/activity_labels.txt", header = FALSE)
    
    subjectTrain <- read.table("UCI HAR Dataset/train/subject_train.txt", header = FALSE)
    activityTrain <- read.table("UCI HAR Dataset/train/y_train.txt", header = FALSE)
    featuresTrain <- read.table("UCI HAR Dataset/train/X_train.txt", header = FALSE)
    
    subjectTest <- read.table("UCI HAR Dataset/test/subject_test.txt", header = FALSE)
    activityTest <- read.table("UCI HAR Dataset/test/y_test.txt", header = FALSE)
    featuresTest <- read.table("UCI HAR Dataset/test/X_test.txt", header = FALSE)

## Part 1 - Merge the training and the test sets to create one dataset
The following code will name and combine the data sets into one.

    subject <- rbind(subjectTrain, subjectTest)
    activity <- rbind(activityTrain, activityTest)
    features <- rbind(featuresTrain, featuresTest)
    
    colnames(features) <- t(featureNames[2])
    colnames(activity) <- "Activity"
    colnames(subject) <- "Subject"
    
    completeData <- cbind(features,activity,subject)

## Part 2 - Extract only the measurements on the mean and standard deviation for each measurement.
This code will extract the mean and standard deviation and allow use to see the dimensions of each.

    columnsWithMeanSTD <- grep(".*Mean.*|.*Std.*", names(completeData), ignore.case=TRUE)

    requiredColumns <- c(columnsWithMeanSTD, 562, 563)
    dim(completeData)

    extractedData <- completeData[,requiredColumns]
    dim(extractedData)

## Part 3 - Use descriptive activity names to name the activities in the dataset

    extractedData$Activity <- as.character(extractedData$Activity)
    for (i in 1:6){
    extractedData$Activity[extractedData$Activity == i] <- as.character(activityLabels[i,2])
    }
    
    extractedData$Activity <- as.factor(extractedData$Activity)

## Part 4 - Label the data set with descriptive variable names
Here we will first call up all of our variable names and look to see how to create names with clearer meaning. For example we could take 't' and turn it into 'Time or 'Mag' into Magnitude. To make it tidy, it should be easy to understand.

    names(extractedData)

    names(extractedData)<-gsub("Acc", "Accelerometer", names(extractedData))
    names(extractedData)<-gsub("Gyro", "Gyroscope", names(extractedData))
    names(extractedData)<-gsub("BodyBody", "Body", names(extractedData))
    names(extractedData)<-gsub("Mag", "Magnitude", names(extractedData))
    names(extractedData)<-gsub("^t", "Time", names(extractedData))
    names(extractedData)<-gsub("^f", "Frequency", names(extractedData))
    names(extractedData)<-gsub("tBody", "TimeBody", names(extractedData))
    names(extractedData)<-gsub("-mean()", "Mean", names(extractedData), ignore.case = TRUE)
    names(extractedData)<-gsub("-std()", "StandardDeviation", names(extractedData), ignore.case = TRUE)
    names(extractedData)<-gsub("-freq()", "Frequency", names(extractedData), ignore.case = TRUE)
    names(extractedData)<-gsub("angle", "Angle", names(extractedData))
    names(extractedData)<-gsub("gravity", "Gravity", names(extractedData))

## Part 5 - From the dataset in part 4, create a second, tidy dataset with the average of each variable for each activity and subject

    extractedData$Subject <- as.factor(extractedData$Subject)
    extractedData <- data.table(extractedData)\

    tidyData <- aggregate(. ~Subject + Activity, extractedData, mean)
    tidyData <- tidyData[order(tidyData$Subject,tidyData$Activity),]
    write.table(tidyData, file = "Tidy.txt", row.names = FALSE)

You should now have a tidy dataset in your working directory named **"Tidy.txt"**.