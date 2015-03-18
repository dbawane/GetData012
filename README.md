
##Peer Assessments - Getting and Cleaning Data Course Project

###Download and Unzip data

```{r}
file_url <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
```


###Check or create data directory

```{r}
if (!file.exists("data")) {
      dir.create("data")
}
```
###Check for file or download

```{r}
if (!file.exists("data/UCI HAR Dataset.zip")) {
      
      download.file(file_url, destfile="data/UCI HAR Dataset.zip", method="curl")
      
}
```

###Check file is unziped otherwise upzip it

```{r}
if (!file.exists("UCI HAR Dataset")) {
      unzip("data/UCI HAR Dataset.zip")
}
```
####===================================================
#### Read training and test data
####===================================================

### training data

```{r}
X_train <- read.table("UCI HAR Dataset/train/X_train.txt")
Y_train <- read.table("UCI HAR Dataset/train/y_train.txt")
Subject_train <- read.table("UCI HAR Dataset/train/subject_train.txt")

```
### test data

```{r}
X_test <- read.table("UCI HAR Dataset/test/X_test.txt")
Y_test <- read.table("UCI HAR Dataset/test/y_test.txt")
Subject_test <- read.table("UCI HAR Dataset/test/subject_test.txt")
```
##### Use View function to check the data
##### The training set contains 7352 observations
##### The test set contains 2947 observations
##### X contains 561 variable, Y and subject contains 1 variable each

###1. Merges the training and the test sets to create one data set.

#### cbind the train Set

```{r}
train <- cbind(X_train, Y_train, Subject_train)
```

#### cbind the test Set

```{r}
test <- cbind(X_test, Y_test, Subject_test)
```

#### rbind the train and test set

```{r}
tdata <- rbind( train, test) 
```

#### check dim to verify

```{r}
dim(tdata)
```

#### Give name to columns

#### read features.txt, it has colums names

```{r}
features <- read.table("UCI HAR Dataset/features.txt")
```

#### Add Activity and Subject

```{r}
namecol <- c(as.vector(features$V2), "Activity", "Subject")
colnames(tdata) <- namecol # check with head(tdata,1)
```

###2. Extracts only the measurements on the mean and standard deviation for each measurement

```{r}
reqcol <- grepl("mean\\()|std\\()|Subject|Activity", namecol)
req_data = tdata[, reqcol] #check head(req_data)
```
###3. Uses descriptive activity names to name the activities in the data set

#### Read activity labels

```{r}
activity_labels <- read.table("UCI HAR Dataset/activity_labels.txt")
req_data$Activity <- factor(req_data$Activity, levels=activity_labels$V1, labels=activity_labels$V2)
```

###4. Appropriately labels the data set with descriptive variable names.

```{r}
disc_name <- colnames(req_data)
#disc_name <- gsub("\\(\\)", "", name_col)
disc_name <- gsub("Acc", "-acceleration", disc_name)
disc_name <- gsub("Mag", "-magnitude", disc_name)
disc_name <- gsub("(Jerk|Gyro)", "-\\1", disc_name)
disc_name <- gsub("BodyBody", "Body", disc_name)
disc_name <- gsub("^t(.*)$", "\\1-time", disc_name)
disc_name <- gsub("^f(.*)$", "\\1-freq", disc_name)
disc_name <- tolower(disc_name)
colnames(req_data) <- disc_name
```
###5. From the data set in step 4, creates a second,independent tidy data set
###with the average of each variable for each activity and each subject.

```{r}
library(plyr)
tidy_data_set <- ddply(req_data, c("subject", "activity"), numcolwise(mean))
```
### Save the data into the file

```{r}
write.table(tidy_data_set, file="tidy_data_set.txt", row.name=FALSE)
```


